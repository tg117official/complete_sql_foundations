-- =========================================================
-- MYSQL SCRIPT FOR CTE PRACTICE
-- Includes:
-- 1. Database creation
-- 2. Table creation
-- 3. Insert statements
-- 4. Notes for different types of CTE
-- 5. Exercises with solutions
-- =========================================================

/* 
A CTE (Common Table Expression) is a temporary named result set created using the WITH clause, 
and it can be used inside the same query like a virtual table.

It helps make SQL easier to read, understand, debug, and write step by step, 
especially when the query is long or complex.

CTEs are very useful when we need intermediate results such as filtered data, aggregated summaries, 
multiple logical steps, or recursive hierarchy logic.

A CTE is not a permanent table and it exists only during the execution of that query; after the query finishes, 
the CTE disappears.

In MySQL, a CTE may be materialized once or merged into the main query by the optimizer, so logically it looks like 
a temporary table, but internally execution can vary.
*/

-- =========================================================
-- CREATE DATABASE
-- =========================================================
CREATE DATABASE IF NOT EXISTS cte_demo_db;
USE cte_demo_db;

-- =========================================================
-- DROP TABLES IF ALREADY EXIST
-- =========================================================
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS departments;

-- =========================================================
-- CREATE DEPARTMENTS TABLE
-- =========================================================
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL,
    location VARCHAR(50) NOT NULL
);

-- =========================================================
-- CREATE EMPLOYEES TABLE
-- =========================================================
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    gender VARCHAR(10),
    age INT,
    job_title VARCHAR(50),
    salary DECIMAL(10,2),
    dept_id INT,
    manager_id INT,
    join_date DATE,
    city VARCHAR(50),
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

-- =========================================================
-- INSERT INTO DEPARTMENTS
-- =========================================================
INSERT INTO departments (dept_id, dept_name, location) VALUES
(1, 'HR', 'Mumbai'),
(2, 'IT', 'Pune'),
(3, 'Finance', 'Delhi'),
(4, 'Sales', 'Bangalore'),
(5, 'Operations', 'Hyderabad');

-- =========================================================
-- INSERT INTO EMPLOYEES
-- =========================================================
INSERT INTO employees (emp_id, emp_name, gender, age, job_title, salary, dept_id, manager_id, join_date, city) VALUES
(101, 'Amit Sharma', 'Male', 32, 'HR Executive', 45000.00, 1, 110, '2021-01-15', 'Mumbai'),
(102, 'Neha Verma', 'Female', 29, 'HR Associate', 42000.00, 1, 110, '2022-03-10', 'Mumbai'),
(103, 'Rohit Patil', 'Male', 35, 'Software Engineer', 70000.00, 2, 111, '2020-06-18', 'Pune'),
(104, 'Sneha Iyer', 'Female', 28, 'Software Engineer', 72000.00, 2, 111, '2021-09-25', 'Pune'),
(105, 'Karan Mehta', 'Male', 31, 'Senior Developer', 95000.00, 2, 111, '2019-11-11', 'Pune'),
(106, 'Pooja Nair', 'Female', 30, 'Accountant', 58000.00, 3, 112, '2020-02-20', 'Delhi'),
(107, 'Vikas Rao', 'Male', 40, 'Finance Analyst', 67000.00, 3, 112, '2018-07-14', 'Delhi'),
(108, 'Meena Joshi', 'Female', 27, 'Sales Executive', 50000.00, 4, 113, '2022-05-05', 'Bangalore'),
(109, 'Arjun Singh', 'Male', 33, 'Sales Manager', 88000.00, 4, 113, '2019-08-19', 'Bangalore'),
(110, 'Priya Desai', 'Female', 45, 'HR Manager', 98000.00, 1, NULL, '2015-04-01', 'Mumbai'),
(111, 'Sandeep Kulkarni', 'Male', 42, 'IT Manager', 120000.00, 2, NULL, '2014-12-12', 'Pune'),
(112, 'Anjali Kapoor', 'Female', 39, 'Finance Manager', 110000.00, 3, NULL, '2016-03-17', 'Delhi'),
(113, 'Ramesh Gowda', 'Male', 44, 'Regional Sales Manager', 115000.00, 4, NULL, '2013-10-30', 'Bangalore'),
(114, 'Tina George', 'Female', 26, 'Operations Executive', 43000.00, 5, 115, '2023-01-09', 'Hyderabad'),
(115, 'Manoj Reddy', 'Male', 38, 'Operations Manager', 90000.00, 5, NULL, '2017-06-23', 'Hyderabad'),
(116, 'Kavita Singh', 'Female', 29, 'Support Engineer', 55000.00, 2, 111, '2022-07-21', 'Pune'),
(117, 'Deepak Yadav', 'Male', 34, 'System Admin', 60000.00, 2, 111, '2021-11-01', 'Pune'),
(118, 'Nisha Arora', 'Female', 31, 'HR Specialist', 47000.00, 1, 110, '2020-10-14', 'Mumbai'),
(119, 'Rahul Jain', 'Male', 36, 'Finance Executive', 62000.00, 3, 112, '2019-01-28', 'Delhi'),
(120, 'Simran Kaur', 'Female', 25, 'Sales Associate', 41000.00, 4, 113, '2023-04-12', 'Bangalore'),
(121, 'Yogesh More', 'Male', 28, 'Operations Analyst', 48000.00, 5, 115, '2022-09-15', 'Hyderabad'),
(122, 'Farah Khan', 'Female', 30, 'Data Analyst', 68000.00, 2, 111, '2021-08-08', 'Pune'),
(123, 'Nitin Sharma', 'Male', 37, 'Recruiter', 46000.00, 1, 110, '2018-05-16', 'Mumbai'),
(124, 'Asha Menon', 'Female', 33, 'Internal Auditor', 75000.00, 3, 112, '2020-12-02', 'Delhi'),
(125, 'Prakash Naidu', 'Male', 29, 'Sales Coordinator', 44000.00, 4, 113, '2022-11-19', 'Bangalore');

