
> [!tip]
> - This is just a revision not an explanation, a refresher of important concepts.
>- We will only be discussing RDBMS (relational databases)
>- In the index section a list of topics you as a developer will need to know.

# ACID properties

#### A – Atomicity (All or Nothing)
A transaction is treated as a single unit: **either everything succeeds, or nothing is applied**.
``` Example
Transferring $100 from Alice to Bob:
    - Deduct $100 from Alice 
    - Add $100 to Bob 

//If the second step fails, the first step is **rolled back**. No partial updates.
```
#### C – Consistency
A transaction must **take the database from one valid state to another** while respecting all rules (constraints, triggers, relationships). 
``` Example
You can’t have a student with a negative age. 
Transactions must preserve these rules.
```

#### I – Isolation
Concurrent transactions **don’t interfere with each other**.
``` Example
Two users updating the same bank account at the same time won’t cause inconsistent balance. 

Databases manage this with 
 - locks 
 - or MVCC (multi-version concurrency control).
```
#### D – Durability
Once a transaction is committed, it **persists permanently**, even if the system crashes.
``` Example
After transferring money and committing the transaction, the new balances remain safe even if the database server restarts immediately afterward.
```


**In short:** ACID ensures that databases behave reliably, even with multiple users and unexpected failures.

# ERD concepts
- **Entities**: Objects or concepts with independent existence in the system (e.g., `Student`, `Order`). Represented as rectangles.
- **Attributes**: Properties of entities (e.g., `Student` has `Name`, `StudentID`). Represented as ovals.
    
    - **Primary Key (PK)**: Unique identifier for an entity (e.g., `StudentID`).
    - **Foreign Key (FK)**: Attribute that links to a primary key of another entity. *Not directly represented in ERD but only as a relation*
    - **Composite Attributes**: Attributes composed of multiple sub-attributes (e.g., `FullName` = `FirstName` + `LastName`).
    - **Derived Attributes**: Attributes calculated from other attributes (e.g., `Age` from `DateOfBirth`).
- **Weak Entities:** Cannot exist without a strong entity.
	-  Usually have a **partial key**.
	- Represented with double rectangles and relationships with double diamonds.

- **Relationship Types**: Define how entities are related.
    - **One-to-One (1:1)**: Each entity in A relates to one entity in B.
    - **One-to-Many (1:N)**: One entity in A relates to many in B.
    - **Many-to-Many (M:N)**: Many entities in A relate to many in B.
    - Recursive Relationships
> [!todo] Research
> What are aggregation relations?
> what are ternary relations?

- **Relationship Attributes**: Attributes that describe the relationship itself (e.g., `EnrollmentDate` in `Student-Course` relationship).

- **Cardinality**: Specifies how many instances of one entity relate to instances of another.
    - Example: `1..1`, `0..1`, `1..*`, `0..*`
- **Participation**: Specifies whether all or some instances participate in the relationship.
    - **Total participation**: Every instance must be involved (mandatory).
    - **Partial participation**: Some instances may not be involved (optional).
- **Derived Attribute**: An attribute whose value **can be calculated or derived from other attributes**. Usually represented with a **dashed oval** in Chen notation.

# DDL
**Data Definition Language** is about **defining or changing the structure of the database**
#### CREATE
``` sql
-- Create a new table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Salary DECIMAL(10,2)
);

-- Create a view
CREATE VIEW HighSalary AS
SELECT FirstName, LastName, Salary
FROM Employees
WHERE Salary > 5000;

-- Create an index
CREATE INDEX idx_lastname ON Employees(LastName);

-- Create a schema
CREATE SCHEMA HR;

```
#### ALTER
change an existing object (add/remove columns, modify constraints).
``` sql
-- Add a new column
ALTER TABLE Employees
ADD DateOfJoining DATE;

-- Modify column data type
ALTER TABLE Employees
MODIFY Salary DECIMAL(12,2);  -- Oracle syntax
-- PostgreSQL uses: ALTER TABLE Employees ALTER COLUMN Salary TYPE DECIMAL(12,2);

-- Drop a column
ALTER TABLE Employees
DROP COLUMN DateOfJoining;

-- Add a constraint
ALTER TABLE Employees
ADD CONSTRAINT chk_salary CHECK (Salary > 0);

--------------------Renaming
-- Rename a table (Oracle)
RENAME Employees TO Staff;

-- Rename a column (PostgreSQL)
ALTER TABLE Employees RENAME COLUMN FirstName TO FName;

-- Rename a table (PostgreSQL)
ALTER TABLE Employees RENAME TO Staff;

```
#### DROP
``` sql
-- Drop a table
DROP TABLE Employees;

-- Drop a view
DROP VIEW HighSalary;

-- Drop an index
DROP INDEX idx_lastname;  -- Syntax may vary: PostgreSQL: DROP INDEX idx_lastname;

-- Drop a schema
DROP SCHEMA HR;

```
####  TRUNCATE
``` sql
-- **Purpose:** Remove all rows from a table quickly (structure stays).  
TRUNCATE TABLE Employees;

-- All rows are gone, but the table and columns still exist. 
-- Auto-increment IDs are reset in many DBs.
```

