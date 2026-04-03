-- ============================================================
-- MULTIPLE COLUMNS IN PARTITION BY AND ORDER BY
-- WINDOW FUNCTIONS DEMO SCRIPT - MYSQL 8.0+
-- ============================================================

-- ============================================================
-- DATABASE SETUP
-- ============================================================

DROP DATABASE IF EXISTS mysql_window_multi_column_demo;
CREATE DATABASE mysql_window_multi_column_demo;
USE mysql_window_multi_column_demo;

-- ============================================================
-- TABLE CREATION
-- ============================================================

DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50) NOT NULL,
    department VARCHAR(30) NOT NULL,
    city VARCHAR(30) NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2) NOT NULL,
    performance_score INT NOT NULL
);

-- ============================================================
-- SAMPLE DATA
-- Data is intentionally designed to show:
-- 1. Same department + different city
-- 2. Same salary within same department
-- 3. Same salary + same city combinations
-- 4. Different hire dates for tie-breaking
-- ============================================================

INSERT INTO employees (emp_id, emp_name, department, city, hire_date, salary, performance_score) VALUES
(1,  'Amit',   'Engineering', 'Pune',      '2021-01-10', 75000, 88),
(2,  'Neha',   'Engineering', 'Pune',      '2021-03-15', 78000, 91),
(3,  'Rohit',  'Engineering', 'Mumbai',    '2021-04-20', 62000, 79),
(4,  'Sneha',  'Engineering', 'Pune',      '2021-06-11', 95000, 94),
(5,  'Karan',  'Engineering', 'Mumbai',    '2022-01-05', 62000, 82),
(6,  'Isha',   'Engineering', 'Pune',      '2022-02-14', 78000, 86),

(7,  'Pooja',  'Sales',       'Mumbai',    '2021-02-17', 54000, 85),
(8,  'Vikas',  'Sales',       'Mumbai',    '2021-05-23', 54000, 81),
(9,  'Ritu',   'Sales',       'Pune',      '2020-12-01', 88000, 92),
(10, 'Manoj',  'Sales',       'Delhi',     '2022-07-19', 67000, 77),
(11, 'Priya',  'Sales',       'Mumbai',    '2023-01-25', 50000, 74),
(12, 'Asha',   'Sales',       'Pune',      '2022-03-08', 88000, 89),

(13, 'Anjali', 'HR',          'Delhi',     '2021-08-10', 58000, 84),
(14, 'Suresh', 'HR',          'Delhi',     '2020-09-14', 82000, 89),
(15, 'Meena',  'HR',          'Jaipur',    '2022-02-11', 52000, 76),
(16, 'Tarun',  'HR',          'Delhi',     '2023-04-09', 52000, 73),
(17, 'Kavita', 'HR',          'Pune',      '2022-11-30', 61000, 87),

(18, 'Arjun',  'Finance',     'Bangalore', '2021-07-07', 70000, 83),
(19, 'Divya',  'Finance',     'Bangalore', '2021-10-18', 70000, 86),
(20, 'Nitin',  'Finance',     'Mumbai',    '2020-06-22', 98000, 93),
(21, 'Shreya', 'Finance',     'Bangalore', '2022-03-13', 56000, 78),
(22, 'Yash',   'Finance',     'Pune',      '2023-05-16', 56000, 75);

-- ============================================================
-- BASE DATA CHECK
-- ============================================================

SELECT * FROM employees ORDER BY department, city, salary DESC, hire_date;

-- ============================================================
-- EXERCISE 1
-- WHAT HAPPENS WITH SINGLE-COLUMN PARTITION
-- Here partition is only by department
-- Row numbering restarts for each department
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn_dept
FROM employees
ORDER BY department, salary DESC;

-- ============================================================
-- EXERCISE 2
-- MULTIPLE COLUMNS IN PARTITION BY
-- Now partition is by department + city combination
-- So numbering restarts for every unique (department, city)
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department, city
        ORDER BY salary DESC
    ) AS rn_dept_city
FROM employees
ORDER BY department, city, salary DESC;

-- ============================================================
-- EXERCISE 3
-- UNDERSTANDING MULTIPLE COLUMNS IN PARTITION BY
-- COUNT() OVER helps visualize the size of each partition
-- Here count is calculated for each (department, city) combination
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    COUNT(*) OVER (
        PARTITION BY department, city
    ) AS employees_in_same_dept_city
FROM employees
ORDER BY department, city, emp_id;

-- ============================================================
-- EXERCISE 4
-- SINGLE-COLUMN ORDER BY
-- Ranking by salary only inside each department
-- If salary ties, RANK() gives same rank
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rank_by_salary_only
FROM employees
ORDER BY department, salary DESC, emp_id;

-- ============================================================
-- EXERCISE 5
-- MULTIPLE COLUMNS IN ORDER BY
-- Ranking by salary DESC and then hire_date ASC
--
-- Important concept:
-- Now hire_date acts as tie-breaker.
-- So rows with same salary may no longer remain tied.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    hire_date,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
    ) AS rank_by_salary_then_hire_date
FROM employees
ORDER BY department, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 6
-- DENSE_RANK() WITH SALARY ONLY
-- Useful to compare with next query
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    DENSE_RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS dense_rank_salary_only
FROM employees
ORDER BY department, salary DESC, emp_id;

