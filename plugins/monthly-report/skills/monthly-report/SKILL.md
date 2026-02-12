---
name: monthly-report
version: "1.0"
description: |
  Monthly SEO report — what changed, why it matters, and what to do next. Compares Month M vs M-1, scores recommendations by impact/confidence/effort, and outputs an actionable report for founders and executives.
  Triggers: monthly report, seo report, monthly review, performance report, traffic report, monthly analysis.
  Requires: Google Search Console MCP server, Webflow MCP server. Optional: Keywords Everywhere API for volume/intent enrichment.
  Workflow: Configure → Fetch → Analyze → Score → Report → Save.
  Modes: /monthly-report (full report), /monthly-report:quick (executive summary + action plan only).
---

# Monthly Report Skill

A decision-making tool for founders and executives. Compares this month vs last month, identifies what changed and why, scores every recommendation, and outputs a prioritized action plan.

This skill is **read-only** — it analyzes GSC + Webflow data but never modifies Webflow content. It points you to `/click-recovery` and `/refresh-content` for execution.

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) (for page inventory and metadata audit)
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (needs API key — for search volume and intent enrichment on top queries)

## Skill Modes

This skill supports two modes for flexibility:

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/monthly-report` | Complete 7-section report with all analysis and recommendations. |
| **Quick** | `/monthly-report:quick` | Executive summary + action plan only. Fast overview for busy founders. |

## Workflow Overview

```
[PREREQUISITES] → CONFIGURE → FETCH → ANALYZE → SCORE → REPORT → SAVE
```

- **PREREQUISITES**: MCP discovery, config loading
- **CONFIGURE**: Mode select, date boundaries, site confirmation
- **FETCH**: GSC performance data (M and M-1), Webflow page inventory, optional KE enrichment
- **ANALYZE**: Build data for all 7 report sections
- **SCORE**: Impact/Confidence/Effort scoring → Priority buckets
- **REPORT**: Render full Markdown report
- **SAVE**: Display in terminal + save to `.claude/reports/`

---

## Conditional Guards (Global)

These guards apply throughout the skill execution.

⚡ GUARD — **Load SEO Copilot config:**
At the start of execution, check if `.claude/seo-copilot-config.json` exists:
- If yes: Load and apply settings:
  - `business.name` → Used for branded query detection
  - `brandVoice.tone` → Match tone in recommendation copy
  - `seo.metaTitleFormat` → Reference in metadata audit
  - `audience.primary` → Frame recommendations for target audience
  - `seo.competitors` → Compare if mentioned in GSC queries
- If no: Proceed with defaults, note: "No SEO Copilot config found. Run `/getting-started` for personalized recommendations."

⚡ GUARD — **User requests abort:**
If user says "stop", "cancel", "abort", or "nevermind" at any phase:
- Confirm: "Stop the workflow? Progress will be lost."
- If confirmed: Exit cleanly with summary of what was completed
- Output any partial results (e.g., data fetched but not analyzed)

⚡ GUARD — **GSC MCP unavailable:**
This skill requires GSC data. If unavailable:
- Inform user: "Monthly Report requires Google Search Console access. Please connect GSC MCP and try again."
- Stop execution — GSC is a hard requirement

⚡ GUARD — **Webflow MCP unavailable:**
This skill requires Webflow data for page inventory, indexation cross-reference, and metadata audit. If unavailable:
- Inform user: "Monthly Report requires Webflow MCP for page inventory and metadata analysis. Please connect Webflow MCP and try again."
- Stop execution — Webflow is a hard requirement

⚡ GUARD — **Keywords Everywhere unavailable:**
If KE API is not available:
- Proceed without volume/intent enrichment
- Note in report: "Search volume and intent data unavailable. Analysis based on GSC impressions only."
- Suggest user add KE API for richer insights

---

## Phase 0: Prerequisites & Configuration

### 0.1 Choose Mode

At the start, determine which mode to run:

```
How would you like to run the Monthly Report?

