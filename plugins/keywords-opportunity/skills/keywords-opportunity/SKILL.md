---
name: keywords-opportunity
version: "1.0"
description: |
  Discover keyword opportunities using GSC + Keywords Everywhere. Surfaces striking distance keywords already ranking on pages 1-3 and uncovers new keywords worth targeting through content creation or expansion.
  Triggers: keyword opportunities, keyword research, keyword gaps, striking distance, new keywords, rank opportunities, keyword discovery, find keywords.
  Requires: Google Search Console MCP server. Optional but strongly recommended: Keywords Everywhere MCP server.
  Workflow: Discover → Fetch → Enrich → Analyze → Score → Report → Save.
  Modes: /keywords-opportunity (full: striking distance + new discovery), /keywords-opportunity:striking (page 1-3 wins only), /keywords-opportunity:discover (new keywords only).
---

# Keyword Opportunity Skill

Two problems, one skill. First: you're already ranking for hundreds of keywords on pages 1–3 — most of them could move with targeted effort. Second: there are entire topics you're not targeting yet but should be.

This skill surfaces both. It's read-only — it points you to `/refresh-content` and `/click-recovery` for execution.

---

## The Two Opportunity Types

**Striking distance** — keywords where you already rank but aren't capturing the traffic you should:
- Positions 4–10: page 1 but not top 3. Small content or CTR improvement = significant traffic gain.
- Positions 11–20: page 2. A focused content push can break into page 1.
- Positions 21–30: page 3. Harder, but high-volume keywords here are worth a refresh.

**New keyword discovery** — topics worth targeting that you're not currently ranking for:
- Queries you appear for in GSC (impressions) but with no dedicated page
- Related and long-tail keywords from Keywords Everywhere based on your top topics
- Long-tail query clusters from GSC that could be consolidated into a single article

---

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Strongly recommended**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) — without it, volume data is unavailable and discovery is limited to GSC impressions only
- **Optional**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) — used to cross-reference existing pages for gap detection

---

## Skill Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/keywords-opportunity` | Both striking distance + new discovery. Full report. |
| **Striking** | `/keywords-opportunity:striking` | Already-ranking keywords on pages 1-3. Fastest ROI. |
| **Discover** | `/keywords-opportunity:discover` | New keyword opportunities not yet targeted. |

---

## Workflow Overview

```
DISCOVER → FETCH → ENRICH → ANALYZE → SCORE → REPORT → SAVE
```

---

## Conditional Guards (Global)

⚡ GUARD — **Load SEO Copilot config:**
At the start of execution, check if `.claude/seo-copilot-config.json` exists:
- If yes: load `business.name` (branded query detection), `seo.primaryKeywords` (anchor topics for KE expansion), `seo.competitors` (filter competitor brand queries), `audience.primary` (frame recommendations)
- If no: proceed with defaults, note "Run `/getting-started` for personalized recommendations."

⚡ GUARD — **GSC MCP unavailable:**
- Stop: "Keyword Opportunity requires Google Search Console. Connect GSC MCP and try again."

⚡ GUARD — **Keywords Everywhere unavailable:**
- Proceed, but add a prominent warning to the report header:
  ```
  ⚠️ Keywords Everywhere not connected.
  Search volume and CPC data unavailable — opportunities are ranked by GSC impressions only.
  For volume-backed prioritization, connect KE: https://github.com/hithereiamaliff/mcp-keywords-everywhere
  ```
- In discovery mode without KE, limit new opportunity output to GSC-derived content gaps only. Skip the KE expansion section.

⚡ GUARD — **Webflow MCP unavailable:**
- Continue without page cross-reference.
- Note: "Webflow MCP not connected — skipping page-to-keyword mapping. Gap detection based on GSC query patterns only."

⚡ GUARD — **User requests abort:**
- Confirm exit. Output any partial results already computed.

---

## Phase 0: DISCOVER

### 0.1 MCP Check

Search for available tools:
- **GSC MCP** (required): if missing → stop
- **Keywords Everywhere** (strongly recommended): if missing → warn and continue
- **Webflow MCP** (optional): if missing → continue without page mapping

### 0.2 Site Selection

Use GSC to list available properties. If multiple: ask user to select. Set `{domain}` from the selected property URL.

### 0.3 Review Activity Log

