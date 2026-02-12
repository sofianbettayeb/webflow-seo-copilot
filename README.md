# Webflow SEO Copilot

Claude Code skills that turn your Webflow CMS into an SEO machine — refresh content, optimize rankings, and recover lost traffic, all through natural language.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add sofianbettayeb/webflow-seo-copilot

# Install skills
/plugin install getting-started@webflow-seo-copilot
/plugin install refresh-content@webflow-seo-copilot
/plugin install click-recovery@webflow-seo-copilot
/plugin install monthly-report@webflow-seo-copilot
/plugin install weekly-report@webflow-seo-copilot

# First time? Run setup to capture your brand voice & SEO goals
/getting-started

# Then use the SEO skills
/refresh-content https://yoursite.com/blog/article-slug
/click-recovery
/monthly-report
/weekly-report
```

**Requirements:**
- [Webflow MCP server](https://developers.webflow.com/mcp/reference/overview) (required)
- [Google Search Console MCP server](https://github.com/sofianbettayeb/gsc-mcp-server) (required for `/click-recovery`, `/monthly-report`, and `/weekly-report`, optional for `/refresh-content`)
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
- Finds content gaps — queries with high impressions but no matching page
- Cross-references Webflow pages with GSC index status
- Audits metadata (missing, duplicate, too long/short)
- Flags high-impression/low-CTR pages and striking distance keywords
- Analyzes internal linking structure and orphan pages
- Scores every recommendation by Impact, Confidence (capped for weekly data), and Effort
- Links back to last monthly report for progress tracking
- Outputs a prioritized action plan in 3 buckets: Must fix, High impact, Nice to have

**This skill is read-only** — it never modifies Webflow content. It points you to `/click-recovery` and `/refresh-content` for execution.

**Usage:**
```
/weekly-report
/weekly-report:quick
```

The full report covers 7 sections: executive summary (with monthly context), performance overview, content performance, technical health, on-page opportunities, internal linking, and action plan. Quick mode outputs only the executive summary and action plan.

---

### When to use each skill

| Scenario | Skill |
|----------|-------|
| First time setup | `/getting-started` — run once to capture brand voice & goals |
| Low CTR, content is fine | `/click-recovery` — fix the pitch |
| Outdated content, rankings dropping | `/refresh-content` — full refresh |
| Both issues | `/click-recovery` first, then `/refresh-content` |
| Monthly performance review | `/monthly-report` — see what changed and what to do next |
| Weekly performance check | `/weekly-report` — weekly pulse with monthly progress tracking |

## About

I'm [Sofian Bettayeb](https://www.checklist-seo.com).

By day, I'm a martech consultant, working with billion-dollar brands like Rolex and Helsana.
By night, I build tools like [AI SEO Copilot](https://webflow.com/apps/detail/ai-seo-copilot) (15k+ installs), [AEO Copilot](https://www.aeo-copilot.com), and blueprints like [Webflow SEO Checklist](https://www.checklist-seo.com) (1k+ downloads) to help my Webflow friends make money with SEO and AEO.

In between, I ride my bikes and play with my kids in Bern, Switzerland.

## Changelog

### v1.2.0 (2026-02-12)

**`/weekly-report`** (new)
- Weekly SEO pulse with same 7-section depth as monthly report
- Week-over-week comparison with 4-week trend data
- Monthly report integration — tracks progress against last monthly report
- Confidence scoring adjusted for weekly data volumes (capped at 3 for single-week observations)
- Thresholds scaled for weekly volumes (content gaps >100 impressions, low traffic <250)
- Quick mode (`/weekly-report:quick`) for executive summary + action plan only
- Saves reports to `.claude/reports/weekly-report-YYYY-WXX.md`
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
