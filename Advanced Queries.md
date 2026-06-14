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

