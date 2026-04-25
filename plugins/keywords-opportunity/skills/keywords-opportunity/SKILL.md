---
name: keywords-opportunity
version: "1.2"
description: |
  Discover keyword opportunities using GSC + Keywords Everywhere. Surfaces striking distance keywords already ranking on pages 1-3, uncovers new keywords worth targeting, and researches keywords for new pages, products, or clusters before you create them.
  Triggers: keyword opportunities, keyword research, keyword gaps, striking distance, new keywords, rank opportunities, keyword discovery, find keywords, keywords for new page, keywords for new product, keyword brief, what keywords should I target, launch new page, new product keywords, new cluster, topic cluster research.
  Requires: Google Search Console MCP server (not required for new-page/new-product/new-cluster modes). Strongly recommended: Keywords Everywhere MCP server.
  Workflow: Discover → Fetch → Clean → Enrich → Analyze → Score → Report → Save.
  Modes: /keywords-opportunity (full: striking distance + new discovery), /keywords-opportunity:striking (page 1-3 wins only), /keywords-opportunity:discover (new keywords only), /keywords-opportunity:new-page (keyword brief for a new page), /keywords-opportunity:new-product (keyword brief for a new product or service page), /keywords-opportunity:new-cluster (keyword map for a new topic cluster).
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
- **Optional**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) — used to cross-reference existing pages for gap detection and diagnosis

---

## Skill Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/keywords-opportunity` | Both striking distance + new discovery. Full report. |
| **Striking** | `/keywords-opportunity:striking` | Already-ranking keywords on pages 1-3. Fastest ROI. |
| **Discover** | `/keywords-opportunity:discover` | New keyword opportunities not yet targeted. |
| **New page** | `/keywords-opportunity:new-page` | Keyword brief for a new page before writing it. No GSC required. |
| **New product** | `/keywords-opportunity:new-product` | Keyword brief for a new product or service page. Commercial intent focus. |
| **New cluster** | `/keywords-opportunity:new-cluster` | Full keyword map for a new topic cluster — hub + editorial + product pages. |

**The new-\* modes have a different workflow** — they don't pull from existing rankings. They start from a description, research keywords from scratch via Keywords Everywhere, and output a keyword brief. See the dedicated section below.

---

## Workflow Overview

```
DISCOVER → FETCH → CLEAN → ENRICH → ANALYZE → SCORE → REPORT → SAVE
```

---

## Conditional Guards (Global)

⚡ GUARD — **Load SEO Copilot config:**
At the start of execution, check if `.claude/seo-copilot-config.json` exists:
- If yes: load `business.name` (branded query detection), `seo.primaryKeywords` (anchor topics for KE expansion), `seo.competitors` (filter competitor brand queries), `audience.primary` (frame recommendations)
- If no: proceed with defaults, note "Run `/getting-started` for personalized recommendations."

⚡ GUARD — **GSC MCP unavailable:**
- For full / striking / discover modes → Stop: "Keyword Opportunity requires Google Search Console. Connect GSC MCP and try again."
- For new-page / new-product / new-cluster modes → proceed without GSC. Note: "GSC not connected — cannibalization check skipped."

⚡ GUARD — **Keywords Everywhere unavailable:**
- Proceed, but add a prominent warning to the report header:
  ```
  ⚠️ Keywords Everywhere not connected.
  Volume data unavailable — opportunities ranked by GSC impressions only.
  Connect KE for volume-backed prioritization: https://github.com/hithereiamaliff/mcp-keywords-everywhere
  ```
- In discovery mode without KE: limit new opportunity output to GSC-derived content gaps only. Skip the KE expansion section.

⚡ GUARD — **Webflow MCP unavailable:**
- Continue without page cross-reference.
- Note: "Webflow MCP not connected — skipping keyword-to-page diagnosis. Gap detection based on GSC query patterns only."

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
- If it exists: read all entries and build a **resolved-issues set** — a list of URLs, keywords, or issue types that were logged as fixed (e.g., "Fixed cannibalization on /webflow-seo", "Refreshed /best-plugins"). This set is used in Phase 3 to suppress false positives.
- Surface a brief summary of recent activity:
  - If `/keywords-opportunity` was run within 30 days → warn: "Ran `/keywords-opportunity` on [date]. Rankings shift slowly — consider waiting 4+ weeks between full runs."
  - Note recent `/refresh-content` runs (pages optimized may now rank higher)
  - Note recent `/click-recovery` runs (CTR fixes — check if position also improved)
- If not found: proceed silently (resolved-issues set is empty)

### 0.4 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract primary keywords and competitors.

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

**Page-level data** (for context and 0-click detection):
- All pages with impressions > 10
- Fields: page, clicks, impressions, CTR, average position

⚡ GUARD — **Low data volume (< 500 total impressions):**
- Warn: "Low data volume. Striking distance analysis may lack statistical confidence. New discovery will rely more heavily on Keywords Everywhere."
- Proceed with caveats noted in report.

### 1.2 Webflow Page Inventory (if Webflow MCP available)

Fetch all published pages and CMS items:
- Static pages: path, SEO title, meta description
- CMS items (all collections): name, slug, meta title, meta description

Build a normalized URL→title map. Used in Phase 3 for gap detection and per-page diagnosis.

---

## Phase 1.3: DATA CLEANING

**Run before enrichment.** Clean the raw GSC query dataset to remove noise that would pollute scoring.

### Anomaly Filters

Remove queries matching any of the following patterns:

**Tracking ID leaks** — numeric or alphanumeric prefix followed by a colon or comma:
- Pattern: `^\d+[:\,]` (e.g., "5603: keywords, to, track", "6473: track rankings of keywords")
- These are instrumentation artifacts, not real search queries

