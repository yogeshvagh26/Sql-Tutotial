# Phase 5: **Joins**

* **INNER JOIN**
* **LEFT JOIN**
* **RIGHT JOIN**
* **FULL JOIN**
* **SELF JOIN**
* **CROSS JOIN**

### Overview

In real‑world databases, **data is spread across multiple tables to avoid redundancy (normalisation). Joins allow you to combine rows from two or more tables based on a related column.** This phase covers all essential join types:


* **INNER JOIN** – matching rows only

* **LEFT JOIN** – all rows from left table + matches from right

* **RIGHT JOIN** – all rows from right table + matches from left

* **FULL JOIN** – all rows from both tables

* **SELF JOIN** – a table joined with itself

* **CROSS JOIN** – every combination of rows (Cartesian product)

---

## 1. **INNER JOIN**


**INNER JOIN returns only the rows where the join condition is met in both tables.** If a row in the left table has no matching row in the right table, it is excluded (and vice versa). This is the most common join.

#### Real-World Example

You have a **Customers table** and an **Orders table**. To get a list of customers who have placed at least one order, you **INNER JOIN** on **CustomerID**. Customers without orders are omitted.

----

#### SQL Syntax

```sql

    SELECT columns
    FROM table1
    INNER JOIN table2 
        ON table1.key = table2.key;
   
```

> (You can also write just JOIN – it means INNER JOIN.)

#### Examples

```sql

    -- Get all orders with customer names
    SELECT 
        Orders.OrderID, 
        Customers.CustomerName
    FROM Orders
    INNER JOIN Customers 
        ON Orders.CustomerID = Customers.CustomerID;

    ------------------------------------------------------------

    -- Using table aliases
    SELECT 
        o.OrderID, 
        c.CustomerName, 
        o.OrderDate
    FROM Orders AS o
    JOIN Customers AS c 
        ON o.CustomerID = c.CustomerID;
    
    ------------------------------------------------------------

    -- Joining three tables
    SELECT 
        o.OrderID, 
        c.CustomerName, 
        p.ProductName
    FROM Orders o
    JOIN Customers c 
        ON o.CustomerID = c.CustomerID
    JOIN OrderItems oi 
        ON o.OrderID = oi.OrderID
    JOIN Products p 
        ON oi.ProductID = p.ProductID;

```
---

### Visual Table Illustration

#### Customers

| CustomerID | Name    |
|------------|---------|
| 1          | Alice   |
| 2          | Bob     |
| 3          | Charlie |

#### Orders
| OrderID | CustomerID | Amount |
|---------|------------|--------|
| 101     | 1          | 250    |
| 102     | 2          | 100    |
| 103     | 2          | 75     |
| 104     | 99         | 200    |

(99 does not exist in Customers)

> INNER JOIN on CustomerID:

| Name  | OrderID | Amount |
|-------|---------|--------|
| Alice | 101     | 250    |
| Bob   | 102     | 100    |
| Bob   | 103     | 75     |


> Charlie has no orders → excluded. OrderID 104 has invalid CustomerID → excluded.

---

#### Practice Questions

> Write a query to list all products and their category names using INNER JOIN (assume Products has CategoryID, Categories has CategoryID and CategoryName).

> Retrieve employee names with their department names.

> Show order details (OrderID, ProductName, Quantity) by joining OrderItems and Products.

---

#### Quiz

**1. INNER JOIN returns:**

    a) All rows from left table
    b) All rows from right table
    c) Only rows that match in both tables


---

### Summary for INNER JOIN

> Use INNER JOIN when you only need related data that exists on both sides.

---

## 2. **LEFT JOIN (LEFT OUTER JOIN)**

**LEFT JOIN returns all rows from the left table, and the matching rows from the right table.** If no match exists, the result columns from the right table contain NULL. It is an outer join.

#### Real-World Example
 
List all customers and their orders (including customers who never placed an order). **LEFT JOIN** puts **Customers** on the left, so every customer appears, with order columns **NULL** if no order exists.

---

#### SQL Syntax
 
```sql

    SELECT columns
    FROM table1
    LEFT JOIN table2 
        ON table1.key = table2.key;

```    

#### Examples

