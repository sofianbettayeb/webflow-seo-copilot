---
name: aeo-optimize
version: "1.1"
description: |
  Transform any page into an AEO-ready answer. Audits 12 AEO dimensions, proposes a full rewrite, gets approval, then publishes to Webflow.
  Triggers: aeo optimize, aeo optimization, optimize for ai, answer engine optimization, aeo audit, optimize for chatgpt.
  Requires: Webflow MCP. Optional: GSC MCP for keyword discovery.
  Modes: /aeo-optimize (full workflow), /aeo-optimize:audit (score only).
---

# AEO Optimize

Transform any page into an AEO-ready answer. Audits 12 dimensions, proposes targeted rewrites, gets one approval, then publishes.

**Modes:**
- `/aeo-optimize` — full workflow: audit → propose → approve → publish
- `/aeo-optimize:audit` — score only, no changes proposed

See CLAUDE.md for standard MCP discovery, config loading, and abort guard.

**MCP requirements:** Webflow MCP (required). GSC MCP (optional — if unavailable, ask user for the primary keyword).

---

## Phase 0 — Setup

### 0.1 MCP check & config
Follow CLAUDE.md standard pattern. Load config — use `business.name` and author info for schema generation (Phase 3.9).

⚡ GUARD — **Webflow MCP unavailable:** Stop. "AEO Optimize requires Webflow MCP to read and update page content."

⚡ GUARD — **GSC unavailable:** Note it, continue. Ask user for the primary keyword in Phase 1.2 if needed.

### 0.2 Page input
Ask the user for the target page:
- A full URL (e.g. `https://www.checklist-seo.com/implementation/disable-webflow-subdomain-indexing`)
- Or a slug (e.g. `disable-webflow-subdomain-indexing`)

### 0.3 Page type detection
Determine whether the target is:
- **CMS item** — URL matches a collection template path (e.g. `/implementation/`, `/go-live/`, `/aio-webflow/`, `/content/`, `/webflow-seo/`)
- **Static page** — all other pages

Store `page_type = "cms"` or `page_type = "static"`.

### 0.4 Activity log check
Read `.claude/reports/{domain}/activity-log.md`. Check the last 10 entries for:
- Recent work on this specific page (avoid re-doing changes < 30 days old)
- Prior `/aeo-optimize` runs on this page
- Recent `/refresh-content` or `/click-recovery` runs that affected this page

---

## Phase 1 — Page & Query Discovery

### 1.1 Fetch GSC queries for this page
Call GSC with `dimensions: ["query"]`, filtered to this specific page URL, 90-day window.

```
startDate: 90 days ago
endDate: today
dimensions: ["query"]
page: {full page URL}
rowLimit: 50
```

If GSC returns < 5 rows or 0 impressions, expand to 180 days. If still empty, ask the user to provide the primary keyword manually.

### 1.2 Identify primary and secondary queries
- **Primary query:** highest impression count among clean queries (apply anomaly filter: strip `^\d+[:\,]`, `%[0-9A-Fa-f]{2}`, JSON fragments)
- **Secondary queries:** top 3 by impressions after the primary
- Store: `primary_query`, `secondary_queries[]`, `primary_position`, `primary_impressions`

Show the user:
```
Primary query: "{primary_query}" — {impressions} impressions, position {position}
Secondary: {q2}, {q3}, {q4}
```

Ask: "Is this the right keyword to optimize for? Or would you like to target a different one?"

Wait for confirmation before proceeding.

### 1.3 Fetch page content from Webflow

**For CMS items:**
- Identify which collection the item belongs to (match URL path to known collection slugs)
- Call `collections_items_list_items` with `slug: {page_slug}` to get the item
- Extract: `name` (H1 proxy), `introduction` or `post-body` (body content), `seo-title`, `meta-description` or `post-summary`, `slug`
- Check for a `schema-markup` field (or `schema`, `json-ld`): `schema_field_exists = true/false`

**For static pages:**
- Call `pages_list` to find the page by slug
- Call `pages_get_content` to retrieve text nodes
- Call `pages_get_metadata` for SEO title and description
- `schema_field_exists = false` (static pages require manual head code injection)

