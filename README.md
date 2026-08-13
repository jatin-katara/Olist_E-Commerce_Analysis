# Olist E-Commerce Analytics: SQL → Python → Power BI

An end-to-end data analytics project examining 100,000+ orders from Olist, Brazil's largest online marketplace connector. The project moves through the full analytics pipeline — SQL-based data modeling, Python-driven exploratory analysis and data-quality investigation, and an interactive Power BI dashboard — to answer real business questions about revenue, customer loyalty, delivery performance, and satisfaction.

---

## Business Problem

Olist connects small Brazilian merchants to major online marketplaces under a single contract, generating transactional data across sales, logistics, customer behavior, and satisfaction spanning all 27 Brazilian states.

This project was built to answer four core business questions:

1. **Where is revenue coming from, and how has it trended over time?**
2. **Who are Olist's customers, and how loyal are they?**
3. **How reliable is the delivery network, and where does it break down?**
4. **What drives customer satisfaction — and what gets in the way of measuring it accurately?**

---

## Dataset

**Source:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle)
**Scope:** 9 relational CSV files · ~100,000 orders · September 2016 – October 2018 · 129.6 MB

| Table | Description |
|---|---|
| `olist_orders_dataset` | One row per order — status and timestamps |
| `olist_order_items_dataset` | One row per line item — product, seller, price, freight |
| `olist_customers_dataset` | One row per order-customer — location and unique ID |
| `olist_order_payments_dataset` | One or more rows per order — payment type and value |
| `olist_order_reviews_dataset` | One row per review — score and comment |
| `olist_products_dataset` | One row per product — category, dimensions, weight |
| `olist_sellers_dataset` | One row per seller — location |
| `olist_geolocation_dataset` | Many rows per zip prefix — latitude/longitude |
| `product_category_name_translation` | Portuguese → English category mapping |

---

## Tech Stack

| Layer | Tools |
|---|---|
| Data storage | SQLite |
| SQL querying | Python (`sqlite3`) |
| Data cleaning & EDA | Python (`pandas`, `matplotlib`, `seaborn`) |
| Dashboarding | Power BI |
| Version control | Git / GitHub |

---

## Project Structure

```
project-root/
├── data/                        # Raw Kaggle CSVs (gitignored)
├── olist_ecommerce.db           # SQLite database (gitignored)
├── sql/
│   ├── sales/                   # Revenue, orders, categories, sellers, payments
│   ├── customers/               # State distribution, repeat rate, RFM segmentation
│   ├── logistics/               # Late delivery rate, freight %, state-pair analysis
│   ├── reviews/                 # Score distribution, category/seller averages, correlation
│   └── finances/                # Payment type analysis
├── analysis/
│   ├── 01_data_cleaning.py              # Timestamp parsing, null handling, outlier flagging
│   ├── 02_eda.py                        # Exploratory visualizations
│   ├── 03_delivery_outlier_investigation.py   # Placeholder-timestamp & review anomaly audit
│   └── export_powerbi_data.py           # Exports cleaned CSVs for Power BI
├── exports/
│   ├── cleaned/                 # Intermediate cleaned DataFrames (gitignored)
│   ├── investigation/           # Outlier audit exports (gitignored)
│   └── powerbi/                 # fact_orders_clean, dim_customers_rfm, agg_monthly_revenue (gitignored)
├── plots/                       # EDA visualizations (PNG)
├── dashboard/
│   └── olist_dashboard.pbix     # Power BI report
├── helper/
│   └── utils.py                 # Shared utility functions
├── README.md
└── FINDINGS.md                  # Full analytical write-up
```

---

## Methodology

**1. Data Engineering (SQL)**
Raw CSVs were loaded into a SQLite database and modeled into a query layer organized by business theme — sales, customers, logistics, reviews, and finances — rather than as one monolithic script. This mirrors how analytics teams structure warehouse queries for reuse.

**2. Cleaning & Exploratory Analysis (Python)**
`pandas` was used for timestamp parsing, null handling, and outlier flagging, followed by exploratory visualization to surface initial trends across revenue, geography, and delivery timing.

