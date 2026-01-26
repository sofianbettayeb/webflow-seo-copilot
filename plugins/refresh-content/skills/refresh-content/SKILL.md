---
name: refresh-content
description: |
  Refresh and optimize Webflow blog articles to improve rankings, recover traffic, and extend content lifespan.
  Triggers: refresh blog, update article, actualise content, improve rankings, recover traffic, refresh content, update blog post, SEO refresh, content refresh, refresh my article.
  Requires: Webflow MCP server. Optional: Google Search Console MCP server for keyword data.
  Workflow: Guided with approval gates — proposes changes before executing via MCP.
---

# Webflow Blog Refresh Skill

Refresh existing blog articles to improve rankings, recover lost traffic, extend content lifespan, and increase CTR.

## Prerequisites

- **Required**: Webflow MCP server connected
- **Optional**: GSC MCP server (enables keyword refresh, long-tail discovery, indexing submission)

Check MCP availability before starting. If GSC unavailable, use fallback keyword research methods (see 1.5).

## Workflow Overview

```
ANALYZE → PLAN → APPROVE → EXECUTE → VERIFY → SUBMIT → MONITOR
```

All major actions require user approval before execution.

---

## Conditional Guards

This skill uses **conditional guards** — checks that only trigger when information is missing or a specific situation is detected. Guards keep the workflow lean for simple refreshes while catching gaps automatically.

A guard follows this pattern:
```
IF [condition is detected] → THEN [perform action]
OTHERWISE → skip silently and continue
```

Guards are embedded inline in the relevant steps below, marked with `⚡ GUARD:`.

---

## Phase 1: Analyze

### 1.0 Locate the Article

Parse the user's input (URL or article name) to find the CMS item.

1. If a URL is provided, extract the collection slug and item slug from the path
   - URL pattern: `site.com/{collection-slug}/{item-slug}`
2. Match the collection slug against the site's collections using `get_collection_list`
3. Fetch the item by slug using `list_collection_items`

⚡ GUARD — **Collection not found by slug:**
If the collection slug from the URL doesn't match any collection, or the item isn't found:
1. Search across ALL collections by item slug
2. If still not found, ask the user which collection the article belongs to

⚡ GUARD — **Article status check:**
After fetching the item, check `isDraft` and `lastPublished`:
- If **draft (never published)** → note this in the plan. Slug changes are safe, no redirect needed, content can be fully rewritten.
- If **published** → standard refresh. Flag any slug changes as requiring a 301 redirect.
- If **archived** → ask user: revive or skip?

### 1.1 Map CMS Schema

**This step is mandatory.** Do not assume field names — they vary per site.

Use Webflow MCP `get_collection_details` to discover:
- Actual field names, types, and slugs for the target collection
- Map each field to its purpose:
  - Title field (e.g., `name`, `title`, `titile`)
  - Meta title field (may differ from display title)
  - Meta description / summary field
  - Rich text body field
  - Image fields (featured, thumbnail)
  - Date fields (published, last modified)
  - Keyword/tag fields (e.g., Synqpro keyword fields)
  - Author reference fields
- Flag any missing fields the refresh needs (e.g., no `last-updated` field)

Store the field mapping for use in Phase 3.

⚡ GUARD — **Dedicated FAQ fields detected:**
If the collection has fields matching patterns like `faq-question-*` / `faq-answer-*`:
- Note how many FAQ slots are available (e.g., 5 Q&A pairs)
- Plan to populate both CMS FAQ fields AND body FAQ section
- If no dedicated FAQ fields exist: add FAQ in body content only

⚡ GUARD — **Author field detected but empty:**
If the collection has an author/contributor reference field AND it's empty on this article:
- Fetch the referenced Authors collection to list available authors
- Include author assignment in the Phase 2 plan
- If the author field is already populated: skip silently

⚡ GUARD — **SEO score field detected:**
If the collection has an SEO score field (e.g., Synqpro score):
- Record the current score value
- Include it in the before/after diff in Phase 3
- Recommend running the SEO tool after updates to verify improvement
- If no score field exists: skip silently

