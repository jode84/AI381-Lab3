# Model selection

## The question
This model predicts whether a telecom customer is likely to cancel their subscription (churn). A false negative misses an at-risk customer (lost revenue), while a false positive wastes retention offers/discounts on a customer who would have stayed anyway.

## Candidates
| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---|---|---|---|---|---|
| `logreg` | Fast, interpretable linear baseline | 0.798 | 0.495 | 0.250 (one-year) | 19.2 | 4 | `a8611436` |
| `random_forest` | Non-linear tree ensemble to capture feature interactions | 0.759 | 0.454 | 0.222 (one-year) | 203.6 | 3160 | `5d5d895c` |
| `gradient_boosting` | Boosted trees for higher accuracy on tabular data | 0.748 | 0.417 | 0.222 (two-year) | 27.0 | 142 | `690044cb` |

## The champion
I picked **`logreg`** (`a8611436`). It achieved the highest AUC (**0.798**) and highest F1 (**0.495**), while also being the fastest (**19.2 ms** p95 latency, within the 25 ms budget) and smallest (**4 KB**). Both tree-based models (`random_forest` and `gradient_boosting`) were slower and scored lower on this dataset, so I did not have to trade away accuracy for speed.

## My promotion rules
- Latency budget (p95): **25.0** ms, because the prediction endpoint serves real-time customer requests and keeping p95 under 25 ms leaves headroom for network and API overhead.
- Minimum gain to replace the champion (F1 points): **0.005**, because smaller improvements could be random split noise and are not worth the operational risk of swapping a production model.

## What I would check next
I would monitor **data drift** (changes in customer behavior or pricing plans over time) and evaluate **low-volume contract slices** (such as two-year contracts where churn is rare, making slice F1 noisy or artificially low).
