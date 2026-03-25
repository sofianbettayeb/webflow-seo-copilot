# Webflow SEO Copilot

Claude Code skills for Webflow SEO — keyword research, content creation, traffic recovery, and weekly/monthly reporting, all through natural language.

## Requirements

| MCP Server | Required for |
|------------|-------------|
| [Webflow MCP](https://developers.webflow.com/mcp/reference/overview) | All skills that touch Webflow content |
| [Google Search Console MCP](https://github.com/sofianbettayeb/gsc-mcp-server) | `/click-recovery`, `/weekly-report`, `/monthly-report`, `/audit:deep`, optional for `/refresh-content` |
| [Keywords Everywhere MCP](https://github.com/hithereiamaliff/mcp-keywords-everywhere) | Optional — adds search volume and intent data to several skills |

`/audit {url}` requires no MCP servers — any public URL works.

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add sofianbettayeb/webflow-seo-copilot

# Install all skills
/plugin install getting-started@webflow-seo-copilot
/plugin install cms-collection-setup@webflow-seo-copilot
/plugin install keywords-opportunity@webflow-seo-copilot
/plugin install write-blog@webflow-seo-copilot
/plugin install refresh-content@webflow-seo-copilot
/plugin install click-recovery@webflow-seo-copilot
/plugin install aeo-optimize@webflow-seo-copilot
/plugin install weekly-report@webflow-seo-copilot
/plugin install monthly-report@webflow-seo-copilot
/plugin install audit@webflow-seo-copilot
/plugin install audit-deep@webflow-seo-copilot
/plugin install topic-map@webflow-seo-copilot

# Run setup first — captures brand voice, SEO goals, and activates activity logging
/getting-started
```

## Skills

### `/getting-started`

One-time setup. Captures your business context, brand voice, audience, and SEO/AEO constraints — saves to `.claude/seo-copilot-config.json`. All other skills read this file automatically. Re-run anytime to update settings.

[Full skill docs →](plugins/getting-started/skills/getting-started/SKILL.md)

---

### `/cms-collection-setup`

Audit or create a Webflow CMS collection for SEO. Review mode scores an existing collection field-by-field (Core, Content, Authority, Enhancement, Social) and offers to add missing fields. Create mode builds a new collection with all recommended fields from scratch.

```
/cms-collection-setup          — guided: choose review or create
/cms-collection-setup:review   — audit an existing collection
/cms-collection-setup:create   — build a new collection
```

[Full skill docs →](plugins/cms-collection-setup/skills/cms-collection-setup/SKILL.md)

---

### `/keywords-opportunity`

Find keywords worth targeting. Pulls 90 days of GSC data, classifies rankings into striking distance tiers (positions 4–30), identifies content gaps, and expands into new topics via Keywords Everywhere. Every opportunity is scored by Impact × Confidence / Effort and routed to the right execution skill. Saves a timestamped report referenced by `/weekly-report` and `/monthly-report`.

```
/keywords-opportunity             — full: striking distance + new discovery
/keywords-opportunity:striking    — page 1-3 wins only (fastest ROI)
/keywords-opportunity:discover    — new keywords not yet targeted
```

[Full skill docs →](plugins/keywords-opportunity/skills/keywords-opportunity/SKILL.md)

---

### `/write-blog`

Create a new SEO and AEO-optimized blog post in Webflow CMS. Runs a brief intake, then recommends the best article format for your keyword and intent — tutorial, direct comparison, alternatives page, listicle, case study, pillar page, and more. Expands keywords, proposes an outline for approval, writes a full draft, applies an AI tells check and humanizer pass, runs an AEO audit, then creates the CMS item. Checks the activity log to avoid duplicate content on the same topic.

```
/write-blog                    — full workflow: brief → format recommendation → outline → draft → audit → CMS draft
/write-blog:outline            — stops after outline approval, no Webflow changes
/write-blog:publish            — creates and publishes immediately after approval
```

[Full skill docs →](plugins/write-blog/skills/write-blog/SKILL.md)

---

### `/refresh-content`

Refresh and optimize an existing Webflow blog article. Updates dates, meta, and keywords; rewrites outdated sections; adds internal links, FAQ, and structured data. Presents a before/after diff for review before publishing.

```
/refresh-content https://yoursite.com/blog/article-slug
```

[Full skill docs →](plugins/refresh-content/skills/refresh-content/SKILL.md)

---

### `/click-recovery`

Fix pages Google already ranks but users don't click. Pulls GSC data to find high-impression, low-CTR pages, proposes new meta titles and descriptions, and publishes approved changes directly to Webflow.

```
/click-recovery
```

[Full skill docs →](plugins/click-recovery/skills/click-recovery/SKILL.md)

---

### `/aeo-optimize`

Optimize any Webflow page for AEO — AI answer engines and featured snippets. Scores 12 dimensions (query alignment, direct answer, question H2s, FAQ, schema, internal links, and more), proposes a full rewrite for approval, then applies changes via MCP.

```
/aeo-optimize https://yoursite.com/blog/article-slug   — full optimization
/aeo-optimize:audit https://yoursite.com/blog/...      — score only, no changes
```

[Full skill docs →](plugins/aeo-optimize/skills/aeo-optimize/SKILL.md)

---

### `/weekly-report`

Weekly SEO pulse. Compares Week W vs W-1, tracks progress against the last monthly report, audits CMS templates and metadata, and scores every recommendation. Read-only — points to `/click-recovery` and `/refresh-content` for execution.

```
/weekly-report         — full 6-section report
/weekly-report:quick   — executive summary + action plan only
```

[Full skill docs →](plugins/weekly-report/skills/weekly-report/SKILL.md)

---

### `/monthly-report`

Monthly SEO review for founders and operators. Compares Month M vs M-1 with 3-month trend, surfaces content gaps, technical issues, and on-page opportunities, and outputs a prioritized action plan in three buckets: Must fix, High impact, Nice to have. Read-only.

```
/monthly-report         — full 7-section report
/monthly-report:quick   — executive summary + action plan only
```

[Full skill docs →](plugins/monthly-report/skills/monthly-report/SKILL.md)

---

### `/audit`

Pre-sale SEO & AEO maturity assessment from any public URL. No MCP servers needed. Scores 4 dimensions (Content, Technical, Authority, Measurement) across 5 maturity levels, surfaces quick wins, and ends with engagement recommendations — ready to present to a prospect.

```
/audit https://example.com
```

[Full skill docs →](plugins/audit/skills/audit/SKILL.md)

---

### `/audit:deep`

Post-sale engagement baseline with GSC + Webflow data. Same maturity model as `/audit` but data-backed: 90-day search analytics, CMS analysis, content gaps, keyword cannibalization, indexation cross-reference, and a phased roadmap (quick wins → optimization → growth). Works with Webflow, WordPress, and any CMS platform.

```
/audit:deep
```

[Full skill docs →](plugins/audit-deep/skills/audit-deep/SKILL.md)

---

### `/topic-map`

Turn keyword exports, URL lists, and GSC data into a structured keyword map. Outputs one table per cluster showing which page targets which keyword, page type (Hub / Editorial / Product), and status (existing / missing / orphan). Surfaces editorial→product linking gaps, cannibalization risks, and a 90-day editorial calendar. Works from any combination of inputs — paste URLs and keywords, or connect Webflow MCP and GSC for auto-fetch.

```
/topic-map                  — full: cluster tables, gap analysis, editorial calendar
/topic-map:audit            — existing structure only, no gap analysis
/topic-map:quick            — cluster tables + top priorities, no editorial calendar
```

[Full skill docs →](plugins/topic-map/skills/topic-map/SKILL.md)

---

## When to use which skill

| Scenario | Skill |
|----------|-------|
| First-time setup | `/getting-started` |
| New blog collection | `/cms-collection-setup:create` |
| Audit existing collection schema | `/cms-collection-setup:review` |
| Find keywords to target | `/keywords-opportunity` |
| Fast wins on rankings already page 1–2 | `/keywords-opportunity:striking` |
| Discover entirely new topics | `/keywords-opportunity:discover` |
| Write a new blog post | `/write-blog` |
| Refresh an outdated article | `/refresh-content` |
| Low CTR, content is fine | `/click-recovery` |
| Both CTR and content issues | `/click-recovery` first, then `/refresh-content` |
| Optimize for AI answer engines | `/aeo-optimize` |
| AEO score only, no changes | `/aeo-optimize:audit` |
| Weekly SEO check | `/weekly-report` |
| Monthly performance review | `/monthly-report` |
| Pre-sale site assessment (no MCP) | `/audit {url}` |
| Post-sale engagement baseline | `/audit:deep` |
| Map which keyword each page targets | `/topic-map` |
| Find missing pages and topic gaps | `/topic-map` |
| Visualize hub → editorial → product structure | `/topic-map` |
| Quick structure check, no calendar | `/topic-map:quick` |

## How skills work together

Every skill reads the activity log (`.claude/reports/{domain}/activity-log.md`) at startup to avoid redundant work and surface recent context. `/keywords-opportunity` saves a report that `/weekly-report` and `/monthly-report` reference automatically. `/write-blog` and `/refresh-content` both feed into `/aeo-optimize`. `/monthly-report` carries open action items forward from the last 4 weekly reports.

Recommended weekly cadence:
1. `/weekly-report` — pulse check, spot what changed
2. `/click-recovery` — fix CTR issues flagged in the report
3. `/refresh-content` — update declining content
4. `/write-blog` — create content for new keyword opportunities

## About

Built by [Sofian Bettayeb](https://www.sofianbettayeb.com) — Marketing expert, creator of [AI SEO Copilot](https://webflow.com/apps/detail/ai-seo-copilot) and [AEO Copilot](https://www.aeo-copilot.com), author of [Webflow SEO Checklist](https://www.checklist-seo.com).

## License

MIT
