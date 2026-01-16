# Enterprise Sales Analytics and Data Modeling Using PostgreSQL

<br>

### Project Overview 

This project demonstrates an end-to-end SQL development and analytics workflow using PostgreSQL. Raw CSV datasets were ingested into a staging schema, validated using SQL-based data profiling techniques, transformed into a normalized core relational model, and exposed through analytical views to support business intelligence and decision-making.

<br>

The project applies enterprise SQL best practices including schema design, data modeling, SQL-based ETL transformations, constraint enforcement, indexing strategies, and analytical querying.


<br>
<br>

### Business Objectives

- Build a production-ready relational database from raw CSV files
- Enforce data integrity and business rules
- Enable scalable analytics and dashboarding
- Answer key sales, customer, inventory, and discount performance questions

<br>
<br>

### Data Flow Architecture

CSV Files  
⬇️  
**Staging Schema** *(Raw Ingestion)*  
⬇️  
**Core Schema** *(Normalized Relational Model)*  
⬇️  
**Analytics Schema** *(Business Views)*  
⬇️  
**Business Queries & Insights** <br>

<br>
<br>

### Schemas Description
#### 1. Staging Schema
- Raw ingestion of CSV files
- No transformations
- Used for data profiling and validation


#### 2. Core Schema
- Fully normalized relational model
- Primary & foreign keys enforced
- Business rules applied via constraints
- Tables include:
  * customers
  * sales
  * salesitems
  * products
  * stock
  * campaigns
  * channels
  * country

#### 3. Analytics Schema
- Read-only analytical views
- Designed to solve business questions and reporting
- Simplifies complex joins

<br>
<br>
<br>

## Implementation Approach


### Phase 1: Data Profiling & Validation
**Objective:** <br>
Ensure source data quality before enforcing business rules.

<br>

**Key Activities:**
1. Create staging schema and staging tables
- Create staging schema
```sql
CREATE SCHEMA staging;
```

<BR>

- Create staging tables
  ```sql
  -- Create staging tables
  
  -- i. staging.customers
    CREATE TABLE staging.customers (
    	customer_id INTEGER,
    	country TEXT,
    	age_range TEXT,
    	signup_date TEXT
    );


  -- ii. staging.campaigns
  CREATE TABLE staging.campaigns(
  	campaign_id INT,
  	campaign_name TEXT,
  	start_date TEXT,
  	end_date TEXT,
  	channel TEXT,
  	discount_type TEXT,
  	discount_value TEXT
  );


  -- iii. staging.channels
  CREATE TABLE staging.channels (
  	channel TEXT,
  	description TEXT
  );


  -- iv. staging.products
  CREATE TABLE staging.products (
  	product_id INT,
  	product_name TEXT,
  	category TEXT,
  	brand TEXT,
  	color TEXT,
  	size TEXT,
  	catalog_price NUMERIC(10,2),
  	cost_price	NUMERIC(10,2),
  	gender TEXT
  );
  

  -- v. staging.sales
  CREATE TABLE staging.sales(
  	sale_id	INT,
  	channel TEXT,
  	discounted INT,	
  	total_amount NUMERIC(10,2),
  	sale_date TEXT,
  	customer_id	INT, 
  	country TEXT
  );
  

  -- vi. staging.salesitems
  CREATE TABLE staging.salesitems(
  	item_id	INT,
  	sale_id	INT,
  	product_id INT,
  	quantity INT,
  	original_price NUMERIC(10,2),	
  	unit_price NUMERIC(10,2),	
  	discount_applied NUMERIC(10,2),	
  	discount_percent TEXT,	
  	discounted INT,
  	item_total NUMERIC(10,2),
  	sale_date TEXT,
  	channel TEXT,
  	channel_campaigns TEXT
  );


  -- vi. staging.stock
  CREATE TABLE staging.stock(
  	country TEXT,
  	product_id INT,
  	stock_quantity INT
  );
  ```

<br>

2. Loaded CSV files into a dedicated staging schema and verify upload
```SQL
-- Verify upload
SELECT * FROM staging.campaigns;
SELECT * FROM staging.channels;
SELECT * FROM staging.customers;
SELECT * FROM staging.products;
SELECT * FROM staging.sales;
SELECT * FROM staging.salesitems;
SELECT * FROM staging.stock;
```