-- =========================================================
-- TYPES OF CTE
-- =========================================================

-- 1. BASIC / SIMPLE CTE
-- Purpose:
-- Create a temporary named result set and use it in the main query.
-- Best when you want to simplify a query.

-- 2. CTE WITH AGGREGATION
-- Purpose:
-- First calculate grouped summary like AVG, COUNT, MAX, MIN,
-- then use that result in outer query.

-- 3. CTE WITH FILTERING
-- Purpose:
-- First isolate important rows in a clean named block,
-- then query them further.

-- 4. MULTIPLE CTEs
-- Purpose:
-- Break complex logic into multiple named steps.
-- One CTE can use another CTE.

-- 5. CTE WITH JOIN
-- Purpose:
-- Prepare one side of the data first, then join with another table.

-- 6. RECURSIVE CTE
-- Purpose:
-- Solve hierarchical or repeated-step problems.
-- Example: employee-manager hierarchy, number series, date series.

-- =========================================================
-- EXERCISE 1
-- Type: Basic / Simple CTE
-- Problem:
-- Show all employees from IT department using CTE.
-- =========================================================
WITH it_employees AS (
    SELECT *
    FROM employees
    WHERE dept_id = 2
)
SELECT emp_id, emp_name, salary, city
FROM it_employees;

-- =========================================================
-- EXERCISE 2
-- Type: Basic / Simple CTE
-- Problem:
-- Show employees whose salary is greater than 80000.
-- =========================================================
WITH high_salary_employees AS (
    SELECT emp_id, emp_name, salary, dept_id
    FROM employees
    WHERE salary > 80000
)
SELECT *
FROM high_salary_employees;

-- =========================================================
-- EXERCISE 3
-- Type: CTE with Aggregation
-- Problem:
-- Find average salary of each department.
-- =========================================================
WITH dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT *
FROM dept_avg_salary;

-- =========================================================
-- EXERCISE 4
-- Type: CTE with Aggregation
-- Problem:
-- Find departments whose average salary is greater than 70000.
-- =========================================================
WITH dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT dept_id, avg_salary
FROM dept_avg_salary
WHERE avg_salary > 70000;

-- =========================================================
-- EXERCISE 5
-- Type: CTE with Aggregation
-- Problem:
-- Find employee count in each department.
-- =========================================================
WITH dept_employee_count AS (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
)
SELECT *
FROM dept_employee_count;

-- =========================================================
-- EXERCISE 6
-- Type: CTE with Join
-- Problem:
-- Show department name along with average salary of that department.
-- =========================================================
WITH dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT d.dept_name, d.location, das.avg_salary
FROM dept_avg_salary das
JOIN departments d
    ON das.dept_id = d.dept_id;

