---
name: topic-map
version: "1.2"
description: |
  Convert keyword exports, URL lists, and performance data into a structured SEO keyword map — with per-cluster keyword ownership tables, product page tracking, editorial→product linking status, cluster maps, gap analysis, cannibalization detection, and a 90-day editorial calendar.
  Triggers: seo structure, site architecture, keyword map, keyword ownership map, page keyword map, what keyword does each page target, product page map, pillar pages, topic clusters, content map, seo map, page structure, editorial calendar, content plan, keyword ownership, content gaps.
  Requires: A page list (manual, Webflow MCP, or GSC) + a keyword list. Strongly recommended: Keywords Everywhere for topic discovery and volume enrichment.
  Workflow: Intake → Normalize → Cluster → KE Enrichment & Topic Discovery → Map → Roles → Gaps → Cannibalization → Prioritize → Output.
  Modes: /topic-map (full analysis + editorial calendar), /topic-map:audit (existing structure only, no gap analysis), /topic-map:quick (structure map only, no editorial calendar).
---

# Topic Map

Most sites have pages and keywords — but no clear view of which page owns which topic, where there are gaps, and what to build next. This skill fixes that.

It takes any combination of URLs, keyword exports, and performance data, and outputs a clear keyword map: per-cluster ownership tables showing which page targets which keyword, product page tracking, editorial→product linking status, gap analysis, cannibalization risks, and a prioritized editorial calendar.

**This skill is read-only** — it produces a structured map and editorial plan, but never modifies Webflow content. It hands off to `/keywords-opportunity:new-page`, `/keywords-opportunity:new-cluster`, `/write-blog`, `/refresh-content`, and `/click-recovery` for execution.

---

## Prerequisites

- **Required (at least one source for pages)**: URL list provided by user, OR Webflow MCP to fetch pages, OR GSC MCP to list indexed pages
- **Required (at least one source for keywords)**: Keyword list provided by user, OR GSC MCP for query data, OR Google Ads keyword export
- **Optional**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server) — for performance data, query-to-URL mapping, and impressions
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) — for search volume, CPC, and competition enrichment
- **Optional**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) — for auto-fetching full page inventory and meta titles

## Skill Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/topic-map` | Complete analysis: cluster map, keyword ownership, gaps, priorities, editorial calendar |
| **Audit** | `/topic-map:audit` | Existing structure only — no gap analysis, no editorial calendar. Good for a quick health check. |
| **Quick** | `/topic-map:quick` | Structure map and top priorities, no editorial calendar. |

## Workflow Overview

```
INTAKE → NORMALIZE → CLUSTER → MAP → ROLES → GAPS → CANNIBALIZATION → PRIORITIZE → OUTPUT
```

---

## Critical Guards

⚡ GUARD — **No page list and no keyword list:**
If the user provides neither a page list nor a keyword list, and neither Webflow MCP nor GSC MCP are connected:
- Stop: "Topic Map needs at least a page list and a keyword list to work. Paste your URLs and keywords, or connect Webflow MCP or GSC MCP."

⚡ GUARD — **Only pages, no keywords:**
If pages are available but no keywords can be sourced:
- Warn: "No keyword list found. Without keywords, the mapper can only assign page roles — clustering and gap analysis won't run. Continue with a role-only audit, or paste your keyword list?"
- If user continues: Run Phase 2 and Phase 4 only (normalize + roles), skip clustering, mapping, and gaps

⚡ GUARD — **Only keywords, no pages:**
If keywords are available but no pages can be sourced:
- Warn: "No page list found. Without pages, the mapper can only cluster keywords and identify gaps — it can't map keywords to existing content. Continue anyway?"
- If user continues: Run Phases 1, 2, 3 only (intake, normalize, cluster). Output a keyword cluster map and gap list only.

⚡ GUARD — **User requests abort:**
If user says "stop", "cancel", "abort", or "nevermind" at any phase:
- Confirm: "Stop the workflow? Progress will be lost."
- If confirmed: exit cleanly with any partial results already computed

---

## Phase 0: INTAKE

Collect all inputs, check tool availability, load config.

### 0.1 MCP Discovery

Search for available tools:
- **Webflow MCP** (optional): Search `+webflow data cms`. If available → use for auto-fetching pages.
- **GSC MCP** (optional): Search `+gsc search analytics`. If available → use for query data and performance.
- **Keywords Everywhere** (strongly recommended): Search `+keywords everywhere volume`.

**Keywords Everywhere is critical for this skill.** It powers two things that can't be done without it:
1. Real search volume on every keyword — without it, impressions are the only signal
2. New topic discovery — related keywords, PASF (People Also Search For), and long-tail expansions the site isn't targeting yet

