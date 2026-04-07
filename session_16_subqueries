-- =========================================================
-- MYSQL SCRIPT FOR SUBQUERY PRACTICE
-- =========================================================

-- Create database
CREATE DATABASE IF NOT EXISTS subquery_demo_db;

-- Use database
USE subquery_demo_db;

-- =========================================================
-- 1. CREATE DEPARTMENT TABLE
-- =========================================================
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    location VARCHAR(50)
);

-- =========================================================
-- 2. CREATE EMPLOYEE TABLE
-- =========================================================
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
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
-- 3. INSERT DATA INTO DEPARTMENTS
-- =========================================================
INSERT INTO departments (dept_id, dept_name, location) VALUES
(1, 'HR', 'Mumbai'),
(2, 'IT', 'Pune'),
(3, 'Finance', 'Delhi'),
(4, 'Sales', 'Bangalore'),
(5, 'Operations', 'Hyderabad');

-- =========================================================
-- 4. INSERT DATA INTO EMPLOYEES
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
-- 10 BASIC SUBQUERY EXERCISES WITH SOLUTIONS
-- Based on:
-- departments
-- employees
-- =========================================================

   
USE subquery_demo_db;

-- =========================================================
-- EXERCISE 1
-- Type: Scalar Subquery
-- Definition:
-- A scalar subquery returns only one value.
--
-- Problem Statement:
-- Find employees whose salary is greater than the average
-- salary of all employees.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);

-- =========================================================
-- EXERCISE 2
-- Type: Scalar Subquery
-- Definition:
-- A scalar subquery returns one value and is often used with =.
--
-- Problem Statement:
-- Find all employees who work in the IT department.
-- =========================================================
SELECT emp_id, emp_name, dept_id
FROM employees
WHERE dept_id = (
    SELECT dept_id
    FROM departments
    WHERE dept_name = 'IT'
);

-- =========================================================
-- EXERCISE 3
-- Type: Multiple Row Subquery using IN
-- Definition:
-- A multiple row subquery returns multiple values.
-- IN is used when outer query needs to match any of those values.
--
-- Problem Statement:
-- Find employees who work in departments located in Delhi
-- or Mumbai or any other matching location returned by subquery.
-- Here we are filtering for Mumbai.
-- =========================================================
SELECT emp_id, emp_name, dept_id
FROM employees
WHERE dept_id IN (
    SELECT dept_id
    FROM departments
    WHERE location = 'Mumbai'
);

-- =========================================================
-- EXERCISE 4
-- Type: Scalar Subquery
-- Definition:
-- A scalar subquery returns one value.
--
-- Problem Statement:
-- Find employees whose salary is greater than the salary
-- of 'Neha Verma'.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > (
    SELECT salary
    FROM employees
    WHERE emp_name = 'Neha Verma'
);

-- =========================================================
-- EXERCISE 5
-- Type: Multiple Row Subquery using IN
-- Definition:
-- A multiple row subquery returns many values.
-- IN checks whether a value exists in that returned list.
--
-- Problem Statement:
-- Find employees who are managers.
-- Hint: managers are those whose emp_id appears as manager_id
-- for some other employee.
-- =========================================================
SELECT emp_id, emp_name, job_title
FROM employees
WHERE emp_id IN (
    SELECT DISTINCT manager_id
    FROM employees
    WHERE manager_id IS NOT NULL
);

-- =========================================================
-- EXERCISE 6
-- Type: Multiple Row Subquery using NOT IN
-- Definition:
-- NOT IN is used to find rows whose value does not exist
-- in the subquery result.
--
-- Problem Statement:
-- Find employees who are not managers.
-- =========================================================
SELECT emp_id, emp_name, job_title
FROM employees
WHERE emp_id NOT IN (
    SELECT DISTINCT manager_id
    FROM employees
    WHERE manager_id IS NOT NULL
);

-- =========================================================
-- EXERCISE 7
-- Type: Scalar Subquery
-- Definition:
-- A scalar subquery returns one value.
--
-- Problem Statement:
-- Find employee(s) who have the highest salary in the company.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);

-- =========================================================
-- EXERCISE 8
-- Type: Correlated Subquery
-- Definition:
-- A correlated subquery depends on the current row of outer query.
-- It executes again for each row of outer query.
--
-- Problem Statement:
-- Find employees whose salary is greater than the average salary
-- of their own department.
-- =========================================================
SELECT e1.emp_id, e1.emp_name, e1.salary, e1.dept_id
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);

-- =========================================================
-- EXERCISE 9
-- Type: Subquery in FROM Clause
-- Definition:
-- A subquery in FROM clause works like a temporary table.
--
-- Problem Statement:
-- Find departments that have at least 4 employees.
-- First calculate employee count department-wise,
-- then filter in outer query.
-- =========================================================
SELECT dept_id, total_employees
FROM (
    SELECT dept_id, COUNT(*) AS total_employees
    FROM employees
    GROUP BY dept_id
) AS dept_count
WHERE total_employees >= 4;