**Percent-encoded characters** — URL-encoded sequences in queries:
- Pattern: `%[0-9A-Fa-f]{2}` (e.g., "webflow%20seo", "how%20to%20rank")
- These are broken URL fragments, not actual user queries

**Quoted anomalies** — queries that appear to be JSON fragments, code snippets, or structured data leaking into GSC:
- Pattern: starts with `{`, `[`, `"`, or contains `=>`, `!=`, `&&`

**Excessive punctuation** — queries with 3+ commas or semicolons (likely structured data):
- Pattern: more than 2 commas, or contains `;`

**Log all filtered queries:**
After cleaning, note in the report header:
```
⚠️ [N] queries removed during data cleaning (tracking IDs, encoded characters, anomalous patterns).
```
Only log if N > 0. Don't list the filtered queries in the report — just the count.

---

## Phase 2: ENRICH WITH KEYWORDS EVERYWHERE

**Run this phase before analysis.** Enrichment transforms raw GSC data into scoreable opportunities.

### 2.1 Volume Source Priority Rule

**GSC impressions are ground truth. KE volume is a supplement.**

Apply this rule to every query before scoring:

```
if KE_volume > 0:
    use_volume = KE_volume          # KE provides sampled market volume
    volume_source = "KE"
elif GSC_impressions_90d >= 50:
    use_volume = GSC_impressions_90d / 3   # rough monthly proxy (90d ÷ 3)
    volume_source = "GSC (KE=0)"    # flag: KE says 0 but GSC confirms real demand
else:
    use_volume = 0
    volume_source = "none"
    # consider filtering from striking distance unless impressions ≥ 200
```

When `volume_source = "GSC (KE=0)"`, label the query in the report as **"GSC-confirmed"** — it has real search demand that KE's sample missed. Treat it as equivalent to a low-volume KE result (50–100/month). Do NOT discard it or label it "emerging" and move on.

Examples of this rule in practice:
- "llm citation monitoring": KE=0, GSC impressions=140 → use_volume = 47/mo, flag as GSC-confirmed
- "webflow seo plugin": KE=320, GSC impressions=900 → use KE volume = 320/mo

### 2.2 Striking Distance Enrichment

For all cleaned GSC queries with average position 1–30:
- Batch fetch from KE: monthly search volume, CPC, competition score, 12-month trend data
- Apply the Volume Source Priority Rule (2.1) to each query
- Store: `{query, position, impressions, CTR, use_volume, volume_source, CPC, competition, trend_direction, trend_pct}`

**Trend direction classification** (required for every query — not optional):
```
if trend shows > 15% growth over last 6 months:  trend_direction = "↑ rising"
elif trend shows > 15% decline over last 6 months: trend_direction = "↓ declining"
else:                                              trend_direction = "→ flat"
```
Store trend_direction on every query. If KE returns no trend data for a query (unavailable), default to "→ unknown".

⚡ GUARD — **KE rate limits or API errors:**
- Retry once with smaller batch (50 queries). If it fails again: proceed with partial enrichment, apply Volume Source Priority Rule for failed queries using GSC impressions.

### 2.3 Discovery Expansion

For the top 20 topics by GSC impressions (group by primary keyword of each page):
- Fetch KE related keywords for each topic
- Fetch KE long-tail suggestions
- Store: `{related_keyword, topic_source, volume, CPC, competition, trend_direction}`

Filter out:
- Keywords already in the GSC dataset with position ≤ 30 (already captured in striking distance)
- Branded queries (queries containing `business.name` from config or site domain)
- Volume < 50/month (too small unless CPC > $5)
- Declining trend ↓ > 30% YoY

### 2.4 Compute Benchmarks

From the enriched dataset, compute:
- Site average CTR by position bracket (1-3, 4-10, 11-20)
- Total clicks across all pages (used for 0-click detection)
- Total impressions in the striking distance window (positions 4-30)
- Median monthly volume across all enriched queries

---

## Phase 3: ANALYZE

### 3.1 Site-Wide CTR Signal Detection

**Run before any other analysis.** Compute:

```
total_clicks    = sum of clicks across all GSC pages
total_impressions = sum of impressions across all GSC pages
site_ctr        = total_clicks / total_impressions
```

⚡ GUARD — **Zero-click or near-zero-click site:**

If `total_clicks < 10` AND `total_impressions > 500`:
- Set flag: `ctr_crisis = True`
- This is a **defining context** for the entire report — the site has search visibility but no traffic
- This changes the framing: the problem isn't just rankings, it's CTR
- The action plan must include `/click-recovery` as a parallel priority alongside `/refresh-content`
- This flag surfaces at the very top of the report — not buried at the bottom

### 3.2 Page-Level Consolidation

**The page is the unit of action — not the individual query.**

Group all cleaned, enriched queries by the ranking page URL:

For each page:
```
page_total_impressions = sum of impressions across all queries ranking from this page
page_top_position      = min(avg_position) across all queries for this page
page_total_clicks      = sum of clicks across all queries for this page
page_queries           = list of all queries, sorted by use_volume desc
page_tier              = tier of the highest-scoring query for this page (A, B, or C)
page_priority_score    = highest priority score across all queries for this page
```

**Why this matters:** A page ranking for 4 queries all in striking distance has 4× the leverage of a page ranking for 1 query. The combined impressions signal is stronger than any single query. Group before scoring.

For each page, select up to 5 representative queries to display (sorted by use_volume desc). Show combined impressions and total queries in the page header.

### 3.3 Per-Page Keyword-to-Title Diagnosis

For each page in the striking distance set (Webflow MCP required for full diagnosis):

