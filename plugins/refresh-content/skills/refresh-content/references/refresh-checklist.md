# Blog Article Refresh Checklist

## Phase 1: Setup & Discovery

### 1. Map CMS Schema
- Call `get_collection_details` for the target collection
- Document all field names, types, and slugs
- Map fields: title, meta title, meta description, body, images, dates, keywords, author
- Flag missing fields (e.g., no `last-updated` field)

### 2. Store Original Content
- Save all current field values before making changes
- These are needed for before/after diff and rollback

## Phase 2: Date & Freshness Signals

### 3. Update "Last Updated" Date
- Locate the `last-updated` or `modified-date` CMS field
- Set to current date
- If field doesn't exist, recommend adding it to the CMS collection

### 4. Add Date to Title
- Prepend or append current year/month to title
- Patterns: "[2026]", "(Updated January 2026)", "in 2026"
- Choose pattern based on existing title structure

### 5. Update Meta Title & Description
- Include freshness signal in meta title (year or "Updated")
- Meta description: mention recent updates or current year
- Keep within limits: title ≤60 chars, description ≤155 chars

### 6. Update All Year References
- Search body content for all previous year mentions
- Replace with current year
- Verify headings, stats, and contextual references are updated

## Phase 3: Content Enhancement

### 7. Propose Fresh Information
- Identify outdated statistics, tools, or references
- Research current data to replace outdated info
- Flag sections that mention specific years or versions

### 8. Add Summary / Key Takeaways
- Add TL;DR or Key Takeaways section at top
- Use 3-5 bullet points for scannability
- Include primary keyword naturally

### 9. Improve Scannability
- Ensure H2/H3 hierarchy is logical
- Break long paragraphs (max 3-4 sentences)
- Add lists where 3+ items are mentioned
- Consider tables for comparisons
- Add image alt texts to all images

### 10. Content Quality Check
- Match existing site tone and voice
- H2 sections: 100-300 words each
- Include E-E-A-T signals (author expertise, data citations, first-hand experience)
- Remove filler content — every sentence should add value

## Phase 4: Keyword Optimization

### 11. Refresh Target Keywords (with GSC)
- Pull top queries driving impressions
- Identify keywords ranking positions 5-20 (opportunity zone)
- Naturally incorporate high-impression keywords

### 12. Refresh Target Keywords (without GSC — fallback)
- Web search for primary keyword to find related searches and PAA
- Analyze top 3 competing articles for keyword gaps
- Identify missing semantic keywords and entities
- Note competing article structure, headings, and word count

### 13. Identify Long-Tail Keywords
- With GSC: find queries with impressions but low CTR
- Without GSC: extract question-based phrases from PAA and related searches
- Add sections or FAQs addressing these queries

### 14. Update Keyword CMS Fields
- Update any keyword/tag fields in the CMS (e.g., Synqpro keyword fields)
- Refresh with current target keywords

## Phase 5: Internal Linking

### 15. Discover Internal Link Opportunities
- List ALL CMS collections on the site
- Fetch item names and slugs from related collections
- Identify 4-8 topically relevant articles
- Criteria: keyword overlap, complementary content, pillar/cluster relationships

### 16. Map Internal Links to Sections
- Assign each internal link to a specific section in the article
- Present as a table: Article | URL Path | Target Section
- Target 2-5 internal links per 1000 words

### 17. Update External Links
- Link to authoritative, recent sources (last 2 years preferred)
- Target 1-3 quality external links
- Prefer .gov, .edu, or recognized industry sources
- Remove or replace broken/outdated external links

## Phase 6: Structured Data & FAQ

### 18. Add/Update FAQ Section
- Research "People Also Ask" via web search
- Use exact PAA phrasing for questions when possible
- Answer questions concisely (40-60 words ideal for featured snippets)
- Include target keyword naturally in 1-2 answers
- Limit to 5-7 FAQs per article
- Structure: H3 for each question, paragraph for each answer

### 19. Add/Update Schema Markup
- Article schema (required)
- FAQ schema (if FAQ section exists)
- HowTo schema (if step-by-step content)
- Breadcrumb schema (recommended)
- Combine using @graph pattern
- Reference `schema-templates.md` for JSON-LD templates

## Phase 7: Verify

### 20. Present Before/After Diff
- Show field-by-field comparison table
- List all content changes: sections added/modified, links added, FAQs
- Include character counts for meta title and description

### 21. Freshness Signals Final Check
- [ ] `dateModified` in schema markup
- [ ] Year in meta title
- [ ] Year in H1 or first H2
- [ ] Updated statistics and data points
- [ ] Recent external source links (< 2 years old)
- [ ] "Updated [Month Year]" visible on page (if field exists)
- [ ] Keyword/tag fields reflect current terminology

## Phase 8: Publish & Submit

### 22. Publish
- Confirm with user before publishing
- Use `publish_collection_items` to publish

### 23. Submit to Google Search Console
- With GSC MCP: request indexing for the updated URL
- Without GSC: inform user to manually submit URL

## Phase 9: Post-Publish Monitoring

### 24. Provide Monitoring Checklist
- [ ] Verify page is indexed: `site:[URL]` in Google (after 48 hours)
- [ ] Check for crawl errors in GSC
- [ ] Monitor position changes for target keywords (7-14 days)
- [ ] Check if FAQ rich results appear in SERP
- [ ] Verify all internal links resolve correctly
- [ ] Review CTR changes (14-30 days)
- [ ] Compare organic traffic (30 days vs. prior 30 days)
