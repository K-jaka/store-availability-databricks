# Store Availability Analytics on Databricks (PySpark + Delta Lake)

**Question:** Which stores and products lose sales because product isn't on the shelf, what does it cost, and where should we act first?

!(Screenshot1.png)
!(Screenshot2.png)
!(Screenshot3.png)


## Data
M5 Forecasting (Walmart): 10 stores, 3,049 products, daily sales 2011–2016. ~__M store-product-day rows after cleaning.
Data from Kaggle (not included in this repo): <https://www.kaggle.com/competitions/m5-forecasting-accuracy>

## Architecture
CSV (Unity Catalog volume) → **bronze** (raw Delta) → **silver** (one row per product × store × day) → **data quality gate** → **gold** (KPIs) → AI/BI dashboard.
Orchestrated as a Databricks job; new data loads incrementally with Delta MERGE (idempotent: a rerun adds 0 rows).

## KPI definitions
- **Velocity:** average daily units over the previous 28 days
- **Likely out-of-stock:** a run of k zero-sales days where P(k zeros | Poisson(velocity)) = exp(−velocity·k) < 1%
- **Availability %:** 1 − likely-OOS product-days / all product-days
- **Estimated lost units / revenue:** velocity × OOS days (× price)
- **ABC:** A = products making up the top 80% of a store's revenue (last 90 days)

## Data quality
- Pre-launch zeros removed (M5 fills days before a product's first sale with 0; otherwise new products look out of stock for years)
- Checks: no negative units, no null dates, unique product-day, price coverage, 10 stores; results logged to `dq_results`; the pipeline stops on failure

## Findings
- ___ (e.g. lowest-availability store and its rate)
- ___ (e.g. share of lost revenue from A-items)
- ___

## Limitations
- No inventory data: out-of-stocks are **inferred** from sales, not observed
- Slow sellers are hard to judge; promotions and events can also cause zero runs

## Next steps
Add stock-on-hand and deliveries → weeks of cover and stock turn; alerts for A-items going out of stock.

## Notebooks
`00_setup` · `01_bronze_ingest` · `silver_logic` · `02_silver_sales_daily` · `03_data_quality` · `04_gold_kpis` · `05_incremental_merge`
