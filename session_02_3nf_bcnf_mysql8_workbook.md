# Data Modeling — Third Normal Form (3NF) and BCNF

**Database:** MySQL 8  
**Scenario:** An online store manages customers, products, orders, order items, and product specialists.

This workbook continues the e-commerce example from the 1NF and 2NF session. It also includes its own database setup, so it can be used independently.

## How to use this workbook

1. Run the exercises in order in **one MySQL Workbench connection**.
2. Execute SQL statements one by one so you can inspect each result.
3. For a temporary change, run `START TRANSACTION`, the change, the `SELECT`, and then `ROLLBACK`. Do not skip the rollback.
4. Run table creation statements **outside** transactions. In MySQL, `CREATE TABLE` and `DROP TABLE` cannot be undone using `ROLLBACK`.
5. Use only the practice database `nf34_practice`. The setup resets the practice tables in that database if they already exist.

---

## Part A — Continue from 2NF

### Exercise 1 — Prepare the e-commerce data

**Concept:** Our 2NF model separates orders, products, and order items. However, the `Orders` table still repeats customer names.

**Task:** Create the practice database, add the 2NF tables, and insert the sample data.

**SQL solution**

```sql
CREATE DATABASE IF NOT EXISTS nf34_practice;
USE nf34_practice;

-- Reset only the tables used by this workbook.
DROP TABLE IF EXISTS order_specialists_bcnf;
DROP TABLE IF EXISTS specialist_products_bcnf;
DROP TABLE IF EXISTS order_product_specialist_3nf;
DROP TABLE IF EXISTS order_items_3nf;
DROP TABLE IF EXISTS orders_3nf;
DROP TABLE IF EXISTS products_3nf;
DROP TABLE IF EXISTS customers_3nf;
DROP TABLE IF EXISTS order_items_2nf;
DROP TABLE IF EXISTS orders_2nf;
DROP TABLE IF EXISTS products_2nf;

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

INSERT INTO orders_2nf
    (order_id, order_date, customer_id, customer_name)
VALUES
    (1001, '2026-10-01', 1, 'Riya'),
    (1002, '2026-10-02', 2, 'Amit'),
    (1003, '2026-10-03', 1, 'Riya'),
    (1004, '2026-10-04', 3, 'Neha'),
    (1005, '2026-10-05', 4, 'Rahul'),
    (1006, '2026-10-06', 2, 'Amit'),
    (1007, '2026-10-07', 3, 'Neha'),
    (1008, '2026-10-08', 4, 'Rahul');

INSERT INTO products_2nf (product_id, product_name)
VALUES
    (101, 'Keyboard'),
    (102, 'Mouse'),
    (103, 'USB Cable'),
    (104, 'Laptop Stand'),
    (105, 'Monitor');

INSERT INTO order_items_2nf (order_id, product_id, quantity)
VALUES
    (1001, 101, 1), (1001, 102, 2), (1001, 103, 1),
    (1002, 102, 1), (1002, 104, 1),
    (1003, 101, 2), (1003, 103, 3),
    (1004, 102, 1), (1004, 104, 2),
    (1005, 101, 1), (1005, 103, 2),
    (1006, 102, 2), (1006, 103, 1),
    (1007, 101, 1), (1007, 104, 1),
    (1008, 105, 1);

SELECT
    (SELECT COUNT(*) FROM orders_2nf) AS orders,
    (SELECT COUNT(*) FROM products_2nf) AS products,
    (SELECT COUNT(*) FROM order_items_2nf) AS order_items;
```

**Expected result:** **8 orders, 5 products, 16 order items**.

---

## Part B — Third Normal Form (3NF)

**3NF in one line:** Remove transitive dependencies that make non-key details depend on other non-key details. Here, customer information should not be repeated in the orders table.

### Exercise 2 — Find a transitive dependency

**Concept:** In the `orders_2nf` table:

- `order_id → customer_id`: An order belongs to one customer.
- `customer_id → customer_name`: A customer ID determines that customer's current name.
- Therefore, `order_id → customer_id → customer_name` is a **transitive dependency**.

`customer_name` describes the customer, not the order. For this exercise, it is the customer's **current profile name**, not a historical name printed on an invoice.

**Task:** Find orders placed by customer 1. Check where the customer's name is stored.

**SQL solution**

