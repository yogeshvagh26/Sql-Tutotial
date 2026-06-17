# Phase 8: Performance Optimization

### Overview

As databases grow from thousands to millions (or billions) of rows, performance becomes critical. Slow queries frustrate users and waste resources. This phase covers the essential tools and techniques to make your SQL queries **fast and efficient**:

* **Indexes** – data structures that speed up data retrieval (like a book’s index).

* **Query Optimization** – writing efficient SQL and understanding how the database executes it.

* **Execution Plans** – the roadmap the database uses to run your query.

* **Performance Tuning** – practical strategies to monitor and improve overall database performance.

---

## 1. Indexes

An index is a separate data structure (usually a B‑tree or hash table) that the database maintains to improve the speed of data retrieval operations (especially **SELECT** queries with **WHERE**, **JOIN**, **ORDER BY**, and **GROUP BY**). Think of it like the index at the back of a book: instead of reading every page to find a topic, you go directly to the correct page number.

#### Key points:

* Indexes are created on one or more columns of a table.

* They dramatically speed up read queries (SELECT).

* They slow down write operations (INSERT, UPDATE, DELETE) because the index must be updated alongside the table.

* Indexes consume storage space (sometimes more than the table itself).

* The most common type is the B‑tree index (default in most databases).

#### Types of indexes:

* **Single‑column index** – on one column.

* **Composite (multi‑column) index** – on multiple columns (order matters!).

* **Unique index** – enforces uniqueness, often used for primary keys.

* **Full‑text index** – for searching text within large text fields.

* **Partial / Filtered index** – on a subset of rows (e.g., WHERE status = 'Active').

#### Real-World Example

You have an Orders table with 10 million rows. Without an index, a query like **SELECT * FROM Orders WHERE CustomerID = 12345;** forces the database to scan all 10 million rows (full table scan). With an index on **CustomerID**, the database uses the index to find matching rows in milliseconds.

---

#### SQL Syntax

```sql

    -- Create an index on a single column
    CREATE INDEX idx_customer_id 
        ON Orders(CustomerID);

    --------------------------------------------------
    -- Create a composite index (order matters: put most selective column first)
    CREATE INDEX idx_customer_date 
        ON Orders(CustomerID, OrderDate);
        
    --------------------------------------------------

    -- Create a unique index
    CREATE UNIQUE INDEX idx_email_unique 
        ON Users(Email);

    --------------------------------------------------

    -- Drop an index
    DROP INDEX idx_customer_id 
        ON Orders;  -- MySQL

    DROP INDEX idx_customer_id;            -- PostgreSQL

```

---

### Examples

**Example 1: Single-column index for WHERE clause**

```sql

    -- Without index: slow on large table
    SELECT * 
    FROM Products 
    WHERE CategoryID = 5;

    -- Create index to speed it up
    CREATE INDEX idx_category 
        ON Products(CategoryID);  

```

**Example 2: Composite index for multiple conditions**

```sql

    -- Query filters on both columns
    SELECT * 
    FROM Orders
    WHERE CustomerID = 100 AND OrderDate > '2025-01-01';

    -- Create composite index (CustomerID first, then OrderDate)
    CREATE INDEX idx_customer_date 
        ON Orders(CustomerID, OrderDate);

```

**Example 3: Index for ORDER BY (avoids filesort)**

```sql

    -- Query with sorting
    SELECT * 
    FROM Employees 
    ORDER BY HireDate DESC;

    -- Create index to avoid sorting step
    CREATE INDEX idx_hiredate 
        ON Employees(HireDate);

```

**Example 4: Covering index (includes all columns in query)**

```sql

    -- Query only needs CustomerID and Total
    SELECT 
        CustomerID, 
        Total 
    FROM Orders 
    WHERE CustomerID BETWEEN 1 AND 1000;

    -- Covering index includes both columns; DB can answer entirely from index, no table access
    CREATE INDEX idx_covering 
        ON Orders(CustomerID, Total);

```

---

### Visual Table Illustration

**Table: Employees (10,000 rows, unsorted by name)**

| EmpID | Name  | Salary | DeptID |
|-------|-------|--------|--------|
| 1     | Alice | 60000  | 10     |
| 2     | Bob   | 50000  | 20     |
| ...   | ...   | ...    | ...    |


