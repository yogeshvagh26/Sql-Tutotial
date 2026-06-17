# Phase 6: **Advanced Queries**

### Overview

After mastering joins and grouping, you're ready for advanced query techniques that allow you to write more powerful, readable, and reusable SQL code:

* **Subqueries** – queries inside other queries (also called nested queries or inner queries).

* **Correlated Subqueries** – subqueries that reference columns from the outer query.

* **Common Table Expressions (CTEs)** – named temporary result sets (modern, cleaner alternative to complex subqueries).

* **Views** – saved queries that act like virtual tables.

* **Temporary Tables** – physical (but session‑scoped) tables for intermediate storage.

> These tools are essential for complex reporting, data transformation, and application development.

---

## 1. **Subqueries**

A **subquery** is a SELECT statement nested inside another SQL statement (SELECT, INSERT, UPDATE, DELETE). Subqueries can be used:

* In the **WHERE** clause (to filter based on another query's result)

* In the **SELECT** clause (as a scalar value)

* In the **FROM** clause (as a derived table)

* With **IN**, **EXISTS**, **ANY**, **ALL** operators

> Subqueries can return:

* A single value (scalar subquery)

* A single row

* A single column (list of values)

* A full table

#### Real-World Example

Find employees who earn more than the company average salary. Instead of calculating average separately, you use a subquery inside **WHERE**.

----

#### SQL Syntax

```sql

    -- Subquery in WHERE (returns single value)
    SELECT columns 
    FROM table
    WHERE column = (SELECT AVG(column) 
                        FROM table);

    ----------------------------------------------------------

    -- Subquery with IN (returns a list)
    SELECT columns 
    FROM table
    WHERE column IN (SELECT column 
                        FROM another_table 
                        WHERE condition);

    ----------------------------------------------------------

    -- Subquery in SELECT (scalar)
    SELECT 
        column1, 
        (SELECT AVG(column2) 
            FROM table2) AS avg_value 
    FROM table1;

    ----------------------------------------------------------

    -- Subquery in FROM (derived table)
    SELECT * 
    FROM (SELECT columns 
            FROM table 
            WHERE condition) AS derived;

```
---

#### Examples

#### Example 1: Subquery in WHERE (scalar)

```sql

    -- Employees earning above average salary
    SELECT 
        name, 
        salary
    FROM Employees
    WHERE salary > (SELECT AVG(salary) 
                        FROM Employees);

```

#### Example 2: Subquery with IN

```sql

    -- Customers who have placed orders (list of customer IDs with orders)
    SELECT customer_name
    FROM Customers
    WHERE customer_id IN (SELECT DISTINCT customer_id 
                            FROM Orders);
    
```

#### Example 3: Subquery in SELECT (scalar)

```sql
    -- Show each product price and the average price of all products
    SELECT 
        product_name, 
        price,
        (SELECT AVG(price) 
            FROM Products) AS overall_avg_price
    FROM Products;
```

#### Example 4: Subquery in FROM (derived table)

```sql

    -- Get average order amount per customer, then filter customers above 500
    SELECT 
        customer_id, 
        avg_amount
    FROM (SELECT 
            customer_id, 
            AVG(amount) AS avg_amount
            FROM Orders
            GROUP BY customer_id) AS customer_avg
    WHERE avg_amount > 500;

```
----

### Visual Table Illustration

#### Employees

| EmpID | Name  | Salary |
|-------|-------|--------|
| 1     | Alice | 60000  |
| 2     | Bob   | 50000  |
| 3     | Carol | 70000  |

#### Subquery: 

```sql
    SELECT AVG(salary) FROM Employees 
    
    → 60000
```

#### Outer Query: 

```sql

    SELECT 
        name, 
        salary 
    FROM Employees 
    WHERE salary > (60000)

```

| Name  | Salary |
|-------|--------|
| Carol | 70000  |


---

#### Practice Questions

1. Find products whose price is greater than the average price of all products.

2. List employees who work in departments located in 'New York' (hint: Employees has dept_id, Departments has dept_id and location).

3. Show order IDs and amounts where the amount exceeds the average amount of orders placed by the same customer (this requires a correlated subquery – but start with simple).

---

#### Quiz

**1. A subquery that returns a single value is called:**

    a) Row subquery

    b) Scalar subquery

    c) Table subquery