If Keywords Everywhere is **not connected**, ask the user before proceeding:

```
⚠️ Keywords Everywhere is not connected.

This skill uses it to suggest new keywords and topics you're not targeting yet —
not just map what you already have. Without it, topic discovery runs on
reasoning only (no volume data).

Options:
1. Connect Keywords Everywhere now → https://github.com/hithereiamaliff/mcp-keywords-everywhere
2. Continue without it — I'll suggest new topics based on GSC data and reasoning,
   but won't have search volume to validate them

Which would you prefer?
```

If the user chooses to continue without KE: proceed, mark every topic suggestion as "volume unvalidated", and include a reminder in the report header.
If the user connects KE: re-check availability before continuing.

Note all tool availability in the report header.

### 0.2 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract:
- `business.name` → used to identify branded queries and homepage focus
- `seo.competitors` → competitor pages may reveal structural gaps
- `audience.primary` → used to validate intent fit of proposed pages
- `seo.metaTitleFormat` → used in editorial calendar page proposals

If not found: proceed with defaults. Note "Run `/getting-started` for personalized recommendations."

### 0.3 Set {domain}

If GSC MCP is available: list properties and ask user to select. Extract domain (e.g., `https://www.checklist-seo.com/` → `checklist-seo.com`).
If GSC not available: ask user for their domain (used only for the report save path and activity log).

Check `.claude/reports/{domain}/activity-log.md` for recent context:
- If `/topic-map` was run in the last 30 days → warn: "A structure map was already generated on [date]. Run again to update it?"
- Read last 5 entries for context (recent skills run, recent publishes)

### 0.4 Collect Inputs

Gather all available inputs. For each input type, try automatic sourcing first, then fall back to asking the user.

**Page list (try in order):**
1. If Webflow MCP available: fetch all pages via `list_pages` + `list_collection_items` per collection. Extract: URL/slug, page title, meta title, meta description, collection name.
2. If GSC MCP available: fetch all indexed pages with any impressions. Extract: URL, total impressions, total clicks, average position.
3. If neither: ask user to paste a list of URLs (one per line, or a sitemap URL).

**Keyword list (try in order):**
1. If GSC MCP available: fetch all queries with impressions ≥ 5 over the last 90 days. Extract: query, impressions, clicks, CTR, position, top page.
2. If user has a keyword export: ask them to paste it (CSV or plain list — handle both).
3. If user has a Google Ads keyword export: accept it, extract keyword + avg monthly searches.
4. If none of the above: ask user for a keyword list directly.

**Optional enrichment inputs:**
- Page performance data (if not auto-fetched from GSC)
- Business priorities (e.g., "focus on converting visitors", "build authority in X topic")
- Target language and market (default: English, global)
- Competitor URLs (for cross-referencing missing topics)

### 0.5 Confirm Inputs

Before processing, confirm what was collected:

```
## Inputs collected

**Pages**: [N] pages from [Webflow MCP / GSC / manual paste]
**Keywords**: [N] keywords from [GSC queries / manual list / Google Ads export]
**Performance data**: [available from GSC / not available]
**Volume data**: [available from Keywords Everywhere / not available]
**Config**: [loaded / not found — using defaults]

Proceed with analysis?
```

If the user says no or wants to add inputs: pause and collect additional inputs before continuing.

---

## Phase 1: NORMALIZE

Clean, deduplicate, and standardize all inputs.

### 1.1 Normalize Pages

For each page:
- Normalize URL: lowercase, strip scheme + www, strip trailing slash
- Deduplicate (same slug from CMS + static list → keep one, prefer the richer record)
- Detect language variants: flag `/fr/`, `/de/`, `/es/` prefixed pages as language variants, keep one per canonical URL
- Extract or infer page title: use meta title if available, otherwise use URL slug (convert hyphens to spaces, title-case)
- Classify page type: homepage (`/`), static page, CMS collection item (by collection name)

Normalized page record:
```
{
  url: "/blog/webflow-seo-checklist",
  title: "The Webflow SEO Checklist",
  type: "cms-item",
  collection: "blog",
  impressions: 1200,
  clicks: 34,
  position: 8.2
}
```

### 1.2 Normalize Keywords

For each keyword:
- Lowercase and strip leading/trailing whitespace
- Deduplicate exact matches
- Deduplicate near-matches: if two keywords differ only by a stop word ("seo checklist" vs "the seo checklist"), keep the version with higher impressions/volume
- Strip special characters that indicate format (e.g., trailing `?`, leading `-`)
- Flag question-format keywords (`how to`, `what is`, `why does`) — these signal informational intent

