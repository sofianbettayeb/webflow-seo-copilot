---
name: aeo-onboard
version: "1.0"
description: |
  Bootstrap a brand in AEO Copilot end-to-end from a URL. Analyzes the site, creates the brand, runs the technical audit, designs 5 buyer-journey topics with listicle-style prompts, and triggers the first tracking cycle. Hands off to /ai-visibility for the baseline report once tracking completes.
  Triggers: aeo onboard, onboard brand, bootstrap aeo, set up aeo brand, new brand aeo copilot, create aeo brand, kickoff aeo, aeo setup.
  Requires: AEO Copilot MCP server. Optional: Webflow MCP for confirming live pages.
  Workflow: Discover → Confirm → Create → Audit → Design Topics → Create Topics → Run Tracking → Log.
  Command: /aeo-onboard {url}
---

# AEO Onboard Skill

End-to-end onboarding for a new brand in AEO Copilot. Takes a single URL and produces a tracked brand with topics, prompts, a tech audit, and a first tracking cycle running. The next step after this skill is `/ai-visibility` to render the client baseline once the tracking cycle finishes.

This skill **writes to AEO Copilot**. Every state-changing call sits behind an explicit approval gate. No silent writes, ever.

## Prerequisites

- **Required**: AEO Copilot MCP server. Exposes `list_brands`, `create_brand`, `scan_brand`, `list_topics`, `create_topic`, `add_prompts`, `run_brand_prompts`, `get_results`, `get_insights`, `get_recommendations`. Repo: https://github.com/sofianbettayeb/aeo-copilot-mcp.
- **Optional**: Webflow MCP — used only to confirm live URLs for the `pages` array on each topic. Skill works without it.

## Workflow Overview

```
DISCOVER → CONFIRM METADATA → CREATE BRAND → TECH AUDIT → DESIGN TOPICS → CREATE TOPICS & PROMPTS → RUN TRACKING → LOG
```

---

## Critical Guards

⚡ GUARD — **AEO Copilot MCP unavailable**: stop. "AEO Onboard requires the AEO Copilot MCP. Connect `aeo-copilot-mcp` and try again."

⚡ GUARD — **No URL passed**: ask. "What's the brand's URL? I won't infer this." Do not proceed without a URL.

⚡ GUARD — **Brand already exists** (domain match in `list_brands`): present 3 options.
1. Add topics to the existing brand (default).
2. Create a new brand with a `-2` suffix (rename later in dashboard).
3. Abort.

⚡ GUARD — **402 from `create_brand`**: stop. "Plan brand limit reached on AEO Copilot. Upgrade or remove a brand to proceed." Log abort reason.

⚡ GUARD — **402 from `add_prompts`**: stop the prompt-creation loop. Record how many prompts landed before the cap. Log: "Prompt cap hit after {N}/{M} prompts on topic `{name}`."

⚡ GUARD — **`run_brand_prompts` always gates**: never call without explicit approval. Phrase verbatim: "About to execute {N} prompts across 4 LLMs (ChatGPT, Claude, Perplexity, Google AIO). This consumes credits. Proceed? (y/N)"

⚡ GUARD — **User aborts** ("stop", "cancel", "nevermind"): confirm "Stop the workflow? Progress will be lost." If confirmed: exit cleanly, write activity-log row noting which phase was reached.

---

## Phase 0: DISCOVER

### 0.1 MCP Discovery

Search at skill start:
- AEO Copilot MCP (required): `+aeo copilot brands`. Missing → stop (see guard).
- Webflow MCP (optional): `+webflow data cms`. Note availability.

### 0.2 Parse URL

Extract `brandDomain` from the input URL:
- Strip protocol (`http://`, `https://`)
- Strip `www.`
- Strip trailing slash
- Lowercase

Store: `inputUrl` (the full URL passed in), `brandDomain`.

### 0.3 Idempotency Check

Call `list_brands`. Check if any brand has a `website` whose domain matches `brandDomain`. If a match exists, surface the existing brand and offer the 3-option menu (see Brand-already-exists guard).

### 0.4 Activity Log Check

Read `.claude/reports/{brandDomain}/activity-log.md` if present. Surface the last 3 rows. If a previous `/aeo-onboard` row exists, note it: "This brand was onboarded on {date}. Re-running will create new topics/prompts." Continue if user confirms.