**Without index on** _Name_: a search for _WHERE Name = 'Smith'_ **scans all rows**.


**With index** _CREATE INDEX idx_name ON Employees(Name);_:

* The index is a sorted tree of names with pointers to rows.

* Query finds 'Smith' in O(log n) time → instant.

---

####  Practice Questions

1. Why do indexes slow down INSERT, UPDATE, and DELETE?

2. When would you not want to create an index?

3. Write a CREATE INDEX statement to speed up queries filtering by ProductName and sorting by Price.

4. What is a covering index?

---

#### Quiz

**1. An index is most beneficial for:**

    a) INSERT operations
    b) SELECT queries with WHERE conditions
    c) DELETE operations on small tables

---

### Summary for Indexes

> Indexes are essential for fast reads, but they come with storage and write‑overhead costs. Choose indexes based on your most frequent and slowest queries.

---

## 2. Query Optimization

**Query optimization is the art (and science) of writing SQL queries that execute efficiently.** The database’s query optimizer tries to find the best execution plan, but you can help it by writing queries that are clear, avoid unnecessary complexity, and leverage indexes effectively.

#### Key principles:

* **Select only needed columns** – avoid SELECT * unless you truly need all columns.

* **Use WHERE filters early** – reduce the dataset as early as possible.

* **Avoid functions on indexed columns** – e.g., WHERE YEAR(order_date) = 2025 prevents index usage; use WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31'.

* **Join on indexed columns** – foreign keys should generally be indexed.

* **Use EXISTS instead of IN for large subqueries** – often faster.

* **Avoid OR in WHERE** if it prevents index usage; use UNION or IN instead.

* **Be mindful of data types** – comparing a VARCHAR column to a number forces conversion and disables indexes.

#### Real-World Example

#### Slow query:

```sql

    SELECT * 
    FROM Orders 
    WHERE YEAR(OrderDate) = 2024 AND MONTH(OrderDate) = 12;

```

> This applies a function to the column, so any index on OrderDate cannot be used.

#### Optimised query:

```sql

    SELECT * 
    FROM Orders 
    WHERE OrderDate BETWEEN '2024-12-01' AND '2024-12-31';

```

> Now the index on OrderDate can be used.

---

#### SQL Syntax (No specific syntax; it's about writing better SQL)

#### Examples 

**Example 1: Avoid SELECT `*`**

```sql

    -- Bad (retrieves all columns, wastes I/O)
    SELECT * 
    FROM Employees 
    WHERE DeptID = 10;

    -- Good (only needed columns)
    SELECT 
        EmpID, 
        Name, 
        Salary 
    FROM Employees 
    WHERE DeptID = 10;

```

**Example 2: Use `EXISTS` vs `IN`**

```sql

    -- Slower on large datasets (subquery executes fully)
    SELECT Name 
    FROM Customers
    WHERE CustomerID IN (SELECT CustomerID 
                            FROM Orders 
                            WHERE Amount > 1000);

    -- Faster (stops at first match)
    SELECT Name 
    FROM Customers c
    WHERE EXISTS (SELECT 1 
                    FROM Orders o 
                    WHERE o.CustomerID = c.CustomerID AND o.Amount > 1000);

```

**Example 3: Avoid `OR` that prevents index use**

```sql

    -- May use index for one part but scan for the other
    SELECT * 
    FROM Products 
    WHERE CategoryID = 1 OR CategoryID = 2;

    -- Better: use IN (often optimised as range scan)
    SELECT * 
    FROM Products 
    WHERE CategoryID IN (1, 2);

    -- Or use UNION
    SELECT * 
    FROM Products 
    WHERE CategoryID = 1
    UNION ALL
    SELECT * 
    FROM Products 
    WHERE CategoryID = 2;

```

**Example 4: Use `LIKE` properly**

```sql

    -- Can use index (prefix search)
    SELECT * 
    FROM Users 
    WHERE Name LIKE 'John%';

    -- Cannot use index (leading wildcard)
    SELECT * 
    FROM Users 
    WHERE Name LIKE '%Smith';

```

---

### Visual Table Illustration

#### Query execution flow (simplified):

