# /write-blog

Create a new SEO and AEO-optimized blog draft in Webflow CMS. Follows project conventions from `/getting-started`, applies the AEO dimensions from `/aeo-optimize`, runs the AI Writing Tell-Tales check from `/refresh-content`, invokes the `/humanizer` skill before writing to Webflow, and includes an open revision loop until the user explicitly approves the draft.

---

## Trigger phrases

- `/write-blog`
- "write a blog post"
- "create a Webflow blog draft"
- "draft an SEO article"
- "draft an AEO article"

---

## MCP requirements

| Tool | Required | Used for |
|------|----------|----------|
| Webflow MCP | Yes | Create CMS item draft, write fields, set SEO fields |
| Keywords Everywhere MCP | Strongly recommended | Keyword expansion — related keywords, long-tail, questions |
| Google Search Console MCP | Optional | Validate intent and query patterns if a matching page already exists |

If Keywords Everywhere is not available, proceed with a GSC-only or reasoning-only approach and warn that volume data is missing.

---

## Modes

| Mode | Command | Behavior |
|------|---------|----------|
| Default | `/write-blog` | Full workflow: brief → keywords → outline → draft → Tell-Tales → /humanizer → revision loop → AEO audit → verification → Webflow draft |
| Outline only | `/write-blog:outline` | Stops after outline approval. No Webflow changes. |
| Auto-publish | `/write-blog:publish` | Creates CMS draft and publishes immediately after user approval. |

---

## Required inputs (always ask if missing)

- **Primary keyword** — provided by user
- **Search intent** — ask explicitly: informational / commercial investigation / product-focused / mixed
- **Article objective** — rank, educate, capture leads, support positioning
- **Target collection** — confirm default blog collection or ask
- **Primary CTA** — optional but recommended
- **Internal links to include** — optional
- **Quote, stat, or personal experience to feature** — ask explicitly, do not treat as optional (see Phase 1)

## Optional inputs (strongly encourage)

- Additional user insights, opinions, research, frameworks, personal examples
- Audience notes (if not already in config)
- Competitors to reference (if not already in config)

---

## Global config reuse

At start, load `.claude/seo-copilot-config.json` created by `/getting-started`. Use it to enforce:

- Brand voice and tone
- Audience and expertise level
- Markets and language preferences
- SEO constraints (meta title format, internal link preferences, competitors)
- AEO constraints (FAQ strategy, citation goals, E-E-A-T signals)
- Content preferences (headings, lists, CTAs)

If the config file is missing, proceed with defaults and recommend running `/getting-started` first.

---

## Skill reuse

### `/keywords-opportunity` — optional, use as a support step

Run when:
- The user wants "the best keyword to write about"
- The user provides a broad seed and wants expansion with evidence
- The site has GSC data and the user wants to prioritize ROI

How to use the result:
- Pick 1 primary keyword (user-confirmed)
- Extract secondary keywords (semantic + long-tail)
- Extract question keywords for FAQ and definition blocks

### `/humanizer` — mandatory (Phase 4.6)

Invoke the `/humanizer` skill on the full draft after the Tell-Tales check. Do not apply humanizer rules manually — call the skill so the user sees the pass output.

### `/aeo-optimize` rubric — mandatory (Phase 5)

Apply the same AEO dimensions and checks as `/aeo-optimize`, adapted for pre-publish drafting:
- Use the user-provided primary keyword as the primary query
- Run the AEO audit pass on the final approved draft before writing to Webflow
- Output a scored table with severity and auto-fix vs approval-required flags

The goal is content that would pass `/aeo-optimize:audit` with minimal changes after publication.

---

## Workflow

### Phase 0 — Setup and guards

1. Verify Webflow MCP is connected. If missing, stop and tell the user.
2. Load `.claude/seo-copilot-config.json` if available.
3. Read `.claude/reports/{domain}/activity-log.md` and surface any recent work on the same topic cluster to avoid duplicates.
4. **Map the target collection schema.** Fetch the collection fields and check for:
   - SEO title field (e.g. `seo-title`, `meta-title`) — warn if missing, note that meta title won't be set
   - Meta description field — warn if missing
   - Image fields (e.g. `main-image`, `thumbnail-image`) — note if present, flag as required before publishing
   - FAQ fields — note if present and how many (shapes Phase 5 FAQ output)
   - Schema markup field — note if present

   Surface a one-line schema summary: *"Collection has: meta description ✓, FAQ fields (5) ✓, main image ✓, no SEO title field ⚠"*

---

### Phase 1 — Brief intake (human input required)

Ask only for missing required fields. Always ask ALL of the following — do not skip any:

- Primary keyword
- Search intent (informational / commercial investigation / product-focused / mixed)
- Article objective (rank / educate / capture leads / support positioning)
- Any specific angle, constraints, or things to avoid
- Internal links and CTA
- **Quote, personal stat, or first-hand experience to feature** — ask this explicitly: *"Do you have a quote, specific number, or personal experience you want featured prominently? This is what keeps the article from sounding generic."* Do not bury this as optional.

