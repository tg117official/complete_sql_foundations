# Session 1 — Data Modeling Foundations

**MySQL 8 | Hands-on workbook | E-commerce data**

## What you will learn

You will use a working database to understand:

1. Business entities, tables, columns, keys, and relationships.
2. Conceptual, logical, and physical data models.
3. The **grain** of a table: what one row represents.
4. Operational (OLTP) queries and transactions.
5. Analytical (OLAP) queries, measures, and dimensions.
6. Why the selling price on an order can differ from today's product price.
7. Joined views, physically denormalized tables, duplicated values, and data freshness.
8. A basic star schema with fact and dimension tables.

**This workbook covers Session 1 only.** The rules for 1NF, 2NF, 3NF, and BCNF are covered in later sessions.

## Before you begin

- Use **MySQL Server 8** and MySQL Workbench or another MySQL client.
- The complete setup is included below. **No other file is needed.**
- Run the setup once, then complete the exercises **in order**. Each exercise has its solution immediately after the task.
- Run SQL inside each code block in the order shown. For a query with a result, compare your result with the stated expected result.
- When an exercise uses `START TRANSACTION`, finish it with `ROLLBACK`. Do not execute `CREATE TABLE`, `DROP TABLE`, or `CREATE VIEW` inside those transactions: in MySQL, these statements can cause an implicit commit.
- **Important:** Exercise 1 intentionally drops and rebuilds a database named `tg_data_modeling_s1`. Use it only on a practice MySQL server. Do not run it on production. Running Exercise 1 again removes any changes you made in that practice database.

**Business rule for every sales exercise:** Only orders with `order_status = 'CONFIRMED'` count toward *confirmed revenue*. `CANCELLED` and `PENDING` orders do not. Revenue is `quantity × unit_price_at_sale`. All amounts are in INR (₹). No tax, shipping, or refunds are included in this small example.

---

# Part 1 — Build and understand the data model

## Exercise 1 — Create the complete practice database

**Concept: Tables and sample data**

An online store needs to keep customers, products, orders, and the items inside each order. We will create those four tables and load sample records.

**Task:** Run the full setup below **once**. Then verify that the four tables contain data.

**Solution — MySQL setup**

