# Data Modeling — Denormalized Data, 1NF and 2NF

**Database:** MySQL 8  
**Scenario:** An online store receives orders containing one or more products.  
**Goal:** Find problems in a table design, reproduce them using SQL, and fix them step by step.

Run the exercises **in order**. Each exercise includes the SQL solution and the result to check. Use MySQL Workbench or another MySQL 8 client.

> **Practice database:** The setup below creates `nf12_practice` and resets only the practice tables named in this workbook. Do not run it inside a production database. Run the statements in order, using the **same MySQL connection**. In each example, run `START TRANSACTION`, then the change, then the `SELECT`, and finally `ROLLBACK`. Do not run `CREATE TABLE`, `DROP TABLE`, or `ALTER TABLE` inside these temporary transactions. Table setup and sample-data inserts are meant to remain available for later exercises.

**Version check (optional):** `SELECT VERSION();` should show your MySQL server version.

---

## Exercise 1 — What is wrong with one large order table?

### Concept

A **denormalized table** may store several kinds of information together. This can be convenient for reading a complete order, but it may also create duplication and make updates difficult.

The first example has an additional problem: it stores **multiple product IDs and quantities in single cells**. This is an *unnormalized* design, not a requirement of denormalization. A denormalized table can still follow 1NF.

### Task

Create a table that stores each order on one row, with comma-separated product IDs and quantities. Display its data.

### SQL solution

```sql
CREATE DATABASE IF NOT EXISTS nf12_practice;
USE nf12_practice;

-- Reset these practice tables if the workbook is run again.
DROP TABLE IF EXISTS order_items_2nf;
DROP TABLE IF EXISTS orders_2nf;
DROP TABLE IF EXISTS products_2nf;
DROP TABLE IF EXISTS order_lines_1nf;
DROP TABLE IF EXISTS raw_orders;

CREATE TABLE raw_orders (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,
    customer_name VARCHAR(40) NOT NULL,
    product_ids VARCHAR(60) NOT NULL,
    quantities VARCHAR(60) NOT NULL
) ENGINE = InnoDB;

INSERT INTO raw_orders
    (order_id, order_date, customer_id, customer_name, product_ids, quantities)
VALUES
    (1001, '2026-10-01', 1, 'Riya',  '101,102,103', '1,2,1'),
    (1002, '2026-10-02', 2, 'Amit',  '102,104',     '1,1'),
    (1003, '2026-10-03', 1, 'Riya',  '101,103',     '2,3'),
    (1004, '2026-10-04', 3, 'Neha',  '104,102',     '2,1'),
    (1005, '2026-10-05', 4, 'Rahul', '103,101',     '2,1'),
    (1006, '2026-10-06', 2, 'Amit',  '102,103',     '2,1'),
    (1007, '2026-10-07', 3, 'Neha',  '101,104',     '1,1'),
    (1008, '2026-10-08', 4, 'Rahul', '105',         '1');

SELECT order_id, customer_name, product_ids, quantities
FROM raw_orders
ORDER BY order_id;
```

### Result to check

There are **8 orders**. Order `1001` contains three products (`101,102,103`) and three corresponding quantities (`1,2,1`). The first quantity belongs to the first product, the second to the second product, and so on.

**Problem:** The connection between a product and its quantity depends on the *position* of a value in a text field. The database cannot easily enforce that the two lists match.

---

## Exercise 2 — Experience the limitations of the raw table

### Concept

In an operational system, we frequently need to find a product, change one quantity, or validate one order item. Comma-separated lists make these tasks harder.

### Task A — Find all orders containing product `102`

### SQL solution

```sql
SELECT order_id, product_ids
FROM raw_orders
WHERE FIND_IN_SET('102', product_ids) > 0
ORDER BY order_id;
```

### Result to check

The order IDs are **1001, 1002, 1004, 1006**. We need a special string function because products are not stored as separate rows.

### Task B — Show how the product and quantity lists can become inconsistent

### SQL solution

```sql
START TRANSACTION;

UPDATE raw_orders
SET quantities = '1,2'
WHERE order_id = 1001;

SELECT order_id, product_ids, quantities
FROM raw_orders
WHERE order_id = 1001;

ROLLBACK;
```

### Result to check

Before `ROLLBACK`, order `1001` has **3 product IDs but only 2 quantities**. MySQL accepts this update because both columns contain ordinary text. After `ROLLBACK`, the original quantities return to `1,2,1`.