```sql
SELECT order_id, customer_id, customer_name
FROM orders_2nf
WHERE customer_id = 1
ORDER BY order_id;
```

**Expected result:** Orders **1001** and **1003** both store the name **Riya**. The name is repeated because the same customer placed two orders.

### Exercise 3 — Demonstrate the update anomaly before 3NF

**Concept:** If we change the customer's name in one order, the other orders may show a different current name for that customer.

**Task:** Change Riya's name in just one order and check both orders.

**SQL solution**

```sql
START TRANSACTION;

UPDATE orders_2nf
SET customer_name = 'Riya Sharma'
WHERE order_id = 1001;

SELECT order_id, customer_id, customer_name
FROM orders_2nf
WHERE customer_id = 1
ORDER BY order_id;

ROLLBACK;
```

**Expected result before rollback:** Order **1001** shows *Riya Sharma*, but order **1003** still shows *Riya*. The rollback restores both rows to *Riya*.

### Exercise 4 — Demonstrate the insertion anomaly before 3NF

**Concept:** A customer can register without placing an order. But our current model has no separate place to store the customer.

**Task:** Register a new customer, Maya (customer 5), using the current table. Notice that you must also invent an order ID and order date.

**SQL solution**

```sql
START TRANSACTION;

INSERT INTO orders_2nf
    (order_id, order_date, customer_id, customer_name)
VALUES
    (1009, '2026-11-03', 5, 'Maya');

SELECT order_id, customer_id, customer_name
FROM orders_2nf
WHERE customer_id = 5;

ROLLBACK;
```

**Expected result before rollback:** Maya appears only because we created a record for order **1009**, even though she has not placed an order. The rollback removes this artificial order.

### Exercise 5 — Demonstrate the deletion anomaly before 3NF

**Concept:** If a customer's details exist only in orders, deleting their last orders also removes all copies of their profile information.

**Task:** Rahul (customer 4) has orders 1005 and 1008. Delete their order items and orders temporarily. Check whether Rahul's name is still available in `orders_2nf`.

**SQL solution**

```sql
START TRANSACTION;

-- Remove child records first because order_items_2nf has a foreign key.
DELETE FROM order_items_2nf
WHERE order_id IN (1005, 1008);

DELETE FROM orders_2nf
WHERE order_id IN (1005, 1008);

SELECT COUNT(*) AS rahul_order_rows
FROM orders_2nf
WHERE customer_id = 4;

ROLLBACK;
```

**Expected result before rollback:** `rahul_order_rows = 0`. The only stored customer-name records for Rahul have disappeared. The rollback restores the orders and order items.

**What we learned:** The table satisfies 2NF, yet update, insertion, and deletion anomalies remain because customer details are stored with orders.

### Exercise 6 — Convert the complete e-commerce model to 3NF

**Concept:** Keep each fact where it belongs.

| Table | What belongs here? | Primary key |
|---|---|---|
| `customers_3nf` | Current customer details | `customer_id` |
| `orders_3nf` | Order date and customer ID | `order_id` |
| `products_3nf` | Product details | `product_id` |
| `order_items_3nf` | Which products and quantities belong to each order | `(order_id, product_id)` |

**Task:** First check that the earlier updates were rolled back. Then create all four tables and copy the data.

**SQL solution — Check before creating tables**

```sql
SELECT customer_id,
       COUNT(DISTINCT customer_name) AS different_names
FROM orders_2nf
GROUP BY customer_id
HAVING COUNT(DISTINCT customer_name) > 1;
```

**Expected result:** **No rows**. If rows appear, finish or roll back any unfinished transaction and restore the correct names before continuing.

**SQL solution — Create and populate the 3NF model**

