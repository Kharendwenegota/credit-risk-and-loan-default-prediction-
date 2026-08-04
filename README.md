# Credit Risk & Loan Default Prediction

Predicting loan default risk to support smarter, faster lending decisions — modeled on real-world credit risk workflows and framed for the South African banking context.

## Business Problem

Lenders lose significant revenue when borrowers default on loans. The ability to accurately assess an applicant's default risk *before* approval directly impacts profitability, regulatory compliance, and customer outcomes.

This project builds an end-to-end credit risk pipeline that answers five core business questions:

1. **Will this applicant default?** — a predictive model scoring default probability at the point of application
2. **Is this applicant already showing signs of financial distress elsewhere?** — early warning signals from external credit bureau history
3. **Does this applicant's history with us predict future default?** — repayment behavior for returning customers
4. **Where is default risk concentrated across the loan portfolio?** — segment-level risk for business strategy
5. **Is this applicant being offered terms they can actually sustain?** — affordability analysis (debt-to-income, annuity-to-income)

## Dataset

This project uses the [Home Credit Default Risk](https://www.kaggle.com/c/home-credit-default-risk) dataset (Kaggle) — ~307,500 loan applications across 8 relational tables covering application data, credit bureau history, prior applications, and repayment behavior.

**Note on regional framing:** Real South African consumer credit data is held privately by credit bureaus and banks and is protected under South Africa's National Credit Act (NCA) and POPIA — it is not publicly available for open-source portfolio work. This project uses the Home Credit dataset as its technical foundation, with analysis and business narrative adapted to reflect South African lending and regulatory context (e.g. NCA affordability principles, ZAR-denominated risk framing).

## Project Structure

```
├── data/                  # raw & processed data (not committed — see .gitignore)
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_feature_engineering.ipynb
│   └── 04_eda.ipynb
├── src/                   # reusable Python modules
├── sql/                   # SQL queries for data analysis
├── dashboards/
│   ├── powerbi/            # credit_risk_dashboard.pbix
│   └── tableau/            # credit_risk_dashboard.twbx
├── app/                   # Streamlit risk-scoring app (planned)
├── outputs/
│   ├── figures/            # exported chart images
│   └── models/             # saved model files (planned)
```

## Methodology & Progress

### ✅ Phase 1 — Data Exploration
- Loaded and profiled `application_train.csv` (307,511 rows × 122 columns)
- Identified severe class imbalance: 91.9% non-default vs. 8.1% default
- Mapped missing-data patterns (concentrated in property/building fields, likely missing-not-at-random)
- Identified key data quality issues: a sentinel placeholder value in `DAYS_EMPLOYED`, and an extreme outlier in `AMT_INCOME_TOTAL`

### ✅ Phase 2 — Data Cleaning
- Fixed the `DAYS_EMPLOYED` anomaly (flagged + corrected ~55,000 placeholder values)
- Converted `DAYS_BIRTH` into an interpretable applicant age
- Capped the extreme income outlier while preserving the original field
- Created missing-value indicator flags for high-missingness columns (missingness itself can be predictive)
- One-hot encoded 16 categorical fields

### ✅ Phase 3 — Feature Engineering
- Merged and aggregated all 8 relational tables (bureau history, prior applications, installment payments, POS/cash balances, credit card balances) up to one row per applicant
- Engineered 30+ new features, including:
  - External credit bureau signals (`BUREAU_OVERDUE_COUNT`, `BUREAU_CREDIT_TYPE_DIVERSITY`)
  - Prior relationship signals (`PREV_APPROVAL_RATE`, `PREV_REFUSED_COUNT`)
  - Repayment behavior signals (`INSTALLMENT_LATE_RATIO`, `INSTALLMENT_STD_DAYS_LATE`)
  - Affordability ratios (`DEBT_TO_INCOME_RATIO`, `ANNUITY_TO_INCOME_RATIO`, `CREDIT_TERM`)
- Final feature set: 307,507 rows × 347 columns

### ✅ Phase 4 — Exploratory Data Analysis & Dashboards
Analyzed default rate patterns across six dimensions: occupation, age, income, loan amount, implied cost of credit, and region rating.

**Key finding:** two of the six patterns (income band, loan amount band) initially appeared non-monotonic and counter-intuitive. Rather than presenting these at face value, each was investigated to root cause:
- **Income band:** the apparent "medium income = highest risk" pattern was a compositional artifact — the lowest income band contained a disproportionate share of pensioners (31% vs. ~10–18% elsewhere), whose income is low but stable and guaranteed, artificially suppressing that band's observed risk. The real, reliable signal is `ANNUITY_TO_INCOME_RATIO`, which shows a clean, monotonic relationship with default risk.
- **Loan amount band:** confounded in two directions — pensioner concentration at the low end, and implicit credit pre-screening at the high end (larger loans are disproportionately issued to already-vetted, lower-risk applicants).

**Overall takeaway:** income and employment *stability* is a more reliable predictor of default than raw income *level* — a theme that carried through the dashboard design and will inform feature selection in modeling.

Interactive dashboards were built in both **Power BI** and **Tableau**, each including:
- KPI summary cards (total applicants, total defaults, overall default rate)
- Six default-rate visualizations, color-coded to distinguish reliable signals from confounded ones
- Explicit caveat annotations directly on charts where the underlying pattern required context
- Interactive filters/slicers for cross-filtering by occupation and age

Full investigative write-ups are included in the repository for transparency:
`income_band_risk_analysis.txt`, `phase4_eda_analysis_occupation_age_income.txt`, `phase4_eda_analysis_loan_cost_region.txt`

### 🔄 Phase 5 — Modeling *(in progress)*
- Baseline: Logistic Regression (interpretable benchmark)
- Comparing against Random Forest, XGBoost, and LightGBM
- Addressing class imbalance via class weighting / SMOTE
- Evaluating using ROC-AUC, Precision, Recall, and F1-score (prioritizing recall on defaulters, given the asymmetric business cost of a missed default)

### ⚪ Phase 6 — Interpretability *(planned)*
- SHAP values to explain individual predictions and global feature importance — important for regulatory transparency in credit decisioning

### ⚪ Phase 7 — Deployment *(planned)*
- Interactive Streamlit app: a loan officer inputs applicant details (income, loan amount, employment length, age) and receives a real-time **Low Risk / High Risk** classification

## Tech Stack

`Python` (pandas, scikit-learn, XGBoost, LightGBM, SHAP) · `SQL` · `Power BI` · `Tableau` · `Streamlit`

## Key Results

*To be completed in Phase 5.*

| Model | ROC-AUC | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic Regression | — | — | — | — |
| Random Forest | — | — | — | — |
| XGBoost | — | — | — | — |
| LightGBM | — | — | — | — |

## Author

Built as a portfolio project demonstrating end-to-end data science skills — from data engineering and critical analytical thinking through to dashboarding and machine learning — for financial services / banking analytics roles.