Normalized keyword record:
```
{
  keyword: "webflow seo checklist",
  impressions: 850,
  clicks: 12,
  position: 9.1,
  volume: 720,
  cpc: 1.20,
  intent: "informational",
  top_page: "/blog/webflow-seo-checklist"
}
```

### 1.3 Flag Input Quality Issues

Note any issues found during normalization:
- Duplicate pages (same URL from multiple sources)
- Multilingual pages detected
- Keywords with no matching page
- Pages with no matching keyword
- Very short keyword lists (< 20 keywords) — warn: "Small keyword set. Results may be incomplete. Consider adding more keywords or connecting GSC/Keywords Everywhere."
- Very large keyword list (> 2000 keywords) — note: "Large keyword set detected. Processing top 500 by impressions/volume for clustering. Full list available on request."

---

## Phase 2: CLUSTER

Group keywords into coherent topic clusters.

### 2.1 Classify Intent

For each keyword, assign one primary intent:

| Intent | Signals |
|--------|---------|
| **Informational** | how, what, why, guide, tutorial, tips, examples, learn, explained, checklist, best practices |
| **Commercial** | best, top, compare, vs, review, alternative, tool, software, pricing |
| **Transactional** | buy, hire, pricing, sign up, free trial, get started, demo, download |
| **Navigational** | brand names, specific product/tool names, login, dashboard |

If intent is ambiguous: assign the dominant intent based on the majority of similar keywords in the same semantic group.

### 2.2 Group by Topic

Cluster keywords into semantic topic groups using these signals:
- Shared root noun or core topic (e.g., "webflow seo", "seo checklist", "seo audit" → topic: SEO)
- Common modifier family (e.g., "webflow cms tutorial", "webflow cms collections", "webflow cms pricing" → topic: Webflow CMS)
- Same search intent + same target audience
- Co-occurrence on the same GSC top page (if available)

**Naming clusters:** Name each cluster after its core topic noun phrase (e.g., "Webflow SEO", "CMS Setup", "Page Speed"). Avoid generic names like "informational content" or "misc".

**Cluster hierarchy:**
- **Pillar cluster**: 1 dominant topic that everything else connects to. Large keyword volume, broad intent.
- **Support cluster**: Specific sub-topics within a pillar. Narrower intent, more specific queries.
- **Standalone cluster**: Topic that doesn't naturally belong under a pillar (e.g., a pricing page cluster).

### 2.3 Identify the Homepage Cluster

The homepage targets the site's primary value proposition query — usually the broadest, most competitive keyword the business wants to own. It is never the target of a specific how-to or tutorial.

Identify it by:
- Highest-volume informational or commercial keyword with no specific qualifier
- Usually maps to the brand's primary product/service category

If no clear homepage keyword is identifiable from the keyword list, flag this as a gap: "No clear homepage target keyword found. Consider: what query should someone type to land on your homepage?"

---

## Phase 2.5: KE ENRICHMENT & TOPIC DISCOVERY

Enrich existing keywords with real volume data, then discover new keywords and topics the site isn't targeting yet. This phase runs after clustering so KE queries are focused on confirmed topic areas.

**Run this phase regardless of how small the initial keyword set is.** Even if the user only has 10 GSC keywords, KE can surface 50+ relevant keywords and topics they're missing.

### 2.5.1 Enrich Existing Keywords

If Keywords Everywhere is available, for each keyword in the normalized set:
- Fetch: monthly search volume, CPC, competition score, trend (rising/flat/declining)
- Update the normalized keyword record with these values
- Flag high-value keywords: volume > 500/month OR CPC > $2 (strong commercial signal)
- Flag declining keywords: trend down > 30% YoY — deprioritize in planning

If KE is not available: note "Volume unvalidated" on all keywords. Use GSC impressions as the only demand signal.

### 2.5.2 Expand Existing Clusters with Related Keywords

For each cluster identified in Phase 2, use KE to discover additional keywords the site could target:

1. Take the top 2–3 keywords per cluster
2. Run `get_related_keywords` on each
3. Run `get_pasf_keywords` (People Also Search For) on each
4. Filter results: keep keywords with volume > 100/month (or top 10 by volume if data is limited)
5. Match against existing pages — any keyword not already covered = potential gap or expansion

Add discovered keywords to the relevant cluster with tag `[KE discovered]`. These feed directly into Phase 6 (Gap Analysis).

### 2.5.3 Discover New Topic Clusters

Beyond expanding existing clusters, actively look for topics the site should be targeting but isn't:

1. For each cluster pillar keyword, run `get_related_keywords` with broad scope
2. Look for keyword groups that form a coherent new cluster (3+ related keywords with combined volume > 300/month)
3. Cross-check against current page inventory — if no page covers this topic → it's a new topic opportunity