**Without optimization:**

```text
Full Table Scan (1 million rows) → Filter (WHERE) → Return (100 rows)
I/O: 1 million row reads
```

**With optimization:**

```text
Index Seek (5 levels) → Fetch 100 rows
I/O: ~log(1M) + 100 reads → dramatically faster
```

---

#### Practice Questions

1. Rewrite SELECT * FROM Products WHERE UPPER(ProductName) = 'LAPTOP' to allow index usage.

2. Which is generally faster: IN with a large subquery or EXISTS? Why?

3. Why is WHERE salary / 12 > 5000 worse than WHERE salary > 60000?

---

#### Quiz

**2. Which of these prevents index usage?**

    a) WHERE order_date = '2025-01-01'
    b) WHERE YEAR(order_date) = 2025
    c) WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31'

---
### Summary for Query Optimization

> Write queries with the optimizer in mind: select only what you need, avoid functions on indexed columns, and use set‑based operations efficiently.

---

## 3. Execution Plans

An **execution plan** (also called query plan or explain plan) **is the step‑by‑step strategy that the database engine uses to execute your query**. It shows:


* Which indexes (if any) are used.

* How tables are joined (nested loops, hash join, merge join).

* The order of operations.

* Estimated row counts and costs.

Reading execution plans helps you identify bottlenecks: full table scans, expensive sorts, or incorrect join orders.

Most databases provide an EXPLAIN command (or EXPLAIN ANALYZE / EXPLAIN (ANALYZE, BUFFERS) in PostgreSQL) to display the plan.

#### Real-World Example

You run a query that takes 10 seconds. You use EXPLAIN and see a Seq Scan (full table scan) on a large table. You add an index, rerun the query, and the plan now shows an Index Scan. The query runs in 0.1 seconds.

---

#### SQL Syntax

```sql

    -- MySQL
    EXPLAIN SELECT * 
    FROM Orders 
    WHERE CustomerID = 100;

    -- MySQL (actual execution stats)
    EXPLAIN ANALYZE SELECT * 
    FROM Orders 
    WHERE CustomerID = 100;

    -- PostgreSQL
    EXPLAIN (ANALYZE, BUFFERS) SELECT * 
    FROM Orders 
    WHERE CustomerID = 100;

    -- SQL Server
    SET STATISTICS PROFILE ON;
    SELECT * 
    FROM Orders 
    WHERE CustomerID = 100;

    -- SQLite
    EXPLAIN QUERY PLAN SELECT * 
    FROM Orders 
    WHERE CustomerID = 100;

```
---
#### Examples

**Example 1: Simple EXPLAIN output (MySQL style)**

```sql
    EXPLAIN SELECT * FROM Employees WHERE DeptID = 10;
```

| id | select_type | table     | type | possible_keys | key      | rows | Extra        |
|----|-------------|-----------|------|---------------|----------|------|--------------|
| 1  | SIMPLE      | Employees | ref  | idx_dept      | idx_dept | 150  | Using index  |

> **Interpretation**: using idx_dept index, estimated 150 rows returned.


**Example 2: Full table scan (bad)**

| id | select_type | table  | type | possible_keys | key  | rows   | Extra        |
|----|-------------|--------|------|---------------|------|--------|--------------|
| 1  | SIMPLE      | Orders | ALL  | NULL          | NULL | 500000 | Using where  |

> **Interpretation**: ALL means full table scan; no index used. Performance will be poor.

**Example 3: Complex join plan (PostgreSQL style)**

```text
    Hash Join  (cost=100.00..500.00 rows=1000 width=64)
    Hash Cond: (o.customer_id = c.customer_id)
    ->  Seq Scan on orders o  (cost=0.00..300.00 rows=10000)
    ->  Hash  (cost=50.00..50.00 rows=1000)
            ->  Seq Scan on customers c  (cost=0.00..50.00 rows=1000)
```
> **Shows**: hash join, sequential scans, estimated costs.

---
### Visual Table Illustration

#### Query:

```sql
    SELECT * 
    FROM Orders 
    WHERE CustomerID = 1 AND Amount > 100;
```    

#### Execution plan (textual tree):

