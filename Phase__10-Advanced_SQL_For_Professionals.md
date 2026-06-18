# Phase 10: Advanced SQL for Professionals

*   **Window Functions**
*   **Ranking Functions**
*   **Partitioning**
*   **Stored Procedures**
*   **Triggers**
*   **Database Architecture Concepts**


---

### Overview

Welcome to the final frontier of SQL. This phase elevates you from a proficient SQL developer to a data professional capable of building complex analytical pipelines, automating database logic, and architecting scalable systems. 

**We cover:**

* **Window Functions** – perform calculations across a set of rows related to the current row (without collapsing rows).

* **Ranking Functions** – assign ranks, dense ranks, and row numbers to result sets.

* **Partitioning** – split large tables into smaller, manageable pieces for performance and maintenance.

* **Stored Procedures** – encapsulate business logic and reusable SQL code on the server.

* **Triggers** – automatically execute logic in response to data changes.

* **Database Architecture Concepts** – understand the big picture: how databases are structured, distributed, and managed in enterprise environments.

---

## 1. Window Functions

**A window function performs a calculation across a set of rows that are related to the current row.** Unlike aggregate functions (which collapse rows into a single output), window functions preserve individual rows while adding an aggregated or computed value to each row.

The "**window**" is defined using the **OVER()** clause, which specifies how to partition and order the rows for the calculation.


#### Key concepts:

* **PARTITION BY** – divides rows into groups (like GROUP BY but without collapsing).

* **ORDER BY** – defines the order within each partition.

* **Frame (optional)** – specifies a subset of rows within the partition (e.g., ROWS BETWEEN 2 PRECEDING AND CURRENT ROW).

#### Common window functions:

* **Aggregate functions used as window functions**: SUM() OVER(), AVG() OVER(), COUNT() OVER(), MIN() OVER(), MAX() OVER().

* **Ranking functions**: ROW_NUMBER(), RANK(), DENSE_RANK(), NTILE().

* **Value functions**: LAG(), LEAD(), FIRST_VALUE(), LAST_VALUE().

#### Real-World Example

A sales manager needs to see:

* Each sale's amount.

* The running total of sales for that salesperson (cumulative sum).

* The percentage of each sale compared to the salesperson's total.

A window function computes the running total and percentage without grouping or losing row-level detail.

---

#### SQL Syntax

```sql
    SELECT column1, column2,
        window_function(column) OVER (
            [PARTITION BY partition_column]
            [ORDER BY order_column]
            [frame_clause]
        ) AS alias
    FROM table;
```

#### Examples

**Example 1: Running total (cumulative sum)**

```sql
    SELECT 
        SaleDate,
        Salesperson,
        Amount,
        SUM(Amount) OVER (PARTITION BY Salesperson ORDER BY SaleDate) AS RunningTotal
    FROM Sales;
```

**Example 2: Moving average (3-day average)**

```sql
    SELECT 
        SaleDate,
        Amount,
        AVG(Amount) OVER (ORDER BY SaleDate 
                            ROWS BETWEEN 2 PRECEDING AND 
                            CURRENT ROW) AS MovingAvg3Days
    FROM Sales;
```

**Example 3: Percent of total per group**

```sql
    SELECT 
        Product,
        Category,
        Amount,
        Amount * 1.0 / SUM(Amount) OVER (PARTITION BY Category) AS PctOfCategory
    FROM Sales;
```

**Example 4: LAG and LEAD – compare with previous/next row**

```sql
    SELECT 
        SaleDate,
        Amount,
        LAG(Amount, 1) OVER (ORDER BY SaleDate) AS PreviousDayAmount,
        LEAD(Amount, 1) OVER (ORDER BY SaleDate) AS NextDayAmount
    FROM DailySales;
```

---

### Visual Table Illustration

**Table: Sales**

| SaleID | Salesperson | SaleDate   | Amount |
|--------|-------------|------------|--------|
| 1      | Alice       | 2025-01-01 | 100    |
| 2      | Alice       | 2025-01-02 | 150    |
| 3      | Alice       | 2025-01-03 | 80     |
| 4      | Bob         | 2025-01-01 | 200    |
| 5      | Bob         | 2025-01-02 | 120    |

**Query:**

```sql
    SELECT 
        Salesperson,
        SaleDate,
        Amount,
        SUM(Amount) OVER (PARTITION BY Salesperson ORDER BY SaleDate) AS RunningTotal
    FROM Sales;
```
**Result:**

