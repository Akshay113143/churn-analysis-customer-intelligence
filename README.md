# COVID-19 India — Data Analysis & Interactive Dashboard

## Problem Statement

India's COVID-19 case data was reported daily, state-by-state, throughout the pandemic, but raw government-sourced data is cumulative, inconsistently formatted, and contains real-world data-quality issues (name inconsistencies, retroactive revisions, placeholder values). Without structured cleaning and analysis, it's difficult to answer basic operational questions: which states carried the heaviest burden, when did the outbreak actually peak, and how did outcomes (recovery, mortality) differ across regions.

## Business Objective

Build an end-to-end analytics pipeline — from raw public data to an interactive dashboard — that answers:
- Which states/UTs were most and least affected, by volume and by outcome severity?
- When did India's first wave peak, and when did the second wave begin?
- How did recovery and death rates vary across states, and where did case volume alone give a misleading picture of severity?
- Is the data reliable enough to support these conclusions, and where are its limits?

## Dataset

**Source:** ["COVID-19 in India"](https://www.kaggle.com/datasets/sudalairajkumar/covid19-in-india) by sudalairajkumar on Kaggle, itself sourced from India's Ministry of Health & Family Welfare via the covid19india.org tracker.

**Files used:**
- `covid_19_india.csv` — primary dataset, state-wise cumulative Confirmed/Cured/Deaths by date
- `StatewiseTestingDetails.csv` — supplementary testing data (reviewed but not central to core analysis due to significant missingness, see Data Quality below)
- `covid_vaccine_statewise.csv` — supplementary vaccination data (reviewed; only overlaps the final two months of the study window)

**Raw shape:** 18,110 rows × 9 columns, covering 2020-01-30 to 2021-08-11, across 46 raw state/UT label variants.

**Project window:** January 2020 – March 2021, selected because the data itself shows this window captures both the complete first wave (peak: September 2020) and the confirmed, dataset-validated start of the second wave (March 2021 reversal) — a natural analytical boundary rather than an arbitrary cutoff.

## Data Quality Findings (real issues identified in the raw file)

- **State name inconsistencies:** typos and formatting variants (`Karanataka`, `Himanchal Pradesh`, `Telengana`, trailing asterisks on 3 state names) inflated the raw unique-state count from India's real 36 states/UTs to 46 apparent labels.
- **Administrative merger not reflected in source data:** `Daman & Diu` and `Dadra and Nagar Haveli` were reported as separate entities despite merging into one UT in January 2020.
- **Non-state bookkeeping rows:** `Unassigned` and `Cases being reassigned to states` (63 rows) are temporary case-attribution buckets, not real states.
- **Mostly-empty columns:** `ConfirmedIndianNational` / `ConfirmedForeignNational` were placeholder (`-`) in 97.5% of rows — India stopped reporting this split early in the pandemic.
- **18 genuine data-revision anomalies (0.14% of cleaned rows):** cumulative Confirmed decreased vs. the prior date for that state on 18 occasions (e.g., Delhi -10,694 on 2021-03-03, Telangana -12,265 on 2021-02-25) — real government corrections in the source data. These were **flagged, not deleted or silently altered**, preserving the actual reported history while documenting the anomaly.

## Data Cleaning Methodology

1. Standardized 6 state-name typos/formatting variants; merged legacy UT labels into their post-2020 unified name.
2. Removed 63 non-state bookkeeping rows.
3. Dropped `ConfirmedIndianNational`/`ConfirmedForeignNational` (unusable, 97.5% placeholder).
4. Dropped `Time` (only 7 distinct batch-reporting timestamps, no analytical value).
5. Parsed `Date` from text to a true date type.
6. Filtered to the Jan 2020–Mar 2021 project window (18,047 → 13,259 rows).
7. Flagged (did not alter) 18 rows with genuine cumulative-count revisions.
8. Validated: 0 duplicate rows, 0 negative values in Cured/Deaths/Confirmed post-cleaning, and a resulting 36 unique states/UTs — matching India's actual state/UT count, used as an internal correctness check.

**Result:** 13,259 rows × 7 core columns, cleaned and ready for feature engineering.

## Feature Engineering

- Calendar fields: Year, Month, Month_Name, Month_Year, Day, Week
- `Active_Cases = Confirmed − Cured − Deaths` (derived; the source data is cumulative-only)
- `Daily_New_Confirmed/Cured/Deaths` — calculated via `groupby(state).diff()`, chronologically **within each state**, not across the flat file
- `Recovery_Rate`, `Death_Rate` — cumulative-basis percentages, divide-by-zero guarded
- `Growth_Rate_Pct` — day-over-day % change in cumulative confirmed, per state

## Tools Used

| Tool | Role |
|---|---|
| Excel | Initial pivot-table exploration: state-wise and monthly breakdowns, top-10 rankings, recovery/death rate comparisons |
| MySQL | Schema design, indexing strategy, and the full basic + advanced (CTE, window function) query set — all validated against real data |
| BigQuery | Cloud data-warehouse workflow demonstration (partitioning, clustering) — included to show cloud SQL skills; the dataset's actual size (13K rows) does not require it for performance |
| Tableau | 4-dashboard interactive story: Executive Overview, State-wise Analysis, Time-Series Analysis, State Deep Dive |
| Python (pandas) | Data cleaning, feature engineering, and cross-validation of every Excel/SQL result |

## SQL Analysis Summary

Full basic query set (national/state/monthly totals, top-10 rankings, recovery/death rate comparisons) plus an advanced set demonstrating CTEs, `ROW_NUMBER()`/`RANK()` window functions, `LAG()` for month-over-month growth, and rolling 7-day averages. Every query was executed and validated against the cleaned dataset — see the project's SQL scripts for the full, runnable set.

## Tableau Dashboard

Four linked dashboards connected via filter actions:
1. **Executive Overview** — national KPIs, overall trend line with 7-day moving average, summary map
2. **State-wise Analysis** — top-10 states by confirmed/deaths side by side, recovery/death rate comparisons (≥1,000 confirmed filter to avoid small-denominator distortion)
3. **Time-Series Analysis** — full daily/monthly trend with annotated peak (17 Sept 2020) and reversal point (March 2021) reference lines
4. **State Deep Dive** — single-state filtered view of all core metrics over time

## Key Findings

1. Maharashtra accounts for 22.8% of India's total confirmed cases in this window (2,773,436 of 12,149,335) — a highly concentrated burden.
2. India's first-wave peak was September 2020 (2,604,518 monthly new cases; single-day peak 97,894 on 17 Sept 2020), followed by 5 consecutive months of decline before a 200.3% month-over-month reversal in March 2021.
3. Kerala and Maharashtra did not share one epidemic curve: Kerala peaked in October 2020 (11,755/day), Maharashtra peaked nearly 3.5× higher in March 2021 (40,414/day) — 5.5 months later.
4. Case volume is a poor proxy for outcome severity: Kerala ranks #2 by confirmed but #9 by deaths; Tamil Nadu is the inverse (#5 confirmed, #2 deaths).
5. Punjab had the highest death rate (2.88%) among states with meaningful case volume (≥1,000 confirmed).
6. Arunachal Pradesh, Nagaland, and Mizoram had the strongest recovery rates (>99%) among the same filtered set.
7. 18 rows (0.14%) reflect genuine government data revisions, transparently flagged rather than hidden or deleted.

## Conclusion

This project demonstrates a complete, honest data-analytics workflow on real, messy public health data — from identifying and documenting genuine data-quality issues, through cleaning decisions with stated reasoning, to cross-validated analysis across three tools (Excel, SQL, Python) and a four-dashboard interactive Tableau story. Every reported figure in this document was computed directly from the cleaned dataset and cross-checked across at least two of the three analytical tools used.

## Repository Structure

```
├── data/
│   ├── covid_19_india_cleaned.csv       # Post-cleaning (Part 2)
│   ├── covid_19_india_features.csv      # Post-feature-engineering (Part 3)
│   └── state_level_summary.csv          # State-level comparison table (Part 8)
├── sql/
│   ├── schema.sql                        # CREATE TABLE + indexing
│   ├── basic_queries.sql
│   └── advanced_queries.sql              # CTEs, window functions
├── tableau/
│   └── covid_india_dashboard.twbx
└── README.md
```
