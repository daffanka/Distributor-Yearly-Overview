# Distributor Yearly Overview

## Introduction
This project demonstrates an end-to-end data workflow starting from a Data Warehouse and culminating in Power BI visualizations. It focuses on distributor performance for a given year, showcasing sales insights derived from Sell-In and Sell-Through datasets. The project aims to provide a comprehensive understanding of distributor contributions to overall performance.

---

## Workflow
1. **Data Extraction**: SQL queries extract relevant Sell-In and Sell-Through data from the Data Warehouse.
2. **Data Transformation**: Data is preprocessed to ensure compatibility with Power BI.
3. **Visualization**: Power BI is used to create dashboards for business insights.

---

## SQL Queries
### Sell-In Data
```sql
-- SELL IN DATA 2024 YTD
-- Sell In Data from 01-10-2023 until 31-07-2023
WITH sell_in_jan_jul AS (
    SELECT
        CAST(a.po_date AS DATE) AS calendar_date,
        b.distributor_company AS distributor_company,
        a.distributor AS distributor_name,
        b.region AS region,
        b.asm AS asm,
        c.brand AS brand,
        UPPER(a.product_code) AS product_id,
        c.product_name AS product_name,
        c.assortment AS assortment,
        a.quantity AS quantity,
        a.total_value AS value
    FROM `gt_schema.fact_gt_sell_in_oct_onward_v` AS a
    LEFT OUTER JOIN `gt_schema.master_distributor` AS b
        ON a.distributor = b.distributor
    LEFT OUTER JOIN `gt_schema.master_product` AS c
        ON UPPER(a.product_code) = UPPER(c.sku)
    WHERE a.po_date BETWEEN '2024-01-01' AND '2024-07-31'
),
-- Sell In Data from 01-08-2024 until 31-08-2024
sell_in_aug AS (
    SELECT
        CAST(a.calendar_date AS DATE) AS calendar_date,
        b.distributor_company AS distributor_company,
        a.distributor_name AS distributor_name,
        b.region AS region,
        b.asm AS asm,
        c.brand AS brand,
        UPPER(a.item_id) AS product_id,
        c.product_name AS product_name,
        c.assortment AS assortment,
        a.final_qty AS quantity,
        a.total_amount AS value
    FROM `gt_schema.fact_gt_sell_in_aug2024_v` AS a
    LEFT OUTER JOIN `gt_schema.master_distributor` AS b
        ON UPPER(a.distributor_name) = UPPER(b.distributor)
    LEFT OUTER JOIN `gt_schema.master_product` AS c
        ON UPPER(a.item_id) = UPPER(c.sku)
    WHERE a.calendar_date BETWEEN '2024-08-01' AND '2024-08-31'
),
-- Sell In Data from 01-09-2024 until 31-12-2024
sell_in_sep_dec AS (
    SELECT
        CAST(a.Date_Formatted AS DATE) AS calendar_date,
        b.distributor_company AS distributor_company,
        a.distributor_name AS distributor_name,
        b.region AS region,
        b.asm AS asm,
        c.brand AS brand,
        UPPER(a.product_id) AS product_id,
        c.product_name AS product_name,
        c.assortment AS assortment,
        a.qty_order AS quantity,
        a.nett_amount_incl_ppn AS value
    FROM `bosnet.po_tracking_all` AS a
    LEFT OUTER JOIN `gt_schema.master_distributor` AS b
        ON a.distributor_id = b.distributor_code
    LEFT OUTER JOIN `gt_schema.master_product` AS c
        ON UPPER(a.product_id) = UPPER(c.sku)
    WHERE a.Date_Formatted BETWEEN '2024-09-01' AND '2024-12-31'
)
SELECT * FROM sell_in_jan_jul
UNION ALL
SELECT * FROM sell_in_aug
UNION ALL
SELECT * FROM sell_in_sep_dec;
```

