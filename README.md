# FMCG-Data-Lakehouse-Post-Acquisition-Sales-Analytics-Automated-Pipeline-
An end-to-end data engineering project on Databricks that consolidates sales data from an acquired company into its parent company's analytics model, using a medallion (Bronze, Silver, Gold) architecture, Delta Lake and incremental loading.

Business Problem

Sports-equipment company, the parent acquired SportsBar (a nutrition-bar company, the child). Each company had its own data model and data quality issues. Leadership needed one unified view of sales, revenue, products and customers.

The task was to:

Built an end-to-end ETL pipeline using Medallion architecture (Bronze/Silver/Gold) on Databricks, PySpark and Delta Lake, ingesting 62K+ order records from 180+ daily files in AWS S3 and merging an acquired company’s data into the parent’s unified star-schema Gold layer.
Automated the pipeline with a multi-task Databricks Workflow (Lakeflow Job) on serverless compute, chaining customer, product and pricing dimension tasks before the incremental fact load, so dependencies run in the correct order with no manual notebook execution.
Engineered incremental loads using Delta Lake MERGE (upserts) and staging tables for idempotent, duplicate-free processing. Standardized inconsistent data (multiple date formats, invalid prices, typos, orphan IDs) and reconciled daily child data with the parent’s monthly grain.

## Architecture
<img width="20005" height="11129" alt="project_architecture" src="https://github.com/user-attachments/assets/97ee6af2-4017-411c-85cd-49e260d29725" />

## Tech Stacks
1.Databricks, 
2.PySpark,
3.Delta Lake,
4.SQL
5.AWS S3
6.Unity Catalog

## Key Features
-Bronze, silver, and gold processing layers for customers, products, pricing, and orders
-Full-load and incremental fact ingestion with staging tables and merge logic
-Data cleaning and standardization for customer names, city values, product categories, prices, and dates
-Delta Lake tables with change data feed support for reliable batch pipelines
-Parent-child data consolidation so child-order data is rolled up into monthly gold facts
-SQL-based analytics view for enriched sales reporting over date, customer, product, and price dimensions

## Key Features
### Data cleaning (Silver layer)
Removed duplicate records on business keys.
Trimmed whitespace and fixed title casing in names and categories.
Corrected city typos (e.g. Bengalore → Bengaluru) and spelling errors (Protien → Protein).
Filled missing cities using business-confirmed corrections.
Parsed multiple inconsistent date formats into a proper date type, and stripped weekday text ("Tuesday, July 01, 2025").
Validated prices: converted numeric strings, fixed negative values and replaced non-numeric values.
Mapped invalid customer IDs to a default key (999999) so fact records are not lost in joins.
### Standardization to the parent model
Derived division, category, product and variant for child products (variant extracted from the product name with regex).
Generated product_code to match the parent's key.
Built the customer field as CustomerName-City and set market, platform and channel to match the parent's schema.
### Pipelines
Full load: processes all historical files and builds the child Gold table.
Incremental load:
Reads only newly arrived files, using a staging table.
Upserts with Delta Lake MERGE, so re-runs are idempotent.
Identifies the affected months, recalculates monthly totals and merges them into the parent fact_orders.
Drops the staging tables afterwards.
File archiving: processed files move from landing/ to processed/.
Grain alignment: child data is daily and parent data is monthly, so child data is aggregated to month level before merging.
### Gold layer and serving
Star schema: fact_orders with dim_customers, dim_products, dim_gross_price and dim_date.
dim_date is generated programmatically at month grain (2024-01 to 2025-12).
vw_fact_orders_enriched is a denormalized view with a derived metric (total_amount_inr = sold_quantity × price_inr).
A Databricks SQL dashboard shows total revenue, quantity sold, unique customers, average selling price, revenue by channel, top products and customers, monthly trend and top variants. Genie is set up for natural-language queries.

### Orchestration
- Automated with a **Databricks Workflow (Lakeflow Job)** named *FMCG Incremental Update*, running on serverless compute.
- Four tasks run in sequence: `dim_processing_customers` → `dim_processing_products` → `dim_processing_prices` → `fact_processing_orders`.
- Dimensions run before the fact load because the fact join depends on `product_code`.
- Notebooks are parameterized with widgets (`catalog`, `data_source`).
