# Webflow SEO Copilot

Claude Code skills that turn your Webflow CMS into an SEO machine — refresh content, optimize rankings, and recover lost traffic, all through natural language.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add sofianbettayeb/webflow-seo-copilot

# Install skills
/plugin install getting-started@webflow-seo-copilot
/plugin install cms-collection-setup@webflow-seo-copilot
/plugin install keywords-opportunity@webflow-seo-copilot
/plugin install refresh-content@webflow-seo-copilot
/plugin install click-recovery@webflow-seo-copilot
/plugin install monthly-report@webflow-seo-copilot
/plugin install weekly-report@webflow-seo-copilot
/plugin install audit@webflow-seo-copilot
/plugin install audit-deep@webflow-seo-copilot
/plugin install aeo-optimize@webflow-seo-copilot

# First time? Run setup to capture your brand voice & SEO goals
# This also installs global activity logging for all Webflow work
/getting-started

# Set up or audit your blog collection schema
/cms-collection-setup

# Then use the SEO skills
/keywords-opportunity          # find striking distance + new keyword opportunities
/aeo-optimize https://yoursite.com/blog/article-slug  # optimize any page for AEO
/refresh-content https://yoursite.com/blog/article-slug
/click-recovery
/monthly-report
/weekly-report

# Audit any site (no MCP needed)
/audit https://example.com

