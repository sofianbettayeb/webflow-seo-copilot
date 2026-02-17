---
name: audit-deep
version: "1.0"
description: |
  Deep SEO & AEO maturity audit with GSC + Webflow data. Full indexation cross-reference, content gaps, cannibalization, CMS template analysis, and data-backed scoring across 4 dimensions and 5 maturity levels.
  Triggers: audit deep, deep audit, post-sale audit, implementation audit.
  Requires: GSC MCP server, Webflow MCP server. Optional: Keywords Everywhere for volume data.
  Workflow: Discover → Fetch → Check → Score → Report → Save.
  Command: /audit:deep
---

# Deep Audit Skill

Full data-backed SEO and AEO maturity audit. Uses GSC search analytics, Webflow CMS data, and URL inspection to score 4 dimensions across 5 maturity levels. Produces a comprehensive report with metadata audit, content gaps, cannibalization analysis, and indexation cross-reference.

This skill is **read-only** — it never modifies Webflow content.

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview)
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (for volume/intent enrichment)

## Maturity Model Reference

See `references/maturity-model.md` for the full rubric. Same model as `/audit` but evaluated with deep signals and higher confidence ceiling (Medium–High).

---

## Critical Guards

⚡ GUARD — **GSC MCP unavailable:**
- Stop: "Deep audit requires GSC. Connect GSC MCP and try again, or run `/audit {url}` for a quick assessment."

⚡ GUARD — **Webflow MCP unavailable:**
- Stop: "Deep audit requires Webflow MCP. Connect Webflow MCP and try again."

⚡ GUARD — **Low traffic (< 500 impressions in 90 days):**
- Warn: "Low traffic site. Statistical confidence is reduced."
- Proceed with caveats.

⚡ GUARD — **User requests abort:**
- Confirm, exit cleanly, output any partial results.

---

## Phase 0: DISCOVER

### 0.1 MCP Discovery

Search for tools BEFORE starting:

- **GSC MCP** (required): Search `+gsc search analytics`. If missing → stop.
- **Webflow MCP** (required): Search `+webflow data cms`. If missing → stop.
- **Keywords Everywhere** (optional): Search `+keywords everywhere volume`. If missing → note in report, proceed.

### 0.2 User Input

Prompt for:

```
Client email (required): [email]
```

Then use MCP to discover:
- GSC properties accessible
- Webflow sites/workspaces accessible
- Analytics sources available

If multiple matches: present a numbered selection list and ask user to choose.
If unique: auto-select and confirm.

### 0.3 Set {domain}

Extract domain from the selected GSC property URL. Normalize: strip `www.`, lowercase.

The `{domain}` is used for:
- **Report save path**: `./{domain}/reports/audit-deep-YYYY-MM-DD.md`
- **Latest pointer**: `./{domain}/reports/latest-deep.md`
- **Activity log**: `./{domain}/reports/activity-log.md`

### 0.4 Review Activity Log

Check `./{domain}/reports/activity-log.md`:
- If it exists: show recent activity summary (last 10 entries)
- **Redundancy check**: if `/audit:deep` was run in the last 30 days → warn with date
- Note if a quick audit exists (`./{domain}/reports/latest-quick.md`) — reference it for comparison
- If not found: proceed silently

### 0.5 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract:
- `business.name` → branded query detection
- `seo.competitors` → competitor mentions in queries
- `audience.primary` → framing recommendations

If not found: proceed with defaults, note "Run `/getting-started` for personalized recommendations."

---

## Phase 1: FETCH

### 1.1 GSC Data (90 days)

**Search analytics:**
- Page-level: all pages with impressions — URL, clicks, impressions, CTR, position
- Query-level: all queries with impressions > 5 — query, page, clicks, impressions, CTR, position

**URL inspection** (representative set):
- Top pages by impressions/clicks (top 20)
- Key template pages (one per CMS collection type)
- Index status, last crawl date, mobile usability

**Coverage/indexing signals** (if available via API surface).

### 1.2 Webflow Data

**Pages and collections:**
- `get_collection_list` → `list_collection_items` per collection
- For each CMS collection: fields, template settings, slug rules, SEO title/description mapping, Open Graph mapping
- All static pages: name, path, SEO title, meta description

**CMS field analysis per collection type:**
Identify best-practice field gaps. Recommended fields by type:
- **Blog**: keyword field, summary field, author reference, updated date, FAQ block support, schema toggles
- **Case study**: industry, services, outcomes metrics, testimonial entity fields
- **Landing page**: CTA field, hero text, social proof fields

**Template HTML:** Extract rendered HTML for key pages (one per collection type + homepage) to verify schema/tracking output matches config.

Sample first 100 items if a collection is large. Note limitation in report.

### 1.3 Web Data

Fetch via WebFetch:
- Homepage HTML (for schema, tracking scripts, site-wide signals)
- sitemap.xml (try `/sitemap_index.xml` as fallback)
- robots.txt