**Limitations found:** Difficult item-level updates, awkward searches, and no direct way for a foreign key to validate each product inside a comma-separated list.

---

## Exercise 3 — Apply First Normal Form (1NF)

### Concept

**1NF:** Store one value in each cell, avoid repeating groups, and make each row uniquely identifiable. In this order example, each row represents **one product in one order**.

An order can have several rows. The pair `(order_id, product_id)` is the **composite primary key** because the same product appears at most once per order. The `quantity` column tells us how many units were ordered.

### Task

Create an order-line table with one product per row. Use the same 8 orders and 16 order items.

### SQL solution

```sql
CREATE TABLE order_lines_1nf (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,
    customer_name VARCHAR(40) NOT NULL,
    product_name VARCHAR(40) NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)
) ENGINE = InnoDB;

INSERT INTO order_lines_1nf
    (order_id, product_id, order_date, customer_id,
     customer_name, product_name, quantity)
VALUES
    (1001, 101, '2026-10-01', 1, 'Riya',  'Keyboard',     1),
    (1001, 102, '2026-10-01', 1, 'Riya',  'Mouse',        2),
    (1001, 103, '2026-10-01', 1, 'Riya',  'USB Cable',    1),
    (1002, 102, '2026-10-02', 2, 'Amit',  'Mouse',        1),
    (1002, 104, '2026-10-02', 2, 'Amit',  'Laptop Stand', 1),
    (1003, 101, '2026-10-03', 1, 'Riya',  'Keyboard',     2),
    (1003, 103, '2026-10-03', 1, 'Riya',  'USB Cable',    3),
    (1004, 104, '2026-10-04', 3, 'Neha',  'Laptop Stand', 2),
    (1004, 102, '2026-10-04', 3, 'Neha',  'Mouse',        1),
    (1005, 103, '2026-10-05', 4, 'Rahul', 'USB Cable',    2),
    (1005, 101, '2026-10-05', 4, 'Rahul', 'Keyboard',     1),
    (1006, 102, '2026-10-06', 2, 'Amit',  'Mouse',        2),
    (1006, 103, '2026-10-06', 2, 'Amit',  'USB Cable',    1),
    (1007, 101, '2026-10-07', 3, 'Neha',  'Keyboard',     1),
    (1007, 104, '2026-10-07', 3, 'Neha',  'Laptop Stand', 1),
    (1008, 105, '2026-10-08', 4, 'Rahul', 'Monitor',      1);

SELECT order_id, product_id, product_name, quantity
FROM order_lines_1nf
ORDER BY order_id, product_id;
```

### Result to check

There are **16 rows**. Order `1001` now has three rows. Every `product_id` has its own `quantity` value.

---

## Exercise 4 — See how 1NF solves item-level problems

### Task A — Find orders containing product `102`

### SQL solution

```sql
SELECT order_id, product_id, quantity
FROM order_lines_1nf
WHERE product_id = 102
ORDER BY order_id;
```

### Result to check

| Order | Product | Quantity |
|---|---|---|
| 1001 | 102 | 2 |
| 1002 | 102 | 1 |
| 1004 | 102 | 1 |
| 1006 | 102 | 2 |

No comma-separated text needs to be parsed.

### Task B — Update one item without changing other items

### SQL solution

```sql
START TRANSACTION;

UPDATE order_lines_1nf
SET quantity = 5
WHERE order_id = 1001 AND product_id = 102;

SELECT order_id, product_id, quantity
FROM order_lines_1nf
WHERE order_id = 1001
ORDER BY product_id;

ROLLBACK;
```

### Result to check

Before the rollback, order `1001` has quantities **1, 5, 1** for products `101`, `102`, and `103`. Only the chosen order item changed. The rollback restores the original quantity of `2` for product `102`.

**What 1NF solved:** Direct item searches, direct item updates, and unambiguous quantity-to-product relationships. **What it did not solve:** Repeated product names, order dates, and customer details.

---

## Exercise 5 — Find the problem that remains after 1NF

### Concept

The `order_lines_1nf` table follows 1NF, but it is still **denormalized**. In this practice database, `product_name` means the **current catalog name**, not the historical name at the time of purchase. For example, the current product name `Mouse` is repeated in multiple orders.

This creates an **update anomaly**: changing a product name in one row does not automatically change it everywhere.

### Task A — Make the same product appear with two names

