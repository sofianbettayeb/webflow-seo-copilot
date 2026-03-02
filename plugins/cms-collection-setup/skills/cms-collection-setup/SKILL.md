---
name: cms-collection-setup
version: "1.0"
description: |
  Review or create a Webflow CMS collection optimized for SEO. Audits existing article/blog collections against a recommended field schema, or builds a new collection from scratch with all Core, Content, Authority, Enhancement, and Social fields.
  Triggers: cms collection, collection setup, review collection, create collection, blog collection, article collection, cms schema, seo schema, cms audit.
  Requires: Webflow MCP server.
  Workflow: Choose mode → Configure → Review or Build → Confirm.
  Modes: /cms-collection-setup (full workflow), /cms-collection-setup:review (audit only), /cms-collection-setup:create (build only).
---

# CMS Collection Setup Skill

Review an existing Webflow CMS collection against the recommended SEO schema — or create a new one from scratch with all the fields that matter for rankings, authority, and sharing.

## Why the Schema Matters

Most Webflow blogs are missing fields that unlock real SEO leverage: a dedicated H1 override, OG image, primary keyword, pillar page reference, structured FAQ. Without these, `/refresh-content` can't populate them, `/click-recovery` can't read meta fields, and schema markup has no source data to pull from.

This skill fixes the foundation.

---

## Recommended Schema

The target schema covers five groups. Every field has a purpose — nothing decorative.

### Core
Fields that control how the article is found and indexed.

| Field | Type | Notes |
|-------|------|-------|
| Name | PlainText | Built-in — every Webflow collection has this |
| Slug | PlainText | Built-in — every Webflow collection has this |
| H1 override | PlainText | Decouples the page H1 from the CMS item name |
| Meta SEO title | PlainText | Shown in SERP. ≤60 chars. |
| Meta SEO description | PlainText | Shown in SERP. ≤155 chars. |

### Content
Fields that control what appears on the page.

| Field | Type | Notes |
|-------|------|-------|
| Main content | RichText | Body of the article |
| Main visual | Image | Hero/featured image |
| Alt text | PlainText | Alt text for the main visual (for CMS-bound images) |
| Reading time | Number | In minutes. Can be calculated or manually set. |
| Updated date | DateTime | Shown to readers and used in schema markup |

### Authority
Fields that build trust signals and topical structure.

| Field | Type | Notes |
|-------|------|-------|
| Author | Reference or MultiReference | Links to an Authors collection |
| Category | Reference | Links to a Categories collection |
| Tags | Option or MultiReference | Predefined option list, or links to a Tags collection |
| Pillar page | Reference | Links to the main pillar article this post supports |

### Enhancement
Fields that power SEO tooling and internal linking.

| Field | Type | Notes |
|-------|------|-------|
| Primary keyword | PlainText | The one keyword this article targets |
| Secondary keywords | PlainText | Comma-separated. Supports `/refresh-content` keyword pass. |
| Related posts | MultiReference | Links to other items in the same collection |
| FAQ | MultiReference or RichText | Structured FAQ items (multi-ref) or inline rich text |

### Social
Fields that control how the article looks when shared.

| Field | Type | Notes |
|-------|------|-------|
| OG title | PlainText | Open Graph title. Can differ from meta title. |
| OG description | PlainText | Open Graph description. ≤200 chars. |
| OG image | Image | Recommended: 1200×630px. Aspect ratio 1.91:1. |

---

## Prerequisites

- **Required**: [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview)

Check MCP availability before starting. No GSC needed for this skill.

---

## Skill Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/cms-collection-setup` | Guided: choose review or create at runtime |
| **Review** | `/cms-collection-setup:review` | Audit an existing collection against the recommended schema |
| **Create** | `/cms-collection-setup:create` | Build a new collection with the full recommended schema |

---

## Conditional Guards (Global)

⚡ GUARD — **Load SEO Copilot config:**
At the start of execution, check if `.claude/seo-copilot-config.json` exists:
- If yes: Load settings — `business.webflowSiteUrl` helps identify the site
- If no: Proceed silently. No config is needed to run this skill.

⚡ GUARD — **Webflow MCP unavailable:**
If Webflow MCP is not connected:
- Stop and inform the user: "This skill requires the Webflow MCP server. Connect it and try again: https://developers.webflow.com/mcp/reference/overview"