-- =========================================================
-- EXERCISE 7
-- Type: CTE with Filtering
-- Problem:
-- From high salary employees, show only those from Pune.
-- =========================================================
WITH high_salary_employees AS (
    SELECT emp_id, emp_name, salary, city
    FROM employees
    WHERE salary > 60000
)
SELECT *
FROM high_salary_employees
WHERE city = 'Pune';

-- =========================================================
-- EXERCISE 8
-- Type: Multiple CTEs
-- Problem:
-- Show department name and employee count for departments
-- having more than 4 employees.
-- =========================================================
WITH dept_employee_count AS (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
),
large_departments AS (
    SELECT dept_id, total_employees
    FROM dept_employee_count
    WHERE total_employees > 4
)
SELECT d.dept_name, ld.total_employees
FROM large_departments ld
JOIN departments d
    ON ld.dept_id = d.dept_id;

-- =========================================================
-- EXERCISE 9
-- Type: Multiple CTEs
-- Problem:
-- Find departments where maximum salary is above 100000.
-- =========================================================
WITH dept_max_salary AS (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY dept_id
),
filtered_departments AS (
    SELECT *
    FROM dept_max_salary
    WHERE max_salary > 100000
)
SELECT d.dept_name, fd.max_salary
FROM filtered_departments fd
JOIN departments d
    ON fd.dept_id = d.dept_id;

-- =========================================================
-- EXERCISE 10
-- Type: CTE with Join
-- Problem:
-- Show employee name, salary and department name using CTE.
-- =========================================================
WITH employee_basic AS (
    SELECT emp_id, emp_name, salary, dept_id
    FROM employees
)
SELECT eb.emp_name, eb.salary, d.dept_name
FROM employee_basic eb
JOIN departments d
    ON eb.dept_id = d.dept_id;

-- =========================================================
-- EXERCISE 11
-- Type: CTE with Aggregation
-- Problem:
-- Find the highest salary in each department.
-- =========================================================
WITH dept_highest_salary AS (
    SELECT dept_id, MAX(salary) AS highest_salary
    FROM employees
    GROUP BY dept_id
)
SELECT *
FROM dept_highest_salary;

-- =========================================================
-- EXERCISE 12
-- Type: CTE with Aggregation + Join
-- Problem:
-- Show department name with highest salary in that department.
-- =========================================================
WITH dept_highest_salary AS (
    SELECT dept_id, MAX(salary) AS highest_salary
    FROM employees
    GROUP BY dept_id
)
SELECT d.dept_name, dhs.highest_salary
FROM dept_highest_salary dhs
JOIN departments d
    ON dhs.dept_id = d.dept_id;

-- =========================================================
-- EXERCISE 13
-- Type: CTE with Aggregation
-- Problem:
-- Find the minimum salary in each department.
-- =========================================================
WITH dept_min_salary AS (
    SELECT dept_id, MIN(salary) AS min_salary
    FROM employees
    GROUP BY dept_id
)
SELECT *
FROM dept_min_salary;

-- =========================================================
-- EXERCISE 14
-- Type: Basic / Simple CTE
-- Problem:
-- Show employees who joined after 2021-01-01.
-- =========================================================
WITH recent_employees AS (
    SELECT emp_id, emp_name, join_date, dept_id
    FROM employees
    WHERE join_date > '2021-01-01'
)
SELECT *
FROM recent_employees;

-- =========================================================
-- EXERCISE 15
-- Type: CTE with Filtering
-- Problem:
-- Find female employees from departments located in Delhi.
-- =========================================================
WITH female_employees AS (
    SELECT emp_id, emp_name, gender, dept_id
    FROM employees
    WHERE gender = 'Female'
)
SELECT fe.emp_id, fe.emp_name, d.dept_name, d.location
FROM female_employees fe
JOIN departments d
    ON fe.dept_id = d.dept_id
WHERE d.location = 'Delhi';

-- =========================================================
-- EXERCISE 16
-- Type: Multiple CTEs
-- Problem:
-- Find employees working in departments whose average salary
-- is greater than 70000.
-- =========================================================
WITH dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
),
rich_departments AS (
    SELECT dept_id
    FROM dept_avg_salary
    WHERE avg_salary > 70000
)
SELECT e.emp_id, e.emp_name, e.salary, e.dept_id
FROM employees e
JOIN rich_departments rd
    ON e.dept_id = rd.dept_id;

-- =========================================================
-- EXERCISE 17
-- Type: Recursive CTE
-- Purpose:
-- Generate numbers from 1 to 10.
-- Recursive CTE has:
-- 1. Anchor part (starting row)
-- 2. Recursive part (repeats until condition fails)
-- =========================================================
WITH RECURSIVE number_series AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1
    FROM number_series
    WHERE n < 10
)
SELECT *
FROM number_series;