<Br>

3. Performed data profiling to identify:
- Duplicate primary keys

```sql
-- Check for duplicate primary keys
  
-- i. For staging.campaigns
SELECT 
  campaign_id,
  COUNT(*) AS id_count
FROM staging.campaigns
GROUP BY campaign_id 
HAVING COUNT(*)> 1;


-- ii. For staging.customers
SELECT 
  customer_id, count(*) as id_count
FROM staging.customers
GROUP BY customer_id
HAVING COUNT(*)>1;


-- iii. For staging.products
SELECT
  product_id,
  COUNT(*) AS id_count
FROM staging.products
GROUP BY product_id
HAVING COUNT(*) > 1;


-- iv. For staging.sales
SELECT 
  sale_id,
  COUNT(*) AS id_count 
FROM staging.sales
GROUP BY sale_id
HAVING COUNT(*) > 1;


-- v. For staging.salesitems
SELECT 
  item_id,
  COUNT(*)
FROM staging.salesitems
GROUP BY item_id
HAVING COUNT(*) > 1;
```
  
- Orphan foreign keys

```sql
-- Check for Orphan Foreign Keys

-- i. sales → customers
SELECT DISTINCT
s.customer_id 
FROM staging.sales s
LEFT JOIN staging.customers c
ON s.customer_id = c.customer_id
WHERE c.customer_id IS NULL;


-- ii. salesitems.sale_id → sales.sale_id
SELECT DISTINCT i.sale_id
FROM staging.salesitems i
LEFT JOIN staging.sales s
ON i.sale_id = s.sale_id
WHERE s.sale_id IS NULL;


-- iii. salesitems.product_id → products.product_id
SELECT DISTINCT i.product_id
FROM staging.salesitems i
LEFT JOIN staging.products p
ON i.product_id = p.product_id
WHERE p.product_id IS NULL;


-- iv stock.product_id → products.product_id
SELECT DISTINCT s.product_id
FROM staging.stock s
LEFT JOIN staging.products p
ON s.product_id = p.product_id
WHERE p.product_id IS NULL;
```

    
- Invalid numeric values

```SQL
-- Negative or zero prices

-- i. in staging.products
SELECT * FROM staging.products
WHERE cost_price <= 0
	OR catalog_price <= 0;
	

-- ii. in staging.sales
SELECT * FROM staging.sales
WHERE total_amount <= 0;


-- iii. in staging.salesitems
SELECT * FROM staging.salesitems
WHERE quantity <= 0 OR 
	original_price <= 0 OR
	unit_price <= 0	OR
	discount_applied < 0 OR
	discounted < 0 OR
	item_total <= 0;


-- iv. in staging.stock
SELECT * FROM staging.stock
WHERE stock_quantity < 0;
```

- Date inconsistencies
```sql
-- Date Validity Check
-- A. NULL date checks

-- i.
SELECT * FROM staging.campaigns
WHERE start_date IS NULL OR end_date IS NULL;


-- ii.
SELECT * FROM staging.customers
WHERE signup_date IS NULL;


-- iii.
SELECT * FROM staging.sales
WHERE sale_date IS NULL;


-- iv
SELECT * FROM staging.salesitems
WHERE sale_date IS NULL;


-- B. Logical order checks (Confirmed campaign start-dates occur before end-dates and no sales were recorded before customer signup dates)
SELECT * FROM staging.campaigns
WHERE start_date > end_date;


-- C. Cross-table date logic (Sales should not occur before customer signup)
SELECT s.* 
FROM staging.sales s
JOIN staging.customers c
ON c.customer_id = s.customer_id
WHERE s.sale_date < c.signup_date;
```

<br>

**Key Data Profiling Findings** <br>
- Primary Keys: No duplicate primary keys detected across all staging tables.
- Referential Integrity: No orphan foreign keys found; all relationships were valid.
- Numeric Validation: All price, quantity, revenue, and stock values were within valid ranges.
- Date Integrity: No missing dates or invalid intra-table date sequences identified.
- Cross-Table Date Issue: 73 sales records occurred before customer signup dates, indicating a potential customer lifecycle inconsistency (e.g., post-purchase registration or source data errors).

<br>

**Resolution:** <br>
No staging-level corrections were applied. The issue was documented and mitigated through enforced constraints and modeling decisions in the core schema.