```sql
-- WARNING: This resets ONLY the practice database named below.
DROP DATABASE IF EXISTS tg_data_modeling_s1;
CREATE DATABASE tg_data_modeling_s1
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;
USE tg_data_modeling_s1;

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(80) NOT NULL,
    city VARCHAR(60) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(80) NOT NULL,
    category VARCHAR(50) NOT NULL,
    current_price DECIMAL(10,2) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    order_status VARCHAR(12) NOT NULL,
    CONSTRAINT fk_s1_orders_customer
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
) ENGINE=InnoDB;

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price_at_sale DECIMAL(10,2) NOT NULL,
    CONSTRAINT fk_s1_items_order
        FOREIGN KEY (order_id) REFERENCES orders(order_id),
    CONSTRAINT fk_s1_items_product
        FOREIGN KEY (product_id) REFERENCES products(product_id)
) ENGINE=InnoDB;

INSERT INTO customers (customer_id, customer_name, city) VALUES
(1, 'Riya',  'Pune'),
(2, 'Rahul', 'Mumbai'),
(3, 'Kiran', 'Pune'),
(4, 'Anaya', 'Delhi'),
(5, 'Arjun', 'Bengaluru'),
(6, 'Meera', 'Hyderabad'),
(7, 'Neha',  'Jaipur'),
(8, 'Kabir', 'Mumbai'),
(9, 'Sara',  'Chennai'),
(10,'Tara',  'Kochi');

INSERT INTO products
    (product_id, product_name, category, current_price) VALUES
(101, 'Keyboard',     'Accessories', 1500.00),
(102, 'Mouse',        'Accessories',  700.00),
(103, 'Monitor',      'Displays',   12000.00),
(104, 'USB-C Hub',    'Accessories', 2300.00),
(105, 'Webcam',       'Cameras',     3500.00),
(106, 'Laptop Stand', 'Accessories', 1800.00),
(107, 'Headphones',   'Audio',       2500.00),
(108, 'SSD',          'Storage',     5500.00),
(109, 'Desk Lamp',    'Accessories', 1200.00),
(110, 'Microphone',   'Audio',       4000.00),
(111, 'Router',       'Networking',  3000.00);

INSERT INTO orders
    (order_id, customer_id, order_date, order_status) VALUES
(1001, 1, '2026-10-01', 'CONFIRMED'),
(1002, 2, '2026-10-02', 'CONFIRMED'),
(1003, 1, '2026-10-02', 'CONFIRMED'),
(1004, 3, '2026-10-03', 'CONFIRMED'),
(1005, 4, '2026-10-03', 'CONFIRMED'),
(1006, 2, '2026-10-04', 'CANCELLED'),
(1007, 5, '2026-10-05', 'CONFIRMED'),
(1008, 3, '2026-10-05', 'CONFIRMED'),
(1009, 6, '2026-10-06', 'CONFIRMED'),
(1010, 7, '2026-10-06', 'CONFIRMED'),
(1011, 8, '2026-10-07', 'CONFIRMED'),
(1012, 9, '2026-10-07', 'CONFIRMED'),
(1013, 4, '2026-10-08', 'PENDING'),
(1014, 6, '2026-10-08', 'CONFIRMED'),
(1015, 7, '2026-10-09', 'CONFIRMED');

INSERT INTO order_items
    (order_item_id, order_id, product_id, quantity, unit_price_at_sale) VALUES
( 1, 1001, 101, 1,  1500.00),
( 2, 1001, 102, 2,   700.00),
( 3, 1002, 103, 1, 12000.00),
( 4, 1002, 104, 1,  2300.00),
( 5, 1003, 101, 1,  1450.00),
( 6, 1003, 105, 1,  3500.00),
( 7, 1004, 102, 1,   700.00),
( 8, 1004, 106, 2,  1800.00),
( 9, 1005, 103, 1, 11800.00),
(10, 1005, 105, 2,  3500.00),
(11, 1006, 101, 2,  1500.00),
(12, 1006, 102, 1,   700.00),
(13, 1007, 104, 1,  2300.00),
(14, 1007, 106, 1,  1800.00),
(15, 1008, 102, 3,   650.00),
(16, 1008, 105, 1,  3500.00),
(17, 1009, 107, 1,  2500.00),
(18, 1009, 102, 2,   700.00),
(19, 1010, 108, 1,  5500.00),
(20, 1010, 109, 1,  1200.00),
(21, 1011, 103, 1, 12000.00),
(22, 1011, 110, 1,  4000.00),
(23, 1012, 104, 2,  2300.00),
(24, 1012, 105, 1,  3500.00),
(25, 1013, 101, 1,  1500.00),
(26, 1013, 110, 1,  4000.00),
(27, 1014, 107, 2,  2400.00),
(28, 1014, 108, 1,  5300.00),
(29, 1015, 109, 2,  1200.00),
(30, 1015, 110, 1,  4000.00);

-- Verify that setup is complete.
SELECT DATABASE() AS current_database, VERSION() AS mysql_version;

SELECT 'customers' AS table_name, COUNT(*) AS row_count FROM customers
UNION ALL SELECT 'products', COUNT(*) FROM products
UNION ALL SELECT 'orders', COUNT(*) FROM orders
UNION ALL SELECT 'order_items', COUNT(*) FROM order_items;
```

**Expected row counts**

| Table | Rows |
|---|---:|
| `customers` | 10 |
| `products` | 11 |
| `orders` | 15 |
| `order_items` | 30 |

**What you learned:** A data model has multiple related tables. We will use this same dataset for every remaining exercise.

## Exercise 2 — Discover conceptual, logical, and physical models

**Concept: Three ways to describe one business system**

- A **conceptual model** describes important business things, such as Customer, Order, and Product.
- A **logical model** adds their attributes and relationships, such as Customer ID and Order Date.
- A **physical model** is how MySQL stores them, using column data types, primary keys, and foreign keys.

**Task:** Identify the four entities. Inspect the columns in each table. Find the MySQL data types used for the order item.

**Solution**

```sql
USE tg_data_modeling_s1;
SHOW TABLES;

DESCRIBE customers;
DESCRIBE products;
DESCRIBE orders;
DESCRIBE order_items;

SHOW CREATE TABLE order_items;
```

**Expected observation:** `order_items` has `order_item_id`, `order_id`, `product_id`, `quantity`, and `unit_price_at_sale`. The last column uses `DECIMAL(10,2)`, which stores prices with two decimal places.

**What you learned:** The *same* business system can be explained at a business level, a table-design level, or a MySQL implementation level.

## Exercise 3 — Find keys and relationships

**Concept: Primary key and foreign key**

A **primary key** uniquely identifies a row. A **foreign key** connects one table to another. For example, `orders.customer_id` points to a customer in `customers`.

**Task:** Use MySQL metadata to see the foreign-key relationships. Then find all orders placed by customer 1.

**Solution**