| Feature                     | `DELETE FROM table_name` (no WHERE)                | `TRUNCATE TABLE table_name`                           |
| --------------------------- | -------------------------------------------------- | ----------------------------------------------------- |
| **Operation type**          | DML (Data Manipulation Language)                   | DDL (Data Definition Language)                        |
| **Deletes**                 | Row by row, logs each deletion                     | Deallocates entire data pages, faster                 |
| **Transaction log**         | Every row deletion is logged (can be rolled back)  | Minimal logging; usually cannot roll back in some DBs |
| **Triggers**                | Activates `DELETE` triggers                        | Does **not** activate `DELETE` triggers               |
| **Identity/Sequence reset** | Does **not** reset identity/auto-increment columns | Resets identity/auto-increment counters               |
| **Speed**                   | Slower for large tables                            | Much faster for large tables                          |
| **Rollback**                | Can be rolled back if in a transaction             | Depends on DB; often cannot rollback in some RDBMS    |
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
ELECT * FROM Employees;  
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

## Joins & aggregation
## Sub-Queries

---

# References
> [!cite]
>```embed
title: "Fetching"
image: "data:image/svg+xml;base64,PHN2ZyBjbGFzcz0ibGRzLW1pY3Jvc29mdCIgd2lkdGg9IjgwcHgiICBoZWlnaHQ9IjgwcHgiICB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWlkWU1pZCI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoMCkiPjxjaXJjbGUgY3g9IjgxLjczNDEzMzYxMTY0OTQxIiBjeT0iNzQuMzUwNDU3MTYwMzQ4ODIiIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0MC4wMDEgNDkuOTk5OSA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49IjBzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9Ijc0LjM1MDQ1NzE2MDM0ODgyIiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0OC4zNTIgNTAuMDAwMSA1MC4wMDAxKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMDYyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iNjUuMzA3MzM3Mjk0NjAzNiIgY3k9Ijg2Ljk1NTE4MTMwMDQ1MTQ3IiBmaWxsPSIjZjhiMjZhIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTQuMjM2IDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMTI1cyI+PC9hbmltYXRlVHJhbnNmb3JtPgo8L2NpcmNsZT48Y2lyY2xlIGN4PSI1NS4yMjEwNDc2ODg4MDIwNyIgY3k9Ijg5LjY1Nzc5NDQ1NDk1MjQxIiBmaWxsPSIjYWJiZDgxIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTcuOTU4IDUwLjAwMDIgNTAuMDAwMikiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjE4NzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjQ0Ljc3ODk1MjMxMTE5NzkzIiBjeT0iODkuNjU3Nzk0NDU0OTUyNDEiIGZpbGw9IiM4NDliODciIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM1OS43NiA1MC4wMDY0IDUwLjAwNjQpIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4yNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMzQuNjkyNjYyNzA1Mzk2NDE1IiBjeT0iODYuOTU1MTgxMzAwNDUxNDciIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDAuMTgzNTUyIDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMzEyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMjUuNjQ5NTQyODM5NjUxMTc2IiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDEuODY0NTcgNTAgNTApIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4zNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjE4LjI2NTg2NjM4ODM1MDYiIGN5PSI3NC4zNTA0NTcxNjAzNDg4NCIgZmlsbD0iI2Y4YjI2YSIgcj0iNSIgdHJhbnNmb3JtPSJyb3RhdGUoNS40NTEyNiA1MCA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjQzNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT48L2c+PC9zdmc+"
description: "Fetching https://www.geeksforgeeks.org/dbms/what-is-database/"
url: "https://www.geeksforgeeks.org/dbms/what-is-database/"
favicon: ""
>```

