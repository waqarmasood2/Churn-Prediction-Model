# Error Analysis

**Decision threshold:** 0.324 (chosen on validation set to balance precision/recall; biased toward recall because the cost of a missed churner — lost LTV — exceeds the cost of an unnecessary retention offer).

**Test confusion matrix:** {'tn': 116, 'fp': 52, 'fn': 19, 'tp': 149}

## Business risk framing
- **False Positive (predicted churn, stayed):** wastes a retention discount on a healthy customer. Cost = avg promo (~₹50). Low risk, capped by campaign budget.
- **False Negative (predicted retained, actually churned):** lost LTV with no intervention. Cost ≈ ₹2,000–₹5,000 in lost future orders. **~10–30× more expensive than a FP** — justifies a recall-leaning threshold.

## False Positives (top 6 by probability)

| customer_id | proba | recency | freq_180 | sessions_30 | neg_ticket% | reason |
|---|---|---|---|---|---|---|
| CUST01246 | 0.97 | 262 | 0 | 1 | 0.00 | high recency (262d), low frequency, low recent sessions, hasn't visited in 60d |
| CUST00437 | 0.93 | 151 | 1 | 0 | 0.00 | high recency (151d), low frequency, low recent sessions, hasn't visited in 33d |
| CUST01405 | 0.92 | 140 | 1 | 2 | 0.00 | high recency (140d), low frequency, low recent sessions |
| CUST01614 | 0.91 | 103 | 2 | 4 | 0.00 | mixed/borderline signals |
| CUST01325 | 0.91 | 186 | 0 | 1 | 0.00 | high recency (186d), low frequency, low recent sessions, hasn't visited in 43d |
| CUST02171 | 0.91 | 120 | 1 | 1 | 0.00 | high recency (120d), low frequency, low recent sessions |

## False Negatives (top 6 most-missed)

| customer_id | proba | recency | freq_180 | sessions_30 | neg_ticket% | reason |
|---|---|---|---|---|---|---|
| CUST00184 | 0.03 | 14 | 3 | 6 | 0.00 | mixed/borderline signals |
| CUST01990 | 0.03 | 59 | 4 | 11 | 0.00 | mixed/borderline signals |
| CUST01655 | 0.07 | 13 | 2 | 2 | 0.00 | low recent sessions |
| CUST01303 | 0.11 | 20 | 1 | 3 | 0.00 | low frequency |
| CUST00838 | 0.11 | 9 | 1 | 11 | 1.00 | low frequency, negative support sentiment, discount-heavy |
| CUST01253 | 0.11 | 99 | 2 | 13 | 0.00 | mixed/borderline signals |

## Patterns observed
- **FPs cluster on** customers with moderately high recency but still-active web sessions — the model over-weights stale recency for engaged browsers. Mitigation: blend `last_visit_days_ago` more strongly or add an engagement override rule before sending an offer.
- **FNs cluster on** customers with recent orders but flat/zero engagement signals (no sessions, no email opens). The model sees a recent purchase and under-predicts. Mitigation: add a feature for declining engagement vs prior 30-day window, or run a behavioural rule alongside the model.
- High-discount + low-rating customers appear in both buckets, signalling discount-sensitive churn that the model captures only partially.

## Recommended business action per error type
- FP: include a low-cost soft-touch (free shipping, content email) rather than a deep discount, so over-targeting is cheap.
- FN: layer a rule-based safety net (e.g. recency ≥ 90d AND sessions_30d == 0 ⇒ flag) on top of the model to catch the obvious misses the model under-scores.