For each new topic opportunity found:
```
New topic: "{topic name}"
Seed keyword: "{keyword}" ({volume}/month)
Related keywords: "{kw1}" ({vol}), "{kw2}" ({vol}), "{kw3}" ({vol})
Total cluster volume: ~{sum}/month
Recommended page: {proposed URL and title}
Audience fit: {why this topic fits the site's audience}
```

### 2.5.4 Compile KE Enrichment Summary

Before moving to Phase 3, output a brief summary:

```
## Keywords Everywhere Enrichment Summary

**Existing keywords enriched**: [N] keywords updated with volume data
**High-value keywords identified**: [N] (volume > 500/month or CPC > $2)
**Declining keywords flagged**: [N] (deprioritized)
**New keywords discovered**: [N] added to clusters via related/PASF expansion
**New topic opportunities**: [N] clusters the site isn't targeting yet

[List new topic opportunities with top keyword + volume]
```

If KE is not available, output instead:
```
⚠️ Keywords Everywhere not connected — topic discovery ran on reasoning only.
New topic suggestions below are based on site context and GSC signals, not validated volume.
Connect KE and re-run for volume-backed topic discovery.
```

---

## Phase 3: MAP

Assign keywords to existing pages.

### 3.1 Match Keywords to Pages

For each keyword, attempt to match it to an existing page:

**Match sources (in order of reliability):**
1. GSC `top_page` link: if the keyword has a `top_page` from GSC data, that is the confirmed match
2. Title match: keyword appears in the page title (exact or partial)
3. URL slug match: keyword words appear in the URL slug
4. Meta description match: keyword appears in the meta description

**Match confidence:**
- **Confirmed**: GSC top_page match
- **Strong**: keyword in title + URL
- **Weak**: keyword in title or URL only
- **No match**: keyword has no corresponding existing page

Store: `keyword → page, confidence, match_source`

### 3.2 Detect Multi-Match Conflicts

If a keyword matches 2+ existing pages:
- Flag as a **cannibalization risk** (handled in Phase 5)
- For the map, assign to the page with the best performance (highest clicks, or highest position)
- Note the conflict for the cannibalization report

### 3.3 Build the Keyword-to-Page Ownership Table

For each page: list all keywords matched to it, their intent, volume, and impressions.
For each keyword: record its assigned page (or "missing — no page" if no match).

---

## Phase 4: ROLES

Assign a structural role to each existing page.

### 4.1 Role Definitions

| Role | Definition | Typical URL pattern |
|------|------------|---------------------|
| **Homepage** | Root URL — targets the broadest brand/category keyword | `/` |
| **Pillar page** | Covers a major topic comprehensively — links to multiple support pages | `/topic` or `/guide/topic` |
| **Support page** | Deep-dives into one specific sub-topic within a pillar — links back up | `/blog/specific-topic` or `/topic/subtopic` |
| **Product page** | Transactional page for a specific service, trip, or offering. The destination that editorial content should link to. Primary metric is conversion, not ranking. | `/trips/gorilla-day-trip`, `/services/seo-audit` |
| **Landing page** | Generic transactional focus — not tied to a specific product (pricing, contact, free trial) | `/pricing`, `/contact`, `/free-trial` |
| **Index page** | Lists/aggregates content (e.g., blog index, category page) — not a ranking target itself | `/blog`, `/resources` |
| **Standalone** | Topic doesn't map under a pillar — independent ranking target | `/about`, one-off guides |

**Product vs. Landing page:** Use "Product page" when the page represents a specific bookable, purchasable, or requestable offering that editorial content should link to. Use "Landing page" for generic conversion pages (pricing, contact, sign-up) that are not cluster-specific.

### 4.2 Assign Roles

Assign roles based on:
- **URL depth**: root = homepage/pillar; deep path = support
- **Keyword volume**: high-volume, broad keywords → pillar or homepage; specific, long-tail → support
- **Cluster assignment**: the page that covers the most keywords in a cluster = pillar for that cluster
- **GSC performance**: high impressions + moderate CTR → likely already a pillar; low impressions + long tail → support
- **Page type**: CMS collection items are usually support pages unless they're collection indexes

Flag role conflicts:
- A CMS item matched to many keywords across multiple clusters → may be over-broad, needs splitting
- A deep URL matched to a pillar-level keyword → may be misplaced in the hierarchy

### 4.3 Output Role Assignment