### 1.4 Keywords Everywhere (Optional)

If available: top 30 queries by impressions — volume, CPC, competition, trend.

---

## Phase 2: CHECK

Run all quick audit checks (see `/audit` skill) on the fetched homepage HTML, PLUS the deep-mode checks below. Every check produces: **Pass/Fail**, **Evidence**, **Source URL/template**.

### Content Checks (Deep)

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| CD1 | Query-to-page alignment | For each top GSC query, a relevant page exists (query appears in page's title or description) | Unmatched queries = content gaps |
| CD2 | No cannibalization | No query cluster has 2+ pages ranking within 5 positions of each other | Conflicting pages + positions |
| CD3 | No thin content | All CMS items have body content ≥ 300 words (or ≥ 50% of fields populated) | Items below threshold |
| CD4 | Topic coverage | GSC query clusters all have corresponding content pages | Uncovered clusters |
| CD5 | CMS field completeness | Per collection: % of items with all SEO fields populated (title, description, keywords) | Completion rate per collection |

### Technical Checks (Deep)

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| TD1 | Indexation cross-reference | All Webflow published pages appear in GSC | Not-indexed list, orphan-indexed list |
| TD2 | Metadata completeness | All CMS templates have title/description rules, not falling back to `name` only | Templates using `name` as title |
| TD3 | Schema coverage per template | Each content template type has appropriate schema (Article for blog, etc.) | Templates missing schema |
| TD4 | Crawl freshness | Top pages crawled within last 30 days | Pages with stale crawl |
| TD5 | Sitemap vs live pages | Sitemap URL count matches Webflow published page count (within 10%) | Mismatch count |
| TD6 | No duplicate meta titles | Zero duplicate `<title>` values across all pages | Duplicate groups |
| TD7 | No metadata length issues | All titles 30–60 chars, all descriptions 70–155 chars | Pages outside range |

### Authority Checks (Deep)

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| AD1 | Branded query volume | Branded queries (containing `business.name`) exist and have clicks | Branded click/impression counts |
| AD2 | Branded trend positive | Branded query clicks not declining month-over-month | Trend direction |
| AD3 | Author attribution coverage | Author byline present on ≥ 80% of blog CMS items | Coverage % |
| AD4 | Author pages exist | CMS has author collection or about page with author details | Author entity evidence |
| AD5 | E-E-A-T consistency | Experience/credential markers consistent across CMS items | Consistency score |

### Measurement Checks (Deep)

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| MD1 | GSC data freshness | GSC has ≥ 90 days of data, most recent data within 3 days | Date range |
| MD2 | Tracking on all templates | Analytics script present on all rendered template pages | Templates missing tracking |
| MD3 | Existing reports | `./{domain}/reports/` contains weekly or monthly reports | Report files found |
| MD4 | Conversion tracking | GA4 events or goal configuration detected | Event patterns found |

---

## Phase 3: SCORE

Same maturity model as `/audit`, evaluated with deep signals. Higher confidence ceiling.

### Scoring Engine

Same level-by-level gate evaluation:
```
score_dimension(dimension, evidence):
  for level in [1, 2, 3, 4, 5]:
    if all gates for this level pass:
      continue
    else:
      return level - 1 (or 1 if level 1 fails)
  return 5
```

### Content Dimension Gates (Deep)

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Quick C1 (title) AND C2 (H1) AND GSC shows ≥ 1 ranking query per indexed page |
| 2 | Level 1 AND CD1 (query-page alignment for top queries) AND quick C4 (FAQ) AND CD5 (≥ 50% CMS field completeness) |
| 3 | Level 2 AND CD2 (no major cannibalization) AND CD4 (topic coverage ≥ 80%) AND CD3 (no thin content) |
| 4 | Level 3 AND pillar pages rank for 50+ queries AND child pages link back AND CD5 (≥ 90% field completeness) |
| 5 | Level 4 AND automated freshness signals AND expanding query footprint (more queries month-over-month) |

### Technical Dimension Gates (Deep)

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Quick T1 (HTTPS) AND T2 (viewport) AND title tags exist on ≥ 80% of pages |
| 2 | Level 1 AND TD1 (all key pages indexed) AND TD6 (no duplicate titles) AND TD5 (sitemap matches) |
| 3 | Level 2 AND TD3 (schema on all content templates) AND TD2 (no templates using `name` as title) AND TD7 (metadata lengths ok) |
| 4 | Level 3 AND Organization + Person schema AND BreadcrumbList AND TD4 (crawl freshness < 30 days for top pages) |
| 5 | Level 4 AND zero technical debt AND programmatic schema AND automated monitoring |

### Authority Dimension Gates (Deep)

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline) |
| 2 | AD1 (branded queries exist) AND AD3 (author attribution ≥ 50%) |
| 3 | Level 2 AND AD2 (branded trend not declining) AND AD4 (author pages exist) AND AD5 (E-E-A-T consistent) |
| 4 | Level 3 AND high branded query ratio (≥ 20% of total queries) AND AI citation evidence |
| 5 | Level 4 AND dominant branded presence AND programmatic authority signals |

