## Phase 1: Data Profiling & Validation
**Objective:** <br>
Ensure source data quality before enforcing business rules.

<br>

**Key Activities:**

#### 1. Create staging schema and staging tables
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
<br>

#### 2. Loaded CSV files into a dedicated staging schema using PostgreSQL’s native import functionality and verify upload
- Verify upload <br>

	- Staging channel table <br>
<img height="250" alt="staging channels" src="https://github.com/user-attachments/assets/cb453baa-0519-4218-87af-11909bfaeee7" /> <br>

		```sql
		SELECT * FROM staging.channels;
		```

	<br>
	
	- Staging customers table <br>
<img height="250" alt="staging customers" src="https://github.com/user-attachments/assets/8bd10481-deea-47a3-a0e5-0f2e1b27d86e" /> <br>

		```sql
		SELECT * FROM staging.customers;
		```

	<br>

	- Staging products table <br>
<img height="250" alt="staging products" src="https://github.com/user-attachments/assets/44e9f079-b535-4d87-accb-1f59a2ecc097" /> <br>


		```sql
		SELECT * FROM staging.products;
		```

	<br>

> **All staging tables were verified for successful ingestion.**


<Br>
<br>

#### 3. Performed data profiling to identify:
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

<br>

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

<br>

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

<br>

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


-- B. Logical order checks
-- (Confirmed campaign start-dates occur before end-dates and no sales were recorded before customer signup dates)
SELECT * FROM staging.campaigns
WHERE start_date > end_date;
```

<br>

- Cross-table date logic (Sales should not occur before customer signup) <br>
<img height="250" alt="cross-table date logic" src="https://github.com/user-attachments/assets/50c22573-cf9a-4772-a1f8-5ac35b851112" />

```sql
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
- Cross-Table Date Issue: 73 sales records occurred before customer signup dates, indicating a potential customer lifecycle inconsistency.

<br>

**Resolution:** <br>
No staging-level corrections were applied. The issue was documented and addressed through enforced constraints and modeling decisions in the core schema.


<br>
<br>