1. Fetch the page's current SEO title and meta description (from Webflow inventory or GSC page title)
2. For the top 3 queries by volume on this page, check:
   - Is the query (or its root phrase) present in the SEO title?
   - Is the query (or its root phrase) present in the meta description?
3. Classify:
   - **Keyword absent from title** → CTR issue + ranking signal issue → `/click-recovery` first
   - **Keyword in title, CTR low** → pitch problem → `/click-recovery`
   - **Keyword in title and description, CTR ok, ranking stuck** → content depth → `/refresh-content`
   - **Multiple keywords, all absent from title** → significant on-page gap → `/click-recovery` then `/refresh-content`

Store diagnosis per page. This feeds the per-action-item diagnostic in the report.

### 3.4 Activity Log Cross-Reference

Before flagging any issue (cannibalization, schema gap, missing field, etc.):

Check the resolved-issues set built from the activity log (Phase 0.3):
- If a URL or keyword cluster appears in the resolved-issues set within the last 90 days → skip the flag and note "(previously addressed — verify still accurate)" instead
- If the log shows `/refresh-content` was run on a specific page within 30 days → lower confidence on striking distance findings for that page (the refresh may not be indexed yet)

This prevents surfacing issues that were already fixed in prior sessions.

### 3.5 Tier Classification

After grouping by page (3.2), classify each page into tiers based on its best-ranking query's position:

| Tier | Position of best query | Label | Effort |
|------|----------------------|-------|--------|
| A | 4–10 | Page 1 — not top 3 | Low |
| B | 11–20 | Page 2 | Medium |
| C | 21–30 | Page 3 | High |

**Volume/impression thresholds (page-level):**

Tier A:
- Keep page if: any query has use_volume ≥ 100/mo OR page_total_impressions ≥ 200

Tier B:
- Keep page if: any query has use_volume ≥ 200/mo OR page_total_impressions ≥ 400

Tier C:
- Keep page if: any query has use_volume ≥ 500/mo (high bar — significant effort)

Sort pages within each tier by `page_total_impressions` descending — the most-exposed pages appear first.

### 3.6 Quick Wins Selection

From the full striking distance set, auto-select 2–3 quick wins using:

```
quick_win_score = page_total_impressions × (1 / page_top_position) × volume_confidence_factor
```

Where:
- `volume_confidence_factor` = 1.5 if KE volume confirmed, 1.0 if GSC-confirmed, 0.5 if no volume data
- Rank all pages by quick_win_score
- Take top 2–3 that are NOT already in the resolved-issues set

These surface at the very top of the report before any tier tables.

### 3.7 New Keyword Discovery

**Source A — GSC content gaps:**
From the cleaned dataset, queries with average position > 30 AND impressions ≥ 50:
- Apply Volume Source Priority Rule — GSC impressions > 50 = real demand
- Group by semantic topic (cluster queries sharing 2+ words or intent)
- For each cluster: sum total impressions, take the highest-use_volume query as the "primary" keyword
- Cross-reference with resolved-issues set — skip clusters already addressed

**Source B — KE expansion keywords (if KE available):**
From Phase 2.3, filter to:
- Volume ≥ 200/month
- Not already in GSC dataset (positions 1–30)
- Competition ≤ 0.7 (attainable)
- Trend ↑ or → (exclude ↓ declining)
- Group by source topic

**Source C — Long-tail cluster consolidation:**
From GSC, clusters of 3+ queries that:
- Share a root phrase
- All rank position > 15
- Total cluster impressions > 200

### 3.8 Opportunity Scoring

**Striking distance — page-level priority score:**
```
volume_score    = min(page_max_volume / 500, 5)        # highest-volume query for this page
position_score  = (30 - page_top_position) / 5         # best position across page queries
impression_score = min(page_total_impressions / 200, 5) # combined exposure signal
trend_modifier  = 1.2 if any query ↑ rising else (0.8 if all queries ↓ declining else 1.0)
impact          = round(((volume_score + position_score + impression_score) / 3) × trend_modifier, 1)
confidence      = 4 if KE_volume > 0 else (3 if GSC_confirmed else 2)
effort          = tier_effort[tier]                    # A=1, B=2, C=3
priority        = (impact × confidence) / effort
```

Expected CTR by position (for gap calculation):
- Position 1: 28%, 2: 15%, 3: 11%, 4: 8%, 5: 7%, 6-10: ~4%, 11-20: ~1.5%

**New discovery — opportunity score:**
```
volume_score  = min(use_volume / 1000, 5)
intent_score  = min(CPC / 2, 5) if CPC > 0 else 2
gap_score     = 5 if no page exists else 3
trend_modifier = 1.2 if ↑ rising else (0.8 if ↓ declining else 1.0)
impact        = round(((volume_score + intent_score + gap_score) / 3) × trend_modifier, 1)
confidence    = 4 if from_KE else (3 if GSC_confirmed else 2)
effort        = 2 if existing_page else 4
priority      = (impact × confidence) / effort
```

Priority buckets (see CLAUDE.md for criteria):
- **Must do** (priority ≥ 8.0) — structural gap or confirmed cannibalization
- **High value** (priority ≥ 4.0) — cluster with 50+ impressions and no matching page, or keyword missing from title
- **Nice to have** (priority ≥ 1.5) — incremental coverage improvements

---

## Phase 4: REPORT

### 4.1 Report Header