### 1.4 Extract content structure
Parse the body content to identify:
- **H1** — first heading or `name` field
- **H2 list** — all second-level headings
- **Intro** — first 100 words of body
- **Paragraphs** — count, average word count per paragraph
- **Bullet lists** — count
- **Tables** — count
- **Steps / numbered lists** — count
- **FAQ section** — detect by H2/H3 containing "FAQ", "Frequently Asked", "Questions"
- **Summary section** — detect by H2 containing "Summary", "Conclusion", "Key Takeaways"
- **Internal links** — extract all `<a href>` pointing to same domain
- **Existing schema** — detect `<script type="application/ld+json">` in content

---

## Phase 2 — AEO Audit

Score each dimension. Output a table showing pass ✅ / partial ⚠️ / fail ❌ with a one-line note.

| # | Dimension | Check | Weight |
|---|-----------|-------|--------|
| 1 | Query alignment | Primary query in H1 AND in first 100 words AND in SEO title | High |
| 2 | Direct answer (≤100 words) | Body opens with a self-contained answer to the primary query | High |
| 3 | Question-based sections | ≥60% of H2s phrased as questions | Medium |
| 4 | Bullet lists | Dense paragraphs (>4 lines) replaced with bullets where content permits | Medium |
| 5 | Tables | Comparison or multi-attribute content formatted as tables (note: tables require manual insertion via Designer — cannot be pushed via API) | Medium |
| 6 | Step-by-step sections | Procedural content uses numbered lists or Step format | Medium |
| 7 | Definition sentences | Key concepts have explicit "X is Y" or "X refers to Y" definitions | Medium |
| 8 | Short paragraphs | No paragraph exceeds 4 sentences; avg ≤ 2.5 sentences | Medium |
| 9 | FAQ section | FAQ exists with ≥ 5 questions relevant to the primary query cluster | High |
| 10 | Schema markup | FAQPage + Article JSON-LD present (HowTo if steps exist) | High |
| 11 | Internal links | 3–5 relevant internal links with descriptive anchor text | Medium |
| 12 | Summary section | Page ends with a summary, conclusion, or key takeaways block | Low |

**Scoring:**
- ✅ Pass = 2 pts
- ⚠️ Partial = 1 pt
- ❌ Fail = 0 pts

Total = `/24`. Express as percentage and maturity level:
- 20–24 (83–100%) — **AEO Ready**
- 14–19 (58–83%) — **Partially Optimized**
- 8–13 (33–58%) — **Needs Work**
- 0–7 (0–33%) — **Not AEO Ready**

In `/aeo-optimize:audit` mode: output the audit table + score + top 3 gaps ranked by impact. Stop here.

In `/aeo-optimize` mode: proceed to Phase 3.

---

## Phase 3 — Proposal Generation

Generate the full rewrite proposal. Each element below must be written, not just described.

Apply humanizer rules to all generated content before presenting:
- No AI vocabulary (seamless, robust, delve, comprehensive, leverage, etc.)
- No em-dash overuse
- No rule-of-three padding
- No inflated symbolism
- Short, direct sentences

### 3.1 Rewritten intro (first 100 words)
Write a new opening paragraph that:
- Directly answers the primary query in the first 2 sentences
- States what the page covers
- Uses the primary query naturally in sentence 1 or 2
- Does not exceed 100 words
- Reads like it was written by a practitioner, not an AI

### 3.2 H1 update (if needed)
If the current H1 doesn't contain the primary query, propose a new H1:
- Contains the primary query
- Under 60 characters
- Framed as a statement or question (not keyword-stuffed)

### 3.3 H2 restructuring
For each existing H2, propose a question-phrased version where the content supports it.

Format:
```
Current: "How to Implement HTTPS"
Proposed: "How Do You Implement HTTPS on a Webflow Site?"
```

Don't force every H2 into a question — only where it makes sense. Keep section meaning intact.

### 3.4 Content reformatting
Identify the 2–3 densest paragraphs. Propose bullet or table replacements.

For each:
```
CURRENT (dense paragraph):
[paste the paragraph]

PROPOSED (reformatted):
[bullet list or table]
```

For procedural content, propose numbered step format. Each step: one sentence + one optional detail sentence.

**When a table is the right format for comparison or multi-attribute content:**

Webflow's CMS API strips `<table>` tags — tables cannot be pushed via the API or MCP. When a table is warranted:

1. Propose a per-item structured list as the body content (what gets saved via API):
```html
<p><strong>Option Name</strong></p>
<ul>
  <li><strong>Attribute 1:</strong> Value</li>
  <li><strong>Attribute 2:</strong> Value</li>
</ul>
```