```sql

    -- All customers, with order IDs (NULL if no order)
    SELECT 
        c.CustomerName, 
        o.OrderID
    FROM Customers c
    LEFT JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

    ------------------------------------------------------------

    -- Find customers with no orders (using WHERE)
    SELECT c.CustomerName
    FROM Customers c
    LEFT JOIN Orders o 
        ON c.CustomerID = o.CustomerID
    WHERE o.OrderID IS NULL;

    ------------------------------------------------------------

    -- Left join with condition on the right table
    SELECT 
        c.CustomerName, 
        o.OrderID, 
        o.Amount
    FROM Customers c
    LEFT JOIN Orders o 
        ON c.CustomerID = o.CustomerID AND o.Amount > 100;

```

---
### Visual Table Illustration

#### Customers

| CustomerID | Name    |
|------------|---------|
| 1          | Alice   |
| 2          | Bob     |
| 3          | Charlie |

#### Orders

| OrderID | CustomerID | Amount |
|---------|------------|--------|
| 101     | 1          | 250    |
| 102     | 2          | 100    |
| 103     | 2          | 75     |
| 104     | 99         | 200    |

#### Applying LEFT JOIN (SQL Query)

```sql

    SELECT 
        c.CustomerName, 
        o.OrderID, 
        o.Amount
    FROM Customers c
    LEFT JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

```
#### Visual Result (LEFT JOIN)

| CustomerName | OrderID | Amount |
|--------------|---------|--------|
| Alice        | 101     | 250    |
| Bob          | 102     | 100    |
| Bob          | 103     | 75     |
| Charlie      | NULL    | NULL   |


> Charlie has no matching order → NULL for order columns.

> Order 104 has CustomerID 99, which doesn’t exist in Customers → excluded because it’s on the right side of the left join.

---

#### Practice Questions

> Show all products, including those never ordered (join with OrderItems). Use LEFT JOIN.

> List all employees and their managers (some employees may not have a manager – left join employee to manager).

> Count the number of orders per customer, including customers with zero orders.

---

#### Quiz

**2. Which join returns all rows from the left table regardless of matches?**

    a) INNER JOIN
    b) LEFT JOIN
    c) RIGHT JOIN


---

### Summary for LEFT JOIN

> LEFT JOIN keeps every row from the left table; unmatched right columns become NULL. Great for “all items from this table, plus related info if available”.

---

## 3. **RIGHT JOIN (RIGHT OUTER JOIN)**

**RIGHT JOIN is the mirror of LEFT JOIN:** **it returns all rows from the right table, and matching rows from the left table.** Rows in the right table with no match in the left table show **NULL** for left table columns.

**Note:** 

**RIGHT JOIN** is less common because you can usually swap tables and use **LEFT JOIN**. But it exists for completeness or when writing queries from a specific perspective.

#### Real-World Example

You want all orders (right table) and the customer names if they exist. Even if an order has an invalid **CustomerID**, you still see the order with **NULL** as customer name. (But typically you'd **LEFT JOIN** Orders to **Customers** instead.)

---

#### SQL Syntax

```sql

    SELECT columns
    FROM table1
    RIGHT JOIN table2 
        ON table1.key = table2.key;

```

#### Examples

```sql
    -- All orders, with customer names if available
    SELECT 
        c.CustomerName, 
        o.OrderID
    FROM Customers c
    RIGHT JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

    ------------------------------------------------------------        

    -- Equivalent to LEFT JOIN swapping tables:
    SELECT 
        c.CustomerName, 
        o.OrderID
    FROM Orders o
    LEFT JOIN Customers c 
        ON o.CustomerID = c.CustomerID;  -- same result

```
--- 

### Visual Table Illustration


#### Customers

| CustomerID | Name    |
|------------|---------|
| 1          | Alice   |
| 2          | Bob     |
| 3          | Charlie |

#### Orders

| OrderID | CustomerID | Amount |
|---------|------------|--------|
| 101     | 1          | 250    |
| 102     | 2          | 100    |
| 103     | 2          | 75     |
| 104     | 99         | 200    |

#### Applying RIGHT JOIN (SQL Query)

> Customers RIGHT JOIN Orders (Orders is right)

```sql

    SELECT 
        c.CustomerName, 
        o.OrderID, 
        o.Amount
    FROM Customers c
    RIGHT JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

```

| CustomerName | OrderID | Amount |
|--------------|---------|--------|
| Alice        | 101     | 250    |
| Bob          | 102     | 100    |
| Bob          | 103     | 75     |
| NULL         | 104     | 200    |

> All rows from Orders (right table) are returned.

> Order 104 has no matching customer → CustomerName is NULL.

---

#### Practice Questions

> Write a RIGHT JOIN to show all orders and their associated employee names (employees on left, orders on right). Assume some orders may have no employee assigned.

> Convert that query to a LEFT JOIN by swapping tables.

> When might RIGHT JOIN be more natural? (Think: you want to list all rows from a reference table that appears second in your FROM clause.)

#### Quiz

**3. RIGHT JOIN returns:**

    a) All rows from left table
    b) All rows from right table
    c) Only matching rows