Check `.claude/reports/{domain}/activity-log.md`:
- If it exists: read the last 10 entries. Surface:
  - Recent keyword opportunity runs (if run within 30 days → warn: "Ran `/keywords-opportunity` on [date]. Rankings shift slowly — consider waiting 4+ weeks between full runs.")
  - Recent `/refresh-content` runs (note which pages were optimized — these may now rank higher)
  - Recent `/click-recovery` runs (those pages had CTR fixed — check if position also improved)
- If not found: proceed silently

### 0.4 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract primary keywords and competitors for use in Phase 2.

### 0.5 Mode Confirmation

If running full mode (`/keywords-opportunity`):
```
Running full Keyword Opportunity analysis:
- Part 1: Striking distance (positions 1–30, enriched with volume)
- Part 2: New keyword discovery (GSC gaps + KE expansion)

This may take a few minutes. Proceed?
```

---

## Phase 1: FETCH

### 1.1 GSC Performance Data (90 days)

Fetch two datasets:

**Query-level data** (the primary dataset):
- All queries with at least 1 impression in the last 90 days
- Fields: query, page, clicks, impressions, CTR, average position
- This is the raw material for both striking distance and gap analysis

**Page-level data** (for context):
- All pages with impressions > 10
- Fields: page, clicks, impressions, CTR, average position

⚡ GUARD — **Low data volume (< 500 total impressions):**
- Warn: "Low data volume. Striking distance analysis may lack statistical confidence. New discovery will rely more heavily on Keywords Everywhere."
- Proceed with caveats noted in report.

### 1.2 Webflow Page Inventory (if Webflow MCP available)

Fetch all published pages and CMS items:
- Static pages: path, SEO title, meta description
- CMS items (all collections): name, slug, meta title, meta description

Build a normalized URL list: `{path} → {seo title or name}`. Used in Phase 3 for gap detection.

---

## Phase 2: ENRICH WITH KEYWORDS EVERYWHERE

**Run this phase before analysis.** Enrichment transforms raw GSC data into opportunity scores.

### 2.1 Striking Distance Enrichment

For all GSC queries with average position 1–30:
- Batch fetch from KE: monthly search volume, CPC, competition score, trend (up/flat/down)
- Store alongside GSC data: `{query, position, impressions, CTR, volume, CPC, competition, trend}`

⚡ GUARD — **KE rate limits or API errors:**
- If a batch fails: retry once with smaller batch (50 queries). If it fails again: proceed with partial enrichment, note affected queries in report.

### 2.2 Discovery Expansion

For the top 20 topics by GSC impressions (group by primary keyword of each page):
- Fetch KE related keywords for each topic
- Fetch KE long-tail suggestions
- Store: `{related_keyword, topic_source, volume, CPC, competition, trend}`

Filter out:
- Keywords already in the GSC dataset with position ≤ 30 (already captured in striking distance)
- Branded queries (queries containing `business.name` from config or site domain)
- Volume < 50/month (too small to prioritize unless CPC > $5)
- Declining trend > 30% YoY (dying queries)

### 2.3 Compute Benchmarks

From the enriched dataset, compute:
- Site average CTR by position bracket (1-3, 4-10, 11-20)
- Median monthly volume across all enriched queries
- Total impression volume in the striking distance window (positions 4-30)

---

## Phase 3: ANALYZE

### 3.1 Striking Distance Classification

Classify all queries with average position 1–30 into tiers:

| Tier | Position Range | Label | Effort | Strategy |
|------|---------------|-------|--------|----------|
| A | 4–10 | Page 1 — not top 3 | Low | CTR fix or minor content tweak |
| B | 11–20 | Page 2 | Medium | Content refresh to hit page 1 |
| C | 21–30 | Page 3 | High | Significant content work or new dedicated page |

**Additional filters for each tier:**

Tier A:
- Keep if volume ≥ 100/month OR impressions ≥ 200 in 90 days
- Sort by: volume × (1 - CTR) — queries with high volume and low CTR are the biggest mismatch

Tier B:
- Keep if volume ≥ 200/month OR impressions ≥ 500 in 90 days
- Sort by: volume × position_gap (20 - avg_position) — closer to page 1 = higher priority

Tier C:
- Keep if volume ≥ 500/month — high bar since effort is significant
- Sort by: volume × CPC — prioritize commercially valuable queries

**Deduplication**: if a page ranks for multiple queries in the same tier, group them under that page. Show top 3 queries per page.

### 3.2 Map Queries to Pages