2. Also generate the full HTML table with inline `style` attributes only (no class names) and present it with this note:

> ⚠️ **Manual step required:** In Webflow Designer, open this page, place an HTML Embed element where the table should appear, and paste this HTML. Replace the structured list with the embed once it's in place.

Table styling must use inline styles only. Vary the visual treatment per article — do not reuse a fixed template.

### 3.5 Definition sentences
For each technical term or concept in the primary keyword cluster, propose a one-sentence definition to insert at first use:
```
Concept: "subdomain indexing"
Definition: "Subdomain indexing is when Google crawls and indexes your webflow.io staging URL alongside your live domain — treating them as separate, competing sites."
```

### 3.6 FAQ section
Write a complete FAQ section with 5–7 questions. Rules:
- Questions must match real GSC queries or natural query variations
- Each answer: 2–4 sentences, self-contained
- Ordered: most important first
- First question must directly address the primary query
- Avoid repeating content already in the body verbatim

Format:
```
## Frequently Asked Questions

### {Question 1}
{Answer 1}

### {Question 2}
{Answer 2}
...
```

### 3.7 Summary section
Write a 3–5 bullet summary section placed at the end of the page:
```
## Summary

- {Key point 1}
- {Key point 2}
- {Key point 3}
- {Key point 4 if needed}
- {Key point 5 if needed}
```

Each bullet: one sentence, action or insight-oriented.

### 3.8 Internal links
Identify 3–5 pages on the site that are topically relevant to this content. For each:
- **Anchor text:** descriptive, not "click here" or "this page"
- **Target:** same domain only, scoped to same section/category where possible
- **Placement:** specific sentence in the body where the link fits naturally

Format for each:
```
Link {n}:
Anchor: "{anchor text}"
Target: {/path/to/page}
Placement: Insert in paragraph starting with "..."
```

To find relevant pages, use the Webflow pages list + known collection slugs. Match by topic overlap with the primary query.

**Scope rules:**
- Maximum 5 new links
- Do not duplicate existing internal links already in the content
- Do not link to the same URL twice
- Prefer pillar pages and closely related checklist items

### 3.9 Schema markup (JSON-LD)
Generate the complete schema block.

**Always include:** `Article`
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{H1}",
  "description": "{meta description}",
  "author": {
    "@type": "Person",
    "name": "{author name from config or ask user}",
    "url": "{author URL from config}"
  },
  "publisher": {
    "@type": "Organization",
    "name": "{business.name from config}",
    "url": "https://{domain}"
  },
  "datePublished": "{original publish date or today}",
  "dateModified": "{today}"
}
```

**Include `FAQPage` when FAQ is added:**
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "{Question 1}",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "{Answer 1}"
      }
    }
    // ... repeat for each FAQ question
  ]
}
```

**Include `HowTo` when numbered steps are added:**
```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "{H1}",
  "step": [
    {
      "@type": "HowToStep",
      "name": "{Step 1 heading}",
      "text": "{Step 1 description}"
    }
  ]
}
```

Output as a single `<script type="application/ld+json">` block combining all applicable types.

### 3.10 SEO title and meta description update (if needed)
If the primary query isn't in the current SEO title:
- Propose a new SEO title (50–60 chars, query-first)
- Propose a new meta description (140–155 chars, includes primary query + clear benefit)

---

## Phase 4 — Approval Gate

Present the full proposal to the user in a structured summary:

```
## AEO Optimization Proposal — {page slug}
Primary query: "{primary_query}"
Current AEO score: {score}/24 ({level})
Post-optimization estimated score: {estimated}/24

### Changes proposed:
1. ✏️  Intro rewrite (first 100 words)
2. 🏷️  H1 update: "{new H1}"
3. 📋  {N} H2s restructured as questions
4. 📝  {N} content blocks reformatted (bullets/tables/steps)
5. 💬  {N} definition sentences added
6. ❓  FAQ section (7 questions)
7. 📌  Summary section (4 bullets)
8. 🔗  {N} internal links added
9. 🧩  Schema markup (Article + FAQPage{+ HowTo})
10. 🏷️  SEO title + meta description updated

[Full proposal follows below]
```

Then show the complete content for each section (3.1–3.10).

Ask: "Do you want to proceed with all changes? Or adjust specific sections before I publish?"

