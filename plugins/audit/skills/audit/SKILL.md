---
name: audit
version: "1.0"
description: |
  Quick SEO & AEO maturity audit from a public URL. Scores 4 dimensions (Content, Technical, Authority, Measurement) across 5 maturity levels using deterministic boolean checks on public signals only.
  Triggers: audit, site audit, seo audit, aeo audit, discovery audit.
  Requires: Web access (WebFetch). No MCP servers needed.
  Workflow: Parse → Prompt → Fetch → Check → Score → Report → Save.
  Command: /audit {url}
---

# Quick Audit Skill

Assess any site's SEO and AEO maturity from a public URL. Runs deterministic boolean checks on public signals, scores 4 dimensions using the AEO Maturity Model, and outputs a prioritized action plan grouped by maturity level.

This skill is **read-only** and requires no MCP servers — just a URL.

## Prerequisites

- **Required**: Web access (WebFetch tool)
- **No MCP servers needed** — quick audit runs on public signals only

## Maturity Model Reference

See `references/maturity-model.md` for the full rubric. Summary:

| Level | Name | One-liner |
|-------|------|-----------|
| 1 | Keywords | Basic keyword SEO. Titles target keywords, content exists. |
| 2 | Answers | Content answers questions. FAQs and intent-matching headers. |
| 3 | Structure | Topic clusters, internal linking, full schema. |
| 4 | Pillar | Authority on core topics. AI engines cite the content. |
| 5 | Authority | Programmatic AEO. Entity recognition. AI citations. |

4 dimensions: **Content**, **Technical**, **Authority**, **Measurement**.
Overall level = minimum dimension score (weakest link).

---

## Critical Guards

⚡ GUARD — **URL not accessible:**
- If WebFetch fails: "Could not fetch {url}. Check the URL is correct and publicly accessible."
- Stop execution.

⚡ GUARD — **Not a Webflow site:**
- Still run the audit — the maturity model applies to any site.
- Note: "This doesn't appear to be a Webflow site. Webflow-specific recommendations may not apply."

⚡ GUARD — **User requests abort:**
- Confirm, exit cleanly, output any partial results.

---

## Phase 0: PARSE & PROMPT

### 0.1 Parse URL & Set {domain}

Extract the domain from the provided URL. Normalize: strip `www.`, lowercase.
- `https://www.checklist-seo.com/blog/guide` → `checklist-seo.com`

The `{domain}` is used for:
- **Report save path**: `./{domain}/reports/audit-quick-YYYY-MM-DD.md`
- **Latest pointer**: `./{domain}/reports/latest-quick.md`
- **Activity log**: `./{domain}/reports/activity-log.md`

Determine the homepage URL from the domain (for sitemap and site-wide checks).

### 0.2 Review Activity Log

Check `./{domain}/reports/activity-log.md`:
- If it exists: show recent activity summary (last 10 entries)
- **Redundancy check**: if `/audit` was run in the last 7 days → warn: "You ran `/audit` on [date]. Run again anyway?"
- If not found: proceed silently

### 0.3 Prompt for Optional URLs

After parsing `{url}`, prompt:

```
Optional inputs (press Enter to skip each):

1. Topic pillar URL — a main topic page (e.g., /blog/seo-guide)
2. Blog post example URL — a typical blog article
```

Do not ask for credentials. Quick audit runs on public signals only.

---

## Phase 1: FETCH

Fetch these resources in a deterministic order:

| # | Resource | How | Required |
|---|----------|-----|----------|
| 1 | Home page HTML | WebFetch `https://{domain}/` | Yes — stop if fails |
| 2 | Pillar page HTML | WebFetch pillar URL (if provided) | No |
| 3 | Blog post HTML | WebFetch blog URL (if provided) | No |
| 4 | sitemap.xml | WebFetch `https://{domain}/sitemap.xml` | No — try `/sitemap_index.xml` as fallback |
| 5 | robots.txt | WebFetch `https://{domain}/robots.txt` | No |
| 6 | PageSpeed | Run PageSpeed via MCP on home URL (and pillar if provided) | No — skip if unavailable |

For each fetch, record: URL attempted, success/fail, HTTP status if available.

---

## Phase 2: CHECK

