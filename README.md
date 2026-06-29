# 📦 BI Supply Chain Dashboard

> Executive Business Intelligence dashboard for supply chain operations management — built with PostgreSQL, Python and Power BI .

**Status: 🚧 Work in Progress — Week 1 of 3 complete**

---

## 🎯 Business Problem

A logistics company faces delivery delays, overstock in some products and stockouts in others. The Operations Director needs a **daily cockpit** to answer

- Are we delivering on time?
- Where are we losing money in logistics?
- Which products and regions are most problematic?
- Ho

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

## 📊 KPIs (in development)

| KPI | Logic | Visual |
|---|---|---|
| On-Time Delivery Rate | On-time deliveries / Total deliveries | Card + Gauge |
| Order Fill Rate | Complete orders / Total orders | Card |
| Gross Margin % | (Revenue - Cost) / Revenue | Card + Trend |
| Avg Shipping Cost | Total shipping cost / No. of orders | Card |
| Avg Delay (days) | AVG (real - scheduled days) | Card |
| Revenue by Region | SUM revenue by region | Map |
| Top 10 Late Products | COUNT late deliveries by product | Bar chart |

---

## 🗓️ Project Timeline

- [x] **Week 1** — Data modelling, PostgreSQL, Python ETL
- [x] **Week 2** — Power BI connection, DAX measures, dashboard wireframe
- [ ] **Week 3** — Dashboard build, publish to Power BI Service, final README

---

*Project started June 2026 | Portfolio project*