⚡ GUARD — **User requests abort:**
If user says "stop", "cancel", "abort", or "nevermind" at any phase:
- Confirm: "Stop the workflow?"
- If confirmed: Exit cleanly. In create mode, note which fields were already created.

---

## Phase 0: Mode Selection

### 0.1 Choose Mode (full mode only)

If running `/cms-collection-setup` (no sub-mode):

```
What would you like to do?

1. Review an existing collection — audit it against the recommended SEO schema
2. Create a new collection — build one from scratch with all the right fields
```

- If **Review**: Jump to Phase 1R
- If **Create**: Jump to Phase 1C

---

## Phase 1R: Review — Select Collection

### 1R.1 Identify the Site

Use Webflow MCP `sites_list` to list available sites.

If multiple sites: ask the user to select one.

Set `{site_id}` and `{domain}` from the selected site.

### 1R.2 List Collections

Use `collections_list` to fetch all collections on the site.

Present them as a numbered list:

```
Collections on {domain}:

1. Blog Posts (slug: blog-posts, items: 47)
2. Team Members (slug: team-members, items: 12)
3. Case Studies (slug: case-studies, items: 8)
...

Which collection would you like to audit?
```

### 1R.3 Fetch Collection Schema

Use `collections_get` with the selected collection ID to fetch the full field list including field types and slugs.

---

## Phase 2R: Review — Audit Against Schema

### 2R.1 Map Existing Fields

For each field in the recommended schema, attempt to find a match in the existing collection.

**Matching rules (in order of confidence):**

1. **Exact slug match** — field slug matches recommended slug exactly (e.g., `meta-seo-title`)
2. **Alias match** — common alternative slugs for the same purpose:
   - H1 override: `h1`, `h1-override`, `heading`, `page-title`
   - Meta SEO title: `seo-title`, `meta-title`, `og-title-seo`, `titile`
   - Meta SEO description: `seo-description`, `meta-description`, `post-summary`, `excerpt`
   - Main content: `body`, `content`, `post-body`, `article-body`, `rich-text`
   - Main visual: `featured-image`, `hero-image`, `thumbnail`, `cover-image`
   - Alt text: `image-alt`, `featured-image-alt`, `alt`, `hero-alt`
   - Reading time: `read-time`, `time-to-read`, `minutes`
   - Updated date: `last-updated`, `date-updated`, `modified-date`, `published-date`
   - Author: `authors`, `author-ref`, `written-by`
   - Category: `categories`, `category-ref`, `topic`
   - Tags: `tag`, `tag-list`, `labels`
   - Pillar page: `pillar`, `pillar-article`, `parent-post`
   - Primary keyword: `keyword`, `target-keyword`, `focus-keyword`
   - Secondary keywords: `keywords`, `secondary-keyword`, `lsi-keywords`
   - Related posts: `related`, `related-articles`, `see-also`
   - FAQ: `faqs`, `faq-section`, `questions`
   - OG title: `og-title`, `social-title`, `open-graph-title`
   - OG description: `og-description`, `social-description`, `open-graph-description`
   - OG image: `og-image`, `social-image`, `open-graph-image`
3. **Type-based inference** — if no slug match, check for fields of the correct type that are unassigned to another schema slot
4. **No match** — field is missing

### 2R.2 Check Type Compatibility

For each matched field, verify the field type is correct or compatible:

| Recommended Type | Compatible Types | Incompatible |
|-----------------|-----------------|--------------|
| PlainText | PlainText | RichText, Number, Image |
| RichText | RichText | PlainText |
| Image | Image | MultiImage |
| Number | Number | — |
| DateTime | DateTime | — |
| Reference | Reference | MultiReference |
| MultiReference | MultiReference | Reference |
| Option | Option | MultiReference |

Flag type mismatches as ⚠️ (field exists but wrong type — cannot be auto-fixed).

### 2R.3 Score Coverage

Calculate a coverage score for each group:

```
Group score = (exact matches + 0.5 × alias matches) / total fields in group × 100
```

Also compute an overall weighted score:

| Group | Weight |
|-------|--------|
| Core | 30% |
| Content | 25% |
| Authority | 20% |
| Enhancement | 15% |
| Social | 10% |

### 2R.4 Output Audit Report

Present a structured report:

