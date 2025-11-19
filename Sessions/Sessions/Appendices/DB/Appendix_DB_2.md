# DML
**(Data Manipulation Language)** is about **manipulating the actual data inside structures**—adding, modifying, or retrieving rows.
#### INSERT
Add new rows into a table.   
``` sql
INSERT INTO Employees (EmployeeID, FirstName, LastName, Salary) VALUES (1, 'John', 'Doe', 4500.00);  

-- Insert multiple rows 
INSERT INTO Employees (EmployeeID, FirstName, LastName, Salary) VALUES
    (2, 'Jane', 'Smith', 5500.00),     
	(3, 'Bob', 'Brown', 6000.00);

```
### UPDATE
Modify existing data in a table.  
``` sql
-- Give a raise to John Doe 
UPDATE Employees SET Salary = 5000.00 WHERE EmployeeID = 1; 
 -- Increase salary by 10% for everyone 
UPDATE Employees SET Salary = Salary * 1.1;
```
### DELETE
Remove rows from a table.  
``` sql
-- Delete a specific employee 
DELETE FROM Employees WHERE EmployeeID = 3;  
-- Delete all employees (like TRUNCATE, but slower and logs each deletion) 
DELETE FROM Employees;
```
### SELECT
Retrieve data from a table (most commonly used).  
``` sql
-- Select all columns S
SELECT * FROM Employees;  
-- Select specific columns 
SELECT FirstName, LastName, Salary FROM Employees;  
-- Select with condition 
SELECT * FROM Employees WHERE Salary > 5000;  
-- Aggregate example 
SELECT COUNT(*) AS NumEmployees FROM Employees;
```
## Soft delete vs Hard deletes
### Hard Delete

- Standard `DELETE` or `TRUNCATE` operation.
- Data is physically removed from the table. 
```sql
DELETE FROM Employees WHERE EmployeeID = 5;
```

- Once committed, the data is gone (unless you have backups).
**Downside:** You lose historical information, which might be needed for auditing or regulatory compliance.
### Soft Delete

- Instead of physically removing a row, we **mark it as deleted** using a flag/column.
- The row still exists in the database but is ignored in normal queries.

``` sql  
-- "Delete" an employee softly 
UPDATE Employees SET IsDeleted = TRUE WHERE EmployeeID = 5;  
-- Query only active employees 
SELECT * FROM Employees WHERE IsDeleted = FALSE;
```

**Advantages:**
- Preserves history for **auditing**.
- Easy to **restore accidentally deleted data**.
- Compliant with **data retention policies**.

**Disadvantages:**

- Slightly more complex queries (`WHERE IsDeleted = FALSE` everywhere).
- Table can grow large over time; need archival strategies.
>[!hint]
> Why Companies Rarely Truly Delete Data
>Most companies implement **Data Retention Policies** for legal, business, and technical reasons:
>
>**Legal / Compliance:**
 >	Regulations like GDPR, HIPAA, SOX, or local laws often **require maintaining certain records for X years**.    
>	Example: Financial transactions must be stored for 7 years.
>**Auditing and Security:**
>	Companies need to **audit past actions**, track changes, or investigate incidents.
>**Business Intelligence / Analytics:**
>	Historical data is useful for trends, reporting, or ML models.
>**Accidental Deletion Recovery:**
>	Soft deletes allow recovery without restoring from backups.

# DQL

## Joins 

| EmployeeID | Name | DepartmentID |
| ---------- | ---- | ------------ |
| 1          | John | 10           |
| 2          | Sara | 20           |
| 3          | Bob  | 10           |
| 4          | Heba | 999          |

|DepartmentID|DepartmentName|
|---|---|
|10|IT|
|20|HR|
|30|Finance|
![[Pasted image 20251115210958.png]]
### INNER JOIN
``` sql
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d
    ON e.DepartmentID = d.DepartmentID;
```

No Finance because no employee is in Dept 30.
purple section

| Join Type | Keeps all rows from        | Missing matches become | Section |
| --------- | -------------------------- | ---------------------- | ------- |
| INNER     | Only matching rows         | Excluded               | purple  |
| LEFT      | Left table (heba)          | NULL                   | blue    |
| RIGHT     | Right table (finance)      | NULL                   | pink    |
| FULL      | Both tables                | NULL                   | all     |
| CROSS     | Neither — all combinations | N/A                    | N/A     |
### CROSS JOIN
If Employees has 3 rows and Departments has 3 rows → 9 results.
has no where clause. 
kills performance

> [!hint]
>  Using WHERE clause to match IDs instead of an explicit INNER JOIN can impact performance, though the difference is often minimal with modern query optimizers.

### SELF JOIN
``` sql
SELECT e.Name AS Employee, m.Name AS Manager
FROM Employees e
LEFT JOIN Employees m
    ON e.ManagerID = m.EmployeeID;

```

