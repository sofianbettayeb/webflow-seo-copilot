# Webflow SEO Copilot — Global Instructions

---

## Shared Skill Conventions

All skills follow these conventions. Each skill references them instead of duplicating the logic.

### MCP Discovery (run once at skill start)

Search for tools in this order. Note availability — never stop execution because a tool is missing unless it's marked required for that skill.

```
Webflow MCP     → search: +webflow data cms
GSC MCP         → search: +gsc search analytics
Keywords Everywhere → search: +keywords everywhere volume
```

**Webflow MCP absent:** Skills that analyze content proceed in read-only mode. Skills that publish warn the user and skip the publish phase — they never stop analysis because of this.

**Keywords Everywhere absent:** Skills that use it for topic discovery should prompt the user once:
> "Keywords Everywhere isn't connected — I can still suggest topics, but won't have search volume to validate them. Connect it for volume-backed recommendations, or continue without it?"
> Options: 1. Connect now (link to install) 2. Continue without it

Do not silently skip topic discovery. Do not ask again after the user decides.

---

### Config Loading (run once at skill start)

Load `.claude/seo-copilot-config.json` if it exists. Extract what the skill needs — each skill specifies which fields it uses. If the file doesn't exist, proceed with defaults and note once: "No config found. Run `/getting-started` to personalize recommendations."

Do not block execution on missing config.

---

### Priority Buckets (use instead of scoring formula)

All skills use three buckets with observable, binary criteria. Do not compute a numeric score.

**Must do** — at least one of:
- A page in the sitemap is returning a 404
- GSC confirms the same keyword ranking on 2+ pages simultaneously
- A page getting impressions has no `<title>` tag
- An indexation error on a page with existing traffic
- A cannibalization pair confirmed by GSC data (not inferred)

**High value** — at least one of:
- A keyword cluster with 50+ monthly impressions (or in top 20% of site's keyword set) has no matching page
- A page's CTR is less than half the expected rate for its average position
- The primary keyword for a page doesn't appear in its title
- A cluster has 2+ support posts but no pillar page
- A page that exists in the site architecture is missing from GSC entirely

**Nice to have** — everything else:
- Coverage gaps on existing pages (missing sub-topics)
- Metadata length issues (no traffic impact confirmed)
- Orphan pages with no internal links
- Programmatic pages with thin content

Items that don't meet "Nice to have" criteria are excluded from the action plan.

---

### Activity Log (run at skill end)

After every skill execution, append to `.claude/reports/{domain}/activity-log.md`.

If the file doesn't exist, create it:
```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Append one row:
```
| YYYY-MM-DD | /skill-name | [one-line summary of what was analyzed, changed, or published] |
```

Log even on early exits — note the reason (e.g., "Aborted: GSC MCP unavailable").

---

### Abort Guard (applies to all skills)

If user says "stop", "cancel", "abort", or "nevermind":
- Confirm: "Stop the workflow? Progress will be lost."
- If confirmed: exit cleanly, output any partial results already computed.

---

### Output Format Guidance

Skills specify the **intent** of each output section, not the exact format. Adapt format to the data available:
- If a table would have more than half its cells empty, use a list instead
- If there are fewer than 3 items in a section, collapse it into a sentence
- Never output a section header with nothing under it
- Prefer specificity over completeness — one concrete finding beats five vague ones

---

## Activity Logging (ad-hoc sessions)

For work done outside a named skill, append a row using `ad-hoc` as the skill name.

Log any session where you:
- Analyzed the site's SEO, performance, or content
- Made changes to pages, CMS items, or settings
- Published the site
- Gave recommendations the user acted on
- Ran a GSC query for the site

Do **not** log sessions that were purely informational with no action taken.

Example rows:
```
| 2026-02-23 | ad-hoc | Fixed keyword cannibalization: retargeted /webflow page. Updated homepage H1, intro, hero alt. Target: 'Webflow SEO Checklist'. Published. |
| 2026-02-20 | /click-recovery | Updated meta titles on 5 pages. Avg position 8–15. Published to checklist-seo.com. |
| 2026-02-15 | /weekly-report | Week W07. Health: STABLE. 1 must-fix, 2 high-impact. Report saved. |
```