```text
    Index Scan using idx_customer on Orders (cost=0.42..8.45 rows=3)
    Index Cond: (CustomerID = 1)
    Filter: (Amount > 100)
```
* **Index Scan** – using index on CustomerID.

* **Index Cond** – how the index is used.

* **Filter** – extra condition applied after fetching rows.

---

#### Practice Questions

1. What does a "Seq Scan" or "Table Scan" in an execution plan indicate?

2. What is the difference between EXPLAIN and EXPLAIN ANALYZE?

3. How can you tell from an execution plan whether your index is being used?

4. What does "Using temporary" or "Using filesort" in MySQL's EXPLAIN mean, and why is it bad?


---

#### Quiz

**3. EXPLAIN shows:**

    a) The actual result of the query
    b) The execution plan chosen by the optimizer
    c) The database schema

---

### Summary for Execution Plans

> Execution plans are your primary diagnostic tool for slow queries. Learn to read them to see which indexes are used and where bottlenecks occur.

---

## 4. Performance Tuning

**Performance tuning is the broader discipline of monitoring, analysing, and improving overall database performance**. It goes beyond individual queries and includes:

* **Server configuration** – memory, disk I/O, CPU, connection pooling.

* **Database structure** – proper normalization, indexing strategy, partitioning.

* **Query-level tuning** – rewriting slow queries.

* **Monitoring** – using slow query logs, performance dashboards.

* **Maintenance** – updating statistics, rebuilding indexes (VACUUM / REINDEX / OPTIMIZE TABLE).

* **Hardware** – faster storage (SSD vs HDD), more RAM.

#### The tuning process:

1. **Identify** – Find the slowest queries (use slow query log, pg_stat_statements, etc.).

2. **Analyze** – Use EXPLAIN to understand why they are slow.

3. **Optimize** – Add indexes, rewrite queries, change schema.

4. **Test** – Verify improvement (measure before and after).

5. **Repeat** – Continue for the next bottleneck.

#### Real-World Example

An e‑commerce site's dashboard page loads in 8 seconds. Investigation reveals:

* A query joining 5 tables without proper indexes on foreign keys.

* Running EXPLAIN shows nested loop joins with full table scans.

* Adding indexes on foreign key columns brings the query down to 0.5 seconds.

* The site now loads in under 1 second.

---

#### SQL Syntax (tuning commands)

**1. ANALYZE (update statistics – PostgreSQL / MySQL)**

```sql
    -- PostgreSQL: updates table statistics for the query planner
    ANALYZE Employees;

    -- MySQL: analyzes table and updates index statistics
    ANALYZE TABLE Employees;
```

**2. REINDEX / OPTIMIZE (rebuild indexes)**

```sql
    -- PostgreSQL
    REINDEX INDEX idx_customer;

    -- MySQL
    OPTIMIZE TABLE Orders;  
```

**3. Slow query log (MySQL example)**

```sql
    -- Enable slow query log (in my.cnf or my.ini)
    SET GLOBAL slow_query_log = 'ON';
    SET GLOBAL long_query_time = 2;  -- log queries slower than 2 seconds
```

**4. Monitoring query performance (PostgreSQL)**

```sql
    -- View most expensive queries
    SELECT 
        query, 
        calls, 
        total_time, 
        mean_time
    FROM pg_stat_statements
    ORDER BY mean_time DESC
    LIMIT 10;
```

---

#### Examples

**Example 1: Using EXPLAIN ANALYZE to measure actual time**

```sql
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT * 
    FROM Orders 
    WHERE OrderDate > '2025-01-01';
```

> Shows actual execution time, buffer hits/misses.

**Example 2: Using a covering index to avoid table access**

```sql

    -- Instead of scanning table
    CREATE INDEX idx_covering 
        ON Orders(CustomerID, OrderDate, Total);

    -- Query that uses only index (covering index)
    SELECT 
        CustomerID, 
        OrderDate, 
        Total 
    FROM Orders 
    WHERE CustomerID = 100;

```

**Example 3: Partitioning for large tables (advanced)**

```sql
    -- PostgreSQL example: range partition by year
    CREATE TABLE Orders_2024 
    PARTITION OF Orders 
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Example 4: VACUUM (PostgreSQL) to reclaim storage**

```sql
    VACUUM ANALYZE Orders;  -- reclaims dead tuples and updates statistics