### 0.5 Site Analysis (read-only)

WebFetch the homepage and `/about` (or `/about-us`, fall back gracefully on 404). Extract signals to seed Phase 1:
- Brand name (from `<title>`, `og:site_name`, or H1)
- Industry (from hero copy, meta description)
- Products (from nav, hero, product pages — list 2–6 concrete product/service names)
- Likely competitors (from "vs" pages, comparison content, customer testimonials mentioning alternatives)

Do not invent. If a signal isn't on the page, mark it `null` and ask the user in Phase 1.

### 0.6 Set Paths & Date

- `report_date` = today (YYYY-MM-DD)
- `tech_audit_path` = `.claude/reports/{brandDomain}/aeo-tech-audit-{report_date}.json`
- `activity_log_path` = `.claude/reports/{brandDomain}/activity-log.md`

`mkdir -p .claude/reports/{brandDomain}/`.

---

## Phase 1: CONFIRM BRAND METADATA (approval gate)

Present a draft brand record to the user. Show what was extracted from the site and ask for edits.

```
Proposed brand record:

  name:         {extracted name}
  website:      {inputUrl}
  industry:     {extracted industry, or "unknown — please specify"}
  products:     {extracted products, or empty list}
  competitors:  {extracted competitors, or empty list}

Edit any field, or type 'ok' to proceed.
```

Loop until user types `ok`. Never call `create_brand` without this confirmation.

Apply voice rules from `references/voice-rules.md` to the `industry` field — short, direct, no filler ("Industrial AI for CAM programming" not "Innovative AI-powered solutions for the CAM programming industry").

---

## Phase 2: CREATE BRAND

Call `create_brand({ name, website, industry, products, competitors })`.

- Capture `brandId` from the response. Save it for every subsequent call.
- 402 → see guard.
- Other errors → surface the error, log abort, stop.

One-line confirmation: "Brand created: `{name}` (id: {brandId})."

---

## Phase 3: TECH AUDIT

### 3.1 Run scan

Call `scan_brand({ id: brandId })`. This pulls schema, sitemap, llms.txt, robots, structured data, and other technical AEO signals.

Save the raw response to `tech_audit_path` as pretty-printed JSON.

### 3.2 Surface findings (evidence-first)

Walk through the response and surface findings to the user. **Never prescriptive**, always descriptive. Same rule as `/ai-visibility` Phase 3.

Examples:
- ✅ "No `llms.txt` detected at `https://{brandDomain}/llms.txt`."
- ✅ "Sitemap at `/sitemap.xml` lists 47 URLs. 3 return 404."
- ✅ "Schema.org markup found on the homepage: `Organization`. No `Product` or `FAQPage` schemas detected on key templates."
- ❌ "You should add an llms.txt file to improve AEO." (prescriptive — drop)
- ❌ "Consider implementing FAQPage schema." (prescriptive — drop)

Apply `references/voice-rules.md` to all surfaced text.

Group findings under three short headers:
- **Crawlability** (sitemap, robots, llms.txt)
- **Structured data** (schema types found and missing)
- **Indexability signals** (canonical, meta robots, hreflang if multi-locale)

If a category has fewer than 2 findings, collapse into one line. Never output a header with nothing under it.

End the section with: "Full audit JSON saved to `{tech_audit_path}`."

---

## Phase 4: DESIGN TOPICS (approval gate)

Generate 5 topics covering the buyer journey. Each topic = `name`, `description`, `keywords[]`, `pages[]`, `prompts[]` (5–6 listicle prompts).

### 4.1 Topic taxonomy (5 axes)

Use the framework in `references/topic-frames.md`. Every brand should produce one topic per axis:

1. **Broad category** — the umbrella term for the brand's primary use case. Top-of-funnel demand.
2. **Primary integration ecosystem** — the dominant tool/platform the brand plugs into.
3. **Secondary integration ecosystems** — adjacent stacks the brand also serves (combine 2–4 in one topic if they share intent).
4. **Industry vertical** — the highest-value sector(s) the brand serves.
5. **Outcome / business problem** — the operational result buyers care about (productivity, quoting, throughput, compliance, etc.).

