# Model Error Analysis & Operational Risk Registry

## 1. Business Impact of Model Errors

### False Positives (Type I Error)
* **Definition:** The model flags a healthy customer as "About to Churn," when they actually intend to stay.
* **Business Risk:** Financial waste. The company spends budget sending proactive retention rewards or discounts to a customer who would have remained anyway, slightly diluting profit margins.

### False Negatives (Type II Error) — The High-Risk Threat
* **Definition:** The model misses a customer, classifying them as "Retained," when they are actually going to leave.
* **Business Risk:** Catastrophic revenue loss. The customer departs entirely, forcing the company to lose their entire Lifetime Value (LTV) and pay expensive new acquisition costs to replace them.

---

## 2. Granular Review Matrix (10 Mock Customer Cases)

| Simulated ID | Metric Breakdown | Model Prediction | Actual Class | Error Type | Deep Operational Root Cause & Remediation Strategy |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **FN_001** | Recency: 4d, Sessions: 1 | 0 (Retained) | 1 (Churned) | **False Negative** | **Auto-Renewal Artifact:** A recurring subscription payment automatically triggered, making the customer look "Recent" (4 days), even though they haven't opened the app in months. **Fix:** Exclude automated financial actions from transactional Recency features. |
| **FN_002** | Sessions: 12, Tickets: 5 | 0 (Retained) | 1 (Churned) | **False Negative** | **Rage Browsing:** High monthly sessions (12) masked severe customer friction (5 unresolved tickets). The user was logging in frequently simply to check on their broken support tickets before leaving. **Fix:** Assign higher weight to active support friction metrics. |
| **FN_003** | Spend: \$15,000, Recency: 45d | 0 (Retained) | 1 (Churned) | **False Negative** | **Corporate Wholesaler:** High lifetime value made the account look highly resilient, but the buyer silently defected to a direct B2B competitor. **Fix:** Build distinct classification thresholds specifically for top-tier enterprise accounts. |
| **FN_004** | Total Orders: 1, Spend: \$40 | 0 (Retained) | 1 (Churned) | **False Negative** | **Low-Value Ghost:** The customer placed a tiny order and immediately lost interest, creating a flat profile that confused the tree variance. **Fix:** Incorporate early onboarding drop-off features. |
| **FN_005** | Discount: 85%, Recency: 8d | 0 (Retained) | 1 (Churned) | **False Negative** | **Promo Opportunist:** The user bought recently using a massive seasonal discount, but intended to leave as soon as standard full pricing returned. **Fix:** Engineer a feature tracking standard full-price versus discounted session rates. |
| **FP_006** | Sessions: 2, Recency: 65d | 1 (Churn) | 0 (Retained) | **False Positive** | **Seasonal Cycle User:** Low session volumes and high recency flagged this account as a flight risk, but they are simply a seasonal buyer who returns twice a year. **Fix:** Add year-over-year baseline comparison features. |
| **FP_007** | Unresolved Tickets: 4 | 1 (Churn) | 0 (Retained) | **False Positive** | **Vocal Advocate:** High support friction flagged this account as at-risk, but the user regularly provides positive feedback and remains loyal despite systemic bugs. **Fix:** Blend text-based sentiment tracking directly into the features. |
| **FP_008** | Sessions: 1, Returns: 3 | 1 (Churn) | 0 (Retained) | **False Positive** | **Bulk Size Tester:** High return rates and low sessions flagged an emergency, but this user purposefully orders multiple clothing sizes at once to return the poor fits. **Fix:** Normalize return quantities against total net order value. |
| **FP_009** | Age: 18-24, Signup: 15d ago | 1 (Churn) | 0 (Retained) | **False Positive** | **Cold Start Newcomer:** A newly registered user who hasn't built a deep operational footprint yet, causing the model to misclassify them as disengaged. **Fix:** Implement a 30-day trial window before running active predictions on new profiles. |
| **FP_010** | Sessions: 3, Loyalty: Silver | 1 (Churn) | 0 (Retained) | **False Positive** | **App Burnout:** The user stopped opening the mobile app but shifted their active purchasing behavior entirely to our web platform. **Fix:** Unify cross-device transaction streams cleanly into a single profile. |