### 1.2 Fetch Article Data

Use Webflow MCP `list_collection_items` with slug filter to retrieve:
- All field values for the target article
- Item ID (needed for update)
- Current publish/draft status
- Last published and last updated dates

**Store the original field values** — these are needed for the before/after diff in Phase 3 and for rollback.

⚡ GUARD — **Empty fields audit:**
After fetching, scan all fields for empty values. Flag any that should be populated:
- Empty keyword fields → include in plan
- Empty topic field → include in plan
- Empty read-time field → include in plan
- Empty author → see author guard above
- Empty FAQ fields → see FAQ guard above
- If all fields are populated: skip silently, only update what the refresh requires

### 1.3 Fetch GSC Data (if available)

Use GSC MCP to retrieve for the article URL:
- Top queries by impressions (last 90 days)
- Queries in positions 5-20 (opportunity zone)
- Queries with high impressions but low CTR
- Click and impression trends

If GSC unavailable, proceed to step 1.5 for fallback keyword research.

### 1.4 Discover Internal Linking Opportunities

**This step is mandatory.** Do not skip internal linking.

1. List ALL CMS collections on the site using `get_collection_list`
2. For each content-related collection (not utility collections like Authors or Clients), fetch item names and slugs using `list_collection_items`
3. Identify 4-8 topically relevant articles based on:
   - Keyword/topic overlap with the article being refreshed
   - Complementary content (guides, reviews, tutorials on related subjects)
   - Pillar/cluster content relationships
4. Map each potential internal link to a specific section in the article
5. Present the internal linking plan as a table:

| Article | URL Path | Target Section |
|---------|----------|----------------|
| [name]  | [slug]   | [section]      |

### 1.5 Keyword Research (GSC Fallback)

When GSC MCP is unavailable, use these alternative methods:

1. **Web search** for the article's primary keyword to find:
   - "People Also Ask" questions (use for FAQ section)
   - Related searches at bottom of SERP
   - Top 3 competing articles — note their headings, structure, and word count
2. **Analyze existing content** for:
   - Primary keyword and variations already used
   - Keyword gaps compared to competing articles
   - Missing semantic keywords and entities
3. Inform user: "GSC data unavailable. Keyword optimization is based on content analysis and SERP research."

### 1.6 Content Audit

Analyze current content for:
- Outdated dates, statistics, or references (flag every year mention)
- Missing or weak H2/H3 heading structure
- Long paragraphs needing breaks (max 3-4 sentences per paragraph)
- Images without alt text
- Missing key takeaways / TL;DR section
- Missing FAQ section
- Missing or outdated schema markup
- Thin sections that need expansion
- Broken or outdated external links

⚡ GUARD — **Embedded images in body content:**
If the rich text body contains `<img>` tags or `<figure>` elements:
- Count and list all embedded images with their current alt text and URLs
- Warn user: "This article contains [N] inline images. Rewriting the body will remove them unless explicitly preserved."
- Ask: preserve existing images in the new content, or proceed without them?
- If preserving: extract each image's URL, alt text, and approximate position (which section it appears in), then re-insert `<img>` tags at appropriate positions in the new content
- If not preserving: proceed and note "inline images removed" in the before/after diff

⚡ GUARD — **CMS image fields have null or bad alt text:**
If CMS image fields (`main-image`, `thumbnail-image`, or similar) have `alt: null` or alt text containing placeholder values like `__wf_reserved_inherit`:
- Flag in the plan: "[N] CMS image fields need alt text updates"
- When updating, pass the full image object with the updated alt value:
  `{"fileId": "existing-id", "url": "existing-url", "alt": "new descriptive alt text"}`
- Generate descriptive alt text based on the article topic and image context
- If image fields have proper alt text already: skip silently