### SQL solution

```sql
START TRANSACTION;

UPDATE order_lines_1nf
SET product_name = 'Wireless Mouse'
WHERE order_id = 1001 AND product_id = 102;

SELECT order_id, product_id, product_name
FROM order_lines_1nf
WHERE product_id = 102
ORDER BY order_id;

ROLLBACK;
```

### Result to check

Before the rollback, order `1001` shows **Wireless Mouse**, while orders `1002`, `1004`, and `1006` still show **Mouse** for the same product ID. The rollback restores the data.

### Task B — See a deletion anomaly

Product `105` (Monitor) appears only in order `1008`. Delete that order item temporarily and check whether its product name remains anywhere.

### SQL solution

```sql
START TRANSACTION;

DELETE FROM order_lines_1nf
WHERE order_id = 1008 AND product_id = 105;

SELECT COUNT(*) AS monitor_rows_left
FROM order_lines_1nf
WHERE product_id = 105;

ROLLBACK;
```

### Result to check

`monitor_rows_left = 0` before rollback. The only stored product name for product `105` has disappeared along with its order item.

### Task C — See an insertion anomaly

A new product, `Webcam` (`106`), has not been ordered yet. Try to register it in this 1NF design without a real order. The table requires order details, so the only way to insert it is to create a made-up order item.

### SQL solution

```sql
START TRANSACTION;

INSERT INTO order_lines_1nf
    (order_id, product_id, order_date, customer_id,
     customer_name, product_name, quantity)
VALUES
    (1009, 106, '2026-10-09', 4, 'Rahul', 'Webcam', 1);

SELECT order_id, customer_id, product_id, product_name, quantity
FROM order_lines_1nf
WHERE product_id = 106;

ROLLBACK;
```

### Result to check

`Webcam` appears, but only because we invented order `1009` and a quantity of `1`. We cannot keep a product catalog separately from orders in this table. After `ROLLBACK`, the made-up order item disappears. **This is the insertion anomaly.**

---

## Exercise 6 — Understand the composite key and partial dependencies

### Concept

A **composite key** uses more than one column to identify a row.

In `order_lines_1nf`, the key is `(order_id, product_id)`:

- `quantity` depends on **both** `order_id` and `product_id`.
- `order_date`, `customer_id`, and `customer_name` depend on **order_id alone**.
- `product_name` depends on **product_id alone**.

A **partial dependency** means a non-key column depends on only part of a composite candidate key.

**2NF:** First satisfy 1NF, then remove partial dependencies. Every non-key attribute must depend on the **whole of every candidate key**, not just part of it.

### Task

Use SQL to observe repeated order and product information.

### SQL solution

```sql
SELECT
    COUNT(*) AS order_line_rows,
    COUNT(DISTINCT order_id) AS different_orders,
    COUNT(DISTINCT product_id) AS different_products
FROM order_lines_1nf;

SELECT order_id, order_date, customer_name
FROM order_lines_1nf
WHERE order_id = 1001
ORDER BY product_id;

SELECT product_id, product_name
FROM order_lines_1nf
WHERE product_id = 102
ORDER BY order_id;
```

### Result to check

The first query shows **16 order lines, 8 orders, and 5 products**. The next two queries show the same order details repeated for every item and the same product name repeated across orders.

**Why this matters:** We are storing order-level and product-level facts again and again inside an order-item table.

---

## Exercise 7 — Apply Second Normal Form (2NF)

### Concept

Separate facts according to what identifies them:

- **Orders:** one row per order; identified by `order_id`.
- **Products:** one row per product; identified by `product_id`.
- **Order Items:** one row per product within an order; identified by `(order_id, product_id)`.

The order-item table stores only information that belongs to the entire order-product combination: here, `quantity`.

### Task

Create the three tables and move the data from the 1NF table.

First, confirm that no temporary changes remain from the previous exercises. The query below should return **zero rows**. If it returns any rows, run Exercise 1 again and repeat the exercises in order, including every `ROLLBACK`.

```sql
SELECT product_id, COUNT(DISTINCT product_name) AS different_names
FROM order_lines_1nf
GROUP BY product_id
HAVING COUNT(DISTINCT product_name) > 1;
```

### SQL solution

