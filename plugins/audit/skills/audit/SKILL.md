---
name: audit
version: "1.1"
description: |
  Quick SEO & AEO maturity assessment from a public URL. Pre-sale discovery tool that scores 4 dimensions across 5 maturity levels. Produces a client-ready report with opportunities, quick wins, and a roadmap — designed to present findings and prepare a proposal.
  Triggers: audit, site audit, seo audit, aeo audit, discovery audit.
  Requires: Web access (WebFetch). No MCP servers needed.
  Workflow: Parse → Prompt → Fetch → Check → Score → Report → Save.
  Command: /audit {url}
---

# Quick Audit Skill

Pre-sale SEO and AEO maturity assessment from a public URL. Runs evidence-based checks on public signals, scores 4 dimensions using the AEO Maturity Model, and outputs a client-ready report with prioritized opportunities and a roadmap to the next level.

This report is designed to present to a client and prepare a proposal. It frames findings as opportunities, quantifies business impact, and leads naturally to an engagement recommendation.

This skill is **read-only** and requires no MCP servers — just a URL.

## Prerequisites

- **Required**: Web access (WebFetch tool)
- **No MCP servers needed** — quick audit runs on public signals only

## Maturity Model Reference

See `references/maturity-model.md` for the full rubric. Summary:

| Level | Name | One-liner |
|-------|------|-----------|
| 1 | Keywords | Basic keyword SEO. Titles target keywords, content exists. |
| 2 | Answers | Content answers questions. FAQs and intent-matching headers. |
| 3 | Structure | Topic clusters, internal linking, full schema. |
| 4 | Pillar | Authority on core topics. AI engines cite the content. |
| 5 | Authority | Programmatic AEO. Entity recognition. AI citations. |

4 dimensions: **Content**, **Technical**, **Authority**, **Measurement**.
Overall level = minimum dimension score (weakest link).

---

## Critical Guards

⚡ GUARD — **URL not accessible:**
- If WebFetch fails: "Could not fetch {url}. Check the URL is correct and publicly accessible."
- Stop execution.

⚡ GUARD — **Not a Webflow site:**
- Still run the audit — the maturity model applies to any site.
- Note: "This doesn't appear to be a Webflow site. Webflow-specific recommendations may not apply."

⚡ GUARD — **User requests abort:**
- Confirm, exit cleanly, output any partial results.

---

## Phase 0: PARSE & PROMPT

### 0.1 Parse URL & Set {domain}

Extract the domain from the provided URL. Normalize: strip `www.`, lowercase.
- `https://www.checklist-seo.com/blog/guide` → `checklist-seo.com`

The `{domain}` is used for:
- **Report save path**: `./{domain}/reports/audit-quick-YYYY-MM-DD.md`
- **Latest pointer**: `./{domain}/reports/latest-quick.md`
- **Activity log**: `./{domain}/reports/activity-log.md`

Determine the homepage URL from the domain (for sitemap and site-wide checks).

### 0.2 Review Activity Log

Check `./{domain}/reports/activity-log.md`:
- If it exists: show recent activity summary (last 10 entries)
- **Redundancy check**: if `/audit` was run in the last 7 days → warn: "You ran `/audit` on [date]. Run again anyway?"
- If not found: proceed silently

### 0.3 Prompt for Optional URLs

After parsing `{url}`, prompt:

```
Optional inputs (press Enter to skip each):

1. Topic pillar URL — a main topic page (e.g., /blog/seo-guide)
2. Blog post example URL — a typical blog article
```

Do not ask for credentials. Quick audit runs on public signals only.

---

## Phase 1: FETCH

Fetch these resources in a deterministic order:

| # | Resource | How | Required |
|---|----------|-----|----------|
| 1 | Home page HTML | WebFetch `https://{domain}/` | Yes — stop if fails |
| 2 | Pillar page HTML | WebFetch pillar URL (if provided) | No |
| 3 | Blog post HTML | WebFetch blog URL (if provided) | No |
| 4 | sitemap.xml | WebFetch `https://{domain}/sitemap.xml` | No — try `/sitemap_index.xml` as fallback |
| 5 | robots.txt | WebFetch `https://{domain}/robots.txt` | No |
| 6 | PageSpeed | Run PageSpeed via MCP on home URL (and pillar if provided) | No — skip if unavailable |