> [!cite] 
>```embed
title: "Fetching"
image: "data:image/svg+xml;base64,PHN2ZyBjbGFzcz0ibGRzLW1pY3Jvc29mdCIgd2lkdGg9IjgwcHgiICBoZWlnaHQ9IjgwcHgiICB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAxMDAgMTAwIiBwcmVzZXJ2ZUFzcGVjdFJhdGlvPSJ4TWlkWU1pZCI+PGcgdHJhbnNmb3JtPSJyb3RhdGUoMCkiPjxjaXJjbGUgY3g9IjgxLjczNDEzMzYxMTY0OTQxIiBjeT0iNzQuMzUwNDU3MTYwMzQ4ODIiIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0MC4wMDEgNDkuOTk5OSA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49IjBzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9Ijc0LjM1MDQ1NzE2MDM0ODgyIiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM0OC4zNTIgNTAuMDAwMSA1MC4wMDAxKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMDYyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iNjUuMzA3MzM3Mjk0NjAzNiIgY3k9Ijg2Ljk1NTE4MTMwMDQ1MTQ3IiBmaWxsPSIjZjhiMjZhIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTQuMjM2IDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMTI1cyI+PC9hbmltYXRlVHJhbnNmb3JtPgo8L2NpcmNsZT48Y2lyY2xlIGN4PSI1NS4yMjEwNDc2ODg4MDIwNyIgY3k9Ijg5LjY1Nzc5NDQ1NDk1MjQxIiBmaWxsPSIjYWJiZDgxIiByPSI1IiB0cmFuc2Zvcm09InJvdGF0ZSgzNTcuOTU4IDUwLjAwMDIgNTAuMDAwMikiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjE4NzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjQ0Ljc3ODk1MjMxMTE5NzkzIiBjeT0iODkuNjU3Nzk0NDU0OTUyNDEiIGZpbGw9IiM4NDliODciIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDM1OS43NiA1MC4wMDY0IDUwLjAwNjQpIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4yNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMzQuNjkyNjYyNzA1Mzk2NDE1IiBjeT0iODYuOTU1MTgxMzAwNDUxNDciIGZpbGw9IiNlMTViNjQiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDAuMTgzNTUyIDUwIDUwKSI+CiAgPGFuaW1hdGVUcmFuc2Zvcm0gYXR0cmlidXRlTmFtZT0idHJhbnNmb3JtIiB0eXBlPSJyb3RhdGUiIGNhbGNNb2RlPSJzcGxpbmUiIHZhbHVlcz0iMCA1MCA1MDszNjAgNTAgNTAiIHRpbWVzPSIwOzEiIGtleVNwbGluZXM9IjAuNSAwIDAuNSAxIiByZXBlYXRDb3VudD0iaW5kZWZpbml0ZSIgZHVyPSIxLjVzIiBiZWdpbj0iLTAuMzEyNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT4KPC9jaXJjbGU+PGNpcmNsZSBjeD0iMjUuNjQ5NTQyODM5NjUxMTc2IiBjeT0iODEuNzM0MTMzNjExNjQ5NDEiIGZpbGw9IiNmNDdlNjAiIHI9IjUiIHRyYW5zZm9ybT0icm90YXRlKDEuODY0NTcgNTAgNTApIj4KICA8YW5pbWF0ZVRyYW5zZm9ybSBhdHRyaWJ1dGVOYW1lPSJ0cmFuc2Zvcm0iIHR5cGU9InJvdGF0ZSIgY2FsY01vZGU9InNwbGluZSIgdmFsdWVzPSIwIDUwIDUwOzM2MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiIGJlZ2luPSItMC4zNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxjaXJjbGUgY3g9IjE4LjI2NTg2NjM4ODM1MDYiIGN5PSI3NC4zNTA0NTcxNjAzNDg4NCIgZmlsbD0iI2Y4YjI2YSIgcj0iNSIgdHJhbnNmb3JtPSJyb3RhdGUoNS40NTEyNiA1MCA1MCkiPgogIDxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MzYwIDUwIDUwIiB0aW1lcz0iMDsxIiBrZXlTcGxpbmVzPSIwLjUgMCAwLjUgMSIgcmVwZWF0Q291bnQ9ImluZGVmaW5pdGUiIGR1cj0iMS41cyIgYmVnaW49Ii0wLjQzNzVzIj48L2FuaW1hdGVUcmFuc2Zvcm0+CjwvY2lyY2xlPjxhbmltYXRlVHJhbnNmb3JtIGF0dHJpYnV0ZU5hbWU9InRyYW5zZm9ybSIgdHlwZT0icm90YXRlIiBjYWxjTW9kZT0ic3BsaW5lIiB2YWx1ZXM9IjAgNTAgNTA7MCA1MCA1MCIgdGltZXM9IjA7MSIga2V5U3BsaW5lcz0iMC41IDAgMC41IDEiIHJlcGVhdENvdW50PSJpbmRlZmluaXRlIiBkdXI9IjEuNXMiPjwvYW5pbWF0ZVRyYW5zZm9ybT48L2c+PC9zdmc+"
description: "Fetching https://www.w3schools.com/sql/"
url: "https://www.w3schools.com/sql/"
favicon: ""
>```