### Measurement Dimension Gates (Deep)

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline) |
| 2 | MD1 (GSC ≥ 90 days) AND analytics installed on homepage |
| 3 | Level 2 AND MD2 (tracking on all templates) AND MD3 (existing reports found) |
| 4 | Level 3 AND MD4 (conversion tracking) AND LLM citation data available |
| 5 | Level 4 AND automated monitoring AND alert-driven optimization |

### Overall Score

```
overall_level = min(content_score, technical_score, authority_score, measurement_score)
confidence = "Medium-High"
```

### Prioritization (Impact/Confidence/Effort)

Score every finding as a recommendation:

| Factor | Scale | Criteria |
|--------|-------|----------|
| Impact | 1-5 | Traffic at risk, % of total, business value |
| Confidence | 1-5 | Data volume, multi-signal confirmation |
| Effort | 1-5 | 1 = metadata fix, 2 = content tweak, 3 = section rewrite, 4 = new content, 5 = architecture change |

```
Priority = (Impact × Confidence) / Effort
```

| Bucket | Criteria | Action |
|--------|----------|--------|
| **Must fix** | Priority ≥ 8.0 | Address immediately |
| **High impact** | Priority ≥ 4.0 | First sprint |
| **Nice to have** | Priority ≥ 1.5 | Backlog |

---

## Phase 4: REPORT & SAVE

### 4.1 Report Structure

Output a single markdown file with these sections:

**Header:**
- Site, generated date, mode (Deep), GSC property, period (last 90 days)
- Confidence: Medium–High
- Data sources: GSC (days of data), Webflow (page/CMS item counts), KE (status), config (status)

**1. AEO Maturity Scorecard**
- Overall level + name
- Table: dimension, score 1–5, level name, key evidence, gap to next
- Weakest/strongest dimension, fastest win

**2. Findings per Dimension**
- Content: overview stats (total pages, avg queries/page, gaps, cannibalization), then detail
- Technical: overview stats, then detail
- Authority: brand presence metrics, E-E-A-T signal coverage
- Measurement: GSC health, tracking coverage, reporting cadence

**3. What's Broken & Consequences**
- Top issues with business impact (framed for executive communication)
- Table: issue, dimension, impact, consequence

**4. Action Plan (level-grouped, then prioritized within level)**
- **Level 1 blockers first** — anything preventing Level 1
- Then Level 2 requirements, Level 3, etc.
- Within each level: order by Priority score (Impact × Confidence / Effort)
- Each action: title, why (with data), impact, effort, steps, skill shortcut

**5. Roadmap to Next Level**
- Current → target level
- Explicit gates per dimension
- Table: action, dimension, effort, skill
- Estimated timeline

**Appendix Tables (deep mode only):**

- **Metadata audit table**: page/template, title rule, description rule, OG rule, issues
- **Content gaps table**: query cluster, impressions, suggested page type, proposed URL
- **Cannibalization table**: query cluster, pages competing, recommended consolidation action
- **Indexation cross-reference table**: Webflow URL, indexed yes/no, inspection status, notes

**Footer:**
- Next steps: `/click-recovery`, `/refresh-content`, `/weekly-report`, re-run `/audit:deep` after changes

### 4.2 Save to File

Save these files:
- `./{domain}/reports/audit-deep-YYYY-MM-DD.md` (timestamped)
- `./{domain}/reports/latest-deep.md` (overwrite each run — other skills read this)

Optionally save supporting datasets as JSON under `./{domain}/reports/data/`:
- `content-gaps.json`
- `cannibalization.json`
- `indexation.json`
- `metadata-audit.json`

Create directories if needed: `mkdir -p ./{domain}/reports/data/`

### 4.3 Activity Log

Append to `./{domain}/reports/activity-log.md`:

```
| YYYY-MM-DD | /audit:deep | Deep audit. Overall Level: X (Name). Content: X, Technical: X, Authority: X, Measurement: X. Must-fix items: N. |
```

Log even on early exit.

---

## Integration with Other Skills

| Finding | Skill | When |
|---------|-------|------|
| Low CTR / bad meta tags | `/click-recovery` | Quick metadata fixes |
| Outdated content | `/refresh-content {url}` | Full content refresh |
| Missing config | `/getting-started` | First-time setup |
| Quick pre-sale assessment | `/audit {url}` | Before connecting MCP |
| Ongoing monitoring | `/weekly-report` | Track progress weekly |
| Monthly progress | `/monthly-report` | Month-end review |

**Workflow:**
1. `/audit {url}` for pre-discovery quick assessment
2. `/audit:deep` post-sale for full baseline
3. Fix issues with `/click-recovery` and `/refresh-content`
4. `/weekly-report` to track progress
5. Re-run `/audit:deep` quarterly to measure maturity progression
