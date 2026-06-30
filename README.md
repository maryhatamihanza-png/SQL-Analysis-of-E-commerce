# SQL and Python-Analysis-of-E-commerce
# Brazilian E-Commerce Marketplace Analytics using SQL

## Project Overview

This project analyzes the Brazilian E-Commerce Public Dataset by Olist using SQL and Python. The goal is to explore marketplace performance across revenue, customer behavior, delivery efficiency, customer satisfaction, seller performance, and geographic activity.

The analysis was completed in a Kaggle notebook by loading multiple CSV files into an in-memory SQLite database and running SQL queries to answer business questions.

## Business Objectives

The project focuses on answering the following business questions:

* How has marketplace revenue changed over time?
* Which product categories generate the most revenue?
* Are customers mostly one-time or repeat buyers?
* Which payment methods are most used?
* How reliable are deliveries?
* Which states experience the longest delivery delays?
* Do delivery delays impact customer review scores?
* Which sellers generate the most revenue?
* Which sellers have weaker delivery performance?
* Which Brazilian states generate the most revenue and highest average order value?

## Tools Used

* SQL
* SQLite
* Python
* Pandas
* NumPy
* Matplotlib
* Kaggle Notebook

## Dataset

The project uses the Brazilian E-Commerce Public Dataset by Olist. The dataset includes marketplace data related to:

* Customers
* Orders
* Payments
* Reviews
* Order items
* Products
* Sellers
* Geolocation
* Product category translation

## Data Model and Relationships

The main relationships used in this analysis are:

```text
customers.customer_id → orders.customer_id
orders.order_id → payments.order_id
orders.order_id → reviews.order_id
orders.order_id → order_items.order_id
order_items.product_id → products.product_id
order_items.seller_id → sellers.seller_id
products.product_category_name → category_translation.product_category_name
```

## Key Analysis Areas

### 1. Revenue Analysis

Monthly revenue was calculated by joining orders with payments and filtering for delivered orders.

```text
SELECT sum(op.payment_value) as total_revenue, strftime('%Y-%m', o.order_purchase_timestamp) as month
FROM orders o
JOIN order_payments op
on o.order_id=op.order_id
where o.order_status='delivered' 
group by month;
```


Business value:

* Identifies revenue trends
* Highlights growth periods
* Supports seasonal performance analysis

### 2. Revenue by Product Category

Product category revenue was analyzed by joining order items, products, and category translation tables.

```text
SELECT sum(op.payment_value) as total_revenue, pcnt.product_category_name_english as product_category
from order_payments op
Left Join order_items oi on oi.order_id=op.order_id
left join products p on p.product_id=oi.product_id
left JOIN product_category_name_translation pcnt on pcnt.product_category_name=p.product_category_name
group by product_category
order by total_revenue DESC;
```

Key finding:

The highest revenue categories included:

* Health and beauty
* Watches and gifts
* Bed, bath, and table
* Sports and leisure
* Computer accessories

Business value:

* Supports category-level marketing strategy
* Helps prioritize inventory and seller partnerships

### 3. Customer Behavior Analysis

Customers were segmented based on their number of delivered orders.

Segments:

```text
One-Time Customer: 1 order
Regular Customer: 2–4 orders
Loyal Customer: 5+ orders
```
```text
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        COUNT(DISTINCT o.order_id) AS order_count
    FROM customers c
    JOIN orders o
        ON c.customer_id = o.customer_id
    GROUP BY c.customer_unique_id
),
customer_segments AS (
    SELECT
        customer_unique_id,
        order_count,
        CASE
            WHEN order_count = 1 THEN 'One Time Customer'
            WHEN order_count >= 5 THEN 'Loyal Customer'
            ELSE 'Regular Customer'
        END AS customer_segmentation
    FROM customer_orders
)
SELECT
    customer_segmentation,
    COUNT(*) AS customer_count
FROM customer_segments
GROUP BY customer_segmentation
ORDER BY customer_count DESC;
```
* Used the below query to test for the accuracy at smaller scale

```text
SELECT
    c.customer_unique_id,
    COUNT(DISTINCT o.order_id) AS order_count
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_unique_id
ORDER BY order_count DESC
LIMIT 20;
```

Key finding:

Most customers were one-time buyers, showing a strong acquisition pattern but limited retention.

Business recommendation:

The marketplace should focus on repeat purchase strategies such as loyalty programs, personalized campaigns, and second-purchase discounts.

### 4. Payment Method Analysis

Payment methods were analyzed by transaction count and total payment value.

```text
SELECT	
	count(order_id) as order_count,
	payment_type
from order_payments
group by payment_type
order by order_count desc;
```

Key finding:

Credit card was the dominant payment method.

Business value:

The marketplace should ensure the credit card checkout experience is fast, reliable, and frictionless.

### 5. Delivery Performance Analysis

Delivery performance was measured by comparing actual delivery dates against estimated delivery dates.

