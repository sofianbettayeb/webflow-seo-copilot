---
name: ai-visibility
version: "1.0"
description: |
  Baseline AI visibility report — a one-page client diagnostic showing how a brand appears across ChatGPT, Claude, Perplexity, and Google AI Overview. Pulls live data from AEO Copilot, computes per-LLM scorecards, surfaces gaps with evidence, and renders a single self-contained HTML file (print-styled for Cmd+P → PDF).
  Triggers: ai visibility, ai visibility report, baseline ai report, llm visibility, llm visibility report, brand visibility audit, aeo baseline, aeo report.
  Requires: AEO Copilot MCP server. Optional: nothing.
  Workflow: Discover → Fetch → Aggregate → Render → Save.
  Command: /ai-visibility
---

# AI Visibility Skill

Baseline diagnostic of a brand's presence across the four major AI engines (ChatGPT, Claude, Perplexity, Google AI Overview). Designed as a **client deliverable** — a single HTML page Sofian hands to a new client at the start of an engagement so they can answer in five minutes:

1. *How visible is my brand in LLMs today?*
2. *What's working and what's not?*

This skill is **read-only** and **diagnostic-only**. It surfaces findings with evidence (specific prompt counts, scores, citations, competitor names) but never prescribes "do this" actions. The client takes the diagnosis to inform their own roadmap.

## Prerequisites

- **Required**: AEO Copilot MCP server — exposes `list_brands`, `list_topics`, `get_results`, `get_insights`, `get_recommendations`. Repository: https://github.com/sofianbettayeb/aeo-copilot-mcp.

## Workflow Overview

```
DISCOVER → FETCH → AGGREGATE → RENDER → SAVE
```

---

## Critical Guards

⚡ GUARD — **AEO Copilot MCP unavailable:**
- Stop: "AI Visibility report requires the AEO Copilot MCP. Connect `aeo-copilot-mcp` and try again."

⚡ GUARD — **Brand has no tracked prompts (empty `get_results`):**
- Stop: "Brand `{name}` has no tracked prompts in AEO Copilot yet. Add topics and prompts in the dashboard, run a tracking cycle, then re-run `/ai-visibility`."

⚡ GUARD — **Very small prompt corpus (< 10 prompts in 90 days):**
- Warn: "Small prompt corpus ({N} prompts in 90 days). Findings will be directional, not statistical. Continue?"
- If declined: stop.

⚡ GUARD — **User requests abort:**
- Confirm: "Stop the workflow? Progress will be lost."
- If confirmed: exit cleanly, output any partial results, still write activity-log row noting abort reason.

---

## Phase 0: DISCOVER

### 0.1 MCP Discovery

Search for tools BEFORE starting:

- **AEO Copilot MCP** (required): search `+aeo copilot brands`. If missing → stop (see guard).

### 0.2 Load Config

Load `.claude/seo-copilot-config.json` if it exists. Extract:
- `business.name` → cross-check against AEO Copilot brand name (sanity check the user picked the right brand).
- `seo.competitors` → highlight these in the competitor landscape section if they appear.

If not found: proceed with defaults, note once "Run `/getting-started` for personalized recommendations."

### 0.3 Brand Selection

Always prompt the user — never auto-select.

1. Call `list_brands`.
2. If 0 brands: stop with "No brands found in AEO Copilot. Add a brand in the dashboard first."
3. If 1 brand: confirm — "Found one brand: `{name}` ({website}). Generate the baseline report for this brand?"
4. If 2+ brands: present a numbered list (name, website, industry) and ask user to pick one.

Store: `brandId`, `brandName`, `brandDomain` (extracted from website URL — strip `https://`, `www.`, trailing slash, lowercase).

### 0.4 Activity Log Check

Read `.claude/reports/{brandDomain}/activity-log.md` if present. Surface:
- Any prior `/ai-visibility` runs in the last 60 days — note the date so the user knows whether this is a true baseline or a re-run.
- Recent activity from other skills on this brand.

If no activity log: proceed silently.

### 0.5 Set Date & Paths

