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
The following entities form the foundation of the core schema : *customers, country, channels, products, sales, salesitems, campaigns, stock*. 

<br>

**Diagram** <br>
<img height="400" alt="er diagram" src="https://github.com/user-attachments/assets/68c5d42f-baef-41d9-a505-877eb0209fed" />


<br>

### ETL Implementation (SQL-Based)
- Extract: Raw CSV data was ingested into staging tables to preserve source fidelity and enable validation.
- Transform: Data was cleansed and reshaped during SQL-based inserts, including data type casting, deduplication, and mapping of textual attributes to surrogate keys.
- Load: Transformed data was loaded into a normalized core schema with enforced foreign key relationships.
- Validate: Post-load verification ensured record-level consistency between staging and core schemas.

<br>

### Core Schema Implementation and Data Loading

#### 1. Create core shema
```sql
CREATE SCHEMA core;
```

<br>

#### 2. Create Core Tables

```sql
-- i. countries (reference table)
CREATE TABLE core.countries (
    country_id SERIAL PRIMARY KEY,
    country_name TEXT NOT NULL UNIQUE
); 


-- ii. channels (reference table)
CREATE TABLE core.channels (
    channel_id SERIAL PRIMARY KEY,
    channel_name TEXT NOT NULL UNIQUE,
    description TEXT
);


--iii. customers
CREATE TABLE core.customers (
    customer_id INTEGER PRIMARY KEY,
    country_id INTEGER NOT NULL,
    age_range TEXT,
    signup_date DATE NOT NULL,

    CONSTRAINT fk_customer_country
        FOREIGN KEY (country_id)
        REFERENCES core.country (country_id)
);


-- iv. products
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


-- v. sales 
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


-- vi. salesitems
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


-- vii. stock
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


-- viii. campaigns
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

#### 3. Load Data Into Core Tables:
- Load country
<img height="250" alt="core country" src="https://github.com/user-attachments/assets/9a9a66cf-40c5-4a0f-9aa1-4a3d19f0cc40" />

```SQL
INSERT INTO core.country (country_name)
SELECT DISTINCT country
FROM staging.customers
WHERE country IS NOT NULL;
```

<br>

- Load channel
<img height="250" alt="core channels" src="https://github.com/user-attachments/assets/52b0e5c8-c932-4156-bada-f60e49d02785" />

```sql
INSERT INTO core.channels (channel_name, description)
SELECT DISTINCT channel, description
FROM staging.channels;
```

<br>

- Load customers
<img height="250" alt="core customers" src="https://github.com/user-attachments/assets/d1e0462d-b810-455f-9349-9fa2be75c3ad" />

```sql
INSERT INTO core.customers (customer_id, country_id, age_range, signup_date)
SELECT 
	c.customer_id, co.country_id, c.age_range, c.signup_date::DATE
FROM staging.customers c
JOIN core.country co
ON c.country = co.country_name;
```

<br>

- Load products
<img height="250" alt="core products" src="https://github.com/user-attachments/assets/857c1fb0-f1c9-426b-a077-921962214159" />

```sql
INSERT INTO core.products
SELECT * 
FROM staging.products;
```

<br>

- Load Sales
<img height="250" alt="core sales" src="https://github.com/user-attachments/assets/284732d4-edc5-4384-92b6-fd49bb233a20" />

```sql
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
```

<br>

- Load salesitems
<img height="250" alt="core salesitems" src="https://github.com/user-attachments/assets/9185864d-36fd-43dd-badf-5e7814176377" />

```sql
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
```

<br>

- Load campaigns
<img height="250" alt="core campaigns" src="https://github.com/user-attachments/assets/31d3f7b2-a2a4-4ead-99ad-7f641ac8d86b" />

```sql
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
```

<br>

- Load stock
<img height="250" alt="core stock" src="https://github.com/user-attachments/assets/3806e397-5bfd-44ad-b754-abeb489884a9" />

```sql
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

#### 4. Verification

