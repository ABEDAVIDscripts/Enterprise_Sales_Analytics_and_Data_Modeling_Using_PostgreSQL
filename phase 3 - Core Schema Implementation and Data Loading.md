## Phase 3: Core Schema Implementation and Data Loading

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