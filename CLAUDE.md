# Webflow SEO Copilot — Global Instructions

## Activity Logging

These instructions apply to **all Webflow work** — whether using a skill (`/refresh-content`, `/click-recovery`, etc.) or working directly in conversation.

### When starting a session on a Webflow site

1. Identify the site domain (from URL, site name, or Webflow MCP)
2. Check `.claude/reports/{domain}/activity-log.md` for recent history
3. If it exists: read the last 5 entries and use them as context (e.g., avoid re-doing recent work, flag if changes were just made)
4. If it doesn't exist: proceed silently — the log will be created on first write

### When ending a session that included any Webflow work

Append a row to `.claude/reports/{domain}/activity-log.md`.

If the file doesn't exist, create it first:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append:

```
| YYYY-MM-DD | ad-hoc | [one-line summary: what was analyzed, changed, or published] |
```

Use `ad-hoc` as the skill name for work done outside of a named skill. Use the skill name (e.g., `/refresh-content`) for skill-executed work (skills handle their own logging — do not duplicate).

### What counts as Webflow work

Log any session where you:
- Analyzed the site's SEO, performance, or content
- Made changes to pages, CMS items, or settings
- Published the site
- Gave recommendations that the user acted on
- Ran a GSC query for the site

Do **not** log sessions that were purely informational with no action taken.

### Log format

Keep summaries to one line. Include:
- What was done (e.g., "Updated H1, intro text, hero image alt on homepage")
- Key pages or keywords affected (e.g., "Target: 'Webflow SEO Checklist'")
- Outcome (e.g., "Published to checklist-seo.com")

Example rows:
```
| 2026-02-23 | ad-hoc | Fixed keyword cannibalization: retargeted /webflow page. Updated homepage H1, intro, hero alt. Target: 'Webflow SEO Checklist'. Published. |
| 2026-02-20 | /click-recovery | Updated meta titles on 5 pages. Avg position 8–15. Published to checklist-seo.com. |
| 2026-02-15 | /weekly-report | Week W07. Health: STABLE. 1 must-fix, 2 high-impact. Report saved. |
```