For each query in the striking distance tiers:
1. Use the GSC `page` field to identify the ranking URL
2. Cross-reference with Webflow page inventory to get current SEO title and meta description
3. Compute: is the primary query in the current title? In the meta description? In the H1 (infer from page name)?

Flag:
- **Keyword absent from title** → high-impact fix
- **Keyword absent from meta description** → medium fix
- **Keyword present in both** → content depth issue → `/refresh-content`

### 3.3 New Keyword Discovery

**Source A — GSC content gaps:**
Queries that appear in GSC with impressions > 50 AND average position > 30:
- These queries are visible to Google but you don't have a strong page for them
- Group by semantic topic (cluster queries sharing 2+ words or intent)
- For each cluster: sum total impressions, take the highest-volume query as the "primary" keyword

**Source B — KE expansion keywords (if KE available):**
From Phase 2.2, filter to:
- Volume ≥ 200/month
- Not already in GSC dataset (positions 1–30)
- Competition ≤ 0.7 (attainable)
- Group by source topic (which existing page/topic spawned this suggestion)

**Source C — Long-tail cluster consolidation:**
From GSC, identify clusters of 3+ queries that:
- Share a root phrase (e.g., "webflow seo", "webflow seo tips", "webflow seo checklist")
- All rank position > 15
- Total cluster impressions > 200

Flag these as candidates for a single consolidated article (or expansion of an existing thin page).

### 3.4 Opportunity Scoring

Score every opportunity using the same ICE model used across other skills:

**Striking distance — opportunity score:**
```
volume_score    = min(volume / 500, 5)          # capped at 5
position_score  = (30 - avg_position) / 5       # closer to page 1 = higher
ctr_gap         = expected_ctr(position) - actual_ctr
impact          = round((volume_score + position_score + ctr_gap * 10) / 3, 1)
confidence      = 4 if volume > 0 else 2        # lower without KE data
effort          = tier_effort[tier]             # A=1, B=2, C=3
priority        = (impact × confidence) / effort
```

Expected CTR by position:
- Position 1: 28%, 2: 15%, 3: 11%, 4: 8%, 5: 7%, 6-10: 3-5%, 11-20: 1-2%

**New discovery — opportunity score:**
```
volume_score  = min(volume / 1000, 5)
intent_score  = min(CPC / 2, 5) if CPC > 0 else 2   # CPC as intent proxy
gap_score     = 5 if no page exists else 3            # full gap vs thin coverage
impact        = round((volume_score + intent_score + gap_score) / 3, 1)
confidence    = 4 if from_KE else 2                   # KE-sourced = higher confidence
effort        = 2 if existing_page else 4             # expand vs create new
priority      = (impact × confidence) / effort
```

Priority buckets (same as other skills):
- **Must pursue** (priority ≥ 8.0): act now
- **High value** (priority ≥ 4.0): next sprint
- **Worth tracking** (priority ≥ 1.5): backlog

---

## Phase 4: REPORT

### 4.1 Report Header

```
# Keyword Opportunities — {domain}

**Date:** YYYY-MM-DD
**Period analyzed:** Last 90 days
**Data sources:**
- ✅ GSC: Connected — [X] queries, [Y] pages, [Z] total impressions
- ✅ Keywords Everywhere: Connected — volume enriched [or ⚠️ Not connected]
- ✅ Webflow: Connected — [N] pages mapped [or ⚠️ Not connected — no page cross-reference]
- ✅ SEO Copilot Config: Loaded [or ℹ️ Not found]

**Summary:**
- Striking distance: [N] keywords across [M] pages — estimated +[X] clicks/month if optimized
- New opportunities: [N] keywords — [X] total monthly searches addressable
- Action plan: [N] must-pursue, [N] high-value, [N] worth tracking
```

---

### 4.2 Part 1 — Striking Distance

**Intro line**: "These keywords already rank on pages 1–3. You're earning impressions but leaving clicks on the table. Ranked by traffic potential."

#### Tier A — Page 1, Positions 4–10 (Fastest ROI)

For each page with Tier A keywords (sorted by priority score descending):