---

### Summary for Subqueries

> Subqueries nest one query inside another. They are powerful but can become hard to read if nested deeply.

---


## 2. **Correlated Subqueries**

**A correlated subquery is a subquery that references columns from the outer query.** It is executed once for each row processed by the outer query. This makes it different from a non‑correlated subquery, which runs once and returns a static result.

Correlated subqueries are **often used with EXISTS, NOT EXISTS, or in SELECT** and **WHERE** clauses where the inner query needs values from the current row of the outer query.

#### Real-World Example

Find employees who earn more than the average salary of their own department (not the whole company). For each employee, the subquery computes the average salary of that employee's department.

---

#### SQL Syntax

```sql

    SELECT outer_alias.column
    FROM table AS outer_alias
    WHERE outer_alias.column operator (
        SELECT aggregate(inner_alias.column)
        FROM table2 AS inner_alias
        WHERE inner_alias.foreign_key = outer_alias.primary_key
    );

```

#### Examples

**Example 1: Employees earning above department average**

```sql

    SELECT 
        name, 
        salary, 
        department_id
    FROM Employees e1
    WHERE salary > (
        SELECT AVG(salary)
        FROM Employees e2
        WHERE e2.department_id = e1.department_id
    );

```


**Example 2: Using EXISTS (find customers who have placed at least one order)**

```sql

    SELECT customer_name
    FROM Customers c
    WHERE EXISTS (
        SELECT 1
        FROM Orders o
        WHERE o.customer_id = c.customer_id
    );

```


**Example 3: Using NOT EXISTS (customers with no orders)**

```sql

    SELECT customer_name
    FROM Customers c
    WHERE NOT EXISTS (
        SELECT 1
        FROM Orders o
        WHERE o.customer_id = c.customer_id
    );

```

**Example 4: Correlated subquery in SELECT**

```sql

    -- For each order, show the order amount and the maximum amount ever spent by that customer
    SELECT 
        order_id, 
        customer_id, 
        amount,
        (SELECT MAX(amount)
            FROM Orders o2
            WHERE o2.customer_id = o1.customer_id) AS customer_max
    FROM Orders o1;

```

---

### Visual Table Illustration

#### Employees

| EmpID | Name  | DeptID | Salary |
|-------|-------|--------|--------|
| 1     | Alice | 10     | 60000  |
| 2     | Bob   | 10     | 50000  |
| 3     | Carol | 20     | 70000  |
| 4     | Dave  | 20     | 65000  |

#### Correlated subquery for each employee:

* For Alice (DeptID 10): subquery computes AVG salary in Dept 10 = (60000+50000)/2 = 55000 → 60000 > 55000 → include.

* For Bob: 50000 > 55000? No → exclude.

* For Carol (DeptID 20): AVG = (70000+65000)/2 = 67500 → 70000 > 67500 → include.

* For Dave: 65000 > 67500? No → exclude.

```sql

    SELECT 
        e1.name, 
        e1.salary, 
        e1.dept_id
    FROM Employees e1
    WHERE e1.salary > (
        SELECT AVG(e2.salary)
        FROM Employees e2
        WHERE e2.dept_id = e1.dept_id
    );

```

#### Result:

| Name  | Salary | DeptID |
|-------|--------|--------|
| Alice | 60000  | 10     |
| Carol | 70000  | 20     |


---

#### Practice Questions

1. Find products whose price is higher than the average price of products in the same category.

2. List students who have scored above the average score of their class (assume Students table with student_id, class_id, score).

3. Write a query using NOT EXISTS to find customers who have never placed an order.

---
#### Quiz

**2. A correlated subquery is executed:**

    a) Once for the entire query
    b) Once for each row of the outer query
    c) Only if the outer query returns at least one row

---

### Summary for Correlated Subqueries

> Correlated subqueries reference outer values and are row‑by‑row evaluated. They are powerful but can be slow on large datasets. Use **EXISTS** / **NOT EXISTS** for existence checks.

---

## 3. **Common Table Expressions (CTE)**

**A Common Table Expression (CTE) is a temporary named result set** that you can reference within a **SELECT**, **INSERT**, **UPDATE**, or **DELETE** statement. 