```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = DATABASE()
  AND REFERENCED_TABLE_NAME IS NOT NULL
ORDER BY TABLE_NAME, COLUMN_NAME;

SELECT order_id, customer_id, order_date, order_status
FROM orders
WHERE customer_id = 1
ORDER BY order_id;
```

**Expected relationships**

| Child table and column | Parent table and column |
|---|---|
| `orders.customer_id` | `customers.customer_id` |
| `order_items.order_id` | `orders.order_id` |
| `order_items.product_id` | `products.product_id` |

Customer 1 has orders **1001** and **1003**.

**What you learned:** One customer can have many orders. The foreign key can appear in many order rows, while each `customer_id` in `customers` is unique.

## Exercise 4 — See how a foreign key protects order data

**Concept: Referential integrity**

An order should not belong to a customer who does not exist. MySQL's InnoDB foreign key can reject such a record.

**Task:** Check whether customer 999 exists. Then try inserting a temporary order for that customer. The INSERT is **supposed to fail**. Check that the invalid order was not saved.

**Solution — first check**

```sql
SELECT COUNT(*) AS matching_customers
FROM customers
WHERE customer_id = 999;
```

**Expected:** `0` customers.

**Solution — run this ONE statement separately; an error is expected**

```sql
INSERT INTO orders (order_id, customer_id, order_date, order_status)
VALUES (1999, 999, '2026-10-10', 'CONFIRMED');
```

**Expected:** MySQL error **1452**, because `customer_id = 999` does not exist. This error is the successful result of the exercise. If MySQL Workbench stops executing a script after an error, select and run the next check separately.

**Solution — final check**

```sql
SELECT COUNT(*) AS invalid_orders
FROM orders
WHERE order_id = 1999;
```

**Expected:** `0` invalid orders.

**What you learned:** A database constraint prevents some wrong data from being saved. It is not only an application-level check.

## Exercise 5 — Understand the grain of each table

**Concept: Grain means what one row represents**

- One `customers` row represents **one customer**.
- One `orders` row represents **one order**.
- One `order_items` row represents **one product line inside an order**.

**Task:** Count orders and order-item rows. Join them, then compare `COUNT(*)` with `COUNT(DISTINCT order_id)`.

**Solution**

```sql
SELECT COUNT(*) AS total_orders FROM orders;
SELECT COUNT(*) AS total_order_items FROM order_items;

SELECT
    COUNT(*) AS rows_after_join,
    COUNT(DISTINCT o.order_id) AS actual_orders
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id;

SELECT o.order_id, COUNT(*) AS items_in_order
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
GROUP BY o.order_id
ORDER BY o.order_id;
```

**Expected:** `15` orders, `30` order items, `30` rows after the join, and `15` distinct orders. Every order in this sample has two item lines.

**What you learned:** A join can increase the number of rows because the tables have different grains. **Thirty joined rows do not mean thirty orders.**

---

# Part 2 — Operational data model (OLTP)

**OLTP** stands for *Online Transaction Processing*. It is a workload used to run everyday business activities, such as looking up an order, placing an order, and updating an order. It often uses many small reads and writes. Data integrity matters.

## Exercise 6 — Retrieve one customer order

**Concept: A targeted transactional query**

**Task:** A customer asks for the details of order 1001. Show the customer, product names, quantities, selling prices, and line amounts.

**Solution**

```sql
SELECT
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price_at_sale,
    oi.quantity * oi.unit_price_at_sale AS line_amount
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
JOIN order_items AS oi ON oi.order_id = o.order_id
JOIN products AS p ON p.product_id = oi.product_id
WHERE o.order_id = 1001
ORDER BY oi.order_item_id;
```

**Expected**

| Order | Customer | Product | Quantity | Price | Amount |
|---|---|---|---:|---:|---:|
| 1001 | Riya | Keyboard | 1 | 1,500 | 1,500 |
| 1001 | Riya | Mouse | 2 | 700 | 1,400 |

Order total = **₹2,900**.

**What you learned:** A transactional query often retrieves a specific business record and the small group of rows related to it.

## Exercise 7 — Insert an order and undo the entire transaction

**Concept: Transaction and rollback**

A transaction can group changes that belong together. `ROLLBACK` reverses uncommitted changes made in that transaction. We will create a customer, an order, and two order items, then undo them.

**Task:** Perform the transaction below. Check the new order total before rollback, then check that the order has disappeared after rollback.

**Solution — run the entire block in sequence**