```
# CMS Schema Audit — {Collection Name}

**Site**: {domain}
**Collection**: {collection display name} ({item count} items)
**Overall SEO Schema Score**: [X]% ([rating])

Rating scale:
- 90–100% → Excellent — fully optimized
- 70–89%  → Good — minor gaps
- 50–69%  → Needs work — key fields missing
- <50%    → Foundation gaps — significant SEO risk

---

## Core Fields (30% weight) — Score: X%

| Recommended Field | Status | Mapped To | Type OK? |
|-------------------|--------|-----------|----------|
| Name | ✅ Exact | name | ✅ PlainText |
| Slug | ✅ Exact | slug | ✅ PlainText |
| H1 override | ❌ Missing | — | — |
| Meta SEO title | ⚠️ Alias | seo-title | ✅ PlainText |
| Meta SEO description | ✅ Exact | meta-seo-description | ✅ PlainText |

## Content Fields (25% weight) — Score: X%

[same table format]

## Authority Fields (20% weight) — Score: X%

[same table format — for reference fields, note target collection]

## Enhancement Fields (15% weight) — Score: X%

[same table format]

## Social Fields (10% weight) — Score: X%

[same table format]

---

## What to Add

Fields missing from this collection, listed by priority:

### Must Add (Core)
- **H1 override** — Type: PlainText — Lets you decouple the page H1 from the CMS item name for keyword optimization
- **Meta SEO title** — Type: PlainText — Required for SERP optimization. Currently using `name` as fallback — creates keyword mismatches.

### Should Add (Content / Authority)
- **Reading time** — Type: Number — Adds trust signal. Populate manually or compute from word count.
- **Pillar page** — Type: Reference → [suggest existing collections] — Powers internal linking strategy.

### Nice to Add (Enhancement / Social)
- **Primary keyword** — Type: PlainText — Enables `/refresh-content` to target keywords per article.
- **OG image** — Type: Image (1200×630px) — Controls social sharing appearance. Without it, platforms grab any image from the page.

---

## Existing Fields Not in Schema

Fields in this collection that don't map to the recommended schema:

| Field | Slug | Type | Suggestion |
|-------|------|------|------------|
| Published date | published-date | DateTime | Can double as "Updated date" |
| Featured | featured | Switch | Keep — not in schema but useful |

---

## Next Steps

1. Add missing fields directly: run `/cms-collection-setup:create` and select "Add missing fields to existing collection"
2. Or add them manually in Webflow Designer
3. Once fields exist, run `/refresh-content [URL]` to populate them for an article
```

### 2R.5 Offer to Add Missing Fields

After the report, ask:

```
Would you like me to add the missing fields to this collection now?

1. Yes — add all missing fields automatically
2. Yes — let me choose which fields to add
3. No — I'll add them manually
```

If option 1 or 2: jump to Phase 3C (field creation), skipping collection creation. Pass the existing collection ID.

---

## Phase 1C: Create — Gather Configuration

### 1C.1 Identify the Site

Use `sites_list` to list available sites.

If multiple sites: ask the user to select one.

Set `{site_id}` and `{domain}`.

### 1C.2 Collection Identity

Ask:

```
Let's set up your new CMS collection.

1. **Display name**: What's this collection called?
   Example: "Blog Posts", "Articles", "Insights"

2. **Slug**: URL path for this collection (auto-suggested from display name — confirm or change)
   Example: "blog-posts", "articles"

3. **Singular name**: Singular form of the display name (used in Webflow UI)
   Example: "Blog Post", "Article"
```

### 1C.3 Configure Flexible Fields

The schema has fields where you choose the structure. Ask these questions upfront before creating anything.

```
A few decisions before I build the collection:

**Author field**
How do you manage authors?
1. Single author — Reference to an Authors collection
2. Multiple authors — MultiReference to an Authors collection
3. No author field — skip this field

[If 1 or 2]: Which collection holds your authors? (list existing collections, or "I'll create one later")

---

**Tags field**
How do you handle tags?
1. Fixed list — Option field (define the tag options now)
2. Flexible — MultiReference to a Tags collection
3. No tags — skip this field

[If 1]: What tags should I create as options? (comma-separated, e.g. "SEO, Content, Design, Dev")
[If 2]: Which collection holds your tags? (list existing collections, or "I'll create one later")

---

**Category field**
1. Yes — Reference to a Categories collection
2. No category — skip this field

[If 1]: Which collection holds your categories? (list existing collections, or "I'll create one later")

---

**Pillar page field**
A pillar page reference links this article to the main topic page it supports — useful for internal linking strategy.
1. Yes — Reference to another collection (e.g., a "Pillar Pages" or "Topics" collection)
2. Yes — Self-reference (link to another article in this same collection)
3. No — skip this field

[If 1]: Which collection? (list existing collections, or "I'll create one later")

---

**FAQ field**
How do you handle FAQs in articles?
1. Separate collection — MultiReference to an FAQ collection (structured, reusable)
2. Inline — RichText field in this collection (simpler, embedded in article)
3. No FAQ — skip this field

[If 1]: Which collection holds your FAQ items? (list existing collections, or "I'll create one later")
```

