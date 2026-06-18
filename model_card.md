# Model Card: 60-Day Customer Churn Prediction Engine

## 1. Intended Use
* **Primary Purpose:** Identifies e-commerce customers who are highly likely to cancel their accounts or disengage from the platform over the next 60 days.
* **Target Audience:** Product Teams, Customer Success Representatives, and Growth Marketing Managers to coordinate targeted retention outreach.
* **Out-of-Scope Uses:** This model should not be used to automatically deny service, alter basic item pricing, or penalize consumers based on their demographic profiles.

## 2. Training & Evaluation Data
* **Source Dataset:** Compiled from `rfm_modeling_snapshot.csv`, spanning 2,400 unique customer entities.
* **Target Balancing:** The operational ecosystem features a balanced target representation, with **46.95% churned instances** versus **53.04% retained instances**.
* **Anti-Leakage Partitioning:** Split using strict point-in-time constraints. All historical operational records matching timestamps past `2025-09-30` were entirely stripped out before model execution.

## 3. Model Architecture & Approach
* **Baseline Configuration:** Logistic Regression (used for linear validation bounds).
* **Champion Configuration:** Random Forest Classifier built with 150 independent estimators, a maximum tree depth parameter of 10, balanced internal class weight allocations, and an optimized classification decision threshold locked at **0.40**.

## 4. Performance Metrics (Optimized Threshold @ 0.40)
* **ROC-AUC Score:** 0.8796 (Strong model discriminative capacity)
* **Model Accuracy:** 81.55%
* **Precision:** 77.32%
* **Recall (Sensitivity):** 89.29% (Catches approximately 9 out of 10 vulnerable customer accounts)
* **F1-Score:** 0.8287

## 5. Model Limitations
* **Cold-Start Vulnerability:** Fails to accurately evaluate brand-new customer profiles who have been registered on the platform for less than 14 business days.
* **Cross-Device Blindspots:** Relies heavily on mobile session tracking; if a customer shifts entirely to offline or desktop touchpoints, the model will falsely flag them as a churn risk.

## 6. Ethical Risks & Framing
* **Algorithmic Discrimination:** Risk that promotional offerings systematically avoid specific demographic groupings based on encoded variables like `age_group` or `city_tier`.
* **Mitigation Plan:** The model's feature importance focus relies entirely on behavioral inputs (sessions, transactions, tickets) rather than personal demographic segments. 

## 7. Ongoing Monitoring Requirements
* **Data Drift Inspections:** Run quarterly evaluations to check if the baseline customer churn distribution shifts drastically away from the original ~47% benchmark.
* **Concept Drift Audits:** Re-train the core model every 90 days to ensure it adapts to changing market conditions, competitor promo seasons, and evolving user app habits.