# Schema Markup Templates for Webflow Blog Articles

## Article Schema

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{{title}}",
  "description": "{{meta_description}}",
  "image": "{{featured_image_url}}",
  "author": {
    "@type": "Person",
    "name": "{{author_name}}",
    "url": "{{author_url}}"
  },
  "publisher": {
    "@type": "Organization",
    "name": "{{site_name}}",
    "logo": {
      "@type": "ImageObject",
      "url": "{{logo_url}}"
    }
  },
  "datePublished": "{{publish_date_iso}}",
  "dateModified": "{{modified_date_iso}}",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "{{canonical_url}}"
  }
}
```

**Webflow implementation**: Add to page custom code (before </body>) or embed block.

## FAQ Schema

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "{{question_1}}",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "{{answer_1}}"
      }
    },
    {
      "@type": "Question",
      "name": "{{question_2}}",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "{{answer_2}}"
      }
    }
  ]
}
```

**Best practices**:
- Answers should be 40-60 words for featured snippet eligibility
- Questions should match "People Also Ask" phrasing exactly
- Maximum 10 FAQs recommended per page

## HowTo Schema

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "{{how_to_title}}",
  "description": "{{how_to_description}}",
  "totalTime": "PT{{minutes}}M",
  "step": [
    {
      "@type": "HowToStep",
      "name": "{{step_1_title}}",
      "text": "{{step_1_description}}",
      "image": "{{step_1_image_url}}"
    },
    {
      "@type": "HowToStep",
      "name": "{{step_2_title}}",
      "text": "{{step_2_description}}",
      "image": "{{step_2_image_url}}"
    }
  ]
}
```

**When to use**: Articles with step-by-step instructions, tutorials, guides.

## Breadcrumb Schema

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "{{site_url}}"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "{{site_url}}/blog"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "{{article_title}}",
      "item": "{{canonical_url}}"
    }
  ]
}
```

## Combining Multiple Schemas

Wrap multiple schemas in a single script tag using @graph:

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Article", ... },
    { "@type": "FAQPage", ... },
    { "@type": "BreadcrumbList", ... }
  ]
}
```

## Webflow Implementation Notes

1. **Page-level custom code**: Settings > Custom Code > Footer Code
2. **CMS template**: Add embed block with dynamic fields
3. **Validation**: Test with Google Rich Results Test before publishing
4. **Date format**: Use ISO 8601 (YYYY-MM-DDTHH:MM:SS+00:00)