Calculation logic:

```sql
julianday(order_delivered_customer_date) 
- julianday(order_estimated_delivery_date)
```

Interpretation:

```text
Positive value = late delivery
Negative value = early delivery
```

Business value:

Delivery performance directly impacts customer experience and review scores.

### 6. Delivery Delay by State

Average delivery delay was analyzed by customer state.

Business value:

This helps identify regions where logistics performance may need improvement.

Potential actions:

* Improve carrier coverage
* Review regional delivery estimates
* Monitor state-level logistics performance

### 7. Customer Satisfaction Analysis

Review scores were analyzed from 1 to 5.

Business value:

Review score distribution helps identify the overall customer satisfaction level and potential operational issues.

### 8. Delivery Delay vs Review Score

This analysis checked whether lower review scores were associated with longer delivery delays.

Business implication:

If lower reviews are linked to delivery delays, improving delivery reliability may directly improve customer satisfaction.

### 9. Seller Performance Analysis

Seller revenue and items sold were analyzed to identify top-performing sellers.

Business value:

Top sellers can be prioritized for support, partnerships, and marketplace growth strategies.

### 10. Seller Delivery Performance

Seller delivery delays were analyzed using sellers with at least 50 orders to avoid misleading results from low-volume sellers.

Business value:

This helps identify sellers who may require operational monitoring or performance improvement plans.

### 11. Geographic Revenue Analysis

Revenue and total orders were analyzed by Brazilian customer state.

Business value:

This identifies the strongest revenue regions and potential geographic expansion opportunities.

### 12. Average Order Value by State

Average order value was calculated by dividing total payment value by total delivered orders.

Business value:

This helps identify states with higher purchasing value and stronger commercial potential.

### 13. Product Category Satisfaction

Average review scores were analyzed by product category.

Business value:

Low-rated categories may require deeper investigation into product quality, seller reliability, delivery issues, or inaccurate product descriptions.

## Key Business Findings

* Revenue performance can be tracked over time to identify growth and seasonality.
* High-revenue product categories can be prioritized for marketing and inventory planning.
* Most customers are one-time buyers, creating a major opportunity for retention improvement.
* Credit card is the dominant payment method.
* Delivery performance varies across states and sellers.
* Customer satisfaction may be affected by delivery performance.
* Marketplace revenue is concentrated across specific sellers and regions.
* Some states have higher average order values, suggesting regional demand differences.

## Business Recommendations

### 1. Improve Delivery Reliability

Focus on states and sellers with weaker delivery performance.

### 2. Strengthen Seller Monitoring

Track seller performance using revenue, order volume, delivery delay, and review score.

### 3. Increase Customer Retention

Develop strategies to increase repeat purchases, such as:

* Loyalty discounts
* Personalized recommendations
* Post-purchase email campaigns
* Second-purchase incentives

### 4. Prioritize High-Value Categories

Invest marketing and seller support into categories with strong revenue and customer satisfaction.

### 5. Expand Geographic Strategy

Analyze high-revenue and high-AOV states to better understand regional demand and growth opportunities.

## Limitations

* The dataset covers historical orders from 2016 to 2018.
* The data is specific to Brazil.
* Customer demographic data is limited.
* Review text was not deeply analyzed.
* SQLite was used for the Kaggle notebook, while PostgreSQL could be used for a production-style GitHub version.

## Repository Structure

```text
brazilian-ecommerce-sql-analysis/
│
├── README.md
├── notebook/
│   └── ecommerce_marketplace_analysis.ipynb
│
├── sql/
│   ├── 01_monthly_revenue.sql
│   ├── 02_category_revenue.sql
│   ├── 03_customer_segmentation.sql
│   ├── 04_payment_method_analysis.sql
│   ├── 05_delivery_performance.sql
│   ├── 06_seller_performance.sql
│   └── 07_geographic_analysis.sql
│
├── visuals/
│   ├── monthly_revenue_trend.png
│   ├── category_revenue.png
│   ├── customer_segments.png
│   ├── payment_methods.png
│   ├── delivery_delay_distribution.png
│   └── state_revenue.png
│
└── documentation/
    ├── business_questions.md
    ├── data_model.md
    └── insights_and_recommendations.md
```

## Sample SQL Query

```sql
SELECT
    strftime('%Y-%m', o.order_purchase_timestamp) AS month,
    ROUND(SUM(p.payment_value), 2) AS revenue
FROM orders o
JOIN payments p
    ON o.order_id = p.order_id
WHERE o.order_status = 'delivered'
GROUP BY month
ORDER BY month;
```

## Conclusion

This project demonstrates how SQL can be used to analyze a real e-commerce marketplace dataset and generate business insights across revenue, customers, sellers, delivery performance, satisfaction, and geography.

The analysis shows the ability to:

* Build SQL queries across multiple related tables
* Translate raw data into business insights
* Identify operational risks and growth opportunities
* Present findings clearly for business decision-making
