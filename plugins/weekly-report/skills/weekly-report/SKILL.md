---
name: weekly-report
version: "1.0"
description: |
  Weekly SEO pulse — what changed this week, how it tracks against the month, and what to do now. Compares Week W vs W-1, scores recommendations by impact/confidence/effort, and outputs an actionable report with monthly progress context.
  Triggers: weekly report, seo weekly, weekly review, weekly performance, traffic this week, weekly analysis.
  Requires: Google Search Console MCP server, Webflow MCP server. Optional: Keywords Everywhere API for volume/intent enrichment.
  Workflow: Discover → Fetch → Analyze → Score → Output.
  Modes: /weekly-report (full report), /weekly-report:quick (executive summary + action plan only).
---

# Weekly Report Skill

A weekly SEO pulse for founders and operators. Compares this week vs last week, identifies what changed and why, scores every recommendation, and outputs a prioritized action plan — with monthly progress context when available.

This skill is **read-only** — it analyzes GSC + Webflow data but never modifies Webflow content. It points you to `/click-recovery` and `/refresh-content` for execution.

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) (for page inventory and metadata audit)
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (needs API key — for search volume and intent enrichment on top queries)

## Skill Modes

Default is full report. Use `/weekly-report:quick` to output only the executive summary and action plan (skips rendering sections 2–5 but still runs all analysis phases).

## Workflow Overview

```
DISCOVER → FETCH → ANALYZE → SCORE → OUTPUT
```

---

## Critical Guards

These guards can stop execution. All other edge cases (no branded queries, zero declining pages, zero content gaps) are handled inline — skip the section and note why.

⚡ GUARD — **GSC MCP unavailable:**
- Stop execution: "Weekly Report requires Google Search Console. Please connect GSC MCP and try again."

⚡ GUARD — **Webflow MCP unavailable:**
- Stop execution: "Weekly Report requires Webflow MCP for page inventory and metadata analysis. Please connect Webflow MCP and try again."

⚡ GUARD — **Very low weekly traffic (<50 impressions):**
- Warn: "Very low traffic this week (<50 impressions). Data may be insufficient for meaningful analysis. Proceed anyway?"
- If declined: stop execution

⚡ GUARD — **User requests abort:**
- Confirm: "Stop the workflow? Progress will be lost."
- If confirmed: exit cleanly, output any partial results

---

## Phase 0: DISCOVER

MCP tools, config, last monthly report, date ranges, site selection.

### 0.1 MCP Discovery

Search for tools BEFORE starting analysis:

- **GSC MCP** (required): Search `+gsc search analytics`. If missing → stop (see guard above).
- **Webflow MCP** (required): Search `+webflow data cms`. If missing → stop (see guard above).
- **Keywords Everywhere** (optional): Search `+keywords everywhere volume`. If missing → note in report header, proceed without.

### 0.2 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract:
- `business.name` → branded query split
- `brandVoice` → recommendation tone
- `seo.metaTitleFormat` → metadata audit reference
- `seo.competitors` → competitor query detection
- `audience.primary` → framing recommendations

If not found: proceed with defaults, note "Run `/getting-started` for personalized recommendations."

### 0.3 Select Site & Set {domain}

List available GSC properties. If multiple, ask user to select. Extract the domain from the selected property URL (e.g., `https://www.checklist-seo.com/` → `checklist-seo.com`).

The `{domain}` value is used throughout for:
- **Report save path**: `.claude/reports/{domain}/weekly-report-YYYY-WXX.md`
- **Monthly report lookup**: `.claude/reports/{domain}/monthly-report-*.md`

### 0.4 Review Activity Log

Once `{domain}` is set, check `.claude/reports/{domain}/activity-log.md`:
- If it exists: read the last 10 entries and surface a brief summary:
  ```
  Recent activity on {domain}:
  - 2026-02-10 /click-recovery — Updated meta titles on 5 pages
  - 2026-02-05 /weekly-report — Week W06 report. Health: STABLE.
  - 2026-02-01 /monthly-report — January 2026 report. Health: UP.
  ```
- **Redundancy check**: if `/weekly-report` was already run for the same week → warn: "You already ran `/weekly-report` for Week [XX] on [date]. Run again anyway?"
- If the log doesn't exist: proceed silently (first run for this domain)

Use recent activity as context for recommendations (e.g., "Since `/click-recovery` was run 2 days ago, CTR changes may not be visible yet").

⚡ GUARD — **Report already exists:**
Check if `.claude/reports/{domain}/weekly-report-YYYY-WXX.md` already exists:
- If yes: "A report for Week [XX] already exists. Overwrite?"
- If declined: stop

### 0.5 Load Last Monthly Report

Search for the most recent `.claude/reports/{domain}/monthly-report-*.md`:
- If found: extract executive summary metrics (clicks, impressions, CTR, position), health trend, and action plan items
- If not found: proceed without, note "No monthly report found. Run `/monthly-report` for month-level context."

### 0.6 Calculate Date Boundaries

Compute 7-day windows (Mon–Sun). If today is mid-week, use the most recent complete window as W.