**If user approves all:** proceed to Phase 5.
**If user wants edits:** incorporate feedback, re-present the affected sections, confirm again.
**If user cancels:** stop. Do not modify anything.

---

## Phase 5 — Execute

### 5.1 Merge content
Take the original page body and apply all approved changes:
- Replace intro with 3.1
- Update H1 with 3.2 (if changed)
- Replace H2 text with 3.3 versions
- Replace/reformat dense paragraphs with 3.4 content
- Insert definition sentences at first use per 3.5
- Append FAQ section (3.6) before Summary
- Append Summary section (3.7) at end
- Insert internal links per 3.8 placements
- Do not remove or rearrange content that wasn't flagged for change

### 5.2 Publish content

**For CMS items:**
- Call `collections_items_update_items_live` with:
  - Updated `introduction` / `post-body` (merged content)
  - Updated `seo-title` (if changed)
  - Updated `meta-description` or `post-summary` (if changed)
  - Updated `schema-markup` field with JSON-LD (if field exists)
- If `schema_field_exists = false`: output the `<script>` block and instruct user to add it in Webflow Designer → Page Settings → Custom Code → Before `</body>` on the collection template, or bind a new Plain Text field.

**For static pages:**
- Call `pages_update_static_content` to update text nodes (intro, H1, H2s, paragraph blocks)
- Call `pages_update_page_settings` to update SEO title and meta description
- Output the `<script>` block for schema and instruct: "Add this to Webflow Designer → Page Settings → Custom Code → Head"
- Call `sites_publish` to publish

**Rate limit handling:**
- If `429` on publish: wait and retry once. If it fails again, confirm to the user that all content changes are saved — publish manually from Webflow Designer.

---

## Phase 6 — Report & Log

### 6.1 Output completion summary
```
## ✅ AEO Optimization Complete — {page slug}

Primary query: "{primary_query}"
AEO score: {before}/24 → {after}/24 ({level_before} → {level_after})

Changes applied:
- ✏️  Intro rewritten (direct answer in first 100 words)
- 🏷️  H1 updated: "{new H1}"
- 📋  {N} H2s converted to questions
- 📝  {N} content blocks reformatted
- 💬  {N} definitions added
- ❓  FAQ added ({N} questions)
- 📌  Summary section added
- 🔗  {N} internal links inserted
- 🧩  Schema: Article + FAQPage{+ HowTo}
- 🏷️  SEO title + meta updated

{schema_note if schema_field_exists = false}
Published: {domain}
```

### 6.2 Save report
Save to `.claude/reports/{domain}/aeo-optimize-{slug}-{YYYY-MM-DD}.md`

Include: audit scores (before/after per dimension), full proposal as reference, schema block, internal links added.

### 6.3 Activity log
Append to `.claude/reports/{domain}/activity-log.md`:
```
| {date} | /aeo-optimize | AEO optimization on /{slug}. Query: "{primary_query}". Score: {before}→{after}/24. FAQ added ({N}q), schema Article+FAQPage{+HowTo}, {N} internal links. Published. |
```

---

## Error handling

| Error | Action |
|-------|--------|
| GSC returns no data for page | Ask user to provide primary keyword manually |
| CMS item not found by slug | Ask user to confirm the slug or provide the collection name |
| Schema field missing from CMS | Output schema block, instruct manual paste in Designer. Suggest running `/cms-collection-setup:review` to add the field |
| Rich text body field not identifiable | Ask user to confirm which field holds the main content |
| Page content too long for single update | Split into intro pass + body pass, update in two API calls |
| 429 rate limit on publish | Retry once after 30s. If still failing, instruct manual publish from Designer |
| Static page text node IDs not found | Fall back to presenting changes as a formatted diff for manual paste |

---

## Integration with other skills

| Situation | Recommended next step |
|-----------|----------------------|
| Page has outdated facts or stale content | Run `/refresh-content` first (actualizes content), then `/aeo-optimize` |
| Page has CTR problem after AEO | Run `/click-recovery` — AEO changes may shift the optimal title |
| Collection missing schema field | Run `/cms-collection-setup:review` first |
| Want to find which pages to AEO-optimize | Run `/keywords-opportunity` — pages with striking distance queries are best candidates |
| Site-wide AEO audit | Run `/audit-deep` to identify all pages below AEO threshold |