⚡ GUARD — **Cross-article consistency:**
If key topics, products, or tools mentioned in this article also appear in other articles discovered during step 1.4:
- Note which other articles mention the same topics
- Check for inconsistencies in descriptions, pricing, feature lists, or positioning
- If inconsistencies found: flag in the plan and recommend aligning messaging
- If no overlapping topics or descriptions are consistent: skip silently

⚡ GUARD — **Major rewrite detected:**
Trigger if ANY of these conditions are met:
- User requests a change in tone, audience, or scope (e.g., "make it a beginner edition", "rewrite for developers", "change the angle")
- Content length will increase or decrease by more than 50%
- More than 3 new sections are being added
- Article structure is fundamentally changing (e.g., 4 items → 7 items in a listicle)

If triggered, generate a **content brief** before writing:
  - Target audience and expertise level
  - Target word count (based on competitor analysis)
  - Proposed heading structure (H2/H3 outline)
  - Key points to cover
  - Tone guidance
  - Images to preserve or replace (if applicable)
- Present the content brief in the Phase 2 plan for approval
- If the refresh is a standard update (same audience, same tone, similar scope): skip the content brief

---

## Phase 2: Plan

Generate a refresh plan. Reference `references/refresh-checklist.md` for the complete checklist.

### Present Plan to User

Format the plan as:

```
## Refresh Plan for: [Article Title]

### Article Status: [Published / Draft / Archived]

### Date & Freshness Signals
- [ ] Update last-modified field to [DATE]
- [ ] Add "[YEAR]" to title: "[NEW TITLE]"
- [ ] Update meta title: "[NEW META TITLE]" ([N] chars)
- [ ] Update meta description: "[NEW META DESCRIPTION]" ([N] chars)
- [ ] Update year references in body content ([N] instances)

### Content Updates
- [ ] Add key takeaways section at top
- [ ] [List specific outdated info to update]
- [ ] Improve scannability: [specific changes]
- [ ] Update alt text for [N] images
- [ ] [Any new sections to add]

### Keywords
- [ ] Target keywords to strengthen: [list]
- [ ] Long-tail opportunities: [list]
- [ ] Update keyword/tag CMS fields: [field name] → [new values]
- [ ] Source: [GSC data / SERP research / content analysis]

### Internal Links
- [ ] Add [N] internal links:
  | Article | URL Path | Target Section |
  |---------|----------|----------------|
  | ...     | ...      | ...            |

### External Links
- [ ] Add outbound links to: [list sources]
- [ ] Remove/replace outdated links: [list]

### Structured Data
- [ ] Add/update FAQ section with [N] questions
- [ ] Add Article schema
- [ ] Add FAQ schema

### Empty Fields to Populate [if any detected by guards]
- [ ] [field name]: [proposed value]

### Submission
- [ ] [GSC: Request indexing] or [Manual: Submit URL to GSC]

Estimated impact: [ranking recovery / traffic boost / CTR improvement]
```

**Wait for user approval before proceeding.**

---

## Phase 3: Execute

After approval, execute changes via Webflow MCP.

### 3.1 Update CMS Fields

Update fields in order using the field mapping from step 1.1:

1. Title field (with year/freshness signal)
2. Meta title field
3. Meta description / summary field
4. Main content (rich text body field) — with all content changes, internal links, FAQ section
5. Keyword/tag fields (update with refreshed target keywords)
6. Image alt texts (if updatable via CMS)
7. Last-modified date field (if it exists)

⚡ GUARD — **Dedicated FAQ fields exist:**
If FAQ CMS fields were detected in step 1.1:
- Populate `faq-question-*` and `faq-answer-*` fields in addition to body content
- If no dedicated FAQ fields: FAQ goes in body only

⚡ GUARD — **Empty fields flagged in step 1.2:**
Populate any empty fields identified by the audit:
- Topic, read-time, keywords, secondary keywords, author, etc.
- Only populate fields included in the approved plan

