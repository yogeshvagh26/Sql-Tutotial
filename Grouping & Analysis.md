# Phase 4: **Grouping & Analysis**

* **GROUP BY**
* **HAVING**
* **Aliases**
* **CASE Statements**
## Overview
In this phase, we move from simple aggregations to grouped analysis. You will learn how to:

* Split data into groups (GROUP BY)

* Filter those groups (HAVING)

* Rename columns or tables temporarily (Aliases)

* Perform conditional logic inside SQL (CASE)

These tools turn **raw data into meaningful summaries** like “total sales per region”, “average salary by department”, or “categorise customers based on spending”.


---

## 1. **GROUP BY**

**GROUP BY groups rows that have the same values in specified columns**, allowing aggregate functions (SUM, COUNT, AVG, etc.) to be calculated per group instead of across the whole table.

#### Real-World Example


A sales manager wants total revenue per product category. Without GROUP BY, SUM(Revenue) gives one total. With GROUP BY Category, you get one total per category.

---

#### SQL Syntax

```sql
    SELECT 
        column1, 
        aggregate_function(column2)
    FROM table
    WHERE condition
    GROUP BY column1;
```

#### Examples

```sql

    -- Count employees in each department
    SELECT 
        department, 
        COUNT(*) AS emp_count
    FROM Employees
    GROUP BY department;

    ----------------------------------------------------

    -- Total sales per region
    SELECT 
        region, 
        SUM(sales_amount) AS total_sales
    FROM Sales
    GROUP BY region;

    ----------------------------------------------------

    -- Average price per product category
    SELECT 
        category, 
        AVG(price) AS avg_price
    FROM Products
    GROUP BY category;

    ----------------------------------------------------

    -- Group by multiple columns (year + month)
    SELECT 
        YEAR(order_date) AS order_year, 
        MONTH(order_date) AS order_month, 
        SUM(amount)
    FROM Orders
    GROUP BY YEAR(order_date), MONTH(order_date);

```
---

### Visual Table Illustration

#### Table: Sales

| SaleID | Product  | Amount | Region |
|--------|----------|--------|--------|
| 1      | Laptop   | 1000   | North  |
| 2      | Mouse    | 20     | South  |
| 3      | Keyboard | 50     | North  |
| 4      | Laptop   | 1000   | South  |

#### Query: 

```sql
    SELECT 
        Region, 
        SUM(Amount) 
    FROM Sales 
    GROUP BY Region;  
```    

#### Result:

| Region | SUM(Amount) |
|--------|-------------|
| North  | 1050        |
| South  | 1020        |

---

#### Practice Questions

> Count how many products exist in each category.

> Calculate the total salary paid per department.

> Show the number of orders placed by each customer.

---

#### Quiz

**1. GROUP BY is used with:**

    a) WHERE only
    b) Aggregate functions
    c) ORDER BY only


---

### Summary for GROUP BY

> GROUP BY creates subgroups; then aggregate functions calculate per group.

----

## 2. **HAVING**

**HAVING filters groups after aggregation,** whereas **WHERE filters rows before aggregation.** Use HAVING to apply conditions on aggregated values (e.g., SUM(amount) > 1000).

#### Real-World Example

Find departments whose average salary exceeds 60,000. **You cannot use WHERE AVG(salary) > 60000 because AVG is calculated after grouping – that’s exactly when HAVING works.**

---

#### SQL Syntax

```sql
    SELECT 
        column1, 
        aggregate_function(column2)
    FROM table
    GROUP BY column1
    HAVING condition_on_aggregate;
```
#### Examples

```sql
    -- Departments with more than 5 employees
    SELECT 
        department, 
        COUNT(*) AS emp_count
    FROM Employees
    GROUP BY department
    HAVING COUNT(*) > 5;

    ----------------------------------------------------

    -- Regions where total sales > 10,000
    SELECT 
        region, 
        SUM(amount) AS total
    FROM Sales
    GROUP BY region
    HAVING SUM(amount) > 10000;

    ----------------------------------------------------

    -- Products with average rating >= 4.5
    SELECT 
        product_id, 
        AVG(rating) AS avg_rating
    FROM Reviews
    GROUP BY product_id
    HAVING AVG(rating) >= 4.5;

    ----------------------------------------------------

    -- Combined with WHERE (filter rows first, then groups)
    SELECT 
        category, 
        AVG(price)
    FROM Products
    WHERE stock > 0
    GROUP BY category
    HAVING AVG(price) BETWEEN 50 AND 200;

```

---

### Visual Table Illustration

#### Table: Orders