| Period | Label | Use |
|--------|-------|-----|
| Week W | Most recent complete Mon–Sun | Primary analysis |
| Week W-1 | Previous 7 days | Comparison baseline |
| Week W-2 | Two weeks ago | 4-week trend |
| Week W-3 | Three weeks ago | 4-week trend |

---

## Phase 1: FETCH

GSC data, Webflow pages + CMS items, URL inspection sample.

### 1.1 GSC Performance Data

Fetch for W and W-1 (optionally W-2, W-3 for trend):

- **Page-level**: all pages with any impressions — URL, impressions, clicks, CTR, position
- **Query-level**: all queries with impressions > 5 — query, page, impressions, clicks, CTR, position

### 1.2 Webflow Page Inventory

**CMS items**: `get_collection_list` → `list_collection_items` per collection. Extract: name, slug, published date, meta title, meta description. Sample first 100 if collection is large.

**Static pages**: `list_pages`. Extract: name, path, SEO title, meta description.

### 1.3 GSC URL Inspection (Top 5)

For the top 5 pages by impressions, run URL inspection:
- Index status, last crawl date, mobile usability
- If rate-limited: reduce to top 3, note in report

### 1.4 Keywords Everywhere Enrichment (Optional)

If available, enrich top 30 queries: search volume, CPC, competition, trend.

---

## Phase 2: ANALYZE

Metrics, content performance, template SEO audit, metadata audit, striking distance.

### 2.1 Executive Summary Metrics

Calculate W vs W-1:
- Total clicks (absolute + %)
- Total impressions (absolute + %)
- Average CTR
- Average position
- Indexed page count (from inspection or GSC page count)

**Health trend:**
- **UP**: clicks ≥ +5% AND (impressions up OR position improved)
- **DOWN**: clicks ≥ -5% AND (impressions down OR position worsened)
- **STABLE**: everything else

**Identify:** biggest win (largest click increase), biggest risk (largest click decrease), top 3 priorities (from scoring engine).

**Monthly context** (if monthly report available): month-to-date aggregates vs last monthly totals — on track / ahead / behind.

### 2.2 Performance Overview

- 4-week trend table: clicks, impressions, CTR, position for W-3 through W
- Branded vs non-branded split (using `business.name` from config)
- Note that conversion data requires GA4 or similar

### 2.3 Content Performance

- **Top 10 pages** by clicks in W (with W-1 comparison)
- **Top 5 growing** — largest click increase W vs W-1 (min 5 clicks in W)
- **Top 5 declining** — largest click decrease W vs W-1 (min 5 clicks in W-1)
- **Content gaps** — GSC queries with high impressions not matching any page title or description

### 2.4 Template SEO Audit

Check how CMS collections handle SEO titles:
- For each collection, inspect the schema fields
- **Flag if collection uses `name` field as the SEO title** instead of a dedicated `seo-title` or `meta-title` field
- Note: "Collection '[name]' uses the `name` field for SEO titles. Consider adding a dedicated SEO title field for independent control over SERP presentation."
- Check if `meta-description` field exists and is populated across items

### 2.5 Metadata Audit

For all Webflow pages (CMS + static):
- Missing meta title / missing meta description
- Duplicate meta titles across pages
- Title too long (>60 chars) / too short (<30 chars)
- Description too long (>155 chars) / too short (<70 chars)

**Indexation cross-reference:**
- Normalize URLs (strip scheme + domain from GSC, combine collection slug + item slug for CMS, lowercase, strip trailing slashes)
- Webflow pages not in GSC = potentially not indexed
- GSC pages not in Webflow = indexed orphans (old/deleted)

### 2.6 On-Page SEO Opportunities

**High impressions, low CTR:**
- Pages in the top 10% by impressions with CTR below site average for their position bracket
- These are candidates for `/click-recovery`

**Striking distance keywords (position 5–15):**
- Sorted by impressions (highest potential first)

**Keyword mismatches:**
- Top GSC query for each page vs meta title — flag if #1 query doesn't appear in title

### Adaptive Thresholds

For sites with < 5,000 weekly impressions, replace absolute thresholds with relative ones:

| Threshold | Standard (≥5K impressions) | Small site (<5K impressions) |
|-----------|---------------------------|------------------------------|
| Content gaps | >100 impressions | Top 5% of queries by impressions |
| High impressions / low CTR | Top 10% by impressions + CTR below average | Same (already relative) |
| Min clicks for growing/declining | 5 clicks | Top 10 pages by click change (any direction) |
| Very low traffic warning | <50 total impressions | Same (absolute floor) |

---

## Phase 3: SCORE

Same scoring engine as `/monthly-report`, with weekly confidence adjustments.

### 3.1 Scoring Dimensions

| Dimension | Scale | Criteria |
|-----------|-------|----------|
| **Impact** | 1-5 | Clicks at risk or recoverable, % of total site traffic, conversion proximity |
| **Confidence** | 1-5 | Data strength, multi-week confirmation, source reliability. **Capped for weekly data** (see below). |
| **Effort** | 1-5 | 1 = metadata change, 2 = content tweak, 3 = section rewrite, 4 = new section/FAQ, 5 = full page creation |

