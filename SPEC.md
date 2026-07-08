# Impure Cinema

**Status:** Advanced — deployed on Netlify, formula refinement phase

## Vision
Aggregates movie ratings from multiple sources into a proprietary weighted score, the **Impurity Score**. Brand metaphor: distillation — refining raw critic/audience data into a purified judgment.

## Data Structure
```javascript
{
  title: string,
  year: number,
  impurityScore: number,      // 0-100, primary output
  miscibilityIndex: number,   // 0-1, critic/audience concordance
  sources: {
    criticsScore: number,
    audienceScore: number,
    weight: number
  }
}
```

## Sources
- IMDB (critics + audience)
- Rotten Tomatoes (critics + audience)
- Letterboxd (audience primary)
- Metacritic (critics primary)

## Scoring Formula
```
impurityScore = (criticsScore * criticWeight) + (audienceScore * audienceWeight)
  criticWeight + audienceWeight = 1.0
  Current tuning: 0.6 critics / 0.4 audience
```

## Miscibility Index
```
miscibilityIndex = 1 - abs(criticsScore - audienceScore) / 100
```
- 0.9–1.0: Consensus
- 0.7–0.9: Generally aligned
- 0.5–0.7: Divergent
- <0.5: Strong divergence (controversial)

## Brand Identity
- Colors: magenta / cyan
- Metaphor: filtering / distillation
- Typography: clean, minimal

## Deployment
- Host: Netlify (live)
- Frontend: single-page app
- Data: API calls to rating sources

## Open Questions / Next Steps
- Validate weight ratio against more pressure-test films
- Confirm Miscibility Index thresholds feel right in practice