```
# Keyword Opportunities — {domain}

**Date:** YYYY-MM-DD
**Period analyzed:** Last 90 days
**Data sources:**
- ✅ GSC: [X] queries analyzed, [Y] pages, [Z] total impressions, [W] total clicks
- ✅ Keywords Everywhere: Volume enriched for [N] queries [or ⚠️ Not connected — GSC impressions used as volume proxy]
- ✅ Webflow: [N] pages mapped for diagnosis [or ⚠️ Not connected — diagnosis limited]
- ✅ SEO Copilot Config: Loaded [or ℹ️ Not found]
- [N] queries removed during data cleaning (tracking IDs, anomalous patterns) [only if N > 0]

**Snapshot:**
- Striking distance: [N] pages across Tiers A/B/C — [X] combined impressions in window
- New opportunities: [N] keyword clusters — ~[X] monthly searches addressable
- Action plan: [N] must-pursue, [N] high-value, [N] worth tracking
```

⚡ **If `ctr_crisis = True`**, insert immediately after the header — before anything else:

```
---
🚨 SITE-WIDE CTR ISSUE

This site has [Z] impressions but only [W] clicks ([X]% CTR) over 90 days.
Normal CTR for page-1 rankings is 3–8%. This signals a structural title/description problem
affecting nearly all pages — not just isolated keyword gaps.

**This changes the priority of everything below.**
Before optimizing for rankings, fix CTR:
→ Run `/click-recovery` first to update meta titles and descriptions across high-impression pages.
→ Then use this report's striking distance findings to drive content improvements.

Striking distance analysis still applies — but `/click-recovery` should run in parallel.
---
```

---

### 4.2 Quick Wins (Always First)

Before any tier tables, show the 2–3 highest-confidence, highest-exposure opportunities. These are pre-decided — the reader doesn't need to parse the full report to find the best move.

```
## ⚡ Quick Wins — Do These First

Selected by: highest combined impressions + lowest position + confirmed demand.

### 1. {Page title or URL}
**Page**: {URL}
**Combined impressions**: {N} across {K} queries
**Best position**: {pos} for "{top query}" ({volume}/mo {↑/→/↓})
**Diagnosis**: {one-line reason — e.g., "Primary keyword absent from title" / "Ranking stuck at position 7 despite matching title — content depth issue" / "4 related queries all ranking 12-18, single refresh could consolidate"}
**Action**: Run `/refresh-content {URL}` [or `/click-recovery`]
**Expected lift**: +{X} clicks/month (conservative estimate)

### 2. {Page title or URL}
[same format]

### 3. {Page title or URL}
[same format]
```

---

### 4.3 Part 1 — Striking Distance

**Intro line**: "These pages already rank on pages 1–3. Sorted by combined impressions — highest exposure first."

#### Tier A — Page 1, Positions 4–10 (Fastest ROI)

For each page in Tier A, sorted by `page_total_impressions` descending:

```
### {Page Title or URL path}
**URL**: {URL}
**SEO title**: "{current title}"
**Combined impressions**: {N} ({K} queries ranking from this page)
**Diagnosis**: {one-line from Phase 3.3 — e.g., "Primary keyword not in title", "Keyword present but CTR is 0.8% vs expected 5% at position 6 — title hook issue", "All keywords in title, ranking stuck — content depth problem"}

| Query | Position | Volume/mo | CTR | Expected CTR | Gap | Trend | Source |
|-------|----------|-----------|-----|--------------|-----|-------|--------|
| {query} | {pos} | {vol} | {ctr}% | {exp}% | {gap}% | ↑/→/↓ | KE / GSC-confirmed |
| ...   |          |           |     |              |     |       |        |

**Action**: [one of:]
- "Keyword absent from title → `/click-recovery`"
- "Keyword in title, CTR low → `/click-recovery` (stronger hook)"
- "Keyword + CTR ok, ranking stuck → `/refresh-content {URL}` (content depth)"
- "CTR crisis + keyword gap → `/click-recovery` first, then `/refresh-content {URL}`"

**Priority**: [Must do / High value / Nice to have] (Score: {priority})
**Estimated impact**: +[X] clicks/month
```

#### Tier B — Page 2, Positions 11–20 (High Potential)

Summary table for all pages + full format for top 5 by priority:

```
| Page | Queries | Combined Impr. | Best Position | Top Query | Volume/mo | Trend | Priority | Action |
|------|---------|---------------|--------------|-----------|-----------|-------|----------|--------|
| {title} | {K} | {N} | {pos} | {query} | {vol} | ↑/→/↓ | {score} | `/refresh-content {url}` |
```

Full format for top 5 (same structure as Tier A).

#### Tier C — Page 3, Positions 21–30 (Worth Pursuing)

Summary table only:

```
| Page | Queries | Combined Impr. | Best Position | Top Query | Volume/mo | Trend | Priority |
|------|---------|---------------|--------------|-----------|-----------|-------|----------|
```

Note: "These pages rank on page 3. High-volume keywords here warrant a full content refresh. Run `/refresh-content {url}` targeting the listed queries."

---

### 4.4 Part 2 — New Keyword Opportunities

**Intro line**: "Topics you're not currently ranking for — or barely visible for. Ranked by traffic potential and attainability."

#### Content Gaps (GSC-derived)

```
| Keyword Cluster | Queries | Total Impr/90d | Est. Volume/mo | Trend | Intent | Action |
|-----------------|---------|---------------|----------------|-------|--------|--------|
| {cluster} | {K} | {N} | {vol} | ↑/→/↓ | info/commercial | Run `/write-blog` / Expand {URL} |
```

For each cluster with priority ≥ 8.0: show all queries in the cluster + a recommended article angle.

Volume column: show GSC estimate (impressions ÷ 3) with label "GSC-confirmed" if KE returned 0.

#### Expansion Keywords (Keywords Everywhere)