---

### Summary for RIGHT JOIN

> RIGHT JOIN is a synonym for LEFT JOIN with table order reversed. Use it sparingly for readability.

---

## 4. **FULL JOIN (FULL OUTER JOIN)**

**FULL JOIN returns all rows from both tables.** When a row in one table has no match in the other, the missing side columns are **NULL**. **It combines the effect of LEFT JOIN and RIGHT JOIN.**

#### Real-World Example

You have a table of **Students** and a table of **Classes**. You want a complete list of all student‑class relationships, showing students without classes and classes without students. **FULL JOIN** does that.

---

#### SQL Syntax
 
```sql

    SELECT columns
    FROM table1
    FULL JOIN table2 
        ON table1.key = table2.key;

``` 

**Note**

>  MySQL does not directly support FULL JOIN; you can emulate it with LEFT JOIN UNION RIGHT JOIN. PostgreSQL, SQL Server, SQLite (as of recent) support it.

#### Examples

```sql

    -- All customers and all orders (including unmatched from either side)
    SELECT 
        c.CustomerName, 
        o.OrderID
    FROM Customers c
    FULL JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

    ------------------------------------------------------------

    -- Find customers without orders AND orders without customers
    SELECT 
        c.CustomerName, 
        o.OrderID
    FROM Customers c
    FULL JOIN Orders o 
        ON c.CustomerID = o.CustomerID
    WHERE c.CustomerID IS NULL OR 
        o.OrderID IS NULL;

```

---

### Visual Table Illustration

#### Customers FULL JOIN Orders


#### Customers

| CustomerID | Name    |
|------------|---------|
| 1          | Alice   |
| 2          | Bob     |
| 3          | Charlie |

#### Orders

| OrderID | CustomerID | Amount |
|---------|------------|--------|
| 101     | 1          | 250    |
| 102     | 2          | 100    |
| 103     | 2          | 75     |
| 104     | 99         | 200    |

#### Applying FULL JOIN (SQL Query)

```sql 

    SELECT 
        c.CustomerName, 
        o.OrderID, 
        o.Amount
    FROM Customers c
    FULL JOIN Orders o 
        ON c.CustomerID = o.CustomerID;

```

#### Visual Result

| CustomerName | OrderID | Amount |
|--------------|---------|--------|
| Alice        | 101     | 250    |
| Bob          | 102     | 100    |
| Bob          | 103     | 75     |
| Charlie      | NULL    | NULL   |
| NULL         | 104     | 200    |

> All rows from both tables are kept.

> Charlie (customer with no order) → order columns NULL.

> Order 104 (no matching customer) → customer columns NULL.

---

#### Practice Questions

> Write a FULL JOIN query to combine Employees (left) and Departments (right) to see all employees and all departments, even if an employee has no department or a department has no employees.

> How would you emulate FULL JOIN in MySQL? (Hint: LEFT JOIN UNION RIGHT JOIN)

---

#### Quiz 

**4. FULL JOIN is also known as:**

    a) FULL OUTER JOIN
    b) CROSS JOIN
    c) SELF JOIN


---

### Summary for FULL JOIN

> FULL JOIN shows all rows from both tables, filling NULL where matches are missing. Useful for comparing two sets.


---

## 5. **SELF JOIN**