```sql
START TRANSACTION;

INSERT INTO customers (customer_id, customer_name, city)
VALUES (20, 'Neeraj', 'Ahmedabad');

INSERT INTO orders (order_id, customer_id, order_date, order_status)
VALUES (1100, 20, '2026-10-10', 'CONFIRMED');

INSERT INTO order_items
    (order_item_id, order_id, product_id, quantity, unit_price_at_sale)
VALUES
    (40, 1100, 107, 1, 2500.00),
    (41, 1100, 106, 2, 1800.00);

SELECT
    o.order_id,
    c.customer_name,
    SUM(oi.quantity * oi.unit_price_at_sale) AS order_total
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.order_id = 1100
GROUP BY o.order_id, c.customer_name;

ROLLBACK;

SELECT COUNT(*) AS orders_after_rollback
FROM orders
WHERE order_id = 1100;
```

**Expected:** Before `ROLLBACK`, the new order total is **₹6,100**. After `ROLLBACK`, `orders_after_rollback = 0`. The temporary customer and order items are also removed.

**What you learned:** Related writes can be handled together. This is an important OLTP requirement. **Do not run a DDL statement such as `CREATE TABLE` between `START TRANSACTION` and `ROLLBACK`.**

---

# Part 3 — Analytical data model (OLAP)

**OLAP** stands for *Online Analytical Processing*. It is a workload for answering questions such as *How much did we sell? Which city had the highest sales? Which products sold most?* Analytical queries often read and summarize many records.

## Exercise 8 — Calculate a business measure

**Concept: Measures need a clear definition**

For this exercise, **confirmed revenue** means the sum of `quantity × unit_price_at_sale` from confirmed orders. An order that is cancelled or pending must not be included.

**Task:** Find the confirmed order count, number of units sold, confirmed revenue, and average order value. Then find revenue by order status.

**Solution**

```sql
SELECT
    COUNT(DISTINCT o.order_id) AS confirmed_orders,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price_at_sale) AS confirmed_revenue,
    ROUND(
        SUM(oi.quantity * oi.unit_price_at_sale)
        / COUNT(DISTINCT o.order_id), 2
    ) AS average_order_value
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.order_status = 'CONFIRMED';

SELECT
    o.order_status,
    COUNT(DISTINCT o.order_id) AS order_count,
    SUM(oi.quantity * oi.unit_price_at_sale) AS amount
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
GROUP BY o.order_status
ORDER BY o.order_status;
```

**Expected confirmed metrics**

| Confirmed orders | Units sold | Revenue | Average order value |
|---:|---:|---:|---:|
| 13 | 35 | ₹106,000.00 | ₹8,153.85 |

**Expected amounts by status:** `CANCELLED = ₹3,700`, `CONFIRMED = ₹106,000`, `PENDING = ₹5,500`. Only the confirmed amount is counted as revenue in this workbook.

**What you learned:** A useful analytical measure is more than a `SUM()`. You must define which business records qualify.

## Exercise 9 — Find an analytical counting mistake

**Concept: The effect of a one-to-many join on counts**

**Task:** Compare a wrong order count with a correct order count using the same confirmed orders.

**Solution**

```sql
SELECT
    COUNT(*) AS incorrect_order_count,
    COUNT(DISTINCT o.order_id) AS correct_order_count
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.order_status = 'CONFIRMED';
```

**Expected:** Incorrect count = **26**. Correct count = **13**.

**What you learned:** Always decide whether you are counting orders, order lines, products, or customers before writing an aggregate query.

## Exercise 10 — Report sales by city

**Concept: A dimension is a field used to group or filter measures**

Here, `city` is a descriptive attribute. `confirmed_revenue` is a measure.

**Task:** Find confirmed revenue by customer city, highest first.

**Solution**

```sql
SELECT
    c.city,
    SUM(oi.quantity * oi.unit_price_at_sale) AS confirmed_revenue
FROM customers AS c
JOIN orders AS o ON o.customer_id = c.customer_id
JOIN order_items AS oi ON oi.order_id = o.order_id
WHERE o.order_status = 'CONFIRMED'
GROUP BY c.city
ORDER BY confirmed_revenue DESC, c.city;
```

**Expected**

| City | Confirmed revenue |
|---|---:|
| Mumbai | ₹30,300 |
| Delhi | ₹18,800 |
| Pune | ₹17,600 |
| Hyderabad | ₹14,000 |
| Jaipur | ₹13,100 |
| Chennai | ₹8,100 |
| Bengaluru | ₹4,100 |

**Check:** The city totals add up to **₹106,000**. Kochi does not appear because Tara has no orders.

**What you learned:** An analytical query groups many transactions into a small result that answers a business question.

## Exercise 11 — Show products with no sales

**Concept: An inner join and a left join answer different questions**

An inner join returns matching rows. A left join can also show records that have no match.