```
| Keyword | Volume/mo | CPC | Competition | Source Topic | Trend | Action |
|---------|-----------|-----|-------------|--------------|-------|--------|
| {keyword} | {vol} | ${cpc} | {0.0-1.0} | {topic} | ↑/→/↓ | Create / Expand {URL} |
```

Group by source topic. Note: "You already rank for '{source topic}' — these are related queries the same audience uses."

#### Long-tail Cluster Consolidation

```
### Cluster: "{root phrase}"
Queries in cluster:
- "{query 1}" — position {X}, {Y} impressions, {↑/→/↓}
- "{query 2}" — position {X}, {Y} impressions, {↑/→/↓}
- "{query 3}" — position {X}, {Y} impressions, {↑/→/↓}
Combined est. monthly searches: ~{Z}
Recommendation: {Create a dedicated article / Expand existing page at {URL}}
```

---

### 4.5 Action Plan

```
## Action Plan

### Must Pursue

| # | Page / Cluster | Impr. | Volume/mo | Trend | Diagnosis | Action | Skill |
|---|---------------|-------|-----------|-------|-----------|--------|-------|
| 1 | {page or cluster} | {N} | {vol} | ↑/→/↓ | {one-line reason} | {what to do} | `/refresh-content {url}` |

### High Value

[same table]

### Worth Tracking

[same table — condensed, no diagnosis column needed]
```

**Diagnosis column** must be filled for every Must Pursue and High Value row. One line, no hedging. Examples:
- "Primary keyword missing from title — CTR is 0.4× expected"
- "Content thin relative to competitors ranking above — needs depth"
- "4 related queries split across 2 pages — consolidate into one"
- "GSC shows 140 impressions but 0 clicks — title likely mismatches intent"
- "Ranking position 8 for [query] with no mention in meta description"

---

### 4.6 Decision Guide

```
| Situation | Right action | Skill |
|-----------|-------------|-------|
| Site-wide CTR < 1% with impressions | Fix titles and descriptions site-wide first | `/click-recovery` |
| Keyword absent from title, rank 4-20 | Add keyword to title | `/click-recovery` |
| Keyword in title, CTR still low | Stronger hook needed | `/click-recovery` |
| Keyword + CTR ok, rank stuck 4-20 | Strengthen content depth | `/refresh-content {url}` |
| Keyword rank 21-30, high volume | Full content refresh targeting this query | `/refresh-content {url}` |
| Multiple queries, same page, all rank 10-20 | Consolidated refresh to cover all | `/refresh-content {url}` |
| GSC-confirmed keyword (KE=0), impressions > 50 | Real demand — treat as low-volume opportunity | `/refresh-content {url}` |
| No page for keyword cluster | Create new article | New content |
| Long-tail cluster, thin existing page | Expand existing page | `/refresh-content {url}` |
| ↑ Rising keyword, mid-position | Prioritize over flat keywords — momentum is real | Act now |
| ↓ Declining keyword | Deprioritize unless extremely high volume | Defer |
```

---

---

## New Page / Product / Cluster Modes

These three modes share a single research pipeline but differ in intake, scoring, and output. They are inverted from the existing modes: instead of analyzing what you already rank for, they research what you *should* target before creating new content.

**GSC is optional** — used only to catch cannibalization with existing pages. **Keywords Everywhere is the primary data source** — warn if missing.

---

### NP Phase 1: INTAKE

Ask the user for the information needed to seed keyword research. Use a single message with all questions at once — don't ask one at a time.

**For `:new-page`:**
```
To research keywords for your new page, I need a few details:

1. What is the page about? (describe the topic, product, or question it answers — 1-3 sentences)
2. Who is the target audience?
3. What type of page is it? (blog post / landing page / static page / CMS item)
4. Primary goal: traffic / leads / conversions / authority
5. Any keywords you already have in mind? (optional — leave blank to start from scratch)
```

**For `:new-product`:**
```
To research keywords for your new product or service page, I need:

1. What is the product or service? (name + short description)
2. Who is the target audience?
3. What problem does it solve or what outcome does it deliver?
4. Price range or tier: free / low-cost / mid-market / premium / enterprise
5. Any direct competitors you're aware of? (optional)
6. Any keywords you already have in mind? (optional)
```

**For `:new-cluster`:**
```
To plan keywords for a new topic cluster, I need:

1. What is the broad topic area? (e.g., "Webflow CMS optimization" or "B2B SaaS onboarding")
2. Who is the target audience?
3. Do you have any existing pages on this topic? (URLs — leave blank if starting from scratch)
4. What's the commercial goal — does this cluster lead to a product or service page?
5. Any seed keywords you already have in mind? (optional)
```

Wait for user response before proceeding.

---

### NP Phase 2: SEED KEYWORD GENERATION

From the intake, generate 8–12 seed keyword variations internally before fetching KE data. Do not show this list to the user — it's used to prime the KE queries.

**Seed generation rules:**

1. **Root phrase**: the clearest 2-3 word description of the topic (e.g., "webflow seo tools")
2. **Variants**: add qualifiers — "best", "for [audience]", "how to", "[root] guide", "[root] checklist"
3. **Long-tail variants**: add specificity — "best [root] for [use case]", "[root] compared", "[root] vs [alternative]"
4. **Commercial variants** (for `:new-product` only): add "price", "cost", "buy", "review", "[product name] vs", "[product category] software"
5. **Question variants**: "what is [root]", "how to [action]", "why [problem]"

For `:new-cluster`: generate seeds per intended page type (hub, editorial, product) separately.

---

### NP Phase 3: KE RESEARCH