Post-load validation confirmed full row-level consistency between the staging and core schemas for all primary transactional and master tables. Record counts for customers, sales, and sales items matched exactly, indicating successful data transfer with no unintended data loss or duplication during the ETL process.

<br>
<br>
<br>

## Phase 3: Indexing Strategy & Performance Validation

#### Indexing Design Principles
- All foreign keys indexed
- High-usage date columns indexed
- Avoided indexing low-cardinality and small tables

<br>

#### Index Implementation

**1. Index Foreign Keys on Transaction Tables**
-  sales table
```sql
CREATE INDEX idx_sales_customer_id
ON core.sales (customer_id);

CREATE INDEX idx_sales_channel_id
ON core.sales (channel_id);

CREATE INDEX idx_sales_country_id
ON core.sales (country_id);
```

<br>

- salesitems table
```sql
CREATE INDEX idx_salesitems_sale_id
ON core.salesitems (sale_id);

CREATE INDEX idx_salesitems_product_id
ON core.salesitems (product_id);
```

<br>

- stock table
```sql
CREATE INDEX idx_stock_product_id
ON core.stock (product_id);
```

<br>
<br>

**2. Date Column Indexing** <br>
sales.sale_date <br>
```sql
CREATE INDEX idx_sales_sale_date
ON core.sales (sale_date);
```

<br>
<br>

**3. Index Usage Validation** <br>
Confirm that PostgreSQL is actually using the created index 

```sql
EXPLAIN ANALYZE
SELECT *
FROM core.sales
WHERE sale_date = '2025-04-01';
```

<img height="250" alt="validate index" src="https://github.com/user-attachments/assets/d8fa66b8-5542-44e8-a7b5-c8a757f4ab65" />
> PostgreSQL correctly selected sequential scans for low-selectivity queries on small tables, and index scans for highly selective predicates, demonstrating proper cost-based optimization behavior.

<br>
<br>
<br>

## Phase 4: Analytical Views (Analytics Schema)
Reusable business views were created to simplify reporting

<br>

#### 1. Create Analytics Schema
```sql
CREATE SCHEMA analytics;
```

<br>

#### 2. Create Foundational Analytical Views <br>
These views act as pre-aggregated, business-friendly abstractions over the core schema. <br>

- How much revenue do we make per day?
<img height="250" alt="1  How much revenue do we make per day" src="https://github.com/user-attachments/assets/efc50662-78d0-4a66-81fc-fe5eada4641c" />

```sql
CREATE VIEW analytics.revenue_per_day AS
SELECT 
	s.sale_date, 
	TRIM(TO_CHAR(s.sale_date, 'Day')) AS day_name,
	COUNT(s.sale_id) AS total_order,
	SUM(s.total_amount) as total_revenue
FROM core.sales s
GROUP BY s.sale_date
ORDER BY s.sale_date;

-- VERIFY
SELECT * FROM analytics.revenue_per_day;
```

<br>

- Which countries generate the most revenue?
<img height="230" alt="2  which countries generate most revenue" src="https://github.com/user-attachments/assets/2155e047-be2f-4b6d-ac52-516325b5bec6" />

```sql
CREATE VIEW analytics.sales_by_country AS
SELECT 
	c.country_name, 
	COUNT(s.sale_id) AS total_order,
	SUM(s.total_amount) AS Most_revenue
FROM core.country c
JOIN core.sales s
ON c.country_id = s.country_id
GROUP BY c.country_name
ORDER BY most_revenue desc;

-- VERIFY
SELECT * FROM analytics.sales_by_country;
```

<br>

- Which sales channel performs best?
<img height="250" alt="3  Which sales channel performs best" src="https://github.com/user-attachments/assets/644a6468-2bfe-41cd-8e1a-84300d632748" />

```sql
CREATE VIEW analytics.sales_by_channel AS
SELECT 
	ch.channel_name, ch.description, 
	COUNT(sale_id) AS total_order,
	SUM(s.total_amount) AS revenue
FROM core.channels ch
JOIN core.sales s
ON ch.channel_id = s.channel_id
GROUP BY ch.channel_name, ch.description;

-- VERIFY
SELECT * FROM analytics.sales_by_channel;
```