-- =========================================================
-- EXERCISE 18
-- Type: Recursive CTE
-- Problem:
-- Generate numbers from 5 to 15.
-- =========================================================
WITH RECURSIVE custom_series AS (
    SELECT 5 AS n
    UNION ALL
    SELECT n + 1
    FROM custom_series
    WHERE n < 15
)
SELECT *
FROM custom_series;

-- =========================================================
-- EXERCISE 19
-- Type: Recursive CTE
-- Problem:
-- Display employee-manager hierarchy starting from top-level managers.
-- Note:
-- Top-level managers have manager_id IS NULL.
-- level_no indicates hierarchy level.
-- =========================================================
WITH RECURSIVE employee_hierarchy AS (
    SELECT
        emp_id,
        emp_name,
        manager_id,
        dept_id,
        1 AS level_no
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT
        e.emp_id,
        e.emp_name,
        e.manager_id,
        e.dept_id,
        eh.level_no + 1
    FROM employees e
    JOIN employee_hierarchy eh
        ON e.manager_id = eh.emp_id
)
SELECT *
FROM employee_hierarchy
ORDER BY level_no, manager_id, emp_id;

-- =========================================================
-- EXERCISE 20
-- Type: Multiple CTEs
-- Problem:
-- Find department name, employee count, average salary,
-- and maximum salary together in final result.
-- =========================================================
WITH dept_employee_count AS (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
),
dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
),
dept_max_salary AS (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY dept_id
)
SELECT
    d.dept_name,
    dec.total_employees,
    das.avg_salary,
    dms.max_salary
FROM departments d
JOIN dept_employee_count dec
    ON d.dept_id = dec.dept_id
JOIN dept_avg_salary das
    ON d.dept_id = das.dept_id
JOIN dept_max_salary dms
    ON d.dept_id = dms.dept_id;

-- =========================================================
-- EXERCISE 21
-- Type: CTE Reused in Final Query
-- Problem:
-- Find employees whose salary is above their department average.
-- =========================================================
WITH dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT e.emp_id, e.emp_name, e.salary, e.dept_id, das.avg_salary
FROM employees e
JOIN dept_avg_salary das
    ON e.dept_id = das.dept_id
WHERE e.salary > das.avg_salary;

-- =========================================================
-- EXERCISE 22
-- Type: CTE Reused in Final Query
-- Problem:
-- Find employees whose salary is equal to department maximum salary.
-- =========================================================
WITH dept_max_salary AS (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY dept_id
)
SELECT e.emp_id, e.emp_name, e.salary, e.dept_id
FROM employees e
JOIN dept_max_salary dms
    ON e.dept_id = dms.dept_id
   AND e.salary = dms.max_salary;

-- =========================================================
-- EXERCISE 23
-- Type: CTE Reused in Final Query
-- Problem:
-- Find employees whose salary is equal to department minimum salary.
-- =========================================================
WITH dept_min_salary AS (
    SELECT dept_id, MIN(salary) AS min_salary
    FROM employees
    GROUP BY dept_id
)
SELECT e.emp_id, e.emp_name, e.salary, e.dept_id
FROM employees e
JOIN dept_min_salary dmn
    ON e.dept_id = dmn.dept_id
   AND e.salary = dmn.min_salary;

-- =========================================================
-- EXERCISE 24
-- Type: Basic / Simple CTE
-- Problem:
-- Show all employees from Mumbai and Bangalore only.
-- =========================================================
WITH selected_city_employees AS (
    SELECT emp_id, emp_name, city, salary
    FROM employees
    WHERE city IN ('Mumbai', 'Bangalore')
)
SELECT *
FROM selected_city_employees;

-- =========================================================
-- EXERCISE 25
-- Type: Multiple CTEs
-- Problem:
-- Find departments where employee count is at least 4
-- and average salary is above 60000.
-- =========================================================
WITH dept_employee_count AS (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
),
dept_avg_salary AS (
    SELECT dept_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept_id
)
SELECT d.dept_name, dec.total_employees, das.avg_salary
FROM departments d
JOIN dept_employee_count dec
    ON d.dept_id = dec.dept_id
JOIN dept_avg_salary das
    ON d.dept_id = das.dept_id
WHERE dec.total_employees >= 4
  AND das.avg_salary > 60000;