For each fetch, record: URL attempted, success/fail, HTTP status if available.

---

## Phase 2: CHECK

Run evidence-based checks on the fetched HTML. Every check produces:
- **Pass/Fail** (boolean)
- **Evidence** (exact snippet, attribute value, or count)
- **Source URL** (which page the evidence came from)

No heuristic language in pass/fail. Heuristics may only appear in recommendations.

**IMPORTANT:** Check IDs (C1, T4, etc.) are for internal skill logic only. They must NEVER appear in the client-facing report. The report uses plain language descriptions instead.

### Content Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| C1 | Title exists | `<title>` tag present and non-empty | Tag content |
| C2 | Exactly one H1 | Count of `<h1>` elements = 1 | Count found, text of H1(s) |
| C3 | Heading hierarchy | No skipped levels (H1→H2→H3, not H1→H3) | First violation if any |
| C4 | FAQ section present | Regex match for "FAQ", "Frequently Asked", or ≥3 question-pattern headings (`?` in H2/H3/H4) | Matched text |
| C5 | Word count ≥ 500 | Visible text word count (strip nav/footer/scripts) | Exact count |
| C6 | Word count ≥ 800 | Same as above, higher threshold | Exact count |
| C7 | Word count ≥ 2000 | Same, pillar-level threshold | Exact count |
| C8 | Internal links present | Count of `<a href>` pointing to same host | Count |
| C9 | Related posts pattern | Links/section matching "related", "you may also like", "more articles" | Matched pattern |
| C10 | Freshness date detected | `<time>` tag, "Last updated", "Published", or structured date pattern (YYYY-MM-DD) in article body | Date found |

### Technical Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| T1 | HTTPS active | URL scheme is `https` | URL |
| T2 | Viewport meta present | `<meta name="viewport">` exists | Tag content |
| T3 | Canonical link present | `<link rel="canonical">` exists | Href value |
| T4 | Open Graph tags present | `<meta property="og:title">` at minimum | Tags found |
| T5 | JSON-LD present | `<script type="application/ld+json">` exists | Schema types detected |
| T6 | Schema: Organization | JSON-LD contains `@type: Organization` | Snippet |
| T7 | Schema: Article | JSON-LD contains `@type: Article` (or subtypes) | Snippet |
| T8 | Schema: FAQPage | JSON-LD contains `@type: FAQPage` | Snippet |
| T9 | Schema: BreadcrumbList | JSON-LD contains `@type: BreadcrumbList` | Snippet |
| T10 | Schema: Person | JSON-LD contains `@type: Person` | Snippet |
| T11 | robots.txt accessible | Fetch succeeded, not empty | Content summary |
| T12 | robots.txt not blocking key paths | No `Disallow: /` for major user agents | Disallow rules found |
| T13 | Sitemap accessible | sitemap.xml fetched and parseable | URL count if available |
| T14 | Clean URL structure | No query-string parameters on core pages (home, pillar, blog) | URLs checked |

### Authority Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| A1 | Author byline present | Text matching "By [Name]", `<span class="author">`, or similar patterns on blog post (if provided) or homepage | Matched text |
| A2 | About/team page discoverable | Internal link containing "about", "team", or "company" in href or text | Link found |
| A3 | Author bio markers | Credentials, role, or experience mentions near author name ("CEO", "years of experience", "certified") | Matched text |
| A4 | External citations present | Outbound links to non-social, non-navigation domains on content pages | Count + domains |

### Measurement Checks

| # | Check | Rule | Evidence |
|---|-------|------|----------|
| M1 | GA4 detected | Script containing `gtag(` or GA measurement ID pattern (`G-XXXXXXX`) | Script snippet |
| M2 | GTM detected | Script containing `GTM-` container ID | Container ID |
| M3 | GSC verification | `<meta name="google-site-verification">` present | Tag content |
| M4 | Other analytics tools | Patterns for PostHog (`posthog`), Segment (`analytics.js`), Hotjar, Mixpanel, Plausible, Fathom | Tools detected |

**M3 limitation:** This check only detects HTML meta tag verification. Domain-level GSC properties use DNS TXT records, which are not visible in page HTML. If M3 fails, note in the report: "GSC verification not detected via HTML. If verified via DNS (domain-level property), this check does not apply." Do not penalize the score when there is reason to believe DNS verification is in use (e.g., analytics scripts suggest GSC awareness, or client confirms domain-level property).

