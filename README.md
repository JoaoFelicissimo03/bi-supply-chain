# 📦 BI Supply Chain Dashboard

> Executive Business Intelligence dashboard for supply chain operations management — built with PostgreSQL, Python and Power BI.

**Status: 🚧 Work in Progress — Week 3 of 3 (Executive Overview page complete)**

---

## 🎯 Business Problem

A logistics company faces delivery delays, overstock in some products and stockouts in others. The Operations Director needs a **daily cockpit** to answer:

- Are we delivering on time?
- Where are we losing money in logistics?
- Which products and regions are most problematic?
- How is gross margin evolving by category?

---

## 🗂️ Dataset

**DataCo Smart Supply Chain for Big Data Analysis** — available on [Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis).

- 180,000+ records | 53 columns
- Due to file size, the dataset is not included in this repository
- Download from Kaggle and place the CSV inside the `/data` folder before running the notebooks

---

## 🏗️ Architecture

Star Schema with 6 tables built in PostgreSQL:

```
fact_orders / fact_shipments
    ├── dim_date
    ├── dim_product
    ├── dim_customer
    └── dim_region
```

![ERD](docs/erd.png)

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| PostgreSQL | Relational database, Star Schema |
| Python (pandas, psycopg2) | ETL pipeline |
| Power BI Desktop | DAX measures, dashboard visuals |
| Power BI Service | Publishing and sharing |
| GitHub | Version control and portfolio |

---

## 🚀 How to Run

1. Clone the repository
2. Download the dataset from Kaggle and place it in `/data`
3. Create a `.env` file in the root folder with your PostgreSQL credentials:

```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=supply_chain
DB_USER=postgres
DB_PASSWORD=your_password
DATA_PATH=data/DataCoSupplyChainDataset.csv
```

4. Install dependencies:
```bash
pip install -r requirements.txt
```

5. Run the notebook `notebooks/01_etl_pipeline.ipynb` top to bottom

---

## 📊 KPIs

| KPI | Logic | Visual | Status |
|---|---|---|---|
| On-Time Delivery Rate | On-time deliveries / Total deliveries | Gauge w/ 85% target | ✅ Done |
| Gross Margin % | AVERAGE of per-order profit margin | Card | ✅ Done |
| Avg Shipping Cost | Total shipping cost / No. of orders | Card | ✅ Done |
| Avg Delay (days) | AVG (real - scheduled days) | Card | ✅ Done |
| Total Revenue | SUM of order revenue | Card | ✅ Done |
| Total Orders | COUNT of orders | Card | ✅ Done |
| Revenue by Geography | SUM revenue, drill-down Market → Region → Country | Filled Map | ✅ Done |
| Top 10 Late Products | COUNT late deliveries by product | Bar chart | ⏳ Page 2 |

A key technical challenge solved this week: the model has two fact tables (`fact_orders` and `fact_shipments`) linked only through `dim_date`, so slicers on Region, Category and Segment weren't reaching shipment-level measures. Fixed using `TREATAS()` in DAX to propagate the filtered `order_id` context from `fact_orders` into `fact_shipments` without relying on a direct model relationship.

---

## 🗓️ Project Timeline

- [x] **Week 1** — Data modelling, PostgreSQL, Python ETL
- [x] **Week 2** — Power BI connection, DAX measures, dashboard wireframe, color theme
- [ ] **Week 3** — Dashboard build *(in progress — Executive Overview page complete, Operations Detail page next)*, publish to Power BI Service, final README

---

*Project started June 2026 | Portfolio project*