-- ============================================================
-- EXERCISE 7
-- DENSE_RANK() WITH MULTIPLE ORDER COLUMNS
-- salary DESC + hire_date ASC
-- Here tie may break because second column is included
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    hire_date,
    DENSE_RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
    ) AS dense_rank_salary_hire_date
FROM employees
ORDER BY department, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 8
-- ROW_NUMBER() WITH MULTIPLE ORDER COLUMNS
-- Here salary DESC is primary ordering
-- hire_date ASC is secondary ordering
-- emp_id ASC is final stable tie-breaker
--
-- This is a best-practice style ordering
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    hire_date,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC, emp_id ASC
    ) AS rn_stable_order
FROM employees
ORDER BY department, salary DESC, hire_date ASC, emp_id ASC;

-- ============================================================
-- EXERCISE 9
-- MULTIPLE COLUMNS IN BOTH PARTITION BY AND ORDER BY
-- Partition = department + city
-- Order = salary DESC, hire_date ASC
--
-- Meaning:
-- for every unique (department, city), create separate numbering
-- and inside that, sort by salary, then hire date
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    hire_date,
    ROW_NUMBER() OVER (
        PARTITION BY department, city
        ORDER BY salary DESC, hire_date ASC
    ) AS rn_dept_city_salary_hire
FROM employees
ORDER BY department, city, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 10
-- AGGREGATE WINDOW FUNCTION WITH MULTIPLE PARTITION COLUMNS
-- Sum salary within each (department, city)
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    SUM(salary) OVER (
        PARTITION BY department, city
    ) AS total_salary_in_dept_city
FROM employees
ORDER BY department, city, salary DESC;

-- ============================================================
-- EXERCISE 11
-- RUNNING TOTAL WITH MULTIPLE ORDER COLUMNS
-- Partition by department
-- Order by hire_date, then emp_id
--
-- emp_id is included as extra tie-breaker for stable sequence
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date, emp_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_salary_in_department
FROM employees
ORDER BY department, hire_date, emp_id;

-- ============================================================
-- EXERCISE 12
-- LAG() WITH MULTIPLE PARTITION AND ORDER COLUMNS
-- Partition = department + city
-- Order = hire_date, emp_id
--
-- Previous salary is found within same department and city only
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    hire_date,
    salary,
    LAG(salary) OVER (
        PARTITION BY department, city
        ORDER BY hire_date, emp_id
    ) AS previous_salary_same_dept_city
FROM employees
ORDER BY department, city, hire_date, emp_id;

-- ============================================================
-- EXERCISE 13
-- LEAD() WITH MULTIPLE PARTITION AND ORDER COLUMNS
-- Next salary is found within same department and city only
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    hire_date,
    salary,
    LEAD(salary) OVER (
        PARTITION BY department, city
        ORDER BY hire_date, emp_id
    ) AS next_salary_same_dept_city
FROM employees
ORDER BY department, city, hire_date, emp_id;

-- ============================================================
-- EXERCISE 14
-- FIRST_VALUE() WITH MULTIPLE ORDER COLUMNS
-- Highest salary in each department
-- If same salary exists, earlier hire_date decides the first row
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    hire_date,
    FIRST_VALUE(emp_name) OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
    ) AS first_employee_in_dept_by_salary_then_hire_date
FROM employees
ORDER BY department, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 15
-- LAST_VALUE() WITH MULTIPLE ORDER COLUMNS
-- Need explicit frame for true last row in full partition
--
-- Here:
-- salary DESC, hire_date ASC
-- so the last row tends toward lowest salary and latest tie position
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    hire_date,
    LAST_VALUE(emp_name) OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_employee_in_dept_by_salary_then_hire_date
FROM employees
ORDER BY department, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 16
-- SIDE-BY-SIDE COMPARISON
-- Compare rank based on:
-- 1. salary only
-- 2. salary + hire_date
--
-- This is one of the best exercises to understand how
-- extra ORDER BY columns affect ranking
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,
    hire_date,

    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rank_salary_only,

    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
    ) AS rank_salary_then_hire_date

FROM employees
ORDER BY department, salary DESC, hire_date ASC;

-- ============================================================
-- EXERCISE 17
-- SIDE-BY-SIDE COMPARISON
-- Compare numbering based on:
-- 1. department only
-- 2. department + city
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    salary,

    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn_department,

    ROW_NUMBER() OVER (
        PARTITION BY department, city
        ORDER BY salary DESC
    ) AS rn_department_city

FROM employees
ORDER BY department, city, salary DESC;

-- ============================================================
-- EXERCISE 18
-- GOOD PRACTICE QUERY
-- Multiple window functions together
-- Using stable outer ORDER BY for readability
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    city,
    hire_date,
    salary,

    ROW_NUMBER() OVER (
        PARTITION BY department, city
        ORDER BY salary DESC, hire_date ASC, emp_id ASC
    ) AS row_num_in_dept_city,

    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS dept_salary_rank,

    DENSE_RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC, hire_date ASC
    ) AS dept_dense_rank_salary_hire,

    LAG(salary) OVER (
        PARTITION BY department, city
        ORDER BY hire_date, emp_id
    ) AS previous_salary_in_dept_city,

    SUM(salary) OVER (
        PARTITION BY department, city
    ) AS total_salary_in_dept_city

FROM employees
ORDER BY department, city, salary DESC, hire_date ASC, emp_id ASC;

-- ============================================================
-- END OF SCRIPT
-- ============================================================
