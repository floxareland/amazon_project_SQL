# amazon_project_SQL
![Amazon Logo](https://github.com/floxareland/amazon_project_SQL/blob/main/amazon-6536326_1280.webp)

# **Amazon Sales Analysis Project**

## **Project Overview**

I analyzed a dataset of over 20,000 sales records from an Amazon-like e-commerce platform, leveraging PostgreSQL for in-depth customer behavior, product performance, and sales trend analysis. This project involved complex SQL queries to address key business challenges, such as revenue analysis, customer segmentation, and inventory management.  

Additionally, I focused on data cleaning, handling null values, and structuring queries to solve real-world business problems. An ERD diagram is included to visually illustrate the database schema and relationships between tables.

## **Database Setup & Design**

### **Schema Structure**

```sql

--Create category Table
CREATE TABLE IF NOT EXISTS category(
category_id INT PRIMARY KEY,
category_name VARCHAR(25)
);

--Create customers Table

CREATE TABLE IF NOT EXISTS customers(
customer_id INT PRIMARY KEY,
first_name VARCHAR(20),
last_name VARCHAR(20),
state VARCHAR(20),
adress VARCHAR(5) DEFAULT ('xxx')
);

--Create sellers Table

CREATE TABLE IF NOT EXISTS sellers(
seller_id INT PRIMARY KEY,
seller_name VARCHAR(25),
origin VARCHAR(5)
);

--Create products Table
CREATE TABLE IF NOT EXISTS products(
product_id INT PRIMARY KEY,
product_name VARCHAR(50),
price FLOAT,
cogs FLOAT,
category_id INT, --FK	
CONSTRAINT products_fk_category FOREIGN KEY(category_id) REFERENCES category(category_id)
);

--Create orders Table
CREATE TABLE IF NOT EXISTS orders(
order_id INT PRIMARY KEY,
order_date DATE,
customer_id INT, --FK
seller_id INT, --FK
order_status VARCHAR(15),
CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
CONSTRAINT orders_fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);

--Create order_items Table
CREATE TABLE IF NOT EXISTS order_items(
order_item_id INT PRIMARY KEY,
order_id INT, --FK
product_id INT, --FK
quantity INT,
price_per_unit FLOAT,
CONSTRAINT order_items_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

--Create payments Table

CREATE TABLE IF NOT EXISTS payments(
payment_id INT PRIMARY KEY,
order_id INT, --FK
payment_date DATE,
payment_status VARCHAR(20),
CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

--Create shippings Table

CREATE TABLE IF NOT EXISTS shippings(
shipping_id INT PRIMARY KEY,
order_id INT, --FK
shipping_date DATE,
return_date DATE,
shipping_providers VARCHAR(15),
delivery_status VARCHAR(15),
CONSTRAINT shipping_fk_orders FOREIGN KEY(order_id) REFERENCES orders(order_id)
);

--Create inventory Table

CREATE TABLE IF NOT EXISTS inventory(
inventory_id INT PRIMARY KEY,
product_id INT, --FK
stock INT,
warehouse_id INT, 
last_stock_date DATE,
CONSTRAINT inventory_fk_products FOREIGN KEY(product_id) REFERENCES products(product_id)
);

ALTER TABLE sellers
ALTER COLUMN origin TYPE VARCHAR(10);
```
## **Task: Data Cleaning**

I cleaned the dataset by:
- **Removing duplicates**: Duplicates in the customer and order tables were identified and removed.
- **Handling missing values**: Null values in critical fields (e.g., customer address, payment status) were either filled with default values or handled using appropriate methods.

---

## **Handling Null Values**

Null values were handled based on their context:
- **Customer addresses**: Missing addresses were assigned default placeholder values.
- **Payment statuses**: Orders with null payment statuses were categorized as “Pending.”
- **Shipping information**: Null return dates were left as is, as not all shipments are returned.

---

## **Objective**

The primary objective of this project is to showcase SQL proficiency through complex queries that address real-world e-commerce business challenges. The analysis covers various aspects of e-commerce operations, including:
- Customer behavior
- Sales trends
- Inventory management
- Payment and shipping analysis
- Forecasting and product performance
  

## **Identifying Business Problems**

Key business problems identified:
1. Low product availability due to inconsistent restocking.
2. High return rates for specific product categories.
3. Significant delays in shipments and inconsistencies in delivery times.
4. High customer acquisition costs with a low customer retention rate.

---

## **Solving Business Problems**

### Solutions Implemented:
1. Find top 10 products by total sales value. Include product name, product name and total sales values.
```sql
ALTER TABLE order_items
ADD COLUMN total_sale FLOAT;
UPDATE order_items 
SET total_sale=quantity * price_per_unit;

SELECT oi.product_id,
p.product_name,
SUM(oi.total_sale) AS total_sale
FROM orders AS o
JOIN order_items AS oi
ON oi.order_id=o.order_id
JOIN products AS p
ON p.product_id=oi.product_id
GROUP BY 1,2
ORDER BY total_sale DESC
LIMIT 10;

```

2. Calculate total revenue generated by each product category. Include the percentage contribution of each category to total revenue.

```sql
SELECT c.category_name,SUM(oi.total_sale),SUM(oi.total_sale)/(SELECT SUM(order_items.total_sale) FROM order_items) *100 AS percentage
FROM order_items AS oi
JOIN products AS p
ON oi.product_id=p.product_id
JOIN category AS c
ON p.category_id=c.category_id
GROUP BY c.category_name
ORDER BY percentage DESC;

```
3. Compute the Average order value for each customer. Include only customers with more than 5 orders.

```sql

SELECT c.customer_id,	
		CONCAT(c.first_name,' ',c.last_name) AS full_name,
		SUM(total_sale)/COUNT(o.order_id) AS avg_order_value,
		COUNT(o.order_id) AS num_orders
FROM orders AS o
JOIN customers AS c
ON c.customer_id=o.customer_id
JOIN order_items AS oi 
ON oi.order_id=o.order_id
GROUP BY c.customer_id,full_name
HAVING COUNT(o.order_id)>5;
```
4. Query monthly total sales over the last two years. Display the sales trend, grouping by month, return current_month sale, last month sale.

```sql
SELECT year,
	   month,
	   total_sale AS current_month_sale,
	   LAG(total_sale,1) OVER(ORDER BY year,month) AS last_month_sale
FROM
(
SELECT EXTRACT(MONTH from o.order_date) AS month,
	   EXTRACT(YEAR from o.order_date) AS year,
	   ROUND(SUM(oi.total_sale)::numeric,2) AS total_sale
FROM orders AS o
JOIN order_items as oi
ON oi.order_id = o.order_id
WHERE order_date >= CURRENT_DATE - INTERVAL '2 YEAR'
GROUP BY 1,2
ORDER BY year,month 
) AS table1

```

5. Find customers who have registered but never placed an order. List customer details and time since their registration.

```sql

SELECT *
FROM customers AS c
LEFT JOIN orders AS o
ON c.customer_id=o.customer_id
WHERE o.order_id IS NULL;
```

6. Identify the best selling product category for each state. Include the total sales for that category within each state.

```sql
SELECT * FROM(
SELECT c.state,
       cat.category_name,
	   SUM(oi.total_sale),
	   RANK()OVER(PARTITION BY c.state ORDER BY SUM(oi.total_sale)DESC) AS rank_state
FROM orders AS o
JOIN customers AS c
ON o.customer_id=c.customer_id
JOIN order_items AS oi
ON o.order_id=oi.order_id
JOIN products AS p
ON oi.product_id=p.product_id
JOIN category AS cat
ON cat.category_id=p.category_id
GROUP BY 1,2
ORDER BY 1,3 DESC 
)
WHERE rank_state=1;
```

7. Calculate the total value of orders placed by each customer over their lifetime. Rank customers based on their CLTV.

```sql
SELECT c.customer_id,CONCAT(c.first_name,' ',c.last_name),SUM(oi.total_sale) AS CLTV, DENSE_RANK() OVER(ORDER BY SUM(oi.total_sale) DESC)
FROM customers AS c
JOIN orders AS o
ON o.customer_id=c.customer_id
JOIN order_items AS oi
ON o.order_id=oi.order_id
GROUP BY 1,2;
```
8. Query products with stock levels below 1o units. Include last restock date and warehouse information.

```sql
SELECT p.product_id,p.product_name,i.stock AS stock_left,i.inventory_id,i.warehouse_id
FROM inventory AS i
JOIN products AS p
ON p.product_id=i.product_id
WHERE i.stock<10;
```
9. Identify the orders where shipping date is later than 3 days after the order date. Include customer, order details and delivery provider. 

```sql
SELECT * 
FROM orders AS o
JOIN customers AS c
ON c.customer_id=o.customer_id
JOIN shippings AS s
ON s.order_id=o.order_id
WHERE s.shipping_date>o.order_date + INTERVAL '3 days';
```
10. Calculate the percentage of succesful payments across all orders. Include breakdowns by payment status.

```sql
SELECT p.payment_status,100*COUNT(*)/(SELECT COUNT(*) FROM payments) payment_percentage_breakdown
FROM payments AS p
JOIN orders AS o
ON o.order_id=p.order_id
GROUP BY 1;
```
11. Find the top 5 sellers based on total sales value. Include both succesful and failed orders and display their percentage of succesful orders.

```sql

WITH top_sellers
AS
(SELECT 
	s.seller_id,
	s.seller_name,
	SUM(oi.total_sale) as total_sale
FROM orders as o
JOIN
sellers as s
ON o.seller_id = s.seller_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5
),

sellers_reports
AS
(SELECT 
	o.seller_id,
	ts.seller_name,
	o.order_status,
	COUNT(*) as total_orders
FROM orders as o
JOIN 
top_sellers as ts
ON ts.seller_id = o.seller_id
WHERE 
	o.order_status NOT IN ('Inprogress', 'Returned')
	
GROUP BY 1, 2, 3
)
SELECT 
	seller_id,
	seller_name,
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END) as Completed_orders,
	SUM(CASE WHEN order_status = 'Cancelled' THEN total_orders ELSE 0 END) as Cancelled_orders,
	SUM(total_orders) as total_orders,
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END)::numeric/
	SUM(total_orders)::numeric * 100 as successful_orders_percentage
	
FROM sellers_reports
GROUP BY 1, 2;
```
12. Calculate the profit margin for each product. Rank products by their profit margin, showing highest to lowest.

```sql
SELECT 
	product_id,
	product_name,
	profit_margin,
	DENSE_RANK() OVER( ORDER BY profit_margin DESC) as product_ranking
FROM
(SELECT 
	p.product_id,
	p.product_name,
	-- SUM(total_sale - (p.cogs * oi.quantity)) as profit,
	SUM(total_sale - (p.cogs * oi.quantity))/sum(total_sale) * 100 as profit_margin
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
GROUP BY 1, 2
) as t1
```
13. Query the top 10 products by the number of returns. Display the return rate as a percentage of total units sold for each product.
```sql
SELECT 
	p.product_id,
	p.product_name,
	COUNT(*) as total_unit_sold,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_returned,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END)::numeric/COUNT(*)::numeric * 100 as return_percentage
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN orders as o
ON o.order_id = oi.order_id
GROUP BY 1, 2
ORDER BY 5 DESC;

```

14.Identify sellers who haven’t made any sales in the last 6 months. Show the last sale date and total sales from those sellers.
```sql
WITH cte1 
AS
(SELECT * FROM sellers
WHERE seller_id NOT IN (SELECT seller_id FROM orders WHERE order_date >= CURRENT_DATE - INTERVAL '6 month')
)

SELECT 
o.seller_id,
MAX(o.order_date) as last_sale_date,
MAX(oi.total_sale) as last_sale_amount
FROM orders as o
JOIN 
cte1
ON cte1.seller_id = o.seller_id
JOIN order_items as oi
ON o.order_id = oi.order_id
GROUP BY 1;

```

15. Return if they have made more than 5 returns and "New" otherwise. Also list customer ID, name, total orders, and total returns.

```sql

SELECT 
c_full_name as customers,
total_orders,
total_return,
CASE
	WHEN total_return > 5 THEN 'Returning_customers' ELSE 'New'
END as cx_category
FROM
(SELECT 
	CONCAT(c.first_name, ' ', c.last_name) as c_full_name,
	COUNT(o.order_id) as total_orders,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_return	
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1
)
```

16. Identify the top 5 customers with the highest number of orders for each state. Include the number of orders and total sales for each customer.

```sql
SELECT * FROM 
(SELECT 
	c.state,
	CONCAT(c.first_name, ' ', c.last_name) as customers,
	COUNT(o.order_id) as total_orders,
	SUM(total_sale) as total_sale,
	DENSE_RANK() OVER(PARTITION BY c.state ORDER BY COUNT(o.order_id) DESC) as rank
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
customers as c
ON 
c.customer_id = o.customer_id
GROUP BY 1, 2
) as t1
WHERE rank <=5
```

17. Calculate the total revenue handled by each shipping provider. Include the total number of orders handled and the average delivery time for each provider.

```sql
SELECT 
	s.shipping_providers,
	COUNT(o.order_id) as order_handled,
	SUM(oi.total_sale) as total_sale,
	COALESCE(AVG(s.return_date - s.shipping_date), 0) as average_days
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
shippings as s
ON 
s.order_id = o.order_id
GROUP BY 1;
```

18. Top 10 product with highest decreasing revenue ratio compare to last year(2022) and current_year(2023). Return product_id, product_name, category_name, 2022 revenue and 2023 revenue decrease ratio at end Round the results.

```sql
WITH last_year_sale
as
(
SELECT 
	p.product_id,
	p.product_name,
	SUM(oi.total_sale) as revenue
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON 
p.product_id = oi.product_id
WHERE EXTRACT(YEAR FROM o.order_date) = 2022
GROUP BY 1, 2
),

current_year_sale
AS
(
SELECT 
	p.product_id,
	p.product_name,
	SUM(oi.total_sale) as revenue
FROM orders as o
JOIN 
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON 
p.product_id = oi.product_id
WHERE EXTRACT(YEAR FROM o.order_date) = 2023
GROUP BY 1, 2
)

SELECT
	cs.product_id,
	ls.revenue as last_year_revenue,
	cs.revenue as current_year_revenue,
	ls.revenue - cs.revenue as rev_diff,
	ROUND((cs.revenue - ls.revenue)::numeric/ls.revenue::numeric * 100, 2) as reveneue_dec_ratio
FROM last_year_sale as ls
JOIN
current_year_sale as cs
ON ls.product_id = cs.product_id
WHERE 
	ls.revenue > cs.revenue
ORDER BY 5 DESC
LIMIT 10


```

## **Learning Outcomes**

Through this project, I was able to:
- Develop and structure a normalized database schema.
- Prepare and clean real-world datasets for effective analysis.
- Apply advanced SQL methods, such as window functions, subqueries, and joins.
- Perform comprehensive business analysis using SQL.
- Enhance query efficiency and manage large datasets effectively.

---

## **Conclusion**

This advanced SQL project showcases my ability to address real-world e-commerce challenges through structured queries. From enhancing customer retention to optimizing inventory and logistics, the analysis offers valuable insights into operational hurdles and their solutions.

Completing this project has deepened my understanding of how SQL can be leveraged to solve complex data problems and support data-driven business decisions.

---

### **Entity Relationship Diagram (ERD)**
![ERD](https://github.com/floxareland/amazon_project_SQL/blob/main/Updated%20ERD%20-%20Amazon.png)

---