### 3.2 Weekly Confidence Caps

| Condition | Max Confidence |
|-----------|---------------|
| Single-week observation only | 3 |
| 2 consecutive weeks same direction | 4 |
| 3+ weeks same direction | 5 |
| Weekly impressions < 50 for a page | 2 |
| Trend confirmed in last monthly report | 5 |

### 3.3 Priority & Buckets

```
Priority = (Impact × Confidence) / Effort
```

| Bucket | Criteria | Action |
|--------|----------|--------|
| **Must fix** | Priority ≥ 8.0 AND (Impact ≥ 4 OR indexation issue) | Address this week |
| **High impact** | Priority ≥ 4.0 AND Impact ≥ 3 | Schedule within 3 days |
| **Nice to have** | Priority ≥ 1.5 | Add to backlog |

Recommendations with Priority < 1.5 are excluded (noise reduction).

### 3.4 Recommendation Format

Each recommendation includes:
1. **Action-oriented title** — what to do
2. **Why it matters** — 1-2 lines with metrics
3. **Expected impact** — qualitative + quantified
4. **Effort** — Low / Medium / High
5. **Steps** — Webflow-specific instructions
6. **Skill shortcut** — e.g., "Run `/click-recovery`"
7. **Monthly cross-reference** — if flagged in last monthly report, note "Also flagged in monthly report — still open"

---

## Phase 4: OUTPUT

Show executive summary + action plan in terminal. Save full report to `.claude/reports/{domain}/`.

### 4.1 Report Structure

The saved report is a Markdown file with these sections:

**Header:**
- Site, period (W dates vs W-1 dates), generated date
- Data sources: GSC, Webflow, KE (connected / not), config (loaded / not), monthly report (loaded / not)

**Section 1 — Executive Summary:**
- Health trend (UP / STABLE / DOWN)
- W vs W-1 metrics table (clicks, impressions, CTR, position, indexed pages)
- Biggest win, biggest risk, top 3 priorities
- Monthly context box: month-to-date vs last monthly totals, open action items count

**Section 2 — Performance Overview:**
- 4-week trend table (W-3 through W)
- Branded vs non-branded split
- Conversions note (cross-reference with GA4)

**Section 3 — Content Performance:**
- Top 10 pages, top 5 growing, top 5 declining, content gaps

**Section 4 — Technical SEO Health:**
- Template SEO audit (seo-title vs name field usage)
- Indexation cross-reference (not indexed, indexed orphans)
- Metadata audit (missing, duplicate, length issues)

**Section 5 — On-Page SEO Opportunities:**
- High impressions / low CTR pages
- Striking distance keywords (position 5–15)
- Keyword mismatches (top query not in title)

**Section 6 — Action Plan:**
- Must Fix (Priority ≥ 8.0) — full format with steps and shortcuts
- High Impact (Priority ≥ 4.0) — full format
- Nice to Have (Priority ≥ 1.5) — condensed (title, why, effort, shortcut)
- Monthly report cross-reference: open/completed items from last monthly report

**Footer:**
- Next steps: run `/click-recovery`, `/refresh-content`, re-run `/weekly-report` next week, `/monthly-report` at month-end
- Monitoring: 3 days (quick fixes live), 1 week (re-run weekly), month-end (monthly report)

In **quick mode** (`/weekly-report:quick`): display only Section 1 and Section 6 in terminal. Still save full report to file.

### 4.2 Save to File

Save to `.claude/reports/{domain}/weekly-report-YYYY-WXX.md` (ISO week number).

- Create directory if needed: `mkdir -p .claude/reports/{domain}/`
- If directory creation fails: output in terminal only, warn user

### 4.3 Suggest Next Skills

Based on findings:
- CTR issues → `/click-recovery`
- Declining content → `/refresh-content [URL]`
- Content gaps → new content targeting top gap queries
- Always → re-run `/weekly-report` next week, `/monthly-report` at month-end

---

## Integration with Other Skills

Weekly Report is **read-only**. Other skills execute.

| Finding | Skill | Why |
|---------|-------|-----|
| Low CTR pages | `/click-recovery` | Fix meta titles and descriptions |
| Declining content | `/refresh-content [URL]` | Full content refresh |
| Missing metadata | `/click-recovery` | Quick metadata fixes |
| Content gaps | Manual / `/refresh-content` | Create or update content |
| Keyword mismatches | `/click-recovery` | Align titles with search queries |
| Template SEO issues | Manual | Add dedicated SEO title fields to CMS collections |

**Cadence:**
1. `/weekly-report` every week for pulse
2. Execute with `/click-recovery` and `/refresh-content`
3. `/monthly-report` at month-end for the full picture

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append a row:

```
| YYYY-MM-DD | /weekly-report | [one-line summary: e.g., "Week W07 report. Health: UP. 2 must-fix, 3 high-impact items. Saved to weekly-report-2026-W07.md"] |
```

The `{domain}` is already set in Phase 0 (from GSC property selection). Log even if the skill exits early due to a guard — note why (e.g., "Aborted: GSC MCP unavailable").