```sql
CREATE TABLE customers_3nf (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(40) NOT NULL
) ENGINE = InnoDB;

CREATE TABLE products_3nf (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(40) NOT NULL
) ENGINE = InnoDB;

CREATE TABLE orders_3nf (
    order_id INT PRIMARY KEY,
    order_date DATE NOT NULL,
    customer_id INT NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers_3nf(customer_id)
) ENGINE = InnoDB;

CREATE TABLE order_items_3nf (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders_3nf(order_id),
    FOREIGN KEY (product_id) REFERENCES products_3nf(product_id)
) ENGINE = InnoDB;

INSERT INTO customers_3nf (customer_id, customer_name)
SELECT DISTINCT customer_id, customer_name
FROM orders_2nf;

INSERT INTO products_3nf (product_id, product_name)
SELECT product_id, product_name
FROM products_2nf;

INSERT INTO orders_3nf (order_id, order_date, customer_id)
SELECT order_id, order_date, customer_id
FROM orders_2nf;

INSERT INTO order_items_3nf (order_id, product_id, quantity)
SELECT order_id, product_id, quantity
FROM order_items_2nf;

SELECT
    (SELECT COUNT(*) FROM customers_3nf) AS customers,
    (SELECT COUNT(*) FROM orders_3nf) AS orders,
    (SELECT COUNT(*) FROM products_3nf) AS products,
    (SELECT COUNT(*) FROM order_items_3nf) AS order_items;
```

**Expected result:** **4 customers, 8 orders, 5 products, 16 order items**.

### Exercise 7 — Repeat the update after 3NF

**Concept:** The customer's current name is stored once in `customers_3nf`. Every order refers to that customer through `customer_id`.

**Task:** Change Riya's name and read all her orders.

**SQL solution**

```sql
START TRANSACTION;

UPDATE customers_3nf
SET customer_name = 'Riya Sharma'
WHERE customer_id = 1;

SELECT o.order_id, c.customer_id, c.customer_name
FROM orders_3nf AS o
JOIN customers_3nf AS c
    ON o.customer_id = c.customer_id
WHERE c.customer_id = 1
ORDER BY o.order_id;

ROLLBACK;
```

**Expected result before rollback:** Both orders **1001** and **1003** show **Riya Sharma**. There is only one customer-name record to update.

### Exercise 8 — Repeat insertion and deletion after 3NF

**Task A:** Register a customer who has not placed an order.

**SQL solution**

```sql
START TRANSACTION;

INSERT INTO customers_3nf (customer_id, customer_name)
VALUES (5, 'Maya');

SELECT customer_id, customer_name
FROM customers_3nf
WHERE customer_id = 5;

ROLLBACK;
```

**Expected result before rollback:** Maya appears in `customers_3nf`, and no new order was needed.

**Task B:** Delete Rahul's orders temporarily. Verify that his customer profile remains.

**SQL solution**

```sql
START TRANSACTION;

DELETE FROM order_items_3nf
WHERE order_id IN (1005, 1008);

DELETE FROM orders_3nf
WHERE order_id IN (1005, 1008);

SELECT customer_id, customer_name
FROM customers_3nf
WHERE customer_id = 4;

ROLLBACK;
```

**Expected result before rollback:** The customer row **4, Rahul** still exists. Deleting orders no longer deletes the customer's profile.

### Exercise 9 — Rebuild complete order details using joins

**Concept:** Normalization does not mean losing information. Tables can be joined when a complete order view is needed.

**Task:** Display customer name, product names, and quantities for order 1001.

**SQL solution**

```sql
SELECT o.order_id,
       c.customer_name,
       p.product_name,
       oi.quantity
FROM orders_3nf AS o
JOIN customers_3nf AS c ON o.customer_id = c.customer_id
JOIN order_items_3nf AS oi ON o.order_id = oi.order_id
JOIN products_3nf AS p ON oi.product_id = p.product_id
WHERE o.order_id = 1001
ORDER BY p.product_id;
```

**Expected result:** Three rows for **Riya**: Keyboard (1), Mouse (2), USB Cable (1).

**3NF recap:** The four-table e-commerce model is in 3NF for the business rules used here. With these same rules, all four tables also happen to satisfy BCNF. We need one extra relationship to see a BCNF problem.

---

## Part C — Boyce-Codd Normal Form (BCNF)

**BCNF in one line:** If some column or combination of columns determines another column, it must also be able to uniquely identify a whole row.

A **superkey** is a column or combination of columns that uniquely identifies a row. A **candidate key** is a superkey with no unnecessary columns.

### Exercise 10 — Create a table that passes 3NF but fails BCNF

**Business rules for current order assignments:**

1. Each product in an order has exactly one assigned specialist.
2. Each specialist handles only one product type.
3. Different specialists can handle the same product in different orders.

**Task:** Record the specialist assigned to each order item.

**SQL solution**

