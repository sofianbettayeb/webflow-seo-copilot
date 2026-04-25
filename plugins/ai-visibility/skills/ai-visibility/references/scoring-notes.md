# AEO Copilot Scoring Reference

Use this reference when generating findings and writing report copy. The skill must reflect the platform's existing methodology — never invent a new formula.

## Composite brand score (0–100)

The overall **AI Visibility Score** returned by `get_insights` is a weighted composite:

| Component | Weight | What it measures |
|-----------|--------|------------------|
| Topic performance | 60% | How visible the brand is across tracked prompts (mention rate, position, source attribution) — averaged across topics with recency decay |
| Technical readiness | 25% | Website signals that affect LLM crawlability — robots.txt, sitemap.xml, schema.org markup, llms.txt presence, content freshness |
| Sentiment health | 15% | The sentiment skew of mentions that do happen — positive lifts the score, negative drags it down |

## Grade bands

| Grade | Score range | Interpretation |
|-------|-------------|----------------|
| A | 85–100 | Strong, defensible visibility across engines |
| B | 70–84 | Solid presence, with specific weak engines or topics |
| C | 55–69 | Mixed — visible in some engines but invisible in others |
| D | 40–54 | Material gaps; brand is regularly missing from relevant prompts |
| F | < 40 | Effectively invisible to LLMs on the tracked corpus |

## Per-prompt score (used inside topic performance)

Each prompt is scored 0–100 from four signals:

| Signal | Weight | Notes |
|--------|--------|-------|
| Visibility | 50% | Full credit if mentioned across all enabled engines; partial credit if mentioned in some |
| Position | 30% | Position 1 = 100, Position 2 = 85, Position 3 = 70 (decay 15 per position) |
| Sentiment | 10% | Positive = 100, Neutral = 50, Negative = 0 |
| Source attribution | 10% | 100 if brand is the cited primary source, else 0 |

Topic scores are weighted averages of prompt scores within each topic, with a recency decay: full weight inside the last 30 days, then −10% per additional 30-day window.

## Technical signal checklist

The 25% technical component sums these binary checks (each contributes its sub-weight to the composite):

| Check | Sub-weight |
|-------|------------|
| Robots.txt allows AI crawlers | 6% |
| Sitemap.xml present and reachable | 6% |
| Schema.org markup detected on key templates | 8% |
| llms.txt present | 5% |

When a check fails, the report should describe the missing signal as a finding (e.g., "No llms.txt detected on the domain"), not as a "do this" instruction.

## Sentiment health derivation

The 15% sentiment component is a normalized score:

```
sentiment_health = (positive × 100 + neutral × 50) / total
```

So 100% = every record is positive, 50% = every record is neutral, 0% = every record is negative. A "healthy" brand sits above 65%.

### Brand sentiment vs category sentiment

The `get_insights.sentiment` payload aggregates sentiment across **all tracked prompts**, including responses where the brand was not mentioned. That number reflects the tone of the surrounding category conversation, not the tone of the brand's own coverage.

Compute and label both:

- **Category sentiment** = platform value from `get_insights.sentiment`. Use this when reporting the platform-computed Sentiment Health score.
- **Brand sentiment** = compute from `get_results` rows where any engine mentioned the brand. This is the sentiment of the brand's actual mentions.

If `mentioned_count < 10`, brand sentiment is too small to report on its own. In that case:
- Put **brand sentiment** as the headline figure on the scorecard (with the count, e.g., "1 mention, neutral").
- Footnote category sentiment so the platform-derived score isn't mistaken for brand reception.

## Voice rules for findings

When the skill writes findings backed by these scores:

1. **Cite the score with its band** — "B grade — 68/100" not just "good".
2. **Cite the count, not just the rate** — "11 of 62 prompts" not just "18%".
3. **Cite the comparison** — "lowest of the 4 engines" or "above the 40% B-grade floor".
4. **Stay diagnostic** — "No llms.txt detected" not "Add an llms.txt file".
5. **Use platform terminology** — "mention rate", "position", "share of voice", "topic", "prompt corpus" — these match the AEO Copilot dashboard so the client sees consistency between report and tool.
