# BankChurners – Data Dictionary
_Last updated: 2025-09-16._

This document describes the **processed** fields used for the banking churn project.  
Scope: features in `bankchurners_processed.csv` (after cleaning and light feature engineering).

## Conventions
- **Type**: `numeric` (continuous/integer), `categorical` (string/ordered), or `binary` (0/1).
- **Kind**: role in modeling: `id`, `target`, `feature`, `engineered`.
- **Notes** call out binning, scaling, or caveats relevant to modeling & explainability.

## Columns

| Field | Type | Kind | Description | Example / Valid Values | Notes |
|---|---|---|---|---|---|
| `id` | numeric | id | Stable unique customer identifier (from `CLIENTNUM` or synthetic) | 15580276 | Not used as a model feature. |
| `churn` | binary | target | 1 = Attrited Customer, 0 = Existing Customer | 0/1 | Derived from `Attrition_Flag`. |
| `customer_age` | numeric | feature | Age in years | 45 | Consider scaling. |
| `gender` | categorical | feature | Gender | `M`, `F` | Use with care in targeting (fairness). |
| `dependent_count` | numeric | feature | Number of dependents | 2 | Often right-skewed. |
| `education_level` | categorical | feature | Highest education attained | `Graduate`, `High School`, `Doctorate`, `Unknown` | One-hot encode. |
| `marital_status` | categorical | feature | Marital status | `Married`, `Single`, `Divorced`, `Unknown` | One-hot encode. |
| `income_category` | categorical | feature | Income bucket | `<$40K`, `$40K–$60K`, `$60K–$80K`, `$80K–$120K`, `$120K+`, `Unknown` | Ordered buckets; treat as categorical unless explicitly ordinal-encoded. |
| `card_category` | categorical | feature | Credit card tier | `Blue`, `Silver`, `Gold`, `Platinum` | Often interacts with usage features. |
| `months_on_book` | numeric | feature | Tenure (months with bank) | 36 | Tenure proxy. |
| `total_relationship_count` | numeric | feature | Number of products held | 3 | Higher = deeper relationship. |
| `months_inactive_12_mon` | numeric | feature | Inactive months in last 12 months | 2 | Strong churn signal. |
| `contacts_count_12_mon` | numeric | feature | Bank contacts in last 12 months | 1 | Combine with usage for context. |
| `credit_limit` | numeric | feature | Credit limit ($) | 10,000 | Scale; watch outliers. |
| `total_revolving_bal` | numeric | feature | Revolving balance ($) | 1,250 | |
| `avg_open_to_buy` | numeric | feature | Credit limit − revolving balance ($) | 8,750 | Linear combo of two inputs. |
| `total_amt_chng_q4_q1` | numeric | feature | Change in **amount** (Q4 vs Q1) | 1.23 | Ratio > 1 implies growth. |
| `total_trans_amt` | numeric | feature | Total transaction amount (last 12m) | 5,400 | |
| `total_trans_ct` | numeric | feature | Total transaction count (last 12m) | 80 | Often predictive with inactivity. |
| `total_ct_chng_q4_q1` | numeric | feature | Change in **count** (Q4 vs Q1) | 0.85 | Ratio < 1 implies decline. |
| `avg_utilization_ratio` | numeric | feature | Average credit utilization (0–1) | 0.29 | Consider bin & scale. |
| `avg_trans_amt` | numeric | engineered | Avg transaction amount = total_trans_amt / total_trans_ct | 67.5 | Guarded for divide-by-zero. |
| `trans_per_month` | numeric | engineered | total_trans_ct / 12 | 6.7 | Smoothed activity rate. |
| `contact_rate_12m` | numeric | engineered | contacts_count_12_mon / 12 | 0.17 | |
| `inactive_3plus_12m` | binary | engineered | 1 if months_inactive_12_mon ≥ 3 else 0 | 0/1 | Useful simple flag for rules. |
| `util_band` | categorical | engineered | Binned utilization: `very_low`, `low`, `medium`, `high`, `very_high` | `medium` | Bins: (−0.001,0.1,0.3,0.6,0.9,1.0]. |

## Feature Engineering Notes
- **`avg_trans_amt`** = `total_trans_amt / total_trans_ct` with zero-division handled → 0.  
- **`trans_per_month`** approximates engagement cadence from annual count.  
- **`contact_rate_12m`** normalizes contacts per month.  
- **`inactive_3plus_12m`** is a coarse risk flag-effective for simple segmentation.  
- **`util_band`** provides an interpretable alternative to raw utilization (bins: very_low ≤ 0.1; low ≤ 0.3; medium ≤ 0.6; high ≤ 0.9; very_high ≤ 1.0).

## Modeling Guidance
- **Preprocessing:** one-hot encode categoricals; standardize numerics.  
- **Calibration:** use probabilistic calibration (e.g., isotonic) for decision thresholds.  
- **Fairness:** avoid using purely demographic fields (e.g., gender) in **targeting** decisions; keep for analysis/explainability if needed.  
- **Leakage:** all features reflect behavior **before** the prediction point; do not create features from future outcomes.

## Governance & Documentation
- **Source:** Kaggle “BankChurners” (single-bank, anonymized).  
- **PII:** none in processed data; `id` is derived from `CLIENTNUM` or synthetic.  
- **Versioning:** update this file if columns are added, renamed, or dropped.

---

