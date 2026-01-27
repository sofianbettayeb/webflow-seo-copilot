# Webflow SEO Copilot

Claude Code skills that turn your Webflow CMS into an SEO machine — refresh content, optimize rankings, and recover lost traffic, all through natural language.

## Installation

```
/plugin marketplace add sofianbettayeb/webflow-seo-copilot
/plugin install refresh-content@webflow-seo-copilot
```

**Requirements:**
- [Webflow MCP server](https://github.com/anthropics/skills) (required)
- Google Search Console MCP server (optional — enables keyword data and indexing submission)

## Skills

### `/refresh-content`

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

Invoke it directly:
```
/refresh-content https://yoursite.com/blog/article-slug
```

Or just ask Claude to refresh, update, or optimize a blog article — the skill activates automatically.

### More skills coming soon

This marketplace will grow with more Webflow SEO skills. Stay tuned.

## About

I'm [Sofian Bettayeb](https://www.checklist-seo.com).

By day, I'm a martech consultant, working with billion-dollar brands like Rolex and Helsana. 
By night, I build tools like [AI SEO Copilot](https://webflow.com/apps/detail/ai-seo-copilot) (15k+ installs), [AEO Copilot](https://www.aeo-copilot.com), and blueprints like [Webflow SEO Checklist](https://www.checklist-seo.com) (1k+ downloads) to help my Webflow friends make money with SEO and AEO.

In between, I ride my bikes and play with my kids in Bern, Switzerland.

## License

MIT
