# 🏭 Enterprise FMCG Data Engineering Pipeline

A production-grade data engineering pipeline built on **Databricks** and **Apache Spark** to unify and process sales data from a **parent FMCG company** and its **child company** into a single consolidated Gold layer — ready for analytics and dashboarding.

---

## 📌 Project Overview

This project solves a real-world data integration challenge: a parent FMCG company and its acquired child company operate with **different data formats, schemas, and ingestion patterns**. This pipeline:

- Ingests raw CSV data from both companies via **AWS S3**
- Processes data through a **Bronze → Silver → Gold** medallion architecture
- Handles both **full load** and **incremental load** scenarios
- Merges child company data into the parent's unified Gold tables using **Delta Lake MERGE**
- Powers a final **FMCG sales dashboard** via a denormalized reporting table

---

## 🏗️ Architecture

![Project Architecture](resources/project_architecture.png)

| Layer | Description |
|-------|-------------|
| **Bronze** | Raw ingestion from S3 CSVs — no transformations, metadata columns added |
| **Silver** | Cleaned & standardized data — deduplication, null handling, typo fixes, schema alignment |
| **Gold** | Business-ready tables — dimension & fact tables merged with parent company data |

---

## 🗂️ Project Structure
```
project-de-fmcg-atlikon/
│
├── 0_data/
│   ├── 1_parent_company/
│   │   ├── full_load/              # dim_customers, dim_products, dim_gross_price, fact_orders
│   │   └── incremental_load/       # incremental fact_orders + SQL query
│   │
│   └── 2_child_company/
│       ├── full_load/              # customers, products, gross_price, daily orders (Jul–Nov 2025)
│       └── incremental_load/       # daily orders (Dec 2025)
│
├── 1_codes/
│   ├── 1_setup/
│   │   ├── setup_catalog.ipynb             # Create fmcg catalog + Bronze/Silver/Gold schemas
│   │   ├── dim_date_table_creation.ipynb   # Generate dim_date table (monthly grain)
│   │   └── utilities.ipynb                 # Shared schema name variables
│   │
│   ├── 2_dimension_data_processing/
│   │   ├── 1_customers_data_processing.ipynb   # Customer Bronze → Silver → Gold → Merge
│   │   ├── 2_products_data_processing.ipynb    # Products Bronze → Silver → Gold → Merge
│   │   └── 3_pricing_data_processing.ipynb     # Gross price Bronze → Silver → Gold → Merge
│   │
│   └── 3_fact_data_processing/
│       ├── 1_full_load_fact.ipynb              # Full load: daily orders → monthly aggregation → merge
│       └── 2_incremental_load_fact.ipynb       # Incremental load: staging → recalculate affected months → merge
│
├── 2_dashboarding/
│   ├── denormalise_table_query_fmcg.txt    # SQL to build denormalized reporting table
│   └── fmcg_dashboard.pdf                 # Final dashboard output
│
└── resources/
    ├── project_architecture.png            # Architecture diagram
    └── databricks_project.excalidraw       # Editable architecture diagram
```

---

## ⚙️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Databricks** | Notebook execution, catalog & schema management |
| **Apache Spark (PySpark)** | Distributed data processing |
| **Delta Lake** | ACID transactions, Change Data Feed, MERGE operations |
| **AWS S3** | Raw data storage |
| **SQL** | Schema setup, denormalized reporting layer |
| **Power BI / PDF** | Final dashboard delivery |

---

## 🔄 Pipeline Execution Order

Run notebooks in this sequence:
```
1. 1_setup/setup_catalog.ipynb
2. 1_setup/dim_date_table_creation.ipynb
3. 2_dimension_data_processing/1_customers_data_processing.ipynb
4. 2_dimension_data_processing/2_products_data_processing.ipynb
5. 2_dimension_data_processing/3_pricing_data_processing.ipynb
6. 3_fact_data_processing/1_full_load_fact.ipynb
7. 3_fact_data_processing/2_incremental_load_fact.ipynb
```

---

## 📊 Data Models

### Dimension Tables (Gold Layer)

| Table | Key Columns |
|-------|-------------|
| `fmcg.gold.dim_customers` | customer_code, customer, market, platform, channel |
| `fmcg.gold.dim_products` | product_code, division, category, product, variant |
| `fmcg.gold.dim_gross_price` | product_code, price_inr, year |
| `fmcg.gold.dim_date` | month_start_date (monthly grain, 2024–2025) |

### Fact Table (Gold Layer)

| Table | Key Columns |
|-------|-------------|
| `fmcg.gold.fact_orders` | date, customer_code, product_code, sold_quantity |

> **Note:** Child company daily data is aggregated to **monthly grain** before merging with the parent's fact table.

---

## 🔑 Key Features

**Medallion Architecture** — Clean separation of raw, cleaned, and business-ready data across Bronze, Silver, and Gold layers.

**Data Quality Fixes (Silver Layer)**
- Deduplication by primary keys
- City name typo corrections (e.g., `Bengaluruu` → `Bengaluru`)
- Spelling fixes (e.g., `Protien` → `Protein`)
- Title-case standardization for product categories and customer names
- Null city imputation confirmed with business team
- Multi-format date normalization for gross price month field
- Negative and non-numeric price handling

**Schema Alignment** — Child company data is restructured to match the parent company's data model before merging.

**Incremental Load with Affected Month Recalculation** — When new daily orders arrive, the pipeline recalculates all affected months in the Gold fact table and re-merges to ensure accuracy.

**Delta MERGE** — All Gold table updates use `whenMatchedUpdate / whenNotMatchedInsert` for upsert safety.

**Staging Tables** — Incremental loads use temporary staging tables in Bronze and Silver to process only the new data, then clean up after processing.

---

## 📈 Dashboard

The final dashboard is built from a denormalized table joining all Gold-layer dimension and fact tables. The SQL query is available at `2_dashboarding/denormalise_table_query_fmcg.txt` and the output dashboard is at `2_dashboarding/fmcg_dashboard.pdf`.

---