Stop and wait for user confirmation before proceeding.

---

### Phase 2 — Keyword expansion (best-effort)

1. Expand with Keywords Everywhere:
   - Related keywords
   - Long-tail variations
   - Question queries ("how to", "what is", "best way to")

2. If GSC is available and the user provides a related existing URL, pull page queries as supporting context (do not require it).

3. Propose:
   - Secondary keyword list (8–20 terms)
   - 3–8 FAQ questions derived from question keywords
   - Recommended content format (checklist, how-to, comparison, glossary) aligned with intent

If KE is not connected, provide a smaller keyword set based on reasoning and user input, and warn about missing volume evidence.

---

### Phase 3 — Outline proposal (human input required)

Produce an outline with:
- H1 (includes primary keyword, sounds natural — not stuffed)
- Intro intent confirmation (what the reader gets in 2 sentences)
- Definition block (short, direct — for AEO)
- H2 sections ordered by user intent
- Where a table helps (comparison, checklist, decision matrix)
- Proposed FAQ questions
- Suggested internal link placements

Ask the user to approve:
- The outline
- The angle
- The CTA placement

**The user may replace the outline entirely with their own structure. Accept it and proceed.** Do not proceed to draft until approved.

---

### Phase 4 — Draft writing

#### Writing style rules

- Short sentences. Short paragraphs (2–3 lines max).
- Concrete explanations over abstract claims.
- No hype, no filler, no AI vocabulary.
- Follow the brand voice and audience level from config.
- Use user-provided quotes, stats, and examples — inject them prominently, not as afterthoughts.

#### Default structure

- **Title (H1)** — includes primary keyword, reads naturally
- **Introduction** — 80–140 words, states clearly what the reader will learn
- **Definition section** — short and direct (feeds AEO answer boxes)
- **Main sections (H2/H3)**
- **At least one scannable element** — checklist, bullets, or comparison table
- **FAQ** — 3–6 Q&As, concise answers (2–4 sentences each)
- **Conclusion** — 60–120 words
- **CTA** — aligned with the article objective

---

### Phase 4.5 — AI Writing Tell-Tales check (mandatory)

Before invoking the humanizer, scan the full draft and fix all violations automatically.

#### Structural patterns to AVOID

- Emoji bullets (✅ ❌ 🚀) — dead giveaway of AI content
- Every section having exactly 3–5 bullet points
- Perfect parallel structure in all headings ("Webflow Designers", "Webflow Developers", "Webflow Marketers")
- "Key Takeaways" as the opening section
- "Final Thoughts" or "Conclusion" headers
- Numbered lists for things that don't need sequence

#### Filler phrases to NEVER USE

- "Here's what it offers:"
- "Let's dive in" / "Let's explore"
- "Whether you're a [X], [Y], or [Z]"
- "In today's [industry]..."
- "From [X] to [Y], [product] has you covered"
- "everything you need to [verb]"
- "takes it to the next level"
- "game-changer" / "revolutionary" / "cutting-edge"
- "In this article, we'll..."
- "Without further ado"

#### Formatting to AVOID

- Excessive em-dashes (—) — **hard limit: 2–3 per article**. Count them. If you have more than 3, replace the rest with commas, colons, or periods. This check is mandatory before Phase 4.6 — do not rely on the humanizer to catch a count you haven't verified.
- Bold on every key phrase
- Every paragraph starting with the product/brand name
- Perfectly uniform paragraph lengths

#### Tables — HTML required

If the draft includes a comparison table or data table, it **must be written as valid HTML** using `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>` tags. Plain text or markdown tables will not render in Webflow's rich text field. Write the table as HTML directly in the draft.

#### Add human signals (actively inject these)

- First-person experience: "I've tested this on 50+ sites"
- Specific numbers: "saves ~15 minutes per page" not "saves time"
- Honest limitations: "It won't help with backlinks or technical crawl issues"
- Casual asides: "honestly, this is the part I use most"
- Opinions with edge: "other tools charge $30/month for less"
- Imperfect structure: vary bullet counts (2 here, 6 there)
- Natural transitions, not formulaic ones

---

### Phase 4.6 — Humanizer pass (mandatory)

**Invoke the `/humanizer` skill on the full draft.** Do not apply humanizer rules manually — call the skill so the output is explicit and visible.

The `/humanizer` skill will:
- Remove AI vocabulary (seamless, robust, delve, comprehensive, leverage, harness, empower, transformative, streamline, cutting-edge, game-changing, innovative)
- Enforce em-dash limit (max 2–3 per article)
- Break rule-of-three padding
- Remove inflated symbolism and vague metaphors
- Eliminate excessive conjunctive phrases (Furthermore, Moreover, In addition to, It is worth noting that)
- Remove negative parallelisms (not only X, but also Y)
- Strip promotional superlatives without evidence
- Enforce short, direct sentences