<br>

- What products generate the most money?
<img height="230" alt="4  What products generate the most money" src="https://github.com/user-attachments/assets/ad236dd6-5ca1-4ec9-9b94-52e2002e386b" />

```sql
CREATE VIEW analytics.top_product AS
SELECT 
	p.product_name, 
	SUM(si.quantity) AS total_quantity,
	SUM(si.unit_price) AS Revenue
FROM core.salesitems si
JOIN core.products p
ON si.product_id = p.product_id
GROUP BY p.product_name
ORDER BY revenue desc;

-- VERIFY
SELECT * FROM analytics.top_product;
```

<br>

- How valuable is each customer over time?
<img height="230" alt="5  How valuable is each customer over time" src="https://github.com/user-attachments/assets/6b728c4f-48f9-477d-9ecf-c9fc6f03a97a" />

```sql
CREATE VIEW analytics.customer_lifetime_value AS
SELECT 
	c.customer_id,
	co.country_name,
	COUNT(s.sale_id) AS total_Order,
	SUM(s.total_amount) AS top_revenue
FROM core.sales s
JOIN core.customers c
ON s.customer_id = c.customer_id
JOIN core.country co
ON c.country_id = co.country_id
GROUP BY c.customer_id, co.country_name
ORDER BY top_revenue desc;

-- VERIFY
SELECT * FROM analytics.customer_lifetime_value;
```

<br>

- Do discounts actually increase revenue?
<img height="250" alt="6  Do discounts actually increase revenue" src="https://github.com/user-attachments/assets/0938630c-3556-4ece-8d99-68eb2f9a0e0b" />

```sql
CREATE VIEW analytics.discount_analysis AS
SELECT 
	s.discounted, 
	COUNT(s.sale_id) AS total_order,
	SUM(s.total_amount) AS Revenue
FROM core.sales s
GROUP BY s.discounted;

-- VERIFY
SELECT * FROM analytics.discount_analysis;
```

<br>
<br>
<br>


## Business Insights & Decision-Oriented Analysis
Using analytical views and core tables, the following 9 real business questions were answered:

<br>

1. How is the business performing overall?
<img height="250" alt="1  How is the business performing overall" src="https://github.com/user-attachments/assets/577d2c47-f826-4951-a5ce-eef3ceae6bed" />

```sql
CREATE VIEW analytics.business_kpi AS
SELECT 
	COUNT(DISTINCT sale_id) AS total_order,
	SUM(total_amount) AS total_revenue,
	ROUND(SUM(total_amount) / COUNT(DISTINCT sale_id), 2) AS avg_order_value
FROM core.sales;	

-- verify
SELECT * FROM analytics.business_kpi;
```

> Insight: 905 orders generating $324K revenue with an average order value of $358 shows strong sales performance with good customer spending per transaction. 

<br>

2. How is revenue trending over time (monthly)?
<img height="250" alt="2  How is revenue trending over time (monthly)" src="https://github.com/user-attachments/assets/4c9d9823-27f1-4713-a1bd-ab41c2791901" />

```sql
SELECT 
	TO_CHAR(sale_date, 'Month') AS months,
	SUM(total_revenue) AS revenue
FROM analytics.revenue_per_day
GROUP BY months
ORDER BY revenue desc;
```

> Insight: May generated the most revenue at $142K, followed by April at $133K, then June at $49K. The large differences between months indicate seasonal patterns or successful promotional campaigns.

<br>

3. Top 2 countries that are driving revenue, and underperforming?
<img height="250" alt="3  Top 2 countries that are driving revenue, and underperforming" src="https://github.com/user-attachments/assets/fd3af6c2-1ab5-45ad-89a0-9cde6f906b76" />

```sql
SELECT * FROM analytics.sales_by_country 
ORDER BY most_revenue DESC;
```

