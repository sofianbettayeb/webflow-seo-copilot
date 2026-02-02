---
name: click-recovery
description: |
  Find pages Google trusts but users ignore — high impressions, low CTR. Get a prioritized list of titles and meta descriptions to rewrite, then publish directly to Webflow.
  Triggers: click recovery, improve CTR, low CTR, fix CTR, wasted impressions, title optimization, meta description optimization, SERP optimization.
  Requires: Google Search Console MCP server, Webflow MCP server. Optional: Keywords Everywhere API for volume/intent validation.
  Workflow: Analyze → Recommend → Approve → Publish to Webflow.
---

# Click Recovery Skill

Find pages where Google already ranks you but users scroll past. These are your fastest wins — no need to build authority, just fix the pitch.

## Prerequisites

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server)
- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) (for publishing)
- **Optional**: [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (needs API key — for search volume and intent data)

## Prerequisites Check

**Search for these tools BEFORE starting analysis:**

**GSC MCP** (required):
- Search: `+gsc search analytics`
- If missing: Stop and inform user: "Click Recovery requires GSC MCP. Install from: https://github.com/sofianbettayeb/gsc-mcp-server"

**Webflow MCP** (required for publishing):
- Search: `+webflow data cms`
- If missing: Run analysis only, skip publishing. Note: "Webflow MCP not connected — recommendations only, no publishing."

**Keywords Everywhere** (optional but check):
- Search: `+keywords everywhere volume`
- If missing: Continue, but ADD to report header:
  ```
  ⚠️ Keywords Everywhere not connected.
  Recommendations based on GSC impressions only.
  Connect KE for search volume and intent data:
  https://github.com/hithereiamaliff/mcp-keywords-everywhere
  ```

## Workflow Overview

```
FETCH → FILTER → VALIDATE → PRIORITIZE → RECOMMEND → APPROVE → PUBLISH
```

After presenting recommendations, ask user which pages to update. On approval, publish directly to Webflow CMS.

---

## Phase 1: Fetch GSC Data

### 1.1 Identify Target Site

Ask the user which property to analyze if multiple GSC properties are available.

### 1.2 Pull Performance Data

Use GSC MCP to fetch the last 90 days of data:

**Query-level data:**
- All queries with impressions > 100
- Include: query, page, impressions, clicks, CTR, position

**Page-level data:**
- All pages with impressions > 500
- Include: page, impressions, clicks, CTR, average position

### 1.3 Calculate Benchmarks

Compute site-wide averages:
- Average CTR by position bracket (1-3, 4-10, 11-20)
- Overall average CTR
- Median impressions per page

These benchmarks help identify underperformers relative to the site's own performance.

---

## Phase 2: Filter Opportunities

### 2.1 CTR Opportunity Filter

Flag pages/queries meeting these criteria:

**High-impression, low-CTR pages:**
- Impressions > 500
- CTR < site average for that position bracket
- Position ≤ 20 (realistic click potential)
- **OR** Position 21-30 with impressions > 1000 (high-volume ranking opportunities)

**Wasted impression queries:**
- Impressions > 100
- CTR < 2% (or < half the position-bracket average)
- Position ≤ 10 (should be getting clicks)

**Ranking opportunities (separate category):**
- Position 21-50 with impressions > 1000
- These pages rank but need content improvement to reach page 1
- Flag for `/refresh-content` rather than just meta tag fixes

### 2.2 Quick Win Filter

Prioritize opportunities where small changes yield big impact:

- Position 1-3 with CTR < 5% → title/description mismatch with intent
- Position 4-10 with CTR < 2% → being outcompeted by better snippets
- High impression queries not reflected in current title → keyword gap

### 2.3 Group by Page

Consolidate query-level findings to page level:
- List all underperforming queries per page
- Sum total wasted impressions per page
- Identify the primary keyword (highest impressions) for each page

### 2.4 Fetch Current Meta Tags from Webflow

**Do this BEFORE making recommendations.** For each flagged page URL:

1. Parse the URL to extract collection slug and item slug
2. Use Webflow MCP `get_collection_list` to find collections
3. Match URL path to collection + item
4. Fetch the CMS item using `list_collection_items` with slug filter
5. Use `get_collection_details` to identify meta title and meta description fields
6. Store current values:
   - Current meta title (or `name` field if no dedicated meta title)
   - Current meta description (or `post-summary` if no dedicated field)

**For static pages (not in CMS):**
1. Use Webflow MCP `list_pages` to find the page by path
2. Use `get_page_metadata` to fetch current SEO title and description
3. Mark as "static page" for different update method later

⚡ GUARD — **Meta description field missing in collection:**
If the collection has no meta description field:
- Use `post-summary` or similar if available
- If no suitable field exists: note in report "No meta description field. Consider adding `meta-description` field to this collection."
- Still generate recommended meta descriptions for manual use

Store all current values — these are needed for the before/after comparison in recommendations.

---

## Phase 3: Validate with Keywords Everywhere

**IMPORTANT**: Do not silently skip this phase. Either:
1. Use KE data to enrich recommendations, OR
2. Explicitly note in the report that KE was unavailable (see Prerequisites Check)

If Keywords Everywhere MCP is available:

### 3.1 Enrich Query Data

For each flagged query, fetch:
- Monthly search volume
- CPC (indicates commercial intent)
- Competition score
- Trend data (rising/falling)

### 3.2 Intent Classification

Classify each query by intent:
- **Informational**: how, what, why, guide, tutorial
- **Commercial**: best, top, review, vs, compare
- **Transactional**: buy, price, discount, deal, near me
- **Navigational**: brand names, specific product names

### 3.3 Validate Demand

Filter out queries with:
- Search volume < 50/month (unless highly commercial)
- Falling trend > 30% YoY (dying queries)

Flag high-value queries:
- Volume > 1000/month
- CPC > $2 (commercial intent)
- Rising trend

---

## Phase 4: Prioritize

### 4.1 Scoring Model

Score each page opportunity (0-100):

| Factor | Weight | Scoring |
|--------|--------|---------|
| Wasted impressions | 30% | Log scale, max at 10k |
| Position | 25% | Higher score for positions 1-10 |
| CTR gap | 20% | Current CTR vs. expected CTR for position |
| Keyword value | 15% | Based on volume × CPC (if available) |
| Quick fix potential | 10% | Title missing primary keyword = +10 |

### 4.2 Rank Opportunities

Sort pages by score, descending. Group into tiers:

- **Tier 1 (Score 70+)**: Immediate action — significant wasted traffic
- **Tier 2 (Score 50-69)**: High priority — solid improvement potential
- **Tier 3 (Score 30-49)**: Worth doing — incremental gains

---

## Phase 5: Recommend

### 5.1 Output Format

Present findings as a structured report:

```
# Click Recovery Report

**Site**: [property URL]
**Period**: Last 90 days
**Total wasted impressions**: [sum across all flagged pages]
**Estimated recoverable clicks**: [impressions × expected CTR improvement]

**Data sources**:
- ✅ GSC: Connected
- ✅ Webflow: Connected [or ⚠️ Not connected — analysis only]
- ✅ Keywords Everywhere: Connected [or ⚠️ Not connected — no volume/intent data]

[If KE not connected, add:]
> ⚠️ **Keywords Everywhere not connected.** Recommendations based on GSC impressions only.
> For search volume and intent data, connect KE: https://github.com/hithereiamaliff/mcp-keywords-everywhere

---

## Tier 1: Immediate Action

### 1. [Page Title]
**URL**: [page URL]
**Current performance**:
- Impressions: [X] | Clicks: [X] | CTR: [X]% | Avg Position: [X]
- Expected CTR for position: [X]% | CTR gap: [X]%

**Top underperforming queries**:
| Query | Impressions | CTR | Position | Volume | Intent |
|-------|-------------|-----|----------|--------|--------|
| ...   | ...         | ... | ...      | ...    | ...    |

**Diagnosis**: [Why is CTR low? Title mismatch? Weak description? Competitor snippets?]

**Recommended changes**:

| Field | Current | Suggested | Chars |
|-------|---------|-----------|-------|
| Meta title | [current title] | [suggested title] | [N]/60 |
| Meta description | [current desc or "❌ Missing"] | [suggested desc] | [N]/155 |

**Keyword framing**: [advice based on intent]

**Estimated impact**: +[X] clicks/month

**Page type**: [CMS item / Static page]

---

[Repeat for each Tier 1 page]

## Tier 2: High Priority
[Summary table with key metrics, current titles, suggested titles]

## Tier 3: Worth Doing
[Summary table with key metrics]

## Ranking Opportunities (Position 21-50)

These pages have high impressions but need more than meta tag fixes to reach page 1:

| Page | Impressions | Position | Recommendation |
|------|-------------|----------|----------------|
| [title] | [X] | [X] | Run `/refresh-content` for full optimization |

---

## Next Steps

1. Review and approve Tier 1 recommendations
2. Publish approved changes directly (this skill handles it)
3. For ranking opportunities: run `/refresh-content [URL]`
4. Monitor CTR in GSC after 2-4 weeks
5. Re-run `/click-recovery` monthly
```

### 5.2 Title & Meta Description Guidelines

When suggesting rewrites:

**Titles (≤60 characters):**
- Lead with primary keyword
- Include a compelling hook (number, year, power word)
- Match search intent (informational = "Guide", commercial = "Best", etc.)
- Differentiate from competitors in SERP

**Meta descriptions (≤155 characters):**
- Include primary keyword naturally
- Add a clear value proposition
- Include a call-to-action or curiosity hook
- Mention specifics (numbers, timeframes, outcomes)

**Intent-based framing:**
- Informational: "Learn how to...", "Complete guide to...", "X steps to..."
- Commercial: "Best X for [use case]", "X vs Y: Which is...", "Top X in [year]"
- Transactional: "Get X today", "Free X", "X% off", "Starting at $X"

### 5.3 Competitor SERP Analysis (Optional)

For Tier 1 pages, offer to analyze the SERP:

1. Web search the primary keyword
2. Extract titles and descriptions of top 5 results
3. Identify patterns (what hooks are they using?)
4. Find differentiation opportunities

---

## Phase 6: Execute

### 6.1 User Approval

After presenting the report, ask the user:

```
Which pages would you like to update?
1. All Tier 1 pages ([N] pages)
2. Select specific pages
3. Review only — don't publish yet
```

If user selects option 3, stop here. They can run the skill again or use `/refresh-content` manually.

### 6.2 Map Pages to CMS Items

For each approved page:

1. Extract the URL path from the GSC data
2. Use Webflow MCP `get_collection_list` to find all collections
3. Match the URL path to a collection slug and item slug
4. Use `list_collection_items` to fetch the CMS item by slug
5. Store the item ID and current field values

⚡ GUARD — **Page not found in CMS (static page):**
If a URL doesn't match any CMS item, check if it's a static page:

1. Use Webflow MCP `list_pages` to search for the page by path
2. If found as a static page:
   - Use `update_page_settings` to update SEO title and meta description
   - Include in the publish workflow
3. If not found anywhere (external URL or deleted page):
   - Inform user: "[URL] not found in Webflow. It may be external or deleted."
   - Skip this page and continue with others

**For static pages like homepage `/` or `/contact`:**
- These are updated via `update_page_settings`, not CMS
- The page must be republished after updating settings

### 6.3 Map CMS Fields

For each CMS item, identify the correct fields using `get_collection_details`:

- **Meta title field**: may be `meta-title`, `seo-title`, `titile`, or the main `name` field
- **Meta description field**: may be `meta-description`, `seo-description`, `post-summary`, or similar

Store the field mapping for updates.

### 6.4 Present Changes for Final Approval

Before publishing, show the exact changes with warnings:

```
## Ready to Publish

| Page | Field | Current | New | Status |
|------|-------|---------|-----|--------|
| [title] | Meta title | [old] | [new] | ✅ 52 chars |
| [title] | Meta title | [old] | [new] | ⚠️ 68 chars — will truncate |
| [title] | Meta title | [old] | [new] | ⚠️ 25 chars — too short |
| [title] | Meta description | [old] | [new] | ✅ 142 chars |
| ... | ... | ... | ... | ... |

Publish these changes to Webflow? (yes/no)
```

**Title length warnings:**
- ⚠️ **> 60 chars**: "Title will be truncated in search results. Consider shortening."
- ⚠️ **< 30 chars**: "Title is short — missing opportunity to include keywords or hooks."
- ✅ **30-60 chars**: Optimal length

**Meta description warnings:**
- ⚠️ **> 155 chars**: "Description may be truncated."
- ⚠️ **< 70 chars**: "Description is short — add more value proposition."
- ✅ **70-155 chars**: Optimal length

If any warnings exist, ask user: "Some titles/descriptions have length issues. Proceed anyway, or revise first?"

**Wait for explicit user confirmation before publishing.**

### 6.5 Update CMS Items

On confirmation, for each approved page:

1. Call Webflow MCP `update_collection_item` with the new meta title and description
2. Log success or failure for each item

### 6.6 Publish Changes

After all items are updated:

1. Ask user: "Publish changes now or save as drafts?"
2. If publish: call `publish_collection_items` for all updated items
3. If draft: inform user changes are saved but not live

### 6.7 Summary

Present a final summary:

```
## Click Recovery Complete

**Updated**: [N] pages ([N] CMS items, [N] static pages)
**Published**: [Yes/No — saved as drafts]

| Page | Type | Status | Title Updated | Description Updated |
|------|------|--------|---------------|---------------------|
| [title] | CMS | ✅ Published | ✅ | ✅ |
| [title] | Static | ✅ Published | ✅ | ✅ |
| [title] | CMS | ⚠️ No desc field | ✅ | ❌ Add field manually |
| [title] | — | ❌ Not found | — | — |

## Re-check Schedule

| When | What to check |
|------|---------------|
| **2-3 weeks** | Verify changes are indexed (`site:URL` in Google). Check GSC for crawl errors. |
| **4-6 weeks** | Measure CTR impact. Compare clicks before/after. Note any position changes. |
| **Monthly** | Re-run `/click-recovery` to find new opportunities as rankings shift. |

## Next Steps
- For ranking opportunities (position 21+), use `/refresh-content [URL]`
- For pages missing meta description field, add field in Webflow Designer
```

---

## Conditional Guards

⚡ GUARD — **GSC MCP unavailable:**
This skill requires GSC data. If unavailable:
- Inform user: "Click Recovery requires Google Search Console access. Please connect GSC MCP and try again."
- Stop execution

⚡ GUARD — **Webflow MCP unavailable:**
If Webflow MCP is not connected:
- Run analysis and recommendations (Phases 1-5)
- Skip execution phase
- Inform user: "Webflow MCP not connected. Recommendations generated but cannot publish. Connect Webflow MCP and run again, or update manually."

⚡ GUARD — **No opportunities found:**
If filtering returns zero pages:
- Inform user: "No significant CTR opportunities found. Your titles and meta descriptions are performing well relative to your positions."
- Suggest: "Consider running `/refresh-content` on older articles to improve rankings first, then re-check CTR."

⚡ GUARD — **Keywords Everywhere unavailable:**
If KE API is not available:
- Proceed without volume/intent data
- Note in report: "Search volume and intent data unavailable. Recommendations based on GSC impressions only."
- Suggest user add KE API for richer insights

⚡ GUARD — **Low traffic site:**
If total impressions < 1000 in 90 days:
- Warn user: "Limited data available. Results may not be statistically significant."
- Proceed but note confidence is lower

---

## Error Handling

| Error | Action |
|-------|--------|
| GSC MCP not connected | Stop and instruct user to connect GSC |
| GSC property not found | List available properties, ask user to select |
| No data for date range | Try shorter range (28 days), warn about limited data |
| Keywords Everywhere API error | Proceed without KE data, note in report |
| Rate limit hit | Pause, retry with exponential backoff |
| Webflow MCP not connected | Run analysis only, skip publishing |
| Page URL not found in CMS | Check if static page via `list_pages`, use `update_page_settings` |
| Page not found anywhere | Skip page, may be external or deleted |
| CMS field not found | Try common field name variations, ask user if still not found |
| No meta description field in collection | Note in report, suggest adding field, still provide recommendations |
| Static page update fails | Check page permissions, may need to publish site |
| Publish fails | Show error, check if item is in draft-only state |
| Meta title > 60 chars | Warn user, suggest shortening before publish |
| Meta description > 155 chars | Warn user, suggest shortening before publish |

---

## Success Metrics

After implementing recommendations, track at each milestone:

**2-3 weeks (indexing check):**
- Are updated pages indexed? (`site:URL` in Google)
- Any crawl errors in GSC?

**4-6 weeks (impact measurement):**
- CTR change per page (compare to pre-update baseline)
- Click change per page
- Position stability (ensure rankings held or improved)

**Monthly (ongoing):**
- Total organic traffic change
- New CTR opportunities from ranking shifts
- Re-run `/click-recovery` to catch new issues

---

## Integration with /refresh-content

Click Recovery focuses on **meta tags** (title + description) for CTR optimization.
Refresh Content does **full content refreshes** (body, keywords, FAQs, schema, internal links).

**When to use each:**

| Scenario | Use |
|----------|-----|
| Low CTR, content is fine | `/click-recovery` — fix the pitch, not the content |
| Outdated content, rankings dropping | `/refresh-content` — full content overhaul |
| Both CTR and content issues | Run `/click-recovery` first for quick wins, then `/refresh-content` for deeper fixes |

**Workflow:**
1. Run `/click-recovery` monthly to catch CTR opportunities
2. Approve and publish meta tag updates directly
3. For pages needing more than meta fixes, run `/refresh-content [URL]`