```sql
CREATE TABLE orders_2nf (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,
    customer_name VARCHAR(40) NOT NULL
) ENGINE = InnoDB;

CREATE TABLE products_2nf (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(40) NOT NULL
) ENGINE = InnoDB;

CREATE TABLE order_items_2nf (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders_2nf(order_id),
    FOREIGN KEY (product_id) REFERENCES products_2nf(product_id)
) ENGINE = InnoDB;

INSERT INTO orders_2nf (order_id, order_date, customer_id, customer_name)
SELECT DISTINCT order_id, order_date, customer_id, customer_name
FROM order_lines_1nf;

INSERT INTO products_2nf (product_id, product_name)
SELECT DISTINCT product_id, product_name
FROM order_lines_1nf;

INSERT INTO order_items_2nf (order_id, product_id, quantity)
SELECT order_id, product_id, quantity
FROM order_lines_1nf;

SELECT (SELECT COUNT(*) FROM orders_2nf) AS orders,
       (SELECT COUNT(*) FROM products_2nf) AS products,
       (SELECT COUNT(*) FROM order_items_2nf) AS order_items;
```

### Result to check

`orders = 8`, `products = 5`, `order_items = 16`.

The product name is now stored **once** in `products_2nf`; order dates are stored **once per order** in `orders_2nf`.

---

## Exercise 8 — Confirm that 2NF solves the earlier anomalies

### Task A — Update the product name once

### SQL solution

```sql
START TRANSACTION;

UPDATE products_2nf
SET product_name = 'Wireless Mouse'
WHERE product_id = 102;

SELECT oi.order_id, p.product_name
FROM order_items_2nf AS oi
JOIN products_2nf AS p
    ON oi.product_id = p.product_id
WHERE p.product_id = 102
ORDER BY oi.order_id;

ROLLBACK;
```

### Result to check

All **4 order items** with product `102` show `Wireless Mouse` before the rollback. There is one product name to maintain, so no inconsistent copies appear in different orders.

### Task B — Add a new product without creating an order

### SQL solution

```sql
START TRANSACTION;

INSERT INTO products_2nf (product_id, product_name)
VALUES (106, 'Webcam');

SELECT product_id, product_name
FROM products_2nf
WHERE product_id = 106;

ROLLBACK;
```

### Result to check

Product `106` exists before rollback, even though no customer has ordered it. This avoids the earlier insertion anomaly.

### Task C — Delete an order item without deleting the product

### SQL solution

```sql
START TRANSACTION;

DELETE FROM order_items_2nf
WHERE order_id = 1008 AND product_id = 105;

SELECT product_id, product_name
FROM products_2nf
WHERE product_id = 105;

ROLLBACK;
```

### Result to check

Product `105` (Monitor) **still exists** in `products_2nf`. Removing its last order item no longer removes its product details.

### Task D — Rebuild the complete order view using a JOIN

### SQL solution

```sql
SELECT
    o.order_id,
    o.customer_name,
    p.product_name,
    oi.quantity
FROM orders_2nf AS o
JOIN order_items_2nf AS oi
    ON o.order_id = oi.order_id
JOIN products_2nf AS p
    ON oi.product_id = p.product_id
WHERE o.order_id = 1001
ORDER BY p.product_id;
```

### Result to check

| Order | Customer | Product | Quantity |
|---|---|---|---|
| 1001 | Riya | Keyboard | 1 |
| 1001 | Riya | Mouse | 2 |
| 1001 | Riya | USB Cable | 1 |

The data is stored in separate tables but can still be combined when needed.

---

## Summary

| Design | What it solves | What still needs attention |
|---|---|---|
| Raw table | Keeps an entire order in one row | Comma-separated values are hard to search, update, and validate |
| **1NF** | One product per row; easy item-level queries and updates | Repeated data can still cause insert, update, and delete anomalies |
| **2NF** | Removes partial dependencies; separates orders, products, and order items | Customer name is still repeated across orders |

**Final observation:** The 2NF design still stores `customer_name` in `orders_2nf`. If the same customer places multiple orders, that name appears multiple times. This is a separate issue to examine when studying **3NF**.

**Important distinction:** A functional dependency means the same determining value must give the same dependent value under the business rules. That rule alone does not stop SQL updates from creating inconsistent copies; normalization reduces that risk. Normalization is particularly useful for keeping operational data consistent. Deliberate denormalization can still be a valid choice for analytical systems. Also, 2NF does **not** require every table to have a composite key; a 1NF table with a single-column candidate key already meets the partial-dependency requirement.