1. **Full report** (recommended) — All 7 sections with detailed analysis and recommendations
2. **Quick summary** — Executive summary + action plan only (faster)
```

- If **Full report** or no mode specified: Run all phases, render all 7 sections
- If **Quick summary**: Run all phases but only render Section 1 (Executive Summary) and Section 7 (Action Plan)

### 0.2 MCP Discovery

**Search for these tools BEFORE starting analysis:**

**GSC MCP** (required):
- Search: `+gsc search analytics`
- If missing: Stop and inform user: "Monthly Report requires GSC MCP. Install from: https://github.com/sofianbettayeb/gsc-mcp-server"

**Webflow MCP** (required):
- Search: `+webflow data cms`
- If missing: Stop and inform user: "Monthly Report requires Webflow MCP. Install from: https://developers.webflow.com/mcp/reference/overview"

**Keywords Everywhere** (optional but check):
- Search: `+keywords everywhere volume`
- If missing: Continue, but ADD to report header:
  ```
  ⚠️ Keywords Everywhere not connected.
  Analysis based on GSC impressions only.
  Connect KE for search volume and intent data:
  https://github.com/hithereiamaliff/mcp-keywords-everywhere
  ```

### 0.3 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract:
- `business.name` → for branded query split
- `brandVoice` → for recommendation tone
- `seo.metaTitleFormat` → for metadata audit reference
- `seo.competitors` → for competitor query detection
- `audience.primary` → for framing recommendations

### 0.4 Calculate Date Boundaries

Compute date ranges for comparison:

| Period | Label | Use |
|--------|-------|-----|
| Month M | Current/most recent complete month | Primary analysis |
| Month M-1 | Previous month | Comparison baseline |
| Month M-2 | Two months ago | 3-month trend (optional) |
| Month M-3 | Three months ago | 3-month trend (optional) |

If today is mid-month, use the most recent **complete** month as M.

### 0.5 Confirm Site

⚡ GUARD — **Multiple GSC properties:**
If multiple properties are available:
- List all properties
- Ask user to select: "Which GSC property should I analyze?"
- Confirm selection before proceeding

⚡ GUARD — **Report already exists:**
Check if `.claude/reports/monthly-report-YYYY-MM.md` already exists for the target month:
- If yes: Ask user: "A report for [Month] already exists. Overwrite it?"
- If user declines: Stop execution

---

## Phase 1: Fetch Data

### 1.1 GSC Performance Data

Fetch performance data for Month M and Month M-1 (and optionally M-2, M-3 for trends):

**Page-level data:**
- All pages with any impressions
- Include: page URL, impressions, clicks, CTR, average position
- Fetch for both M and M-1

**Query-level data:**
- All queries with impressions > 10
- Include: query, page, impressions, clicks, CTR, position
- Fetch for both M and M-1

### 1.2 Webflow Page Inventory

Fetch the full page inventory from Webflow:

**CMS items:**
1. Use `get_collection_list` to find all collections
2. For each collection, use `list_collection_items` to get all items
3. Extract: item name, slug, published date, meta title, meta description, body content (if accessible)

⚡ GUARD — **Large site (>100 CMS items):**
If a collection has more than 100 items:
- Sample the first 100 items
- Note in report: "Large collection ([N] items) — sampled first 100. Run again with specific collection for full audit."

**Static pages:**
1. Use `list_pages` to get all static pages
2. Extract: page name, path, SEO title, meta description

### 1.3 GSC URL Inspection (Sampled)

For the top 20 pages by impressions, run URL inspection if available:
- Index status (indexed / not indexed / crawled but not indexed)
- Last crawl date
- Mobile usability issues

⚡ GUARD — **URL inspection rate limit:**
If rate-limited:
- Inspect top 10 pages only
- Note: "URL inspection limited to top 10 pages due to API rate limits."

### 1.4 Keywords Everywhere Enrichment (Optional)

If KE is available, enrich the top 30 queries by impressions:
- Monthly search volume
- CPC (commercial intent signal)
- Competition score
- Trend data (rising/falling)

---

## Phase 2: Analyze

Build the data structures needed for each report section.

### 2.1 Executive Summary Metrics

Calculate:
- Total clicks M vs M-1 (absolute + %)
- Total impressions M vs M-1 (absolute + %)
- Average CTR M vs M-1
- Average position M vs M-1
- Total indexed pages (from URL inspection or GSC page count)

Determine health trend:
- **UP**: clicks increased ≥5% AND (impressions up OR position improved)
- **DOWN**: clicks decreased ≥5% AND (impressions down OR position worsened)
- **STABLE**: everything else

Identify:
- **Biggest win**: page with largest click increase M vs M-1
- **Biggest risk**: page with largest click decrease M vs M-1
- **Top 3 priorities**: preview from scoring engine (Phase 3)

### 2.2 Performance Overview

Build traffic table with 3-month trend:

| Metric | M-2 | M-1 | M | Trend |
|--------|-----|-----|---|-------|
| Clicks | ... | ... | ... | ↑/↓/→ |
| Impressions | ... | ... | ... | ↑/↓/→ |
| CTR | ... | ... | ... | ↑/↓/→ |
| Avg Position | ... | ... | ... | ↑/↓/→ |

**Branded vs Non-Branded Split:**
- Branded queries: contain `business.name` from config (case-insensitive)
- If no config: ask user for brand name, or skip split
- Calculate clicks/impressions for each group

⚡ GUARD — **No branded queries detected:**
If zero queries match brand name:
- Skip branded split
- Note: "No branded queries detected. Check that the brand name in config matches how users search."

### 2.3 Content Performance

**Top 10 traffic pages:**
- Sorted by clicks in Month M
- Include: URL, clicks M, clicks M-1, change, impressions, CTR, position

**Top 5 growing pages:**
- Largest click increase M vs M-1 (minimum 10 clicks in M to qualify)

**Top 5 declining pages:**
- Largest click decrease M vs M-1 (minimum 10 clicks in M-1 to qualify)

⚡ GUARD — **Zero declining pages:**
If no pages declined:
- Note: "No significant declines detected. All pages maintained or grew traffic."
- Still show top pages and growing pages

**Content gaps:**
Identify GSC queries with >500 impressions that don't match any Webflow page's meta title or meta description:
- These represent topics users search for but the site doesn't explicitly target
- Group by theme where possible

⚡ GUARD — **Zero content gaps:**
If all high-impression queries are covered:
- Note: "No major content gaps found. Existing pages cover your high-impression queries well."

### 2.4 Technical SEO Health

**Indexation cross-reference:**

1. Build set of Webflow published pages (normalize: strip trailing slashes, lowercase)
2. Build set of GSC pages with impressions (normalize: strip scheme + domain, lowercase)
3. Cross-reference:
   - **Not indexed**: Webflow pages NOT appearing in GSC = potentially not indexed
   - **Indexed orphans**: GSC pages NOT in Webflow = old/deleted pages still in Google's index

**URL normalization logic:**
- GSC URLs: `https://example.com/blog/my-post` → `/blog/my-post`
- Webflow CMS items: collection slug + item slug → `/blog/my-post`
- Webflow static pages: page path → `/about`, `/contact`
- Strip trailing slashes, lowercase everything