**A SELF JOIN is a regular join but the table is joined with itself.** You use different aliases to treat the same table as two separate tables. It’s essential for hierarchical or comparative data within a single table.


#### Real-World Example

An **Employees** table has **EmployeeID** and **ManagerID** (which references another employee’s ID). To list each employee with their manager’s name, you **SELF JOIN** the **Employees** table twice: once as **emp**, once as **mgr**.

---

#### SQL Syntax

```sql

    SELECT 
        A.column, 
        B.column
    FROM table A
    JOIN table B 
        ON A.related_key = B.primary_key;

```    

#### Examples 

```sql

    -- Find employee-manager pairs
    SELECT 
        e.Name AS Employee, 
        m.Name AS Manager
    FROM Employees e
    LEFT JOIN Employees m 
        ON e.ManagerID = m.EmployeeID;

    ------------------------------------------------------------
    
    -- Find products with the same price (compare within same table)
    SELECT 
        p1.ProductName, 
        p2.ProductName, 
        p1.Price
    FROM Products p1
    JOIN Products p2 
        ON p1.Price = p2.Price AND p1.ProductID < p2.ProductID;

    ------------------------------------------------------------

    -- Find duplicate email addresses
    SELECT a.Email
    FROM Users a
    JOIN Users b 
        ON a.Email = b.Email AND 
            a.UserID <> b.UserID;

```
----

### Visual Table Illustration

Using an Employees table where **ManagerID** references **EmployeeID.**

#### Table: Employees

| EmpID | Name    | ManagerID |
|-------|---------|-----------|
| 1     | Alice   | NULL      |
| 2     | Bob     | 1         |
| 3     | Charlie | 1         |
| 4     | David   | 2         | 

> SELF JOIN with LEFT JOIN to show managers:

```sql

    SELECT 
        e.Name AS Employee, 
        m.Name AS Manager
    FROM Employees e
    LEFT JOIN Employees m 
        ON e.ManagerID = m.EmpID;

```

#### Visual Result

| Employee | Manager |
|----------|---------|
| Alice    | NULL    |
| Bob      | Alice   |
| Charlie  | Alice   |
| David    | Bob     |


#### Practice Questions

> Write a query to find pairs of customers who live in the same city (using a SELF JOIN).

> List each employee and the number of subordinates they have (group by manager after self join).

> Find all pairs of movies released in the same year (excluding same movie).

----

#### Quiz

**5. A SELF JOIN uses:**

    a) Two different tables
    b) The same table with two aliases
    c) Always INNER JOIN

---

### Summary for SELF JOIN
 
> SELF JOIN treats one table as two virtual tables. Use it for hierarchies, comparisons, or duplicates within a single table.

----

## 6. **CROSS JOIN**

**CROSS JOIN produces the Cartesian product of two tables:** every row from the first table is combined with every row from the second table. **If table A has 3 rows and table B has 4 rows, the result has 3×4 = 12 rows.** 

> No join condition is needed.

#### Real-World Example

You need to generate all possible combinations of colours and sizes for a product (e.g., t‑shirt). A **CROSS JOIN** between a **Colors** table and a **Sizes** table gives that.

---

#### SQL Syntax

```sql

    SELECT *
    FROM table1
    CROSS JOIN table2;

    ------------------------------------------------------------

    -- Or using comma (old style, not recommended):
    SELECT * 
    FROM table1, table2;

``` 

#### Examples

```sql

    -- All combinations of colors and sizes
    SELECT 
        c.ColorName, 
        s.SizeName
    FROM Colors c
    CROSS JOIN Sizes s;

    ------------------------------------------------------------

    -- Generate a date range by cross joining with a numbers table
    SELECT 
        DATE_ADD('2025-01-01', INTERVAL n.n DAY) AS calendar_date
    FROM Numbers n
    WHERE n.n < 365;

    ------------------------------------------------------------

    -- CROSS JOIN with WHERE (effectively becomes an inner join – not typical)
    SELECT * 
    FROM Products p
    CROSS JOIN Categories c
    WHERE p.CategoryID = c.CategoryID;  -- This is better as INNER JOIN

```
---

### Visual Table Illustration