| Customer | Amount |
|----------|--------|
| Alice    | 100    |
| Bob      | 200    |
| Alice    | 50     |
| Bob      | 50     |
| Charlie  | 1000   |

#### Query: 

```sql
    SELECT 
        Customer, 
        SUM(Amount) 
    FROM Orders 
    GROUP BY Customer 
    HAVING SUM(Amount) > 150;
```
**Step 1** – Group by Customer:

Alice (150), Bob (250), Charlie (1000)

**Step 2** – Apply HAVING (>150): Bob and Charlie.

| Customer | SUM(Amount) |
|----------|-------------|
| Bob      | 250         |
| Charlie  | 1000        |

--- 

#### Practice Questions

> Find product categories where the average price is above $500.

> Show customers who have placed more than 3 orders.

> List months (e.g., January, February) where total sales were less than $5,000.
---

#### Quiz

**2. Which clause filters groups after aggregation?**

    a) WHERE
    b) GROUP BY
    c) HAVING

---

### Summary for HAVING

> HAVING is to GROUP BY what WHERE is to rows – it filters groups based on aggregate results.

---

## 3. **Aliases**

**Aliases give temporary names to columns (AS) or tables (AS) to make queries easier to read or to label output.** They do not change the actual database.

#### Real-World Example

Instead of a **column named SUM(price * quantity), rename it TotalRevenue**. Instead of a table named Orders, refer to it as O in a long query.

---

#### SQL Syntax
 
```sql

    -- Column alias
    SELECT 
        column_name AS alias_name 
    FROM table;

    ----------------------------------------------------

    -- Table alias
    SELECT t.column 
    FROM table AS t;

    ----------------------------------------------------

    -- Alias without AS (optional in most DBMS)
    SELECT column alias 
    FROM table;

```

### Examples

```sql

    -- Column alias
    SELECT 
        first_name AS "First Name", 
        last_name AS "Last Name" 
    FROM Employees;

    ----------------------------------------------------

    -- Alias with space or special chars -> use double quotes
    SELECT 
        salary * 1.1 AS "Salary After Raise" 
    FROM Employees;
    
    ----------------------------------------------------


    -- Table alias (especially useful in JOINs)
    SELECT 
        o.order_id, 
        c.name
    FROM Orders AS o
    JOIN Customers AS c ON o.customer_id = c.id;

    ----------------------------------------------------

    -- Aggregate with alias
    SELECT 
        department, 
        AVG(salary) AS avg_salary
    FROM Employees
    GROUP BY department
    HAVING AVG(salary) > 50000;

```
---

### Visual Table Illustration

#### Query without alias:

```sql

    SELECT 
        CONCAT(first_name, ' ', last_name) 
    FROM Employees; 

    → output column has no meaningful name.

```
#### Query with alias:

```sql 
    SELECT 
        CONCAT(first_name, ' ', last_name) AS full_name 
    FROM Employees;
```        

#### Result:

| full_name      |
|----------------|
| John Smith     |
| Alice Johnson  |


---

#### Practice Questions

> Write a query that displays product name and price with a 10% increase, aliasing the new column as price_with_tax.

> Use a table alias to shorten a query: SELECT * FROM Products AS p WHERE p.price < 100.

> Create an alias for the COUNT(*) column.

---

#### Quiz

**3. Which keyword is used to assign a temporary name?**

    a) NAME
    b) AS
    c) ALIAS

---

### Summary for Aliases

> Aliases improve readability, especially with complex expressions, functions, or multiple tables.

---

## 4. **CASE Statements**

**CASE provides if‑then‑else logic inside SQL.** It evaluates conditions and returns a value when the first condition is met (like a switch or if‑else). 

**CASE can be used in SELECT, WHERE, ORDER BY, and even GROUP BY.**

---

#### Real-World Example

Classify orders as **‘Small’, ‘Medium’, or ‘Large’** based on amount:

    CASE WHEN amount < 100 THEN 'Small' WHEN amount BETWEEN 100 AND 500 THEN 'Medium' ELSE 'Large' END

---

#### SQL Syntax

```sql
    CASE
        WHEN condition1 THEN result1
        WHEN condition2 THEN result2
        ...
        ELSE default_result
    END
```    

#### Examples

