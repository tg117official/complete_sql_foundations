-- =========================================================
-- MYSQL SCRIPT FOR PIVOT AND UNPIVOT PRACTICE
-- Note:
-- MySQL does not provide native PIVOT / UNPIVOT syntax.
-- So in MySQL:
-- 1. Pivot is commonly done using CASE + aggregate functions
-- 2. Unpivot is commonly done using UNION ALL
-- =========================================================

-- =========================================================
-- CREATE DATABASE
-- =========================================================
CREATE DATABASE IF NOT EXISTS pivot_unpivot_demo_db;
USE pivot_unpivot_demo_db;

-- =========================================================
-- DROP TABLE IF EXISTS
-- =========================================================
DROP TABLE IF EXISTS sales_data;
DROP TABLE IF EXISTS employee_scores;

-- =========================================================
-- TABLE 1: SALES DATA
-- Purpose:
-- Useful for pivot exercises
-- =========================================================
CREATE TABLE sales_data (
    sale_id INT PRIMARY KEY,
    sales_year INT,
    sales_month VARCHAR(20),
    product_name VARCHAR(50),
    sales_amount DECIMAL(10,2)
);

-- =========================================================
-- INSERT DATA INTO SALES_DATA
-- =========================================================
INSERT INTO sales_data (sale_id, sales_year, sales_month, product_name, sales_amount) VALUES
(1, 2025, 'January',  'Laptop',   50000),
(2, 2025, 'February', 'Laptop',   55000),
(3, 2025, 'March',    'Laptop',   52000),
(4, 2025, 'January',  'Mobile',   30000),
(5, 2025, 'February', 'Mobile',   32000),
(6, 2025, 'March',    'Mobile',   31000),
(7, 2025, 'January',  'Tablet',   20000),
(8, 2025, 'February', 'Tablet',   22000),
(9, 2025, 'March',    'Tablet',   21000),
(10, 2025, 'April',   'Laptop',   60000),
(11, 2025, 'April',   'Mobile',   34000),
(12, 2025, 'April',   'Tablet',   23000),
(13, 2025, 'May',     'Laptop',   61000),
(14, 2025, 'May',     'Mobile',   35000),
(15, 2025, 'May',     'Tablet',   24000);

-- =========================================================
-- TABLE 2: EMPLOYEE SCORES
-- Purpose:
-- Useful for unpivot exercises
-- =========================================================
CREATE TABLE employee_scores (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    communication_score INT,
    technical_score INT,
    management_score INT
);

-- =========================================================
-- INSERT DATA INTO EMPLOYEE_SCORES
-- =========================================================
INSERT INTO employee_scores (emp_id, emp_name, communication_score, technical_score, management_score) VALUES
(101, 'Amit', 78, 88, 70),
(102, 'Neha', 85, 91, 75),
(103, 'Rohit', 72, 95, 68),
(104, 'Sneha', 90, 89, 82),
(105, 'Karan', 68, 84, 73),
(106, 'Pooja', 88, 86, 80);

-- =========================================================
-- SHORT NOTES
-- =========================================================

-- PIVOT
-- Purpose:
-- Convert row values into columns.
-- In MySQL, pivot is commonly created using CASE with SUM, MAX, COUNT, etc.

-- UNPIVOT
-- Purpose:
-- Convert columns into rows.
-- In MySQL, unpivot is commonly created using UNION ALL.

-- =========================================================
-- PIVOT EXERCISE 1
-- Problem:
-- Show product-wise sales for January, February, and March in separate columns.
-- Type:
-- Basic Pivot using CASE + SUM
-- =========================================================
SELECT
    product_name,
    SUM(CASE WHEN sales_month = 'January'  THEN sales_amount ELSE 0 END) AS January,
    SUM(CASE WHEN sales_month = 'February' THEN sales_amount ELSE 0 END) AS February,
    SUM(CASE WHEN sales_month = 'March'    THEN sales_amount ELSE 0 END) AS March
FROM sales_data
GROUP BY product_name;

-- =========================================================
-- PIVOT EXERCISE 2
-- Problem:
-- Show product-wise sales for April and May in separate columns.
-- Type:
-- Basic Pivot using CASE + SUM
-- =========================================================
SELECT
    product_name,
    SUM(CASE WHEN sales_month = 'April' THEN sales_amount ELSE 0 END) AS April,
    SUM(CASE WHEN sales_month = 'May'   THEN sales_amount ELSE 0 END) AS May
FROM sales_data
GROUP BY product_name;