CTEs are defined using the **WITH** clause and exist only for the duration of that single query. They improve readability and allow recursion (recursive CTEs).

> **Unlike subqueries**, CTEs can be referenced multiple times in the same query and can be chained.


#### Real-World Example

You need to calculate the average salary per department, then list employees earning above that department's average. Using a CTE makes the query clearer than a nested subquery.

---

#### SQL Syntax

```sql

    WITH cte_name AS (
        -- CTE query
        SELECT columns 
        FROM table 
        WHERE condition
    )
    -- Main query that uses the CTE
    SELECT * 
    FROM cte_name;

```
#### Multiple CTEs:

```sql
    WITH cte1 AS (...),
        cte2 AS (...)
    SELECT * FROM cte1 JOIN cte2 ON ...;
```
---
#### Examples

#### Example 1: Simple CTE


```sql

    WITH DeptAvg AS (
        SELECT 
            department_id, 
            AVG(salary) AS avg_salary
        FROM Employees
        GROUP BY department_id
    )
    SELECT 
        e.name, 
        e.salary, 
        d.avg_salary
    FROM Employees e
    JOIN DeptAvg d 
        ON e.department_id = d.department_id
    WHERE e.salary > d.avg_salary;

```

#### Example 2: CTE used twice

```sql

    WITH HighValueOrders AS (
        SELECT 
            customer_id, 
            SUM(amount) AS total_spent
        FROM Orders
        GROUP BY customer_id
        HAVING SUM(amount) > 1000
    )
    SELECT 
        COUNT(*) AS top_customers_count 
    FROM HighValueOrders;
    -- But you can also join with it elsewhere

```


#### Example 3: Recursive CTE (hierarchy)

```sql

    WITH RECURSIVE OrgHierarchy AS (
        -- Anchor member
        SELECT 
            emp_id, 
            name, 
            manager_id, 1 AS level
        FROM Employees
        WHERE manager_id IS NULL
        UNION ALL
        -- Recursive member
        SELECT 
            e.emp_id, 
            e.name, 
            e.manager_id, 
            oh.level + 1
        FROM Employees e
        JOIN OrgHierarchy oh 
            ON e.manager_id = oh.emp_id
    )
    SELECT * 
    FROM OrgHierarchy;

```

---

### Visual Table Illustration

#### Employees

| EmpID | Name  | DeptID | Salary |
|-------|-------|--------|--------|
| 1     | Alice | 10     | 60000  |
| 2     | Bob   | 10     | 50000  |
| 3     | Carol | 20     | 70000  |

#### CTE: DeptAvg

| DeptID | avg_salary |
|--------|------------|
| 10     | 55000      |
| 20     | 70000      |


**Main query** joins Employees with CTE and filters salary > avg_salary. Result: Alice (60000 > 55000), Carol (70000 > 70000? false because equal, not greater). So only Alice.



### Query (CTE)

```sql

    WITH DeptAvg AS (
        SELECT 
            dept_id, 
            AVG(salary) AS avg_salary
        FROM Employees
        GROUP BY dept_id
    )
    SELECT 
        e.name, 
        e.salary, 
        e.dept_id
    FROM Employees e
    JOIN DeptAvg d ON e.dept_id = d.dept_id
    WHERE e.salary > d.avg_salary;

```

#### Output : 

| Name  | Salary | DeptID |
|-------|--------|--------|
| Alice | 60000  | 10     |

> Carol is excluded because 70000 is not greater than 67500 (equal).

----

#### Practice Questions

1. Write a CTE that finds the total sales per product category, then use it to list categories with total sales > $10,000.

2. Use a CTE to calculate the average order value per customer, then find customers whose average order value is above the overall average.

3. (Recursive) Given a Employees table with emp_id, manager_id, write a CTE to list all subordinates of a specific manager (e.g., manager_id = 1).

---

#### Quiz

**3. A CTE is defined using:**

    a) CREATE TABLE
    b) WITH clause
    c) AS inside SELECT

---


### Summary for CTE

> CTEs make complex queries more readable and allow recursive queries. They are temporary and scoped to a single SQL statement.

---


## 4. **Views**

**A view is a virtual table based on the result set of a SELECT query.** 

> It does not store data physically (except for materialized views in some databases). 

> Views simplify complex queries, provide security by restricting access to specific columns/rows, and act as a layer of abstraction.

