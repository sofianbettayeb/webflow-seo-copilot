---
name: refresh-content
version: "2.0"
description: |
  Make an older post current again. Updates outdated facts, stats, tool references, and advice. Adds new sections for topics that have emerged since publication. Does not change the post's core topic or target keyword — that's write-blog's job.
  Triggers: refresh blog, update article, actualise content, update post, outdated content, content refresh, refresh my article, make this post current, update this article.
  Requires: Webflow MCP (to read and update content). Optional: GSC MCP for keyword data, Keywords Everywhere for related term discovery.
  Workflow: Read → Audit → Propose → Approve → Update → Publish.
---

# Refresh Content

Older posts go stale. Tools get renamed, stats become outdated, advice stops being valid, new developments make sections incomplete. This skill finds what's stale and proposes targeted updates — without changing the post's purpose or target keyword.

**What this skill does:** Updates facts, dates, tool references, statistics, and adds sections for topics that emerged after the post was published.

**What this skill does NOT do:** Change the target keyword, restructure the post's argument, or rewrite sections that are still accurate. For a new post, use `/write-blog`.

**Publishing target:** Webflow CMS items and static pages. If Webflow MCP is unavailable, outputs proposed changes as a diff for manual application.

---

## Prerequisites

- **Required:** Webflow MCP — to read current content and apply updates
- **Optional:** GSC MCP — to check if the post is still getting impressions (helps prioritize what to fix)
- **Optional:** Keywords Everywhere — to check if related terms have emerged worth adding

See CLAUDE.md for standard MCP discovery, config loading, and abort guard.

---

## Modes

| Mode | Command | Use Case |
|------|---------|----------|
| **Full** | `/refresh-content` | Read → audit → propose → single approval → update → publish |
| **Audit only** | `/refresh-content:audit` | Analysis and proposed changes only — nothing written to Webflow |

---

## Workflow

```
READ → AUDIT → PROPOSE → [ONE APPROVAL] → UPDATE → PUBLISH
```

There is **one approval gate** after PROPOSE. The user reviews all proposed changes at once, approves or adjusts, then execution runs without further interruption.

---

## Phase 0: Setup

Follow CLAUDE.md for: MCP discovery, config loading (use `brandVoice.tone`, `brandVoice.avoid`, `audience.expertiseLevel`), and abort guard.

**Get target article** — ask the user for a URL or article name if not provided:
- Parse URL to identify CMS collection + item slug, or static page
- Fetch from Webflow: full body content, title, meta title, meta description, publish date
- Store the publish date — this is the staleness baseline

**Activity log check:**
- If `/refresh-content` ran on this URL in the last 30 days → warn: "Refreshed on [date]. Run again?"
- If `/click-recovery` recently updated this page's meta title → note it to avoid overwriting

**GSC check (if available):**
Fetch last 90 days of queries for this page URL. Note impressions, primary query, position. If impressions < 10: "Low traffic — this post may have been demoted. A refresh helps, but check whether it's worth maintaining."

---

## Phase 1: Audit

Read the full post and check for staleness across four categories. For each issue found, record: location, what's stale, proposed fix.

### 1.1 Outdated facts and statistics

Flag:
- Statistics with a year (e.g., "In 2022, 43% of...") where the year is more than 18 months ago
- Claims about a tool's pricing, features, or market position
- "As of [date]" statements now outdated
- Past-tense predictions ("by 2025...") that can now be evaluated

Propose the updated fact, or mark "needs verification" for the user to fill in.

### 1.2 Tool and product references

Flag:
- Tools renamed, acquired, shut down, or significantly changed
- Pricing references that are likely outdated
- Step-by-step instructions tied to a UI that has changed
- Competitor comparisons using old positioning

### 1.3 Missing topics

Based on the post's publish date and topic, identify what has emerged since publication:
- New tools, methods, or standards in this topic area
- Platform updates (e.g., Webflow launched a feature, Google changed behavior)
- Developments that make the current coverage incomplete

If GSC data available: check for queries landing on this page that the post doesn't cover — those are explicit addition candidates.

If Keywords Everywhere available: run `get_related_keywords` on the primary keyword, flag high-volume terms the post doesn't address.

For each missing topic: propose a section title and 2–3 sentence summary of what it should cover. The user provides the actual expertise — this skill proposes the structure.

### 1.4 Outdated advice

Flag recommendations that are:
- No longer best practice
- Based on platform behavior that has changed
- Dependent on tools or services that no longer exist

### 1.5 Metadata

Check: does the meta title still reflect the primary keyword? Is the meta description still accurate? Flag if a year in the title needs updating (e.g., "Best Tools in 2023").

---

## Phase 2: Propose

Compile all findings into a single structured proposal. This is the only approval gate.

```
## Refresh Proposal — [Post Title]

**URL:** [url]
**Published:** [date] ([X months ago])
**GSC:** [X impressions / primary query: "[query]" / pos [X]] or [No GSC data]

### Outdated facts ([N])
1. [Section] — Current: "[quote]" → Proposed: "[update]"
2. [...]

### Tool references ([N])
1. Current: "[tool ref]" → Proposed: "[update or flag as needs verification]"

### Missing topics ([N])
1. Add section: "[Title]"
   What to cover: [2–3 sentences]
   Why: [e.g., "Webflow launched X after this post was published"]

### Outdated advice ([N])
1. [Section] — Current: "[excerpt]" → Proposed: "[fix or remove]"

### Metadata
| Field | Current | Proposed |
|-------|---------|----------|
| Meta title | [current] | [proposed] |
| Meta description | [current] | [proposed] |

**Total:** [N] facts · [N] tools · [N] new sections · [N] advice fixes · [N] metadata

Approve all, or tell me which to skip.
```

⚡ GUARD — **Nothing stale found:**
Output: "This post looks current — no outdated facts, tool changes, or missing topics found." Stop. Do not propose cosmetic changes.

⚡ GUARD — **Audit only mode:** Output the proposal and stop. Do not write to Webflow.

---

## Phase 3: Update

On approval, apply every approved change surgically:
- Edit only the relevant sentences/sections — do not rewrite surrounding content
- For new sections: insert after the most relevant existing section, or at the end
- For new sections: add the heading and summary only — flag clearly that the user needs to add the substantive content

After applying all changes, show a brief diff:
```
✅ Updated [N] outdated facts
✅ Updated [N] tool references
✅ Added [N] section(s): "[title]" — add your content here
✅ Updated meta title / description
⏭️ Skipped: [N] items

Ready to publish?
```

⚡ GUARD — **Webflow MCP unavailable:** Output the full diff as copy-pasteable markdown for manual application.

---

## Phase 4: Publish

Ask once: "Publish now or save as draft?"

---

## Activity Log

Follow CLAUDE.md format. Example:
```
| YYYY-MM-DD | /refresh-content | [Post title]: updated 3 stats, 2 tool refs, added "[section]" section. Published. |
```