For each page, output:
```
/blog/webflow-seo-checklist
  Role: Pillar page
  Cluster: Webflow SEO
  Keywords owned: 12 (740 avg monthly impressions)
  Top keyword: "webflow seo checklist" (pos 8.2, 850 imp)
  Performance: 34 clicks / 1200 imp / 2.8% CTR
```

---

## Phase 5: CANNIBALIZATION DETECTION

Find pages competing for the same keywords.

### 5.1 Identify Cannibalizing Pairs

A cannibalization risk exists when:
- 2+ pages rank for the same keyword in GSC (confirmed by `top_page` data showing the same keyword appearing across multiple pages)
- 2+ pages are mapped to the same keyword in Phase 3 with Strong or Confirmed confidence
- 2+ pages belong to the same cluster AND are assigned the same role (e.g., two pillar pages for the same topic)

### 5.2 Severity Classification

| Severity | Condition | Recommended action |
|----------|----------|--------------------|
| **High** | Same keyword confirmed on 2+ pages in GSC with similar positions | Consolidate pages, set canonical, or differentiate clearly |
| **Medium** | Same keyword matched to 2+ pages with Strong confidence but not confirmed in GSC | Review pages, check if one should be retargeted |
| **Low** | Similar topic but different intent (e.g., one informational, one transactional) | Monitor — may be fine if intent is clearly differentiated |

### 5.3 Consolidation Recommendations

For each high-severity pair:
- Identify which page should "win" (better performance, better alignment with keyword intent)
- Recommend: consolidate content into the winning page, redirect the other, or differentiate intent explicitly
- Flag for `/refresh-content` on the winning page

---

## Phase 6: GAPS

Identify missing pages.

### 6.1 Unmatched Keyword Clusters

Any keyword cluster with no matching existing page = a **content gap**.

