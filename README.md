# Alternative Credit Risk Scoring System for Kenyan Borrowers

A two-semester final year project (BSc. Data Science, KCA University) that predicts **repayment difficulty among Kenyan credit users** from nationally representative household survey data, with explainability and fairness checks built into the design.

> **Status: in progress.** Data exploration is complete and documented. Modelling, explainability, fairness auditing and deployment are planned for Semester 2. See [Project status](#project-status) for exactly what is and isn't done.

---

## Why this project

Many Kenyan borrowers, particularly users of mobile and digital-app loans, have no formal credit history, so conventional credit scoring cannot assess them. The alternative-data scoring systems that do exist are largely proprietary, and most published research on them relies on lender-held data that other researchers cannot access or reproduce.

This project asks a narrower, reproducible question: **how much repayment risk can be explained using only public, government-collected survey data, and can that be done transparently and fairly?**

What it aims to add beyond applying a gradient-boosted model to a survey:

- A **precisely defined target** (self-reported repayment difficulty, not formal default), with sensitivity analysis on alternative definitions.
- A **full evaluation suite**: ROC-AUC, PR-AUC, precision, recall, F1, confusion matrix and calibration, instead of accuracy alone.
- **SHAP explanations** at both global and individual level.
- A **subgroup fairness audit** (sex, age band, rural/urban) where sample sizes allow.
- A **leakage audit** of every feature against the survey's question timing.
- Everything reproducible from open data.

## Data

**2024 FinAccess Household Survey** (Central Bank of Kenya, Kenya National Bureau of Statistics, FSD Kenya): 20,871 completed household interviews and 3,816 variables.

The data is **not included in this repository**. To reproduce the work:

1. Download the anonymised, weighted dataset (Excel format), the questionnaire and the manual from <https://finaccess.knbs.or.ke/reports-and-datasets>.
2. Save the Excel file as `data/2024_Finaccess_Publicdata.xlsx` at the repository root.
3. Run the notebook. On first run it builds a Parquet cache for faster loading.

Please use the data in line with the terms set by its publishers.

## Target variable

Survey question **E2E**: whether, in the past 12 months, the respondent paid late, missed a payment, paid less, or did not pay at all on a loan.

| Group | Respondents | Share |
|---|---:|---:|
| Always paid on time (`defaulted = 0`) | 6,160 | 47.0% |
| Any repayment difficulty (`defaulted = 1`) | 6,951 | 53.0% |
| **Modelling population** | **13,111** | **100%** |

Excluded: 7,720 respondents who were not asked (no credit use in the period) and 40 who answered "don't know" or refused.

## Preliminary finding

Repayment difficulty varies sharply by loan product among current users. **All figures are unweighted and descriptive, not causal.**

| Loan product | Current users reporting difficulty | n |
|---|---:|---:|
| Hustler Fund | 77.8% | 4,975 |
| Digital (app) loans | 77.2% | 369 |
| Sacco loans | 36.9% | 850 |

Groups with very small samples (for example credit cards, n = 22) are deliberately not interpreted. Full tables are in `findings_log.md`.

## Project status

| Stage | Status |
|---|---|
| Data sourcing and validation | Done |
| Target variable definition (E2E) | Done |
| Feature selection (15 candidate features, no missing values) | Done |
| Exploratory analysis | Done |
| Proposal, SRS, presentation | Drafted |
| Leakage audit, survey weights decision | Planned (Semester 1) |
| Logistic regression baseline | Next |
| Gradient-boosted model, tuning, calibration | Planned |
| SHAP explainability | Planned |
| Subgroup fairness audit | Planned |
| API, dashboard, containerisation, cloud deployment | Planned (Semester 2) |

## Repository structure

```
.
├── notebooks/
│   ├── 01_data_exploration_clean.ipynb   # clean, re-runnable exploration (Sections 1-7)
│   └── archive_scratch.ipynb             # original rough notebook, kept for the record
├── docs/                                  # proposal, SRS, presentation, findings summary
├── findings_log.md                        # running log of confirmed findings
├── .gitignore                             # keeps raw data and caches out of version control
└── README.md
```

## Running the notebook

```bash
pip install pandas openpyxl pyarrow jupyter
```

Open `notebooks/01_data_exploration_clean.ipynb`, choose **Restart Kernel and Run All**, and it should run top to bottom without errors once the data file is in place.

## Planned stack

Python, pandas, scikit-learn, XGBoost or LightGBM, SHAP, FastAPI, Streamlit, Docker, AWS (free tier).

## Limitations

- The target is **self-reported** repayment difficulty, not a formally recorded default, so it may reflect recall or social-desirability bias.
- The survey is **cross-sectional**, so results describe association, not causation, and transaction-level features such as payment velocity or days past due are not available.
- Current statistics are **unweighted**. Survey weights will be applied, or their omission justified, before any population-level claim is made.
- This is an academic prototype. It is **not** a production lending tool and must not be used for real credit decisions.

## Author

Jayson Shammah Ochieng, BSc. Data Science, KCA University.