Run deterministic boolean checks on the fetched HTML. Every check produces:
- **Pass/Fail** (boolean)
- **Evidence** (exact snippet, attribute value, or count)
- **Source URL** (which page the evidence came from)

No heuristic language in pass/fail. Heuristics may only appear in recommendations.

### Content Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| C1 | Title exists | `<title>` tag present and non-empty | Tag content |
| C2 | Exactly one H1 | Count of `<h1>` elements = 1 | Count found, text of H1(s) |
| C3 | Heading hierarchy | No skipped levels (H1→H2→H3, not H1→H3) | First violation if any |
| C4 | FAQ section present | Regex match for "FAQ", "Frequently Asked", or ≥3 question-pattern headings (`?` in H2/H3/H4) | Matched text |
| C5 | Word count ≥ 500 | Visible text word count (strip nav/footer/scripts) | Exact count |
| C6 | Word count ≥ 800 | Same as above, higher threshold | Exact count |
| C7 | Word count ≥ 2000 | Same, pillar-level threshold | Exact count |
| C8 | Internal links present | Count of `<a href>` pointing to same host | Count |
| C9 | Related posts pattern | Links/section matching "related", "you may also like", "more articles" | Matched pattern |
| C10 | Freshness date detected | `<time>` tag, "Last updated", "Published", or structured date pattern (YYYY-MM-DD) in article body | Date found |

### Technical Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| T1 | HTTPS active | URL scheme is `https` | URL |
| T2 | Viewport meta present | `<meta name="viewport">` exists | Tag content |
| T3 | Canonical link present | `<link rel="canonical">` exists | Href value |
| T4 | Open Graph tags present | `<meta property="og:title">` at minimum | Tags found |
| T5 | JSON-LD present | `<script type="application/ld+json">` exists | Schema types detected |
| T6 | Schema: Organization | JSON-LD contains `@type: Organization` | Snippet |
| T7 | Schema: Article | JSON-LD contains `@type: Article` (or subtypes) | Snippet |
| T8 | Schema: FAQPage | JSON-LD contains `@type: FAQPage` | Snippet |
| T9 | Schema: BreadcrumbList | JSON-LD contains `@type: BreadcrumbList` | Snippet |
| T10 | Schema: Person | JSON-LD contains `@type: Person` | Snippet |
| T11 | robots.txt accessible | Fetch succeeded, not empty | Content summary |
| T12 | robots.txt not blocking key paths | No `Disallow: /` for major user agents | Disallow rules found |
| T13 | Sitemap accessible | sitemap.xml fetched and parseable | URL count if available |
| T14 | Clean URL structure | No query-string parameters on core pages (home, pillar, blog) | URLs checked |

### Authority Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| A1 | Author byline present | Text matching "By [Name]", `<span class="author">`, or similar patterns on blog post (if provided) or homepage | Matched text |
| A2 | About/team page discoverable | Internal link containing "about", "team", or "company" in href or text | Link found |
| A3 | Author bio markers | Credentials, role, or experience mentions near author name ("CEO", "years of experience", "certified") | Matched text |
| A4 | External citations present | Outbound links to non-social, non-navigation domains on content pages | Count + domains |

### Measurement Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| M1 | GA4 detected | Script containing `gtag(` or GA measurement ID pattern (`G-XXXXXXX`) | Script snippet |
| M2 | GTM detected | Script containing `GTM-` container ID | Container ID |
| M3 | GSC verification | `<meta name="google-site-verification">` present | Tag content |
| M4 | Other analytics tools | Patterns for PostHog (`posthog`), Segment (`analytics.js`), Hotjar, Mixpanel, Plausible, Fathom | Tools detected |

---

## Phase 3: SCORE

Apply the maturity model rubric deterministically. For each dimension, evaluate level gates in order (1→5). Stop at the first level that fails.

### Scoring Engine

```
score_dimension(dimension, evidence):
  for level in [1, 2, 3, 4, 5]:
    if all gates for this level pass:
      continue
    else:
      return level - 1 (or 1 if level 1 fails)
  return 5
```

