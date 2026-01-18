## Phase 6: Business Insights & Decision-Oriented Analysis
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