**Metadata audit:**
For all Webflow pages, check:
- **Missing meta title**: no title set or using default/placeholder
- **Missing meta description**: empty or not set
- **Duplicate meta titles**: same title on multiple pages
- **Title too long**: > 60 characters
- **Title too short**: < 30 characters
- **Description too long**: > 155 characters
- **Description too short**: < 70 characters

### 2.5 On-Page SEO Opportunities

**High impressions, low CTR (CTR optimization):**
- Pages with impressions > 500 AND CTR < 2%
- Or pages where CTR is < half the site average for that position bracket
- These are candidates for `/click-recovery`

**Striking distance keywords (position 5-15):**
- Queries where position is between 5 and 15
- Sorted by impressions (highest potential)
- A small ranking improvement could move these to top positions

**Keyword mismatches:**
- Compare top GSC query for each page against the page's meta title
- If the #1 query (by impressions) doesn't appear in the meta title → mismatch
- These are quick wins for title optimization

### 2.6 Internal Linking & Structure

**Link graph from CMS body content:**
If CMS body content is accessible (rich text fields):
1. Parse body content for internal links (hrefs containing the site domain or relative paths)
2. Build a simple link graph: which pages link to which

⚡ GUARD — **No CMS body content accessible:**
If body content is not available or not in a parseable format:
- Skip link graph analysis
- Note: "Internal link analysis unavailable — CMS body content not accessible via API."
- Still check for orphan pages using GSC + Webflow cross-reference