### 1C.4 Confirm Before Building

Present a summary of the configuration:

```
## Collection Plan

**Name**: {display name}
**Slug**: {slug}
**Singular**: {singular name}

| Group | Field | Type | Notes |
|-------|-------|------|-------|
| Core | Name | PlainText | Built-in |
| Core | Slug | PlainText | Built-in |
| Core | H1 override | PlainText | |
| Core | Meta SEO title | PlainText | |
| Core | Meta SEO description | PlainText | |
| Content | Main content | RichText | |
| Content | Main visual | Image | |
| Content | Alt text | PlainText | |
| Content | Reading time | Number | In minutes |
| Content | Updated date | DateTime | |
| Authority | Author | {Reference/MultiReference} | → {collection name or "skip"} |
| Authority | Category | Reference | → {collection name or "skip"} |
| Authority | Tags | {Option/MultiReference} | {options list or collection} |
| Authority | Pillar page | Reference | → {collection name or "skip"} |
| Enhancement | Primary keyword | PlainText | |
| Enhancement | Secondary keywords | PlainText | |
| Enhancement | Related posts | MultiReference | → self |
| Enhancement | FAQ | {MultiReference/RichText} | {collection or inline} |
| Social | OG title | PlainText | |
| Social | OG description | PlainText | |
| Social | OG image | Image | 1200×630px recommended |

**Fields to create**: {count} (Name and Slug are built-in — skipping)

Proceed?
```

**Wait for explicit user confirmation before creating anything.**

---

## Phase 2C: Create — Build the Collection

### 2C.1 Create the Collection

Use `collections_create` with:
- `displayName`: from 1C.2
- `singularName`: from 1C.2
- `slug`: from 1C.2

⚡ GUARD — **Slug conflict:**
If the API returns an error indicating the slug already exists:
- Suggest alternatives: `{slug}-2`, `{slug}-articles`, etc.
- Ask user to confirm the new slug before retrying

Store the returned `collection_id` — needed for all field creation and self-references.

### 2C.2 Create Fields (in order)

Create fields in the exact order below. After each field creation, log progress:

```
✅ H1 override (PlainText)
✅ Meta SEO title (PlainText)
⏳ Meta SEO description...
```

**Field creation order and specs:**

**Core:**

| Field | API type | displayName | slug suggestion |
|-------|----------|-------------|-----------------|
| H1 override | PlainText | H1 Override | h1-override |
| Meta SEO title | PlainText | Meta SEO Title | meta-seo-title |
| Meta SEO description | PlainText | Meta SEO Description | meta-seo-description |

**Content:**

| Field | API type | displayName | slug suggestion | Notes |
|-------|----------|-------------|-----------------|-------|
| Main content | RichText | Main Content | main-content | |
| Main visual | Image | Main Visual | main-visual | |
| Alt text | PlainText | Alt Text | alt-text | |
| Reading time | Number | Reading Time | reading-time | Help text: "In minutes" |
| Updated date | DateTime | Updated Date | updated-date | |

**Authority:**

| Field | API type | displayName | slug suggestion | Condition |
|-------|----------|-------------|-----------------|-----------|
| Author | Reference or MultiReference | Author | author | If chosen — use target collection ID from 1C.3 |
| Category | Reference | Category | category | If chosen — use target collection ID |
| Tags | Option or MultiReference | Tags | tags | If Option: use defined options; if MultiRef: use target collection ID |
| Pillar page | Reference | Pillar Page | pillar-page | If chosen — use target collection ID |

⚡ GUARD — **Referenced collection is "I'll create one later":**
If a reference target wasn't set because the user will create the collection later:
- Skip the Reference/MultiReference field for now
- Log it in the summary as "⚠️ Skipped — add manually after creating the target collection"
- Do not error out

**Enhancement:**