⚡ GUARD — **KE unavailable for new-\* modes:**
```
⚠️ Keywords Everywhere isn't connected.
New page/product/cluster modes rely on KE for volume and competition data.
Without it, I can only suggest keywords based on your description — with no volume validation.

Options:
1. Connect Keywords Everywhere: https://github.com/hithereiamaliff/mcp-keywords-everywhere
2. Continue without volume data (keyword suggestions only — no confidence in search demand)
```
Wait for user choice. If they continue without KE: note in brief, skip volume-dependent scoring.

**If KE available:**

**Step 1 — Volume check on seeds:**
- Fetch: monthly volume, CPC, competition, 12-month trend for all seed keywords
- Apply Volume Source Priority Rule from Phase 2.1 (KE volume > GSC impressions if available)

**Step 2 — Related keyword expansion:**
- For the top 3 seeds by volume: fetch KE related keywords
- For the top 3 seeds by CPC (`:new-product` mode): fetch KE related keywords separately
- Deduplicate across all fetched results

**Step 3 — PASF (People Also Search For):**
- Fetch PASF keywords for the top seed by volume
- These reveal adjacent intent — important for `:new-cluster` editorial page planning

**Step 4 — Filter:**
Remove from consideration:
- Volume < 30/month AND CPC < $1 (too small to act on)
- Declining trend ↓ > 30% YoY (avoid shrinking topics)
- Unrelated to the described topic (semantic drift from expansion)

**Step 5 — Intent classification:**

Classify each remaining keyword by intent:

| Intent | Signal | Role |
|--------|--------|------|
| Informational | "how to", "what is", "guide", "checklist", "tips", "tutorial" | Blog / editorial |
| Commercial | "best", "vs", "review", "alternative", "compare" | Editorial → Product |
| Transactional | "buy", "price", "cost", "hire", "[product] for [use case]" | Product / landing page |
| Navigational | brand name queries | Ignore for new pages |

For `:new-product`: prioritize commercial + transactional. Informational keywords go in the "supporting content" section.
For `:new-page` (blog): prioritize informational. Commercial keywords go in "related opportunities" section.
For `:new-cluster`: sort all intent types — they map to different page roles.

---

### NP Phase 3.5: TOPIC MAP CROSS-REFERENCE

If `.claude/reports/{domain}/latest-topic-map.md` exists:
- Read it before proceeding
- For each candidate primary keyword: check if the topic map already has a page assigned to this keyword or cluster
- If a page is already mapped to this keyword → note: "This keyword is already mapped to {URL} in your topic map. Creating a new page risks cannibalization. Consider expanding {URL} instead."
- If the candidate cluster has ❌ Missing status in the topic map → confirm this is a genuine gap and flag it as validated in the brief

This avoids recommending pages that are already planned or in progress.

---

### NP Phase 4: CANNIBALIZATION CHECK

If GSC is connected AND Webflow MCP is connected:

For each candidate primary keyword (top 5 by volume/CPC):
- Check GSC: does the site already rank for this keyword on any page?
- Check Webflow inventory: does a page already exist targeting this keyword?

If a page already exists and ranks for the keyword:
```
⚠️ Cannibalization risk: "{keyword}" — your existing page at {URL} already ranks at position {pos}.
Creating a new page targeting this keyword would split authority and likely hurt both pages.
Recommendation: {expand existing page / differentiate clearly / use as secondary keyword instead}
```

Flag the keyword as `⚠️ cannibalizes {URL}` in the brief output. Do not remove it — surface it for the user to decide.

---

### NP Phase 5: SCORE AND SELECT

Score every candidate keyword using:

```
volume_score      = min(volume / 500, 5)
commercial_score  = min(CPC / 2, 5)   # proxy for intent value
attainability     = (1 - competition) × 5   # competition 0-1 → attainability 0-5
trend_modifier    = 1.2 if ↑ else (0.8 if ↓ else 1.0)
opportunity_score = round(((volume_score + commercial_score + attainability) / 3) × trend_modifier, 1)
```

**Primary keyword selection:**

Select the keyword with the highest opportunity_score that:
- Matches the page goal (informational for blog, transactional/commercial for product)
- Has attainability > 2.0 (competition < 0.6)
- Is not flagged as a cannibalization risk (prefer a cannibalization-safe keyword)

If the highest-scoring keyword has a cannibalization flag, still surface it but recommend it as secondary.

**Secondary keywords (3–5):**

From remaining candidates, select by:
1. Same intent cluster as primary (closely related)
2. High volume, lower competition (attainability > 3.0)
3. Long-tail variants of the primary (lower competition, high specificity)

**LSI / supporting keywords (5–10 for `:new-cluster`):**

Lower-volume related terms that should appear naturally in the content. Not targeting pages directly — used to signal topical depth to search engines.

---

### NP Phase 6: OUTPUT — KEYWORD BRIEF

Output a structured brief the user can hand directly to `/write-blog`, `/cms-collection-setup`, or use to brief a writer.

---

**For `:new-page` and `:new-product`:**

