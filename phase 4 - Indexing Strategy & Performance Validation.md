## Phase 4: Indexing Strategy & Performance Validation

### Indexing Design Principles
- All foreign keys indexed
- High-usage date columns indexed
- Avoided indexing low-cardinality and small tables

<br>

### Index Implementation

#### 1. Index Foreign Keys on Transaction Tables
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

#### 2. Date Column Indexing <br>
sales.sale_date <br>
```sql
CREATE INDEX idx_sales_sale_date
ON core.sales (sale_date);
```

<br>
<br>

#### 3. Index Usage Validation <br>
Confirm that PostgreSQL is actually using the created index 

```sql
EXPLAIN ANALYZE
SELECT *
FROM core.sales
WHERE sale_date = '2025-04-01';
```

<img height="250" alt="validate index" src="https://github.com/user-attachments/assets/d8fa66b8-5542-44e8-a7b5-c8a757f4ab65" /> <BR>

> PostgreSQL correctly selected sequential scans for low-selectivity queries on small tables, and index scans for highly selective predicates, demonstrating proper cost-based optimization behavior.

<br>
<br>
<br>