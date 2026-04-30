# 🚲 Databricks Bike Data Lakehouse Project

This repository documents an end-to-end Databricks Lakehouse built with the **Medallion Architecture**. It ingests raw CRM and ERP files, cleans and standardizes them, models them into a star schema, and prepares the workflow for orchestration in Databricks.

## 📖 Overview
The project is organized into three data layers and one automation layer. The final goal is to produce reliable analytics tables for reporting and BI:

*   **Bronze:** Raw ingestion with no transformations.
*   **Silver:** Cleansing, standardization, and validation.
*   **Gold:** Business-ready dimensional modeling.
*   **Pipeline:** Orchestration with Databricks Workflows.

---

## 🗂️ Repository Structure
```text
script/
├── init_lakehouse.ipynb
├── bronze/
│   └── bronze_layer.ipynb
├── silver/
│   ├── silver_orchestration.ipynb
│   ├── crm/
│   └── erp/
└── gold/
    ├── gold_orchestration.ipynb
    ├── gold_dim_customers.ipynb
    ├── gold_dim_products.ipynb
    └── gold_fact_sales.ipynb
```

---

## ⚙️ Setup Instructions

**1. Prepare the workspace**
Use a Databricks workspace with Unity Catalog enabled. Create or confirm the following schemas:
*   `bronze`
*   `silver`
*   `gold`

**2. Create the raw file volume**
Create a volume in the Bronze schema to act as the landing zone:
`workspace.bronze.raw_sources`

**3. Upload source data**
Place the six source CSV files into the `raw_sources` volume.

**4. Initialize the lakehouse**
Run the initialization script. This notebook prepares the schemas and storage needed for the project.
```bash
script/init_lakehouse.ipynb
```

---

## 🚀 Execution Flow

### 🥉 Bronze Layer
Run the ingestion notebook:
```bash
script/bronze/bronze_layer.ipynb
```
This notebook reads each source CSV and writes it as a Delta table in `workspace.bronze` using a consistent naming convention.

### 🥈 Silver Layer
Run the orchestration notebook:
```bash
script/silver/silver_orchestration.ipynb
```
This notebook triggers the Silver transformation notebooks in sequence. The Silver layer cleans, standardizes, and validates data before writing to `workspace.silver`.

### 🥇 Gold Layer
Run the dimensional modeling notebook:
```bash
script/gold/gold_orchestration.ipynb
```
This notebook builds the dimensional model in `workspace.gold`, including customer, product, and sales tables for analytics.

---

## 📊 Data Model Outputs

**Silver Outputs**
The Silver layer produces cleaned, standardized tables such as:
*   `workspace.silver.crm_customers`
*   `workspace.silver.crm_products`
*   `workspace.silver.crm_sales`
*   `workspace.silver.erp_customers`
*   `workspace.silver.erp_locations`
*   `workspace.silver.erp_product_categories`

**Gold Outputs**
The Gold layer creates the final Star Schema:
*   `workspace.gold.dim_customers`
*   `workspace.gold.dim_products`
*   `workspace.gold.fact_sales`

---

## 🔄 Pipeline Automation
The pipeline is designed to run as a Databricks Workflow named `loading_bike_data_lakehouse`. 

**Recommended Task Order:**
1.  Bronze ingestion
2.  Silver orchestration
3.  Gold orchestration



---

## 🎯 Purpose & Usage
This project demonstrates a practical Databricks Lakehouse implementation using:
*   Medallion Architecture
*   Unity Catalog governance
*   Delta tables
*   Notebook-based ETL
*   Dimensional modeling
*   Workflow orchestration


```