For each gap cluster:
- Name the missing page (proposed title based on the cluster's top keyword)
- Propose a URL slug
- List the keywords it would target
- Estimate total keyword opportunity (sum of monthly volumes, or sum of impressions if no volume data)

### 6.2 Topic Gaps vs. Coverage Gaps

**Topic gap**: An entire topic cluster has no existing page at all → research keywords first with `/keywords-opportunity:new-page` (single page) or `/keywords-opportunity:new-cluster` (full cluster), then create with `/write-blog`.

**Coverage gap**: A page exists for the topic, but it only covers part of the keyword cluster → the existing page needs expansion (via `/refresh-content`), or a support page needs to be created for the uncovered sub-topic.

### 6.3 Structural Gaps

Beyond individual keywords, identify structural gaps:
- **Missing pillar page**: A cluster has multiple support pages but no dedicated pillar tying them together
- **Missing index page**: A CMS collection has no collection index page linking to its items
- **Orphan pages**: Pages with no keywords mapped to them and no internal link context — these may not be indexable or rankable
- **Homepage keyword gap**: The homepage targets no clear primary keyword or the primary keyword isn't in its title/meta

---

## Phase 7: PRIORITIZE

Use the priority buckets from CLAUDE.md. No numeric formula — assign buckets based on observable criteria.

### 7.1 Priority Buckets

**Must do** — at least one structural problem confirmed in the data:
- GSC confirms same keyword ranking on 2+ pages simultaneously (cannibalization)
- A page in the sitemap is returning a 404
- An indexation error on a page with existing traffic

**High value** — clear opportunity with data backing:
- A keyword cluster in the top 20% of site impressions with no matching page
- The primary keyword for a page doesn't appear in its title
- A cluster has 2+ support posts but no pillar page
- A page that exists in the architecture is missing from GSC entirely

**Nice to have** — everything else:
- Coverage gaps on existing pages (missing sub-topics)
- Metadata length issues with no confirmed traffic impact
- Orphan pages with no internal links

### 7.3 Recommendation Types Scored

- Cannibalization fixes (existing conflict)
- Structural gap fixes (missing pillar, orphan pages)
- New page creation (topic gaps)
- Existing page expansions (coverage gaps)
- Keyword retargeting (page targets wrong keyword for its cluster)
- Metadata fixes (keyword not in title — quick wins)

### 7.4 Recommendation Format

Each scored recommendation:
1. **Action title** — specific and directive (e.g., "Create pillar page targeting 'Webflow SEO'")
2. **Why it matters** — keyword volume, traffic at stake, or structural problem
3. **Expected impact** — clicks/month estimate if data available, or qualitative (High / Medium / Low)
4. **Effort** — Low / Medium / High
5. **Skill shortcut** — e.g., "Use `/write-blog` to create this page", "Use `/refresh-content` to expand"
6. **Dependencies** — e.g., "Create this pillar before creating support pages under it"

---

## Phase 8: OUTPUT

Build and save the full structure map, plus editorial calendar.

### 8.1 Keyword Map — Per-Cluster Tables

**This is the first and mandatory output.** Render before anything else. Output one table per cluster, with a summary table at the end. Every page gets exactly one row somewhere. No exceptions.

#### Per-cluster table format

For each cluster, render an H3 header and table, followed by two summary lines:

```
### Cluster: {Cluster Name}

| URL | Page Type | Primary Keyword | Volume | GSC Imp | Status |
|-----|-----------|-----------------|--------|---------|--------|
| /rwanda | Hub | rwanda tours | 1,200/mo | 433 | ✅ Exists |
| /gorilla-trekking | Editorial | gorilla trekking rwanda | 3,600/mo | 1,200 | ✅ Exists |
| /volcanoes-np | Editorial | volcanoes national park rwanda | 2,400/mo | — | ❌ Missing |
| /trips/gorilla-day-trip | Product | gorilla day trip | 480/mo | 620 | ✅ Exists |
| /trips/kigali-city-tour | Product | kigali city tour | 480/mo | 890 | ✅ Exists |

**Cluster health**: 3/5 pages exist | 2 gaps | 0 cannibalization risks
**Linking**: Editorial → Product: ⚠️ 1/2 (gorilla-trekking links to gorilla trip, not to kigali tour)
```

**Sort order within each table:**
1. Hub page (if exists)
2. Pillar page (if exists and distinct from hub)
3. Editorial/Support pages — existing rows first, then ❌ Missing rows
4. Product pages — existing rows first, then ❌ Missing rows

**Status values:**
- `✅ Exists` — page is live
- `❌ Missing` — topic has demand but no page exists
- `⏳ Planned` — page is in sitemap or planned but not yet live
- `⚠️ Orphan` — page exists but no keywords map to it
- `🚫 Cannibalizes {url}` — keyword is split across 2+ pages

**Column rules:**
- `Volume` = KE monthly volume if available — omit column if KE not connected
- `GSC Imp` = 90-day GSC impressions — omit column if GSC not connected
- If neither KE nor GSC is available: replace both columns with a single `Demand` column with qualitative values (High / Medium / Low / Unknown)

**Cluster health line:** `{N}/{total} pages exist | {N} gaps | {N} cannibalization risks`

**Linking line (check editorial → product links):**
- `✅ All editorial pages link to a product page in this cluster`
- `⚠️ {N}/{M} editorial pages link to a product page — missing: {page list}`
- `ℹ️ No product pages in this cluster — no linking check required`

Linking status is determined by checking whether editorial/support pages in the cluster contain a contextual in-content link (not just navigation) to at least one product page in the same cluster.

---

#### After all cluster tables: Full Site Summary Table

Render a compact single table covering all existing pages (no gap rows) for a quick full-site scan:

```
## Full Site Summary — {domain}

| URL | Type | Primary Keyword | Cluster | Status |
|-----|------|-----------------|---------|--------|
| / | Hub | brand keyword | All | ✅ |
| /rwanda | Hub | rwanda tours | Rwanda | ✅ |
| /gorilla-trekking | Editorial | gorilla trekking rwanda | Rwanda | ✅ |
| /trips/gorilla-day-trip | Product | gorilla day trip | Rwanda | ✅ |
| /pricing | Landing page | — | — | ✅ |
```

No volume/impressions columns — just keyword ownership at a glance. Sort: Homepage → Hubs → per-cluster pages → standalone pages → landing/utility pages.

### 8.2 Cluster Map (Visual Hierarchy)

Render the site's topic architecture as an indented tree. This comes after the Page Inventory Table and shows relationships:

```
## SEO Cluster Map — {domain}

### Homepage
  / — "{Homepage title}"
    Primary keyword: "{keyword}" ({volume}/month or {impressions} impressions)
    Role: Homepage | Status: Existing

### Cluster: Webflow SEO [PILLAR]
  /blog/webflow-seo-checklist — "The Webflow SEO Checklist"
    Role: Pillar page | Status: Existing | Keywords: 12
    Primary keyword: "webflow seo checklist" (850 imp, pos 8.2)
  /blog/webflow-seo-settings — "Webflow SEO Settings Explained"
    Role: Support page | Status: Existing | Keywords: 4
  [MISSING] /blog/webflow-cms-dynamic-lists — "Webflow CMS Dynamic Lists"
    Role: Support page | Status: GAP
    Target keywords: "webflow dynamic list" (320/mo), "cms dynamic content webflow" (180/mo)

### New Topic Opportunities [from KE discovery]
  [NOT TARGETED] Topic: "Webflow Multilingual SEO"
    Proposed page: /blog/webflow-multilingual-seo
    Top keyword: "webflow multilingual" (880/mo)
    Related: "webflow localization" (590/mo), "webflow multiple languages" (320/mo)
    Total opportunity: ~1,790/mo
```

### 8.3 Keyword Ownership Table

Full table mapping every keyword to its assigned page:

```
## Keyword Ownership Table

| Keyword | Intent | Volume/Imp | Assigned Page | Role | Status | Confidence |
|---------|--------|------------|---------------|------|--------|------------|
| webflow seo checklist | Informational | 850 imp | /blog/webflow-seo-checklist | Pillar | Existing | Confirmed |
| webflow seo guide | Informational | 420 imp | /blog/webflow-seo-checklist | Pillar | Existing | Strong |
| webflow cms tutorial | Informational | 640 imp | /blog/webflow-cms-tutorial | Pillar | Existing | Confirmed |
| webflow dynamic list | Informational | 320 imp | — | Support | GAP | — |
```

### 8.3 Cannibalization Report

```
## Cannibalization Risks

### High severity
- "webflow seo" → /blog/webflow-seo-checklist (pos 8) AND /blog/webflow-seo-guide (pos 11)
  → Consolidate into one page. Recommended winner: /blog/webflow-seo-checklist (better CTR)

### Medium severity
- "cms tutorial" → /blog/webflow-cms-tutorial AND /blog/webflow-cms-guide
  → Review intent differentiation. If similar, merge.
```

### 8.4 Gap Analysis

```
## Content Gaps

### Topic Gaps (new pages to create)
| Proposed Page | Cluster | Keywords | Volume/Imp | Priority |
|---------------|---------|----------|------------|----------|
| "Webflow CMS Dynamic Lists" | CMS Setup | 3 keywords | 500 imp | High |
| "Webflow Multilingual SEO" | Webflow SEO | 2 keywords | 280 imp | Medium |

### Coverage Gaps (existing pages to expand)
| Page | Missing Sub-topics | Keywords | Skill |
|------|-------------------|----------|-------|
| /blog/webflow-seo-checklist | Missing "image alt text" and "sitemap settings" sections | 180 imp | `/refresh-content` |

### Structural Gaps
- No pillar page for cluster "Analytics & Tracking" — 3 support pages exist but nothing tying them together
- /blog/old-post has no keywords mapped and no internal links — potential orphan
```

### 8.5 Priority Action Plan

```
## Priority Action Plan

### Must Do
1. Create pillar page: "Webflow SEO Guide" targeting "webflow seo" cluster
   Why: 8 keywords, 1,200 total impressions, no owning page
   Effort: High | Use: `/write-blog`
   Impact: Est. +80–120 clicks/month if page reaches position 5–10

2. Resolve cannibalization: merge /blog/webflow-seo-guide into /blog/webflow-seo-checklist
   Why: Two pages splitting authority on "webflow seo checklist" (position 8 vs 11)
   Effort: Medium | Use: `/refresh-content` on winner, then redirect loser
   Impact: Consolidates authority → potential +1–3 position improvement

### High Value
3. Add keyword to homepage meta title
   Why: Homepage doesn't include "{primary keyword}" in title — confirmed by GSC data
   Effort: Low | Use: `/click-recovery`
   Impact: CTR improvement on highest-impression page

[...]

### Nice to Have
[Condensed table: action, cluster, keywords, effort, skill]
```

### 8.6 Editorial Calendar (Full mode only)

A 90-day schedule for executing the must-do and high-value recommendations:

```
## 90-Day Editorial Calendar

> Based on: [N] must-do + [N] high-value recommendations
> Assumes: ~1 new page or major content update per week

### Week 1–2: Quick wins (low effort, high value)
- Fix homepage meta title → use `/click-recovery`
- Resolve [page] vs [page] cannibalization → use `/refresh-content`

### Week 3–4: Pillar page creation
- Research keywords: `/keywords-opportunity:new-page` → topic: "webflow seo guide"
- Create: "Webflow SEO Guide" (pillar for Webflow SEO cluster) → use `/write-blog` with brief from above
  Target keyword: "webflow seo" | Proposed slug: /blog/webflow-seo-guide
  Keywords to cover: [list top 5]

### Week 5–6: Support page creation
- Research keywords for new cluster: `/keywords-opportunity:new-cluster` → topic: "Webflow CMS"
- Create: "Webflow CMS Dynamic Lists" (support under CMS Setup pillar) → use `/write-blog`
- Expand: /blog/webflow-seo-checklist (add image alt + sitemap sections) → use `/refresh-content`

### Week 7–8: Structural fixes
- Create collection index for [collection] → Webflow Designer
- Add internal links from pillar to support pages → Webflow Designer + `/refresh-content`

### Week 9–12: Nice-to-have and monitoring
- Create: [additional gap pages — lower priority]
- Monitor: Re-run `/weekly-report` to track whether new pages are indexing and ranking

> After executing: run `/weekly-report` to measure traffic impact, `/click-recovery` to optimize CTRs on new pages.
```

### 8.7 Report Header

Every report includes a header:

```
# SEO Keyword Map — {domain}
**Generated**: {date}
**Pages analyzed**: {N} | **Keywords analyzed**: {N}
**Clusters**: {N} | {N} with gaps | {N} with linking issues
**Content gaps**: {N} | **Cannibalization risks**: {N}

**Data sources**:
- Pages: [Webflow MCP / GSC / manual paste]
- Keywords: [GSC queries / manual list / Google Ads export]
- Volume enrichment: [Keywords Everywhere ✅ / not connected ⚠️]
- Performance data: [GSC ✅ / not available ⚠️]
- Config: [Loaded ✅ / not found ℹ️]
```

### 8.8 Save Report

Save to `.claude/reports/{domain}/topic-map-YYYY-MM-DD.md`.

Also save a stable pointer file:
`.claude/reports/{domain}/latest-topic-map.md` — overwrite with each run (always points to the latest map).

Other skills (e.g., `/weekly-report`, `/write-blog`) should check for `latest-topic-map.md` to avoid recommending pages or clusters that are already mapped or planned.

If directory creation fails: output in terminal only, warn user.

---

## Adaptive Behavior

### Small sites (< 20 pages, < 50 keywords)
- Skip cannibalization section (not enough pages to conflict)
- Simplify cluster map to a flat list with roles
- Editorial calendar: 30 days instead of 90

### Large sites (> 200 pages, > 1,000 keywords)
- Cap keyword processing at top 500 by impressions/volume
- Cluster at topic level only — skip sub-cluster detail
- Output summary map only (not full keyword ownership table) — offer full table as a separate file

### No performance data (no GSC, no volume)
- Proceed with structure-only analysis
- Scores are based on keyword count per cluster and reasoning about intent — not impressions or volume
- Note in every recommendation: "No performance data. Scores are based on structural reasoning only."

### Non-English sites
- Detect language from URL patterns (`/fr/`, `/de/`) or keyword language
- Do not mix languages in a single cluster
- Flag multilingual structure: "Detected [N] language variants. Mapping each language independently."

---

## Integration with Other Skills

Topic Map is **planning-only**. Other skills execute.

| Recommendation type | Skill | Why |
|---------------------|-------|-----|
| New page creation (keyword already known) | `/write-blog` | Full SEO + AEO-optimized draft |
| New page (keyword not yet researched) | `/keywords-opportunity:new-page` | Research keywords first → brief feeds `/write-blog` |
| New product or service page | `/keywords-opportunity:new-product` | Commercial keyword brief → then `/write-blog` |
| New topic cluster (all pages missing) | `/keywords-opportunity:new-cluster` | Plan the full cluster keyword map → then re-run `/topic-map` to verify |
| Existing page expansion | `/refresh-content` | Content refresh targeting new keywords |
| Meta title / CTR issues | `/click-recovery` | Fast metadata fixes |
| Cannibalization fix (winner page) | `/refresh-content` | Strengthen the winning page |
| Keyword volume enrichment | `/keywords-opportunity` | Full striking distance + new topic analysis |
| Missing CMS schema fields | `/cms-collection-setup` | Add missing SEO fields to collections |
| Weekly tracking after execution | `/weekly-report` | Measure impact of new/refreshed pages |

---

## Error Handling

| Error | Action |
|-------|--------|
| No pages and no keywords | Stop — see Critical Guard |
| Pages only, no keywords | Role audit only — see Critical Guard |
| GSC MCP not connected | Use manual inputs, note in report |
| Webflow MCP not connected | Use manual URL list or GSC pages |
| Keywords Everywhere not connected | Proceed without volume data, note in report |
| Very large keyword list (> 2000) | Cap at 500 by impressions/volume, note |
| Multilingual pages detected | Separate by language, map each independently |
| No cluster identifiable from keywords | Ask user for a seed topic list to anchor clustering |
| Report save fails | Output in terminal, warn user to save manually |

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

```
| YYYY-MM-DD | /topic-map | [one-line summary: e.g., "Mapped 47 pages, 312 keywords. 5 clusters, 8 gaps, 2 cannibalization risks. Calendar: 12 weeks. Report saved."] |
```

Log even if the skill exits early — note reason (e.g., "Aborted: no keyword list provided").
