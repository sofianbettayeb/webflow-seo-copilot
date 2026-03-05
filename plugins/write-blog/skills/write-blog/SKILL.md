# /write-blog

Create a new SEO and AEO-optimized blog draft in Webflow CMS. Follows project conventions from `/getting-started`, applies the AEO dimensions from `/aeo-optimize`, runs the AI Writing Tell-Tales check from `/refresh-content`, and mandates a `/humanizer` pass before writing to Webflow.

---

## Trigger phrases

- `/write-blog`
- "write a blog post"
- "create a Webflow blog draft"
- "draft an SEO article"
- "draft an AEO article"

---

## MCP requirements

| Tool | Required | Used for |
|------|----------|----------|
| Webflow MCP | Yes | Create CMS item draft, write fields, set SEO fields |
| Keywords Everywhere MCP | Strongly recommended | Keyword expansion — related keywords, long-tail, questions |
| Google Search Console MCP | Optional | Validate intent and query patterns if a matching page already exists |

If Keywords Everywhere is not available, proceed with a GSC-only or reasoning-only approach and warn that volume data is missing.

---

## Modes

| Mode | Command | Behavior |
|------|---------|----------|
| Default | `/write-blog` | Full workflow: brief → keywords → outline approval → draft → AI tell-tales check → humanizer → AEO audit → Webflow CMS draft |
| Outline only | `/write-blog:outline` | Stops after outline approval. No Webflow changes. |
| Auto-publish | `/write-blog:publish` | Creates CMS draft and publishes immediately after user approval. |

---

## Required inputs (always ask if missing)

- **Primary keyword** — provided by user
- **Search intent** — ask explicitly: informational / commercial investigation / product-focused / mixed
- **Article objective** — rank, educate, capture leads, support positioning
- **Target collection** — confirm default blog collection or ask
- **Primary CTA** — optional but recommended
- **Internal links to include** — optional

## Optional inputs (strongly encourage)

- User insights, opinions, research, quotes, frameworks, personal examples
- Audience notes (if not already in config)
- Competitors to reference (if not already in config)

---

## Global config reuse

At start, load `.claude/seo-copilot-config.json` created by `/getting-started`. Use it to enforce:

- Brand voice and tone
- Audience and expertise level
- Markets and language preferences
- SEO constraints (meta title format, internal link preferences, competitors)
- AEO constraints (FAQ strategy, citation goals, E-E-A-T signals)
- Content preferences (headings, lists, CTAs)

If the config file is missing, proceed with defaults and recommend running `/getting-started` first.

---

## Skill reuse

### `/keywords-opportunity` — optional, use as a support step

Run when:
- The user wants "the best keyword to write about"
- The user provides a broad seed and wants expansion with evidence
- The site has GSC data and the user wants to prioritize ROI

How to use the result:
- Pick 1 primary keyword (user-confirmed)
- Extract secondary keywords (semantic + long-tail)
- Extract question keywords for FAQ and definition blocks

### `/aeo-optimize` rubric — mandatory

Apply the same AEO dimensions and checks as `/aeo-optimize`, adapted for pre-publish drafting:
- Use the user-provided primary keyword as the primary query
- Run the AEO audit pass on the draft before writing to Webflow
- Output a short pass/partial/fail table and apply fixes automatically

The goal is content that would pass `/aeo-optimize:audit` with minimal changes after publication.

---

## Workflow

### Phase 0 — Setup and guards

1. Verify Webflow MCP is connected. If missing, stop and tell the user.
2. Load `.claude/seo-copilot-config.json` if available.
3. Read `.claude/reports/{domain}/activity-log.md` and surface any recent work on the same topic cluster to avoid duplicates.

---

### Phase 1 — Brief intake (human input required)

Ask only for missing required fields:
- Primary keyword
- Search intent
- Article objective
- Any specific angle, constraints, or things to avoid
- Internal links and CTA
- **Ask explicitly for insights, quotes, research, personal POV** — this is what separates the article from generic AI output

Stop and wait for user confirmation before proceeding.

---

### Phase 2 — Keyword expansion (best-effort)

1. Expand with Keywords Everywhere:
   - Related keywords
   - Long-tail variations
   - Question queries ("how to", "what is", "best way to")

2. If GSC is available and the user provides a related existing URL, pull page queries as supporting context (do not require it).

3. Propose:
   - Secondary keyword list (8–20 terms)
   - 3–8 FAQ questions derived from question keywords
   - Recommended content format (checklist, how-to, comparison, glossary) aligned with intent

If KE is not connected, provide a smaller keyword set based on reasoning and user input, and warn about missing volume evidence.

---

### Phase 3 — Outline proposal (human input required)

Produce an outline with:
- H1 (includes primary keyword, sounds natural — not stuffed)
- Intro intent confirmation (what the reader gets in 2 sentences)
- Definition block (short, direct — for AEO)
- H2 sections ordered by user intent
- Where a table helps (comparison, checklist, decision matrix)
- Proposed FAQ questions
- Suggested internal link placements

Ask the user to approve:
- The outline
- The angle
- The CTA placement

Do not proceed to draft until approved.

---

### Phase 4 — Draft writing

#### Writing style rules

- Short sentences. Short paragraphs (2–3 lines max).
- Concrete explanations over abstract claims.
- No hype, no filler, no AI vocabulary.
- Follow the brand voice and audience level from config.
- Use user-provided insights, quotes, and examples — these are mandatory inputs, not optional flavour.

