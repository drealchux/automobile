# Automobile Sales Analytics

End-to-end analytics project on a scale-model vehicle sales dataset (2,747 order lines / 298
orders / 89 customers / 19 countries, Jan 2018 to May 2020). Two notebooks: one for descriptive,
business-facing exploration, one for predictive modeling and forecasting.

## Notebook 1: Exploratory & Business Intelligence Analysis

`notebooks/Auto_Sales_Analytics.ipynb`

1. How is revenue trending over time, and are there seasonal patterns?
2. Which product lines and products drive the business?
3. How much revenue comes from Small / Medium / Large deals?
4. How healthy is order fulfillment (cancellations, disputes, holds)?
5. Which countries and cities are our strongest and weakest markets?
6. Who are our most valuable customers, and how concentrated is revenue?
7. Which customers are at risk of churning?
8. Are we pricing in line with MSRP, or discounting heavily?

Each section pairs a technical result with a **"What this means for the business"** explanation
aimed at non-technical stakeholders (Sales, Marketing, Ops, Executives).

## Notebook 2: Regression & Forecasting Analysis

`notebooks/Auto_Sales_Regression_Forecasting.ipynb`

- Encodes categorical variables correctly (one-hot for nominal fields, ordinal for the naturally
  ranked `DEALSIZE`).
- Builds two models to predict order revenue: a **Linear Regression** baseline and a
  **Gradient Boosting Regressor**, compared on MAE, RMSE, and R².
- Produces a **6-month revenue forecast** using a 12-month rolling-trend extrapolation,
  cross-checked against a statsmodels Holt's linear trend model.
- Every chart uses Plotly with a consistent multi-color palette (predictions colored by product
  line, model comparisons color-coded per model, forecast lines color-coded by method).

## Project Structure

```
Automobile project/
├── data/
│   └── Auto Sales data.csv                      # Raw source data
├── notebooks/
│   ├── Auto_Sales_Analytics.ipynb               # Exploratory / BI analysis (start here)
│   └── Auto_Sales_Regression_Forecasting.ipynb  # Predictive modeling & forecasting
├── reports/
│   ├── auto_sales_cleaned_line_items.csv   # Full cleaned dataset, line-item grain
│   ├── auto_sales_order_level.csv          # One row per order (status/deal size analysis)
│   ├── auto_sales_customer_summary.csv     # Revenue/orders per customer, Pareto-ranked
│   └── auto_sales_churn_risk.csv           # Customer churn risk tiers with revenue at stake
├── requirements.txt
└── README.md
```

## Setup

A virtual environment (`.venv`) is used to isolate project dependencies.

### 1. Create and activate the virtual environment

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1        # PowerShell
# or: .venv\Scripts\activate.bat  # cmd.exe
```

### 2. Install dependencies

```powershell
pip install -r requirements.txt
```

### 3. Register the Jupyter kernel (first time only)

```powershell
python -m ipykernel install --user --name auto-sales-analytics --display-name "Python 3 (Auto Sales Analytics)"
```

### 4. Launch Jupyter and open a notebook

```powershell
jupyter notebook notebooks/Auto_Sales_Analytics.ipynb
# or: jupyter notebook notebooks/Auto_Sales_Regression_Forecasting.ipynb
```

Select the **"Python 3 (Auto Sales Analytics)"** kernel, then Run All.

## Regenerating the Notebooks

Both notebooks are generated from scripts rather than hand-edited, so either can be rebuilt
cleanly or extended with new sections:

```powershell
python src/build_notebook.py                 # rebuilds Auto_Sales_Analytics.ipynb
python src/build_regression_notebook.py       # rebuilds Auto_Sales_Regression_Forecasting.ipynb
```

## Data Notes

- Source file uses `latin1` encoding (accented characters in European address fields) and
  `DD/MM/YYYY` date format, both handled in the notebook's loading step.
- Data quality was validated: 0 missing values, 0 duplicate rows, and `SALES` is confirmed to
  equal `QUANTITYORDERED × PRICEEACH`.
- `DAYS_SINCE_LASTORDER` is frozen at the time of each transaction in this export; churn-risk
  figures reflect recency as of the data extract, not real time. Re-run against a fresh export
  for current churn signals.
- 2020 data covers only January–May; avoid comparing it to full-year 2018/2019 totals without
  annualizing.

## Key Findings (Summary)

See **Section 15 (Executive Summary)** of the notebook for the full write-up. Headlines:

- Revenue is strongly seasonal, peaking every year in **October–November**.
- **Classic Cars** and **Vintage Cars** dominate product-line revenue.
- **Large deals** are rare by count but drive a disproportionate share of revenue.
- Order fulfillment is healthy: cancellations/disputes are a small, identifiable minority.
- Revenue is geographically concentrated in the **USA**, with a strong European second tier.
- Customer revenue shows **moderate concentration**: the top ~20 customers drive ~45% of
  revenue, a real priority tier, but the broader base isn't over-reliant on a few whales.
- A **churn-risk list**, ranked by revenue at stake, is exported to
  `reports/auto_sales_churn_risk.csv` for Customer Success.
- Pricing tracks MSRP reasonably closely on average, with deal-by-deal negotiation worth a
  policy review with Sales leadership.

## Key Findings, Notebook 2 (Regression & Forecasting)

See **Section 8 (Executive Summary)** of `Auto_Sales_Regression_Forecasting.ipynb` for the full
write-up. Headlines:

- Gradient Boosting outperforms the Linear Regression baseline on order-revenue prediction,
  confirming real non-linear interactions between product line, deal size, and order quantity.
- Quantity ordered, MSRP, product line, and deal size are the strongest revenue drivers per
  feature importance, a useful prioritization signal for Sales and Product leadership.
- A 6-month revenue forecast (rolling trend + Holt's exponential smoothing cross-check) puts
  expected monthly revenue in a similar range across both methods, a reasonable planning
  baseline given only ~29 months of history.
- **Worth flagging:** an early version of the rolling-trend forecast method (fit on a 6-month
  window) produced impossible negative revenue by month 4, an artifact of a single large
  holiday-season spike (Nov 2019) skewing a short trend-fitting window. Widening the window to
  12 months (a full seasonal cycle) fixed this. The notebook documents the failure and fix
  directly, since it's a useful cautionary example for anyone extending this analysis.