```

---
### Visual Table Illustration

* **Before tuning:**

    | Query            | Execution Time (avg) |
    |------------------|----------------------|
    | Dashboard query  | 8.2s                 |

    ---

* **After adding indexes:**

    | Query            | Execution Time (avg) |
    |------------------|----------------------|
    | Dashboard query  | 0.35s                |

    ---

* **After increasing server memory (buffer pool):**

    | Query            | Execution Time (avg) |
    |------------------|----------------------|
    | Dashboard query  | 0.12s                |


---

#### Practice Questions

1. What is the difference between ANALYZE and REINDEX?

2. How do you identify which queries are the slowest in your production database?

3. Why is it important to update database statistics regularly?

4. What does VACUUM do in PostgreSQL? Why is it needed?


#### Quiz

**4. Which of the following is not a database‑level performance tuning technique?**

    a) Adding indexes
    b) Using a covering index
    c) Adding a ORDER BY clause to every query


---

### Summary for Performance Tuning

Performance tuning is a continuous process: identify slow queries, analyze plans, apply fixes (indexes, rewrites, configuration), and monitor results.


---

### Practice Questions (Comprehensive)

> Given the following slow query on a table Sales with 5 million rows:

```sql
    SELECT
        ProductID,
        SUM(Amount) AS TotalSales
    FROM Sales
    WHERE YEAR(SaleDate) = 2024
    GROUP BY ProductID
    ORDER BY TotalSales DESC
    LIMIT 10;
```

It takes 15 seconds.

1. Why is this query slow? Identify at least two issues.

2. How would you rewrite the query to improve performance?

3. What index(es) would you create to speed it up?

4. How would you use EXPLAIN to verify the improvement?

5. What maintenance tasks would you schedule to keep performance stable?

---

### Answers (self‑check):

```sql
    -- 1. Issues: YEAR(SaleDate) prevents index use; no index on SaleDate; sorting/grouping on large dataset; SELECT * would be worse.
    -- 2. Rewrite:
    SELECT
        ProductID,
        SUM(Amount) AS TotalSales
    FROM Sales
    WHERE SaleDate BETWEEN '2024-01-01' AND '2024-12-31'
    GROUP BY ProductID
    ORDER BY TotalSales DESC
    LIMIT 10;

    -- 3. Create a composite index covering SaleDate and ProductID, and possibly include Amount for covering:
    CREATE INDEX idx_sales_date_product 
        ON Sales(SaleDate, ProductID);
    -- Or a covering index:
    CREATE INDEX idx_covering 
        ON Sales(SaleDate, ProductID, Amount);

    -- 4. Use EXPLAIN to see Index Scan vs Table Scan, and lower row estimates.

    -- 5. Regular ANALYZE to update stats; monitor slow query log; vacuum/optimize regularly.
```

---

### Quiz (Answers)

1. b) SELECT queries with WHERE conditions

2. b) WHERE YEAR(order_date) = 2025

3. b) The execution plan chosen by the optimizer

4. c) Adding a ORDER BY clause to every query

---

### Interview Questions

1. **What is the difference between a clustered and a non‑clustered index?**

    **Answer**

    A clustered index determines the physical order of data rows (table itself is the index). There can be only one per table. A non‑clustered index is a separate structure that points to the data rows. Primary keys are often clustered by default.

    ---

2. **How does a composite index work? Does column order matter?**

    **Answer**

    Yes, column order matters. The index is sorted first by the first column, then the second, etc. Queries that filter on the first column (or first + second) can use the index. Queries that filter only on the second column typically cannot.

    ---

3. **What is a full table scan and why is it problematic?**

    **Answer**

    A full table scan reads every row of the table to find matches. It is problematic for large tables because it requires massive I/O. Indexes reduce this to a few index levels + row lookups.

    ---

4. **What is the difference between EXPLAIN and EXPLAIN ANALYZE?**

    **Answer**

    EXPLAIN shows the plan the optimizer estimates (costs, row counts). EXPLAIN ANALYZE actually runs the query and shows actual execution times and row counts, which is more accurate for tuning.

    ---

5. **Can you have too many indexes?**

    **Answer**

    Yes. Each index adds overhead to INSERT, UPDATE, DELETE. Also, if the optimizer chooses the wrong index, performance can degrade. Maintain only indexes that benefit your most important queries.

    ---

6. **What is a covering index?**

    **Answer**

    An index that contains all columns needed by a query. The query can be answered entirely from the index without accessing the table data, which is very fast.

    ---

7. **How often should you run ANALYZE or update statistics?**

    **Answer**

    After significant data changes (large inserts/updates/deletes). Many databases have autovacuum / auto‑analyze features; tune their frequency to match your workload.

    ---


### Assignment

**Scenario:**

> You are the DBA for a growing online bookstore. The database has grown to 2 million orders. Users complain about slow reports.

**Tasks:**

1. **Analyze the current schema** (simplified):

```sql

    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT,
        OrderDate DATETIME,
        Status VARCHAR(20),
        Total DECIMAL(10,2)
    );

    CREATE TABLE OrderItems (
        ItemID INT PRIMARY KEY,
        OrderID INT,
        ProductID INT,
        Quantity INT,
        Price DECIMAL(10,2)
    );
    -- No indexes besides primary keys.