| Field | API type | displayName | slug suggestion | Notes |
|-------|----------|-------------|-----------------|-------|
| Primary keyword | PlainText | Primary Keyword | primary-keyword | |
| Secondary keywords | PlainText | Secondary Keywords | secondary-keywords | Help text: "Comma-separated" |
| Related posts | MultiReference | Related Posts | related-posts | Self-reference — use this collection's ID |
| FAQ | MultiReference or RichText | FAQ | faq | Per 1C.3 choice |

**Social:**

| Field | API type | displayName | slug suggestion | Notes |
|-------|----------|-------------|-----------------|-------|
| OG title | PlainText | OG Title | og-title | |
| OG description | PlainText | OG Description | og-description | |
| OG image | Image | OG Image | og-image | Help text: "Recommended: 1200×630px (1.91:1)" |

---

## Phase 3: Summary

### 3.1 Present Results

```
## Collection Created — {Collection Name}

**Site**: {domain}
**Collection ID**: {id}
**Fields created**: {X} of {Y} planned

### Field Summary

| Group | Field | Status | Webflow Slug |
|-------|-------|--------|--------------|
| Core | Name | ✅ Built-in | name |
| Core | Slug | ✅ Built-in | slug |
| Core | H1 override | ✅ Created | h1-override |
| Core | Meta SEO title | ✅ Created | meta-seo-title |
| Core | Meta SEO description | ✅ Created | meta-seo-description |
| Content | Main content | ✅ Created | main-content |
| Content | Main visual | ✅ Created | main-visual |
| Content | Alt text | ✅ Created | alt-text |
| Content | Reading time | ✅ Created | reading-time |
| Content | Updated date | ✅ Created | updated-date |
| Authority | Author | ✅ Created | author |
| Authority | Category | ✅ Created | category |
| Authority | Tags | ✅ Created | tags |
| Authority | Pillar page | ⚠️ Skipped | — Create target collection first |
| Enhancement | Primary keyword | ✅ Created | primary-keyword |
| Enhancement | Secondary keywords | ✅ Created | secondary-keywords |
| Enhancement | Related posts | ✅ Created | related-posts |
| Enhancement | FAQ | ✅ Created | faq |
| Social | OG title | ✅ Created | og-title |
| Social | OG description | ✅ Created | og-description |
| Social | OG image | ✅ Created | og-image |

### Skipped / Needs Follow-up

[Any skipped fields and why]

---

### What to Do Next

1. **In Webflow Designer**: Bind the new fields to your collection template page elements
2. **Pillar page field** (if skipped): Create the target collection, then add the Reference field manually or re-run this skill
3. **Populate content**: Use `/refresh-content [URL]` on existing articles to populate SEO fields
4. **OG image dimensions**: Add a custom dimension note to the Webflow image field help text: "1200×630px, 1.91:1 ratio"
5. **Schema markup**: Once fields are populated, `/refresh-content` will inject Article + FAQ JSON-LD schema automatically

### Skills That Read These Fields

| Skill | Fields Used |
|-------|-------------|
| `/refresh-content` | All fields — primary keyword, secondary keywords, meta titles, FAQ, updated date |
| `/click-recovery` | Meta SEO title, Meta SEO description |
| `/weekly-report` | Meta SEO title (CMS template audit) |
```

---

## Error Handling

| Error | Action |
|-------|--------|
| Webflow MCP not connected | Stop. Direct user to connect MCP. |
| Site not found | List available sites, ask user to select |
| Collection slug already exists | Suggest alternatives, ask user to confirm |
| Referenced collection ID not found | Warn user — the target collection may have been deleted or not created yet. Skip the Reference field and log it. |
| Self-reference fails (related posts) | Collection must exist before creating self-references. This field is always created last — if it fails, note it in the summary. |
| Option field with no options | Create the field with an empty option list. Webflow allows adding options later via Designer. |
| Field creation rate limit | Pause 2 seconds between fields. If rate limit persists, pause 10 seconds and retry once. |
| Unknown API error on field creation | Log the error, skip the field, continue with remaining fields. Flag in summary. |

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

**Determining `{domain}`**: Use the domain of the selected Webflow site.

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append:

**Review mode:**
```
| YYYY-MM-DD | /cms-collection-setup | Reviewed collection "{name}". Schema score: X%. Missing: [list key missing fields]. |
```

**Create mode:**
```
| YYYY-MM-DD | /cms-collection-setup | Created collection "{name}" with X fields across Core, Content, Authority, Enhancement, Social groups. |
```

Log even if execution was partial (e.g., "Created 15 of 18 fields — 3 skipped due to missing reference collections").