```
# Keyword Brief — {page description}

**Generated**: YYYY-MM-DD
**Page type**: {blog post / landing page / product page / CMS item}
**Target audience**: {audience}
**Primary goal**: {traffic / leads / conversion}

---

## Primary Keyword

**{primary keyword}**
- Volume: {N}/mo | CPC: ${X} | Competition: {0.0–1.0} | Trend: {↑/→/↓}
- Intent: {informational / commercial / transactional}
- Rationale: {1 sentence — why this is the right primary keyword for this page and goal}
{if cannibalization risk}: ⚠️ Note: existing page at {URL} ranks for this — differentiate clearly

## Recommended URL Slug
`/{suggested-slug}`

## Recommended Title Tag (50–60 chars)
"{Suggested Title | Brand}"

## Recommended H1
"{Suggested H1}"

## Recommended Meta Description (140–155 chars)
"{Suggested meta description}"

---

## Secondary Keywords (target naturally in subheadings + body)

| Keyword | Volume/mo | CPC | Competition | Trend | Use in |
|---------|-----------|-----|-------------|-------|--------|
| {keyword} | {N} | ${X} | {0.0–1.0} | ↑/→/↓ | H2 / intro / FAQ |
| ... | | | | | |

---

## Related Keywords (for topical depth — not primary targets)

{keyword1}, {keyword2}, {keyword3}, {keyword4}, {keyword5}

---

## Content Angle

{2-3 sentences: recommended article format (tutorial / comparison / guide / listicle / case study), key sections to cover, what differentiates a strong answer for this keyword}

---

## Keyword Gaps to Address

Related keywords worth creating separate pages for later (don't stuff into this page):

| Keyword | Volume/mo | Suggested page | Priority |
|---------|-----------|----------------|----------|
| {keyword} | {N} | {/suggested-slug} | High / Medium |

---

## Next Steps

- → Run `/write-blog` with this brief to create the page
- → If a CMS collection: Run `/cms-collection-setup` first to confirm keyword fields exist
{if cannibalization risk}: → Run `/click-recovery` on {URL} first to differentiate the existing page before creating this one
```

---

**For `:new-cluster`:**

```
# Cluster Keyword Map — {cluster topic}

**Generated**: YYYY-MM-DD
**Cluster topic**: {topic}
**Target audience**: {audience}
**Commercial goal**: {product/service page at {URL} or "none identified"}

---

## Cluster Overview

| Page Role | Count | Keywords identified | Content gaps |
|-----------|-------|---------------------|--------------|
| Hub | 1 | {N} | {N} |
| Editorial | {N} | {N} | {N} |
| Product | {N} | {N} | {N} |

---

## Hub Page

**Target keyword**: {hub keyword}
- Volume: {N}/mo | CPC: ${X} | Competition: {0.0–1.0} | Trend: ↑/→/↓
- Slug: `/{suggested-slug}`
- Title: "{Suggested Title}"
- Purpose: anchor page for the cluster — covers the broad topic, links to all editorial and product pages

---

## Editorial Pages

For each recommended editorial page:

### Editorial {N}: {page title}
**Target keyword**: {keyword} | {N}/mo | ${CPC} CPC | {competition} | {↑/→/↓}
- Slug: `/{suggested-slug}`
- Title: "{Suggested Title}"
- Intent: {informational / commercial}
- Links to: Hub page + {product page slug if applicable}
- Secondary keywords: {keyword1}, {keyword2}

---

## Product Pages

For each product/service page identified:

### Product {N}: {page title}
**Target keyword**: {keyword} | {N}/mo | ${CPC} CPC | {competition} | {↑/→/↓}
- Slug: `/{suggested-slug}`
- Title: "{Suggested Title}"
- Intent: transactional / commercial
- Supporting editorial pages: {list}
- Secondary keywords: {keyword1}, {keyword2}

---

## Internal Linking Plan

| From | To | Anchor text | Why |
|------|----|-------------|-----|
| Hub | Editorial {N} | {suggested anchor} | Topic coverage |
| Editorial {N} | Product {N} | {suggested anchor} | Conversion path |
| Editorial {N} | Hub | {suggested anchor} | Authority flow |

---

## Creation Priority

| Page | Keyword | Volume/mo | Effort | Priority |
|------|---------|-----------|--------|----------|
| Hub | {keyword} | {N} | Low | Must do first |
| Editorial {N} | {keyword} | {N} | Medium | Create after hub |
| Product {N} | {keyword} | {N} | Low | May exist — check |

---

## Next Steps

- → Create Hub page first: Run `/write-blog` with primary keyword "{hub keyword}"
- → Then each editorial page in priority order
- → Check `/topic-map` after creating pages to verify cluster structure
```

---

### NP Phase 7: SAVE

Save the brief to:
- `.claude/reports/{domain}/keyword-brief-{slug}-YYYY-MM-DD.md` — timestamped
- `.claude/reports/{domain}/latest-keyword-brief.md` — overwrite each run

If no domain is set (GSC not connected): save to `./keyword-briefs/keyword-brief-{date}.md` in the working directory.

Create directories if needed.

---

### NP Activity Log

**After every new-\* mode run**, append to `.claude/reports/{domain}/activity-log.md`:

```
| YYYY-MM-DD | /keywords-opportunity:new-page | Keyword brief for "{primary keyword}". Volume: {N}/mo, CPC: ${X}. {N} secondary keywords. {Cannibalization: yes/no}. |
| YYYY-MM-DD | /keywords-opportunity:new-product | Product keyword brief for "{product name}". Primary: "{keyword}" ({N}/mo, ${X} CPC). {N} transactional keywords identified. |
| YYYY-MM-DD | /keywords-opportunity:new-cluster | Cluster map for "{topic}". {N} pages planned: 1 hub, {N} editorial, {N} product. Top keyword: "{keyword}" ({N}/mo). |
```

---

Save two files:
- `.claude/reports/{domain}/keywords-opportunity-YYYY-MM-DD.md` — full timestamped report
- `.claude/reports/{domain}/latest-keywords-opportunity.md` — overwrite on each run

Create directories if needed.

After saving:
```
Report saved to .claude/reports/{domain}/keywords-opportunity-YYYY-MM-DD.md

⚡ Quick wins:
1. [top quick win — page + action]
2. [second quick win]
3. [third quick win]

[If ctr_crisis]: 🚨 Site-wide CTR issue detected. Run `/click-recovery` before content work.

Run `/refresh-content {url}` to execute content optimizations.
Run `/click-recovery` to fix meta titles and descriptions for CTR gaps.
```