You can query a view like a regular table. Some views are updatable (if they meet certain conditions).


#### Real-World Example

A manager needs to see only employee names and departments, not salaries. Create a view **EmployeePublic** that excludes the salary column. Then grant access to the view instead of the base table.

---

#### SQL Syntax

```sql

    CREATE VIEW view_name AS
        SELECT columns
        FROM tables
        WHERE condition;

    -- Then use it
    SELECT * 
    FROM view_name;

    -- Drop a view
    DROP VIEW view_name;

```

#### Examples


#### Example 1: Simple view

```sql

    CREATE VIEW ActiveEmployees AS
        SELECT 
            emp_id, 
            name, 
            department_id
        FROM Employees
        WHERE status = 'Active';

```

#### Example 2: View with joins

```sql

    CREATE VIEW OrderDetailsView AS
        SELECT 
            o.order_id, 
            c.customer_name, 
            o.order_date, 
            o.amount
        FROM Orders o
        JOIN Customers c 
            ON o.customer_id = c.customer_id;

```

#### Example 3: View with aggregation (usually not updatable)

```sql

    CREATE VIEW DeptSummary AS
        SELECT 
            department_id, 
            COUNT(*) AS emp_count, 
            AVG(salary) AS avg_salary
        FROM Employees
        GROUP BY department_id;

```

#### Example 4: Updatable view (simple)

```sql

    CREATE VIEW EmployeesSimple AS
        SELECT 
            emp_id, 
            name, 
            salary 
        FROM Employees;

    -- You can update salary through this view (if it maps directly)
    UPDATE EmployeesSimple 
    SET salary = 65000 
    WHERE emp_id = 1;

```

---

### Visual Table Illustration

#### Base table: Employees (columns: emp_id, name, salary, ssn, dept_id)

#### View: EmployeePublic

```sql

    CREATE VIEW EmployeePublic AS
        SELECT 
            emp_id, 
            name, 
            dept_id 
            FROM Employees;

```
> When you **SELECT * FROM EmployeePublic;**, you see only emp_id, name, dept_id. The salary and SSN are hidden.

---

#### Practice Questions

1. Create a view named **HighValueOrders** that shows orders with amount > 500, including order_id, customer_id, and amount.

2. Query that view to count how many high‑value orders exist.


3. Create a view that joins Products and Categories to show product name and category name.


4. Drop a view.

---
#### Quiz

**4. A view is:**

    a) A physical copy of data
    b) A saved query that acts like a virtual table
    c) A temporary table that disappears after session


---

### Summary for Views

> Views encapsulate complex logic, enhance security, and simplify frequent queries. They don’t store data unless materialized.

---


## 5. **Temporary Tables**

**Temporary tables are physical tables that exist only for the duration of a session or transaction (depending on database).** 

> They are **useful for storing intermediate results**, especially when you need to **process data in steps**, or **when you want to index or manipulate a subset repeatedly**.

**Unlike CTEs (which exist only for one query),** temporary tables can be used across **multiple queries in the same session**. They are **automatically dropped when the session ends** (or when the transaction ends, depending on type).

#### Real-World Example

You are generating a monthly report that requires complex transformations. Instead of writing a giant single query, you store intermediate results in a temporary table, index it, then run multiple analytic queries against it.

---

#### SQL Syntax

#### MySQL / PostgreSQL / SQLite (global temporary)

```sql

    CREATE TEMPORARY 
        TABLE temp_table_name (
            column1 datatype,
            column2 datatype
        );

    -- Or create from a SELECT
    CREATE TEMPORARY 
        TABLE temp_table_name AS

    SELECT columns 
    FROM base_table 
    WHERE condition;

```

#### SQL Server

```sql
    -- Local temporary table (single #)
    CREATE TABLE #temp_table (column1 datatype);
    
    -- Global temporary table (double ##)
    CREATE TABLE ##temp_table (column1 datatype);
```

---


#### Examples

#### Example 1: Create empty temp table and insert


```sql

    CREATE TEMPORARY 
        TABLE TopCustomers (
            customer_id INT,
            total_spent DECIMAL(10,2)
        );

    INSERT INTO TopCustomers
        SELECT 
            customer_id, 
            SUM(amount)
        FROM Orders
        GROUP BY customer_id
        HAVING SUM(amount) > 1000;

```