```sql
    -- Simple classification
    SELECT 
        product_name, 
        price,
        CASE
            WHEN price < 50 THEN 'Budget'
            WHEN price BETWEEN 50 AND 200 THEN 'Standard'
            ELSE 'Premium'
        END AS price_tier
    FROM Products;

    ----------------------------------------------------

    -- Using CASE in aggregate (conditional counting)
    SELECT
        COUNT(CASE WHEN status = 'Shipped' THEN 1 END) AS shipped_orders,
        COUNT(CASE WHEN status = 'Pending' THEN 1 END) AS pending_orders
    FROM Orders;

    ----------------------------------------------------

    -- CASE in ORDER BY (custom sort)
    SELECT 
        name, 
        category
    FROM Products
    ORDER BY
        CASE category
            WHEN 'Electronics' THEN 1
            WHEN 'Furniture' THEN 2
            ELSE 3
        END;

    ----------------------------------------------------        

    -- CASE in WHERE (conditional filter)
    SELECT * FROM Employees
    WHERE
        CASE
            WHEN department = 'Sales' THEN salary > 60000
            WHEN department = 'IT' THEN salary > 80000
            ELSE salary > 50000
        END;

```    
---
### Visual Table Illustration

#### Table: Students

| Name    | Score |
|---------|-------|
| Alice   | 85    |
| Bob     | 45    |
| Charlie | 92    |
| Diana   | 67    |

#### Query:

```sql

    SELECT 
        Name, 
        Score,
    CASE
        WHEN Score >= 90 THEN 'A'
        WHEN Score >= 80 THEN 'B'
        WHEN Score >= 70 THEN 'C'
        WHEN Score >= 60 THEN 'D'
        ELSE 'F'
    END AS Grade
    FROM Students;

```

#### Result

| Name    | Score | Grade |
|---------|-------|-------|
| Alice   | 85    | B     |
| Bob     | 45    | F     |
| Charlie | 92    | A     |
| Diana   | 67    | D     |

---

#### Practice Questions
 
> Write a query that labels employees as ‘Junior’ (salary < 40000), ‘Mid’ (40000-70000), ‘Senior’ (>70000).

> Use CASE to count how many orders are ‘High’ (>1000), ‘Medium’ (500-1000), ‘Low’ (<500).

> Sort products by a custom priority: ‘Discontinued’ last, then by name.

---

#### Quiz

**4. Can CASE be used inside SELECT, WHERE, and ORDER BY?**

    a) Only SELECT
    b) Only WHERE and ORDER BY
    c) All three (and more)

---

### Summary for CASE
    
> CASE brings conditional logic into SQL, enabling dynamic values, custom grouping, and flexible sorting.

---

## **Practice Questions (Comprehensive)**

#### Use a table Sales:

| SaleID | Region | Product  | Amount | SaleDate   |
|--------|--------|----------|--------|------------|
| 1      | North  | Laptop   | 1200   | 2025-01-10 |
| 2      | South  | Mouse    | 25     | 2025-01-15 |
| 3      | North  | Keyboard | 80     | 2025-02-01 |
| 4      | South  | Laptop   | 1100   | 2025-02-10 |
| 5      | East   | Mouse    | 30     | 2025-02-20 |
| 6      | North  | Monitor  | 300    | 2025-03-01 |


#### Write queries:


> Show total sales amount per region. Use alias total_sales.

> Show regions where total sales exceed 1000.

> Count how many sales occurred in each product category (Product column).

> For each region, show the average amount, but only for regions with an average > 200.

> Use CASE to classify each sale amount: ‘High’ (>1000), ‘Medium’ (100–1000), ‘Low’ (<100). Display SaleID, Amount, and the classification.

> Write a query that groups sales by a custom season based on SaleDate:

* ‘Winter’ for January, February

* ‘Spring’ for March, April, May

* Else ‘Other’. Then count how many sales per season.

#### Answers (self‑check):

```sql

    -- 1
    SELECT 
        Region, 
        SUM(Amount) AS total_sales 
    FROM Sales 
    GROUP BY Region;

    ----------------------------------------------------

    -- 2
    SELECT 
        Region, 
        SUM(Amount) AS total_sales 
    FROM Sales 
    GROUP BY Region 
    HAVING SUM(Amount) > 1000;

    ----------------------------------------------------

    -- 3
    SELECT 
        Product, 
        COUNT(*) AS sale_count 
    FROM Sales 
    GROUP BY Product;

    ----------------------------------------------------

    -- 4
    SELECT 
        Region, 
        AVG(Amount) AS avg_amount 
    FROM Sales 
    GROUP BY Region 
    HAVING AVG(Amount) > 200;

    ----------------------------------------------------

    -- 5
    SELECT 
        SaleID, 
        Amount,
        CASE
            WHEN Amount > 1000 THEN 'High'
            WHEN Amount BETWEEN 100 AND 1000 THEN 'Medium'
            ELSE 'Low'
        END AS classification
    FROM Sales;
    
    ----------------------------------------------------

    -- 6
    SELECT 
        CASE
            WHEN MONTH(SaleDate) IN (1,2) THEN 'Winter'
            WHEN MONTH(SaleDate) IN (3,4,5) THEN 'Spring'
            ELSE 'Other'
        END AS season,
        COUNT(*) AS sale_count
    FROM Sales
    GROUP BY 
        CASE
            WHEN MONTH(SaleDate) IN (1,2) THEN 'Winter'
            WHEN MONTH(SaleDate) IN (3,4,5) THEN 'Spring'
            ELSE 'Other'
        END;

```
---
### **Quiz (Answers)**

