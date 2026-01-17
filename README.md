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


#### Implementation Approach
## Phase 1: Data Profiling & Validation
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
<br>

2. Loaded CSV files into a dedicated staging schema and verify upload
- Verify upload <br>
	- Staging campaigns table <br>
<img height="250" alt="staging campaigns" src="https://github.com/user-attachments/assets/636be919-9c12-4d19-90e6-12442c553dcc" /> <br>

		```SQL
		SELECT * FROM staging.campaigns;
		```

	<br>
	
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

	- Staging sales table <br>
<img height="250" alt="staging sales" src="https://github.com/user-attachments/assets/c7526db0-20ec-4c6a-a79d-80693d91d873" /> <br>

	
		```sql
		SELECT * FROM staging.sales;
		```

	<br>

	- Staging salesitems table <br>
<img height="250" alt="staging salesitems" src="https://github.com/user-attachments/assets/b21ae2f4-7a48-405e-8166-4b1bc2a6aff2" /> <br>

		```sql
		SELECT * FROM staging.salesitems;
		```
	<br>

	- Staging stock table <br>
<img height="250" alt="staging stock" src="https://github.com/user-attachments/assets/e6ab34ff-c58b-40e7-88f8-f720d2ecd1d8" /> <br>

		```sql	
		SELECT * FROM staging.stock;
		```

<Br>
<br>

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
<br>

## Phase 2: Logical Data Modeling

### Modeling Philosophy
This phase emphasizes that a production database is more than stored data. <br>
The database was designed around clear structure, enforced rules, and explicit relationships. <br>

**Database = Structure + Rules + Relationships** 

<br>


### Core Schema Data Modeling Principles
- Normalized to Third Normal Form (3NF)
- Textual attributes centralized to eliminate duplication
- No calculated or aggregated values stored
- Referential integrity enforced using foreign keys
- Business rules enforced at the database level


<br>


### Entity Relationship Diagram (ERD)
An Entity Relationship Diagram (ERD) was created to validate and communicate the logical model. <br>
The following entities form the foundation of the core schema : *customers, country, channels, products, sales, salesitems, campaigns, stock*. <br>

**Diagram** <br>
<img height="400" alt="er diagram" src="https://github.com/user-attachments/assets/68c5d42f-baef-41d9-a505-877eb0209fed" />


<br>

### ETL Implementation (SQL-Based)
- Extract: Raw CSV data was ingested into staging tables to preserve source fidelity and enable validation.
- Transform: Data was cleansed and reshaped during SQL-based inserts, including data type casting, deduplication, and mapping of textual attributes to surrogate keys.
- Load: Transformed data was loaded into a normalized core schema with enforced foreign key relationships.
- Validate: Post-load verification ensured record-level consistency between staging and core schemas.

<br>

### Core Schema Implementation nd Data Loading

- Create core shema
```sql
CREATE SCHEMA core;
```

<br>

- Create Core Tables

