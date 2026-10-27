# Factor scorecard

Five descriptive metrics per fund, adapted from *Analysis of Mutual Funds* (Sood, Arora, Nellurvalappil; https://github.com/abhinavsood/mutual-fund-analysis). Each is simple arithmetic on figures a source returned, and each is shown beside its inputs.

| Metric | Computed as | Range | Per horizon? |
|---|---|---|---|
| **Risk score** | Source's risk level mapped onto 0–1, lower risk scoring higher (see below) | 0–1 | No |
| **Credit-rating score** | Source's rating placed on 0–1 with its best grade at 1.0 (e.g. 4 of 5 Morningstar stars → 0.8; CRISIL rank 2 of 5, where rank 1 is best → 0.8) | 0–1 | No |
| **Family-confidence ratio** | Scheme AUM ÷ fund-family AUM, same currency and as-of date | 0–1 | No |
| **Beat-category flag** | "Above" if the fund's return exceeds its category average for that horizon, "Below" if not, "Level" within 0.05 percentage points | label | Yes |
| **Category-volatility ratio** | Category's worst return ÷ category's best return for that horizon, only where the source reports both and both are positive | 0–1 | Yes |

## Risk score

Map the source's own scale onto 0–1 with lower risk scoring higher, and print the mapping used. For SEBI's six-level riskometer: Low 1.0, Low to Moderate 0.8, Moderate 0.6, Moderately High 0.4, High 0.2, Very High 0.0. For a five-level scale: Low 1.0, Below Average 0.75, Average 0.5, Above Average 0.25, High 0.0. A scale you can't place is "not available", not a guess.

## Reading the metrics

- **Family-confidence ratio** is how much of its family's assets the scheme holds. The study read a high ratio as the family's confidence in the scheme; it is also simply size, so present it as a share, not a vote.
- **Category-volatility ratio** close to 1 means the category's funds landed close together; close to 0 means a wide spread between best and worst. Where the worst return is negative the ratio has no meaning — say "category spread [worst]% to [best]%" instead.
- **Credit-rating score** compares only ratings from the same agency. Never place a CRISIL rank and a Morningstar rating on one scale.

## What not to do with them

The study averaged these five into an "expectation" per horizon, rounded it into a good/bad label, and trained random-forest classifiers on it. Test accuracy came out at 94–99%, but only because 90%+ of schemes were labeled "bad": a model that called every fund bad would score almost as well, and because the label was built from the features, the model was learning the formula back. That is why this skill:

- never combines the five into one score or label;
- never treats a missing input as zero (the study's imputer did, which scored an unrated fund as riskiest);
- reports association across the funds analyzed only descriptively, with the sample size, and never as a predictive "importance".
