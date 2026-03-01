# Cloud Data Warehouse Migration: Snowflake & dbt

## Project Overview
This project demonstrates the migration of a traditional, on-premise retail data warehouse into a highly scalable Modern Data Stack. It transforms raw, disparate datasets (CRM and ERP data) into a business-ready Star Schema using the Medallion Architecture (Bronze, Silver, Gold). 

By utilizing **Snowflake** for cloud compute and **dbt (data build tool)** for transformation and testing, this pipeline treats data as code, ensuring high data quality, version control, and automated lineage.

## Architecture & Tech Stack
* **Data Lake / Storage:** Google Cloud Platform (GCP)
* **Cloud Data Warehouse:** Snowflake
* **Data Transformation & Orchestration:** dbt (Data Build Tool)
* **Version Control:** Git & GitHub



## Data Modeling Strategy: The Medallion Architecture
The project strictly adheres to the Medallion architecture to ensure logical separation of concerns:

### 1. Raw Layer (Bronze)
* Data is loaded from GCP into Snowflake via External Stages.
* Represents the exact state of the source systems (CRM sales/customer data and ERP location/product data).

### 2. Staging Layer (Silver)
* **Objective:** Cleanse, standardize, and prepare data for modeling.
* **Transformations applied via dbt:**
  * Standardized data types (e.g., string to date conversions).
  * Extracted and mapped synthetic category IDs from legacy product keys.
  * Stripped legacy prefixes/suffixes to align primary and foreign keys across systems.
  * Added `dwh_create_date` audit timestamps using `CURRENT_TIMESTAMP()`.
* **Materialization:** Deployed as Snowflake Views.

### 3. Marts Layer (Gold)
* **Objective:** Deliver a Kimball-style Star Schema optimized for BI tools (e.g., Power BI, Tableau).
* **Transformations applied via dbt:**
  * Built `dim_customers` and `dim_products` by joining CRM and ERP data, handling missing legacy attributes via `COALESCE`.
  * Preserved historical product records to maintain data integrity in the fact table.
  * Built `fact_sales` by joining cleansed transaction data to surrogate keys.
  * Applied business-friendly aliasing for final presentation.
* **Materialization:** Deployed as Snowflake Tables for high query performance.

## Automated Data Quality & Testing
A critical component of this pipeline is automated Quality Assurance (QA). Instead of writing manual validation scripts, data integrity is enforced via dbt YAML configurations.

* **Primary Key Integrity:** `unique` and `not_null` tests applied to all surrogate keys in the dimension tables.
* **Referential Integrity:** `not_null` tests applied to all foreign keys in the fact table to ensure no orphan records exist.
* **Source Alignment:** Staging models are rigorously tested to catch anomalies before they enter the Gold layer.

## Repository Structure
```text
├── models/
│   ├── staging/          # Silver layer transformations and stg_models.yml tests
│   ├── marts/            # Gold layer Star Schema models and marts_models.yml tests
│   └── sources.yml       # Snowflake raw database mapping
├── dbt_project.yml       # Core dbt configuration file
└── README.md