### Sell-Through Data
```sql
-- SELL THROUGH DATA 2024 YTD
-- Sell Through Data from 01-01-2024 until 31-08-2024
WITH sell_through_jan_aug AS (
    SELECT
        CAST(a.calendar_date AS DATE) AS calendar_date,
        a.customer_id AS cust_id,
        b.store_name AS cust_name,
        b.distributor_se AS distributor_se,
        b.spv AS spv,
        d.distributor_company AS distributor_company,
        b.distributor AS distributor_name,
        b.region AS region,
        b.asm AS asm,
        c.brand AS brand,
        UPPER(a.product_code) AS product_id,
        c.product_name AS product_name,
        c.assortment AS assortment,
        a.quantity AS quantity,
        a.total_amount AS value
    FROM `accurate.fact_sales_po_accurate_v` AS a
    LEFT OUTER JOIN `gt_schema.dim_store_database_gt_t` AS b
        ON UPPER(a.customer_id) = UPPER(b.cust_id)
    LEFT OUTER JOIN `gt_schema.master_product` AS c
        ON UPPER(a.product_code) = UPPER(c.sku)
    LEFT OUTER JOIN `gt_schema.master_distributor` AS d
        ON UPPER(b.distributor) = UPPER(d.distributor)
    WHERE a.calendar_date BETWEEN '2024-01-01' AND '2024-08-31'
),
-- Sell Through Data from 01-09-2024 until 31-12-2024
sell_through_sep_dec AS (
    SELECT
        CAST(a.OrderDate AS DATE) AS calendar_date,
        a.correction AS cust_id,
        b.store_name AS cust_name,
        b.distributor_se AS distributor_se,
        b.spv AS spv,
        d.distributor_company AS distributor_company,
        b.distributor AS distributor_name,
        b.region AS region,
        b.asm AS asm,
        c.brand AS brand,
        UPPER(a.ProductID) AS product_id,
        c.product_name AS product_name,
        c.assortment AS assortment,
        a.Quantity AS quantity,
        a.NettAmountIncTax AS value
    FROM `bosnet.st_data_all` AS a
    LEFT OUTER JOIN `gt_schema.dim_store_database_gt_t` AS b
        ON UPPER(a.correction) = UPPER(b.cust_id)
    LEFT OUTER JOIN `gt_schema.master_product` AS c
        ON UPPER(a.ProductID) = UPPER(c.sku)
    LEFT OUTER JOIN `gt_schema.master_distributor` AS d
        ON UPPER(b.distributor) = UPPER(d.distributor)
    WHERE a.OrderDate BETWEEN '2024-09-01' AND '2024-12-31'
)
SELECT * FROM sell_through_jan_aug
UNION ALL
SELECT * FROM sell_through_sep_dec;
```

---

## Data Modelling
**Power BI Data Modelling**
![image](https://github.com/user-attachments/assets/96352205-6013-4f33-b344-6129b0fd4613)

- **Metrics**:
  - **Basket Size**: `Quantity Sales / Active Outlets (AO)`
  - **Contribution Analysis**: Sell-In and Sell-Through contributions by distributor branch.

- **Filters**:
  - Filter by distributor company.
  
- **Definitions**:
  - **AO**: Active Outlet — Outlets that have recorded transactions.
  - **BA**: Beauty Advisor.

---

## Power BI Visualizations
- Visuals include:
  - Sales trends by month and distributor.
  - Regional performance breakdowns.
  - Brand contribution analysis.
  - Store grade analysis contribution.
  - Comparison distributor company and  by branch performance to National performance.
  - Store with dedicated BA vs Non BA comparison on distributors to national wise.
  - SKU assortment performance breakdowns by distributors to national wise.
  - Distributors branch proportion to each distributors company.
 


---

## How to Reproduce
1. **Run the SQL queries** provided above to extract data from your Data Warehouse.
2. **Load data into Power BI** using the provided `.pbix` file.
3. Customize visualizations to explore insights.

