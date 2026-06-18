# Model Card — Churn Prediction (60-day window)

**Model version:** v1.0 · **Date:** 2025-09-30 snapshot · **Owner:** Retention Analytics

## 1. Intended use
- **Primary:** Score the active customer base daily/weekly to produce a churn-risk probability that the CRM uses to prioritise retention campaigns (email, WhatsApp, win-back offers) for customers likely to churn in the next 60 days.
- **Secondary:** Feed risk tier into the RFM segment view so CX agents see context before responding to a ticket.
- **Users:** Retention marketing, lifecycle ops, CX leadership.

## 2. Out-of-scope / do NOT use for
- Denying service, refunds, or price increases to "high-risk" customers.
- Credit, fraud, or any irreversible decisions about a customer.
- Scoring customers acquired through channels not represented in training data (e.g. a new B2B channel).
- Long-horizon (>90-day) churn — the label was a 60-day window.
- Individual blame/coaching of CX agents based on a single customer's score.

## 3. Data
- **Source:** `rfm_modeling_snapshot.csv`, 2,400 customers, snapshot 2025-09-30.
- **Splits:** Provided `split` column — train 1,680 / validation 360 / test 360. Stratification preserved by source.
- **Features (28):** demographics (city_tier, age_group, channel, loyalty_tier, preferred_category, marketing_consent); order behaviour over 180d (recency, frequency, monetary, return rate, avg discount, avg rating, category diversity); support 90d (ticket count, negative rate, resolution hours); tenure (days_since_signup); web 30d (sessions, views, cart/wishlist adds, abandoned carts, email opens, campaign clicks, last visit days).
- **Target:** `churn_next_60d` (binary, ~50% positive rate in this synthetic dataset).
- **Leakage controls:** Only pre-snapshot windows used; target column dropped from features; split column excluded; intervention_history not used as a feature (it overlaps the target window).

## 4. Approach
- **Baseline:** Logistic Regression with class_weight="balanced", standardised numerics, one-hot categoricals.
- **Final model:** Gradient Boosting Classifier (sklearn) — 300 trees, depth 3, learning rate 0.05 — inside the same `ColumnTransformer` pipeline.
- **Threshold selection:** PR-curve sweep on validation; chose the threshold maximising F1, then biased toward recall when the cost asymmetry (lost LTV ≈ 10–30× cost of an extra retention touch) warranted it. Final threshold = **0.32**.

## 5. Performance (test set, n=360)
| Metric | Baseline LR @0.5 | Strong GBM @0.32 |
|---|---|---|
| ROC-AUC | ~0.84 | **0.87** |
| PR-AUC | ~0.81 | **0.85** |
| Precision | ~0.72 | 0.74 |
| Recall | ~0.78 | **0.89** |
| F1 | ~0.75 | **0.81** |
| Confusion (tn/fp/fn/tp) | — | 116 / 52 / 19 / 149 |

See `metrics.json` for exact values.

## 6. Top drivers (GBM feature importance, top 8)
recency_days, frequency_180d, last_visit_days_ago, sessions_30d, monetary_180d, product_views_30d, days_since_signup, avg_rating_180d. Higher recency / lower engagement → higher predicted churn. Full top-20 in `metrics.json`.

## 7. Limitations
- Trained on a single 2025-09-30 snapshot — no temporal validation. Will need backtest on rolling snapshots before production.
- ~50% positive base rate is unusual; in production the positive rate will likely be lower, which will shift PR-AUC and require threshold re-tuning.
- No customer-level cost data, so threshold is justified qualitatively, not via an explicit cost matrix.
- Sparse signals for very new customers (<30 days since signup) — the model is least reliable here.
- Categorical encoders use one-hot — new category values at inference are ignored (handled via `handle_unknown="ignore"`).

## 8. Ethical risks
- **Differential treatment:** Retention offers driven by score could systematically under-serve a demographic if the score correlates with age_group or city_tier. Recommend a fairness audit by `city_tier` and `age_group` before broad rollout, and a guardrail that every customer remains eligible for baseline service quality regardless of score.
- **Self-fulfilling prophecy:** A customer flagged "low risk" gets no nurture and churns. Counter by reserving a holdout that receives standard touch regardless of score.
- **Privacy:** No PII in features; do not surface raw scores to customers.

## 9. Monitoring (post-deployment)
- **Input drift:** PSI on each numeric feature weekly; alert >0.2.
- **Output drift:** distribution of churn probabilities, % above threshold.
- **Performance:** rolling 60-day backtest of precision/recall as labels mature.
- **Business KPI:** retention rate of treated-vs-holdout high-risk customers; campaign cost per retained customer.
- **Operational:** prediction latency, API error rate, model file checksum.
- **Retraining triggers:** PSI > 0.25 on ≥3 features OR PR-AUC drops > 0.05 from baseline OR every 90 days, whichever first.

## 10. When NOT to use the model
- During or right after a major catalogue / pricing change (training distribution invalid).
- For a customer with <14 days of history (insufficient signal).
- As the sole input to any decision that affects the customer adversely.
