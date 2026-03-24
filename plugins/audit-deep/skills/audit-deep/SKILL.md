---
name: audit-deep
version: "1.2"
description: |
  Deep SEO & AEO maturity audit with GSC + Webflow data. Post-sale baseline that scores 4 dimensions across 5 maturity levels with data-backed evidence. Produces a client-ready report with quantified opportunities, engagement plan, and appendix data tables. Works with Webflow, WordPress, and other CMS platforms.
  Triggers: audit deep, deep audit, post-sale audit, implementation audit.
  Requires: GSC MCP server. Recommended: Webflow MCP server. Optional: PageSpeed MCP, Keywords Everywhere.
  Workflow: Discover → Fetch → Check → Score → Report → Save.
  Command: /audit:deep
---

# Deep Audit Skill

Post-sale SEO and AEO maturity audit with full data access. Uses GSC search analytics, Webflow CMS data, and URL inspection to score 4 dimensions across 5 maturity levels. Produces a client-ready engagement baseline with quantified opportunities, a phased roadmap, and supporting data tables.

This report is designed as a professional deliverable — the foundation for an engagement plan. It frames findings as business opportunities with data-backed impact estimates, and ends with a clear engagement structure.

This skill is **read-only** — it never modifies Webflow content.

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Recommended**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) — enables CMS schema checks (TD1–TD3, TD5, CD3, CD5). If unavailable, the skill runs in GSC-only mode and skips Webflow-dependent checks. Works with any CMS (WordPress, Webflow, Squarespace, etc.).
- **Optional**: [PageSpeed MCP](https://github.com/your-pagespeed-mcp) (for Core Web Vitals — LCP, TBT, CLS, mobile score)
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (for volume/intent enrichment)

## Maturity Model Reference

See `references/maturity-model.md` for the full rubric. Same model as `/audit` but evaluated with deep signals and higher confidence ceiling (Medium–High).

---

## Critical Guards

⚡ GUARD — **GSC MCP unavailable:**
- Stop: "Deep audit requires GSC. Connect GSC MCP and try again, or run `/audit {url}` for a quick assessment."

⚡ GUARD — **Webflow MCP unavailable:**
- Warn: "Webflow MCP not connected. Running in GSC-only mode — CMS schema checks (field completeness, indexation cross-reference, template schema) will be skipped. Connect Webflow MCP for full coverage, or continue with reduced scope."
- Continue with GSC + web signals. Mark all Webflow-dependent checks as `N/A — GSC-only mode` in the report.

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
- **Webflow MCP** (recommended): Search `+webflow data cms`. If missing → warn and continue in GSC-only mode.
- **PageSpeed MCP** (optional): Search `+pagespeed performance audit`. If missing → note, will skip Core Web Vitals section.
- **Keywords Everywhere** (optional): Search `+keywords everywhere volume`. If missing → note in report, proceed.

### 0.2 CMS Platform Detection

Before prompting for user input, detect the CMS platform from the homepage HTML. Fetch the homepage and look for:

| Signal | Platform |
|--------|----------|
| `/wp-content/uploads/` or `wp-json` in HTML | WordPress |
| `data-wf-site` attribute or `webflow.com` script | Webflow |
| `cdn.shopify.com` | Shopify |
| `static.wixstatic.com` | Wix |
| `squarespace-cdn.com` | Squarespace |
| No match | Unknown / Custom |

Record `{platform}` and use it to:
- Determine which CMS checks are applicable (see Phase 2 check tags)
- Adapt the report header data sources line
- If Webflow is **not** detected but Webflow MCP is connected → warn: "Webflow MCP is connected but this site does not appear to be Webflow. CMS checks will be skipped."
- If WordPress is detected but Webflow MCP is the only CMS MCP → automatically enter GSC-only mode without asking

### 0.3 User Input

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

### 0.4 Set {domain}

Extract domain from the selected GSC property URL. Normalize: strip `www.`, lowercase.

The `{domain}` is used for:
- **Report save path**: `./{domain}/reports/audit-deep-YYYY-MM-DD.md`
- **Latest pointer**: `./{domain}/reports/latest-deep.md`
- **Activity log**: `./{domain}/reports/activity-log.md`

### 0.5 Review Activity Log

Check `./{domain}/reports/activity-log.md`:
- If it exists: show recent activity summary (last 10 entries)
- **Redundancy check**: if `/audit:deep` was run in the last 30 days → warn with date
- Note if a quick audit exists (`./{domain}/reports/latest-quick.md`) — reference it for comparison
- If not found: proceed silently

### 0.6 Load Config

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

**Large dataset handling:** If query+page data is very large (thousands of rows), do not read the full file into context. Instead, extract targeted analyses:
- Top 100 queries by impressions
- Top 50 queries by clicks
- Cannibalization candidates: queries where 2+ pages appear — use a targeted script (`jq` or `python3`) rather than reading the raw file
- Sample first 200 rows for pattern analysis

Note any sampling in the report footnote.

### 1.2 Webflow Data `[Webflow only — skip if GSC-only mode]`

**Pages and collections:**
- `get_collection_list` → `list_collection_items` per collection
- For each CMS collection: fields, template settings, slug rules, SEO title/description mapping, Open Graph mapping
- All static pages: name, path, SEO title, meta description

**CMS schema review per collection:**
For each blog/article/case study collection, run the recommended schema check against the five groups defined in `/cms-collection-setup`:

| Group | Fields checked | Weight |
|-------|---------------|--------|
| Core | H1 override, Meta SEO title, Meta SEO description | 30% |
| Content | Main content (RichText), Main visual (Image), Alt text, Reading time, Updated date | 25% |
| Authority | Author (ref/multi-ref), Category, Tags, Pillar page | 20% |
| Enhancement | Primary keyword, Secondary keywords, Related posts, FAQ | 15% |
| Social | OG title, OG description, OG image | 10% |

Use alias matching — `seo-title`, `meta-title`, `og-title-seo` all count for Meta SEO Title, etc.

For each collection, compute:
- Per-group coverage score (0–100%)
- Overall weighted schema score
- List of missing fields by group

Surface schema scores in the Technical section of the report and in the CMS field completeness check (CD5). Collections scoring below 60% overall are a "Must fix" level finding.

**Template HTML:** Extract rendered HTML for key pages (one per collection type + homepage) to verify schema/tracking output matches config.

Sample first 100 items if a collection is large. Note limitation in report.

### 1.3 PageSpeed Data (Optional)

If PageSpeed MCP is available, run audits on:
- Homepage
- Top 2 pages by GSC impressions

Extract and record:
- Mobile performance score (0–100)
- Desktop performance score (0–100)
- LCP (Largest Contentful Paint)
- TBT (Total Blocking Time)
- CLS (Cumulative Layout Shift)
- Key diagnostics: render-blocking resources, image sizes, unused JS

Surface scores in the Technical section. Mobile score below 50 is a "Must fix" finding.

If PageSpeed MCP is unavailable, note: "Core Web Vitals data not available — connect PageSpeed MCP for performance diagnostics."

### 1.4 Crawl Tool Data (Optional)

Ask the user once:
> "Do you have a Screaming Frog or Sitebulb export? If so, share the path to the issues overview CSV and I'll incorporate it."
> Options: 1. Yes — provide path  2. No — skip

If provided, read the CSV and map these issue types to the existing check framework:

| Crawl Tool Issue | Maps to Check |
|-----------------|---------------|
| H1: Missing | C2 / H1 coverage |
| Meta Description: Missing | TD7 / metadata completeness |
| Page Titles: Below 30 Characters | TD7 / metadata completeness |
| Images: Over 100 KB | Performance |
| Images: Missing Size Attributes | CLS / performance |
| Images: Missing Alt Text | Accessibility / SEO |
| Content: Lorem Ipsum Placeholder | Content quality |
| Directives: Noindex | TD1 / indexation |
| Links: Internal Nofollow Outlinks | Internal linking equity |
| Content: Low Content Pages | CD3 / thin content |
| Security: Missing X-Frame-Options/CSP/HSTS | Security (low SEO impact) |

When crawl data is available, it takes precedence over web-signal estimates. Note in the report: "Structural issues confirmed by [Tool] crawl of [N] pages."

### 1.5 Web Data

Fetch via WebFetch:
- Homepage HTML (for schema, tracking scripts, site-wide signals)
- sitemap.xml (try `/sitemap_index.xml` as fallback)
- robots.txt

### 1.6 Keywords Everywhere (Optional)

If available: top 30 queries by impressions — volume, CPC, competition, trend.

---

## Phase 2: CHECK

Run all quick audit checks (see `/audit` skill) on the fetched homepage HTML, PLUS the deep-mode checks below. Every check produces: **Pass/Fail**, **Evidence**, **Source URL/template**.

**IMPORTANT:** Check IDs (CD1, TD1, etc.) are for internal skill logic only. They must NEVER appear in the client-facing report. The report uses plain language descriptions and data evidence instead.

### Content Checks (Deep)

| # | Check | Rule | Evidence | Requires |
|---|-------|------|----------|----------|
| CD1 | Query-to-page alignment | For each top GSC query, a relevant page exists (query appears in page's title or description) | Unmatched queries = content gaps | GSC |
| CD2 | No cannibalization | No query cluster has 2+ pages ranking within 5 positions of each other | Conflicting pages + positions | GSC |
| CD3 | No thin content | All CMS items have body content ≥ 300 words (or ≥ 50% of fields populated) | Items below threshold | Webflow or crawl tool |
| CD4 | Topic coverage | GSC query clusters all have corresponding content pages | Uncovered clusters | GSC |
| CD5 | CMS field completeness | Per collection: % of items with all SEO fields populated (title, description, keywords) | Completion rate per collection | Webflow |
| CD6 | Editorial → product linking | For each editorial/blog page ranking in GSC, at least one contextual in-content link points to a relevant product/service page | Pages with no product link = missed conversion path | GSC + WebFetch |

### Technical Checks (Deep)

| # | Check | Rule | Evidence | Requires |
|---|-------|------|----------|----------|
| TD1 | Indexation cross-reference | All published pages appear in GSC | Not-indexed list, orphan-indexed list | Webflow |
| TD2 | Metadata completeness | All CMS templates have title/description rules, not falling back to `name` only | Templates using `name` as title | Webflow |
| TD3 | Schema coverage per template | Each content template type has appropriate schema (Article for blog, etc.) | Templates missing schema | Webflow |
| TD4 | Crawl freshness | Top pages crawled within last 30 days | Pages with stale crawl | GSC |
| TD5 | Sitemap vs live pages | Sitemap URL count matches published page count (within 10%) | Mismatch count | Webflow or sitemap fetch |
| TD6 | No duplicate meta titles | Zero duplicate `<title>` values across all pages | Duplicate groups | GSC or crawl tool |
| TD7 | No metadata length issues | All titles 30–60 chars, all descriptions 70–155 chars | Pages outside range | GSC or crawl tool |
| TD8 | Core Web Vitals | Mobile PageSpeed ≥ 50, LCP < 4s, CLS < 0.1 | Score, LCP, TBT, CLS values | PageSpeed MCP |

For checks marked "Webflow" in the Requires column: if Webflow MCP is unavailable, mark as `N/A — GSC-only mode` and note which signals were used as a partial proxy (e.g., sitemap URL count from WebFetch for TD5).

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
| 2 | Level 1 AND CD1 (query-page alignment for top queries) AND quick C4 (FAQ) AND CD5 (≥ 50% CMS field completeness — skip if GSC-only mode) |
| 3 | Level 2 AND CD2 (no major cannibalization) AND CD4 (topic coverage ≥ 80%) AND CD3 (no thin content) AND CD6 (editorial pages have product links) |
| 4 | Level 3 AND pillar pages rank for 50+ queries AND child pages link back AND CD5 (≥ 90% field completeness) |
| 5 | Level 4 AND automated freshness signals AND expanding query footprint (more queries month-over-month) |

### Technical Dimension Gates (Deep)

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Quick T1 (HTTPS) AND T2 (viewport) AND title tags exist on ≥ 80% of pages |
| 2 | Level 1 AND TD1 (all key pages indexed — skip if GSC-only) AND TD6 (no duplicate titles) AND TD5 (sitemap matches) |
| 3 | Level 2 AND TD3 (schema on all content templates — skip if GSC-only) AND TD2 (no templates using `name` as title — skip if GSC-only) AND TD7 (metadata lengths ok) AND TD8 (mobile PageSpeed ≥ 50 — skip if no PageSpeed MCP) |
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

### 4.1 Report Principles

The report is a **professional client deliverable** — the foundation for an engagement. Follow these rules:

1. **No check IDs** — Never write CD1, TD3, AD2, etc. in the report. Use plain language: "content gaps", "indexation coverage", "branded search trend".
2. **Opportunities, not failures** — Frame findings as untapped potential with quantified upside. "You have X impressions going to pages without optimized titles — that's Y potential clicks per month" not "X pages have bad titles".
3. **No implementation details** — Describe **what** needs attention and **why** it matters with data. Never describe **how** to fix it in Webflow/GSC. The how is the engagement.
4. **Quantify everything** — Use GSC data to attach numbers: impressions, clicks, positions, trends. "500 monthly impressions with 0.8% CTR" is more persuasive than "low CTR".
5. **Business language** — Revenue potential, competitive positioning, market visibility. Not technical jargon.
6. **Lead to engagement plan** — The report should make the client confident that you understand their situation and have a plan.

### 4.2 Report Structure

Output a single markdown file with these sections:

---

**Header:**
```
# SEO & AEO Maturity Audit — {domain}

**Prepared for:** {domain}
**Date:** YYYY-MM-DD
**Assessment type:** Deep ({platform} + GSC data)
**Confidence:** Medium–High
**Period analyzed:** Last 90 days
**Data sources:** GSC ([X] days of data, [Y] pages, [Z] queries)[, Webflow ([N] pages, [M] CMS items across [K] collections) — omit if GSC-only mode][, PageSpeed (mobile [score]/100, desktop [score]/100) — omit if unavailable][, [Crawl Tool] crawl ([N] pages) — omit if not provided], Keywords Everywhere ([status])
**CMS platform:** {platform}
```

Adapt the data sources line to only list sources that were actually used.

---

**1. About This Assessment**

Brief methodology section (4-5 sentences):
- What data sources were analyzed and their coverage
- How scoring works (4 dimensions, 5 maturity levels, evidence-based with data backing)
- Confidence level and what it means
- How this relates to any prior quick audit (if `./{domain}/reports/latest-quick.md` exists: "This builds on the discovery assessment from [date], now with full data access.")

---

**2. Executive Summary**

5-7 sentences for a decision-maker:
- Current maturity level and what it means for the business
- Total search footprint (pages indexed, queries ranking, total impressions/clicks)
- The single biggest opportunity with estimated impact (use GSC data)
- What's already strong (give credit — builds trust and shows fairness)
- Key risk if nothing changes (framed as competitive or visibility consequence)
- What reaching the next level would unlock in business terms

---

**3. SEO & AEO Maturity Scorecard**

Overall level (number + name) with a one-line business interpretation.

Table with NO check IDs:

| Dimension | Score | Level | What's Working | Biggest Opportunity |
|-----------|-------|-------|----------------|---------------------|
| Content | X/5 | Name | [data-backed strength] | [quantified opportunity] |
| Technical | X/5 | Name | [data-backed strength] | [quantified opportunity] |
| Authority | X/5 | Name | [data-backed strength] | [quantified opportunity] |
| Measurement | X/5 | Name | [data-backed strength] | [quantified opportunity] |

Below the table:
- Strongest dimension and why it's a competitive advantage
- Weakest dimension and estimated business impact (use real numbers)

---

**4. Key Findings**

For each dimension, write a **data-backed narrative** (not a pass/fail table):

**Content:**
- Lead with strengths (total content pages, queries ranking, strong topics)
- Content gaps: queries with high impressions but no matching page (quantify with impressions)
- Cannibalization: where pages compete against each other (list specific pages + queries)
- Thin content: CMS items below quality thresholds
- CMS field completeness: what's populated vs. what's missing across collections
- End with: what a structured content strategy would unlock

**Technical:**
- Lead with strengths (indexation rate, schema coverage, clean URLs)
- Indexation gaps: published pages not appearing in GSC
- Metadata issues: duplicates, length problems, templates falling back to item name
- Schema gaps: templates missing appropriate structured data
- **CMS schema coverage**: for each content collection, report the schema score (Core/Content/Authority/Enhancement/Social) and overall %. Flag collections below 60% as a "Must fix" — they block `/refresh-content` from populating key fields. Point to `/cms-collection-setup:review` to fix.
- End with: what technical optimization would unlock (more indexed pages = more visibility)

**Authority:**
- Lead with strengths (branded queries, author presence, E-E-A-T signals)
- Branded search trend (growing/stable/declining — with numbers)
- Author attribution coverage across content
- E-E-A-T consistency across the site
- End with: what stronger authority signals would mean for AI citations and rankings

**Measurement:**
- Lead with strengths (GSC data quality, analytics coverage)
- Tracking gaps across templates
- Reporting cadence and whether data-driven decisions are possible
- End with: what a measurement framework would enable

---

**5. Opportunities & Business Impact**

Prioritized table of opportunities. Use ICE scoring internally and show the score — it makes prioritization logic transparent and defensible in client conversations:

| Priority | ICE | Opportunity | Estimated Impact | What You're Leaving on the Table |
|----------|-----|-------------|------------------|----------------------------------|
| Must fix | 9.0 | [opportunity] | [quantified: impressions, clicks, pages affected] | [business consequence] |
| High impact | 6.0 | [opportunity] | [quantified] | [business consequence] |
| Nice to have | 2.5 | [opportunity] | [quantified] | [business consequence] |

ICE = (Impact × Confidence) / Effort. Show one decimal place. Clients appreciate the rigor and it prevents scope arguments.

For each opportunity:
- Quantify with GSC data where possible (impressions at risk, potential click gain from CTR improvement, etc.)
- Frame as business outcomes (visibility, competitive position, AI readiness)
- Do NOT include how to fix — that's the engagement

---

**6. Quick Wins**

3-5 items that can show measurable results within the first 1-2 weeks. These build client confidence.

For each quick win:
- What it is (outcome, not implementation)
- Estimated impact (use data: "X pages affected, Y impressions at stake")
- Why it's quick (low effort, high confidence)

Frame as: "These are the first things we'd address — you'll see changes in search performance within 2-3 weeks."

---

**7. Strategic Roadmap**

Phased engagement plan:

**Phase 1: Foundation (Weeks 1-2)**
- Quick wins from Section 6
- Must-fix items from Section 5
- Expected outcome: [specific, measurable]

**Phase 2: Optimization (Weeks 3-6)**
- High-impact items from Section 5
- Content gap priorities
- Expected outcome: [specific, measurable]

**Phase 3: Growth (Weeks 7-12)**
- Nice-to-have items
- Advanced schema, authority building
- Expected outcome: [specific, measurable]

**Ongoing: Monitoring**
- Weekly performance tracking
- Monthly progress reviews
- Quarterly re-assessment

Include a maturity progression target:
```
Current: Level X → Phase 1 target: Level Y → Phase 3 target: Level Z
```

---

**8. Engagement Recommendation**

Professional proposal bridge:

"Based on this assessment, here's what we recommend:"

**Recommended engagement:**
- Scope: [what's included — reference the phases above]
- Approach: [quick wins first, then strategic, with progress tracking]
- Tracking: Weekly reports to measure impact, monthly reviews, quarterly deep audit re-run
- Expected outcomes: [specific maturity level targets, visibility improvements]

End with a confident closing line and clear next step (e.g., "Let's schedule a call to walk through the roadmap and align on priorities.").

---

**Appendix A: Metadata Audit**

Table (no check IDs, clean data):

| Page / Template | Title | Description | Open Graph | Issues |
|-----------------|-------|-------------|------------|--------|
| [page] | [title or rule] | [description or rule] | [status] | [plain language issue] |

---

**Appendix B: Content Gaps**

| Query Cluster | Monthly Impressions | Current Coverage | Suggested Action |
|---------------|---------------------|------------------|------------------|
| [cluster] | [number] | No matching page / partial match | [create/expand/consolidate] |

---

**Appendix C: Cannibalization**

| Query Cluster | Competing Pages | Positions | Suggested Action |
|---------------|-----------------|-----------|------------------|
| [cluster] | [page1, page2] | [pos1, pos2] | [consolidate/differentiate/redirect] |

---

**Appendix D: Indexation Cross-Reference**

| URL | In Webflow | In GSC | Status | Notes |
|-----|------------|--------|--------|-------|
| [url] | Yes/No | Indexed/Not indexed | [status] | [plain language note] |

---

### 4.3 Save to File

Save these files:
- `./{domain}/reports/audit-deep-YYYY-MM-DD.md` (timestamped)
- `./{domain}/reports/latest-deep.md` (overwrite each run — other skills read this)

Save supporting datasets as JSON under `./{domain}/reports/data/`:
- `content-gaps.json`
- `cannibalization.json`
- `indexation.json`
- `metadata-audit.json`
- **`baseline.json`** — structured snapshot for `/weekly-report` delta comparisons:

```json
{
  "domain": "{domain}",
  "date": "YYYY-MM-DD",
  "platform": "{platform}",
  "maturity": {
    "overall": 0,
    "content": 0,
    "technical": 0,
    "authority": 0,
    "measurement": 0
  },
  "search_footprint": {
    "total_impressions_90d": 0,
    "total_clicks_90d": 0,
    "avg_ctr": 0.0,
    "avg_position": 0.0,
    "pages_with_impressions": 0,
    "queries_with_impressions": 0
  },
  "top_pages": [
    {"url": "", "impressions": 0, "clicks": 0, "ctr": 0.0, "position": 0.0}
  ],
  "pagespeed": {
    "mobile_score": null,
    "desktop_score": null,
    "lcp_mobile": null,
    "tbt_mobile": null,
    "cls_mobile": null
  }
}
```

`/weekly-report` loads `baseline.json` to compute week-over-week deltas against this engagement starting point.

Create directories if needed: `mkdir -p ./{domain}/reports/data/`

### 4.4 Activity Log

Append to `./{domain}/reports/activity-log.md`:

```
| YYYY-MM-DD | /audit:deep | Deep audit ({platform}). Overall Level: X. Content: X, Technical: X, Authority: X, Measurement: X. Must-fix: N. [Mobile PageSpeed: XX/100.] |
```

Log even on early exit.

---

## Integration with Other Skills

| Finding | Skill | When |
|---------|-------|------|
| Low CTR / bad meta tags | `/click-recovery` | Quick metadata fixes |
| Outdated content | `/refresh-content {url}` | Full content refresh |
| Striking distance keywords (positions 4-30) | `/keywords-opportunity:striking` | Prioritized page 1-3 rankings with volume data |
| Content gaps / new keyword topics | `/keywords-opportunity:discover` | KE-powered new keyword discovery |
| Full keyword opportunity map | `/keywords-opportunity` | Striking distance + new discovery |
| CMS schema gaps (missing fields, low schema score) | `/cms-collection-setup:review` | Add missing fields to existing collections |
| No blog collection yet | `/cms-collection-setup:create` | Build a new collection with the full recommended schema |
| Low AEO scores on key pages | `/aeo-optimize {url}` | Add FAQ, schema, direct answers, question H2s per page |
| Missing config | `/getting-started` | First-time setup |
| Quick pre-sale assessment | `/audit {url}` | Before connecting MCP |
| Ongoing monitoring | `/weekly-report` | Track progress weekly |
| Monthly progress | `/monthly-report` | Month-end review |

**Engagement workflow:**
1. `/audit {url}` for pre-sale discovery
2. `/audit:deep` post-sale for engagement baseline
3. Quick wins with `/click-recovery` and `/refresh-content`
4. `/weekly-report` to track and demonstrate progress
5. Re-run `/audit:deep` quarterly to measure maturity progression
