---
name: getting-started
version: "1.1"
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
CHOOSE MODE → INTERVIEW → GENERATE → SAVE → CONFIRM
```

---

## Phase 0: Choose Setup Mode

Start by asking:

```
How would you like to set up your SEO Copilot?

1. **Quick setup** (3 questions) — Best for solo freelancers who want to get started fast
   I'll ask: business, audience, tone. Everything else uses smart defaults.

2. **Full setup** (7 sections) — Best for teams or agencies who need precise brand guidelines
   Complete interview covering all SEO/AEO constraints.

3. **Import existing config** — Have a config from another project? Paste it or provide a path.
```

If **Quick setup**: Jump to Phase 1 Quick Mode.
If **Full setup**: Run Phase 1 Full Mode.
If **Import**: Jump to Phase 3.5 Import Config.

---

## Phase 1: Interview

### Quick Mode (3 questions)

For users who chose quick setup:

```
Quick Setup — I'll infer the rest from your answers.

1. **Business**: What's your business name and what do you do?
   Example: "Acme Agency — Webflow development for SaaS startups"

2. **Webflow site URL**: What's your site? (so other skills can fetch/analyze pages)
   Example: "https://acme.agency"

3. **Audience & tone**: Who do you write for, and how should it sound?
   Example: "Startup founders, friendly but professional"
```

After quick mode answers, **infer the rest**:
- From "Webflow freelancers" → infer `expertiseLevel: intermediate`
- From "SaaS startups" → infer pain points: scaling, efficiency, growth
- From "friendly but professional" → infer `formality: conversational`, `tone: ["Friendly", "Professional"]`
- Use sensible defaults for SEO/AEO settings

Offer: "Want me to analyze [site URL] to detect your current brand voice? (yes/no)"
If yes: Fetch 2-3 blog posts, analyze tone, headings, formality, and pre-fill `brandVoice`.

---

### Full Mode (7 sections)

For users who chose full setup. Accept free-form answers. Provide examples.

### 1.1 Business Overview

```
Let's set up your SEO Copilot config.

1. **Business name**: What's your company/brand name?
2. **What you do**: One sentence describing your product/service
3. **Webflow site URL**: Your site URL (so other skills can fetch pages)
4. **Primary goal**: What's the main business outcome you want from SEO?
   - More leads
   - More sales
   - Brand awareness
   - Thought leadership
   - Other: ___

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
   Example: "Marketing managers at B2B SaaS companies, 50-200 employees"