Present the humanized draft output to the user. Then move immediately to the revision loop (Phase 4.7) — do not skip to the AEO audit.

---

### Phase 4.7 — Revision loop (human input required)

**This phase stays open until the user explicitly approves the draft.**

Present the humanized draft and ask:
*"Does this read the way you'd write it? List any changes — content, tone, structure, missing points, anything. I'll revise and come back. Reply 'approved' when it's ready for the AEO audit."*

For each round of feedback:
1. Apply all requested changes
2. Re-run Tell-Tales check on changed sections only (do not re-run full Phase 4.5)
3. Re-present the updated draft with a summary of what changed
4. Ask again: *"Any further changes, or approved?"*

Do not proceed to Phase 5 until the user says "approved" or equivalent confirmation.

**Typical revision rounds: 2–5. Do not treat the first feedback round as the last.**

---

### Phase 5 — AEO audit pass

Evaluate the approved draft across the `/aeo-optimize` dimensions. Output a scored table with severity:

| Dimension | Status | Severity | Action |
|-----------|--------|----------|--------|
| Definition block | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| FAQ coverage | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| E-E-A-T signals | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| Heading structure | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| Internal links | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| Meta title + description | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |
| Schema opportunity | Pass / Partial / Fail | — | Auto-fix / Needs approval / Info only |

**Severity levels:**
- **Auto-fix** — fix is safe to apply without changing meaning (e.g. add missing schema type, reorder a heading). Apply and note.
- **Needs approval** — fix changes content meaning or tone (e.g. rewrite a paragraph, change a claim). Show proposed change and ask.
- **Info only** — gap exists but cannot be resolved without user input or post-publish data (e.g. no E-E-A-T signals available, schema field missing from collection).

---

### Phase 5.5 — External claims verification gate

Before writing to Webflow, scan the final draft for:
- References to third-party tools, products, or services
- Pricing claims (any $ amount or tier description)
- Feature comparisons attributing capabilities to external tools
- Statistics or data points not sourced from the user's own data

If any are found, surface a verification checklist:

```
⚠️ Verify before publishing:
- [ ] [Tool name]: pricing tier described as [X] — confirm current pricing
- [ ] [Tool name]: engines tracked listed as [X] — confirm current coverage
- [ ] [Stat]: "[claim]" — confirm source or remove
```

Do not proceed to Webflow creation until the user confirms or removes flagged items.

---

### Phase 6 — Webflow CMS draft creation

1. Use the collection schema mapped in Phase 0.
2. Create a CMS item as a draft with all available fields:

   **Always populate:**
   - Title (`name`)
   - Slug (lowercase, hyphenated, includes primary keyword)
   - Rich text body (`post-body` or equivalent)
   - Meta description (if field exists)
   - Keywords and secondary keywords (if fields exist)
   - Topic (if field exists)
   - Read time estimate (if field exists)
   - Author (if field exists and author ID is known)
   - FAQ fields Q1–Q5 (if fields exist — populate from FAQ section of article, then **remove the FAQ section from the rich text body**. Leaving FAQ in both the body and dedicated fields causes duplication when the template renders both.)

   **Populate if field exists:**
   - SEO title / meta title (50–60 chars, includes primary keyword)
   - Schema markup (if dedicated field exists)

   **Flag as missing after creation:**
   - Image fields (`main-image`, `thumbnail-image`, or equivalent) — if present in schema but not populated, output: *"⚠️ Draft created without images. Add main image and thumbnail before publishing."*
   - SEO title field — if not present in collection: *"⚠️ No SEO title field found. Meta title won't be set via API — update manually in Designer or run `/cms-collection-setup:review` to add the field."*

3. If the collection schema is unknown or inconsistent, recommend running `/cms-collection-setup` first.

---

### Phase 7 — References (optional)

If the user asked for references, or if the config requires citations:
- Prefer reputable sources (industry publications, research reports, major vendor docs)
- Add as a short "Sources" section at the end or as inline mentions
- Keep citations lightweight — do not over-link

If the user did not request references and config does not require them, skip.

---

### Phase 8 — Save and log

Append to `.claude/reports/{domain}/activity-log.md`:

```
| YYYY-MM-DD | /write-blog | Drafted "[article title]". Primary keyword: "[keyword]". Collection: [name]. CMS item ID: [id]. Status: draft / published. Pending: [image / SEO title / publish] if applicable. |
```

---

## Output (Webflow-ready summary)

Return to the user:
- CMS item name
- Slug
- Meta title (or warning if field missing)
- Meta description
- Draft item ID
- Any pending actions before publishing (images, SEO title, Designer bindings)

---

## Non-goals

- Does not perform a full site audit → use `/audit` or `/audit-deep`
- Does not replace ongoing optimization of existing URLs → use `/refresh-content`, `/click-recovery`, `/aeo-optimize` after publication
