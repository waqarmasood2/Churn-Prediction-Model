# Part 3 — Churn Prediction Model & Model Card

Author: Waqar Masood (AIML_IITP_2506409)

## What's here
| File | Purpose |
|---|---|
| `churn_model.ipynb` | End-to-end notebook: load → features → leakage check → split → baseline → strong model → eval → threshold → interpretability → save |
| `model.pkl` | Final pipeline (`ColumnTransformer` + `GradientBoostingClassifier`) saved with joblib. Bundle: `{"model", "threshold", "features"}` |
| `metrics.json` | Validation + test metrics for baseline & strong model, selected threshold, top-20 features |
| `error_analysis.md` | FP & FN analysis with 12 specific customer examples and business interpretation |
| `model_card.md` | Structured model card |
| `requirements.txt` | Pinned deps |

## How to reproduce
```bash
pip install -r requirements.txt
jupyter nbconvert --execute --to notebook --inplace churn_model.ipynb
# or run train.py-equivalent cells inside the notebook
```

## How to load the model
```python
import joblib, pandas as pd
bundle = joblib.load("model.pkl")
model, thr = bundle["model"], bundle["threshold"]
proba = model.predict_proba(X_df)[:,1]
pred  = (proba >= thr).astype(int)
```
`X_df` must contain the columns listed in `bundle["features"]["num"] + bundle["features"]["cat"]`.

## Data
Uses the provided `rfm_modeling_snapshot.csv` (snapshot = 2025-09-30). This snapshot is constructed only from on/before-snapshot information (recency, 30/90/180-day windows ending at snapshot, demographics). The target `churn_next_60d` is held out and never used as input. See **Leakage prevention** in the notebook.

## Headline test results (strong model, threshold = 0.32)
ROC-AUC **0.87** · PR-AUC **0.85** · Recall **0.89** · Precision **0.74** · F1 **0.81**.