1. b) Aggregate functions

2. c) HAVING

3. b) AS

4. c) All three (and more)

---

### **Interview Questions**

**1. Q: What is the difference between WHERE and HAVING?**

**A:** 
> WHERE filters rows before aggregation; 

> HAVING filters groups after aggregation. 

> You can use HAVING with aggregate conditions (e.g., SUM(amount) > 1000) but not WHERE.

---

**2. Q: Can you use a column alias in GROUP BY?**

**A:** 

**In most SQL databases (e.g., MySQL, PostgreSQL) yes,** but in SQL Server, no. It’s safer to repeat the expression or use a subquery.

---
**3. Q: Write a CASE expression that returns ‘Even’ or ‘Odd’ for an order_id.**

**A:** 

CASE WHEN order_id % 2 = 0 THEN 'Even' ELSE 'Odd' END

---

**Q: How do you group by a calculated field like YEAR(order_date)?**

**A:** 
```sql

    SELECT 
        YEAR(order_date) AS order_year, 
        SUM(amount) 
    FROM Orders 
    GROUP BY YEAR(order_date);

```
> (or use the alias in some DBMS).     

---

**5. Q: What happens if you use GROUP BY without any aggregate function?**

**A:** 

**You’ll get one row per distinct combination of grouped columns, but non‑aggregated columns will return arbitrary (often first) values** – this is allowed in some DBMS (like MySQL with ONLY_FULL_GROUP_BY disabled) but is not standard SQL and can be misleading.

---

## **Assignment**

> Scenario: You are a data analyst for an online bookstore. 

> Tables: Books, Customers, Orders, OrderDetails.

### Tasks:

**1. Create the following tables (adapt data types as needed):**

```sql

    CREATE TABLE Books (
        BookID INT PRIMARY KEY,
        Title VARCHAR(200),
        Genre VARCHAR(50),
        Price DECIMAL(6,2)
    );

    ----------------------------------------------------

    CREATE TABLE Customers (
        CustomerID INT PRIMARY KEY,
        Name VARCHAR(100),
        City VARCHAR(50),
        JoinDate DATE
    );

    ----------------------------------------------------

    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT,
        OrderDate DATE,
        Status VARCHAR(20)
    );

    ----------------------------------------------------

    CREATE TABLE OrderDetails (
        DetailID INT PRIMARY KEY,
        OrderID INT,
        BookID INT,
        Quantity INT
    );

```

2. **Insert sufficient sample data (at least 6 books, 4 customers, 8 orders with multiple details).**

3. **Write and execute SQL queries to answer:**

    **Grouped total:** For each genre, calculate total revenue (SUM(Price * Quantity) using JOINs – but for now, assume OrderDetails has a Price column or join with Books). (If you cannot join yet, simplify by adding UnitPrice to OrderDetails.)

    **Filter groups:** Show only genres whose total revenue exceeds $200.

    **CASE classification:** For each order, classify its total value (sum of item prices) as ‘Low’ (<50), ‘Medium’ (50–200), ‘High’ (>200). Use CASE.

    **Alias usage:** Write a query that uses table aliases to join Books and OrderDetails, and column aliases for calculated totals.

    **Custom grouping with CASE:** Group customers into ‘New’ (joined in last 30 days) and ‘Existing’, then count how many orders each group placed.

    **HAVING with COUNT:** Find customers who have placed more than 2 orders.

4. **Bonus:** Use CASE inside ORDER BY to show orders first by status: ‘Pending’ first, then ‘Shipped’, then ‘Cancelled’.


---- 

## **Summary of Phase 4**

| Concept   | Purpose                                                                 |
|-----------|-------------------------------------------------------------------------|
| **GROUP BY**  | Divide data into groups for per-group aggregation.                      |
| **HAVING**    | Filter groups based on aggregate results (like `WHERE` for groups).     |
| **Aliases**   | Temporarily rename columns or tables for readability.                   |
| **CASE**      | Implement conditional logic inside SQL – dynamic columns, custom sorts. |

---


<br/><br/><br/>

<center><b>Happy Learning! 😊</b></center>