```sql
CREATE TABLE order_product_specialist_3nf (
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    specialist_id INT NOT NULL,
    PRIMARY KEY (order_id, product_id),
    UNIQUE KEY uq_order_specialist (order_id, specialist_id),
    FOREIGN KEY (order_id, product_id)
        REFERENCES order_items_3nf(order_id, product_id)
) ENGINE = InnoDB;

INSERT INTO order_product_specialist_3nf
    (order_id, product_id, specialist_id)
VALUES
    (1001, 101, 501), (1001, 102, 502), (1001, 103, 504),
    (1002, 102, 502), (1002, 104, 505),
    (1003, 101, 503), (1003, 103, 504),
    (1004, 102, 502), (1004, 104, 505),
    (1005, 101, 501), (1005, 103, 504),
    (1006, 102, 502), (1006, 103, 504),
    (1007, 101, 503), (1007, 104, 505),
    (1008, 105, 506);

SELECT specialist_id,
       COUNT(DISTINCT product_id) AS different_products
FROM order_product_specialist_3nf
GROUP BY specialist_id
ORDER BY specialist_id;

SELECT specialist_id, COUNT(*) AS assignment_rows
FROM order_product_specialist_3nf
GROUP BY specialist_id
HAVING COUNT(*) > 1
ORDER BY specialist_id;
```

**Expected result:** **6 specialists** (501–506). Each currently handles **1 product type**. The second query shows that several specialists, including specialist 502, appear in multiple order assignments. Therefore, `specialist_id` does **not** uniquely identify a row.

**Why does this table satisfy 3NF?** There are two candidate keys:

- `(order_id, product_id)` identifies the assigned specialist.
- `(order_id, specialist_id)` identifies the product, because each specialist handles only one product.

Every column is part of at least one candidate key. Therefore, each column is a **prime attribute** and the 3NF rule is satisfied.

**Why does it fail BCNF?** `specialist_id → product_id`, but specialist ID **does not identify one entire row**. For example, specialist 502 appears in multiple orders. A determinant that is not a superkey violates BCNF.

### Exercise 11 — Experience the remaining BCNF anomalies

**Task A — Update anomaly:** Assign specialist 501 to product 102 in one order. But specialist 501 is already assigned to product 101 elsewhere.

**SQL solution**

```sql
START TRANSACTION;

UPDATE order_product_specialist_3nf
SET specialist_id = 501
WHERE order_id = 1002 AND product_id = 102;

SELECT specialist_id,
       COUNT(DISTINCT product_id) AS different_products
FROM order_product_specialist_3nf
GROUP BY specialist_id
HAVING COUNT(DISTINCT product_id) > 1;

ROLLBACK;
```

**Expected result before rollback:** Specialist **501** handles **2 different products** according to the table. The business rule is broken even though the declared primary key and unique key are still valid. After rollback, the inconsistent assignment disappears.

**Task B — Deletion anomaly:** Remove the only order assignment for specialist 506.

**SQL solution**

```sql
START TRANSACTION;

DELETE FROM order_product_specialist_3nf
WHERE order_id = 1008 AND product_id = 105;

SELECT COUNT(*) AS specialist_506_rows
FROM order_product_specialist_3nf
WHERE specialist_id = 506;

ROLLBACK;
```

**Expected result before rollback:** `specialist_506_rows = 0`. There is no longer a record of which product specialist 506 handles. It is also impossible to register a specialist here without an order assignment.

**Task C — Insertion anomaly:** Check whether specialist 507 can be recorded independently of an order.

**SQL solution**

```sql
SELECT specialist_id, product_id
FROM order_product_specialist_3nf
WHERE specialist_id = 507;
```

**Expected result:** No rows. This table requires `order_id` and `product_id` for every row, and each order-product combination is already assigned. It cannot act as a standalone specialist registration table.

### Exercise 12 — Fix the BCNF violation

**Concept:** A specialist's product belongs in a separate table because `specialist_id` determines `product_id`. Order assignments need to store only the order and specialist.

| Table | Purpose | Primary key |
|---|---|---|
| `specialist_products_bcnf` | Which product a specialist handles | `specialist_id` |
| `order_specialists_bcnf` | Which specialists are assigned to orders | `(order_id, specialist_id)` |

**Task:** Create the two BCNF tables and move the data into them.

**SQL solution**