**Link distribution:**
- Average internal links per page
- Pages with most internal links (over-linked)
- Pages with fewest internal links (under-linked)

**Orphan pages:**
- Pages with zero internal links pointing to them (from the link graph)
- Cross-reference with Webflow sitemap/inventory

**Under-linked pages:**
- High-traffic pages (top 20 by clicks) with fewer than 3 internal links pointing to them
- These deserve more internal link support

**Pillar page health:**
- If identifiable from site structure (e.g., `/blog/topic` → `/blog/topic/subtopic`):
  - Check if pillar pages link to all child pages
  - Check if child pages link back to pillar

---

## Phase 3: Score Recommendations

### 3.1 Scoring Engine

Every recommendation gets scored on three dimensions:

| Dimension | Scale | Criteria |
|-----------|-------|----------|
| **Impact** | 1-5 | Clicks at risk or recoverable, % of total site traffic, conversion proximity |
| **Confidence** | 1-5 | Data strength (sample size), multi-month confirmation (consistent trend), data source reliability |
| **Effort** | 1-5 | 1 = metadata change, 2 = content tweak, 3 = section rewrite, 4 = new section/FAQ, 5 = full page creation |

### 3.2 Priority Calculation

```
Priority = (Impact × Confidence) / Effort
```

### 3.3 Bucket Assignment

| Bucket | Criteria | Action |
|--------|----------|--------|
| **Must fix** | Priority ≥ 8.0 AND (Impact ≥ 4 OR indexation issue) | Address this month |
| **High impact** | Priority ≥ 4.0 AND Impact ≥ 3 | Schedule within 2 weeks |
| **Nice to have** | Priority ≥ 1.5 | Add to backlog |

Recommendations with Priority < 1.5 are excluded from the report (noise reduction).

### 3.4 Recommendation Format

Each recommendation includes:

1. **Action-oriented title** — what to do (e.g., "Rewrite meta title for /blog/seo-guide to match top query")
2. **Why it matters** — 1-2 lines with supporting metrics (e.g., "This page lost 340 clicks (-45%) month-over-month. The #1 query 'seo guide 2026' doesn't appear in the title.")
3. **Expected impact** — qualitative + quantified (e.g., "High — recovering even 50% of lost clicks = ~170 clicks/month")
4. **Effort level** — Low / Medium / High with explanation
5. **Exact steps** — Webflow-specific instructions (e.g., "Open CMS → Blog → seo-guide → Update Meta Title field to: ...")
6. **Skill shortcut** — which skill to run (e.g., "Run `/click-recovery`" or "Run `/refresh-content /blog/seo-guide`")

---

## Phase 4: Compile Report

Render the full Markdown report using the template below. In Quick mode, only render Section 1 and Section 7.

### Report Template

