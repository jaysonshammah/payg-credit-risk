# Findings Log — Alternative Credit Risk Scoring System

A running record of what's been confirmed during data exploration (Sections 1–7 of
`notebooks/01_data_exploration_clean.ipynb`). Written in plain sentences so it can be
lifted directly into the Work in Progress document later.

---

## 1. Data source — confirmed

- **Dataset:** 2024 FinAccess Household Survey (Central Bank of Kenya, KNBS, FSD Kenya).
- **Full sample:** 20,871 households, 3,816 variables — matches the survey's official
  reported sample size, confirming the dataset loaded correctly (not a truncated subset).
- Loaded via a Parquet cache for speed, with the original Excel file as the one-time
  source of truth (see Section 2 of the notebook).

## 2. Target variable — confirmed

- **Target column:** `E2E` — "In the past 12 months, were you late paying any of your
  loans, missed a payment, paid less, or never paid any amount?"
- **Modelling population:** 13,111 respondents who reported using credit in the past
  12 months and gave a usable answer (excludes 7,720 who weren't asked, and 40 who
  answered "don't know" or refused).
- **Class balance (binary target, `defaulted`):**
  - 0 (always paid on time): 6,160 respondents — 47.0%
  - 1 (any repayment difficulty): 6,951 respondents — 53.0%
- This is a healthy, close-to-even class balance — no aggressive resampling
  (SMOTE, class weighting, etc.) should be needed for the baseline model.

## 3. Feature set — confirmed

15 candidate features selected, **zero missing values across all of them**:

| Group | Columns | What they capture |
|---|---|---|
| Loan-type usage | `C1_14`–`C1_23` (10 columns) | Whether the respondent currently uses, used to use, or never used each of 10 credit products (credit card, mobile banking loan, mobile provider loan, Sacco loan, microfinance loan, digital loan, shylock, group/chama loan, government institution loan, Hustler Fund) |
| Asset ownership | `B1L__4`, `B1L__5` | Purchase of machinery/equipment/tools for business or for farming/livestock in the past 12 months |
| Demographic / geographic | `county`, `A07`, `A08` | County, sex, age-related context |

Full missing-value columns are screening-type (yes/currently/used-to/never) questions,
which is almost certainly why there's no missingness — unlike open-ended questions
(e.g. income amounts), these appear to require a response.

## 4. Exploratory finding — default rate by loan type

Computed with sample size (`n`) alongside every rate, since percentages alone are
misleading on small groups.

**Reportable (large n, clear signal):**

| Loan type | Current users: default rate | n |
|---|---:|---:|
| Hustler Fund (C1_23) | 77.8% | 4,975 |
| Digital loans (C1_19) | 77.2% | 369 |
| Sacco loans (C1_17) | 36.9% (notably *lower*) | 850 |

**Not reportable — sample too small to trust:**
- Credit card (C1_14), "Currently use": n = 22
- C1_15, "Not stated": n = 1

**Interpretation (preliminary, unweighted):** repayment difficulty is strongly
associated with digital/app-based lending products (Hustler Fund, digital loans),
and notably lower among Sacco borrowers. This is consistent with publicly reported
concerns about Kenya's digital lending sector (see literature review) and gives the
project a real, defensible headline finding — but it is descriptive/associative at
this stage, not yet a modelled, controlled result.

## 5. Data quality notes (for the Limitations section)

- The target is **self-reported** repayment difficulty, not a formally recorded
  default — may be subject to recall or social-desirability bias.
- The survey is **cross-sectional** (one interview per household) — all figures
  describe association, not causation, and no transaction-level/time-series
  features (payment velocity, days-past-due) are available.
- All figures in this log are **unweighted**. Official FinAccess sampling weights
  should be applied, or their omission explicitly justified, before any
  population-level claims are made in the final report.

## 6. Known code issues fixed during this phase

For the record (useful if asked how the pipeline evolved):
- Original load used `nrows=1000` during initial exploration — caught and removed;
  full dataset (20,871 rows) confirmed afterward.
- Early keyword search for relevant variables included overly broad terms
  (`food`, `income`) that pulled in unrelated survey questions — removed.
- First loan-type-by-default-rate table looped over *all* `C1_*` columns, which
  accidentally mixed in bank-name sub-columns and savings questions — fixed by
  filtering to only the confirmed main loan-type columns.
- First `pd.concat` of per-loan-type tables produced a sprawl of mismatched,
  mostly-empty columns because each mini-table's grouped column kept its own
  name (`C1_14`, `C1_15`, ...) — fixed by setting `counts.index.name =
  'usage_status'` before concatenating, so all mini-tables share one column.

## 7. Next steps

- Train logistic regression baseline (Section 8).
- Train and tune gradient-boosted model (XGBoost/LightGBM), benchmark against baseline.
- Apply SHAP for global and local explainability.
- Run subgroup fairness check (sex, age band, rural/urban) once sample sizes allow.