#### Table : Colors
| Color |
|-------|
| Red   |
| Blue  |

#### Table : Sizes

| Size |
|------|
| S    |
| M    |
| L    |

```sql

    SELECT 
        c.Color, 
        s.Size
    FROM Colors c
    CROSS JOIN Sizes s;

``` 
#### CROSS JOIN result:

| Color | Size |
|-------|------|
| Red   | S    |
| Red   | M    |
| Red   | L    |
| Blue  | S    |
| Blue  | M    |
| Blue  | L    |


----
###  Practice Questions

> Write a CROSS JOIN between Shifts (Morning, Evening) and Days (Mon, Tue, Wed) to generate a schedule template.

> How many rows will a CROSS JOIN produce if table A has 10 rows and table B has 5 rows?

> When would you use CROSS JOIN in reporting?

---

#### Quiz

**6. CROSS JOIN requires:**

    a) An ON clause
    b) No join condition
    c) A WHERE clause

---

### Summary for CROSS JOIN

> CROSS JOIN creates all row combinations. Use it for generating test data, combinations, or when you explicitly need a Cartesian product.

---
---
---
---

## Practice Questions (Comprehensive)


#### Use these tables:

#### Students

| StudentID | Name  |
|-----------|-------|
| 1         | Alice |
| 2         | Bob   |
| 3         | Carol |

#### Enrollments

| EnrollmentID | StudentID | Course  |
|--------------|-----------|---------|
| 101          | 1         | Math    |
| 102          | 1         | Science |
| 103          | 2         | Math    |
| 104          | 3         | Art     |

#### Departments (for SELF JOIN example – optional)

| DeptID | Name    | ParentDeptID |
|--------|---------|--------------|
| 1      | Sales   | NULL         |
| 2      | IT      | 1            |
| 3      | Support | 1            |

#### Write queries:

1. **INNER JOIN** – List all enrollments with student names.

2. **LEFT JOIN** – List all students and their courses (include students with no enrollments).

3. **RIGHT JOIN** – List all enrollments and student names (should match INNER JOIN result here – why?)

4. **FULL JOIN** – Not supported in MySQL, but write the logic to show all students and all enrollments (use UNION of LEFT and RIGHT).

5. **SELF JOIN** – On Departments table, show each department and its parent department name.

6. **CROSS JOIN** – Generate all possible combinations of students and courses (ignore actual enrollments).

#### Answers:

```sql

    -- 1 INNER JOIN
    SELECT 
        s.Name, 
        e.Course
    FROM Students s
    INNER JOIN Enrollments e 
        ON s.StudentID = e.StudentID;

    ------------------------------------------------------------

    -- 2 LEFT JOIN
    SELECT 
        s.Name, 
        e.Course
    FROM Students s
    LEFT JOIN Enrollments e 
        ON s.StudentID = e.StudentID;

    ------------------------------------------------------------

    -- 3 RIGHT JOIN (same as INNER because right table's StudentID all exist in Students? Check: Enrollment 104 has StudentID 3 which exists. But if there were an orphan enrollment, RIGHT JOIN would show it. Here it matches INNER.)
    SELECT 
        s.Name, 
        e.Course
    FROM Students s
    RIGHT JOIN Enrollments e 
        ON s.StudentID = e.StudentID;

    ------------------------------------------------------------

    -- 4 FULL JOIN emulation (students without enrollments + enrollments without students)
    SELECT 
        s.Name, 
        e.Course
    FROM Students s
    LEFT JOIN Enrollments e 
        ON s.StudentID = e.StudentID
    UNION
    SELECT 
        s.Name, 
        e.Course
    FROM Students s
    RIGHT JOIN Enrollments e 
        ON s.StudentID = e.StudentID;

    ------------------------------------------------------------

    -- 5 SELF JOIN
    SELECT d.Name AS Department, p.Name AS ParentDepartment
    FROM Departments d
    LEFT JOIN Departments p ON d.ParentDeptID = p.DeptID;

    ------------------------------------------------------------

    -- 6 CROSS JOIN
    SELECT s.Name, e.Course
    FROM Students s
    CROSS JOIN Enrollments e;  -- careful: this gives many rows (3 students × 4 enrollments = 12)
    -- Better: use DISTINCT courses? But the question said "ignore actual enrollments" so any course from Enrollments table.

```
---

