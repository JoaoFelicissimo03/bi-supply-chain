# 📦 BI Supply Chain Dashboard

> An end-to-end Business Intelligence project simulating an executive cockpit for supply chain operations — built with PostgreSQL, Python, and Power BI. HEE HEE

![Dashboard Preview](docs/dashboard_preview.png)

---

## 📌 Table of Contents

- [Business Problem](#-business-problem)
- [Dataset](#-dataset)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Dashboard](#-dashboard)
- [KPIs & DAX Measures](#-kpis--dax-measures)
- [Key Technical Challenges](#-key-technical-challenges)
- [Conclusions](#-conclusions)
- [How to Run](#-how-to-run)
- [Project Structure](#-project-structure)

---

## 🎯 Business Problem

A logistics company is experiencing delivery delays, overstock in some product lines and stockouts in others. The Operations Director needs a **daily executive cockpit** that answers four critical questions at a glance:

- Are we delivering on time — and if not, where are we failing?
- Where are we losing money in logistics?
- Which products and regions are the most problematic?
- How is gross margin evolving over time?

This dashboard was designed to answer all four questions without requiring the Director to run a single query or open a spreadsheet.

---

## 🗂️ Dataset

| Field | Detail |
|---|---|
| **Name** | DataCo Smart Supply Chain for Big Data Analysis |
| **Source** | Kaggle |
| **Link** | [kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis) |
| **Volume** | 180,519 records — 53 columns |
| **Format** | CSV |
| **Period** | 2015–2018 |

The dataset contains order, shipment, product, customer, and region data in a single flat file — making it a realistic starting point for Star Schema modelling, since real-world data rarely arrives pre-structured.

> **Note:** The dataset is not included in this repository due to file size. Download it from Kaggle and place it in the `/data` folder before running the ETL notebook.

---

## 🏗️ Architecture

The data was modelled into a **Star Schema** with 6 tables in PostgreSQL, separating facts from dimensions for optimal query performance and Power BI compatibility.

```
fact_orders
├── order_id (PK)
├── customer_id (FK → dim_customer)
├── product_id (FK → dim_product)
├── date_id   (FK → dim_date)
├── region_id (FK → dim_region)
├── quantity_ordered
├── unit_price
├── total_revenue
└── profit_margin

fact_shipments
├── shipment_id (PK)
├── order_id    (FK → fact_orders)
├── date_id     (FK → dim_date)
├── shipping_mode
├── days_for_shipping_real
├── days_for_shipment_scheduled
├── late_delivery_flag
└── shipping_cost
```

```
dim_product    →  product_id, product_name, category, department, unit_cost
dim_customer   →  customer_id, customer_segment, market, country
dim_date       →  date_id, full_date, year, quarter, month, month_name, week, is_weekend
dim_region     →  region_id, region, country, market
```

![ERD](docs/erd.png)

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **PostgreSQL** | Relational database — Star Schema design and storage |
| **Python** (pandas, psycopg2, python-dotenv) | ETL pipeline — data extraction, transformation and loading |
| **Jupyter Notebook** | Interactive ETL development and documentation |
| **Power BI Desktop** | Data modelling, DAX measures, and dashboard visuals |
| **Power BI Service** | Publishing and sharing |
| **GitHub** | Version control and portfolio |

---

## 📊 Dashboard

The dashboard is structured across **two pages**, each serving a distinct audience need.

### Page 1 — Executive Overview

Designed for the Operations Director's daily briefing. Answers the question: *"How are we performing right now?"*

| Visual | Content |
|---|---|
| Gauge | On-Time Delivery Rate vs. 85% industry benchmark |
| Card | Gross Margin % |
| Card | Average Shipping Cost |
| Card | Average Delay Days |
| Card | Total Orders |
| Card | Total Revenue |
| Filled Map | Revenue by Geography — drill-down from Market → Region → Country |

Slicers: **Year range** (date slider), **Region**, **Category**, **Customer Segment**

### Page 2 — Operations Detail

Designed for deeper operational analysis. Answers the question: *"Where exactly is the problem and why?"*

| Visual | Content |
|---|---|
| Bar Chart (horizontal) | Top 10 Products by Late Deliveries |
| Line Chart | Gross Margin % Trend (2015–2018) with drill-down by Year → Quarter → Month |
| Table | Performance by Region — Revenue, On-Time Rate, Gross Margin % |

Slicers are **synchronised** across both pages — a filter applied on Page 1 persists when navigating to Page 2.

---

## 📐 KPIs & DAX Measures

All measures are stored in a dedicated `_Measures` table for organisation.

```dax
Total Revenue =
SUM(fact_orders[total_revenue])

Total Orders =
COUNTROWS(fact_orders)

Gross Margin % =
AVERAGE(fact_orders[profit_margin])

Avg Shipping Cost =
DIVIDE(
    SUM(fact_shipments[shipping_cost]),
    COUNTROWS(fact_orders)
)

Avg Delay Days =
VAR FilteredOrders = VALUES(fact_orders[order_id])
VAR AvgDelay =
    CALCULATE(
        AVERAGEX(
            fact_shipments,
            fact_shipments[days_for_shipping_real] - fact_shipments[days_for_shipment_scheduled]
        ),
        TREATAS(FilteredOrders, fact_shipments[order_id])
    )
RETURN FORMAT(AvgDelay, "0.00") & " days"

On-Time Delivery Rate =
VAR FilteredOrders = VALUES(fact_orders[order_id])
RETURN
DIVIDE(
    CALCULATE(
        COUNTROWS(fact_shipments),
        fact_shipments[late_delivery_flag] = FALSE(),
        TREATAS(FilteredOrders, fact_shipments[order_id])
    ),
    CALCULATE(
        COUNTROWS(fact_shipments),
        TREATAS(FilteredOrders, fact_shipments[order_id])
    )
)

Late Deliveries by Product =
VAR FilteredOrders = VALUES(fact_orders[order_id])
RETURN
CALCULATE(
    COUNTROWS(fact_shipments),
    fact_shipments[late_delivery_flag] = TRUE(),
    TREATAS(FilteredOrders, fact_shipments[order_id])
)

On-Time Target = 0.85
```

---

## 🔧 Key Technical Challenges

Several non-trivial problems were encountered and solved during this project. These are worth documenting both for transparency and as evidence of real-world problem solving.

### 1. Timestamp duplicates in dim_date

The raw `order date` column contained full timestamps (e.g. `2018-01-13 12:27:00`), causing `drop_duplicates()` to treat each unique minute as a distinct date — producing ~65,000 rows instead of the expected ~1,100 unique dates. Fixed by converting with `pd.to_datetime()` and stripping the time component with `.dt.date` before deduplication.

### 2. PostgreSQL connection state errors

After a failed SQL transaction, psycopg2 enters an `InFailedSqlTransaction` state and blocks all subsequent commands. Fixed with `conn.rollback()` to reset the connection before retrying.

### 3. Composite key for dim_region

Region names are not globally unique — "Eastern Asia" maps to both China and Japan. A single `region` column as the join key produced an `InvalidIndexError` during the pandas lookup. Fixed by building a composite `(region, country)` index for the lookup, ensuring uniqueness.

### 4. Boolean type mismatch on insertion

The `late_delivery_flag` column in the source CSV is stored as integer (0/1). PostgreSQL's BOOLEAN type rejected the integer values on insert. Fixed with `.astype(bool)` before loading.

### 5. Cross-filtering between two fact tables

The model has two fact tables — `fact_orders` and `fact_shipments` — connected through `dim_date` and a 1:1 `order_id` relationship. Slicers connected to `dim_region`, `dim_product`, and `dim_customer` filtered `fact_orders` correctly but did not propagate to `fact_shipments`, leaving shipment-level KPIs (On-Time Delivery Rate, Avg Delay Days) unresponsive to those filters.

Solved using `TREATAS()` in DAX — a function that applies a table of values as a filter to a column in another table, bypassing the need for a direct model relationship:

```dax
VAR FilteredOrders = VALUES(fact_orders[order_id])
RETURN CALCULATE(
    ...,
    TREATAS(FilteredOrders, fact_shipments[order_id])
)
```

This propagates the filtered `order_id` context from `fact_orders` into `fact_shipments`, making all slicers work correctly across both fact tables.

---

## 💡 Conclusions

*(Full conclusions section to be added upon project completion)*

Key findings from the data (2015–2018):

- **On-Time Delivery Rate of 45.18%** — well below the 85% industry benchmark, indicating systemic delivery problems across all regions, not isolated incidents.
- **Gross margin declined from ~12% in 2015–2017 to 9.23% in 2018**, suggesting increasing logistics costs or pricing pressure in the most recent year.
- **Nike Men's CJ Elite is the most delayed product** with 6.8K late deliveries — nearly double the second-worst product. This warrants immediate investigation at the supplier or warehouse level.
- **Central America and Western Europe** generate the highest revenue ($1.6M and $1.4M respectively) but show below-average on-time delivery rates, meaning the highest-value markets are also the most at-risk.

---

## 🚀 How to Run

### Prerequisites

- Python 3.8+
- PostgreSQL 13+
- Power BI Desktop (free)

### Setup

1. Clone this repository:
```bash
git clone https://github.com/your-username/bi-supply-chain.git
cd bi-supply-chain
```

2. Install Python dependencies:
```bash
pip install -r requirements.txt
```

3. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis) and place it in `/data`:
```
data/DataCoSupplyChainDataset.csv
```

4. Create a PostgreSQL database named `supply_chain`.

5. Create a `.env` file in the root folder:
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=supply_chain
DB_USER=postgres
DB_PASSWORD=your_password
DATA_PATH=data/DataCoSupplyChainDataset.csv
```

6. Run the SQL script to create the schema:
```bash
psql -U postgres -d supply_chain -f sql/01_create_tables.sql
```

7. Run the ETL notebook top to bottom:
```
notebooks/01_etl_pipeline.ipynb
```

8. Open `powerbi/supply_chain.pbix` in Power BI Desktop.

---

## 📁 Project Structure

```
bi-supply-chain/
│
├── data/                              ← not tracked by Git (file too large)
│   └── DataCoSupplyChainDataset.csv
│
├── notebooks/
│   └── 01_etl_pipeline.ipynb         ← Python ETL: extract, transform, load
│
├── sql/
│   └── 01_create_tables.sql          ← Star Schema DDL
│
├── powerbi/
│   ├── supply_chain.pbix             ← Power BI report file
│   └── dashboard_background.pptx    ← Custom slide backgrounds
│
├── docs/
│   ├── erd.png                       ← Entity-Relationship Diagram
│   └── dashboard_preview.png         ← Dashboard screenshot
│
├── .gitignore
├── requirements.txt
└── README.md
```

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

*Built in June 2026 — Portfolio project by João Ferreira*  
*Dataset: DataCo Smart Supply Chain for Big Data Analysis (Kaggle)*