```sql
CREATE TABLE specialist_products_bcnf (
    specialist_id INT PRIMARY KEY,
    product_id INT NOT NULL,
    FOREIGN KEY (product_id) REFERENCES products_3nf(product_id)
) ENGINE = InnoDB;

CREATE TABLE order_specialists_bcnf (
    order_id INT NOT NULL,
    specialist_id INT NOT NULL,
    PRIMARY KEY (order_id, specialist_id),
    FOREIGN KEY (order_id) REFERENCES orders_3nf(order_id),
    FOREIGN KEY (specialist_id)
        REFERENCES specialist_products_bcnf(specialist_id)
) ENGINE = InnoDB;

INSERT INTO specialist_products_bcnf (specialist_id, product_id)
SELECT DISTINCT specialist_id, product_id
FROM order_product_specialist_3nf;

INSERT INTO order_specialists_bcnf (order_id, specialist_id)
SELECT order_id, specialist_id
FROM order_product_specialist_3nf;

SELECT
    (SELECT COUNT(*) FROM specialist_products_bcnf) AS specialists,
    (SELECT COUNT(*) FROM order_specialists_bcnf) AS assignments;
```

**Expected result:** **6 specialists** and **16 order assignments**.

### Exercise 13 — Verify that BCNF removes the repeated specialist-product information

**Task A — Insert and update a specialist without an order.**

**SQL solution**

```sql
START TRANSACTION;

INSERT INTO specialist_products_bcnf (specialist_id, product_id)
VALUES (507, 105);

SELECT specialist_id, product_id
FROM specialist_products_bcnf
WHERE specialist_id = 507;

UPDATE specialist_products_bcnf
SET product_id = 104
WHERE specialist_id = 507;

SELECT specialist_id, product_id
FROM specialist_products_bcnf
WHERE specialist_id = 507;

ROLLBACK;
```

**Expected result before rollback:** Specialist 507 is registered without any order; their product changes from **105 to 104** by updating one record.

**Task B — Remove an order assignment without losing specialist details.**

**SQL solution**

```sql
START TRANSACTION;

DELETE FROM order_specialists_bcnf
WHERE order_id = 1008 AND specialist_id = 506;

SELECT specialist_id, product_id
FROM specialist_products_bcnf
WHERE specialist_id = 506;

ROLLBACK;
```

**Expected result before rollback:** Specialist **506** still handles product **105**, even though their order assignment has been removed.

**Task C — Reconstruct the specialist assignments for order 1001.**

**SQL solution**

```sql
SELECT os.order_id,
       sp.product_id,
       p.product_name,
       os.specialist_id
FROM order_specialists_bcnf AS os
JOIN specialist_products_bcnf AS sp
    ON os.specialist_id = sp.specialist_id
JOIN products_3nf AS p
    ON sp.product_id = p.product_id
WHERE os.order_id = 1001
ORDER BY sp.product_id;
```

**Expected result:** Order 1001 has Keyboard–501, Mouse–502, and USB Cable–504.

**Important limitation:** This BCNF split stores the specialist-product relationship only once. However, it does **not by itself enforce** the original rule that each product in an order has exactly one specialist. A full operational system must enforce that rule separately. Also, if specialist assignments can change over time, historical assignments need an appropriate history design.

---

## Final comparison — 1NF, 2NF, 3NF, and BCNF

| Normal form | Simple rule | Problem addressed | Example |
|---|---|---|---|
| **1NF** | One value per cell; no repeating groups | Comma-separated products and quantities | Store one order item per row |
| **2NF** | Remove partial dependencies on candidate keys | Product details repeated in order items | Put products in a separate table |
| **3NF** | Remove problematic transitive dependencies | Customer name repeated across orders | Put customers in a separate table |
| **BCNF** | Every determinant must be a superkey | A specialist determines a product but cannot identify a whole assignment row | Separate specialists' products from order assignments |

### Four statements to remember

- **1NF:** One value in each cell.
- **2NF:** No partial dependency.
- **3NF:** No problematic transitive dependency.
- **BCNF:** Every determinant must be a superkey.

**Technical accuracy:** The full 3NF rule permits a non-trivial dependency `X → A` if either `X` is a superkey **or** `A` belongs to at least one candidate key (a *prime attribute*). BCNF removes that second exception: for every non-trivial dependency, `X` must be a superkey. This is why the specialist assignment table can satisfy 3NF but fail BCNF.