## Quiz (Answers)

1. c) Only rows that match in both tables

2. b) LEFT JOIN

3. b) All rows from right table

4. a) FULL OUTER JOIN

5. b) The same table with two aliases

6. b) No join condition

---

### Interview Questions

1. **Q: What is the difference between INNER JOIN and LEFT JOIN?**

    **Answer :**

    INNER JOIN returns only matching rows; LEFT JOIN returns all rows from the left table, with NULL for non‑matching right rows.

---

2. **Q: Can you use JOIN without ON?**

    **Answer :**

    Yes, that is a CROSS JOIN (Cartesian product). In some SQL dialects, JOIN without ON is a cross join; but it’s better to be explicit.

---

3. **Q: How do you join a table to itself? Give a use case.**

    **Answer :**
    
    Use a SELF JOIN with aliases. Use case: employee hierarchy (find manager names), or finding duplicate values.

---
4. **Q: What is the performance implication of CROSS JOIN on large tables?**

    **Answer :**

    It can produce huge results (millions of rows) and should be used only when necessary. Always consider if a more specific join is needed.
---
5. **Q: How does FULL JOIN differ from UNION of two LEFT JOINs?**

    **Answer :**

    FULL JOIN is exactly that: it keeps all rows from both tables. UNION of two LEFT JOIN queries (on opposite sides) emulates FULL JOIN. However, FULL JOIN is usually more efficient.
---
6. **Q: Write a query to find employees who have the same salary using a self‑join.**

    **Answer :**

    ```sql 

        SELECT 
            a.Name, 
            b.Name, 
            a.Salary 
        FROM Employees a JOIN Employees b 
            ON a.Salary = b.Salary AND 
            a.EmployeeID < b.EmployeeID;

    ```

---

## Assignment

### Scenario: 
> You are building a reporting system for a library. 

####  Tables:

* **Books** (BookID, Title, AuthorID, Genre)

* **Authors** (AuthorID, Name, Country)

* **Loans** (LoanID, BookID, MemberID, LoanDate, ReturnDate)

* **Members** (MemberID, FullName, JoinDate)

#### Tasks:

1. Create all four tables with appropriate data types and insert at least 6 books, 4 authors, 5 members, and 10 loans (some books never loaned, some members never borrowed).

    ---

2. Write SQL queries for each join type:

    > INNER JOIN: Show all loans with book titles and member names.

    > LEFT JOIN: List all books and the number of times they have been loaned (include books with zero loans). Use LEFT JOIN and GROUP BY.

    > RIGHT JOIN: List all members and the count of books they borrowed (including members with no loans). (Then rewrite using LEFT JOIN to see the difference.)

    > FULL JOIN: (If your DBMS supports it) Show all books and all members, even if no loans. If not supported, write the UNION emulation.

    > SELF JOIN: The Authors table does not have a self‑reference. Instead, create a simple table Categories with CategoryID, CategoryName, ParentCategoryID. Insert a few rows (e.g., Electronics → Computers → Laptops). Then write a SELF JOIN to show each category and its parent category name.

    > CROSS JOIN: Generate a report of all possible book–member combinations (for a hypothetical promotion). Limit the output to the first 10 rows if too large.
    ----

3. Bonus: Find books that have never been loaned using a LEFT JOIN with a WHERE condition on LoanID IS NULL.

    ---

4. Provide the SQL script with clear comments and the output for each query.

---

## **Summary of Phase 5**

| Join Type    | Returns                                                                 |
|--------------|-------------------------------------------------------------------------|
| **INNER JOIN**   | Rows that match in both tables                                          |
| **LEFT JOIN**    | All rows from left table, matched rows from right (NULL if none)        |
| **RIGHT JOIN**   | All rows from right table, matched rows from left (NULL if none)        |
| **FULL JOIN**    | All rows from both tables, NULL on missing side                         |
| **SELF JOIN**    | A table joined with itself (using aliases)                              |
| **CROSS JOIN**   | Cartesian product: every combination of rows from both tables           |

----

<br/><br/><br/>

<center><b>Happy Learning! 😊</b></center>