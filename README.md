# Credit Card Default Risk Prediction

End-to-end machine learning pipeline that predicts whether a customer will default on their credit card, using demographic, employment, and credit-bureau data.

**Best model:** Tuned XGBoost — **ROC-AUC ≈ 0.996** on held-out validation data

---

## Overview

Credit card issuers need to estimate default risk *before* extending or adjusting credit — not after a customer has already missed payments. This project builds a full, reproducible pipeline that takes raw customer data and outputs a calibrated default-probability score per customer, along with the reasoning behind each prediction.

The notebook covers the complete workflow a production risk model would need to justify: data cleaning, exploratory analysis, feature engineering, model comparison, hyperparameter tuning, explainability (SHAP), and final deployment-ready artifacts.

## Dataset

| | Rows | Columns | Default rate |
|---|---|---|---|
| `train.csv` | 45,528 | 19 | 8.12% |
| `test.csv` | 11,383 | 18 (unlabeled) | — |

Each row is one customer with three groups of features:

- **Demographics** — age, gender, marital/family size, home/car ownership
- **Income & employment** — yearly income, employment tenure, occupation type
- **Credit bureau history** — credit limit, utilization %, credit score, prior defaults, recent default flag

Target: `credit_card_default` (1 = defaulted, 0 = did not)

## Results

| Model | ROC-AUC | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic Regression | ~0.94 | — | — | — |
| Random Forest (tuned) | ~0.995 | — | — | — |
| **XGBoost (tuned)** | **0.996** | — | — | — |

*(See the notebook for the full, exact metrics table and confusion matrices for every model.)*

### Key drivers of default risk

1. **Prior default history** (`prev_defaults`, `default_in_last_6months`) — by far the strongest signal; correlates with the target at ~0.77–0.81
2. **Credit utilization & leverage** — `credit_limit_used(%)`, engineered `debt_to_income` and `credit_limit_to_income` ratios
3. **Employment stability** — tenure and unemployment status, secondary signal

Confirmed via both built-in feature importance and SHAP value analysis — see Section 11 of the notebook.

## Project workflow

1. Load and inspect data
2. Exploratory data analysis — target distribution, missing values, feature relationships
3. Missing value handling & categorical cleanup (leakage-safe, fit only on training folds)
4. Drop identifiers (`customer_id`, `name`)
5. Feature engineering — 5 new financial ratio features
6. Stratified train/validation split
7. `ColumnTransformer` preprocessing pipeline (impute → scale/encode)
8. Train Logistic Regression, Random Forest, XGBoost
9. Compare ROC-AUC, precision, recall, F1, confusion matrices
10. Hyperparameter tuning via `RandomizedSearchCV`
11. Feature importance + SHAP explainability
12. Save best model pipeline with `joblib`
13. Generate predictions on the test set
14. Business insights & recommendations

## Repository structure

```
.
├── credit_risk_prediction.ipynb   # Full end-to-end notebook
├── best_credit_risk_model.joblib  # Saved pipeline (preprocessing + tuned XGBoost)
├── test_predictions.csv           # Predicted probabilities for test.csv
├── train.csv                      # Training data (labeled)
├── test.csv                       # Test data (unlabeled)
└── README.md
```

## Tech stack

`Python` · `pandas` · `NumPy` · `scikit-learn` · `XGBoost` · `SHAP` · `matplotlib` · `seaborn` · `joblib`

## Getting started

```bash
# clone the repo
git clone https://github.com/usmanirfnn/credit-card-default-risk.git
cd credit-card-default-risk

# install dependencies
pip install pandas numpy scikit-learn xgboost shap matplotlib seaborn joblib jupyter

# launch the notebook
jupyter notebook credit_risk_prediction.ipynb
```

### Using the saved model directly

```python
import joblib
import pandas as pd

model = joblib.load("best_credit_risk_model.joblib")

# new_data must have the same engineered features as in the notebook
predictions = model.predict_proba(new_data)[:, 1]  # probability of default
```

## Business recommendations

- **Tiered risk-based limits** — use the predicted probability, not just a binary flag, to set credit limits and interest rates on a sliding scale
- **Early-warning monitoring** — track `default_in_last_6months` for existing cardholders and flag rapidly for proactive intervention
- **Affordability checks at origination** — incorporate `debt_to_income` and `credit_limit_to_income` into underwriting rules
- **Threshold tuning by use case** — lower the decision threshold to prioritize recall (catch more risk) or raise it to prioritize precision (fewer false declines), depending on the business context

## Limitations & next steps

- Model performance leans heavily on a small number of bureau fields; adding transaction-level/time-series data would likely reduce that dependence and improve robustness
- Recommend periodic retraining and drift monitoring as spending behavior and macroeconomic conditions shift
- Before production deployment: A/B test against the current risk process and audit for fairness/bias across demographic groups

## Author

**Usman Irfan** — Business Analytics student, AI/ML & Data Science
[GitHub](https://github.com/usmanirfnn) · Karachi, Pakistan

## License

This project is available under the MIT License.