```markdown
# Monthly SEO Report — [Month Year]

**Site**: [property URL]
**Period**: [Month M] vs [Month M-1]
**Generated**: [date]

**Data sources**:
- ✅ GSC: Connected
- ✅ Webflow: Connected
- ✅ Keywords Everywhere: Connected [or ⚠️ Not connected — no volume/intent data]
- ✅ SEO Copilot Config: Loaded [or ℹ️ Not found — using defaults]

---

## 1. Executive Summary

**Health trend**: 🟢 UP / 🟡 STABLE / 🔴 DOWN

| Metric | [M-1] | [M] | Change |
|--------|-------|-----|--------|
| Clicks | [X] | [X] | [+/-X] ([+/-X%]) |
| Impressions | [X] | [X] | [+/-X] ([+/-X%]) |
| CTR | [X%] | [X%] | [+/-X pp] |
| Avg Position | [X] | [X] | [+/-X] |
| Indexed Pages | [X] | [X] | [+/-X] |

**Biggest win**: [Page title] — [metric improvement]
**Biggest risk**: [Page title] — [metric decline]

**Top 3 priorities**:
1. [Priority 1 — one-line summary]
2. [Priority 2 — one-line summary]
3. [Priority 3 — one-line summary]

---

## 2. Performance Overview

### Traffic Trend (3 months)

| Metric | [M-2] | [M-1] | [M] | Trend |
|--------|-------|-------|-----|-------|
| Clicks | [X] | [X] | [X] | ↑/↓/→ |
| Impressions | [X] | [X] | [X] | ↑/↓/→ |
| CTR | [X%] | [X%] | [X%] | ↑/↓/→ |
| Avg Position | [X] | [X] | [X] | ↑/↓/→ |

### Branded vs Non-Branded

| Segment | Clicks | Impressions | CTR | % of Total |
|---------|--------|-------------|-----|------------|
| Branded | [X] | [X] | [X%] | [X%] |
| Non-Branded | [X] | [X] | [X%] | [X%] |

> **Note**: Branded queries use "[business name]" from config. Non-branded = everything else.

### Conversions

> Conversion data is not available via GSC API. If you track conversions in GA4 or another tool, cross-reference the top pages below with your conversion data.

---

## 3. Content Performance

### Top 10 Pages by Traffic

| # | Page | Clicks (M) | Clicks (M-1) | Change | Impressions | CTR | Position |
|---|------|-----------|-------------|--------|-------------|-----|----------|
| 1 | [title] | [X] | [X] | [+/-X%] | [X] | [X%] | [X] |
| ... | ... | ... | ... | ... | ... | ... | ... |

### Top 5 Growing Pages

| Page | Clicks (M-1) | Clicks (M) | Change | Why |
|------|-------------|-----------|--------|-----|
| [title] | [X] | [X] | [+X%] | [brief reason] |

### Top 5 Declining Pages

| Page | Clicks (M-1) | Clicks (M) | Change | Likely Cause | Action |
|------|-------------|-----------|--------|--------------|--------|
| [title] | [X] | [X] | [-X%] | [reason] | [skill to run] |

### Content Gaps

Queries with >500 impressions not matching any page title or description:

| Query | Impressions | Clicks | Position | Opportunity |
|-------|-------------|--------|----------|-------------|
| [query] | [X] | [X] | [X] | [Create new page / Update existing page] |

---

## 4. Technical SEO Health

### Indexation Status

| Status | Count | Pages |
|--------|-------|-------|
| ✅ Indexed & in Webflow | [X] | — |
| ⚠️ Not indexed (in Webflow, not in GSC) | [X] | [list] |
| ⚠️ Indexed orphans (in GSC, not in Webflow) | [X] | [list] |

### Metadata Audit

| Issue | Count | Pages |
|-------|-------|-------|
| ❌ Missing meta title | [X] | [list] |
| ❌ Missing meta description | [X] | [list] |
| ⚠️ Duplicate meta titles | [X] | [list] |
| ⚠️ Title too long (>60 chars) | [X] | [list] |
| ⚠️ Title too short (<30 chars) | [X] | [list] |
| ⚠️ Description too long (>155 chars) | [X] | [list] |
| ⚠️ Description too short (<70 chars) | [X] | [list] |

---

## 5. On-Page SEO Opportunities

### High Impressions, Low CTR

Pages where Google shows you but users don't click:

| Page | Impressions | CTR | Position | Top Query | Action |
|------|-------------|-----|----------|-----------|--------|
| [title] | [X] | [X%] | [X] | [query] | Run `/click-recovery` |

### Striking Distance Keywords (Position 5-15)

A small ranking push could move these to top positions:

| Query | Page | Position | Impressions | Potential |
|-------|------|----------|-------------|-----------|
| [query] | [title] | [X] | [X] | [estimated click gain] |

### Keyword Mismatches

Top query for the page doesn't appear in the meta title:

| Page | Meta Title | Top Query (by impressions) | Impressions |
|------|-----------|---------------------------|-------------|
| [title] | [current title] | [query] | [X] |

---

## 6. Internal Linking & Structure

### Link Distribution

| Metric | Value |
|--------|-------|
| Average internal links per page | [X] |
| Most linked page | [title] ([X] links) |
| Least linked page (with traffic) | [title] ([X] links) |

### Orphan Pages

Pages with zero internal links pointing to them:

| Page | Clicks | Impressions | Action |
|------|--------|-------------|--------|
| [title] | [X] | [X] | Add internal links from related pages |

### Under-Linked High-Traffic Pages

Top pages that deserve more internal link support:

| Page | Clicks | Internal Links | Recommended |
|------|--------|---------------|-------------|
| [title] | [X] | [X] | Add [X] more links from related content |

### Pillar Page Health

| Pillar | Child Pages | Links to Children | Links Back | Status |
|--------|-------------|-------------------|------------|--------|
| [title] | [X] | [X/X] | [X/X] | ✅ / ⚠️ Incomplete |

---

## 7. Action Plan

### Must Fix (Priority ≥ 8.0)

[For each recommendation:]

#### [Action-oriented title]

**Why it matters**: [1-2 lines with metrics]
**Expected impact**: [qualitative + quantified]
**Effort**: [Low/Medium/High — explanation]

**Steps**:
1. [Webflow-specific step]
2. [Webflow-specific step]
3. ...

**Shortcut**: Run `[/skill-name]`

---

### High Impact (Priority ≥ 4.0)

[Same format as above]

---

### Nice to Have (Priority ≥ 1.5)

[Condensed format — title, why, effort, shortcut only]

---

## Next Steps

1. Start with **Must Fix** items — they have the highest ROI
2. Use `/click-recovery` for CTR optimization tasks
3. Use `/refresh-content [URL]` for content refresh tasks
4. Re-run `/monthly-report` next month to track progress

## Monitoring Checklist

| When | What to check |
|------|---------------|
| **1 week** | Verify indexation changes (submit URLs via GSC if needed) |
| **2 weeks** | Check metadata updates are reflected in SERPs |
| **4 weeks** | Re-run `/monthly-report` to measure impact and find new opportunities |
```

