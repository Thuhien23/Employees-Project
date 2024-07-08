
## Conduct exploratory data analysis (EDA) to gain deeper insights into the dataset, utilizing MySQL.
**How many departments are there in the “employees” database?**

**Dataset**: https://tinyurl.com/56nt4vcc


```sql
SELECT 
    COUNT(DISTINCT dept_no) as number_dept
FROM
    employees.dept_emp;
```


**What is the total amount of money spent on salaries for all contracts starting after the 1st of January 1997?**

```sql
SELECT
    SUM(salary) as total_salary
FROM
    employees.salaries
WHERE
    from_date > '1997-01-01';
```


**What is the average annual salary paid to employees who started after the 1st of January 1997?**

```sql
SELECT 
    AVG(salary) AS avg_salary
FROM
    employees.salaries
WHERE
    from_date > '1997-01-01';
```



**Round the average amount of money spent on salaries for all contracts that started after the 1st of January 1997 to a precision of cents.**

```sql
SELECT 
    ROUND((salary), 2) AS avg_salary
FROM
    employees.salaries
WHERE
    from_date > '1997-01-01';
```

**Extract a list containing information about all managers’ employee number, first and last name, department number, and hire date.**

```sql
SELECT 
    e.emp_no, e.first_name, e.last_name, dm.dept_no, e.hire_date
FROM
    employees.employees e
        JOIN
    employees.dept_manager dm ON e.emp_no = dm.emp_no;
```



**Join the 'employees' and the 'dept_manager' tables to return a subset of all the employees whose last name is Markovitch. See if the output contains a manager with that name.**

```sql
SELECT 
    *
FROM
    employees.employees e
        LEFT JOIN
    employees.dept_manager dm ON e.emp_no = dm.emp_no
WHERE
    e.last_name = 'Markovitch'
ORDER BY dm.dept_no DESC , e.emp_no;
```

**Select the first and last name, the hire date, and the job title of all employees whose first name is “Margareta” and have the last name “Markovitch”.**

```sql
SELECT 
    e.first_name, e.last_name, e.hire_date, t.title
FROM
    employees.employees e
        JOIN
    employees.titles t ON e.emp_no = t.emp_no
WHERE
    e.first_name = 'Margareta'
        AND e.last_name = 'Markovitch'
ORDER BY e.emp_no;
```

**Select all managers’ first and last name, hire date, job title, start date, and department name.**

```sql
SELECT 
    e.first_name,
    e.last_name,
    e.hire_date,
    t.title,
    dm.from_date,
    d.dept_name
FROM
    employees.employees e
        JOIN
    employees.dept_manager dm ON e.emp_no = dm.emp_no
        JOIN
    employees.departments d ON dm.dept_no = d.dept_no
        JOIN
    employees.titles t ON e.emp_no = t.emp_no
WHERE
    t.title = 'Manager';
```

**Extract the information about all department managers who were hired between the 1st of January 1990 and the 1st of January 1995.**

```sql
SELECT 
    *
FROM
    employees.dept_manager dm
WHERE
    dm.emp_no IN (SELECT 
            e.emp_no
        FROM
            employees.employees e
        WHERE
            e.hire_date BETWEEN '1990-01-01' AND '1995-01-01');
```

**Select the entire information for all employees whose job title is “Assistant Engineer”.**

```sql
SELECT 
    *
FROM
    employees.employees e
WHERE
    EXISTS( SELECT 
            *
        FROM
            employees.titles t
        WHERE
            t.emp_no = e.emp_no
                AND t.title = 'Assistant Engineer');
```

**Assign employee number 110022 as a manager to all employees from 10001 and 10020, and employee number 110039 as a manager to all employee from 10021 to 10040**

```sql
SELECT 
    A.*
FROM
    (SELECT 
        e.emp_no AS employeer_ID,
            MIN(de.dept_no) AS department_code,
            (SELECT 
                    emp_no
                FROM
                    employees.dept_emp
                WHERE
                    emp_no = 110022) AS manager_ID
    FROM
        employees.employees e
    JOIN employees.dept_emp de ON e.emp_no = de.emp_no
    WHERE
        e.emp_no <= 10020
    GROUP BY e.emp_no
    ORDER BY e.emp_no) AS A 
UNION SELECT 
    B.*
FROM
    (SELECT 
        e.emp_no AS employeer_ID,
            MIN(de.dept_no) AS department_code,
            (SELECT 
                    emp_no
                FROM
                    employees.dept_emp
                WHERE
                    emp_no = 110039) AS manager_ID
    FROM
        employees.employees e
    JOIN employees.dept_emp de ON e.emp_no = de.emp_no
    WHERE
        e.emp_no > 10020
    GROUP BY e.emp_no
    ORDER BY e.emp_no
    LIMIT 10) AS B
```

