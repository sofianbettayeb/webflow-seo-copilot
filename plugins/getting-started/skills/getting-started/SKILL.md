---
name: getting-started
description: |
  One-time setup that captures business goals, audience, markets, brand voice, SEO and AEO constraints.
  Saves to your project's AI config file for persistent guidelines across all SEO Copilot skills.
  Triggers: getting started, setup, configure, onboarding, seo setup, initialize, config.
  Run once per project. Other skills will reference this config automatically.
disable-model-invocation: true
---

# Getting Started

One-time setup that captures your business context and saves it to a persistent config file. Other Webflow SEO Copilot skills (`/refresh-content`, `/click-recovery`) will reference this config for consistent, on-brand outputs.

## What This Skill Does

1. **Interviews you** about your business, audience, and SEO/AEO goals
2. **Generates a config file** at `.claude/seo-copilot-config.json`
3. **Other skills read this config** to match your brand voice, target the right keywords, and follow your constraints

Run this once per project. Re-run to update your config.

---

## Workflow

```
INTERVIEW → GENERATE → SAVE → CONFIRM
```

---

## Phase 1: Interview

Ask the user each section. Accept free-form answers. Provide examples to guide responses.

### 1.1 Business Overview

```
Let's set up your SEO Copilot config. First, tell me about your business:

1. **Business name**: What's your company/brand name?
2. **What you do**: One sentence describing your product/service
3. **Primary goal**: What's the main business outcome you want from SEO?
   - More leads
   - More sales
   - Brand awareness
   - Thought leadership
   - Other: ___
```

### 1.2 Target Audience

```
Who are you trying to reach?

1. **Primary audience**: Who is your ideal customer? (role, industry, company size)
2. **Expertise level**: How technical is your audience?
   - Beginner (explain everything)
   - Intermediate (knows basics)
   - Expert (skip fundamentals)
3. **Pain points**: What problems are they trying to solve?
4. **Decision factors**: What matters most to them when choosing a solution?
```

### 1.3 Markets & Languages

```
Where do you operate?

1. **Primary markets**: Which countries/regions? (e.g., US, UK, DACH, Global)
2. **Languages**: Which languages do you publish in? (e.g., English, German, French)
3. **Local considerations**: Any regional terms, spelling preferences, or cultural notes?
   - Example: "Use British English for UK content"
   - Example: "Avoid idioms that don't translate"
```

### 1.4 Brand Voice

```
How should your content sound?

1. **Tone**: Pick 2-3 words that describe your voice
   - Examples: Professional, Friendly, Authoritative, Casual, Witty, Technical, Approachable
2. **Formality**:
   - Formal (no contractions, third person)
   - Conversational (contractions OK, second person "you")
   - Casual (slang OK, first person)
3. **Avoid**: Any words, phrases, or styles to never use?
   - Example: "Never use 'synergy' or 'leverage'"
   - Example: "Avoid exclamation marks"
4. **Examples**: Link to 1-2 articles that represent your ideal voice (optional)
```

### 1.5 SEO Constraints

```
SEO-specific guidelines:

1. **Primary keywords**: What are your 3-5 most important keywords/topics?
2. **Competitors**: Who are your main SEO competitors? (domains)
3. **Keyword density**: Any preferences? (default: natural usage, no stuffing)
4. **Internal linking**: Any pillar pages or key articles to always link to?
5. **External linking**:
   - Preferred sources to cite (e.g., industry publications, .gov, .edu)
   - Sources to avoid (e.g., competitors, low-quality sites)
6. **Meta title format**: Any branding suffix?
   - Example: "| Brand Name" at the end
   - Example: "Brand: " at the start
```

### 1.6 AEO Constraints (Answer Engine Optimization)

```
For AI answer engines (ChatGPT, Perplexity, AI Overviews):

1. **Citation goal**: Do you want to be cited by AI answer engines?
   - Yes, prioritize AI-friendly formatting
   - Not a priority
2. **FAQ strategy**:
   - Use "People Also Ask" phrasing exactly
   - Rephrase questions in brand voice
   - Mix of both
3. **Structured data**: Which schema types matter most?
   - Article (always)
   - FAQ (for list/guide content)
   - HowTo (for tutorials)
   - Product (for product pages)
   - Other: ___
4. **E-E-A-T signals**: What expertise signals should we highlight?
   - Author credentials
   - Company experience (years in business, clients served)
   - Data/research citations
   - First-hand experience phrases
```

### 1.7 Content Preferences

```
Content style preferences:

1. **Heading style**:
   - Sentence case ("How to optimize your site")
   - Title Case ("How To Optimize Your Site")
2. **List style**:
   - Bullet points preferred
   - Numbered lists preferred
   - Mix based on context
3. **Paragraph length**:
   - Short (2-3 sentences max)
   - Medium (3-4 sentences)
   - Flexible
4. **CTA style**: How do you phrase calls-to-action?
   - Example: "Get started free"
   - Example: "Book a demo"
   - Example: "Learn more"
```

---

## Phase 2: Generate Config

After collecting all answers, generate a JSON config:

```json
{
  "version": "1.0",
  "created": "YYYY-MM-DD",
  "updated": "YYYY-MM-DD",

  "business": {
    "name": "...",
    "description": "...",
    "primaryGoal": "..."
  },

  "audience": {
    "primary": "...",
    "expertiseLevel": "beginner|intermediate|expert",
    "painPoints": ["...", "..."],
    "decisionFactors": ["...", "..."]
  },

  "markets": {
    "regions": ["US", "UK", "..."],
    "languages": ["en", "de", "..."],
    "localNotes": "..."
  },

  "brandVoice": {
    "tone": ["Professional", "Friendly"],
    "formality": "conversational",
    "avoid": ["synergy", "leverage"],
    "exampleUrls": ["https://..."]
  },

  "seo": {
    "primaryKeywords": ["...", "..."],
    "competitors": ["competitor.com", "..."],
    "keywordDensity": "natural",
    "pillarPages": ["/guide/main-topic", "..."],
    "preferredSources": [".gov", ".edu", "industrypub.com"],
    "avoidSources": ["competitor.com"],
    "metaTitleFormat": "| Brand Name"
  },

  "aeo": {
    "prioritizeCitations": true,
    "faqStrategy": "paa-exact|brand-voice|mix",
    "schemaTypes": ["Article", "FAQ", "HowTo"],
    "eeatSignals": {
      "authorCredentials": true,
      "companyExperience": "10+ years, 500+ clients",
      "dataCitations": true,
      "firstHandExperience": true
    }
  },

  "content": {
    "headingStyle": "sentence-case|title-case",
    "listStyle": "bullets|numbers|mixed",
    "paragraphLength": "short|medium|flexible",
    "ctaStyle": "Get started free"
  }
}
```

---

## Phase 3: Save Config

### 3.1 Choose Location

Ask the user:

```
Where should I save your SEO Copilot config?

1. Project root: `.claude/seo-copilot-config.json` (recommended — version controlled)
2. Custom path: ___
```

### 3.2 Write File

Use the Write tool to save the JSON config to the chosen path.

### 3.3 Update .gitignore (Optional)

Ask:

```
Should this config be version controlled?

1. Yes — commit it with your project (recommended for teams)
2. No — add to .gitignore (for sensitive business info)
```

If no, add `.claude/seo-copilot-config.json` to `.gitignore`.

---

## Phase 4: Confirm

Present the saved config:

```
## SEO Copilot Config Saved

**Location**: .claude/seo-copilot-config.json

### Summary

| Setting | Value |
|---------|-------|
| Business | [name] — [description] |
| Audience | [primary audience], [expertise level] |
| Markets | [regions], [languages] |
| Voice | [tone words], [formality] |
| Primary Keywords | [keywords] |
| Schema Types | [types] |

### What Happens Next

Other Webflow SEO Copilot skills will automatically read this config:

- `/refresh-content` — Matches your brand voice, uses your keywords, follows your heading style
- `/click-recovery` — Frames titles/descriptions to match your tone and audience

To update your config, run `/getting-started` again.
```

---

## Config Reference for Other Skills

Other skills should check for and load this config at startup:

```
1. Check if `.claude/seo-copilot-config.json` exists
2. If yes: Load and apply settings
3. If no: Use defaults, suggest running `/getting-started`
```

**How settings apply:**

| Config Field | Used By | How |
|--------------|---------|-----|
| `brandVoice.tone` | All content generation | Match tone in rewrites |
| `brandVoice.avoid` | All content generation | Filter out forbidden words |
| `audience.expertiseLevel` | Content complexity | Adjust jargon level |
| `seo.primaryKeywords` | Keyword optimization | Prioritize these keywords |
| `seo.pillarPages` | Internal linking | Always suggest linking to these |
| `seo.metaTitleFormat` | Title generation | Apply brand suffix/prefix |
| `aeo.faqStrategy` | FAQ generation | Use PAA phrasing or brand voice |
| `content.headingStyle` | All headings | sentence-case or Title Case |

---

## Conditional Guards

⚡ GUARD — **Config already exists:**
If `.claude/seo-copilot-config.json` already exists:
- Show current config summary
- Ask: "Update existing config or start fresh?"
- If update: Pre-fill answers from existing config

⚡ GUARD — **Minimal setup option:**
If user says "quick setup" or "minimal":
- Ask only essential questions (business name, audience, tone)
- Use sensible defaults for everything else
- Note which fields have defaults in the config

⚡ GUARD — **Missing answers:**
If user skips a question:
- Use a sensible default
- Mark as `"value": null` or `"value": "default"` in config
- Note in summary: "Some fields use defaults. Run `/getting-started` again to customize."

---

## Error Handling

| Error | Action |
|-------|--------|
| Cannot write to .claude/ | Create directory first, or ask for alternative path |
| Config file corrupted | Back up old file, create fresh config |
| User aborts mid-interview | Save partial config as draft, offer to resume |

---

## Example Session

```
User: /getting-started

Claude: Let's set up your SEO Copilot config. First, tell me about your business:

1. Business name:
2. What you do:
3. Primary goal:

User: 1. Acme SaaS  2. Project management for agencies  3. More leads

Claude: Got it. Who are you trying to reach?

[... continues through all sections ...]

Claude:
## SEO Copilot Config Saved

**Location**: .claude/seo-copilot-config.json

| Setting | Value |
|---------|-------|
| Business | Acme SaaS — Project management for agencies |
| Audience | Agency owners/PMs, Intermediate |
| Markets | US, UK, English |
| Voice | Professional, Friendly, Conversational |
| Primary Keywords | project management, agency tools, client management |

Your other SEO Copilot skills will now use these settings automatically.
```