```sql
-- 1. countries (reference table)
CREATE TABLE core.countries (
    country_id SERIAL PRIMARY KEY,
    country_name TEXT NOT NULL UNIQUE
); 


--2. channels (reference table)
CREATE TABLE core.channels (
    channel_id SERIAL PRIMARY KEY,
    channel_name TEXT NOT NULL UNIQUE,
    description TEXT
);


--3. customers
CREATE TABLE core.customers (
    customer_id INTEGER PRIMARY KEY,
    country_id INTEGER NOT NULL,
    age_range TEXT,
    signup_date DATE NOT NULL,

    CONSTRAINT fk_customer_country
        FOREIGN KEY (country_id)
        REFERENCES core.country (country_id)
);


-- 4. products
CREATE TABLE core.products (
    product_id INTEGER PRIMARY KEY,
    product_name TEXT NOT NULL,
    category TEXT,
    brand TEXT,
    color TEXT,
    size TEXT,
    catalog_price NUMERIC(10,2),
    cost_price NUMERIC(10,2),
    gender TEXT
);


-- 5. sales 
CREATE TABLE core.sales (
    sale_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    channel_id INTEGER NOT NULL,
    country_id INTEGER NOT NULL,
    sale_date DATE NOT NULL,
    total_amount NUMERIC(10,2),
    discounted BOOLEAN,

    CONSTRAINT fk_sales_customer
        FOREIGN KEY (customer_id)
        REFERENCES core.customers (customer_id),

    CONSTRAINT fk_sales_channel
        FOREIGN KEY (channel_id)
        REFERENCES core.channels (channel_id),

    CONSTRAINT fk_sales_country
        FOREIGN KEY (country_id)
        REFERENCES core.countries (country_id)
);


-- 6. salesitems
CREATE TABLE core.salesitems (
    item_id INTEGER PRIMARY KEY,
    sale_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    original_price NUMERIC(10,2),
    unit_price NUMERIC(10,2),
    discount_applied NUMERIC(10,2),

    CONSTRAINT fk_item_sale
        FOREIGN KEY (sale_id)
        REFERENCES core.sales (sale_id),

    CONSTRAINT fk_item_product
        FOREIGN KEY (product_id)
        REFERENCES core.products (product_id)
);


-- 7. stock
CREATE TABLE core.stock(
	country_id INT NOT NULL,
	product_id INT NOT NULL,
	stock_quantity INT,

PRIMARY KEY (country_id, product_id),

CONSTRAINT fk_stock_country
	FOREIGN KEY (country_id)
	REFERENCES core.country (country_id),

CONSTRAINT fk_stock_products
	FOREIGN KEY (product_id)
	REFERENCES core.products (product_id)
);


-- 8. campaigns
CREATE TABLE core.campaigns (
	campaign_id INT PRIMARY KEY,
	campaign_name TEXT,
	start_date DATE NOT NULL,
	end_date DATE NOT NULL,
	channel_id INT NOT NULL,
	discount_type TEXT,
	discount_value TEXT,

CONSTRAINT fk_campaigns_channel
	FOREIGN KEY (channel_id)
	REFERENCES core.channels (channel_id),

CONSTRAINT chk_campaign_dates
	CHECK (start_date <= end_date)
);
```

<br>
<br>

- Load Data Into Core Tables
```SQL
-- 1. load country
INSERT INTO core.country (country_name)
SELECT DISTINCT country
FROM staging.customers
WHERE country IS NOT NULL;


-- 2. load channel
INSERT INTO core.channels (channel_name, description)
SELECT DISTINCT channel, description
FROM staging.channels;


-- 3. load customers
INSERT INTO core.customers (customer_id, country_id, age_range, signup_date)
SELECT 
	c.customer_id, co.country_id, c.age_range, c.signup_date::DATE
FROM staging.customers c
JOIN core.country co
ON c.country = co.country_name;


-- 4. Load products
INSERT INTO core.products
SELECT * 
FROM staging.products;


-- 5. Load Sales
INSERT INTO core.sales 
	(sale_id, customer_id, channel_id, country_id, sale_date, total_amount, discounted)
SELECT 
	s.sale_id, 
	s.customer_id, 
	ch.channel_id, 
	co.country_id,
	s.sale_date::DATE,
	s.total_amount, 
	(s.discounted = 1) AS discounted
FROM staging.sales s
JOIN core.channels ch
ON s.channel = ch.channel_name
JOIN core.country co
ON s.country = co.country_name;


-- 6. core.salesitems
INSERT INTO core.salesitems 
	(item_id, sale_id, product_id, quantity, original_price, unit_price, discount_applied)
SELECT
	item_id, 
	sale_id, 
	product_id,
	quantity,
	original_price,
	unit_price,
	discount_applied
FROM staging.salesitems;


-- 7. Load campaigns
INSERT INTO core.campaigns
	(campaign_id, campaign_name, start_date, end_date, channel_id, discount_type, discount_value)
SELECT 
	ca.campaign_id, 
	ca.campaign_name, 
	ca.start_date::DATE, 
	ca.end_date::DATE, 
	ch.channel_id, 
	ca.discount_type, 
	ca.discount_value
FROM staging.campaigns ca
JOIN core.channels ch
ON ca.channel = ch.channel_name;



-- 8. Load stock
INSERT INTO core.stock (country_id, product_id, stock_quantity)
SELECT 
	c.country_id, 
	s.product_id, 
	s.stock_quantity
FROM staging.stock s
JOIN core.country c
ON s.country = c.country_name;
```

<BR>

- Verification

-- 1. Customers table
SELECT COUNT(*) FROM staging.customers;
SELECT COUNT(*) FROM core.customers;
output: 1000 in both schemas

-- 2. Sales table
SELECT COUNT(*) FROM staging.sales;
SELECT COUNT(*) FROM core.sales;
output: 905 in both schemas

-- 3. Salesitems
SELECT COUNT(*) FROM staging.salesitems;
SELECT COUNT(*) FROM core.salesitems;
output: 2253 in both schemas