⚡ GUARD — **Slug change on published article:**
If the slug is being changed AND `lastPublished` exists:
- Warn user: "This article was previously published. Changing the slug will break existing links and search rankings."
- Add 301 redirect instruction to post-publish checklist: old slug → new slug
- Ask user to confirm or keep the old slug
- If the article is a draft (never published): change slug freely, no warning needed

⚡ GUARD — **New external links added:**
If new external links are being added to the article:
- Fetch each new URL to verify it returns a valid response (not 404 or error)
- Flag any broken links before publishing
- If a link is broken: warn user and suggest removing or replacing it
- If all links are valid: skip silently and continue

### 3.2 Generate Schema

Reference `references/schema-templates.md` for JSON-LD templates.

Construct schema markup by substituting actual values from CMS fields:
- Article schema (required for every refresh)
- FAQ schema (if FAQ section was added)
- Breadcrumb schema (recommended)
- HowTo schema (if step-by-step content)

Combine multiple schemas using the `@graph` pattern from the templates.

Add schema to page custom code or embed block via Webflow MCP.

### 3.3 Present Before/After Diff

Before asking user to confirm publication, show a change summary:

```
## Changes Summary

| Field | Before | After |
|-------|--------|-------|
| Title | [old title] | [new title] |
| Meta title | [old] | [new] ([N] chars) |
| Meta description | [old] | [new] ([N] chars) |
| Keywords | [old or empty] | [new] |
| SEO Score | [old score or N/A] | [re-check after publish] |

### Content Changes
- Sections added: [list]
- Sections modified: [list]
- Internal links added: [N] (to: [list slugs])
- External links added/updated: [N]
- FAQ questions added: [N] (body + [N] CMS fields)
- Year references updated: [old year] → [new year] ([N] instances)
- Schema markup: [added/updated]
- Empty fields populated: [list field names]
```

### 3.4 Confirm Before Publishing

Ask user: "Ready to publish these changes? Or would you like to review in the Webflow Designer first?"

### 3.5 Publish

On user confirmation, use `publish_collection_items` to publish the updated item.

---

## Phase 4: Submit

### If GSC MCP available:
1. Request indexing for updated URL
2. Inform user: "Indexing requested. Check GSC in 24-48 hours for status."

### If GSC unavailable:
- Inform user: "Manually submit this URL to Google Search Console for faster indexing: [URL]"

---

## Phase 5: Post-Publish Monitoring

Provide the user with a monitoring checklist:

```
## Post-Publish Checklist (follow up in 7 days)

- [ ] Verify page is indexed: search `site:[article URL]` in Google
- [ ] Check for crawl errors in GSC
- [ ] Monitor position changes for target keywords (7-14 days)
- [ ] Check if FAQ rich results appear in SERP
- [ ] Verify all internal links resolve correctly
- [ ] Review CTR changes (14-30 days)
- [ ] Compare organic traffic to page (30 days vs. prior 30 days)
```

⚡ GUARD — **Slug was changed on a published article:**
Add to the checklist:
- [ ] Set up 301 redirect: `/{old-slug}` → `/{new-slug}` (Webflow: Settings → Hosting → 301 Redirects)
- [ ] Verify the redirect works by visiting the old URL

⚡ GUARD — **SEO score field exists:**
Add to the checklist:
- [ ] Run SEO tool (e.g., Synqpro) to verify score improved after refresh
- [ ] Compare: old score [X] → new score [check]

---

## Content Quality Guidelines

When rewriting or adding content, follow these standards:

### Tone & Voice
- Match the existing site's tone (analyze 2-3 other articles if unsure)
- Write for the target audience's expertise level
- Be direct and actionable — avoid filler and fluff

### Structure
- Every H2 section: 100-300 words
- Paragraphs: max 3-4 sentences
- Use bullet lists for 3+ items
- Use tables for comparisons
- Include a key takeaways section (3-5 bullet points)

### E-E-A-T Signals
- Reference the author's credentials where relevant
- Cite specific data, statistics, or studies
- Include first-hand experience signals ("we tested", "based on our analysis")
- Link to authoritative sources

