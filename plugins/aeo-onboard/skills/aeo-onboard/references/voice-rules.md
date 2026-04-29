# Voice Rules — AEO Onboard

Topic descriptions and tech-audit findings are read by clients and by Sofian. They have to sound like a sharp practitioner wrote them, not a chatbot.

Apply this checklist before rendering any generated copy (industry field, topic descriptions, audit findings, prompt phrasing).

## Hard rules

1. **No em dashes.** Replace with commas, periods, or colons. Em dashes are the single strongest AI tell.
2. **No rule-of-three padding.** "services, locations, and entities" → "services and entities". If the third item carries no extra weight, cut it.
3. **No filler.** Cut these phrases:
   - "showcasing", "highlighting", "reflecting", "symbolizing"
   - "innovative", "cutting-edge", "industry-leading", "world-class"
   - "passionate about", "results-driven", "holistic", "seamless", "leverage", "synergy"
   - "It is important to note that", "presumed in order"
4. **No copula avoidance.** Use "is" and "are". "serves as", "stands as", "functions as" are AI-shaped substitutes.
5. **No vague attributions.** "Industry observers", "experts believe", "some analysts" — cite the actual data point or drop the claim.
6. **No prescriptions in audit findings.** Describe the state, never recommend the fix. "No llms.txt detected" not "You should add llms.txt."
7. **Cite the count, not just the rate.** "11 of 62 prompts" beats "18%".

## Voice targets

- **Sharp.** Every sentence does work. If a sentence isn't carrying a number, a name, or a decision, it shouldn't be there.
- **Credible.** Cite the source ("Schema.org `Organization` found on `/`"). Cite the comparison ("3 of 47 sitemap URLs return 404").
- **Direct.** Subject, verb, evidence. Short sentences are fine.

## Audit pass

Before writing the file or sending to the user, ask: "What about this would make a smart reader assume an LLM wrote it?" Answer in one sentence, then revise the offending lines. Single audit pass — don't loop.

Common remaining tells after a first pass:
- Even sentence rhythm (every clause the same length)
- Slogan-y closers ("That's the place to start.")
- Hedge words ("seems to suggest", "could potentially")

## Examples

### Industry field

**AI-shaped:** "Innovative AI-powered solutions for the CAM programming industry, helping manufacturers achieve unprecedented levels of automation."
**Humanized:** "Industrial AI for CAM programming."

### Topic description

**AI-shaped:** "This topic showcases the brand's expertise in AI-powered CAM programming automation, highlighting how it leverages cutting-edge technology to deliver seamless integration with industry-leading platforms."
**Humanized:** "Buyers searching for ways to automate CAM programming. Broad-category topic, captures top-of-funnel demand from machine shops and aerospace primes evaluating AI-assisted toolpath generation."

### Audit finding

**AI-shaped:** "It is important to note that the brand's website is currently missing an llms.txt file, which is a critical signal for LLM crawlers and should be added to improve AEO performance."
**Humanized:** "No `llms.txt` at `/llms.txt` (404)."
