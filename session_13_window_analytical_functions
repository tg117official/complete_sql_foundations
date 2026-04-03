-- ============================================================
-- ANALYTICAL WINDOW FUNCTIONS DEMO SCRIPT - MYSQL 8.0+
-- ============================================================
-- Functions covered:
-- 1. LAG()
-- 2. LEAD()
-- 3. FIRST_VALUE()
-- 4. LAST_VALUE()
--
-- Important:
-- These functions depend heavily on ORDER BY inside OVER()
-- because previous, next, first, last all depend on row sequence.
-- ============================================================

-- ============================================================
-- DATABASE SETUP
-- ============================================================

DROP DATABASE IF EXISTS mysql_analytical_window_demo;
CREATE DATABASE mysql_analytical_window_demo;
USE mysql_analytical_window_demo;

-- ============================================================
-- TABLE CREATION
-- ============================================================

DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(50) NOT NULL,
    department VARCHAR(50) NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2) NOT NULL,
    performance_score INT NOT NULL
);

-- ============================================================
-- INSERT SAMPLE DATA
-- ============================================================

INSERT INTO employees (emp_id, emp_name, department, hire_date, salary, performance_score) VALUES
(1,  'Amit',   'Engineering', '2021-01-10', 75000, 88),
(2,  'Neha',   'Engineering', '2021-03-15', 78000, 91),
(3,  'Rohit',  'Engineering', '2021-04-20', 62000, 79),
(4,  'Sneha',  'Engineering', '2021-06-11', 95000, 94),
(5,  'Karan',  'Engineering', '2022-01-05', 62000, 82),

(6,  'Pooja',  'Sales',       '2021-02-17', 54000, 85),
(7,  'Vikas',  'Sales',       '2021-05-23', 54000, 81),
(8,  'Ritu',   'Sales',       '2020-12-01', 88000, 92),
(9,  'Manoj',  'Sales',       '2022-07-19', 67000, 77),
(10, 'Priya',  'Sales',       '2023-01-25', 50000, 74),

(11, 'Anjali', 'HR',          '2021-08-10', 58000, 84),
(12, 'Suresh', 'HR',          '2020-09-14', 82000, 89),
(13, 'Meena',  'HR',          '2022-02-11', 52000, 76),
(14, 'Tarun',  'HR',          '2023-04-09', 52000, 73),
(15, 'Kavita', 'HR',          '2022-11-30', 61000, 87);

-- ============================================================
-- BASE DATA CHECK
-- ============================================================

SELECT * FROM employees;

-- ============================================================
-- 1) LAG()
-- ============================================================
-- LAG() returns value from previous row based on the ORDER BY
-- inside OVER().
--
-- Important:
-- Without ORDER BY, previous row has no clear meaning.
--
-- Here we compare current employee salary with previous employee
-- salary based on hire_date across the full table.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LAG(salary) OVER (
        ORDER BY hire_date
    ) AS previous_salary
FROM employees
ORDER BY hire_date;

-- ============================================================
-- 2) LAG() WITH PARTITION BY
-- ============================================================
-- Here previous salary is calculated inside each department only.
-- So department becomes the partition/window.
--
-- For every department, ordering happens by hire_date.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LAG(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS previous_salary_in_department
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 3) LEAD()
-- ============================================================
-- LEAD() returns value from next row based on the ORDER BY
-- inside OVER().
--
-- Here we fetch next employee salary based on hire_date
-- across the full table.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LEAD(salary) OVER (
        ORDER BY hire_date
    ) AS next_salary
FROM employees
ORDER BY hire_date;

-- ============================================================
-- 4) LEAD() WITH PARTITION BY
-- ============================================================
-- Here next salary is calculated only inside each department.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LEAD(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS next_salary_in_department
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 5) LAG() + DIFFERENCE FROM PREVIOUS ROW
-- ============================================================
-- This is a very practical analytical use case.
-- We compare current salary with previous salary.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LAG(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS previous_salary,
    salary - LAG(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS difference_from_previous
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 6) LEAD() + DIFFERENCE FROM NEXT ROW
-- ============================================================
-- Compare current salary with next salary inside each department.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LEAD(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS next_salary,
    LEAD(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) - salary AS difference_with_next
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 7) FIRST_VALUE()
-- ============================================================
-- FIRST_VALUE() returns the first value in the ordered window.
--
-- Here salary is ordered DESC inside each department,
-- so first value becomes highest salary in that department.
--
-- Important:
-- FIRST_VALUE() depends on ORDER BY.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS highest_salary_in_department
FROM employees
ORDER BY department, salary DESC;

-- ============================================================
-- 8) FIRST_VALUE() USING HIRE DATE
-- ============================================================
-- Here we fetch the earliest hired employee salary in each department
-- by ordering hire_date ascending.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS earliest_hired_emp_salary_in_department
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 9) LAST_VALUE()
-- ============================================================
-- LAST_VALUE() returns the last value in the CURRENT FRAME.
--
-- Important caution:
-- LAST_VALUE() often confuses students.
-- If frame is not explicitly defined, result may not mean
-- "last row of full partition".
--
-- So to get the true last value in the full partition,
-- explicitly define the frame:
-- ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
--
-- Here salary is ordered DESC inside each department,
-- so last value becomes lowest salary in that department.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_salary_in_department
FROM employees
ORDER BY department, salary DESC;

-- ============================================================
-- 10) LAST_VALUE() USING HIRE DATE
-- ============================================================
-- Here we fetch the salary of the latest hired employee
-- in each department.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS latest_hired_emp_salary_in_department
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 11) COMBINED ANALYTICAL FUNCTIONS IN ONE QUERY
-- ============================================================
-- This query shows that multiple analytical window functions
-- can be used together in one SELECT.
--
-- All functions here use the same window specification so that
-- output remains easy to understand in classroom teaching.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    salary,

    LAG(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS previous_salary,

    LEAD(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS next_salary,

    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS first_salary_in_department_by_hire_date,

    LAST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_salary_in_department_by_hire_date

FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- 12) OPTIONAL: SHOW CURRENT, PREVIOUS, NEXT EMPLOYEE NAME
-- ============================================================
-- LAG() and LEAD() can also return text columns.
-- ============================================================

SELECT
    emp_id,
    emp_name,
    department,
    hire_date,
    LAG(emp_name) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS previous_employee_name,
    LEAD(emp_name) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS next_employee_name
FROM employees
ORDER BY department, hire_date;

-- ============================================================
-- IMPORTANT NOTES
-- ============================================================
-- 1. LAG() and LEAD() need ORDER BY for meaningful previous/next row
-- 2. FIRST_VALUE() and LAST_VALUE() also depend on ORDER BY
-- 3. LAST_VALUE() often needs explicit frame definition
-- 4. PARTITION BY creates separate logical windows
-- 5. Outer ORDER BY controls final display of rows
-- ============================================================