# Index
red for not covered and not needed parts
yellow for needed but not covered
## Covered
**1. Fundamentals**
<mark style="background: #FFF3A3A6;">- What a database is</mark>
<mark style="background: #FF5582A6;">- OLTP vs OLAP </mark>
<mark style="background: #FFF3A3A6;">- DBMS vs RDBMS </mark>
- ACID properties
<mark style="background: #FFF3A3A6;">- Data models (relational, document, key-value, graph) </mark>
**2. SQL Essentials**
- DDL: CREATE, ALTER, DROP
- DML: INSERT, UPDATE, DELETE
- DQL: SELECT, WHERE, ORDER BY, DISTINCT
- Joins: INNER, LEFT, RIGHT, FULL
- Aggregation: GROUP BY, HAVING
- Subqueries (scalar, correlated, IN/EXISTS)
## Not covered
If it is green then it is important to know.

**3. Table Design**
<mark style="background: #BBFABBA6;">- Primary keys, foreign keys</mark>
<mark style="background: #BBFABBA6;">- Unique, check, not null constraints</mark>
<mark style="background: #BBFABBA6;">- Data types and when to use them</mark>
<mark style="background: #BBFABBA6;">- Auto-increment / identity columns</mark>
<mark style="background: #BBFABBA6;">- Default values</mark>

**4. Normalization**
- 1NF, 2NF, 3NF
- Why denormalization is sometimes used

**5. Indexing**
- B-tree vs hash indexes
<mark style="background: #BBFABBA6;">- When to index and when not to</mark>
- Composite indexes and index order
- Index impact on writes

**6. Transactions & Concurrency**
<mark style="background: #BBFABBA6;">- Transaction lifecycle</mark>
<mark style="background: #BBFABBA6;">- Isolation levels</mark>
<mark style="background: #BBFABBA6;">- Locks, deadlocks, optimistic vs pessimistic locking</mark>

**7. Query Optimization**
<mark style="background: #BBFABBA6;">- EXPLAIN / EXPLAIN ANALYZE</mark>
<mark style="background: #BBFABBA6;">- How indexes affect query plans</mark>
<mark style="background: #BBFABBA6;">- Common performance issues (full table scans, functions on indexed columns)</mark>

**8. Stored Code**
<mark style="background: #BBFABBA6;">- Views</mark>
<mark style="background: #BBFABBA6;">- Stored procedures</mark>
<mark style="background: #BBFABBA6;">- Triggers</mark>
<mark style="background: #BBFABBA6;">- Functions</mark>

**9. Security**
- User roles and privileges
- Row-level security
<mark style="background: #BBFABBA6;">- SQL injection prevention</mark>

**10. Backup & Recovery**
- Logical vs physical backup
- Point-in-time recovery
- WAL (Postgres) / redo logs (Oracle)

**11. Database Architecture**
<mark style="background: #BBFABBA6;">- Replication (master–replica)</mark>
<mark style="background: #BBFABBA6;">- Sharding and partitioning</mark>
- High availability basics

**12. NoSQL Overview (if needed)**
- Document databases (MongoDB)
<mark style="background: #BBFABBA6;">- Key-value stores (Redis)</mark>
<mark style="background: #BBFABBA6;">- When NoSQL is preferred over RDBMS</mark>


#conceptual 