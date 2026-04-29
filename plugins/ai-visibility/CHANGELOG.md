# Changelog — ai-visibility

## 1.3.0 — 2026-04-29

Added a render-verification pass. Caught a real bug on the `CloudNC` baseline where Google AIO (aggregate-only, zero mentions) was missing from every appendix row's "Missing on" cell. The skill now self-checks before declaring success, so future reports can't ship with the same class of regression.

### Added

- **Phase 4.5 — VERIFY.** Five deterministic acceptance checks run on the rendered HTML before writing the file. Failures loop back to Phase 4 with a specific diff message; abort after two consecutive failures rather than ship a partial report.
  1. No leftover `{{placeholder}}` tokens in the output.
  2. Section completeness — every `<!-- ========== ... ========== -->` marker in `template.html` appears in the render.
  3. Engine coverage in appendix — every engine in `engines_to_report` either appears in ≥1 gap row's "Missing on" cell, is in `aggregate_partial_engines` (exempt), or has 100% mention rate (also exempt).
  4. Gap row count == `len(gap_inventory)`.
  5. Internal numeric consistency — `totalCitations`, per-engine `mentionCount`, and `gapPromptCount` all match the Phase 2 aggregates.

- **Phase 1.2c — Compute the canonical engine list.** Single source of truth: `engines_to_report` (always all four), with `aggregate_zero_engines` and `aggregate_partial_engines` derived from per-row availability and mention totals. All downstream phases read this one variable instead of re-deriving from `engines_with_per_row`.

### Changed

- **Phase 2.3 — Gap inventory.** Three explicit rules for building `missed_engines` per gap row. Most importantly: aggregate-only engines with zero mentions across the corpus (e.g., Google AIO when `googleAioMentions: 0` for every topic) are appended to **every** gap row's missed list. Aggregate-only with partial mentions remain omitted from per-row missed and footnoted on the engine card. Phase 4.5.3 enforces this.

### Notes

- No template changes. The bug fix lives in skill logic only; existing reports re-render correctly under 1.3.0.
- Cache rebuild required (1.2.0 → 1.3.0).
- Estimated overhead per run: ~2 seconds for the verification pass.

## 1.2.0 — 2026-04-29

Tested against the `CloudNC` brand baseline. Two changes to the report layout, surfaced through a real client run.

### Layout

- **"How to read this report" moved to the top.** Previously sat at the bottom in the methodology footer; clients reached the scorecard before knowing the framing of the score. Block now renders right after the TL;DR, above the Executive scorecard, so the reader has the methodology context before seeing the grade.
- **"What's next" card is the new render target for the bottom of the report.** The methodology block no longer competes with the CTA for the closing slot.

### Notes

- No skill workflow changes. Phase 4 placeholder fill remains identical; only the position of the methodology div in `template.html` changed.
- Cache rebuild required (this is the v1.1.0 → v1.2.0 bump).

## 1.1.0 — 2026-04-25

Tested against the `Thunder` brand baseline. Surfaced and fixed several issues that would have made v1.0.0 ship a broken or untrustworthy report on real client data.

### Bugs fixed

- **Print color stripping (P0).** Bar fills, sentiment segments, pills, and grade badges rendered white-on-white in PDF because Chrome's print engine defaults to `print-color-adjust: economy`. Added `print-color-adjust: exact !important` on all colored elements and a global `@media print` backstop. Verified bars and pills now render in Cmd+P → PDF.
- **Engine schema mismatch.** SKILL.md assumed all four engines (ChatGPT, Claude, Perplexity, Google AIO) would return per-row data via `get_results`. Some brands only have per-row data for three engines, with the fourth available only as aggregates in `get_insights.mentionVolume`. Added Phase 1.2a (engine field detection) and a guard that falls back to aggregate data with a footnote rather than fabricating per-prompt detail.
- **Date filter unreliable.** `get_results(brandId, from=X, to=Y)` was observed to return zero rows when filtered by date even though data was inside the window. Changed Phase 1.2 to fetch unfiltered and trim client-side by `runDate`.

### New section

- **External source influence.** Aggregates the domains LLMs cite when answering the brand's prompts (top 10), with a "Third-party" tag in the section header. Surfaces whether the brand's own domain is part of the evidence base. When the brand domain appears in 0 or < 5% of citations, an additional gap finding is generated automatically. Added as Phase 2.5 (aggregation), Section 8 (template), and a new finding source in Phase 3.2.

### Visual polish

- **Per-engine logos.** Inline SVG marks for ChatGPT, Claude, Perplexity, Google AIO, with brand colors. Self-contained, print-safe. Logos and CSS documented in `references/engine-logos.md`.
- **Label rename.** "Google AI Overview" → "Google AIO" everywhere. Two-word label avoids line-wrapping on tight 4-column grids.

### Content quality

- **Mandatory voice pass.** Added Phase 3.4: every piece of generated copy is run through the humanizer checklist before render. Em dashes, rule-of-three padding, copula avoidance, and AI-vocabulary words ("showcasing", "highlighting", "natural beachhead") are stripped. New `references/voice-rules.md` documents the rules with before/after examples.
- **Adaptive data window labels.** Replaced hardcoded "Last 90 days" in the methodology footer with a `dataWindowLabel` placeholder that reflects the actual `runDate` range: "the last sync", "the past N days", or "the past 90 days". The skill no longer claims a 90-day window when the data is one week old.

### Robustness

- **Brand picker filtering.** `list_brands` returns test/example/autogenerated brands alongside real ones. Phase 0.3 now filters them out by heuristics (no website, `*.example.*` domains, names ending in 3–4 random alphanumerics, names starting with "Test"/"Manual Test") and shows a "+N test brands hidden" footer.
- **Config cross-check.** When `seo-copilot-config.json` has a `business.name` that matches an AEO Copilot brand, the picker pre-selects it.
- **Brand vs category sentiment.** `get_insights.sentiment` aggregates across all tracked prompts, including ones where the brand wasn't mentioned. The platform's Sentiment Health number can read 80%+ when the brand actually has 1 neutral mention. `references/scoring-notes.md` now documents this distinction and the skill's scorecard driver clarifies which is which.
- **Three-band corpus threshold.** Replaced the single < 10 prompt warning with three bands: < 10 (stop), 10–49 (label findings as directional), 50+ (statistical). The corpus band is shown in the methodology footer.

### Files changed

- `.claude-plugin/plugin.json` — version 1.1.0, description updated
- `skills/ai-visibility/SKILL.md` — major rewrite of phases, guards, and fetch logic
- `skills/ai-visibility/template.html` — print-color CSS, logo head, source-influence section, label updates
- `skills/ai-visibility/references/scoring-notes.md` — brand vs category sentiment section
- `skills/ai-visibility/references/voice-rules.md` — new
- `skills/ai-visibility/references/engine-logos.md` — new
- `plugins/ai-visibility/CHANGELOG.md` — new

## 1.0.0 — 2026-04-24

Initial release.