### Content Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | C1 (title exists) AND C2 (exactly one H1) |
| 2 | Level 1 AND C4 (FAQ present) AND C6 (≥800 words) AND C3 (heading hierarchy ok) |
| 3 | Level 2 AND C8 (internal links ≥ 5) AND C9 (related posts pattern) |
| 4 | Level 3 AND C7 (≥2000 words on pillar page, if provided) AND C10 (freshness date) |
| 5 | Level 4 AND freshness date within last 6 months AND entity-rich markup detected |

### Technical Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | T1 (HTTPS) AND T2 (viewport) AND C1 (title exists) |
| 2 | Level 1 AND T3 (canonical) AND T13 (sitemap accessible) AND T4 (OG tags) |
| 3 | Level 2 AND T5 (JSON-LD present) AND T7 (Article schema) |
| 4 | Level 3 AND T6 (Organization schema) AND T10 (Person schema) AND T9 (BreadcrumbList) |
| 5 | Level 4 AND SameAs links to authoritative profiles AND comprehensive structured data |

### Authority Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline — no author signals = Level 1) |
| 2 | A1 (author byline) AND A2 (about/team page discoverable) |
| 3 | Level 2 AND A3 (author bio with credentials) AND A4 (external citations ≥ 1) |
| 4 | Level 3 AND strong E-E-A-T throughout (experience language, data citations, multiple evidence types) |
| 5 | Level 4 AND knowledge panel signals AND multiple platform authority |

### Measurement Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline — no tracking = Level 1) |
| 2 | (M1 OR M2) (any analytics installed) AND M3 (GSC verification) |
| 3 | Level 2 AND M4 (multiple tracking tools detected) |
| 4 | Level 3 AND advanced tracking setup (multiple measurement tools visible) |
| 5 | Level 4 AND enterprise-grade measurement stack |

### Overall Score

```
overall_level = min(content_score, technical_score, authority_score, measurement_score)
confidence = "Low-Medium" (always, for quick mode)
```

---

## Phase 4: REPORT & SAVE

### 4.1 Report Structure

Output a single markdown file with these sections in order:

**1. AEO Maturity Scorecard**
- Overall level (number + name)
- Confidence: Low–Medium
- Table: dimension, score 1–5, level name, evidence links (which checks passed/failed)
- Weakest dimension, strongest dimension, fastest win

**2. Findings per Dimension**
- For each dimension (Content, Technical, Authority, Measurement):
  - What was found (passed checks with evidence)
  - What's missing (failed checks with evidence)
  - Source URLs for each finding

**3. What's Broken & Consequences**
- Top 3–5 failures with business impact statements
- Table: issue, impact (High/Medium/Low), consequence (what happens if not fixed)

**4. Action Plan (grouped by level)**
- **Level 1 blockers first** — anything preventing a dimension from consistently reaching Level 1
- Then **Level 2 requirements**, then Level 3, etc.
- Within a level, order: Technical → Content → Measurement → Authority
- Do NOT use impact/effort scoring in quick mode — strict level ordering only
- Each action: what to do, which dimension it improves, evidence reference

**5. Roadmap to Next Level**
- Current level and target level (current + 1)
- Explicit gates to pass for the next level per dimension
- Table: action, dimension, effort, suggested skill

**Footer:**
- "Run `/audit:deep` with GSC + Webflow access for a complete audit with data-backed prioritization."

### 4.2 Save to File

Save two files:
- `./{domain}/reports/audit-quick-YYYY-MM-DD.md` (timestamped)
- `./{domain}/reports/latest-quick.md` (overwrite each run — other skills read this)

Create directories if needed: `mkdir -p ./{domain}/reports/`

### 4.3 Activity Log

Append to `./{domain}/reports/activity-log.md`:

```
| YYYY-MM-DD | /audit | Quick audit. Overall Level: X (Name). Content: X, Technical: X, Authority: X, Measurement: X. |
```

Log even on early exit (e.g., "Aborted: URL not accessible").

---

## Integration with Other Skills

| Finding | Skill | When |
|---------|-------|------|
| Low CTR / bad meta tags | `/click-recovery` | After connecting GSC |
| Outdated content | `/refresh-content {url}` | For specific pages |
| Missing config | `/getting-started` | First-time setup |
| Full data-backed audit | `/audit:deep` | After connecting GSC + Webflow |
| Ongoing monitoring | `/weekly-report` | After fixing issues |
