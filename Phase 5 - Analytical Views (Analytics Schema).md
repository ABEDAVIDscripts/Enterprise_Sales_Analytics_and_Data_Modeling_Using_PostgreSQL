## Phase 5: Analytical Views (Analytics Schema)
Reusable business views were created to simplify reporting <br>



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
```

<br>

- What products generate the most money?
<img height="230" alt="4  What products generate the most money" src="https://github.com/user-attachments/assets/ad236dd6-5ca1-4ec9-9b94-52e2002e386b" />

```sql
CREATE VIEW analytics.product_performance AS
SELECT 
	p.product_name, 
	SUM(si.quantity) AS total_quantity,
	SUM(si.unit_price) AS Revenue
FROM core.salesitems si
JOIN core.products p
ON si.product_id = p.product_id
GROUP BY p.product_name
ORDER BY revenue desc;
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