**Obtain a result set containing the employee number, first name, and last name of all employees with a number higher than 109990. Create a fourth column in the query, indicating whether this employee is also a manager, according to the data provided in the *dept_manager* table, or a regular employee.**

```sql
SELECT 
    e.emp_no,
    e.first_name,
    e.last_name,
    CASE
        WHEN dm.emp_no IS NOT NULL THEN 'Manager'
        ELSE 'Employee'
    END AS 'is_manager'
FROM
    employees.employees e
        LEFT JOIN
    employees.dept_manager dm ON dm.emp_no = e.emp_no
WHERE
    e.emp_no > 109990;
```

**Extract a dataset containing the following information about the managers: employee number, first name, and last name. Add two columns at the end – one showing the difference between the maximum and minimum salary of that employee, and another one saying whether this salary raise was higher than $30,000 or NOT.**

```sql
SELECT 
    e.emp_no,
    e.first_name,
    e.last_name,
    MAX(s.salary) - MIN(s.salary) AS salary_difference,
    CASE
        WHEN MAX(s.salary) - MIN(s.salary) > 30000 THEN 'Salary was raised by more then $30,000'
        ELSE 'Salary was NOT raised by more then $30,000'
    END AS salary_raise
FROM
    employees.employees e
        JOIN
    employees.dept_manager dm ON e.emp_no = dm.emp_no
        JOIN
    employees.salaries s ON dm.emp_no = s.emp_no
GROUP BY e.emp_no;
```

**Extract the employee number, first name, and last name of the first 100 employees, and add a fourth column, called “current_employee” saying “Is still employed” if the employee is still working in the company, or “Not an employee anymore” if they aren’t.**

```sql
SELECT 
    e.emp_no,
    e.first_name,
    e.last_name,
    CASE
        WHEN MAX(de.to_date) > SYSDATE() THEN 'Is still employed'
        ELSE 'Not an employee anymore'
    END AS current_employee
FROM
    employees.employees e
        JOIN
    employees.dept_emp de ON e.emp_no = de.emp_no
GROUP BY e.emp_no
LIMIT 100;
```

**Write a query that upon execution, assigns a row number to all managers we have information for in the "employees" database (regardless of their department).**

```sql
SELECT
    emp_no,
    dept_no,
    ROW_NUMBER() OVER (ORDER BY emp_no) AS row_num
FROM
employees.dept_manager;
```

**Write a query that upon execution, assigns a sequential number for each employee number registered in the "employees" table. Partition the data by the employee's first name and order it by their last name in ascending order (for each partition).**

```sql
SELECT
emp_no,
first_name,
last_name,
ROW_NUMBER() OVER (PARTITION BY first_name ORDER BY last_name) AS row_num
FROM
employees.employees;
```

**Use window functions to add the following two columns to the final output:**

- a column containing the row number of each row from the obtained dataset, starting from 1.
- a column containing the sequential row numbers associated to the rows for each manager, where their highest salary has been given a number equal to the number of rows in the given partition, and their lowest - the number 1.

```sql
SELECT
dm.emp_no,
    salary,
    ROW_NUMBER() OVER () AS row_num1,
    ROW_NUMBER() OVER (PARTITION BY emp_no ORDER BY salary DESC) AS row_num2
FROM
employees.dept_manager dm
    JOIN 
    employees.salaries s ON dm.emp_no = s.emp_no
ORDER BY row_num1, emp_no, salary ASC;
```

**Write a query containing a window function to obtain all salary values that employee number 10560 has ever signed a contract for.**

Order and display the obtained salary values from highest to lowest.

```sql
SELECT 
    emp_no, salary, ROW_NUMBER() OVER w AS rank_salary
FROM
    employees.salaries
WHERE
    emp_no = 10560
WINDOW w AS (PARTITION BY emp_no ORDER BY salary DESC);
```

**Write a query that upon execution, displays the number of salary contracts that each manager has ever signed while working in the company.**

```sql
SELECT 
    dm.emp_no, COUNT(s.salary) AS num_contract
FROM
    employees.dept_manager dm
        JOIN
    employees.salaries s ON dm.emp_no = s.emp_no
GROUP BY dm.emp_no
ORDER BY dm.emp_no;
```