> Insight: Germany ($75K) and France ($72K) are the strongest markets. Spain ($41K) and Portugal($30k) are weaker markets that need focused improvement strategies.

<br>

4. Which channel should receive more investment?
<img height="250" alt="4  Which channel should receive more investment" src="https://github.com/user-attachments/assets/f4842cee-0f08-4623-8a0e-04977e194353" />

```sql
SELECT * FROM analytics.sales_by_channel
ORDER BY revenue ASC;
```

> Insight: E-commerce generates $172K compared to App Mobile's $153K, E-commerce is more effective at converting sales and generating higher value.

<br>

5. Which products generate the most revenue and sales volume?
<img height="250" alt="5  Which products generate the most revenue and sales volume" src="https://github.com/user-attachments/assets/9597d442-7fe9-4538-82ea-0c5bf180fe91" />

```sql
SELECT * FROM analytics.product_performance
ORDER BY revenue desc;
```

> Insight: The top three products are Bold High-Waist Dress ($601), Modern High-Waist Trousers ($591), and Polished Wrap Trousers ($563), each selling consistently at 24-26 units.

<br>

6. Who are the top 10 most valuable customers?
<img height="250" alt="6  Who are the top 10 most valuable customers" src="https://github.com/user-attachments/assets/5d697b71-b6f0-40f5-b345-1e26a5a224d7" />

```sql
SELECT * FROM analytics.customer_lifetime_value
ORDER BY top_revenue DESC
LIMIT 10;
```

> Insight: The highest-value customer is "customer_id 99" from Spain with $2,679 across 7 orders. The top 10 customers come from different countries, showing wide market reach, with purchase frequency ranging from 3 to 7 orders.

<br>

7. How does customer spending behavior vary by country?
<img height="250" alt="7  How does customer spending behavior vary by country" src="https://github.com/user-attachments/assets/ef004138-d2f0-47cd-a637-eb791ce09159" />

```sql
SELECT 
	*, 
	ROUND(most_revenue/total_order, 2) AS avg_country_value
FROM analytics.sales_by_country
ORDER BY most_revenue DESC;
```

> Insight: Customers in Portugal spend the most per order ($374), followed by Spain ($364) and France ($363). Germany generates the highest total revenue through more orders rather than higher spending per order.

<br>

8. Do discounted orders contribute significantly to revenue?
<img height="250" alt="8  Do discounted orders contribute significantly to revenue" src="https://github.com/user-attachments/assets/25f5729c-0faa-4341-9d7a-2cbabe5f89b9" />

```sql
SELECT * FROM analytics.discount_analysis;
```

> Insight: Discounts represent only 7.3% of revenue ($24K vs $301K). Either selective usage or discount strategy needs optimization.


<br>

9. Which products are at risk of stockout by country? <br>
   When risk of stockout = "<10" <br>
<img height="250" alt="9  Which products are at risk of stockout by country" src="https://github.com/user-attachments/assets/29d61faa-059f-4ee6-83e5-da1fdc364cda" />

```sql
SELECT 
	p.product_name, co.country_name, st.stock_quantity
FROM core.stock	st
JOIN core.country co
ON st.country_id = co.country_id
JOIN core.products p
ON st.product_id = p.product_id
WHERE st.stock_quantity < 10
ORDER BY st.stock_quantity ASC;
```

> 550 products have fewer than 10 units in stock, a serious shortage risk in top markets like Germany and France. Quick restocking is essential to avoid missing sales opportunities.


<br>
<br>
<br>

## Tools and Key Outcomes

### Tools & Technologies
- PostgreSQL
- SQL (DDL, DML, Constraints, Indexing)
- ER Modeling
- EXPLAIN ANALYZE (Query Optimization)

<br>
<br>

### Key Outcomes
- Built a production-style relational database
- Implemented SQL-based ETL without external tools
- Enforced business rules at the database level
- Delivered analytics-ready views and queries
- Designed a system suitable for dashboards and BI tools


