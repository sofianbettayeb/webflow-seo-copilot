# Webflow AEO Maturity Model

5 levels, 4 dimensions. Each dimension is scored 1-5 per level. Overall level = lowest dimension score (weakest link determines maturity).

---

## Levels

| Level | Name | Description |
|-------|------|-------------|
| 1 | Keywords | Basic on-page keyword SEO. Titles and H1s target keywords. Content exists but isn't structured for answers. |
| 2 | Answers | Content answers real questions. FAQs exist. Headers match search intent. Some schema markup. |
| 3 | Structure | Systematic topic coverage. Internal linking strategy. AI-friendly site structure. Full schema implementation. |
| 4 | Pillar | Acknowledged authority on core topics. Pillar/cluster architecture. AI engines cite the content. |
| 5 | Authority | Programmatic AEO. Continuous adaptation. Entity recognition. Consistent AI citations across platforms. |

---

## Dimension Scoring Rubrics

### Content Dimension

| Level | Criteria | URL Audit Signals | Deep Audit Signals |
|-------|----------|-------------------|-------------------|
| 1 | Keywords in title/H1. Thin or generic content. No FAQs. | Title contains target keyword. <500 words visible. No FAQ section. | GSC shows <5 ranking queries per page. Thin content across CMS. |
| 2 | Answers search questions. FAQ sections present. Headers use question format. | FAQ/Q&A sections visible. H2/H3 use question phrasing. >800 words. | GSC queries match page content. FAQ schema detected. Multiple queries per page. |
| 3 | Topic clusters with internal links. Content covers subtopics systematically. | Clear content hierarchy. Related posts linked. Multiple content types. | Full keyword map coverage. No major content gaps. Low cannibalization. |
| 4 | Pillar pages with comprehensive coverage. Content structured for AI extraction. | Pillar page with 2000+ words. Table of contents. Definitive answers. | Pillar pages rank for 50+ queries. Child pages link back. AI-friendly formatting throughout. |
| 5 | Programmatic freshness. Content adapts to new queries. Comprehensive entity coverage. | Dates current. Content regularly updated. Entity-rich markup. | Automated freshness signals. Full entity coverage. Expanding query footprint. |

### Technical Dimension

| Level | Criteria | URL Audit Signals | Deep Audit Signals |
|-------|----------|-------------------|-------------------|
| 1 | Site works. Basic meta tags exist. May have indexation gaps. | Has `<title>`, viewport meta. SSL active. Basic HTML structure. | Missing meta titles/descriptions. Indexation issues. No sitemap or broken sitemap. |
| 2 | Clean URLs. Sitemap exists. Meta tags set across pages. Mobile-friendly. | sitemap.xml accessible. Clean URL slugs. Meta description present. | All key pages indexed. No duplicate titles. Sitemap matches live pages. |
| 3 | Schema markup (Article, FAQ, HowTo). Fast loading. Structured data on all content pages. | JSON-LD schema in source. PageSpeed acceptable. Open Graph tags. | Full schema coverage. All CMS templates configured. No crawl errors. |
| 4 | Advanced schema (Organization, Person, BreadcrumbList). Entity markup. Knowledge panel signals. | Organization/Person schema. Breadcrumbs. Author pages linked. | Advanced schema on all pages. Entity relationships defined. URL inspection clean. |
| 5 | Full entity graph. Automated schema generation. Edge-case technical excellence. | SameAs links to authoritative profiles. Comprehensive structured data. | Programmatic schema. Zero technical debt. Automated monitoring. |

### Authority Dimension

| Level | Criteria | URL Audit Signals | Deep Audit Signals |
|-------|----------|-------------------|-------------------|
| 1 | No author attribution. No E-E-A-T signals. Anonymous content. | No author byline. No about page or credentials. | No branded query volume. No external signals. |
| 2 | Author mentioned. About page exists. Some credentials shown. | Author name visible. About/team page exists. | Some branded searches. Author name appears in GSC queries. |
| 3 | Author pages with credentials. External citations referenced. First-hand experience shown. | Detailed author bio. Data citations in content. Experience language ("we tested", "in our experience"). | Branded queries growing. Content cited by other sites. |
| 4 | Known entity. AI engines cite the content. Recognized expertise. | Content appears in AI overviews (check manually). Strong E-E-A-T throughout. | AI citation monitoring shows mentions. High branded query ratio. |
| 5 | Consistent AI citations. Knowledge panel. Recognized authority across platforms. | Knowledge panel exists. Multiple AI platforms cite content. | Dominant branded presence. Programmatic authority signals. |

### Measurement Dimension

| Level | Criteria | URL Audit Signals | Deep Audit Signals |
|-------|----------|-------------------|-------------------|
| 1 | No tracking. No monitoring. Flying blind. | No analytics scripts detected. No GSC verification meta tag. | GSC not connected or no data. No reporting cadence. |
| 2 | Basic analytics installed. GSC connected. Occasional checking. | GA4/GTM script present. GSC verification tag found. | GSC shows 90+ days of data. Basic metrics tracked. |
| 3 | Regular monitoring. CTR tracking. Content performance reviewed. | Multiple tracking tools. Structured data testing tool hints. | Weekly/monthly reports exist. CTR optimization active. Content gaps tracked. |
| 4 | AEO-specific metrics. LLM citation monitoring. Conversion tracking from organic. | Advanced tracking setup. Multiple measurement tools visible. | LLM citation data available. Conversion paths mapped. ROI tracked. |
| 5 | Full AEO dashboard. Automated alerts. Continuous optimization loop. | Enterprise-grade measurement stack. | Automated monitoring. Alert-driven optimization. Full attribution model. |

---

## Scoring Rules

### Per-Dimension Score
- Evaluate Level 1 gates first. If Level 1 fails, score = 1 and stop.
- If Level 1 passes, evaluate Level 2 gates. If Level 2 fails, score = 1.
- Score is the highest level whose gates **all** pass.
- Partial = score at the level below + note what's missing.

### Overall Maturity Level
- **Overall level = minimum dimension score** (weakest link)
- Example: Content=3, Technical=2, Authority=3, Measurement=2 → Overall Level 2

### URL Audit vs Deep Audit Confidence
- URL audit: confidence capped at Low–Medium (public signals only)
- Deep audit: confidence up to Medium–High (full GSC + Webflow data)

---

## Scorecard Output Format

```
## AEO Maturity Scorecard

**Overall Level**: [X] — [Level Name]
**Confidence**: [Low/Medium/High] (based on data available)

| Dimension | Score | Level Name | Gap to Next |
|-----------|-------|------------|-------------|
| Content | [X]/5 | [name] | [what's missing] |
| Technical | [X]/5 | [name] | [what's missing] |
| Authority | [X]/5 | [name] | [what's missing] |
| Measurement | [X]/5 | [name] | [what's missing] |

**Weakest dimension**: [dimension] — this is holding you back
**Strongest dimension**: [dimension] — this is your foundation
**Fastest win**: [specific action to move up one level]
```