#### Example 2: Create temp table from SELECT

```sql

    CREATE TEMPORARY 
        TABLE HighValueOrders AS

    SELECT * 
    FROM Orders 
    WHERE amount > 1000;
    
```

#### Example 3: Use temp table in multiple queries

```sql

    -- Create temp table of active employees
    CREATE TEMPORARY 
        TABLE ActiveEmps AS
        SELECT * 
        FROM Employees 
        WHERE status = 'Active';

    -- First analysis
    SELECT 
        department_id, 
        COUNT(*) 
    FROM ActiveEmps 
    GROUP BY department_id;

    -- Second analysis
    SELECT AVG(salary) 
    FROM ActiveEmps;

    -- Temp table is dropped automatically at end of session
    
```

#### Example 4: Temporary table with index (for performance)

```sql
    CREATE TEMPORARY 
        TABLE TempData AS
        SELECT 
            order_id, 
            customer_id, 
            amount 
        FROM Orders 
        WHERE order_date > '2025-01-01';

    CREATE INDEX idx_customer 
        ON TempData(customer_id);

    -- Now run heavy queries using TempData
    
```
---

### Visual Table Illustration

**Session A** creates a temporary table **TempOrders** with data from **Orders**.

**Session B** cannot see **TempOrders** (it's session‑private in most DBMS).

After session A ends, **TempOrders** is automatically dropped.

---

#### Practice Questions

1. Create a temporary table containing all products priced above $100.


2. Insert into that temp table a new product (just for testing).

3. Write a query that joins the temp table with Orders to see which expensive products have been ordered.


4. Explain the lifetime of a temporary table in your database system.

---

#### Quiz

**5. A temporary table:**

    a) Exists permanently until dropped
    b) Exists only for the current session (or transaction)
    c) Is shared across all users

---

### Summary for Temporary Tables

> Temporary tables store intermediate results and are automatically cleaned up.

> They are useful for multi‑step processing and performance optimisation.

---

## Practice Questions (Comprehensive)

#### Use these tables:

Table : **Orders**

| OrderID | CustomerID | Amount | OrderDate  |
|---------|------------|--------|------------|
| 1       | 101        | 250    | 2025-01-10 |
| 2       | 102        | 600    | 2025-01-15 |
| 3       | 101        | 150    | 2025-02-01 |
| 4       | 103        | 800    | 2025-02-10 |

Table : **Customers**

| CustomerID | Name    |
|------------|---------|
| 101        | Alice   |
| 102        | Bob     |
| 103        | Charlie |
| 104        | David   |


####  Write queries using the advanced techniques:

1. Subquery: Find customers who have placed an order with amount > 500.

2. Correlated Subquery: For each customer, show their name and the amount of their highest order.

3. CTE: Write a CTE to compute total spent per customer, then show customers who spent more than $500.

4. View: Create a view CustomerOrders that shows customer name, order ID, and amount. Then query it to list all orders for 'Alice'.


5. Temporary Table: Create a temporary table MarchOrders containing orders from March 2025 (assume some data), then use it to count how many orders.

#### Answers (self‑check):

```sql

    -- 1
    SELECT name 
    FROM Customers
    WHERE CustomerID IN (SELECT CustomerID 
                            FROM Orders 
                            WHERE Amount > 500
                        );

    ----------------------------------------------------------

    -- 2 (correlated)
    SELECT 
        name, (SELECT MAX(Amount) 
                FROM Orders o 
                WHERE o.CustomerID = c.CustomerID
                ) AS max_order
    FROM Customers c;

    ----------------------------------------------------------

    -- 3
    WITH CustomerTotal AS (
        SELECT 
            CustomerID, 
            SUM(Amount) AS total_spent
        FROM Orders
        GROUP BY CustomerID
    )
    SELECT 
        c.Name, 
        ct.total_spent
    FROM Customers c
    JOIN CustomerTotal ct 
        ON c.CustomerID = ct.CustomerID
    WHERE ct.total_spent > 500;

    ----------------------------------------------------------
    
    -- 4
    CREATE VIEW CustomerOrders AS
        SELECT 
            c.Name, 
            o.OrderID, 
            o.Amount
        FROM Customers c
        JOIN Orders o 
            ON c.CustomerID = o.CustomerID;

    SELECT * 
    FROM CustomerOrders 
    WHERE Name = 'Alice';

    ----------------------------------------------------------

    -- 5
    CREATE TEMPORARY 
        TABLE MarchOrders AS
        SELECT * 
        FROM Orders 
        WHERE MONTH(OrderDate) = 3;

    SELECT COUNT(*) FROM MarchOrders;

```