-- =========================================================
-- PIVOT EXERCISE 3
-- Problem:
-- Show month-wise sales of Laptop, Mobile and Tablet as separate columns.
-- Type:
-- Reverse-style Pivot using CASE + SUM
-- =========================================================
SELECT
    sales_month,
    SUM(CASE WHEN product_name = 'Laptop' THEN sales_amount ELSE 0 END) AS Laptop,
    SUM(CASE WHEN product_name = 'Mobile' THEN sales_amount ELSE 0 END) AS Mobile,
    SUM(CASE WHEN product_name = 'Tablet' THEN sales_amount ELSE 0 END) AS Tablet
FROM sales_data
GROUP BY sales_month
ORDER BY FIELD(sales_month, 'January', 'February', 'March', 'April', 'May');

-- =========================================================
-- PIVOT EXERCISE 4
-- Problem:
-- Show how many sales records exist for each product in each month
-- for January, February, March.
-- Type:
-- Pivot using CASE + COUNT style with SUM
-- =========================================================
SELECT
    product_name,
    SUM(CASE WHEN sales_month = 'January' THEN 1 ELSE 0 END) AS Jan_Record_Count,
    SUM(CASE WHEN sales_month = 'February' THEN 1 ELSE 0 END) AS Feb_Record_Count,
    SUM(CASE WHEN sales_month = 'March' THEN 1 ELSE 0 END) AS Mar_Record_Count
FROM sales_data
GROUP BY product_name;

-- =========================================================
-- PIVOT EXERCISE 5
-- Problem:
-- Show year-wise January sales for each product.
-- Type:
-- Pivot with grouping on year
-- =========================================================
SELECT
    sales_year,
    SUM(CASE WHEN product_name = 'Laptop' AND sales_month = 'January' THEN sales_amount ELSE 0 END) AS Laptop_January,
    SUM(CASE WHEN product_name = 'Mobile' AND sales_month = 'January' THEN sales_amount ELSE 0 END) AS Mobile_January,
    SUM(CASE WHEN product_name = 'Tablet' AND sales_month = 'January' THEN sales_amount ELSE 0 END) AS Tablet_January
FROM sales_data
GROUP BY sales_year;

-- =========================================================
-- PIVOT EXERCISE 6
-- Problem:
-- Show total sales of each product across all months in column format.
-- Type:
-- Single-row Pivot
-- =========================================================
SELECT
    SUM(CASE WHEN product_name = 'Laptop' THEN sales_amount ELSE 0 END) AS Laptop_Total,
    SUM(CASE WHEN product_name = 'Mobile' THEN sales_amount ELSE 0 END) AS Mobile_Total,
    SUM(CASE WHEN product_name = 'Tablet' THEN sales_amount ELSE 0 END) AS Tablet_Total
FROM sales_data;

-- =========================================================
-- PIVOT EXERCISE 7
-- Problem:
-- Show only January to April sales for each product in column format.
-- Type:
-- Multi-column Pivot
-- =========================================================
SELECT
    product_name,
    SUM(CASE WHEN sales_month = 'January' THEN sales_amount ELSE 0 END) AS January,
    SUM(CASE WHEN sales_month = 'February' THEN sales_amount ELSE 0 END) AS February,
    SUM(CASE WHEN sales_month = 'March' THEN sales_amount ELSE 0 END) AS March,
    SUM(CASE WHEN sales_month = 'April' THEN sales_amount ELSE 0 END) AS April
FROM sales_data
GROUP BY product_name;

-- =========================================================
-- PIVOT EXERCISE 8
-- Problem:
-- Show month-wise maximum sales amount for each product in columns.
-- Type:
-- Pivot using CASE + MAX
-- =========================================================
SELECT
    sales_month,
    MAX(CASE WHEN product_name = 'Laptop' THEN sales_amount END) AS Laptop_Max,
    MAX(CASE WHEN product_name = 'Mobile' THEN sales_amount END) AS Mobile_Max,
    MAX(CASE WHEN product_name = 'Tablet' THEN sales_amount END) AS Tablet_Max
FROM sales_data
GROUP BY sales_month
ORDER BY FIELD(sales_month, 'January', 'February', 'March', 'April', 'May');

-- =========================================================
-- UNPIVOT EXERCISE 1
-- Problem:
-- Convert communication_score, technical_score, management_score
-- columns into rows.
-- Type:
-- Basic Unpivot using UNION ALL
-- =========================================================
SELECT emp_id, emp_name, 'communication_score' AS score_type, communication_score AS score_value
FROM employee_scores
UNION ALL
SELECT emp_id, emp_name, 'technical_score' AS score_type, technical_score AS score_value
FROM employee_scores
UNION ALL
SELECT emp_id, emp_name, 'management_score' AS score_type, management_score AS score_value
FROM employee_scores;