```
### {Page Title or URL path}
**Current ranking URL**: {URL}
**SEO title**: "{current title}"

| Query | Position | Volume/mo | CTR | Expected CTR | Gap | Trend |
|-------|----------|-----------|-----|--------------|-----|-------|
| {query} | {pos} | {vol} | {ctr}% | {expected}% | {gap}% | ↑/→/↓ |
| ...   |          |           |     |              |     |       |

**Diagnosis**: [Is the keyword in the title? In the meta description? In the content?]

**Recommended action**: [one of:]
- "Keyword absent from title → run `/click-recovery` to update meta title + description"
- "Keyword in title, CTR still low → title may need a stronger hook. Run `/click-recovery`."
- "Keyword present in meta tags, ranking stuck → content depth issue. Run `/refresh-content {URL}`."

**Priority**: [Must pursue / High value / Worth tracking] (Score: {priority})
**Estimated impact**: +[X] clicks/month (volume × CTR improvement)
```

#### Tier B — Page 2, Positions 11–20 (High Potential)

Summary table + top opportunities in full format:

```
| Page | Top Query | Volume/mo | Position | Priority | Action |
|------|-----------|-----------|----------|----------|--------|
| {title} | {query} | {vol} | {pos} | {score} | `/refresh-content {url}` |
```

For top 5 by priority: full format (same as Tier A above).

#### Tier C — Page 3, Positions 21–30 (Worth Pursuing)

Summary table only — these need significant work:

```
| Page | Top Query | Volume/mo | Position | CPC | Priority |
|------|-----------|-----------|----------|-----|----------|
```

Add note: "These keywords rank on page 3. To move them, run `/refresh-content {url}` for a full content refresh targeting these queries."

---

### 4.3 Part 2 — New Keyword Opportunities

**Intro line**: "Topics you're not currently targeting — or are barely visible for. Ranked by traffic potential and attainability."

#### Content Gaps (GSC-derived)

Queries you appear for in Google but don't have a strong page for (position > 30):

```
| Keyword Cluster | Top Query | Impressions/90d | Volume/mo | Intent | Action |
|-----------------|-----------|-----------------|-----------|--------|--------|
| {cluster name} | {query} | {impressions} | {vol} | info/commercial | Create new article |
| ... | | | | | Expand existing page |
```

For each cluster with priority ≥ 8.0: show full cluster (all queries in the group) and a recommended article angle.

#### Expansion Keywords (Keywords Everywhere)

New keywords from KE related to your existing top topics — not yet in your GSC data:

```
| Keyword | Volume/mo | CPC | Competition | Source Topic | Trend | Action |
|---------|-----------|-----|-------------|--------------|-------|--------|
| {keyword} | {vol} | ${cpc} | {0.0-1.0} | {topic} | ↑/→/↓ | Create / Expand |
```

Group by source topic. For each group, note: "You already rank for '{source topic}' — these are related keywords the same audience searches."

#### Long-tail Cluster Consolidation

Clusters of GSC queries that could be served by one consolidated article:

```
### Cluster: "{root phrase}"
Queries in cluster:
- "{query 1}" — position {X}, {Y} impressions
- "{query 2}" — position {X}, {Y} impressions
- "{query 3}" — position {X}, {Y} impressions
Combined monthly searches: ~{Z} (estimated from KE)
Recommendation: {Create a dedicated article / Expand existing page at {URL} to cover all these queries}
```

---

### 4.4 Action Plan

Consolidated action plan, sorted by priority score:

```
## Action Plan

### Must Pursue

| # | Keyword / Cluster | Volume/mo | Type | Action | Skill |
|---|-------------------|-----------|------|--------|-------|
| 1 | {keyword} | {vol} | Striking / New | {brief action} | `/refresh-content {url}` or `/click-recovery` |

### High Value

[Same table format — condensed]

### Worth Tracking

[Same table format — condensed]

---

## Decision Guide

| Situation | Right action | Skill |
|-----------|-------------|-------|
| Keyword in title, rank 4-10, CTR low | Fix the pitch — update title & description | `/click-recovery` |
| Keyword in title, rank 4-10, CTR ok | Strengthen content depth for query | `/refresh-content {url}` |
| Keyword not in title, rank 4-20 | Add keyword to title + strengthen content | `/click-recovery` then `/refresh-content` |
| Keyword rank 21-30, high volume | Full content refresh targeting this query | `/refresh-content {url}` |
| No page for keyword cluster (gap) | Create new article targeting the cluster | New content (manual or future skill) |
| KE expansion keyword, no page | Create new article | New content |
| Long-tail cluster, weak existing page | Expand existing page to cover all queries | `/refresh-content {url}` |
```

---

## Phase 5: SAVE

Save two files:

- `.claude/reports/{domain}/keywords-opportunity-YYYY-MM-DD.md` — full timestamped report
- `.claude/reports/{domain}/latest-keywords-opportunity.md` — overwrite on each run (other skills can reference this)

