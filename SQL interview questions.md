## SQL Frequently Asked Interview Questions


### 1) Write a SQL query to find the second highest salary from the table emp

```sql
SELECT MAX(salary) AS second_max
FROM employee
WHERE salary NOT IN (SELECT MAX(salary) FROM employee);

OR

SELECT MAX(salary)
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```


### 2) Write a SQL query to find the numbers which consecutively occurs 3 times.

```sql
WITH cte AS (
    SELECT 
        num,
        LEAD(num, 1) OVER (ORDER BY id) AS next_num,
        LAG(num, 1) OVER (ORDER BY id) AS prev_num
    FROM numbers
)
SELECT num, COUNT(*) AS repetition
FROM cte 
WHERE num = next_num AND num = prev_num
GROUP BY num;
```


### 3) Write a SQL query to find the days when temperature was higher than its previous dates

```sql
WITH cte AS (
    SELECT 
        days,
        temp,
        LAG(temp, 1) OVER () AS temp1
    FROM temperature
)
SELECT days
FROM cte 
WHERE temp < temp1;
```


### 4) Write a SQL query to delete Duplicate rows in a table.

```sql
SELECT name, email, COUNT(name)  
FROM student_contacts  
GROUP BY name, email  -- Finding duplicate records using HAVING (COUNT)
HAVING COUNT(name) > 1;

-- Finding duplicate records
SELECT * 
FROM student_contacts AS s1
JOIN student_contacts AS s2  
ON s1.id > s2.id AND s1.email = s2.email;

-- deleting duplicate record
DELETE s1 
FROM student_contacts AS s1
JOIN student_contacts AS s2
ON s1.id > s2.id AND s1.email = s2.email; -- ID should be either greater than or lesser than as we need to keep one ID and delete the other.

-- Delete Using Row_number() function 
DELETE FROM student_contacts WHERE id IN (
    SELECT id
    FROM (
        SELECT id, email,
            ROW_NUMBER() OVER(PARTITION BY name ORDER BY name) AS row_num
        FROM student_contacts
    ) abc
    WHERE row_num > 1
);
```


### 5)  Write a SQL query to display year on year growth for each product. (Column name – transaction_id, Product_id, transaction_date, spend). Output will have year, product_id & yoy_growth.

```sql
SELECT 
    product_id,
    YEAR(transaction_date) AS p_year,
    spend,
    (spend - LAG(spend, 1) OVER(PARTITION BY product_id ORDER BY transaction_date)) * 100
    / LAG(spend, 1) OVER(PARTITION BY product_id ORDER BY transaction_date) AS yoy_growth
FROM product;
```


### 6) Write a SQL query to find rolling average of posts on daily bais for each user_id.(Column_name – user_id, date, post_count). Round up the average upto two decimal places.

```sql
SELECT 
    *,
    ROUND(AVG(post_count) OVER (ORDER BY user_id ROWS BETWEEN 1 PRECEDING AND CURRENT ROW), 2) AS rolling_avg
FROM user; --to find rolling avg for EACH user id add partition by user_id
```


### 7) QUES Write a SQL query to get emp id and department for each employee who recently joined the organization and still in working. (column - emp id, first name, last name, date of join, date of exit , department.)

```sql
SELECT 
    *
FROM 
    table_name
WHERE 
    date_of_join >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)  -- Filter employees who joined within the last month
    AND (date_of_exit IS NULL OR date_of_exit > CURDATE()); -- Filter employees with no exit date or an exit date in the future
```


### 8) How many rows will come in outputs of Left, Right, Inner and outer join from two tables having duplicate rows.

```sql
--values
INSERT INTO demo_l VALUES
(1),(2),(4),(6),(null),(1);
INSERT INTO demo_r VALUES
(1),(0),(2),(3),(5),(2);
```
```sql
SELECT *
FROM demo_l l
LEFT JOIN demo_r r  -- Left join 
ON l.id = r.id;

SELECT *
FROM demo_l l
RIGHT JOIN demo_r r  -- Right join 
ON l.id = r.id;

SELECT *
FROM demo_l l
JOIN demo_r r  -- Inner join 
ON l.id = r.id;
```


### 9) Write a MySQL query to get mean, median and mode for earning? (Column – Emp_id, salary)

```sql
-- mean is the average values
-- mode is the frequency of occurence. count the repetitive numbers. no. with highest count is mode.
-- Finding median

SET @row_index := -1;

WITH cte AS (
    SELECT 
        @row_index := @row_index + 1 AS rowindex,  -- Sets an incremental index starting from 0
        salary
    FROM 
        employee
    ORDER BY 
        salary
)

SELECT AVG(salary) AS median
FROM cte
WHERE rowindex IN (FLOOR(@row_index / 2), CEIL(@row_index / 2));

-- If the list has an odd number of items, e.g., 3, then rowindex will read 2 (starting from 0, 1, 2).
-- Rowindex value will be divided by 2, which gives 1, and the value at 1 will be the median (starting from 0, 1).
-- For an even-numbered list, we will have 2 middle values; FLOOR and CEIL will convert them to whole numbers, and that will be the index value.
-- The average will be taken out of both numbers that lie at the index value, and that will be the median.
```

### 10) find output of count fns. in various forms

```sql
select 
	  count(*),				-- count fns counts the number of rows.
    count(1),
    count(2),
    count(col_name)			-- but if specified with a col name which has null in it, then null is not counted.
from dummy;
```

### 11) Employee earning more than their managers

```sql
--table
CREATE TABLE employee2 (
    id INT,
    name VARCHAR(20),
    salary INT,
    managerid INT
);

INSERT INTO employee2 VALUES
(1, 'joe', 7000, 3),
(2, 'henry', 8000, 4),
(3, 'sam', 6000, NULL),
(4, 'max', 9000, NULL);
```

```sql
SELECT e.name
FROM employee2 e
JOIN employee2 m
-- on e.id = m.managerid    this is incorrect for this ques as it matches employee id with managers id
ON e.managerid = m.id   -- Correct join condition to match employee's manager's id with manager's id
WHERE e.salary > m.salary;
```

### 12) To check the datatype 

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'orders'; -- To check the data type of all columns at once

SELECT data_type
FROM information_schema.columns
WHERE table_name = 'orders'
AND column_name = 'customerid'; -- To check the data type of a specific column ('customerid' in this case)
```

### 13) Use Alter table and update table. Find age from DOB.

```sql
-- crete table
CREATE TABLE employee7 (
    empid INT,
    empname VARCHAR(20),
    dob DATE
);

-- insert values
INSERT INTO employee7 VALUES
(1, 'abc', '1993-10-11'),
(2, 'xyz', '1995-10-11');

-- adding column 'age' to the table
ALTER TABLE employee7
ADD COLUMN age INT;

-- updating age column to find age from DOB
UPDATE employee7
SET age = TIMESTAMPDIFF(YEAR, dob, CURDATE());
```





