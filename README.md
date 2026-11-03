# Credit Risk & Loan Default Prediction

An end-to-end credit risk analysis: from raw lending data to engineered features, exploratory insights, an interactive Power BI dashboard and (next) a default-prediction model. Built on the Home Credit dataset, with the business framing adapted to the South African lending context.

## Table of Contents

- [Business Problem](#business-problem)
- [Project Status](#project-status)
- [Key Findings So Far](#key-findings-so-far)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [Tech Stack](#tech-stack)
- [Author](#author)

## Business Problem

Lenders lose revenue when borrowers default. Being able to judge an applicant's default risk *before* approval affects profitability, regulatory compliance and customer outcomes.

This project is built around five business questions:

| # | Question | Approach |
|---|---|---|
| 1 | Will this applicant default? | Predictive model scoring default probability at application time |
| 2 | Is the applicant already under financial stress elsewhere? | Early-warning signals from credit bureau history |
| 3 | Does the applicant's history with us predict default? | Repayment behaviour on previous loans and installments |
| 4 | Where is default risk concentrated in the portfolio? | Segment-level default rates (income, age, occupation, loan size, region) |
| 5 | Can the applicant sustain the terms offered? | Affordability ratios (debt-to-income, annuity-to-income) |

## Project Status

| Phase | Description | Status |
|---|---|---|
| 1 | Data exploration | ✅ Complete |
| 2 | Data cleaning | ✅ Complete |
| 3 | Feature engineering | ✅ Complete |
| 4 | Exploratory data analysis | ✅ Complete |
| 5 | Power BI dashboard | 🔄 In progress |
| 6 | Modelling | ⚪ Planned |
| 7 | Interpretability (SHAP) | ⚪ Planned |
| 8 | Deployment (Streamlit app) | ⚪ Planned |

## Key Findings So Far

- **Severe class imbalance:** 91.9% of applicants did not default and 8.1% did. A model that always predicts "no default" would be about 92% accurate while catching no defaulters, so evaluation must focus on the default class.
- **Two default-rate patterns looked misleading at first** (income band and loan-amount band). Each was investigated rather than taken at face value:
  - **Income band:** the lowest income band contains a much larger share of pensioners (31% vs roughly 10–18% in the other bands). Pensioner income is low but stable, which can mask the true risk of that band.
  - **Loan-amount band:** pensioner share is also highest at the low end, which confounds the pattern.
- **Affordability matters:** the annuity-to-income ratio falls steadily as income rises (about 0.24 in the lowest band to 0.12 in the highest) and rises steadily with loan size (about 0.12 to 0.23). It is a more direct measure of repayment burden than income level alone.
- **Takeaway being tested:** income *stability* and affordability may be better risk signals than raw income *level*. This will be confirmed in modelling.

## Dataset

[Home Credit Default Risk](https://www.kaggle.com/c/home-credit-default-risk) (Kaggle): about 307,500 loan applications plus related tables for credit bureau history, previous applications and repayment behaviour.

**Why this dataset:** real South African consumer credit data is held privately by banks and credit bureaus and is protected by the National Credit Act (NCA) and POPIA, so it is not available for open portfolio work. Home Credit provides the technical foundation, while the business narrative reflects South African lending principles (for example, NCA-style affordability assessment).

## Methodology

### Phase 1: Data Exploration (`01_data_exploration.ipynb`)
- Profiled `application_train.csv` (307,511 rows × 122 columns).
- Measured the class imbalance and mapped missing-data patterns, which are concentrated in property and building fields.
- Identified two data quality issues: a placeholder value in `DAYS_EMPLOYED` and extreme outliers in `AMT_INCOME_TOTAL`.

### Phase 2: Data Cleaning (`02_data_cleaning.ipynb`)
- Flagged and corrected about 55,000 placeholder values (`365243`) in `DAYS_EMPLOYED`.
- Converted `DAYS_BIRTH` into applicant age in years.
- Capped income at the 99.5th percentile in a new column, keeping the original field.
- Added missing-value flags for columns with more than 40% missing, since missingness can itself be predictive.
- Dropped 4 rows with an invalid gender code and one-hot encoded 16 categorical columns.
- Result: 307,507 rows × 313 columns.

### Phase 3: Feature Engineering (`03_feature_engineering.ipynb`)
Aggregated the supporting tables to one row per applicant and merged them in, adding 34 new features:

| Source | Example features |
|---|---|
| Credit bureau | `BUREAU_COUNT`, `BUREAU_OVERDUE_COUNT`, `BUREAU_CREDIT_TYPE_DIVERSITY` |
| Previous applications | `PREV_APPROVAL_RATE`, `PREV_REFUSED_COUNT` |
| Installment payments | `INSTALLMENT_LATE_RATIO`, `INSTALLMENT_STD_DAYS_LATE` |
| POS cash and credit card | `POS_DPD_MAX`, `CC_UTILIZATION_AVG` |
| Affordability | `DEBT_TO_INCOME_RATIO`, `ANNUITY_TO_INCOME_RATIO`, `CREDIT_TERM` |

Applicants with no history in a table get a `NO_*_HISTORY` flag. Counts are filled with 0, while ratios and averages stay missing, because 0 would wrongly suggest "no debt" instead of "no data".

Result: 307,507 rows × 347 columns.

### Phase 4: Exploratory Data Analysis (`04_eda.ipynb`)
Default rates were analysed across six dimensions: income, age, occupation, loan amount, implied cost of credit (credit amount vs goods price) and region rating. A slim dataset (27 columns) is exported for the dashboard.

### Phase 5: Power BI Dashboard *(in progress)*
An interactive dashboard built on the exported EDA dataset, with KPI cards, default-rate visuals and slicers. Charts will carry short notes where a pattern is confounded (see Key Findings).

### Phase 6: Modelling *(planned)*
- Compare Logistic Regression, Random Forest, XGBoost and LightGBM on a stratified 80/20 split.
- Handle class imbalance with class weights (and `scale_pos_weight` for XGBoost).
- Judge models on ROC-AUC, precision, recall and F1 for the default class rather than accuracy.
- Results will be added here once modelling is complete.

### Phase 7: Interpretability *(planned)*
SHAP values for global feature importance and for explaining individual predictions, which matters for transparency in credit decisions.

### Phase 8: Deployment *(planned)*
A Streamlit app where a loan officer enters applicant details and receives a Low Risk / High Risk classification.

## Repository Structure

```
├── data/                    # raw and processed data (not committed; see .gitignore)
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_eda.ipynb
│   └── 05_modeling.ipynb    # planned
├── src/                     # reusable Python modules
├── sql/                     # SQL queries for data analysis
├── dashboards/
│   └── powerbi/             # dashboard data export and .pbix file
├── app/                     # Streamlit risk-scoring app (planned)
└── outputs/
    ├── figures/             # exported chart images
    └── models/              # trained models (planned)
```

## How to Reproduce

1. Download the Home Credit dataset from [Kaggle](https://www.kaggle.com/c/home-credit-default-risk) into `data/raw/`.
2. Install the dependencies: `pip install pandas numpy matplotlib jupyter`
3. Run the notebooks in order. Each one reads the previous notebook's output:

| Notebook | Reads | Writes |
|---|---|---|
| `01_data_exploration` | raw application data | none |
| `02_data_cleaning` | raw application data | `data/processed/application_train_cleaned.csv` |
| `03_feature_engineering` | cleaned data and raw supporting tables | `data/processed/application_train_features.csv` |
| `04_eda` | feature dataset | figures and `dashboards/powerbi/credit_risk_dashboard_data.csv` |

## Tech Stack

`Python` (pandas, NumPy, Matplotlib) · `Power BI` · `SQL` · scikit-learn, XGBoost, LightGBM, SHAP and Streamlit (planned)

## Author

**Kharendwe** · [GitHub: Kharendwenegota](https://github.com/Kharendwenegota)

Built as a portfolio project showing end-to-end analytics skills, from data cleaning and critical analysis through dashboarding to machine learning, for financial services and banking analytics roles.