**Task:** Find products that have not appeared in any order. Also find customers who have not placed an order.

**Solution**

```sql
SELECT p.product_id, p.product_name
FROM products AS p
LEFT JOIN order_items AS oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL
ORDER BY p.product_id;

SELECT c.customer_id, c.customer_name, c.city
FROM customers AS c
LEFT JOIN orders AS o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL
ORDER BY c.customer_id;
```

**Expected:** Product **111 / Router** has no order lines. Customer **10 / Tara / Kochi** has no orders.

**What you learned:** Reporting sometimes needs to include **zero activity**, not just successful matches. Choose the join based on the business question.

## Exercise 12 — Report sales by product category

**Concept: Analytical grouping and aggregation**

**Task:** Find the number of units sold and confirmed revenue for each product category.

**Solution**

```sql
SELECT
    p.category,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price_at_sale) AS confirmed_revenue
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
JOIN products AS p ON p.product_id = oi.product_id
WHERE o.order_status = 'CONFIRMED'
GROUP BY p.category
ORDER BY confirmed_revenue DESC, p.category;
```

**Expected**

| Category | Units | Confirmed revenue |
|---|---:|---:|
| Displays | 3 | ₹35,800 |
| Accessories | 20 | ₹26,600 |
| Cameras | 5 | ₹17,500 |
| Audio | 5 | ₹15,300 |
| Storage | 2 | ₹10,800 |

**Check:** The revenue adds up to **₹106,000**. Networking does not appear because its only product has no sales.

**What you learned:** Categories and cities are examples of dimensions. Quantities and amounts are examples of measures.

## Exercise 13 — Compare current price with historical selling price

**Concept: Two prices can represent different facts**

A product has a *current catalog price*, but an order line records the *price the customer paid at that time*. These two values do not need to be equal.

**Task:** Find confirmed order lines where the historical selling price differs from the current catalog price. Then calculate revenue in the correct and incorrect ways.

**Solution**

```sql
SELECT
    oi.order_id,
    p.product_name,
    oi.quantity,
    oi.unit_price_at_sale,
    p.current_price
FROM order_items AS oi
JOIN orders AS o ON o.order_id = oi.order_id
JOIN products AS p ON p.product_id = oi.product_id
WHERE o.order_status = 'CONFIRMED'
  AND oi.unit_price_at_sale <> p.current_price
ORDER BY oi.order_id, p.product_name;

SELECT
    SUM(oi.quantity * oi.unit_price_at_sale) AS correct_revenue,
    SUM(oi.quantity * p.current_price) AS wrong_revenue
FROM order_items AS oi
JOIN orders AS o ON o.order_id = oi.order_id
JOIN products AS p ON p.product_id = oi.product_id
WHERE o.order_status = 'CONFIRMED';
```

**Expected:** Five order lines have different prices (orders **1003, 1005, 1008, and 1014**, with two different product lines in order 1014). Correct revenue = **₹106,000**. Revenue incorrectly calculated using today's prices = **₹106,800**.

**What you learned:** Repeating a number is not always a mistake. `unit_price_at_sale` and `current_price` describe **different business facts**.

---

# Part 4 — Denormalized reporting data

An operational model often keeps business entities in separate tables. An analytical team may need one flat dataset that already contains customer, product, and order details. **Denormalization** intentionally brings related data together, usually to simplify reading or reporting. It can create extra copies of names and locations that must be kept consistent.

## Exercise 14 — Create a joined reporting view

**Concept: A view is a saved SQL query, not a stored copy of all its result rows**

**Task:** Create `v_sales_detail` with one row per order item, including the customer name, city, product, category, quantity, price, and calculated amount. Count the rows and revenue.

**Solution**

```sql
CREATE OR REPLACE VIEW v_sales_detail AS
SELECT
    oi.order_item_id,
    o.order_id,
    o.order_date,
    o.order_status,
    c.customer_id,
    c.customer_name,
    c.city AS customer_city,
    p.product_id,
    p.product_name,
    p.category,
    oi.quantity,
    oi.unit_price_at_sale,
    oi.quantity * oi.unit_price_at_sale AS line_amount
FROM order_items AS oi
JOIN orders AS o ON o.order_id = oi.order_id
JOIN customers AS c ON c.customer_id = o.customer_id
JOIN products AS p ON p.product_id = oi.product_id;

SELECT order_item_id, order_id, customer_name, product_name, line_amount
FROM v_sales_detail
ORDER BY order_item_id
LIMIT 5;

SELECT
    COUNT(*) AS view_rows,
    SUM(CASE WHEN order_status = 'CONFIRMED'
             THEN line_amount ELSE 0 END) AS confirmed_revenue
FROM v_sales_detail;
```

