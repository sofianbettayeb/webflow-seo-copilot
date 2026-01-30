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

- **Required**: [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server) connected
- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) connected (for publishing changes)
- **Optional**: Keywords Everywhere API key (for search volume and intent data)

If GSC is unavailable, this skill cannot run. Inform the user and stop.
If Webflow MCP is unavailable, run analysis only and output recommendations without publishing.

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

**Wasted impression queries:**
- Impressions > 100
- CTR < 2% (or < half the position-bracket average)
- Position ≤ 10 (should be getting clicks)

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

---

## Phase 3: Validate with Keywords Everywhere (Optional)

If Keywords Everywhere API is available:

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
- **Title**: [current] → [suggested]
- **Meta description**: [current] → [suggested]
- **Keyword framing**: [advice based on intent]

**Estimated impact**: +[X] clicks/month

---

[Repeat for each Tier 1 page]

## Tier 2: High Priority
[Summary table with key metrics]

## Tier 3: Worth Doing
[Summary table with key metrics]

---

## Next Steps

1. Review Tier 1 recommendations
2. Run `/refresh-content [URL]` to execute changes
3. Monitor CTR in GSC after 2-4 weeks
4. Re-run `/click-recovery` monthly to find new opportunities
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

⚡ GUARD — **Page not found in CMS:**
If a URL doesn't match any CMS item:
- It may be a static page (not in CMS) or an external URL
- Inform user: "[URL] is not a CMS item. Update the page manually in Webflow Designer."
- Skip this page and continue with others

### 6.3 Map CMS Fields

For each CMS item, identify the correct fields using `get_collection_details`:

- **Meta title field**: may be `meta-title`, `seo-title`, `titile`, or the main `name` field
- **Meta description field**: may be `meta-description`, `seo-description`, `post-summary`, or similar

Store the field mapping for updates.

### 6.4 Present Changes for Final Approval

Before publishing, show the exact changes:

```
## Ready to Publish

| Page | Field | Current | New |
|------|-------|---------|-----|
| [title] | Meta title | [old] | [new] ([N] chars) |
| [title] | Meta description | [old] | [new] ([N] chars) |
| ... | ... | ... | ... |

Publish these changes to Webflow? (yes/no)
```

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

**Updated**: [N] pages
**Published**: [Yes/No — saved as drafts]

| Page | Status | New Meta Title |
|------|--------|----------------|
| [title] | ✅ Published | [new title] |
| [title] | ✅ Published | [new title] |
| [title] | ⚠️ Not in CMS | Manual update needed |

## Next Steps
- Monitor CTR in GSC (2-4 weeks)
- Re-run `/click-recovery` monthly
- For full content refreshes, use `/refresh-content [URL]`
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
| Page URL not found in CMS | Skip page, inform user to update manually |
| CMS field not found | Try common field name variations, ask user if still not found |
| Publish fails | Show error, check if item is in draft-only state |
| Meta title > 60 chars | Warn user, suggest shortening before publish |
| Meta description > 155 chars | Warn user, suggest shortening before publish |

---

## Success Metrics

After implementing recommendations, user should track:
- CTR change per page (2-4 weeks)
- Click change per page (2-4 weeks)
- Position stability (ensure rankings held)
- Total organic traffic change (30 days)

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
