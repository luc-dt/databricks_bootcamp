# Databricks Bootcamp 2026

Welcome to the **Databricks Data Lakehouse Project** 

This repository contains a complete, real-world **Data Lakehouse implementation** built on Databricks, including datasets, notebooks, SQL examples, and exercises. Everything here is designed to help you understand how modern data teams use Databricks in practice, from data ingestion and transformation to analytics-ready data products.

## 🏗️ Architecture

This project follows the **Medallion Architecture**:

### 🥉 Bronze Layer
- Raw data ingestion  
- Schema inference and storage as Delta tables  

### 🥈 Silver Layer
- Data cleaning and standardization  
- Type casting and validation  

### 🥇 Gold Layer
- Dimensional Data Model (Business Transformation)
- Ready for BI and analysis  

---

## 📊 Data Processing Workflow

This section describes the complete end-to-end data processing pipeline implemented in the `bike_lakehouse_2026` notebooks.

### **Step 1: Bronze Layer - Raw Data Ingestion**
**Notebook:** `Bronze.ipynb`

**Purpose:** Ingest raw data from source systems into the Bronze layer.

**Process:**
1. **Read CSV files** from Unity Catalog Volumes:
   - `source_crm/cust_info.csv` → Customer information
   - `source_crm/prd_info.csv` → Product information

2. **Schema Inference:**
   - Use `inferSchema=True` to automatically detect data types
   - Preserve headers with `header=True`

3. **Write to Bronze Tables:**
   - `workspace.bronze.crm_cust_info` (Customer data)
   - `workspace.bronze.crm_prd_info` (Product data)
   - Use Delta format with `overwrite` mode

**Key Code:**
```python
df = (
    spark.read.option("header", True)
    .option("inferSchema", True)
    .csv("/Volumes/workspace/bronze/source_systems/source_crm/cust_info.csv")
)
df.write.mode("overwrite").saveAsTable("workspace.bronze.crm_cust_info")
```

---

### **Step 2: Silver Layer - Data Cleaning & Standardization**
**Notebook:** `Silver_crm_cust_info.ipynb`

**Purpose:** Clean, standardize, and transform raw data for consistency and quality.

**Process:**

1. **Read from Bronze:**
   ```python
   df = spark.table("workspace.bronze.crm_cust_info")
   ```

2. **Data Cleaning - Trimming Whitespace:**
   - Loop through all string columns
   - Apply `trim()` function to remove leading/trailing spaces
   ```python
   for field in df.schema.fields:
       if isinstance(field.dataType, StringType):
           df = df.withColumn(field.name, trim(col(field.name)))
   ```

3. **Data Normalization:**
   - Standardize marital status: `S` → `Single`, `M` → `Married`
   - Standardize gender: `F` → `Female`, `M` → `Male`
   ```python
   df = df.withColumn(
       "cst_marital_status",
       F.when(F.upper(F.col("cst_marital_status")) == "S", "Single")
        .when(F.upper(F.col("cst_marital_status")) == "M", "Married")
        .otherwise("n/a")
   )
   ```

4. **Column Renaming (Business Naming Conventions):**
   - `cst_id` → `customer_id`
   - `cst_key` → `customer_key`
   - `cst_firstname` → `first_name`
   - `cst_lastname` → `last_name`
   - `cst_marital_status` → `marital_status`
   - `cst_gndr` → `gender`
   - `cst_create_date` → `created_date`

5. **Write to Silver Table:**
   ```python
   df.write.format("delta").mode("overwrite").saveAsTable("workspace.silver.cust_customers")
   ```

---

### **Step 3: Gold Layer - Dimensional Modeling**
**Notebook:** `Gold_dim_customers.ipynb`

**Purpose:** Create analytics-ready dimensional tables for BI and reporting.

**Process:**

1. **Business Transformation:**
   - Create dimension table with surrogate keys
   - Apply window functions for row numbering
   - Select only necessary columns for analytics

2. **Create Customer Dimension:**
   ```sql
   SELECT 
       ROW_NUMBER() OVER (ORDER BY ci.customer_id) AS customer_key,
       ci.customer_id AS customer_id,
       ci.first_name,
       ci.last_name
   FROM workspace.silver.cust_customers AS ci
   ```

3. **Write to Gold Table:**
   ```python
   df.write.mode("overwrite").format("delta").saveAsTable("workspace.gold.dim_customers")
   ```

**Output:** Analytics-ready `dim_customers` table ready for:
- BI dashboards
- Reporting
- Data analysis
- Machine learning

---

## 📁 Project Structure

```
databricks_bootcamp/
├── README.md
└── bike_lakehouse_2026/
    ├── Bronze.ipynb                 # Raw data ingestion
    ├── Silver_crm_cust_info.ipynb  # Data cleaning & standardization
    └── Gold_dim_customers.ipynb    # Dimensional modeling
```

---

## 🔄 Execution Order

To run this project end-to-end:

1. **Bronze Layer:** Run `Bronze.ipynb`
   - Ingests raw CSV files into Bronze tables

2. **Silver Layer:** Run `Silver_crm_cust_info.ipynb`
   - Reads from `workspace.bronze.crm_cust_info`
   - Writes to `workspace.silver.cust_customers`

3. **Gold Layer:** Run `Gold_dim_customers.ipynb`
   - Reads from `workspace.silver.cust_customers`
   - Writes to `workspace.gold.dim_customers`

**Note:** Each layer depends on the previous layer's output.

---

## 🗂️ Unity Catalog Structure

```
workspace (catalog)
├── bronze (schema)
│   ├── crm_cust_info (table)
│   └── crm_prd_info (table)
├── silver (schema)
│   └── cust_customers (table)
└── gold (schema)
    └── dim_customers (table)
```

---

## 🛠️ Technologies Used

- **Databricks** - Unified data analytics platform
- **Apache Spark** - Distributed computing engine
- **PySpark** - Python API for Spark
- **Spark SQL** - SQL interface for Spark
- **Delta Lake** - ACID transactions and time travel
- **Unity Catalog** - Unified governance solution

---

## 📚 Key Concepts Demonstrated

* ✅ **Medallion Architecture** (Bronze → Silver → Gold)
* ✅ **Delta Lake** for ACID transactions
* ✅ **Unity Catalog** for data governance
* ✅ **Schema Inference** from CSV files
* ✅ **Data Quality** transformations (trimming, normalization)
* ✅ **Dimensional Modeling** (surrogate keys, window functions)
* ✅ **PySpark DataFrames** API
* ✅ **Spark SQL** for business transformations
---

## 🛡️ License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and share this project with proper attribution.