If the brand genuinely doesn't fit one of these axes (e.g., no integration ecosystem because it's a standalone B2C product), substitute with a second outcome topic or a use-case topic. Document the substitution in the topic description so it's traceable.

### 4.2 Prompt design

Each topic gets 5–6 listicle-style prompts. Templates in `references/prompt-frames.md`. Span:
- 1 broad list ("List the best {category} tools")
- 1 year-specific ("Top {N} {category} tools in 2026")
- 1 persona-targeted ("Best {category} for {persona}")
- 1 integration/use-case-specific ("Which {category} works with {integration}?")
- 1 competitor/alternative ("Best alternatives to {competitor}")
- Optional 6th: industry- or outcome-framed

Vary phrasing. Avoid identical sentence structures. Apply `references/voice-rules.md`.

### 4.3 Pages array

For each topic, attach 1–3 live URLs from the brand's site that map to that topic's intent (extracted in Phase 0.5 or from the user). If Webflow MCP is connected, you can verify URLs are live and indexable. If not, use the URLs the homepage links to.

### 4.4 Present for approval

Show the user all 5 topics in a compact format:

```
Topic 1: {name} ({axis})
  Description: {description}
  Keywords: {3–5 keywords}
  Pages: {urls}
  Prompts ({N}):
    1. {prompt}
    2. {prompt}
    ...

Topic 2: ...
```

Ask: "Edit any topic, or type 'ok' to create all 5." Loop on edits.

Do not proceed to Phase 5 without explicit `ok`.

---

## Phase 5: CREATE TOPICS & PROMPTS

For each approved topic:

1. Call `create_topic({ id: brandId, name, description, pages, keywords })`. Capture `topicId`.
2. Call `add_prompts({ id: brandId, topicId, prompts })`.
3. One-line confirmation: "Topic `{name}` created with {N} prompts."

If `add_prompts` returns 402 → see guard. Stop the loop, but keep already-created topics. Log how many topics/prompts landed.

After all 5 topics:
```
✅ {N} topics created. {M} prompts added.
```

---

## Phase 6: RUN TRACKING (approval gate)

Show the cost preview verbatim:

```
About to execute {M} prompts across 4 LLMs (ChatGPT, Claude, Perplexity, Google AIO). This consumes credits. Proceed? (y/N)
```

On `y` → `run_brand_prompts({ id: brandId })` (no `topicId` to run all topics).
On `N` → skip tracking. The brand, topics, and prompts already exist; the user can run tracking from the dashboard later.

After triggering:
```
Tracking cycle started. Results land in AEO Copilot in ~5–15 minutes depending on prompt count and engine load.

Next: once the cycle completes, run /ai-visibility to render the baseline report for {brandName}.
```

---

## Phase 7: ACTIVITY LOG

Append to `activity_log_path`. Create the file with the standard header if it doesn't exist:

```markdown
# Activity Log — {brandDomain}

| Date | Skill | Summary |
|------|-------|---------|
```

Append one row:

```
| YYYY-MM-DD | /aeo-onboard | Created brand {name}. Tech audit: schema={x}, sitemap={y}, llms.txt={z}. {N} topics, {M} prompts. Tracking cycle started. |
```

If the user aborted at any phase, log the row anyway with the reason:
```
| YYYY-MM-DD | /aeo-onboard | Aborted at Phase {X}: {reason}. Brand {created/not created}. |
```

---

## Final message

```
✅ AEO onboarding complete for {brandName} ({brandDomain}).

  Brand ID:     {brandId}
  Topics:       {N}
  Prompts:      {M}
  Tech audit:   {tech_audit_path}
  Tracking:     {started | skipped}

Next: /ai-visibility to render the baseline report once tracking completes.
```

---

## References

- `references/voice-rules.md` — Humanizer rules. Apply to topic descriptions and audit findings.
- `references/topic-frames.md` — 5-axis topic taxonomy with worked examples per industry.
- `references/prompt-frames.md` — Listicle prompt templates.

## Integration with Other Skills

| After this skill | Run | Why |
|---|---|---|
| Tracking cycle finishes | `/ai-visibility` | Render the client baseline report. |
| Tech audit shows schema/llms.txt gaps | `/aeo-optimize` | Page-level AEO rewrite. |
| Topic gaps surface in baseline | `/keywords-opportunity:discover` | Find new content topics. |