### SEO Writing
- Primary keyword in H1, first paragraph, and 1-2 H2s
- Use keyword variations naturally — don't force exact match
- FAQ answers: 40-60 words (featured snippet optimal)
- Meta title: ≤60 characters
- Meta description: ≤155 characters

### Webflow Rich Text Limitations
Be aware of these constraints when editing body content via the CMS API:
- **Tables**: HTML `<table>` is supported in rich text but may need custom CSS in the site's CMS template to render well. Warn user if adding tables for the first time.
- **Embeds**: not supported in rich text fields via API. Use Webflow Designer for embed blocks (e.g., schema script tags).
- **Images**: can be referenced by URL in `<img>` tags but cannot be uploaded via the CMS API. Existing inline images can be preserved by keeping their `<img>`/`<figure>` tags intact in the new content. New images must be uploaded via the Webflow Designer or Asset API.
- **Custom attributes**: rich text strips most custom HTML attributes. Only standard tags and attributes are preserved.

---

## Bulk Refresh Mode (Optional)

When refreshing multiple articles in one session:

1. Accept a list of slugs, or use a collection filter (e.g., "all articles last published before [date]")
2. Run Phase 1 analysis on all articles
3. Prioritize by:
   - Oldest `lastPublished` date
   - Lowest SEO score (if score field exists)
   - Highest traffic potential (if GSC available)
4. Present a prioritized list to the user
5. Execute sequentially with approval gates per article (or per batch if user prefers)

---

## Freshness Signals Checklist

When updating for freshness, ensure ALL of these are addressed:

- [ ] `dateModified` in Article schema markup
- [ ] Year in meta title
- [ ] Year in H1 or first H2
- [ ] Updated statistics and data points
- [ ] Recent external source links (published within last 2 years)
- [ ] "Updated [Month Year]" visible on page (if CMS field exists)
- [ ] Keyword/tag fields reflect current terminology

---

## Rollback

If issues arise after publishing:
- The previous field values were stored in step 1.2
- Revert by calling `update_collection_items` with the original `fieldData`
- Re-publish after reverting
- Inform user of the rollback and investigate the issue

---

## Error Handling

| Error | Action |
|-------|--------|
| Webflow MCP unavailable | Stop and inform user to connect Webflow MCP |
| GSC MCP unavailable | Use fallback keyword research (step 1.5), skip GSC submission steps |
| CMS field not found | Check field mapping from step 1.1; recommend field structure if missing |
| Schema validation fails | Show error, provide corrected schema |
| Collection item not found | Search across all collections by slug, then ask user |
| Publish fails | Show error, confirm item is not in draft-only state |
| Content too long for field | Warn user, suggest trimming or splitting content |
| Slug change on published article | Warn about broken links, recommend 301 redirect |
| Broken external link detected | Warn user, suggest alternative URL or removal |
| Inline images lost during rewrite | Warn user before executing, offer to preserve |
| CMS image alt text is null/placeholder | Update image field with descriptive alt text |

---

## FAQ Generation Guidelines

When creating FAQ section:
- Research "People Also Ask" via web search for the article's primary keyword
- Use exact PAA phrasing for questions when possible
- Answers: 40-60 words (featured snippet optimal)
- Include target keyword naturally in 1-2 answers
- Limit to 5-7 FAQs per article
- Structure: H3 for each question, paragraph for each answer
- Generate matching FAQ schema markup

⚡ GUARD — **Dedicated FAQ CMS fields exist:**
- Match FAQ count to available CMS slots (e.g., 5 fields = 5 FAQs max in CMS)
- Populate CMS fields with the same Q&A used in body content
- If more FAQs are needed than CMS slots allow: extras go in body only

---

## Success Metrics

After refresh, track (user should monitor):
- Position changes for target keywords (7-14 days)
- Click-through rate changes (14-30 days)
- Organic traffic to page (30 days)
- FAQ rich result appearances in SERP
- Internal link click-through from linked articles