**3. Data-Quality Investigation**
While validating the relationship between delivery performance and review scores, the review-score distribution turned out to be unexpectedly bimodal and noisy. A targeted investigation traced this to two independent data-quality issues — a backend placeholder-timestamp bug and inconsistent mobile-UI tap behavior on the review form — both of which distort the apparent link between delivery delays and customer satisfaction. This is documented in detail in `FINDINGS.md`.

**4. Customer Segmentation**
An RFM (Recency, Frequency, Monetary) model was built to segment customers into loyalty tiers such as Champions, At Risk, and Lost Customers, feeding directly into the Power BI customer view.

**5. Dashboarding (Power BI)**
Cleaned, aggregated tables were exported from Python and modeled into a two-page interactive Power BI report for executive and operational audiences.

---

## Key Insights

Full write-up in [`FINDINGS.md`](./FINDINGS.md). Headlines:

- **R$15.4M GMV** across 96,478 delivered orders — revenue grew roughly 9x from early 2017 to a stable plateau in 2018, with a clear Black Friday spike in November 2017.
- **São Paulo dominates order volume** at 41.92% of all orders, with the top three states (SP, RJ, MG) accounting for 66.51% of total volume — a strong seller-concentration signal.
- **6.76% late delivery rate** overall, concentrated in northern and northeastern states far from the São Paulo seller base — pointing to a geography-driven logistics gap rather than a systemic one.
- **Review scores are bimodal and noisy**: 57.78% five-star, but one-star is the second most common score. Root-cause analysis traced this to a backend timestamp bug and mobile UI tap behavior contaminating the delivery-satisfaction signal — a data-quality finding, not just a business one.
- **Revenue and volume diverge by category** — Watches & Gifts earns roughly 75% more per order than Bed, Bath & Table despite roughly half the transaction volume, suggesting different margin strategies are warranted by category.

---

## Dashboard

The Power BI report (`dashboard/olist_dashboard.pbix`) contains two pages:

| Page | Contents |
|---|---|
| **Executive Sales & Revenue Overview** | KPI cards (GMV, Orders, Avg Review Score, Late Rate, Repeat Rate), monthly revenue trend, top sellers, top categories, and geographic revenue distribution |
| **Customer & Operational Insights** | Category freight-to-revenue ratio, average review score by delivery status, RFM segment breakdown, and order volume by delivery status |

All visuals are controlled through five interactive slicers across the top of each page:

- **Date Range** — `15-09-2016` to `29-08-2018`
- **Customer State** — multi-select (e.g., AC, AL, AM, SP)
- **Product Category** — multi-select (e.g., agro_industry_and_commerce, air_conditioning)
- **RFM Segment** — Champions, At Risk, Lost Customers, etc.
- **Shipping Type** — Inter-State vs. Intra-State

### Preview
![Dashboard Overview](dashboard/screenshots/dashboard_screenshot1.png)
![Dashboard Overview](dashboard/screenshots/dashboard_screenshot2.png)

---

## How to Reproduce

**Prerequisites:** Python 3.8+, Power BI Desktop (free)

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/olist-ecommerce-analytics.git
cd olist-ecommerce-analytics

# 2. Install dependencies
pip install pandas matplotlib seaborn

# 3. Download the dataset from Kaggle and place all 9 CSVs into /data
#    https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

# 4. Build the SQLite database
python day1_setup.py

# 5. Run the analysis pipeline in order
python analysis/01_data_cleaning.py
python analysis/02_eda.py
python analysis/03_delivery_outlier_investigation.py
python analysis/export_powerbi_data.py

# 6. Open dashboard/olist_dashboard.pbix in Power BI Desktop
#    Update the data source path to your local /exports/powerbi/ folder if prompted
```

---

## Skills Demonstrated

- Relational data modeling and theme-organized SQL query design
- Python-based data cleaning, EDA, and statistical investigation
- Root-cause analysis of a real data-quality issue affecting business metrics
- RFM customer segmentation
- Power BI dashboard design with DAX measures and interactive slicers
- End-to-end pipeline design from raw data to stakeholder-facing reporting

---

## Findings

See [`FINDINGS.md`](./FINDINGS.md) for the complete analytical write-up, organized by theme — Sales, Customers, Logistics, and Satisfaction — with business questions, methods, results, and takeaways for each.