## Aggregation
means **summarizing data** — combining multiple rows into a single value (**row**) using an **aggregate function**.

| Function  |
| --------- |
| `COUNT()` |
| `SUM()`   |
| `AVG()`   |
| `MIN()`   |
| `MAX()`   |
``` sql
SELECT 
    COUNT(*) AS TotalEmployees,
    AVG(Salary) AS AvgSalary,
    MAX(Salary) AS HighestSalary,
    MIN(Salary) AS LowestSalary
FROM Employees;

```

### Group by
This gives **one row per** **group by Item** (unique)

``` sql
-- Average salary per department
SELECT 
    DepartmentID,
    AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY DepartmentID;

```
### Having
`HAVING` is like a `WHERE` for groups — it **filters** **after** **aggregation**.
> [!caution]
>HAVING clause is evaluated before the SELECT clause this ia why you can’t use aliases in there.


``` sql
-- Departments with more than 5 employees
SELECT 
    DepartmentID,
    COUNT(*) AS NumEmployees
FROM Employees
GROUP BY DepartmentID
HAVING COUNT(*) > 5;

```

### Key notes
- Columns in `SELECT` that are **not aggregated** must appear in `GROUP BY`.  (In case of aggregation)
- `WHERE` filters **before** aggregation; `HAVING` filters **after**.  
- Aggregates ignore `NULL` values (except `COUNT(*)`, which counts all rows).
## Sub-Queries
> [!definition]
> **queries inside another query** — they allow you to use the result of one query as input to another.

```sql
SELECT column
FROM table
WHERE column IN (SELECT column FROM another_table WHERE condition);
```

### a. Single-row subquery

Returns **one value** (one row, one column).  
Used with operators like `=`, `<`, `>`, etc.

**Example:**

``` sql
-- Find employees who earn more than the average salary 
SELECT FirstName, Salary 
FROM Employees 
WHERE Salary > (SELECT AVG(Salary) FROM Employees);
```

### b. Multi-row subquery

Returns **multiple rows** (one column, many rows).  
Used with operators like `IN`, `ANY`, or `ALL`.

**Example:**

``` sql
-- Find employees who work in departments that have interns 
SELECT FirstName, DepartmentID 
FROM Employees 
WHERE DepartmentID IN ( SELECT DepartmentID FROM Interns );
```

### c. Multi-column subquery

Returns **multiple columns** — usually used with tuples `(col1, col2)`.

``` sql
-- Find employees with the same (DepartmentID, JobTitle) as employee 101 
SELECT * 
FROM Employees 
WHERE (DepartmentID, JobTitle) 
	IN ( SELECT DepartmentID, JobTitle     
			FROM Employees     
			WHERE EmployeeID = 101 
		);
```

### d. Correlated subquery

The **inner query depends on the outer query’s current row**.  
It runs **once for each row** of the outer query.

``` sql
-- Find employees who earn more than the average salary in their own department 
SELECT e.FirstName, e.DepartmentID, e.Salary 
FROM Employees e 
WHERE e.Salary > ( 
	SELECT AVG(Salary)     
	FROM Employees     
	WHERE DepartmentID = e.DepartmentID 
	);
```

> [!hint]
> Notice that the inner query refers to `e.DepartmentID` from the outer query.  
That’s what makes it _correlated_ — it can’t run on its own.

#### Where You Can Use Subqueries
You can use subqueries almost anywhere:
##### a. In `WHERE`:
- In the **WHERE** clause → to filter rows
``` sql
SELECT * FROM Employees 
WHERE DepartmentID IN (SELECT DepartmentID FROM Departments WHERE Location = 'Cairo');
```
##### b. In `FROM`:
- In the **FROM** clause → as a virtual table (called a _derived table_)
``` sql
SELECT d.DepartmentID, d.AvgSalary 
FROM ( 
	SELECT DepartmentID, AVG(Salary) AS AvgSalary
	FROM Employees 
	GROUP BY DepartmentID
	) d 
WHERE d.AvgSalary > 5000;
```

> [!todo] Exercise
> What is he trying to do here? is there other simpler ways to achieve this
###### c. In `SELECT`:
- In the **SELECT** list → to calculate a value for each row
``` sql
SELECT FirstName, 
	(SELECT DepartmentName 
	 FROM Departments d 
	 WHERE d.DepartmentID = e.DepartmentID
	 ) AS DeptName 
FROM Employees e;`
```


> [!todo] Exercise
> Is there other simpler ways to achieve this? which is more optimal?

> [!todo] Excercise
> ```embed
title: "SQL 50 - Study Plan - LeetCode"
image: "https://assets.leetcode.com/static_assets/others/Top_SQL_50_static_cover_picture.png"
description: "Crack SQL Interview in 50 Qs"
url: "https://leetcode.com/studyplan/top-sql-50/"
favicon: ""
aspectRatio: "100"
>```

#conceptual 