---

## Integration with Other Skills

This skill is **read-only** — it identifies opportunities and routes to the right execution skill.

| Finding | Right Skill | Why |
|---------|-------------|-----|
| Site-wide CTR crisis (0 or near-0 clicks) | `/click-recovery` | Fix titles and descriptions before anything else |
| Keyword absent from title, rank 4-20 | `/click-recovery` | Fastest ranking and CTR fix |
| Keyword present, rank stuck 4-20 | `/refresh-content {url}` | Content depth issue |
| Keyword rank 21-30 | `/refresh-content {url}` | Full content refresh targeting the query |
| Multiple queries same page, all rank 10-20 | `/refresh-content {url}` | Consolidated refresh |
| Page ranked but getting 0 clicks — AEO gap | `/aeo-optimize {url}` | Structured content + FAQ + schema to win featured snippets |
| High-volume new keyword gap | `/write-blog` | New article — run `/write-blog` targeting the cluster keyword |
| Long-tail cluster, thin existing page | `/refresh-content {url}` | Expand to cover the cluster |
| CMS missing keyword fields | `/cms-collection-setup:review` | Add primary/secondary keyword fields first |
| Launching a new page | `/keywords-opportunity:new-page` | Research keywords before writing — then feed brief into `/write-blog` |
| Launching a new product or service page | `/keywords-opportunity:new-product` | Commercial keyword brief → feeds `/write-blog` or Webflow page creation |
| Planning a new topic cluster | `/keywords-opportunity:new-cluster` → `/topic-map` | Build cluster keyword map, then verify structure with `/topic-map` |

**Recommended workflow:**
1. Run `/keywords-opportunity` monthly to refresh the opportunity map
2. If CTR crisis detected: run `/click-recovery` immediately (parallel to content work)
3. Execute Tier A quick wins with `/click-recovery` (CTR fixes) or `/refresh-content` (content depth)
4. For new keyword gaps with no matching page: run `/write-blog` targeting the cluster keyword
5. Plan Tier B and new content for the following sprint
6. Re-run in 4-6 weeks to measure position movement

**Cadence guidance:**
- Re-run every 4-6 weeks minimum (rankings shift slowly)
- After running `/refresh-content` on a page, wait 3-4 weeks before re-checking its position
- After running `/click-recovery`, wait 2-3 weeks before checking CTR impact

---

## Error Handling

| Error | Action |
|-------|--------|
| GSC MCP not connected | Stop. Direct user to connect GSC. |
| GSC property not found | List available properties, ask user to select |
| No data in date range | Try 28-day fallback. If still empty, inform user. |
| KE API error / rate limit | Retry once with smaller batch (50 queries). If fails again: apply Volume Source Priority Rule using GSC impressions for affected queries. |
| KE not connected | Apply Volume Source Priority Rule throughout — GSC impressions > 50 treated as real demand. |
| Webflow MCP not connected | Skip keyword-to-title diagnosis. Show page URLs only. |
| Zero striking distance results | "No ranking keywords in positions 4-30 above thresholds. Either top-3 rankings dominate, or more GSC data is needed." |
| Zero new discovery results | "No new keyword opportunities above thresholds. Consider lowering volume minimum." |
| KE expansion returns no results | Note in report. Suggest running KE manually. |
| All queries filtered by anomaly detection | This shouldn't happen — if > 50% of queries are filtered, log a warning and proceed with remaining. |
| Activity log cross-reference suppresses all findings | Warn: "All findings appear in recent activity log as resolved. Run `/keywords-opportunity` again in 4-6 weeks to see new opportunities." |
| Report directory creation fails | Output report to terminal only. Warn about save failure. |

---

## Thresholds Reference

| Threshold | Value | Rationale |
|-----------|-------|-----------|
| GSC impressions = real demand | ≥ 50/90d | Confirmed by Google's own data — KE=0 doesn't override |
| Tier A min volume | 100/mo or 200 impr/90d | Either qualifier passes |
| Tier B min volume | 200/mo or 400 impr/90d | Either qualifier passes |
| Tier C min volume | 500/mo | High bar — significant effort |
| New gap min impressions | 50/90d | Enough signal to act on |
| KE expansion min volume | 200/mo | Below this, new content ROI is low |
| KE expansion max competition | 0.7 | Above this, attainability drops |
| Rising trend threshold | > 15% growth in 6mo | Boosts confidence and priority score |
| Declining trend threshold | > 15% decline in 6mo | Reduces priority score |
| CTR crisis threshold | < 10 clicks, > 500 impressions | Site-wide CTR problem |
| Resolved-issues lookback | 90 days | Issues fixed earlier may have re-emerged |

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

**Full mode:**
```
| YYYY-MM-DD | /keywords-opportunity | Striking: N pages (Tier A: N, B: N, C: N). Discovery: N clusters. CTR crisis: [yes/no]. Top action: [top quick win one-liner]. |
```

**Striking mode:**
```
| YYYY-MM-DD | /keywords-opportunity:striking | Striking distance: N pages. Tier A: N, Tier B: N, Tier C: N. Top page: [URL] ({N} impr.). |
```

**Discover mode:**
```
| YYYY-MM-DD | /keywords-opportunity:discover | New keyword discovery: N opportunities. Content gaps: N, KE expansion: N, Long-tail clusters: N. |
```

**New page/product/cluster modes** — logged in the NP Phase 7 section above.

Log even on early exit (e.g., "Aborted: KE not connected. GSC-only: N striking pages found.").