**Expected:** The view contains **30** rows. Confirmed revenue = **₹106,000**.

**What you learned:** The view exposes a flat-looking table to the reader, but MySQL gets its data from the underlying tables whenever the view is queried. A regular MySQL view is **not** a materialized view.

## Exercise 15 — Create a physical denormalized table

**Concept: A stored flat table duplicates descriptive values**

**Task:** Copy the view into a real table named `sales_flat`. Count the rows and see how often Riya's name appears.

**Solution**

```sql
CREATE TABLE sales_flat ENGINE=InnoDB AS
SELECT * FROM v_sales_detail;

SELECT COUNT(*) AS flat_rows FROM sales_flat;

SELECT customer_id, customer_name, COUNT(*) AS repeated_rows
FROM sales_flat
WHERE customer_id = 1
GROUP BY customer_id, customer_name;

-- First check: the copied table has no indexes yet.
SHOW INDEX FROM sales_flat;

-- Add a key so individual rows can be updated safely in MySQL Workbench.
ALTER TABLE sales_flat ADD PRIMARY KEY (order_item_id);

-- Check again: the PRIMARY index is now present.
SHOW INDEX FROM sales_flat;
```

**Expected:** `sales_flat` contains **30** rows. Riya's name appears in **4** rows. The first `SHOW INDEX` returns no rows because `CREATE TABLE ... SELECT` does not automatically create indexes. After the `ALTER TABLE`, the second `SHOW INDEX` displays `PRIMARY` on `order_item_id`.

**What you learned:** A stored flat table is easy to query but contains repeated descriptions. It is a **snapshot** of the source at the time it was created, not a live copy that updates by itself. Adding a primary key also allows Exercise 16 to work with MySQL Workbench's usual Safe Updates setting.

## Exercise 16 — Demonstrate an inconsistent update

**Concept: Update anomaly caused by independently editable copies**

In this dataset, `customer_city` is copied from the customer's **current profile**, not the delivery city at the time of each order. If a separate column recorded the historical delivery city, different values across orders could be correct.

**Task:** Temporarily change the current profile city for just one of Riya's rows in `sales_flat`. Group Riya's sales by city. Then roll back the change.

**Solution**

```sql
START TRANSACTION;

UPDATE sales_flat
SET customer_city = 'Nashik'
WHERE order_item_id = 1;

SELECT
    customer_id,
    customer_name,
    customer_city,
    SUM(line_amount) AS confirmed_revenue
FROM sales_flat
WHERE customer_id = 1
  AND order_status = 'CONFIRMED'
GROUP BY customer_id, customer_name, customer_city
ORDER BY customer_city;

ROLLBACK;

SELECT COUNT(DISTINCT customer_city) AS cities_after_rollback
FROM sales_flat
WHERE customer_id = 1;
```

**Expected before rollback**

| Customer | City | Revenue |
|---|---|---:|
| Riya | Nashik | ₹1,500 |
| Riya | Pune | ₹6,350 |

After rollback, `cities_after_rollback = 1` (Pune).

**What you learned:** If one customer's city is stored repeatedly and only one copy changes, reports become inconsistent. A flat reporting table needs controlled loading and updating.

## Exercise 17 — Compare a live view with a stored snapshot

**Concept: Data freshness**

**Task:** Temporarily rename product 101 in the `products` table. Compare what the joined view shows with what the earlier snapshot shows. Then roll back.

**Solution**

```sql
START TRANSACTION;

UPDATE products
SET product_name = 'Mechanical Keyboard'
WHERE product_id = 101;

SELECT
    v.order_item_id,
    v.product_name AS name_from_live_view,
    f.product_name AS name_from_saved_snapshot
FROM v_sales_detail AS v
JOIN sales_flat AS f ON f.order_item_id = v.order_item_id
WHERE v.order_item_id = 1;

ROLLBACK;
```

**Expected before rollback:** `name_from_live_view = Mechanical Keyboard` and `name_from_saved_snapshot = Keyboard`.

**What you learned:** A view shows current source values. A stored reporting table needs a refresh or load process to receive later changes.

---

# Part 5 — Build a small analytical star schema

A **fact table** stores measurements, such as quantity and sales amount. A **dimension table** stores descriptions used for filtering and grouping, such as customer city, product category, and date. These tables can form a **star schema**.

For this example, one fact row represents **one order item**. This is the **fact grain**.

```text
                 dim_customer
                      |
 dim_product ---- fact_sales ---- dim_date

Each row in fact_sales = one order item.
```