---

## Phase 3: SCORE

Apply the maturity model rubric. For each dimension, evaluate level gates in order (1→5). Stop at the first level that fails.

### Scoring Engine

```
score_dimension(dimension, evidence):
  for level in [1, 2, 3, 4, 5]:
    if all gates for this level pass:
      continue
    else:
      return level - 1 (or 1 if level 1 fails)
  return 5
```

### Content Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | C1 (title exists) AND C2 (exactly one H1) |
| 2 | Level 1 AND C4 (FAQ present) AND C6 (≥800 words) AND C3 (heading hierarchy ok) |
| 3 | Level 2 AND C8 (internal links ≥ 5) AND C9 (related posts pattern) |
| 4 | Level 3 AND C7 (≥2000 words on pillar page, if provided) AND C10 (freshness date) |
| 5 | Level 4 AND freshness date within last 6 months AND entity-rich markup detected |

### Technical Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | T1 (HTTPS) AND T2 (viewport) AND C1 (title exists) |
| 2 | Level 1 AND T3 (canonical) AND T13 (sitemap accessible) AND T4 (OG tags) |
| 3 | Level 2 AND T5 (JSON-LD present) AND T7 (Article schema) |
| 4 | Level 3 AND T6 (Organization schema) AND T10 (Person schema) AND T9 (BreadcrumbList) |
| 5 | Level 4 AND SameAs links to authoritative profiles AND comprehensive structured data |

### Authority Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline — no author signals = Level 1) |
| 2 | A1 (author byline) AND A2 (about/team page discoverable) |
| 3 | Level 2 AND A3 (author bio with credentials) AND A4 (external citations ≥ 1) |
| 4 | Level 3 AND strong E-E-A-T throughout (experience language, data citations, multiple evidence types) |
| 5 | Level 4 AND knowledge panel signals AND multiple platform authority |

### Measurement Dimension Gates

| Level | Gates (all must pass) |
|-------|----------------------|
| 1 | Always passes (baseline — no tracking = Level 1) |
| 2 | (M1 OR M2 OR M4) (any analytics installed) AND M3 (GSC verification — see M3 limitation note) |
| 3 | Level 2 AND multiple tracking tools detected (≥ 2 of M1, M2, M4) |
| 4 | Level 3 AND advanced tracking setup (multiple measurement tools visible) |
| 5 | Level 4 AND enterprise-grade measurement stack |

### Overall Score

```
overall_level = min(content_score, technical_score, authority_score, measurement_score)
confidence = "Low-Medium" (always, for quick mode)
```

---

## Phase 4: REPORT & SAVE

### 4.1 Report Principles

The report is a **client-facing deliverable** designed to present findings and prepare a proposal. Follow these rules:

1. **No check IDs** — Never write C1, T4, M3, etc. in the report. Use plain language: "page title", "Open Graph tags", "search console verification".
2. **Opportunities, not failures** — Frame findings as untapped potential, not broken things. "You're leaving X on the table" not "X is broken".
3. **No implementation details** — Say **what** needs attention and **why** it matters. Never say **how** to fix it (e.g., don't write "In Webflow Designer, go to Page Settings → Open Graph"). The how is the work you sell.
4. **Business language** — Traffic, visibility, clicks, rankings, competitive advantage. Not "pass/fail", "boolean", "gates".
5. **Lead to engagement** — The report should make the client want to work with you. End with a clear next step.

### 4.2 Report Structure

Output a single markdown file with these sections in order:

---

**Header:**
```
# SEO & AEO Maturity Assessment — {domain}

**Prepared for:** {domain}
**Date:** YYYY-MM-DD
**Assessment type:** Discovery (public signals)
**Confidence:** Low–Medium (based on publicly available data)
**Pages analyzed:** [list URLs checked]
```

---

**1. About This Assessment**

Brief methodology section (3-4 sentences):
- What was analyzed (homepage, pillar page if provided, blog post if provided, sitemap, robots.txt)
- How scoring works (4 dimensions, 5 maturity levels, evidence-based)
- What this assessment covers and what it doesn't (public signals only — no search analytics, no CMS data, no traffic numbers)
- Confidence caveat: "A deep audit with search analytics and CMS access would increase confidence and reveal data-backed opportunities."

---

**2. Executive Summary**

3-5 sentences that a decision-maker can read and understand immediately:
- Current maturity level and what it means in plain language
- The single biggest opportunity (framed as upside, not problem)
- What's already strong (give credit — builds trust)
- One sentence on what reaching the next level would unlock

---

**3. SEO & AEO Maturity Scorecard**

Overall level (number + name) with a one-line interpretation.

Table with NO check IDs:

| Dimension | Score | Level | What's Working | Biggest Opportunity |
|-----------|-------|-------|----------------|---------------------|
| Content | X/5 | Name | [strength] | [opportunity] |
| Technical | X/5 | Name | [strength] | [opportunity] |
| Authority | X/5 | Name | [strength] | [opportunity] |
| Measurement | X/5 | Name | [strength] | [opportunity] |

Below the table:
- Strongest dimension and why it matters
- Weakest dimension and what it's costing

---

**4. Key Findings**

For each dimension, write a **narrative paragraph** (not a pass/fail table):
- Lead with what's working (builds credibility, shows you're fair)
- Then describe what's missing and why it matters in business terms
- Use evidence (quotes, counts, schema types found) but never check IDs
- End each dimension with: what reaching the next level would require (outcome, not how-to)