| Salesperson | SaleDate   | Amount | RunningTotal |
|-------------|------------|--------|--------------|
| Alice       | 2025-01-01 | 100    | 100          |
| Alice       | 2025-01-02 | 150    | 250          |
| Alice       | 2025-01-03 | 80     | 330          |
| Bob         | 2025-01-01 | 200    | 200          |
| Bob         | 2025-01-02 | 120    | 320          |

---

#### Practice Questions

1. Write a query to show each employee's salary and the average salary of their department using a window function.

2. Calculate the running total of orders per customer, ordered by order date.

3. For each product, show its price and the difference from the previous product's price (ordered by price).

#### Quiz

**1. What is the main difference between GROUP BY and a window function?**

    a) Window functions are faster
    b) Window functions preserve individual rows while adding aggregated values
    c) GROUP BY can only be used with SUM()

**Answer**: b

---

## 2. Ranking Functions

Ranking functions assign a rank, row number, or percentile to each row within a partition of a result set. They are a subset of window functions and are essential for top‑N queries, pagination, and data analysis.

| Function        | Description                                                                                          |
|-----------------|------------------------------------------------------------------------------------------------------|
| ROW_NUMBER()    | Assigns a unique sequential integer (1, 2, 3, ...) within each partition. Ties are broken arbitrarily. |
| RANK()          | Same as row number, but when there are ties, they receive the same rank and the next rank is skipped (e.g., 1, 2, 2, 4). |
| DENSE_RANK()    | Same as RANK, but no gaps (e.g., 1, 2, 2, 3).                                                       |
| NTILE(n)        | Divides rows into n roughly equal buckets (1 to n). Useful for quartiles, percentiles.               |

---

#### Real-World Example

* **ROW_NUMBER**: paginate a list of employees (show rows 11–20).

* **RANK**: get top 3 salespeople (if two are tied for #1, they both get rank 1, and the next gets rank 3).

* **DENSE_RANK**: assign medals (gold, silver, bronze) – if tied for gold, both get gold, and the next gets silver.

* **NTILE**: assign customers to 10 deciles based on spending.

---

#### SQL Syntax

```sql
    SELECT 
        column,
        ROW_NUMBER() OVER (PARTITION BY partition_col 
                            ORDER BY order_col) AS row_num,
        RANK() OVER (PARTITION BY partition_col 
                        ORDER BY order_col) AS rank_col,
        DENSE_RANK() OVER (PARTITION BY partition_col 
                            ORDER BY order_col) AS dense_rank_col,
        NTILE(4) OVER (ORDER BY order_col) AS quartile
    FROM table;
```

#### Examples

**Example 1: ROW_NUMBER for pagination**

```sql

    SELECT * FROM (
        SELECT 
            EmployeeID,
            Name,
            Salary,
            ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum
        FROM Employees
    ) AS Ranked
    WHERE RowNum BETWEEN 11 AND 20;  -- second page of top salaries

```

**Example 2: RANK vs DENSE_RANK**

```sql
    SELECT 
        Name,
        Score,
        RANK() OVER (ORDER BY Score DESC) AS Rank,
        DENSE_RANK() OVER (ORDER BY Score DESC) AS DenseRank
    FROM Students;
```

**Example 3: NTILE – top 25% performers**

```sql
    SELECT 
        Name,
        Sales,
        NTILE(4) OVER (ORDER BY Sales DESC) AS Quartile
    FROM Salespeople
    WHERE Quartile = 1;  -- top 25%
```    

**Example 4: Ranking within departments**

```sql
    SELECT 
        EmployeeID,
        Name,
        Department,
        Salary,
        RANK() OVER (PARTITION BY Department ORDER BY Salary DESC) AS DeptSalaryRank
    FROM Employees;
```

---

### Visual Table Illustration

#### Table: Students

| Name  | Score |
|-------|-------|
| Alice | 95    |
| Bob   | 90    |
| Carol | 90    |
| David | 85    |

**Query:**

```sql
    SELECT 
        Name,
        Score,
        ROW_NUMBER() OVER (ORDER BY Score DESC) AS RowNum,
        RANK() OVER (ORDER BY Score DESC) AS Rank,
        DENSE_RANK() OVER (ORDER BY Score DESC) AS DenseRank
    FROM Students;
```

**Result:**

| Name  | Score | RowNum | Rank | DenseRank |
|-------|-------|--------|------|-----------|
| Alice | 95    | 1      | 1    | 1         |
| Bob   | 90    | 2      | 2    | 2         |
| Carol | 90    | 3      | 2    | 2         |
| David | 85    | 4      | 4    | 3         |

---

#### Practice Questions


1. List all products ordered by price descending, and assign a row number.

2. Find the top 3 best-selling products in each category using RANK().

3. Divide customers into 5 quintiles based on their total purchase amount using NTILE.

---

#### Quiz

**2. If two students have the same score, which function skips the next rank value?**

    a) ROW_NUMBER()
    b) RANK()
    c) DENSE_RANK()