---

### Quiz (Answers)

1. b) Scalar subquery

2. b) Once for each row of the outer query

3. b) WITH clause

4. b) A saved query that acts like a virtual table

5. b) Exists only for the current session (or transaction)

----

### Interview Questions


1. **Q: What is the difference between a subquery and a CTE?**

    **Answer :**

    Both are temporary result sets. CTEs are defined at the top using **WITH**, can be referenced multiple times, and support recursion. Subqueries are nested inline. CTEs generally improve readability.

    ---

2. **Q: When would you use a temporary table instead of a CTE?**

    **Answer :**

    When you need to reuse the intermediate result across multiple queries, index it for performance, or when the intermediate result is very large and you want to avoid re‑computation.

    ---

3. **Q: Can you update a view?**

    **Answer :**

    **Yes, if the view is updatable** – meaning it maps directly to a single base table without aggregates, DISTINCT, GROUP BY, or complex joins. Some databases allow updates on simple views. Use **WITH CHECK OPTION** to enforce conditions.

    ---

4. **Q: What is a recursive CTE? Give an example.**

    **Answer :**

    A CTE that references itself. Used for hierarchical data like org charts, bill of materials, or graph traversal. Example: listing all subordinates under a manager.

    ---

5. **Q: Difference between WHERE and HAVING – and can you use a subquery in both?**

    **Answer :**

    **WHERE** filters rows before grouping; **HAVING** filters groups after aggregation. Subqueries can be used in both clauses.

---

## Assignment

### Scenario: 

> You are building a sales analytics dashboard. Use the following schema:


```sql

    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        ProductName VARCHAR(100),
        CategoryID INT,
        Price DECIMAL(10,2)
    );
    
    ----------------------------------------------------------

    CREATE TABLE Categories (
        CategoryID INT PRIMARY KEY,
        CategoryName VARCHAR(50)
    );

    ----------------------------------------------------------

    CREATE TABLE Sales (
        SaleID INT PRIMARY KEY,
        ProductID INT,
        SaleDate DATE,
        Quantity INT,
        Discount DECIMAL(5,2)
    );

```

> Insert sufficient sample data (at least 5 categories, 10 products, 20 sales records with various dates and quantities).

### Tasks:

1. Subquery – Write a query to find products whose price is above the average price of products in their own category (use a correlated subquery).

2. CTE – Write a CTE to calculate total revenue (Price * Quantity * (1 - Discount/100)) per product. Then use the CTE to find the top 3 products by revenue.


3. View – Create a view MonthlySales that shows year, month, category name, and total revenue (aggregated). Query the view to show the best month for each category.


4. Temporary Table – Create a temporary table HighValueSales containing sales where revenue (calculated) is greater than $1000. Then write a query using this temporary table to count how many such sales occurred per category.


5. Combine techniques – Using a CTE or subquery, identify categories where the maximum product price is more than twice the minimum product price within that category.


6. Bonus (Recursive CTE) – If your DBMS supports it, create a simple hierarchy table (e.g., Employees with emp_id, manager_id) and write a recursive CTE to output the full chain of command for each employee.

----

## Summary of Phase 6

| Concept              | Purpose                                                                                   |
|----------------------|-------------------------------------------------------------------------------------------|
| **Subquery**             | Nest a query inside another; can be scalar, row, column, or table.                        |
| **Correlated Subquery**  | Subquery that references outer query; executed per row of outer query.                    |
| **CTE (WITH)**           | Named temporary result set for a single query; improves readability and allows recursion. |
| **View**                 | Virtual table based on a saved SELECT; adds abstraction and security.                     |
| **Temporary Table**      | Physical table that exists for session/transaction; good for multi-step processing.       |


----


<br/><br/><br/>
<center> <b>Happy Learning! 😊</b> </center>












    



    




    





