#### Default structure

- **Title (H1)** — includes primary keyword, reads naturally
- **Introduction** — 80–140 words, states clearly what the reader will learn
- **Definition section** — short and direct (feeds AEO answer boxes)
- **Main sections (H2/H3)**
- **At least one scannable element** — checklist, bullets, or comparison table
- **FAQ** — 3–6 Q&As, concise answers (2–4 sentences each)
- **Conclusion** — 60–120 words
- **CTA** — aligned with the article objective

---

### Phase 4.5 — AI Writing Tell-Tales check (mandatory)

Before humanizer and AEO audit, scan the full draft against the following. Fix all violations automatically.

#### Structural patterns to AVOID

- Emoji bullets (✅ ❌ 🚀) — dead giveaway of AI content
- Every section having exactly 3–5 bullet points
- Perfect parallel structure in all headings ("Webflow Designers", "Webflow Developers", "Webflow Marketers")
- "Key Takeaways" as the opening section
- "Final Thoughts" or "Conclusion" headers
- Numbered lists for things that don't need sequence

#### Filler phrases to NEVER USE

- "Here's what it offers:"
- "Let's dive in" / "Let's explore"
- "Whether you're a [X], [Y], or [Z]"
- "In today's [industry]..."
- "From [X] to [Y], [product] has you covered"
- "everything you need to [verb]"
- "takes it to the next level"
- "game-changer" / "revolutionary" / "cutting-edge"
- "In this article, we'll..."
- "Without further ado"

#### Formatting to AVOID

- Excessive em-dashes (—) — limit to 2–3 per article
- Bold on every key phrase
- Every paragraph starting with the product/brand name
- Perfectly uniform paragraph lengths

#### Add human signals (actively inject these)

- First-person experience: "I've tested this on 50+ sites"
- Specific numbers: "saves ~15 minutes per page" not "saves time"
- Honest limitations: "It won't help with backlinks or technical crawl issues"
- Casual asides: "honestly, this is the part I use most"
- Opinions with edge: "other tools charge $30/month for less"
- Imperfect structure: vary bullet counts (2 here, 6 there)
- Natural transitions, not formulaic ones

---

### Phase 4.6 — Humanizer pass (mandatory)

Run a full humanizer pass on the draft before the AEO audit and before writing to Webflow.

Apply the following rules to every sentence:

- No AI vocabulary: no "seamless", "robust", "delve", "comprehensive", "leverage", "harness", "empower", "transformative", "streamline", "cutting-edge", "game-changing", "innovative"
- No em-dash overuse — max 2–3 per article (already checked in Phase 4.5, confirm here)
- No rule-of-three padding — avoid listing exactly three parallel items just to fill a pattern
- No inflated symbolism or vague metaphors
- No excessive conjunctive phrases ("Furthermore", "Moreover", "In addition to", "It is worth noting that")
- No negative parallelisms ("not only X, but also Y")
- No promotional superlatives without evidence
- Short, direct sentences — if a sentence has more than two clauses, split it

If a sentence fails multiple checks, rewrite it. Do not patch — rewrite.

Present the humanized draft to the user before proceeding. Ask: "Does this read the way you'd write it? Any adjustments before I run the AEO audit?"

---

### Phase 5 — AEO audit pass

Evaluate the humanized draft across the `/aeo-optimize` dimensions. Output a short table:

| Dimension | Status | Note |
|-----------|--------|------|
| Definition block | Pass / Partial / Fail | One-line note |
| FAQ coverage | Pass / Partial / Fail | One-line note |
| E-E-A-T signals | Pass / Partial / Fail | One-line note |
| Heading structure | Pass / Partial / Fail | One-line note |
| Internal links | Pass / Partial / Fail | One-line note |
| Meta title + description | Pass / Partial / Fail | One-line note |
| Schema opportunity | Pass / Partial / Fail | One-line note |

Apply fixes automatically unless a fix changes meaning — in that case, ask for approval.

---

### Phase 6 — Webflow CMS draft creation

1. Identify the blog collection.
2. Create a CMS item as a draft with:
   - Title
   - Slug (lowercase, hyphenated, includes primary keyword)
   - Meta title (50–60 chars, includes primary keyword)
   - Meta description (120–155 chars, benefit-led)
   - Rich text body
   - FAQ field (if the collection schema supports it)
   - Schema markup field (if present)

If the collection schema is unknown or inconsistent, recommend running `/cms-collection-setup` first.

---

### Phase 7 — References (optional)

If the user asked for references, or if the config requires citations:
- Prefer reputable sources (industry publications, research reports, major vendor docs)
- Add as a short "Sources" section at the end or as inline mentions
- Keep citations lightweight — do not over-link

If the user did not request references and config does not require them, skip.

---

### Phase 8 — Save and log

Append to `.claude/reports/{domain}/activity-log.md`:

```
| YYYY-MM-DD | /write-blog | Drafted "[article title]". Primary keyword: "[keyword]". Collection: [name]. CMS item ID: [id]. Status: draft / published. |
```

---

## Output (Webflow-ready summary)

Return to the user:
- CMS item name
- Slug
- Meta title
- Meta description
- Draft URL or item ID (if available)
- Final article body as written to Webflow

---

## Non-goals

- Does not perform a full site audit → use `/audit` or `/audit-deep`
- Does not replace ongoing optimization of existing URLs → use `/refresh-content`, `/click-recovery`, `/aeo-optimize` after publication
