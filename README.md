# Webflow SEO Copilot

Claude Code skills for Webflow SEO — content refresh, optimization, and ranking recovery.

Built by [Sofian Bettayeb](https://www.checklist-seo.com), Webflow SEO and AEO Expert.

## Skills

### `/refresh-content`

Refresh and optimize existing Webflow blog articles to improve rankings, recover lost traffic, and extend content lifespan.

**What it does:**
- Updates dates, meta titles, and meta descriptions with freshness signals
- Refreshes keywords using GSC data or SERP research
- Discovers and adds internal linking opportunities
- Generates FAQ sections with schema markup
- Adds Article, FAQ, and Breadcrumb structured data
- Audits and populates empty CMS fields
- Presents a before/after diff for review before publishing

**Requirements:**
- Webflow MCP server (required)
- Google Search Console MCP server (optional — enables keyword data and indexing submission)

## Installation

### Claude Code

Add the marketplace and install:

```
/plugin marketplace add sofian-bettayeb/webflow-seo-copilot
/plugin install refresh-content@webflow-seo-copilot
```

### Manual

Copy the skill to your Claude Code skills directory:

```bash
cp -r plugins/refresh-content/skills/refresh-content ~/.claude/skills/refresh-content
```

## Usage

In Claude Code, invoke the skill directly:

```
/refresh-content https://yoursite.com/blog/article-slug
```

Or let Claude detect it automatically when you ask to refresh, update, or optimize a blog article.

## License

MIT
