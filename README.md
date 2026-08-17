# Churn Analysis and Customer Intelligence

An end-to-end customer churn analysis project that combines SQL-based data engineering with Python (pandas) for cleaning, KPI calculation, and visualization to uncover why customers churn and where revenue is at risk.

## Overview

The project consolidates customer, subscription, and support data (originally provided as an Excel workbook) into a SQLite database, cleans and merges it into a single analytical dataset, and derives key churn and retention metrics along with supporting visualizations.

## Dataset

- **Source file**: `customer_churn_data_raw.xlsx`
- Loaded into a local SQLite database (`customer_churn.db`) with three main tables:
  - **Customer** — demographic details (name, gender, DOB, state, country, etc.)
  - **Subscription** — plan type, monthly charges, subscription/renewal/cancellation dates
  - **Support** — customer complaints, complaint dates, escalations

## Workflow

1. **Data Import** — Read Excel sheets into a SQLite database, then load tables into pandas DataFrames.
2. **Data Cleaning**
   - Standardized column names and categorical values (e.g. gender labels)
   - Converted date columns to proper datetime format
   - Filled missing `country` values using a state-to-country mapping
   - Dropped irrelevant columns (e.g. `interests`, `pincode`, free-text comments)
   - Deduplicated support records, aggregating complaint counts per customer
3. **Data Merging** — Combined customer, subscription, and support tables into a single analytical DataFrame on `customerid`.
4. **Feature Engineering**
   - `churn_flag` — derived from presence of a cancellation date
   - `tenure_days` — customer tenure in days (active or until cancellation)
   - `churn_risk` — categorized as low / medium / high based on churn score
5. **KPI Calculation**
   - Churn Rate & Retention Rate
   - Churn Rate by Plan Type
   - Average Revenue Per User (ARPU)
   - Average Customer Tenure
   - Revenue at Risk (from churned customers)
   - Escalation Rate
   - Average Complaints per User
   - Correlation between Escalations and Churn
6. **Visualization** — Monthly churn trend, churn rate by plan type, churn rate by state, and other exploratory plots using matplotlib/seaborn.

## Tech Stack

- Python (pandas, numpy, matplotlib, seaborn)
- SQLite3
- Jupyter Notebook

## Files

- `Churn_Analysis_and_Customer_Intelligence.ipynb` — main analysis notebook
- `customer_churn_data_raw.xlsx` — raw input dataset
- `exported_churn_data.csv` — cleaned and merged dataset exported for further use (e.g. dashboarding)

## Key Insights

The analysis surfaces churn drivers such as plan type, geographic patterns, and support escalations — enabling risk-based customer segmentation and identifying revenue at risk from churned users.
