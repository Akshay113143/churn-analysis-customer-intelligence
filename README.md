# Churn Analysis and Customer Intelligence

An end-to-end customer churn analytics project that combines SQL-based data engineering with
Python (pandas) for cleaning, KPI calculation, segment analysis and visualisation — built on a
**1,000-customer, 742-support-ticket** subscription dataset to explain *why* customers churn,
*which segments* churn, and *how much revenue* is at risk.

## Overview

The project consolidates customer, subscription, and support data (provided as an Excel workbook)
into a SQLite database, cleans and merges it into a single analytical table, derives 20+ business
KPIs, and then drills into segment-level churn intelligence: a contract × plan churn matrix, a
churn-driver lift table, support-experience effects, risk-score validation, and revenue impact.

## Dataset

- **Source file**: `customer_churn_data_raw.xlsx`
- **Scale**: 1,000 customers · 1,000 subscription records · 742 support tickets (526 unique complainants)
- Loaded into a local SQLite database (`customer_churn.db`) with three tables:
  - **db_customer** — demographics (name, gender, DOB, state, country, interests, pincode)
  - **db_subscription** — plan type, contract type, acquisition channel, monthly charges, CLTV,
    vendor churn score, subscription / renewal / cancellation dates, cancellation reason
  - **db_support** — one row per support ticket: complaint date, escalation flag, CSAT score, free-text comment

The raw workbook deliberately contains real-world data-quality problems: inconsistent gender labels
(`Men`/`Women` vs `Male`/`Female`), 80 rows with a missing `country`, empty junk columns
(`pincode`, `col_1`), free-text noise, dates stored as strings, and multiple support rows per customer.

## Workflow

1. **Data Import** — Every Excel sheet is written to a SQLite table, then read back into pandas
   via SQL queries (`sqlite_master` used to discover tables dynamically).
2. **Data Cleaning**
   - Standardised column names and categorical values (`Men` → `Male`, `Women` → `Female`)
   - Converted all date columns to proper `datetime`
   - Recovered 80 missing `country` values from a state → country mapping
   - Dropped non-analytical columns (`interests`, `pincode`, `col_1`, free-text `comment`)
   - Deduplicated 742 support tickets down to one row per customer, keeping the latest ticket and
     retaining a `complaint_count`
3. **Data Merging** — customer + subscription + support joined on `customerid` into one analytical
   table of 1,000 rows × 27 columns.
4. **Feature Engineering**
   - `churn_flag` — derived from the presence of a cancellation date
   - `tenure_days` — days active (to cancellation, or to today for active customers)
   - `churn_risk` — low / med / high bands from the churn score
   - `complaints_total`, `escalated_any`, `avg_csat` — per-customer support aggregates computed in
     **SQL** over the full ticket table (the dedup step alone would understate escalations)
   - `score_decile` — risk-score deciles used for lift analysis
5. **KPI Calculation (20+ metrics)** — churn & retention rate, churn by plan / contract /
   acquisition channel / gender / state, ARPU, average tenure, CLTV by churn status, revenue at
   risk, escalation rate, average complaints per user, CSAT by churn status, escalation–churn
   correlation, risk-segment distribution.
6. **Segment-Level Churn Intelligence (Section 5)**
   - Contract × plan churn matrix (worst cell isolated)
   - Churn-driver table with lift vs the base rate for every categorical driver
   - Support-experience analysis (churn rate by ticket volume and by escalation)
   - Risk-score validation via a decile lift table
   - Revenue impact: MRR lost, annualised revenue at risk, concentration by segment and reason
   - Tenure-at-cancellation distribution, state-level churn (n ≥ 30 filter)
7. **Visualisation** — monthly churn trend with a 6-month rolling average, churn by contract /
   plan / state, correlation heatmap, segment-matrix heatmap, decile-lift chart, tenure and
   ticket-volume charts.

## Headline Results

| KPI | Value |
|---|---|
| Customers analysed | 1,000 |
| Support tickets processed | 742 |
| Overall churn rate | **29.0%** (retention 71.0%) |
| Monthly-contract churn | **47.2%** |
| Annual-contract churn | **6.3%** (7.5× gap) |
| Worst segment (Monthly + Basic) | **63.1%** churn |
| Basic vs Premium plan churn | 38.5% vs 18.2% |
| Churn if a ticket was escalated | 52.9% vs 22.8% (no escalation) |
| Avg CSAT — churned vs retained | 37.5 vs 69.3 (r = −0.68) |
| ARPU | 14.39 |
| Avg tenure (all customers) | 1,100 days |
| Avg tenure at cancellation | 678 days; 39% of churn inside 12 months |
| MRR lost to churn | 25.6% of book (91.3% of it from monthly contracts) |
| Risk score | Top 3 deciles = 30% of the base, capture **70.7%** of all churners |

## Tech Stack

- Python (pandas, numpy, matplotlib, seaborn)
- SQLite3 (SQL ingestion + per-customer aggregation)
- Jupyter Notebook

## Files

- `Churn_Analysis_and_Customer_Intelligence.ipynb` — main analysis notebook (executed, with outputs)
- `customer_churn_data_raw.xlsx` — raw input dataset (3 sheets)
- `customer_churn.db` — SQLite database built from the workbook
- `exported_churn_data.csv` — cleaned, merged, feature-engineered dataset (1,000 × 27) for dashboarding
- `kpi_summary.csv` — the consolidated KPI table
- `generate_dataset.py` — script used to build the raw workbook

## Key Insights

1. **Contract type is the dominant churn driver.** Monthly contracts churn at 47.2% vs 6.3% for
   annual — a 7.5× gap that survives every other cut of the data, and monthly customers account
   for 91.3% of all lost recurring revenue.
2. **Low-price plans compound the contract effect.** The Monthly + Basic cell churns at 63.1%,
   more than 2× the base rate; Annual + Standard churns at 3.2%.
3. **Support experience is an early-warning signal.** Churn rises monotonically with ticket volume
   (16.0% → 35.4% → 47.1% → 68.8%), and an escalation more than doubles churn (52.9% vs 22.8%).
4. **The risk score is usable for targeting.** Contacting only the top 3 score deciles reaches 30%
   of the customer base but covers 70.7% of everyone who eventually churned.
5. **Churn is not just an onboarding problem.** 39% of cancellations happen inside the first year,
   but the median customer cancels at 476 days — retention effort has to extend past year one.