2. **Expertise level**: How technical is your audience?
   - Beginner (explain everything)
   - Intermediate (knows basics)
   - Expert (skip fundamentals)
   [If omitted, I'll infer from audience description]

3. **Pain points**: What problems are they trying to solve?
   [If omitted, I'll suggest common pain points based on your audience]

4. **Decision factors**: What matters most when choosing a solution like yours?
   Examples:
   - Price vs. features vs. support quality
   - Ease of use vs. power/flexibility
   - Speed to implement vs. customization
   - Reputation/reviews vs. personal demo
```

**Smart inference**: If you say "Webflow freelancers," I'll suggest:
- Expertise: Intermediate
- Pain points: Finding clients, pricing projects, scaling solo work
- Decision factors: Price, ease of use, time savings

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
4. **AI tell-tales to avoid**: Common AI patterns that don't match your brand
   - Default banned: "game-changer", "revolutionary", "let's dive in", "in today's [X]"
   - Emoji bullets (✅ ❌)
   - "Key Takeaways" / "Final Thoughts" headers
   - Add any others: ___
5. **Human signals to include**:
   - First-person experience phrases you use
   - Your typical aside style (e.g., "honestly", "frankly", "look")
   - Specific numbers/data you can reference
6. **Examples**: Link to 1-2 articles that represent your ideal voice (optional)
```

### 1.5 SEO Constraints

```
SEO-specific guidelines:

1. **Primary keywords**: What are your 3-5 most important keywords/topics?
2. **Competitors**: Who are your main SEO competitors? (domains)
3. **Keyword density**: Any preferences? (default: natural usage, no stuffing)
4. **Pillar pages**: Key articles to always link to? (full URLs)
   Example: "https://yoursite.com/guide/complete-webflow-seo"
   [I'll validate these URLs are live]
5. **External linking**:
   - Preferred sources to cite (e.g., industry publications, .gov, .edu)
   - Sources to avoid (e.g., competitors, low-quality sites)
6. **Meta title format**: Any branding suffix?
   - Example: "| Brand Name" at the end
   - Example: "Brand: " at the start
```

**Validation**: When pillar page URLs are provided, I'll fetch each one to confirm it's live. If any return 404, I'll flag it before saving the config.

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
  "setupMode": "quick|full",

  "business": {
    "name": "...",
    "description": "...",
    "webflowSiteUrl": "https://...",
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
    "avoidAiPatterns": {
      "phrases": ["game-changer", "let's dive in", "in today's"],
      "structures": ["emoji-bullets", "key-takeaways-header", "final-thoughts-header"],
      "formatting": ["excessive-em-dashes", "bold-every-phrase"]
    },
    "humanSignals": {
      "firstPerson": true,
      "asideStyle": "honestly",
      "specificNumbers": true
    },
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

### 3.0 Import Config (if chosen in Phase 0)

If user chose "Import existing config":

```
Paste your existing config JSON, or provide a path:
- Paste JSON directly
- File path: /path/to/seo-copilot-config.json
- URL: https://... (if hosted)
```

After import:
1. Validate JSON structure
2. Show summary of imported settings
3. Ask: "Anything you'd like to update before saving?"
4. If yes: Jump to relevant interview section
5. If no: Proceed to save

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

### Share This Config

To use this config in another project:
1. Copy `.claude/seo-copilot-config.json` to the new project
2. Or run `/getting-started` and choose "Import existing config"
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
| `business.webflowSiteUrl` | All skills | Fetch pages, validate URLs, analyze content |
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

⚡ GUARD — **User requests abort:**
If user says "stop", "cancel", "abort", or "nevermind" at any phase:
- Confirm: "Stop the setup? Any answers given so far will be lost."
- If confirmed: Exit cleanly
- If partial answers exist: Offer to save partial config as draft to `.claude/seo-copilot-config.draft.json`

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

⚡ GUARD — **Pillar page URL validation:**
When pillar page URLs are provided:
1. Fetch each URL to check if it's live
2. If 404 or error: "⚠️ [URL] returned [status]. Is this correct?"
3. If redirect: "ℹ️ [URL] redirects to [new URL]. Use the new URL?"
4. Store only validated URLs in config

⚡ GUARD — **Brand voice analysis offered:**
If user provides a Webflow site URL and hasn't filled brand voice:
- Offer: "Want me to analyze your existing content to detect your brand voice?"
- If yes: Fetch 2-3 blog posts from the site
- Analyze: tone, formality, average sentence length, common phrases
- Present findings: "Based on your content, your voice seems [Friendly, Professional]. Formality: conversational. Sound right?"
- Let user confirm or adjust

⚡ GUARD — **Smart inference from audience:**
When user describes their audience, automatically infer:
- "Webflow freelancers" → `expertiseLevel: intermediate`, pain points: client acquisition, pricing, scaling
- "Enterprise marketing teams" → `expertiseLevel: intermediate`, decision factors: security, support, integrations
- "Beginners learning SEO" → `expertiseLevel: beginner`, pain points: overwhelm, jargon, where to start
- Present inferences: "Based on 'Webflow freelancers,' I'm assuming intermediate expertise and these pain points: [list]. Adjust?"

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

---

## Activity Log

After every execution, append a row to `.claude/reports/{domain}/activity-log.md`.

**Determining `{domain}`**: Extract from the Webflow site URL configured during setup. If no Webflow site is identified yet, use the `business.name` slug as a fallback (e.g., "Acme SaaS" → `acme-saas`).

If the file doesn't exist, create it with the header:

```markdown
# Activity Log — {domain}

| Date | Skill | Summary |
|------|-------|---------|
```

Then append a row:

```
| YYYY-MM-DD | /getting-started | [one-line summary: e.g., "Config created. Business: Acme SaaS. Markets: US, UK. Voice: Professional. Saved to seo-copilot-config.json"] |
```

Log even if re-running to update config (e.g., "Config updated. Changed primary keywords and brand voice.").