# Deep audit (requires GSC + Webflow MCP)
/audit:deep
```

## Activity Logging

Every session on a Webflow site is automatically logged to `.claude/reports/{domain}/activity-log.md` — whether you're using a skill or just working directly in conversation.

Running `/getting-started` once installs a global instruction (`~/.claude/CLAUDE.md`) that activates this behavior across all sessions.

The log tracks:
- Skill executions (`/refresh-content`, `/click-recovery`, etc.)
- Ad-hoc work (SEO fixes, page edits, content changes, GSC analysis)
- What was changed and what was published

Each skill reads the log at startup to avoid redundant work and surface recent context.

**Requirements:**
- [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) (required)
- [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server) (required for `/click-recovery`, `/monthly-report`, `/weekly-report`, and `/audit:deep`, optional for `/refresh-content`)
- `/audit {url}` requires no MCP servers — works with any public URL
- [Keywords Everywhere MCP server](https://github.com/hithereiamaliff/mcp-keywords-everywhere) (optional, needs API key)

## Why Keep Content Fresh?

Stale content loses rankings. It's that simple.

- **SEO** — Google's freshness algorithm boosts updated content. Old dates in titles kill CTR. Competitors publish newer articles and take your spot.
- **AEO** — AI answer engines (ChatGPT, Perplexity, AI Overviews) prefer recent sources. Updated articles with structured FAQs get cited. Stale ones get ignored.

A single refresh can recover months of lost traffic.

## Skills

### `/getting-started`

[View full skill →](plugins/getting-started/skills/getting-started/SKILL.md)

One-time setup that captures your business context and saves it to a persistent config file. Run once per project.

**What it captures:**
- Business goals and primary outcomes
- Target audience and expertise level
- Markets, regions, and languages
- Brand voice and tone preferences
- SEO constraints (keywords, competitors, link preferences)
- AEO constraints (citation goals, FAQ strategy, schema types)
- Content style (headings, lists, CTAs)

**What happens:**
- Creates `.claude/seo-copilot-config.json` in your project
- Other skills read this config for consistent, on-brand outputs
- Re-run anytime to update your settings

**Usage:**
```
/getting-started
```

---

### `/keywords-opportunity`

[View full skill →](plugins/keywords-opportunity/skills/keywords-opportunity/SKILL.md)

Find keywords worth targeting — both rankings you already have that deserve more traffic, and new topics you're not targeting yet.

**What it does:**
- Pulls 90 days of GSC query data and classifies rankings into three striking distance tiers (positions 4-10, 11-20, 21-30)
- Enriches every query with Keywords Everywhere: monthly volume, CPC, competition, trend
- Identifies content gaps — GSC queries with impressions but no dedicated page
- Expands into new keywords via Keywords Everywhere related and long-tail suggestions for your top topics
- Clusters long-tail GSC queries into consolidation candidates (one article for a whole cluster)
- Scores every opportunity by Impact × Confidence / Effort and routes to the right skill
- Saves a timestamped report per site, referenced by weekly/monthly reports

**Usage:**
```
/keywords-opportunity             — full: striking distance + new discovery
/keywords-opportunity:striking    — page 1-3 wins only (fastest ROI)
/keywords-opportunity:discover    — new keywords not yet targeted
```

---

### `/cms-collection-setup`

[View full skill →](plugins/cms-collection-setup/skills/cms-collection-setup/SKILL.md)

Review an existing Webflow CMS collection against the recommended SEO schema — or create a new one from scratch with all the fields that matter for rankings, authority, and sharing.

**What it does:**
- Audits an existing collection field-by-field against the recommended schema (Core, Content, Authority, Enhancement, Social)
- Scores coverage per group with an overall SEO schema health score
- Detects exact matches, alias matches, and type mismatches
- Flags missing fields with explanations of why each one matters
- Offers to add missing fields to an existing collection in one step
- Creates a new collection from scratch with all 20+ recommended fields
- Handles flexible fields: Author (single ref or multi), Tags (option list or multi-ref), FAQ (multi-ref or inline), Pillar page (external or self-reference)
- Skips reference fields gracefully when the target collection doesn't exist yet

**Usage:**
```
/cms-collection-setup          — guided: choose review or create
/cms-collection-setup:review   — audit an existing collection
/cms-collection-setup:create   — build a new collection
```

---

### `/refresh-content`

<img width="1243" height="434" alt="image" src="https://github.com/user-attachments/assets/9f0adcea-8916-4d00-88c3-dce950bb2b3a" />

[View full skill →](plugins/refresh-content/skills/refresh-content/SKILL.md)

Refresh and optimize existing Webflow blog articles to improve rankings, recover lost traffic, and extend content lifespan.

**What it does:**
- Maps your CMS schema automatically — works with any Webflow collection structure
- Updates dates, meta titles, and meta descriptions with freshness signals
- Refreshes keywords using GSC data or SERP research (fallback when GSC is unavailable)
- Discovers and adds internal linking opportunities across all your collections
- Generates FAQ sections with schema markup (populates dedicated CMS FAQ fields when available)
- Adds Article, FAQ, Breadcrumb, and HowTo structured data
- Audits and populates empty CMS fields (keywords, topics, alt text, author, read time)
- Preserves inline images during content rewrites
- Presents a before/after diff for review before publishing
- Provides a post-publish monitoring checklist

The skill uses conditional guards — smart checks that only trigger when something is missing or needs attention. Simple refreshes stay fast; complex ones get the full treatment.

**Usage:**
```
/refresh-content https://yoursite.com/blog/article-slug
```

Or just ask Claude to refresh, update, or optimize a blog article — the skill activates automatically.

---

### `/click-recovery`

[View full skill →](plugins/click-recovery/skills/click-recovery/SKILL.md)

Find pages Google already trusts but users ignore. No crawling, no code — just fast wins with measurable impact.

**What it does:**
- Pulls GSC data to find high-impression, low-CTR pages
- Validates queries with Keywords Everywhere (optional) for volume and intent
- Scores and prioritizes opportunities by wasted traffic potential
- Suggests new meta titles and descriptions based on search intent
- Handles both CMS items and static pages
- Warns about title/description length issues before publishing
- Publishes approved changes directly to Webflow CMS

**Why it's cool:**
Google already ranks these pages. You're not building authority — you're fixing the pitch. A better title and description can recover clicks you're already earning impressions for.

**Usage:**
```
/click-recovery
```

The skill analyzes your GSC data, presents a prioritized report, and asks which pages to update. Approve the changes and they're published to Webflow.

---

### `/monthly-report`

[View full skill →](plugins/monthly-report/skills/monthly-report/SKILL.md)

A decision-making tool for founders and executives. Compares this month vs last month, scores every recommendation, and outputs a prioritized action plan.

**What it does:**
- Compares Month M vs M-1 with 3-month trend data
- Splits branded vs non-branded traffic
- Identifies top growing and declining pages
- Finds content gaps — queries with high impressions but no matching page
- Cross-references Webflow pages with GSC index status
- Audits metadata (missing, duplicate, too long/short)
- Flags high-impression/low-CTR pages and striking distance keywords
- Analyzes internal linking structure and orphan pages
- Scores every recommendation by Impact, Confidence, and Effort
- Outputs a prioritized action plan in 3 buckets: Must fix, High impact, Nice to have

**This skill is read-only** — it never modifies Webflow content. It points you to `/click-recovery` and `/refresh-content` for execution.

**Usage:**
```
/monthly-report
/monthly-report:quick
```

The full report covers 7 sections: executive summary, performance overview, content performance, technical health, on-page opportunities, internal linking, and action plan. Quick mode outputs only the executive summary and action plan.

---

### `/weekly-report`

[View full skill →](plugins/weekly-report/skills/weekly-report/SKILL.md)

A weekly SEO pulse that compares this week vs last week, tracks progress against the monthly report, and outputs a prioritized action plan.

**What it does:**
- Compares Week W vs W-1 with 4-week trend data
- Splits branded vs non-branded traffic
- Identifies top growing and declining pages
- Finds content gaps with adaptive thresholds (relative for small sites, absolute for large)
- Audits CMS templates — flags collections using `name` instead of a dedicated SEO title field
- Cross-references Webflow pages with GSC index status
- Audits metadata (missing, duplicate, too long/short)
- Flags high-impression/low-CTR pages and striking distance keywords
- Scores every recommendation by Impact, Confidence (capped for weekly data), and Effort
- Links back to last monthly report for progress tracking
- Saves reports per site: `.claude/reports/{domain}/weekly-report-YYYY-WXX.md`
- Outputs a prioritized action plan in 3 buckets: Must fix, High impact, Nice to have

**This skill is read-only** — it never modifies Webflow content. It points you to `/click-recovery` and `/refresh-content` for execution.

**Usage:**
```
/weekly-report
/weekly-report:quick
```

The full report covers 6 sections: executive summary (with monthly context), performance overview, content performance, technical health, on-page opportunities, and action plan. Quick mode outputs only the executive summary and action plan.

---

### `/audit`

[View full skill →](plugins/audit/skills/audit/SKILL.md)

Pre-sale SEO & AEO maturity assessment from a public URL. No MCP servers needed — just a URL. Produces a client-ready report you can present and use to prepare a proposal.

**What it does:**
- Scores 4 dimensions (Content, Technical, Authority, Measurement) across 5 maturity levels
- Identifies opportunities and quantifies business impact — framed for client conversations
- Surfaces quick wins that show results in week 1
- Provides a roadmap to the next maturity level
- Ends with engagement recommendations — natural bridge to a proposal
- Saves reports per site: `./{domain}/reports/audit-quick-YYYY-MM-DD.md`

**Designed for:** Pre-sale discovery. Run it on a prospect's site, present the findings, close the deal.

**Usage:**
```
/audit https://example.com
```

---

### `/audit:deep`

[View full skill →](plugins/audit-deep/skills/audit-deep/SKILL.md)

Post-sale SEO & AEO maturity audit with GSC + Webflow data. Same maturity model as `/audit` but with data-backed evidence, quantified opportunities, and a phased engagement plan.

**What it does:**
- Full search analytics (90 days): quantifies every opportunity with real traffic data
- CMS analysis: collections, fields, templates, SEO configuration gaps
- Content gap analysis, keyword cannibalization detection, thin content flagging
- Indexation cross-reference (Webflow vs GSC) with data tables
- Prioritized opportunities with estimated business impact
- Phased engagement roadmap: quick wins → optimization → growth
- Appendix with supporting data: metadata audit, content gaps, cannibalization, indexation
- Saves reports per site: `./{domain}/reports/audit-deep-YYYY-MM-DD.md`

**Designed for:** Post-sale engagement baseline. Sets up the work plan with data-backed priorities.

**Usage:**
```
/audit:deep
```

---

### When to use each skill

| Scenario | Skill |
|----------|-------|
| First time setup | `/getting-started` — run once to capture brand voice & goals |
| New blog collection | `/cms-collection-setup:create` — build a fully optimized schema from scratch |
| Audit existing collection schema | `/cms-collection-setup:review` — score coverage, find gaps, add missing fields |
| Find keywords to target (striking distance + new) | `/keywords-opportunity` — full keyword map with volume and action plan |
| Already ranking page 1-2, want more traffic | `/keywords-opportunity:striking` — fastest ROI, positions 4-20 |
| Looking for entirely new topics to target | `/keywords-opportunity:discover` — KE expansion + content gap analysis |
| Pre-sale site assessment (no MCP needed) | `/audit {url}` — client-ready report, proposal-ready |
| Post-sale engagement baseline | `/audit:deep` — data-backed roadmap with GSC + Webflow |
| Low CTR, content is fine | `/click-recovery` — fix the pitch |
| Outdated content, rankings dropping | `/refresh-content` — full refresh |
| Both issues | `/click-recovery` first, then `/refresh-content` |
| Page ranking but not getting clicks (AEO gap) | `/aeo-optimize` — FAQ, schema, direct answer, question H2s |
| Audit AEO score only (no changes) | `/aeo-optimize:audit` — score 12 dimensions, get gap report |
| Monthly performance review | `/monthly-report` — see what changed and what to do next |
| Weekly performance check | `/weekly-report` — weekly pulse with monthly progress tracking |

## About

I'm [Sofian Bettayeb](https://www.checklist-seo.com).

By day, I'm a martech consultant, working with billion-dollar brands like Rolex and Helsana.
By night, I build tools like [AI SEO Copilot](https://webflow.com/apps/detail/ai-seo-copilot) (15k+ installs), [AEO Copilot](https://www.aeo-copilot.com), and blueprints like [Webflow SEO Checklist](https://www.checklist-seo.com) (1k+ downloads) to help my Webflow friends make money with SEO and AEO.

In between, I ride my bikes and play with my kids in Bern, Switzerland.

## Changelog

### v1.6.0 (2026-03-04)

**`/aeo-optimize`** (new)
- Transforms any Webflow page or CMS item into an AEO-ready answer
- Auto-detects primary query from GSC (user confirms before proceeding)
- Supports both CMS items (rich text + schema CMS field) and static pages (text nodes + manual head code)
- 12-dimension AEO audit with 0–24 scoring: query alignment, direct answer, question H2s, bullets/tables/steps, definitions, FAQ, schema, internal links, summary
- Full rewrite proposal shown for approval before any change is made
- JSON-LD schema generation: Article always, FAQPage when FAQ added, HowTo when steps added
- Internal link insertion: 3–5 max, scoped to same section/category, descriptive anchors
- Humanizer applied to all generated content before proposing
- Modes: `/aeo-optimize {url}` (full), `/aeo-optimize:audit {url}` (score only)
- Saves timestamped report: `.claude/reports/{domain}/aeo-optimize-{slug}-YYYY-MM-DD.md`

### v1.5.0 (2026-03-03)

**`/keywords-opportunity`** (new)
- Striking distance analysis: positions 4-30 classified into Tier A (page 1, not top 3), Tier B (page 2), Tier C (page 3)
- Keywords Everywhere enrichment: volume, CPC, competition, trend for every ranked query
- Content gap detection: GSC queries with impressions but position > 30 and no dedicated page
- KE expansion: related and long-tail keyword suggestions for top-ranking topics
- Long-tail cluster consolidation: groups of GSC queries that could be served by one article
- ICE scoring (Impact × Confidence / Effort) with priority buckets (must pursue / high value / worth tracking)
- Decision guide mapping each finding to the right execution skill
- Saves timestamped report per site: `.claude/reports/{domain}/keywords-opportunity-YYYY-MM-DD.md`
- Modes: full (striking + discover), `:striking` (page 1-3 wins), `:discover` (new topics only)
- Cadence guidance: re-run every 4-6 weeks, after `/refresh-content` runs wait 3-4 weeks to measure

### v1.4.0 (2026-03-02)

**`/cms-collection-setup`** (new)
- Review an existing CMS collection against the recommended SEO schema with per-group coverage scoring
- Alias matching — detects common field name variations (e.g., `seo-title`, `meta-title` map to Meta SEO Title)
- Type compatibility check — flags fields that exist but with the wrong type
- Offer to add missing fields to an existing collection after the audit
- Create mode builds a new collection with all Core, Content, Authority, Enhancement, and Social fields
- Handles flexible fields: Author (single Reference or MultiReference), Tags (Option or MultiReference), FAQ (MultiReference or RichText), Pillar page (external or self-reference)
- Skips reference fields gracefully when target collection doesn't exist yet
- Progress tracker during field creation
- Activity log appended after every execution

### v1.3.1 (2026-02-12)

**`/audit`** (updated)
- Reports are now client-facing and proposal-ready
- Findings framed as opportunities with business impact, not pass/fail diagnostics
- No check IDs or implementation details in reports — the "how" is the work you sell
- New sections: About This Assessment, Executive Summary, Quick Wins, Recommended Next Steps
- Engagement options (Deep Audit / Quick Wins First / Full Engagement) replace technical footer
- GSC verification check now accounts for domain-level DNS verification
- Measurement gate updated: any analytics tool (not just GA4/GTM) counts for Level 2

**`/audit:deep`** (updated)
- Reports are now professional client deliverables for engagement baseline
- All opportunities quantified with GSC data (impressions, clicks, positions)
- Phased engagement roadmap: Foundation (weeks 1-2), Optimization (3-6), Growth (7-12)
- New section: Engagement Recommendation with scope, approach, and expected outcomes
- Appendix tables use plain language, no check IDs
- Designed to present to clients and set up a structured work plan

### v1.3.0 (2026-02-12)

**`/audit`** (new)
- Quick SEO & AEO maturity audit from any public URL — no MCP servers needed
- 5-level maturity model across 4 dimensions (Content, Technical, Authority, Measurement)
- Evidence-based checks with deterministic scoring
- Per-site storage: `./{domain}/reports/audit-quick-YYYY-MM-DD.md` + `latest-quick.md`

**`/audit:deep`** (new)
- Deep SEO & AEO maturity audit with GSC + Webflow data
- Same maturity model with deep audit signals and Medium–High confidence
- Content gap analysis, keyword cannibalization detection, thin content flagging
- Indexation cross-reference (Webflow pages vs GSC indexed pages)
- CMS template audit: field completeness, SEO title mapping, schema coverage
- Impact/Confidence/Effort prioritization within each maturity level
- Appendix tables: metadata audit, content gaps, cannibalization, indexation
- Per-site storage: `./{domain}/reports/audit-deep-YYYY-MM-DD.md` + `latest-deep.md`

### v1.2.0 (2026-02-12)

**`/weekly-report`** (new)
- Weekly SEO pulse — 6-section report with executive summary, content performance, technical health, and action plan
- Week-over-week comparison with 4-week trend data
- Per-site report storage: `.claude/reports/{domain}/weekly-report-YYYY-WXX.md`
- Monthly report integration — tracks progress against last monthly report per domain
- Template SEO audit — flags CMS collections using `name` instead of dedicated SEO title field
- Adaptive thresholds — relative (top N%) for small sites, absolute for large sites
- Confidence scoring adjusted for weekly data volumes (capped at 3 for single-week observations)
- Quick mode (`/weekly-report:quick`) for executive summary + action plan only
- Read-only — points to `/click-recovery` and `/refresh-content` for execution

### v1.1.0 (2026-02-12)

**`/monthly-report`** (new)
- Monthly SEO report for founders and executives
- Month-over-month comparison with 3-month trend
- Branded vs non-branded traffic split
- Content performance: top pages, growing, declining, content gaps
- Technical SEO health: indexation cross-reference, metadata audit
- On-page opportunities: CTR optimization, striking distance, keyword mismatches
- Internal linking analysis: orphan pages, under-linked pages, pillar health
- Impact/Confidence/Effort scoring engine with priority buckets
- Read-only — points to `/click-recovery` and `/refresh-content` for execution
- Quick mode (`/monthly-report:quick`) for executive summary + action plan only
- Saves reports to `.claude/reports/monthly-report-YYYY-MM.md`

### v1.0.0 (2026-02-02)

**`/getting-started`**
- Initial release
- Captures business goals, audience, markets
- Brand voice configuration (tone, formality, words to avoid)
- SEO constraints (keywords, competitors, meta title format)
- AEO constraints (citation goals, FAQ strategy, E-E-A-T signals)
- Content preferences (headings, lists, CTAs)
- Saves to `.claude/seo-copilot-config.json`
- Other skills read config automatically

**`/refresh-content`**
- Initial release
- CMS schema auto-mapping
- Conditional guards system
- Internal linking discovery
- FAQ generation with CMS field support
- Schema markup (Article, FAQ, Breadcrumb, HowTo)
- Before/after diff
- Webflow rich text limitations handling

**`/click-recovery`**
- Initial release
- GSC data analysis for CTR opportunities
- Keywords Everywhere integration (optional)
- Tiered prioritization (Tier 1/2/3 + Ranking Opportunities)
- Meta title and description recommendations
- Static page support via `update_page_settings`
- Title/description length warnings
- Direct publishing to Webflow CMS
- Re-check cadence guidance (2-3 weeks, 4-6 weeks, monthly)

## License

MIT