-- =========================================================
-- EXERCISE 10
-- Type: Multiple Row Subquery using ALL
-- Definition:
-- ALL compares a value with all values returned by subquery.
-- Condition becomes true only if comparison succeeds for every row.
--
-- Problem Statement:
-- Find employees whose salary is greater than ALL salaries
-- of employees in HR department.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE dept_id = (
        SELECT dept_id
        FROM departments
        WHERE dept_name = 'HR'
    )
);



USE subquery_demo_db;

-- =========================================================
-- EXERCISE 1
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query runs first and returns one value: minimum salary.
-- Outer query returns employees matching that value.
--
-- Problem:
-- Find employee(s) who have the lowest salary in the company.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary = (
    SELECT MIN(salary)
    FROM employees
);

-- =========================================================
-- EXERCISE 2
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns earliest join date.
-- Outer query finds employees who joined on that date.
--
-- Problem:
-- Find employee(s) who joined the company first.
-- =========================================================
SELECT emp_id, emp_name, join_date
FROM employees
WHERE join_date = (
    SELECT MIN(join_date)
    FROM employees
);

-- =========================================================
-- EXERCISE 3
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns the maximum age.
-- Outer query finds employees whose age equals that value.
--
-- Problem:
-- Find the oldest employee(s) in the company.
-- =========================================================
SELECT emp_id, emp_name, age
FROM employees
WHERE age = (
    SELECT MAX(age)
    FROM employees
);

-- =========================================================
-- EXERCISE 4
-- Type: Multiple Row Single Column Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns dept_id values for departments in Pune or Hyderabad.
-- Outer query returns employees whose dept_id matches any returned value.
--
-- Problem:
-- Find employees working in departments located in Pune or Hyderabad.
-- =========================================================
SELECT emp_id, emp_name, dept_id, city
FROM employees
WHERE dept_id IN (
    SELECT dept_id
    FROM departments
    WHERE location IN ('Pune', 'Hyderabad')
);

-- =========================================================
-- EXERCISE 5
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns salary of Amit Sharma.
-- Outer query finds employees earning less than Amit Sharma.
--
-- Problem:
-- Find employees whose salary is less than Amit Sharma's salary.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary < (
    SELECT salary
    FROM employees
    WHERE emp_name = 'Amit Sharma'
);

-- =========================================================
-- EXERCISE 6
-- Type: Multiple Row Single Column Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns all dept_id values that have at least one female employee.
-- Outer query returns all employees in those departments.
--
-- Problem:
-- Find employees who work in departments where at least one female employee exists.
-- =========================================================
SELECT emp_id, emp_name, dept_id
FROM employees
WHERE dept_id IN (
    SELECT DISTINCT dept_id
    FROM employees
    WHERE gender = 'Female'
);

-- =========================================================
-- EXERCISE 7
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns average age of all employees.
-- Outer query filters employees older than that value.
--
-- Problem:
-- Find employees whose age is greater than the average age of all employees.
-- =========================================================
SELECT emp_id, emp_name, age
FROM employees
WHERE age > (
    SELECT AVG(age)
    FROM employees
);

-- =========================================================
-- EXERCISE 8
-- Type: Correlated Subquery
-- Clause: WHERE
-- Execution:
-- For each outer employee row, inner query counts employees in the same department.
-- Outer row is kept only if department size is more than 3.
--
-- Problem:
-- Find employees who work in departments having more than 3 employees.
-- =========================================================
SELECT e1.emp_id, e1.emp_name, e1.dept_id
FROM employees e1
WHERE (
    SELECT COUNT(*)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
) > 3;