---

## Phase 5: Output

### 5.1 Display in Terminal

Output the full rendered report in the terminal.

### 5.2 Save to File

Save the report to `.claude/reports/monthly-report-YYYY-MM.md` where YYYY-MM is the report month.

⚡ GUARD — **Can't create reports directory:**
If `.claude/reports/` doesn't exist:
- Create it: `mkdir -p .claude/reports/`
- If creation fails: Output report in terminal only, warn: "Could not create reports directory. Report displayed above but not saved to file."

⚡ GUARD — **File already exists:**
This was already handled in Phase 0 (overwrite confirmation). If we reach this point, overwrite is approved — proceed with saving.

### 5.3 Suggest Next Skills

After saving, suggest next actions based on findings:

```
## What to Do Next

Based on this report, here are the recommended next steps:

[If CTR issues found:]
→ Run `/click-recovery` to fix meta titles and descriptions for [N] underperforming pages

[If declining content found:]
→ Run `/refresh-content [URL]` on your top declining pages:
  - [page 1 URL]
  - [page 2 URL]

[If content gaps found:]
→ Consider creating new content targeting: [top gap queries]

[Always:]
→ Re-run `/monthly-report` next month to track progress
```

---

## Phase 6: Follow-Up

Provide a monitoring checklist at the end of the report:

