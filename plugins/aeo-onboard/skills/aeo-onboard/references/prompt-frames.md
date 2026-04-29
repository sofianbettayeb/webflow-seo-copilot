# Prompt Frames — Listicle Templates

Every prompt should be the kind of question a buyer types into ChatGPT, Claude, Perplexity, or Google AIO when looking for a list of options. Listicles dominate the AEO surface area for category and comparison queries.

Each topic gets 5–6 prompts. Span the templates below; do not repeat the same sentence structure twice in one topic.

## Templates

### Broad list
- `List the best {category} tools.`
- `What are the best {category} platforms?`
- `Top {category} software available today.`

### Year-specific
- `Top {N} {category} tools in 2026.`
- `Best {category} platforms in 2026.`
- `{N} leading {category} solutions of 2026.`

### Persona-targeted
- `Best {category} for {persona}.`
- `Which {category} tools do {persona} use?`
- `Top {category} platforms recommended for {persona}.`

### Integration / stack-specific
- `Which {category} works best with {integration}?`
- `Best {category} add-ons for {integration}.`
- `{integration}-compatible {category} tools.`

### Use-case / outcome
- `Best {category} for {use_case}.`
- `What is the best {category} for {outcome}?`
- `{category} solutions that improve {metric}.`

### Industry vertical
- `Best {category} for {industry}.`
- `What {category} platforms are used in {industry}?`
- `Top {category} solutions for {industry} compliance.`

### Competitor / alternative
- `Best alternatives to {competitor}.`
- `What are the top alternatives to {competitor} for {use_case}?`
- `Compare {brand} vs {competitor} for {persona}.`

## Construction rules

1. **Vary structure.** No two prompts in the same topic should start with the same word.
2. **Use concrete entities.** Real personas ("aerospace machine shop owners"), real integrations ("Fusion 360"), real verticals ("aerospace and defense"). Generic prompts ("best tools for businesses") get filtered out by LLMs.
3. **Avoid the brand's own name.** The point of tracking is to see whether the brand surfaces *unprompted*. Brand-name prompts test direct retrieval, not visibility.
4. **One competitor prompt minimum.** Comparison queries are high-intent and a primary AEO target.
5. **Apply voice rules.** No em dashes, no filler. See `references/voice-rules.md`.

## Validated example — cloudnc.com, Axis 1 (AI CAM Programming Automation)

```
1. List the best AI CAM programming tools.
2. Top AI-assisted CAM software in 2026.
3. Best automated toolpath generation tools for machine shops.
4. Which AI CAM tools work with Fusion 360?
5. Best AI CAM software for aerospace machine shops.
6. Compare CloudNC vs Mastercam AI for CAM automation.
```