-- =========================================================
-- UNPIVOT EXERCISE 2
-- Problem:
-- Unpivot only Amit's scores into rows.
-- Type:
-- Filtered Unpivot
-- =========================================================
SELECT emp_id, emp_name, 'communication_score' AS score_type, communication_score AS score_value
FROM employee_scores
WHERE emp_name = 'Amit'
UNION ALL
SELECT emp_id, emp_name, 'technical_score' AS score_type, technical_score AS score_value
FROM employee_scores
WHERE emp_name = 'Amit'
UNION ALL
SELECT emp_id, emp_name, 'management_score' AS score_type, management_score AS score_value
FROM employee_scores
WHERE emp_name = 'Amit';

-- =========================================================
-- UNPIVOT EXERCISE 3
-- Problem:
-- Unpivot all scores and show only score values greater than 85.
-- Type:
-- Unpivot with outer filtering
-- =========================================================
SELECT *
FROM (
    SELECT emp_id, emp_name, 'communication_score' AS score_type, communication_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT emp_id, emp_name, 'technical_score' AS score_type, technical_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT emp_id, emp_name, 'management_score' AS score_type, management_score AS score_value
    FROM employee_scores
) AS unpivoted_scores
WHERE score_value > 85;

-- =========================================================
-- UNPIVOT EXERCISE 4
-- Problem:
-- Unpivot all scores and sort by employee and score type.
-- Type:
-- Ordered Unpivot
-- =========================================================
SELECT emp_id, emp_name, score_type, score_value
FROM (
    SELECT emp_id, emp_name, 'communication_score' AS score_type, communication_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT emp_id, emp_name, 'technical_score' AS score_type, technical_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT emp_id, emp_name, 'management_score' AS score_type, management_score AS score_value
    FROM employee_scores
) AS unpivoted_scores
ORDER BY emp_id, score_type;

-- =========================================================
-- UNPIVOT EXERCISE 5
-- Problem:
-- Show average score by score type after unpivoting.
-- Type:
-- Unpivot + Aggregation
-- =========================================================
SELECT score_type, AVG(score_value) AS avg_score
FROM (
    SELECT 'communication_score' AS score_type, communication_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT 'technical_score' AS score_type, technical_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT 'management_score' AS score_type, management_score AS score_value
    FROM employee_scores
) AS unpivoted_scores
GROUP BY score_type;

-- =========================================================
-- UNPIVOT EXERCISE 6
-- Problem:
-- Show highest score by score type after unpivoting.
-- Type:
-- Unpivot + MAX
-- =========================================================
SELECT score_type, MAX(score_value) AS max_score
FROM (
    SELECT 'communication_score' AS score_type, communication_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT 'technical_score' AS score_type, technical_score AS score_value
    FROM employee_scores
    UNION ALL
    SELECT 'management_score' AS score_type, management_score AS score_value
    FROM employee_scores
) AS unpivoted_scores
GROUP BY score_type;

-- =========================================================
-- UNPIVOT EXERCISE 7
-- Problem:
-- Show employee-wise all scores in row format only for employees
-- whose technical score is greater than 85.
-- Type:
-- Conditional Unpivot
-- =========================================================
SELECT emp_id, emp_name, 'communication_score' AS score_type, communication_score AS score_value
FROM employee_scores
WHERE technical_score > 85
UNION ALL
SELECT emp_id, emp_name, 'technical_score' AS score_type, technical_score AS score_value
FROM employee_scores
WHERE technical_score > 85
UNION ALL
SELECT emp_id, emp_name, 'management_score' AS score_type, management_score AS score_value
FROM employee_scores
WHERE technical_score > 85;

-- =========================================================
-- UNPIVOT EXERCISE 8
-- Problem:
-- Convert scores into rows and rename score_type values in short form.
-- Type:
-- Presentation-style Unpivot
-- =========================================================
SELECT emp_id, emp_name, 'Communication' AS score_type, communication_score AS score_value
FROM employee_scores
UNION ALL
SELECT emp_id, emp_name, 'Technical' AS score_type, technical_score AS score_value
FROM employee_scores
UNION ALL
SELECT emp_id, emp_name, 'Management' AS score_type, management_score AS score_value
FROM employee_scores;