| When | What to Check | How |
|------|---------------|-----|
| **1 week** | Indexation of any new/updated pages | Check GSC Coverage report or `site:URL` in Google |
| **2 weeks** | Metadata changes reflected in SERPs | Search brand + key terms, verify updated titles/descriptions |
| **4 weeks** | Re-run monthly report | Run `/monthly-report` to measure changes and find new opportunities |

---

## Error Handling

| Error | Action |
|-------|--------|
| GSC MCP not connected | Stop and instruct user to connect GSC |
| Webflow MCP not connected | Stop and instruct user to connect Webflow |
| GSC property not found | List available properties, ask user to select |
| No data for date range | Try shorter range (28 days), warn about limited data |
| Multiple GSC properties | List all, ask user to select |
| Keywords Everywhere API error | Proceed without KE data, note in report |
| Rate limit hit (URL inspection) | Reduce sample size, note limitation |
| Large collection (>100 items) | Sample first 100, note in report |
| No CMS body content accessible | Skip internal link analysis, note in report |
| Low traffic site (<1000 impressions) | Warn about statistical significance, proceed with caveats |
| Report directory creation fails | Display in terminal only, warn user |
| Report file already exists | Ask overwrite confirmation (Phase 0 guard) |
| No branded queries found | Skip branded split, note in report |
| Zero declining pages | Note positive trend, skip decline section |
| Zero content gaps | Note good coverage, skip gaps section |
| URL normalization mismatch | Log mismatched URLs, attempt fuzzy matching on slug |

---

## Key Data Combination Logic

### URL Normalization

GSC returns full URLs. Webflow uses slugs. Normalize both to match:

```
GSC:     https://example.com/blog/my-post  →  /blog/my-post
Webflow: collection "blog" + slug "my-post" →  /blog/my-post
Static:  page path "/about"                →  /about
```

- Strip scheme and domain from GSC URLs
- Combine collection slug + item slug for CMS items
- Use page path directly for static pages
- Strip trailing slashes
- Lowercase everything

### Indexation Cross-Reference

```
Webflow published pages  ←→  GSC pages with impressions

Match     = indexed and live (good)
Webflow only = potentially not indexed (investigate)
GSC only     = indexed orphan — old/deleted page still in Google (clean up)
```

### Content Gaps

```
GSC queries with >500 impressions
  MINUS
Queries that appear in any Webflow page's meta title OR meta description
  EQUALS
Content gaps — topics users search for but the site doesn't explicitly target
```

### Branded Query Split

```
Branded = queries containing business.name (from config), case-insensitive
Non-branded = everything else

If no config or no business.name: ask user or skip split
```

---

## Integration with Other Skills

Monthly Report is **read-only** — it identifies issues and recommends actions. Other skills execute those actions.

| Finding | Recommended Skill | Why |
|---------|-------------------|-----|
| Low CTR pages | `/click-recovery` | Fix meta titles and descriptions |
| Declining content | `/refresh-content [URL]` | Full content refresh with updated keywords |
| Missing metadata | `/click-recovery` | Quick metadata fixes |
| Content gaps | Manual / `/refresh-content` | Create or update content for uncovered queries |
| Keyword mismatches | `/click-recovery` | Align titles with actual search queries |

**Workflow:**
1. Run `/monthly-report` monthly to assess overall SEO health
2. Use the Action Plan to prioritize work
3. Execute with `/click-recovery` and `/refresh-content`
4. Re-run `/monthly-report` next month to measure impact

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

**Determining `{domain}`**: Extract from the selected GSC property URL (e.g., `https://www.checklist-seo.com/` → `checklist-seo.com`). Same logic as `/weekly-report`.

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append a row:

```
| YYYY-MM-DD | /monthly-report | [one-line summary: e.g., "February 2026 report. Health: UP. 4 must-fix, 6 high-impact items. Saved to monthly-report-2026-02.md"] |
```

Log even if the skill exits early due to a guard — note why (e.g., "Aborted: GSC MCP unavailable").