## Exercise 18 — Create dimensions and a sales fact table

**Concept: The same source can be organized for analytical use**

**Task:** Build three dimension tables and one fact table using our existing operational data. Check the row counts.

**Solution**

```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_name VARCHAR(80) NOT NULL,
    customer_city VARCHAR(60) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_name VARCHAR(80) NOT NULL,
    category VARCHAR(50) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE NOT NULL UNIQUE,
    calendar_year INT NOT NULL,
    calendar_month INT NOT NULL
) ENGINE=InnoDB;

INSERT INTO dim_customer (customer_key, customer_name, customer_city)
SELECT customer_id, customer_name, city
FROM customers;

INSERT INTO dim_product (product_key, product_name, category)
SELECT product_id, product_name, category
FROM products;

INSERT INTO dim_date (date_key, full_date, calendar_year, calendar_month)
SELECT DISTINCT
    CAST(DATE_FORMAT(order_date, '%Y%m%d') AS UNSIGNED),
    order_date,
    YEAR(order_date),
    MONTH(order_date)
FROM orders;

CREATE TABLE fact_sales (
    order_item_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    customer_key INT NOT NULL,
    product_key INT NOT NULL,
    date_key INT NOT NULL,
    order_status VARCHAR(12) NOT NULL,
    quantity INT NOT NULL,
    line_amount DECIMAL(12,2) NOT NULL,
    CONSTRAINT fk_s1_fact_customer
        FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key),
    CONSTRAINT fk_s1_fact_product
        FOREIGN KEY (product_key) REFERENCES dim_product(product_key),
    CONSTRAINT fk_s1_fact_date
        FOREIGN KEY (date_key) REFERENCES dim_date(date_key)
) ENGINE=InnoDB;

INSERT INTO fact_sales
    (order_item_id, order_id, customer_key, product_key,
     date_key, order_status, quantity, line_amount)
SELECT
    oi.order_item_id,
    o.order_id,
    o.customer_id,
    oi.product_id,
    CAST(DATE_FORMAT(o.order_date, '%Y%m%d') AS UNSIGNED),
    o.order_status,
    oi.quantity,
    oi.quantity * oi.unit_price_at_sale
FROM order_items AS oi
JOIN orders AS o ON o.order_id = oi.order_id;

SELECT 'dim_customer' AS table_name, COUNT(*) AS row_count FROM dim_customer
UNION ALL SELECT 'dim_product', COUNT(*) FROM dim_product
UNION ALL SELECT 'dim_date', COUNT(*) FROM dim_date
UNION ALL SELECT 'fact_sales', COUNT(*) FROM fact_sales;
```

**Expected row counts**

| Table | Rows |
|---|---:|
| `dim_customer` | 10 |
| `dim_product` | 11 |
| `dim_date` | 9 |
| `fact_sales` | 30 |

**What you learned:** The fact stores the transaction measurement. Dimensions supply descriptive information. The example uses source IDs as dimension keys to keep the exercise simple; larger warehouses often use separate surrogate keys and track changes to dimension values.

## Exercise 19 — Reconcile star-schema revenue with the source

**Concept: Report totals must agree across models**

**Task:** Find confirmed revenue by category from the star schema, then compare total confirmed revenue from the operational tables and the fact table.

**Solution**

```sql
SELECT
    p.category,
    SUM(f.line_amount) AS confirmed_revenue
FROM fact_sales AS f
JOIN dim_product AS p ON p.product_key = f.product_key
WHERE f.order_status = 'CONFIRMED'
GROUP BY p.category
ORDER BY confirmed_revenue DESC, p.category;

SELECT
    (SELECT SUM(oi.quantity * oi.unit_price_at_sale)
     FROM orders AS o
     JOIN order_items AS oi ON oi.order_id = o.order_id
     WHERE o.order_status = 'CONFIRMED') AS source_revenue,
    (SELECT SUM(line_amount)
     FROM fact_sales
     WHERE order_status = 'CONFIRMED') AS star_revenue;
```

**Expected:** The category totals match Exercise 12. `source_revenue = 106000.00` and `star_revenue = 106000.00`.

**What you learned:** Different models may organize the data differently, but the same business measure should still produce the same result.

## Exercise 20 — Use a date dimension for daily sales

**Concept: Slice a measure by time**

**Task:** Calculate confirmed revenue per day using `fact_sales` and `dim_date`.

**Solution**

```sql
SELECT
    d.full_date,
    SUM(f.line_amount) AS confirmed_revenue
FROM fact_sales AS f
JOIN dim_date AS d ON d.date_key = f.date_key
WHERE f.order_status = 'CONFIRMED'
GROUP BY d.full_date
ORDER BY d.full_date;
```