**Answer**: b


---

## 3. Partitioning (Table Partitioning)

**Partitioning is the physical division of a large table into smaller, more manageable pieces** (partitions) **based on a column** (e.g., date range, geographical region). It is different from PARTITION BY in window functions (logical grouping). This is a physical database design feature that:

* Improves query performance (partition pruning – scanning only relevant partitions).

* Simplifies data management (archiving old partitions, dropping partitions quickly).

* Increases availability (maintenance on one partition doesn't lock the whole table).

**Types of partitioning:**

* **Range partitioning** – by ranges (e.g., sales_date).

* **List partitioning** – by discrete values (e.g., region = 'North', 'South').

* **Hash partitioning** – by a hash function for even distribution.

* **Composite partitioning** – combination of two methods (e.g., range + hash).

---

#### Real-World Example


An e‑commerce platform stores 10 million orders. Queries often filter by OrderDate. By range‑partitioning the Orders table by month, the database can quickly scan only the relevant month's partition instead of the entire table.

---

#### SQL Syntax (Examples vary by DBMS)

**MySQL Partitioning:**

```sql
    CREATE TABLE Orders (
        OrderID INT,
        OrderDate DATE,
        Amount DECIMAL(10,2)
    )
    PARTITION BY RANGE (YEAR(OrderDate)) (
        PARTITION p2022 VALUES LESS THAN (2023),
        PARTITION p2023 VALUES LESS THAN (2024),
        PARTITION p2024 VALUES LESS THAN (2025),
        PARTITION p_future VALUES LESS THAN MAXVALUE
    );
```

**PostgreSQL Partitioning (declarative):**

```sql
    CREATE TABLE Orders (
        OrderID INT,
        OrderDate DATE NOT NULL,
        Amount DECIMAL(10,2)
    ) PARTITION BY RANGE (OrderDate);

    CREATE TABLE Orders_2024 PARTITION OF Orders
        FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

    CREATE TABLE Orders_2025 PARTITION OF Orders
        FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

#### Examples

**Example 1: Range partition by date (PostgreSQL)**

```sql

    -- Parent table
    CREATE TABLE Sales (
        SaleID INT,
        SaleDate DATE,
        ProductID INT,
        Amount DECIMAL
    ) PARTITION BY RANGE (SaleDate);

    -- Create partitions
    CREATE TABLE Sales_2023 PARTITION OF Sales
        FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
    CREATE TABLE Sales_2024 PARTITION OF Sales
        FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

```

**Example 2: List partition by region (MySQL)**

```SQL
    CREATE TABLE Customers (
        CustomerID INT,
        Name VARCHAR(100),
        Region VARCHAR(20)
    )
    PARTITION BY LIST (Region) (
        PARTITION p_north VALUES IN ('North', 'Northeast'),
        PARTITION p_south VALUES IN ('South', 'Southeast'),
        PARTITION p_west VALUES IN ('West', 'Southwest')
    );
```

**Example 3: Hash partition for even distribution (MySQL)**

```sql
    CREATE TABLE UserSessions (
        SessionID INT,
        UserID INT,
        LoginTime DATETIME
    )
    PARTITION BY HASH (UserID)
    PARTITIONS 4;
```

**Example 4: Dropping a partition (fast archiving)**

```sql
    -- MySQL
    ALTER TABLE Sales DROP PARTITION p2022;

    -- PostgreSQL (detach then drop)
    ALTER TABLE Sales DETACH PARTITION Sales_2022;
    DROP TABLE Sales_2022;
```

---

### Visual Table Illustration

#### Orders table partitioned by year:

| Partition Name | Contains Years | Size (approx) |
|----------------|----------------|---------------|
| p2022          | 2022           | 2 GB          |
| p2023          | 2023           | 2.5 GB        |
| p2024          | 2024           | 3 GB          |

**Query:** 

```sql
    SELECT * 
    FROM Orders 
    WHERE OrderDate = '2024-06-15';
```    

> Only scans p2024 – not the entire 7.5 GB.

---

#### Practice Questions

1. What is the benefit of partitioning a large table by date?

2. Write a CREATE TABLE statement with range partitioning by year for a Transactions table.

3. How do you add a new partition for the upcoming year?

---

#### Quiz

**3. Partitioning helps with:**

    a) Data encryption
    b) Performance through partition pruning
    c) Enforcing primary keys

**Answer**: b

---

## 4. Stored Procedures

**A stored procedure is a pre‑compiled collection of SQL statements and logic stored on the database server**. It is executed as a single unit and can accept input parameters, return output parameters, and perform complex operations.

#### Advantages:

* **Modularity** – encapsulate business logic in the database.

* **Performance** – reduce network traffic (send just the procedure call, not multiple queries).

* **Security** – grant users permission to execute a procedure without giving direct table access.

* **Maintainability** – update logic in one place.

**Disadvantages:**

* Vendor‑specific syntax (portability issues).

* Harder to version‑control and debug compared to application code.

#### Real-World Example

An application needs to add a new order: insert into _Orders_, insert into _OrderItems_, update stock, and check inventory. Instead of sending 4 separate queries over the network, the app calls a stored procedure _PlaceOrder()_ with the order details. The procedure handles all steps and returns a success/failure status.


---

#### SQL Syntax (MySQL)


```sql

    DELIMITER //

    CREATE PROCEDURE procedure_name (
        [IN param1 datatype, OUT param2 datatype, INOUT param3 datatype]
    )
    BEGIN
        -- SQL statements
        -- Control flow (IF, CASE, LOOP, etc.)
        -- Error handling
    END //

    DELIMITER ;

    -- Execute
    CALL procedure_name(param1, @out_param);
```

#### Examples

**Example 1: Simple procedure with no parameters**

```sql

    DELIMITER //
    CREATE PROCEDURE GetTopProducts()
    BEGIN
        SELECT * FROM Products ORDER BY Price DESC LIMIT 10;
    END //
    DELIMITER ;

    CALL GetTopProducts();
```

**Example 2: Procedure with IN and OUT parameters**

```sql

    DELIMITER //
    CREATE PROCEDURE GetEmployeeCount(
        IN dept_id INT,
        OUT emp_count INT
    )
    BEGIN
        SELECT COUNT(*) INTO emp_count FROM Employees WHERE DepartmentID = dept_id;
    END //
    DELIMITER ;

    CALL GetEmployeeCount(10, @count);
    SELECT @count;

```

**Example 3: Procedure with transaction and error handling**

```sql

    DELIMITER //
    CREATE PROCEDURE PlaceOrder(
        IN p_CustomerID INT,
        IN p_ProductID INT,
        IN p_Quantity INT,
        OUT p_OrderID INT
    )
    BEGIN
        DECLARE EXIT HANDLER FOR SQLEXCEPTION
        BEGIN
            ROLLBACK;
            RESIGNAL;
        END;
        
        START TRANSACTION;
        
        -- Check stock
        IF (SELECT Stock FROM Products WHERE ProductID = p_ProductID) < p_Quantity THEN
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient stock';
        END IF;
        
        -- Insert order
        INSERT INTO Orders (CustomerID, OrderDate) VALUES (p_CustomerID, NOW());
        SET p_OrderID = LAST_INSERT_ID();
        
        -- Insert order item
        INSERT INTO OrderItems (OrderID, ProductID, Quantity) VALUES (p_OrderID, p_ProductID, p_Quantity);
        
        -- Update stock
        UPDATE Products SET Stock = Stock - p_Quantity WHERE ProductID = p_ProductID;
        
        COMMIT;
    END //
    DELIMITER ;

    CALL PlaceOrder(5, 10, 2, @order_id);
    SELECT @order_id;
```

**Example 4: Procedure with loop**

```sql
    DELIMITER //
    CREATE PROCEDURE InsertSampleData(IN num_rows INT)
    BEGIN
        DECLARE i INT DEFAULT 1;
        WHILE i <= num_rows DO
            INSERT INTO Logs (Message, LogTime) VALUES (CONCAT('Log entry ', i), NOW());
            SET i = i + 1;
        END WHILE;
    END //
    DELIMITER ;
```

---

### Visual Table Illustration


**Application ↔ Database interaction:**

#### Without procedure:

```text
    App → (SQL1) → DB
    App ← (Result1) ← DB
    App → (SQL2) → DB
    App ← (Result2) ← DB
```

> Network round trips: many.

#### With procedure:

```text
    App → CALL Procedure(params) → DB
        (Procedure executes all SQL internally)
    App ← (Result) ← DB
```        

> Network round trips: one.

---

#### Practice Questions

1. Write a stored procedure that takes a ProductID and returns the product name and price.

2. Write a procedure that updates the salary of an employee by a given percentage.

3. What are the advantages of using stored procedures over embedding SQL in application code?

---

#### Quiz

**4. A stored procedure is:**

    a) A temporary table
    b) A set of SQL statements stored and executed on the server
    c) A view with parameters

**Answer**: b

---

## 5. Triggers

**A trigger is a stored procedure that automatically executes (fires) in response to a data modification event (INSERT, UPDATE, DELETE) on a specific table**. Triggers are used to enforce complex business rules, audit changes, update summary tables, or maintain history.

**Timing**: BEFORE or AFTER the event.

**Scope**: FOR EACH ROW (row‑level trigger) or FOR EACH STATEMENT (statement‑level trigger).

**Caution**: Triggers are powerful but can introduce hidden complexity, performance overhead, and hard‑to‑debug issues. Use them sparingly.

#### Real-World Example
 
 An audit trail: whenever an employee's salary is updated, automatically log the old and new salary, who made the change, and the timestamp into an AuditLog table.

---

#### SQL Syntax (MySQL)

```sql
    CREATE TRIGGER trigger_name
        {BEFORE | AFTER} {INSERT | UPDATE | DELETE}
        ON table_name FOR EACH ROW
        BEGIN
            -- Trigger logic; can reference OLD and NEW row values
        END;
```        

#### Examples

**Example 1: BEFORE INSERT – enforce business rule**

```sql
    DELIMITER //
    CREATE TRIGGER CheckOrderQuantity
    BEFORE INSERT ON OrderItems
    FOR EACH ROW
    BEGIN
        IF NEW.Quantity <= 0 THEN
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Quantity must be positive';
        END IF;
    END //
    DELIMITER ;
```

**Example 2: AFTER UPDATE – audit log**

```sql
    DELIMITER //
    CREATE TRIGGER AuditSalaryChange
    AFTER UPDATE ON Employees
    FOR EACH ROW
    BEGIN
        IF OLD.Salary != NEW.Salary THEN
            INSERT INTO AuditLog (EmployeeID, OldSalary, NewSalary, ChangeDate, ChangedBy)
            VALUES (NEW.EmployeeID, OLD.Salary, NEW.Salary, NOW(), USER());
        END IF;
    END //
    DELIMITER ;
```
**Example 3: AFTER DELETE – keep history**

```sql
    DELIMITER //
    CREATE TRIGGER ArchiveDeletedOrder
    AFTER DELETE ON Orders
    FOR EACH ROW
    BEGIN
        INSERT INTO Orders_Archive (OrderID, CustomerID, OrderDate, DeletedAt)
        VALUES (OLD.OrderID, OLD.CustomerID, OLD.OrderDate, NOW());
    END //
    DELIMITER ;
```

**Example 4: BEFORE UPDATE – prevent NULL values**

```sql
    DELIMITER //
    CREATE TRIGGER PreventNullName
    BEFORE UPDATE ON Customers
    FOR EACH ROW
    BEGIN
        IF NEW.Name IS NULL OR NEW.Name = '' THEN
            SET NEW.Name = OLD.Name;  -- keep old value
        END IF;
    END //
    DELIMITER ;
```

---

### Visual Table Illustration

**Trigger flow:**

```text
    User issues UPDATE on Employees
        ↓
    BEFORE UPDATE trigger fires
        ↓ (Can modify NEW values or raise error)
    Row is updated
        ↓
    AFTER UPDATE trigger fires
        ↓ (Can insert into audit log, etc.)
    Commit (if no errors)
```

**Audit table after update:**

| EmployeeID | OldSalary | NewSalary | ChangeDate              | ChangedBy |
|------------|-----------|-----------|-------------------------|-----------|
| 101        | 50000     | 55000     | 2025-06-19 10:00:00     | app_user  |

---
#### Practice Questions

1. Write a trigger that ensures the Price in Products is never set to negative.

2. Create an audit trigger that logs all DELETE operations on the Customers table.

3. What is the difference between BEFORE and AFTER triggers?

4. When would you use a trigger instead of application‑level logic?

---

#### Quiz

**5. In a trigger, OLD and NEW refer to:**

    a) Previous and current values of rows in the trigger
    b) Two separate tables
    c) Old and new trigger names

**Answer**: a

---

## 6. Database Architecture Concepts

Database architecture describes the structure, components, and design patterns of database systems. As a professional, you need to understand the high‑level architecture of modern databases.

**Key concepts:**

1. Client‑Server Model – Database servers listen for client connections (applications, reporting tools) and process queries.

2. Database Engine Components:

    * **Query Parser** – parses SQL syntax.

    * **Optimizer** – generates execution plans.

    * **Executor** – executes the plan.

    * **Storage Engine** – manages data on disk (InnoDB in MySQL, Buffer Pool in PostgreSQL).

    * **Transaction Manager** – handles ACID, concurrency, locking.

    * **Log Manager** – writes transaction logs (Write‑Ahead Logging – WAL) for durability.

3. Storage Architecture:

    * **Tablespaces / Data files** – where data is physically stored.

    * **Page/Block** – smallest unit of storage (e.g., 8 KB in PostgreSQL).

    * **Buffer Pool / Shared Buffers** – in‑memory cache for frequently accessed data.

4. High Availability & Replication:

    * **Master‑Slave Replication** – one primary (writes) + multiple replicas (reads).

    * **Clustering** – multiple nodes working together (e.g., Galera Cluster for MySQL).

    * **Sharding** – horizontal partitioning across multiple servers (distributed databases).

    * **Failover** – automatic switching to a standby server in case of failure.

5. Disaster Recovery:
    
    * **Backup & Restore** – full, incremental, and point‑in‑time recovery.

    * **Write‑Ahead Logging (WAL)** – ensures durability and enables recovery.

6. Data Warehousing vs OLTP:

    * **OLTP (Online Transaction Processing)** – high‑volume, low‑latency, frequent small transactions (e.g., banking).

    * **OLAP (Online Analytical Processing)** – read‑intensive, complex aggregations (e.g., reporting, BI).

---

#### Real-World Example

* An e‑commerce giant like Amazon uses:

* OLTP databases (e.g., DynamoDB, RDS) for order processing.

* Replication to distribute read traffic across replicas.

* Sharding to split order data by customer ID across multiple servers.

* WAL and backups to ensure durability and recovery.

* Data warehouses (Redshift, Snowflake) for analytics on historical data.

---

### Visual Table Illustration

#### High‑level Database Architecture (Simplified):

```text
    Client Applications
            ↓
    [Connection Manager / Router]
            ↓
    ┌─────────────────────────────────────┐
    │         Database Engine             │
    │  ┌─────────────────────────────┐   │
    │  │ Query Parser & Optimizer    │   │
    │  └─────────────────────────────┘   │
    │  ┌─────────────────────────────┐   │
    │  │ Execution Engine            │   │
    │  └─────────────────────────────┘   │
    │  ┌─────────────────────────────┐   │
    │  │ Transaction / Lock Manager  │   │
    │  └─────────────────────────────┘   │
    │  ┌─────────────────────────────┐   │
    │  │ Storage Engine (Buffer Pool)│   │
    │  └─────────────────────────────┘   │
    └─────────────────────────────────────┘
            ↓
    [Data Files] ←→ [WAL / Transaction Logs]
```

#### Replication Architecture:

```text

        [Master]
           ↓
    ┌──────┼──────┐
    ↓      ↓      ↓
 [Replica1] [Replica2] [Replica3]
 (Read-only) (Read-only) (Read-only)

```
---
#### Practice Questions

1. What is the role of the query optimizer?

2. Explain the difference between OLTP and OLAP.

3. Why is WAL important for database durability?

4. What is sharding and when would you use it?

---

#### Quiz

**6. Write‑Ahead Logging (WAL) ensures:**

    a) Faster queries
    b) Durability and recovery
    c) Data encryption

**Answer**: b

---

### Practice Questions (Comprehensive)

**Scenario:** 

> You are building an analytics dashboard for a retail chain.

1. **Window Function**: Write a query to show each sale's amount, and the running total of sales for that store, ordered by date.

2. **Ranking**: For each product category, rank products by total sales (using DENSE_RANK) and display only the top 3 per category.

3. **Partitioning**: Design a partition strategy for a Sales table with 100 million rows, which is frequently queried by SaleDate.

4. **Stored Procedure**: Write a stored procedure that, given a date range, returns the total sales per category.

5. **Trigger**: Write a trigger that, whenever a sale is inserted, updates a summary table DailySales (with SaleDate, TotalAmount).

6. **Architecture**: Explain how you would architect a system to handle high write throughput and read scalability for this retail chain.

**Answers (self‑check):**

```sql

    -- 1
    SELECT 
        StoreID,
        SaleDate,
        Amount,
        SUM(Amount) OVER (PARTITION BY StoreID ORDER BY SaleDate) AS RunningTotal
    FROM Sales;

    -- 2
    WITH RankedProducts AS (
        SELECT 
            Category,
            ProductID,
            SUM(Amount) AS TotalSales,
            DENSE_RANK() OVER (PARTITION BY Category ORDER BY SUM(Amount) DESC) AS Rank
        FROM Sales
        GROUP BY Category, ProductID
    )
    SELECT * FROM RankedProducts WHERE Rank <= 3;

    -- 3. Range partition by month/year (PostgreSQL)
    CREATE TABLE Sales PARTITION BY RANGE (SaleDate);
    CREATE TABLE Sales_2024 PARTITION OF Sales FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

    -- 4. Stored procedure (MySQL)
    DELIMITER //
    CREATE PROCEDURE GetSalesByCategory(IN start_date DATE, IN end_date DATE)
    BEGIN
        SELECT Category, SUM(Amount) AS TotalSales
        FROM Sales s
        JOIN Products p ON s.ProductID = p.ProductID
        WHERE SaleDate BETWEEN start_date AND end_date
        GROUP BY Category;
    END //
    DELIMITER ;

    -- 5. Trigger
    DELIMITER //
    CREATE TRIGGER UpdateDailySales
    AFTER INSERT ON Sales
    FOR EACH ROW
    BEGIN
        INSERT INTO DailySales (SaleDate, TotalAmount)
        VALUES (NEW.SaleDate, NEW.Amount)
        ON DUPLICATE KEY UPDATE TotalAmount = TotalAmount + NEW.Amount;
    END //
    DELIMITER ;

    -- 6. Architecture: master for writes, multiple read replicas for dashboard queries; partition by date; cache frequently accessed aggregates.
```

---

### Quiz (Answers)

1. b) Window functions preserve individual rows while adding aggregated values

2. b) RANK()

3. b) Performance through partition pruning

4. b) A set of SQL statements stored and executed on the server

5. a) Previous and current values of rows in the trigger

6. b) Durability and recovery

---

### Interview Questions

1. Explain the difference between ROW_NUMBER(), RANK(), and DENSE_RANK().

    **Answer :**

    A: ROW_NUMBER() assigns a unique number per row, breaking ties arbitrarily. RANK() gives the same rank to ties but skips the next rank value. DENSE_RANK() gives the same rank to ties without skipping.

    ---

2. When would you use a stored procedure versus a view?

    **Answer :**
    
    A: Use a view when you need a virtual table (read‑only, simple logic). Use a stored procedure when you need to perform multiple operations, control flow, transactions, or accept parameters for dynamic logic.

    --- 

3. What are the disadvantages of using triggers?

    **Answer :**

    A: Triggers are implicit – they can cause unexpected side effects. They are hard to debug, can create performance bottlenecks, and may lead to deadlocks. Overuse makes the database difficult to maintain.

    ---

4. What is the difference between horizontal and vertical partitioning?

    **Answer :**

    A: Horizontal partitioning (sharding) splits rows of a table across different tables or servers (e.g., by date). Vertical partitioning splits columns (e.g., moving large BLOB fields to a separate table).

    ---

5. How does indexing work with partitioning?

    **Answer :**

    A: In most databases, you can have local indexes (index per partition) or global indexes. Local indexes are more common and easier to maintain. Partition‑wise index scans speed up queries.

    ---

6. What is an execution plan and how do you use it?

    **Answer :**

    A: An execution plan shows how the database engine will execute a query (join order, index usage, cost). You use EXPLAIN to see it. It helps you identify full table scans, expensive sorts, or missing indexes.

    ---

7. What is the role of the WAL (Write‑Ahead Log) in database architecture?

    **Answer :**

    A: WAL records changes before they are written to the data files. It ensures durability (can recover from crashes) and enables point‑in‑time recovery. It also supports replication by shipping logs to replicas.

    
---

### Assignment

**Scenario:** 

> You are tasked with building a financial reporting system for a bank.


#### Tables:

```sql

    CREATE TABLE Accounts (
        AccountID INT PRIMARY KEY,
        CustomerID INT,
        Balance DECIMAL(15,2)
    );

    CREATE TABLE Transactions (
        TransactionID INT PRIMARY KEY,
        AccountID INT,
        TransactionDate DATE,
        Amount DECIMAL(15,2),
        Type VARCHAR(10) -- 'Deposit', 'Withdrawal'
    );

    CREATE TABLE DailyBalances (
        AccountID INT,
        BalanceDate DATE,
        ClosingBalance DECIMAL(15,2),
        PRIMARY KEY (AccountID, BalanceDate)
    );
```

**Tasks:**

1. **Window Function**: Write a query to show each transaction along with the running balance (cumulative sum of amounts) for that account, ordered by transaction date.

2. **Ranking**: For each customer, rank their accounts by balance (highest first). Show customer ID, account ID, balance, and rank.

3. **Partitioning**: Propose a partition strategy for the Transactions table, which has 500 million rows. Justify your choice.

4. **Stored Procedure**: Write a stored procedure UpdateDailyBalances(IN p_Date DATE) that:

    * For each account, calculates the closing balance for that date.

    * Inserts or updates the DailyBalances table.

5. **Trigger**: Write a trigger on the Transactions table that:

    * Before inserting a transaction, checks that the account balance will not go negative.

    * If the withdrawal exceeds the current balance, raise an error and prevent the transaction.

6. **Architecture**: Design a high‑availability architecture for this banking system. Discuss replication, failover, backups, and disaster recovery.

---

## Summary of Phase 10

| Concept                  | Purpose                                                                 |
|--------------------------|-------------------------------------------------------------------------|
| **Window Functions**     | Perform calculations across a set of rows without grouping (e.g., running totals). |
| **Ranking Functions**    | Assign ranks, row numbers, and percentiles (ROW_NUMBER, RANK, DENSE_RANK, NTILE). |
| **Partitioning**         | Physically divide large tables for performance and maintenance.         |
| **Stored Procedures**    | Encapsulate and reuse business logic on the database server.            |
| **Triggers**             | Automatically execute logic in response to data modifications.          |
| **Database Architecture**| Understand the big picture: engine components, replication, sharding, OLTP vs OLAP. |


---

## Final Summary: SQL Professional Mastery

> You have now completed all 10 phases of the SQL Professional course:

| Phase | Topic                                                                   |
|-------|-------------------------------------------------------------------------|
| 1     | SQL Fundamentals                                                        |
| 2     | Filtering & Data Manipulation                                           |
| 3     | Functions (Scalar & Aggregate)                                          |
| 4     | Grouping & Analysis (GROUP BY, HAVING, CASE)                            |
| 5     | Joins (All types)                                                       |
| 6     | Advanced Queries (Subqueries, CTEs, Views, Temp Tables)                 |
| 7     | Database Design (Normalization, Constraints, Relationships)             |
| 8     | Performance Optimization (Indexes, Execution Plans)                     |
| 9     | Transactions & Security (ACID, Permissions)                             |
| 10    | Advanced SQL for Professionals (Window Functions, Procedures, Triggers, Architecture) |

---

You now have enterprise‑ready SQL skills to:

* Design efficient, normalized database schemas.

* Write complex analytical queries with window functions.

* Automate business logic with procedures and triggers.

* Optimize performance with indexes and partitioning.

* Ensure data safety with transactions and security best practices.

* Architect scalable and highly available database systems.


**Next Steps:**

* Practice with real datasets (Kaggle, public data).

* Explore cloud databases (AWS RDS, Azure SQL, GCP Cloud SQL).

* Learn NoSQL alternatives (MongoDB, DynamoDB) for unstructured data.

* Consider database certifications (Oracle, Microsoft, PostgreSQL).

---

<br/><br/><br/><br/>
### Congratulations, SQL Master! 🥳🎉