**Write a query that upon execution retrieves a result set containing all salary values that employee 10560 has ever signed a contract for. Use a window function to rank all salary values from highest to lowest in a way that equal salary values bear the same rank and that gaps in the obtained ranks for subsequent rows are allowed.**

```sql
SELECT 
    emp_no, salary, rank() over w as rank_salary
FROM
    employees.salaries
WHERE
    emp_no = 10560
WINDOW w AS(PARTITION BY emp_no ORDER BY salary desc);
```

**21.Write a query that upon execution retrieves a result set containing all salary values that employee 10560 has ever signed a contract for. Use a window function to rank all salary values from highest to lowest in a way that equal salary values bear the same rank and that gaps in the obtained ranks for subsequent rows are not allowed.**

```sql
SELECT 
    emp_no, salary, dense_rank() over w as rank_salary
FROM
    employees.salaries
WHERE
    emp_no = 10560
WINDOW w AS(PARTITION BY emp_no ORDER BY salary desc);
```

**Write a query that ranks the salary values in descending order of all contracts signed by employees numbered between 10500 and 10600 inclusive. Let equal salary values for one and the same employee bear the same rank. Also, allow gaps in the ranks obtained for their subsequent rows.**

```sql
SELECT 
    e.emp_no, s.salary, rank() over w as employee_salary_ranking
FROM
    employees.employees e join employees.salaries s on e.emp_no = s.emp_no 
    where e.emp_no between 10500 and 10600
WINDOW w AS(PARTITION BY e.emp_no ORDER BY s.salary desc);
```

**Write a query that ranks the salary values in descending order of the following contracts from the "employees" database:**

- contracts that have been signed by employees numbered between 10500 and 10600 inclusive.
- contracts that have been signed at least 4 full-years after the date when the given employee was hired in the company for the first time.

In addition, let equal salary values of a certain employee bear the same rank. Do not allow gaps in the ranks obtained for their subsequent rows.

```sql
SELECT
    e.emp_no,
    DENSE_RANK() OVER w as employee_salary_ranking,
    s.salary,
    e.hire_date,
    s.from_date,
    (YEAR(s.from_date) - YEAR(e.hire_date)) AS years_from_start
FROM
employees.employees e
JOIN
    employees.salaries s ON s.emp_no = e.emp_no
    AND YEAR(s.from_date) - YEAR(e.hire_date) >= 4
WHERE e.emp_no BETWEEN 10500 AND 10600
WINDOW w as (PARTITION BY e.emp_no ORDER BY s.salary DESC);
```

**Write a query that can extract the following information from the "employees" database:**

- the salary values (in ascending order) of the contracts signed by all employees numbered between 10500 and 10600 inclusive
- a column showing the previous salary from the given ordered list
- a column showing the subsequent salary from the given ordered list
- a column displaying the difference between the current salary of a certain employee and their previous salary
- a column displaying the difference between the next salary of a certain employee and their current salary

Limit the output to salary values higher than $80,000 only.

Also, to obtain a meaningful result, partition the data by employee number.

```sql
SELECT 
	emp_no, salary, LAG(salary) OVER w AS previous_salary, 
	LEAD(salary) OVER w AS next_salary, 
	salary-LAG(salary) OVER w AS diff_salary_current_previous, 
	LEAD(salary) OVER w - salary AS diff_salary_next_current	
FROM
    employees.salaries
WHERE emp_no BETWEEN 10500 AND 10600 and salary > 80000 
WINDOW w as (PARTITION BY emp_no ORDER BY salary DESC);
```

**Create a query that upon execution returns a result set containing the employee numbers, contract salary values, start, and end dates of the first ever contracts that each employee signed for the company.**

```sql
SELECT 
    s.emp_no, s.salary, s.from_date, s.to_date
FROM
    employees.salaries s
WHERE
    s.from_date = (SELECT 
            MIN(from_date)
        FROM
            employees.salaries
        WHERE
            emp_no = s.emp_no);
```

**26. Consider the employees' contracts that have been signed after the 1st of January 2000 and terminated before the 1st of January 2002 (as registered in the "dept_emp" table).**

Create a MySQL query that will extract the following information about these employees:

- Their employee number
- The salary values of the latest contracts they have signed during the suggested time period
- The department they have been working in (as specified in the latest contract they've signed during the suggested time period)
- Use a window function to create a fourth field containing the average salary paid in the department the employee was last working in during the suggested time period. Name that field "average_salary_per_department".

```sql
SELECT
    de2.emp_no, d.dept_name, s2.salary, AVG(s2.salary) OVER w AS average_salary_per_department
FROM
    (SELECT
    de.emp_no, de.dept_no, de.from_date, de.to_date
FROM
    employees.dept_emp de
        JOIN
(SELECT
emp_no, MAX(from_date) AS from_date
FROM
employees.dept_emp
GROUP BY emp_no) de1 ON de1.emp_no = de.emp_no
WHERE
    de.to_date < '2002-01-01'
AND de.from_date > '2000-01-01'
AND de.from_date = de1.from_date) de2
JOIN
    (SELECT
    s1.emp_no, s.salary, s.from_date, s.to_date
FROM
    employees.salaries s
    JOIN
    (SELECT
emp_no, MAX(from_date) AS from_date
FROM
employees.salaries
    GROUP BY emp_no) s1 ON s.emp_no = s1.emp_no
WHERE
    s.to_date < '2002-01-01'
AND s.from_date > '2000-01-01'
AND s.from_date = s1.from_date) s2 ON s2.emp_no = de2.emp_no
JOIN
    employees.departments d ON d.dept_no = de2.dept_no
GROUP BY de2.emp_no, d.dept_name
WINDOW w AS (PARTITION BY de2.dept_no)
ORDER BY de2.emp_no, salary;
```

**Use a CTE (a Common Table Expression) and a SUM() function in the SELECT statement in a query to find out how many male employees have never signed a contract with a salary value higher than or equal to the all-time company salary average.** 

```sql
WITH cte AS (
SELECT AVG(salary) AS avg_salary FROM employees.salaries
)
SELECT
SUM(CASE WHEN s.salary < c.avg_salary THEN 1 ELSE 0 END) AS no_salaries_below_avg,
COUNT(s.salary) AS no_of_salary_contracts
FROM employees.salaries s JOIN employees.employees e ON s.emp_no = e.emp_no AND e.gender = 'M' JOIN cte c;
```

**Use a CTE (a Common Table Expression) and (at least one) COUNT() function in the SELECT statement of a query to find out how many male employees have never signed a contract with a salary value higher than or equal to the all-time company salary average.**

```sql
WITH cte AS (
SELECT AVG(salary) AS avg_salary FROM employees.salaries
)
SELECT
COUNT(CASE WHEN s.salary < c.avg_salary THEN s.salary ELSE NULL END) AS no_salaries_below_avg_w_count,
COUNT(s.salary) AS no_of_salary_contracts
FROM employees.salaries s JOIN employees.employees e ON s.emp_no = e.emp_no AND e.gender = 'M' JOIN cte c;
```

**Use two common table expressions and a SUM() function in the SELECT statement of a query to obtain the number of male employees whose highest salaries have been below the all-time average.** 

```sql
WITH cte1 AS (
SELECT AVG(salary) AS avg_salary FROM employees.salaries
),
cte2 AS (
SELECT s.emp_no, MAX(s.salary) AS max_salary
FROM employees.salaries s
JOIN employees.employees e ON e.emp_no = s.emp_no AND e.gender = 'M'
GROUP BY s.emp_no
)
SELECT
SUM(CASE WHEN c2.max_salary < c1.avg_salary THEN 1 ELSE 0 END) AS highest_salaries_below_avg
FROM employees.employees e
JOIN cte2 c2 ON c2.emp_no = e.emp_no
JOIN cte1 c1;
```

**Use two common table expressions and a COUNT() function in the SELECT statement of a query to obtain the number of male employees whose highest salaries have been below the all-time average.** 

```sql
WITH cte_avg_salary AS (
SELECT AVG(salary) AS avg_salary FROM employees.salaries
),
cte_m_highest_salary AS (
SELECT s.emp_no, MAX(s.salary) AS max_salary
FROM employees.salaries s JOIN employees.employees e ON e.emp_no = s.emp_no AND e.gender = 'M'
GROUP BY s.emp_no
)
SELECT
COUNT(CASE WHEN c2.max_salary < c1.avg_salary THEN c2.max_salary ELSE NULL END) AS max_salary
FROM employees.employees e
JOIN cte_m_highest_salary c2 ON c2.emp_no = e.emp_no
JOIN cte_avg_salary c1;
```