-- =========================================================
-- EXERCISE 9
-- Type: Correlated Subquery
-- Clause: WHERE
-- Execution:
-- For each employee, inner query finds max salary in that employee's department.
-- Outer query keeps rows where employee salary equals that max value.
--
-- Problem:
-- Find the highest paid employee(s) in each department.
-- =========================================================
SELECT e1.emp_id, e1.emp_name, e1.dept_id, e1.salary
FROM employees e1
WHERE e1.salary = (
    SELECT MAX(e2.salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);

-- =========================================================
-- EXERCISE 10
-- Type: Correlated Subquery
-- Clause: WHERE
-- Execution:
-- For each employee, inner query finds minimum salary in that employee's department.
-- Outer query keeps rows where employee salary equals that min value.
--
-- Problem:
-- Find the lowest paid employee(s) in each department.
-- =========================================================
SELECT e1.emp_id, e1.emp_name, e1.dept_id, e1.salary
FROM employees e1
WHERE e1.salary = (
    SELECT MIN(e2.salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);

-- =========================================================
-- EXERCISE 11
-- Type: Scalar Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns average salary of Finance department.
-- Outer query returns employees whose salary is greater than that value.
--
-- Problem:
-- Find employees whose salary is greater than the average salary of Finance department.
-- =========================================================
SELECT emp_id, emp_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE dept_id = (
        SELECT dept_id
        FROM departments
        WHERE dept_name = 'Finance'
    )
);

-- =========================================================
-- EXERCISE 12
-- Type: Single Row Multiple Column Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns one row with two columns: dept_id and manager_id of Amit Sharma.
-- Outer query matches employees having the same pair.
--
-- Problem:
-- Find employees who have the same department and same manager as Amit Sharma.
-- =========================================================
SELECT emp_id, emp_name, dept_id, manager_id
FROM employees
WHERE (dept_id, manager_id) = (
    SELECT dept_id, manager_id
    FROM employees
    WHERE emp_name = 'Amit Sharma'
);

-- =========================================================
-- EXERCISE 13
-- Type: Multi Row Multiple Column Subquery
-- Clause: WHERE
-- Execution:
-- Inner query returns multiple (dept_id, manager_id) pairs from female employees.
-- Outer query returns employees whose pair matches any returned pair.
--
-- Problem:
-- Find employees whose department-manager combination is same as any female employee.
-- =========================================================
SELECT emp_id, emp_name, dept_id, manager_id
FROM employees
WHERE (dept_id, manager_id) IN (
    SELECT dept_id, manager_id
    FROM employees
    WHERE gender = 'Female'
      AND manager_id IS NOT NULL
);

-- =========================================================
-- EXERCISE 14
-- Type: Scalar Correlated Subquery
-- Clause: SELECT
-- Execution:
-- For each employee row, inner query counts how many employees are in the same department.
--
-- Problem:
-- Show each employee along with total number of employees in their department.
-- =========================================================
SELECT
    e1.emp_id,
    e1.emp_name,
    e1.dept_id,
    (
        SELECT COUNT(*)
        FROM employees e2
        WHERE e2.dept_id = e1.dept_id
    ) AS dept_employee_count
FROM employees e1;

-- =========================================================
-- EXERCISE 15
-- Type: Scalar Correlated Subquery
-- Clause: SELECT
-- Execution:
-- For each employee row, inner query finds highest salary in that employee's department.
--
-- Problem:
-- Show each employee with the highest salary of their department.
-- =========================================================
SELECT
    e1.emp_id,
    e1.emp_name,
    e1.salary,
    e1.dept_id,
    (
        SELECT MAX(e2.salary)
        FROM employees e2
        WHERE e2.dept_id = e1.dept_id
    ) AS dept_highest_salary
FROM employees e1;

-- =========================================================
-- EXERCISE 16
-- Type: Subquery in FROM
-- Clause: FROM
-- Execution:
-- Inner query creates department-wise minimum salary summary.
-- Outer query filters departments whose minimum salary is below 45000.
--
-- Problem:
-- Find departments where the minimum salary is less than 45000.
-- =========================================================
SELECT dept_id, min_salary
FROM (
    SELECT dept_id, MIN(salary) AS min_salary
    FROM employees
    GROUP BY dept_id
) AS dept_min_salary
WHERE min_salary < 45000;

-- =========================================================
-- EXERCISE 17
-- Type: Subquery in FROM
-- Clause: FROM
-- Execution:
-- Inner query creates city-wise employee count summary.
-- Outer query returns only cities having more than 3 employees.
--
-- Problem:
-- Find cities having more than 3 employees.
-- =========================================================
SELECT city, total_employees
FROM (
    SELECT city, COUNT(*) AS total_employees
    FROM employees
    GROUP BY city
) AS city_summary
WHERE total_employees > 3;

-- =========================================================
-- EXERCISE 18
-- Type: Scalar Subquery
-- Clause: HAVING
-- Execution:
-- Main query groups employees by city.
-- Inner query returns overall average salary.
-- HAVING keeps only those cities whose average salary is above overall average salary.
--
-- Problem:
-- Find cities whose average salary is greater than the overall average salary.
-- =========================================================
SELECT city, AVG(salary) AS avg_salary
FROM employees
GROUP BY city
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);

-- =========================================================
-- EXERCISE 19
-- Type: Correlated Subquery with EXISTS
-- Clause: WHERE
-- Execution:
-- For each employee in outer query, inner query checks whether
-- another employee exists in the same department with higher salary.
-- If yes, current row is returned.
--
-- Problem:
-- Find employees who are not the highest paid in their department.
-- =========================================================
SELECT e1.emp_id, e1.emp_name, e1.dept_id, e1.salary
FROM employees e1
WHERE EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
      AND e2.salary > e1.salary
);

-- =========================================================
-- EXERCISE 20
-- Type: Correlated Subquery with NOT EXISTS
-- Clause: WHERE
-- Execution:
-- For each department row, inner query checks whether any employee exists in that department.
-- If no employee exists, that department is returned.
--
-- Problem:
-- Find departments that currently have no employees.
-- =========================================================
SELECT d.dept_id, d.dept_name, d.location
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.dept_id = d.dept_id
);