**Expected**

| Date | Confirmed revenue |
|---|---:|
| 2026-10-01 | ₹2,900 |
| 2026-10-02 | ₹19,250 |
| 2026-10-03 | ₹23,100 |
| 2026-10-05 | ₹9,550 |
| 2026-10-06 | ₹10,600 |
| 2026-10-07 | ₹24,100 |
| 2026-10-08 | ₹10,100 |
| 2026-10-09 | ₹6,400 |

**Check:** These amounts total **₹106,000**. October 4 has only a cancelled order, so it does not appear in this confirmed-sales report. If you need to show dates with zero sales, start from `dim_date` with a `LEFT JOIN` and apply the status condition inside the join, not in the `WHERE` clause.

**What you learned:** Date, product, and customer dimensions help organize analytical questions. The fact table determines what is being measured.

---

# Part 6 — Choose the model for the problem

## Exercise 21 — Compare four ways to read the same business data

**Concept: Data models serve different purposes**

**Task:** Run one query for each requirement. Note which source is most convenient.

**Solution**

```sql
-- A. Customer support: look up one operational order.
SELECT o.order_id, c.customer_name, o.order_status
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
WHERE o.order_id = 1001;

-- B. Dashboard: summarize current data from a joined view.
SELECT customer_city, SUM(line_amount) AS revenue
FROM v_sales_detail
WHERE order_status = 'CONFIRMED'
GROUP BY customer_city
ORDER BY revenue DESC;

-- C. Stored reporting copy: no joins needed for these fields.
SELECT order_item_id, customer_name, product_name, line_amount
FROM sales_flat
ORDER BY order_item_id
LIMIT 5;

-- D. Dimensional model: summarize a measure by category.
SELECT p.category, SUM(f.line_amount) AS revenue
FROM fact_sales AS f
JOIN dim_product AS p ON p.product_key = f.product_key
WHERE f.order_status = 'CONFIRMED'
GROUP BY p.category
ORDER BY revenue DESC;
```

**Expected:** Query A returns one order; Query B produces the city totals from Exercise 10; Query C displays five stored sales lines; Query D produces the category totals from Exercise 12.

**What you learned:** One system can use an operational model, a live joined view, a stored flat table, and a star schema. The best choice depends on how the data will be used. **A flat table is not automatically faster**, and a normalized operational model is not always the easiest way to write analytical reports.

---

# Session 1 recap

| Idea | Evidence from the exercises |
|---|---|
| Tables represent related business entities. | 1–3: customers, products, orders, order items |
| Keys connect tables and protect valid references. | 3–4: foreign keys and rejected order |
| A row has a specific grain. | 5 and 9: 15 orders, 30 lines; 13 confirmed orders, 26 confirmed lines |
| OLTP supports everyday records and transactions. | 6–7: order lookup and rollback |
| OLAP answers grouped business questions. | 8, 10–12: revenue, cities, categories |
| Historical selling prices differ from today's prices. | 13: ₹106,000 versus ₹106,800 |
| Views and physical copies are different. | 14–15: joined view versus saved flat table |
| Repeated descriptions can become inconsistent. | 16: Riya shown in two cities |
| A stored copy can be older than its source. | 17: different product names |
| Facts and dimensions organize analytical work. | 18–20: star-schema load, reconciliation, daily report |
| Model choice depends on the workload. | 21: four reporting and operational queries |

## Final checks

If you completed the entire workbook **in order**, and ran every `ROLLBACK` shown, run:

```sql
SELECT COUNT(*) AS customer_rows FROM customers;
SELECT COUNT(*) AS order_rows FROM orders;
SELECT COUNT(*) AS order_item_rows FROM order_items;
SELECT COUNT(*) AS flat_rows FROM sales_flat;
SELECT COUNT(*) AS fact_rows FROM fact_sales;

SELECT SUM(line_amount) AS confirmed_revenue
FROM fact_sales
WHERE order_status = 'CONFIRMED';
```

**Expected:** 10 customers, 15 orders, 30 order items, 30 flat rows, 30 fact rows, and confirmed revenue of **₹106,000.00**.

**Next session:** Start with a deliberately poorly structured order table, see problems caused by lists of values and repeating groups, then work toward **First Normal Form (1NF)**.

## MySQL 8 documentation

- [Foreign keys](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)
- [Statements causing implicit commits](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)
- [Views](https://dev.mysql.com/doc/refman/8.0/en/views.html)
- [CREATE TABLE ... SELECT](https://dev.mysql.com/doc/refman/8.0/en/create-table-select.html)