- `report_date` = today (YYYY-MM-DD)
- `report_path` = `.claude/reports/{brandDomain}/ai-visibility-baseline-{report_date}.html`
- `activity_log_path` = `.claude/reports/{brandDomain}/activity-log.md`

Create the directory if it doesn't exist: `mkdir -p .claude/reports/{brandDomain}/`.

---

## Phase 1: FETCH

Call these four MCP tools in parallel:

### 1.1 `list_topics(brandId)`
Topic catalog with name, description, target customers, geographic focus, differentiation, pages, keywords. Used to label the topic-performance table and group gap prompts.

### 1.2 `get_results(brandId, from=90d ago, to=today, limit=500)`
The evidence corpus. Each row: prompt, topic, per-LLM mention status (`chatgptMentioned`, `claudeMentioned`, `perplexityMentioned`, `googleAioMentioned`), per-LLM position, per-LLM sentiment, sources, competitors.

If `limit=500` is hit (corpus is larger), record this and add a footnote in the methodology.

### 1.3 `get_insights(brandId)`
Pre-computed: visibility score, sentiment breakdown, competitive share, weekly trends, top topics, competitor list. **Use the platform's score as-is** — do not recompute.

### 1.4 `get_recommendations(brandId)`
Used as one signal feeding the "what's not working" findings. **Never surface as prescriptions** — translate them into descriptive findings ("X gap detected" not "do Y to fix it").

---

## Phase 2: AGGREGATE

Compute four derived datasets the API doesn't return directly. Every aggregation must preserve the underlying counts so they can be cited as evidence.

### 2.1 Per-LLM Scorecard

For each engine in `[chatgpt, claude, perplexity, googleAio]`, iterate `get_results` rows and compute:

- **Mention rate %** = `mentioned_count / total_prompts × 100`
- **Avg position** when mentioned (skip rows where the engine didn't mention; report "n/a" if 0 mentions)
- **Sentiment split**: counts of positive / neutral / negative across mentioned rows
- **Strongest topic**: topic with highest mention rate on this engine (ties broken by prompt count)
- **Weakest topic**: topic with lowest mention rate on this engine (where prompt count ≥ 3 — ignore tiny topics)

### 2.2 Topic Performance Table

For each topic, compute:
- Prompt count
- Combined visibility % across all 4 engines (sum of mentions / (4 × prompt_count) × 100)
- Avg position when mentioned (across all 4 engines)
- Sentiment skew: dominant sentiment label (positive / neutral / negative / mixed)

Sort by visibility % descending.

### 2.3 Gap Inventory

A prompt is a **gap** if the brand is mentioned in 0 or 1 of the 4 engines. For each gap prompt, capture:
- The prompt text
- The topic it belongs to
- Which engines missed the brand
- Which competitors appeared in those engines (deduplicated, ranked by frequency)

### 2.4 Competitor Share-of-Voice on Gaps

Across all gap prompts, count competitor appearances. Top 5 by count. For each, also record:
- Number of gap prompts they appear in
- Topics where they dominate (topics where this competitor's count > brand's count)

---

## Phase 3: BUILD FINDINGS

Generate the narrative content for sections 5 (What's working) and 6 (What's not working). Every finding follows the same structure:

```
{one-sentence finding} — {evidence line with concrete numbers}
```

**Evidence-first principle (non-negotiable):** every finding must cite at least one of:
- A prompt count (e.g., "11 of 62 prompts")
- A score with its band (e.g., "B grade — 68/100")
- A specific competitor name (e.g., "Competitor X appears in 19 of 26 'vs' prompts")
- A specific source URL or topic name

No floating claims like "the brand performs well" without a number behind it.

### 3.1 What's Working — 3 to 5 findings

Source from:
- Engines or topics with mention rate ≥ 40%
- Topics with positive sentiment ≥ 50% of mentions
- Position metrics where avg position ≤ 3
- Source attribution wins (brand cited as primary source)

### 3.2 What's Not Working — 3 to 5 findings

Source from:
- Engines or topics with mention rate ≤ 20%
- Topics with high prompt count (≥ 5) and zero brand mentions on any engine
- Sentiment patterns where negative ≥ 25% of mentions
- Competitor dominance: topics where one competitor outperforms the brand by 2× or more
- Technical gaps surfaced by `get_recommendations` — translate prescriptions into descriptive findings (e.g., recommendation "Add llms.txt" → finding "No llms.txt detected on the domain")

### 3.3 Format Adaptation

Per repo CLAUDE.md output guidance:
- If a section has fewer than 3 findings, collapse into a paragraph instead of bullets.
- Never output a section header with nothing under it.
- Prefer specificity over completeness — one concrete finding beats five vague ones.

---

## Phase 4: RENDER

### 4.1 Load template

Read `template.html` from this skill's directory. It contains the full HTML/CSS skeleton with `{{placeholder}}` tokens for every dynamic value.

### 4.2 Fill placeholders

Replace placeholders with computed values. The template defines all of them — see the template file for the full list. Key groups:

- Header: brand name, domain, date, overall grade, TL;DR
- Executive scorecard: 3 score cards (AI Visibility, Technical Readiness, Sentiment Health) with band labels
- Per-LLM grid: 4 cards × {mention rate, avg position, sentiment bar, strongest topic, weakest topic}
- Topic table: rows from Phase 2.2
- What's working / what's not working: rendered as `<li>` evidence lines
- Competitor landscape: top 5 with bar widths proportional to share-of-voice
- Appendix table: full gap-prompt inventory from Phase 2.3
- Footer: methodology, data window, prompt corpus size, last platform sync

### 4.3 Visual design

Use the `frontend-design` skill conventions — no generic AI aesthetic. Required:
- Editorial serif for headlines (e.g., Fraunces, Source Serif Pro), clean sans-serif for body
- Neutral professional palette — white background, near-black text, one accent color for scores/charts
- Print-styled: `@page { size: A4; margin: 16mm }`, `page-break-inside: avoid` on cards and tables, hide nav/buttons in `@media print`
- Self-contained: inline CSS, web fonts via `<link>` only (no external JS, no images that aren't inline SVG or data URIs)

### 4.4 Section order

1. Header (brand + grade + TL;DR)
2. Executive scorecard
3. Per-LLM visibility (4-card grid)
4. Topic performance (table)
5. What's working (evidence-backed findings)
6. What's not working (evidence-backed findings)
7. Competitor landscape (top 5 bar chart)
8. Appendix: gap prompts (full evidence table)
9. Methodology footer

---

## Phase 5: SAVE & LOG

### 5.1 Write report

Write the rendered HTML to `report_path`.

### 5.2 Activity log

Append to `activity_log_path`. If the file doesn't exist, create it with the standard header:

```markdown
# Activity Log — {brandDomain}

| Date | Skill | Summary |
|------|-------|---------|
```

Append one row:

```
| YYYY-MM-DD | /ai-visibility | Baseline AI visibility report. Grade: {X}. Visibility: {N}%. Strongest engine: {engine} ({M}%). Weakest: {engine} ({M}%). {C} competitors tracked. |
```

Log even on early exit (with reason).

### 5.3 Success message

Output to the user:

```
✅ Baseline AI visibility report saved.

Path: .claude/reports/{brandDomain}/ai-visibility-baseline-{date}.html

To export as PDF: open the file in a browser, press Cmd+P, choose "Save as PDF".

Top finding: {one-line TL;DR pulled from the report header}.
```

---

## References

- `references/scoring-notes.md` — AEO Copilot scoring formula (60% topic / 25% technical / 15% sentiment, A–F bands). Load this when generating findings to ensure language stays consistent with how the platform scores.
- `template.html` — HTML/CSS skeleton with placeholders.

## Integration with Other Skills

| Finding | Skill | When |
|---------|-------|------|
| Low engine visibility, content gaps | `/aeo-optimize {url}` | Page-level AEO rewrite |
| Schema/llms.txt gaps surfaced | `/aeo-optimize:audit` | Quick AEO scoring |
| Topic coverage gaps | `/keywords-opportunity:discover` | Find new content topics |
| Track progress over time | AEO Copilot dashboard | Weekly automated tracking |