Example tone: "The site has a strong FAQ section with 11 question-answer pairs and proper schema markup — this is exactly what AI answer engines look for when selecting sources. However, there's no mechanism for visitors to discover related content, which means each page visit is a dead end instead of a journey deeper into the site."

---

**5. Opportunities & Business Impact**

Table of the top 3-5 opportunities, framed as upside:

| # | Opportunity | Impact | What You're Leaving on the Table |
|---|-------------|--------|----------------------------------|
| 1 | [opportunity] | High/Medium/Low | [business consequence in plain language] |

Focus on:
- Traffic and visibility impact
- Competitive disadvantage
- AI/AEO readiness gaps
- User experience consequences

Do NOT include how to fix these — that's the engagement.

---

**6. Quick Wins**

2-3 items that can show visible results within the first week of an engagement. These build client confidence that progress is immediate.

For each quick win:
- What it is (outcome, not implementation)
- Why it matters (business impact)
- Effort level: Low (hours, not days)

Frame as: "These are the first things we'd address — you'll see changes within days, not months."

---

**7. Roadmap to Next Level**

Current level → target level (current + 1).

Table showing what each dimension needs to advance:

| Dimension | Current | Target | What's Needed | Effort |
|-----------|---------|--------|---------------|--------|
| [dim] | X | X+1 | [outcome description] | Low/Medium/High |

Below the table: a brief narrative on what reaching the next level means for the business (more visibility, better AI coverage, competitive positioning).

---

**8. Recommended Next Steps**

This is the proposal bridge. 2-3 clear options:

**Option A: Deep Audit** (recommended if GSC + Webflow access available)
- "With access to your search analytics and CMS, we can quantify these opportunities with traffic data, identify content gaps, and build a prioritized engagement plan with ROI estimates."

**Option B: Quick Wins First**
- "Start with the quick wins identified above, then reassess. This is a good option if you want to see results before committing to a larger engagement."

**Option C: Full Engagement**
- "Address all opportunities in a structured roadmap — quick wins in week 1, strategic improvements over 4-8 weeks, with weekly progress tracking."

End with a single call-to-action line.

---

### 4.3 Save to File

Save two files:
- `./{domain}/reports/audit-quick-YYYY-MM-DD.md` (timestamped)
- `./{domain}/reports/latest-quick.md` (overwrite each run — other skills read this)

Create directories if needed: `mkdir -p ./{domain}/reports/`

### 4.4 Activity Log

Append to `./{domain}/reports/activity-log.md`:

```
| YYYY-MM-DD | /audit | Quick audit. Overall Level: X (Name). Content: X, Technical: X, Authority: X, Measurement: X. |
```

Log even on early exit (e.g., "Aborted: URL not accessible").

---

## Integration with Other Skills

| Finding | Skill | When |
|---------|-------|------|
| Need data-backed audit | `/audit:deep` | After connecting GSC + Webflow |
| Low CTR / bad meta tags | `/click-recovery` | After connecting GSC |
| Outdated content | `/refresh-content {url}` | For specific pages |
| Missing config | `/getting-started` | First-time setup |
| Ongoing monitoring | `/weekly-report` | After fixing issues |
