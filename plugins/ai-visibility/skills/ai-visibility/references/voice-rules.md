# Voice Rules — AI Visibility Findings

Findings are read by clients in five minutes. They have to sound like a sharp practitioner wrote them, not a chatbot.

Apply this checklist before rendering any generated copy (TL;DR, scorecard drivers, findings, source intro).

## Hard rules

1. **No em dashes.** Replace with commas, periods, or colons. Em dashes are the single strongest AI tell.
2. **No rule-of-three padding.** "services, locations, and entities" → "services and entities". If the third item carries no extra weight, cut it.
3. **No filler.** Cut these phrases:
   - "showcasing", "highlighting", "reflecting", "symbolizing"
   - "natural beachhead", "currently aligned", "the four major"
   - "It is important to note that", "presumed in order"
   - "is the strongest signal for X to associate Y with Z" → "is how X maps Y to Z"
4. **No copula avoidance.** Use "is" and "are". "serves as", "stands as", "functions as" are AI-shaped substitutes.
5. **No vague attributions.** "Industry observers", "experts believe", "some analysts" — cite the actual data point or drop the claim.
6. **No negative parallelism.** "It's not just X, it's Y" is a chatbot construction.
7. **No generic upbeat closers.** "There's a strong foundation to build on", "exciting opportunities ahead" — drop them. End on the data.

## Voice targets

- **Sharp.** Every sentence does work. If a sentence isn't carrying a number, a name, or a decision, it shouldn't be there.
- **Credible.** Cite the count, not just the rate ("11 of 62" beats "18%"). Cite the comparison ("lowest of the 4 engines" beats "low").
- **Direct.** Subject, verb, evidence. Short sentences are fine. Mix in one longer sentence for rhythm.

## Audit pass

Before writing the file, ask: "What about this would make a smart reader assume an LLM wrote it?" Answer in one sentence, then revise.

Common remaining tells after a first pass:
- Even sentence rhythm (every clause the same length)
- Clean parallelism in lists ("X is N, Y is M, Z is K")
- Slogan-y closers ("That's the place to push first")
- Hedge words that weren't in the original ("seems to suggest", "could potentially")

If any are present, rewrite the offending line. Single audit pass is enough — don't loop.

## Examples

### TL;DR

**AI-shaped:**
> Brand X is effectively invisible across the four major AI engines — 1 mention in 100 engine-prompt chances over the last week of tracking. The big GSIs (Accenture, Deloitte, IBM) own the conversation. The single signal that does exist sits on Perplexity inside Contact Centers, position 2 — a natural beachhead topic, and the only one currently aligned with Brand X's positioning.

**Humanized:**
> Brand X isn't showing up in AI answers. One mention across 100 engine-prompt chances last week. Accenture, Deloitte, IBM, and Capgemini own the conversation. The single signal sits on Perplexity, in Contact Centers, position 2, neutral sentiment. That's a topic Brand X's positioning already targets, and it's the only engine where the brand registers at all.

### Finding

**AI-shaped:**
> No schema.org markup detected on key templates. Schema markup is the strongest crawlability signal for LLMs to associate the site with services, locations, and entities — its absence drags the technical readiness score by 8 of 25 sub-weight points.

**Humanized:**
> No schema.org markup on key templates. Schema is how LLMs map a site to specific services and entities. Without it, Brand X loses 8 of the 25 technical sub-weight points.