```

2. **Given the following two slow queries:**

    * Query A: 
        
        ```sql
            SELECT * 
            FROM Orders 
            WHERE YEAR(OrderDate) = 2023 AND Status = 'Shipped'; 
            --(takes 12s)
            
        ```

    * Query B: 
        ```sql
            SELECT 
                o.OrderID, 
                SUM(oi.Quantity * oi.Price) AS Revenue 
            FROM Orders o 
            JOIN OrderItems oi 
                ON o.OrderID = oi.OrderID 
            WHERE o.OrderDate > '2025-01-01' 
            GROUP BY o.OrderID 
            ORDER BY Revenue DESC 
            LIMIT 20; -- (takes 25s)
        ```

3. **Write a tuning report:**

    * a. Identify the problems with each query (be specific).

    * b. Rewrite each query to be more efficient (where possible).

    * c. Recommend specific indexes to create (write the CREATE INDEX statements).

    * d. Use EXPLAIN to justify your recommendations (write what you expect to see).

    * e. Suggest any additional database maintenance tasks (statistics, vacuum, etc.) to keep performance optimal.

4. **Bonus** (practical): If your DBMS supports it, run the queries on a test environment with sample data, use EXPLAIN ANALYZE before and after your changes, and record the improvements (provide screenshots or logs).

---

### Summary of Phase 8

| Concept              | Purpose                                                                 |
|----------------------|-------------------------------------------------------------------------|
| **Indexes**              | Speed up searches, sorts, and joins; cost to writes and storage.        |
| **Query Optimization**   | Write efficient SQL: avoid functions on indexed columns, use `EXISTS` wisely, select only needed columns. |
| **Execution Plans**      | Diagnostic tool (`EXPLAIN`) to see the database's strategy and find bottlenecks. |
| **Performance Tuning**   | Holistic approach: monitoring, indexing, rewriting, config tuning, and maintenance. |

---

### Summary: SQL Complete Roadmap

> You have now completed a full journey from SQL basics to advanced performance tuning:


| Phase | Topic                                                                 |
|-------|------------------------------------------------------------------------|
| 1     | Fundamentals (SELECT, INSERT, WHERE, ORDER BY, LIMIT, DISTINCT)       |
| 2     | Filtering & Manipulation (AND/OR, IN, BETWEEN, LIKE, UPDATE, DELETE, TRUNCATE) |
| 3     | Functions (String, Numeric, Date, Aggregates)                          |
| 4     | Grouping & Analysis (GROUP BY, HAVING, Aliases, CASE)                 |
| 5     | Joins (INNER, LEFT, RIGHT, FULL, SELF, CROSS)                         |
| 6     | Advanced Queries (Subqueries, CTEs, Views, Temp Tables)               |
| 7     | Database Design (PK, FK, Constraints, Normalization, Relationships)    |
| 8     | Performance Optimization (Indexes, Query Tuning, Execution Plans)      |

----


<br/><br/><br/>
<center> <b>Happy Learning! 😊</b> </center>