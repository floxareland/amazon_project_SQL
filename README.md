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