Create directories if needed.

After saving:
```
Report saved to .claude/reports/{domain}/keywords-opportunity-YYYY-MM-DD.md

Top 3 actions:
1. [highest priority action]
2. [second highest]
3. [third highest]

Run `/refresh-content {url}` to execute content optimizations.
Run `/click-recovery` to fix meta title/description for CTR gaps.
```

---

## Integration with Other Skills

This skill is **read-only** — it identifies opportunities and routes you to the right execution skill.

| Finding | Right Skill | Why |
|---------|-------------|-----|
| Keyword not in title, rank 4-20 | `/click-recovery` | Fix the meta title — quickest position booster |
| Keyword present, rank stuck 4-20 | `/refresh-content {url}` | Content depth needs strengthening |
| Keyword rank 21-30 | `/refresh-content {url}` | Full refresh targeting the specific query |
| High-volume new keyword gap | Manual content creation | No existing page to optimize |
| Long-tail cluster, thin existing page | `/refresh-content {url}` | Expand to cover the full cluster |
| CMS missing keyword fields | `/cms-collection-setup:review` | Add primary/secondary keyword fields before refreshing |
| Page schema gaps blocking refresh | `/cms-collection-setup:review` | Fix schema before optimizing |

**Recommended workflow:**
1. Run `/keywords-opportunity` monthly to refresh the opportunity map
2. Execute Tier A strikes immediately with `/click-recovery` (CTR fixes) or `/refresh-content` (content depth)
3. Plan Tier B and new content work for the following sprint
4. Re-run in 4-6 weeks to measure position movement

**Cadence guidance:**
- Striking distance positions shift slowly — re-run every 4-6 weeks minimum
- New discovery can run monthly (KE data is relatively stable)
- After running `/refresh-content` on a page, wait 3-4 weeks before re-checking its positions

---

## Error Handling

| Error | Action |
|-------|--------|
| GSC MCP not connected | Stop. Direct user to connect GSC. |
| GSC property not found | List available properties, ask user to select |
| No data in date range | Try 28-day fallback. If still empty, inform user. |
| KE API error / rate limit | Retry once with smaller batch. If fails again: proceed without volume data for affected queries, note in report. |
| KE not connected | Proceed with GSC impressions as proxy for volume. Warn prominently. |
| Webflow MCP not connected | Skip page cross-reference. Use GSC page URLs only. |
| Zero striking distance results | "All your ranking keywords are in positions 1–3 or 31+. Either your top-3 rankings are solid, or more GSC data is needed. Check back after more impressions accumulate." |
| Zero new discovery results | "No new keyword opportunities found above thresholds. Consider lowering volume minimum or expanding topics with Keywords Everywhere manually." |
| KE expansion returns no results | Note in report. Suggest running KE manually on top-performing pages. |
| Report directory creation fails | Output report to terminal only. Warn user about save failure. |

---

## Thresholds Reference

| Threshold | Value | Rationale |
|-----------|-------|-----------|
| Tier A min volume | 100/month | Below this, traffic gain is marginal |
| Tier A min impressions | 200/90d | Alternative qualifier without KE data |
| Tier B min volume | 200/month | Higher bar — more effort required |
| Tier B min impressions | 500/90d | Alternative qualifier |
| Tier C min volume | 500/month | High bar — significant effort needed |
| New gap min impressions | 50/90d | Enough signal to act on |
| KE expansion min volume | 200/month | Below this, new content ROI is low |
| KE expansion max competition | 0.7 | Above this, attainability drops significantly |
| Min volume for KE call | 50/month | Below this, filter before spending KE credits |
| Declining trend filter | > 30% YoY decline | Don't invest in dying queries |

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append:

**Full mode:**
```
| YYYY-MM-DD | /keywords-opportunity | Striking distance: N keywords across M pages (+X est. clicks/mo). New discovery: N opportunities. Top action: [top priority action]. |
```

**Striking mode:**
```
| YYYY-MM-DD | /keywords-opportunity:striking | Striking distance: N keywords across M pages. Tier A: N, Tier B: N, Tier C: N. |
```

**Discover mode:**
```
| YYYY-MM-DD | /keywords-opportunity:discover | New keyword discovery: N opportunities. Content gaps: N, KE expansion: N, Long-tail clusters: N. |
```

Log even on early exit (e.g., "Aborted: KE not connected. GSC-only analysis returned N striking distance keywords.").
