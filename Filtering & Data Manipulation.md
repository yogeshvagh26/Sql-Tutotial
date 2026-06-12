# Phase 2: **Filtering & Data Manipulation**
* AND, OR, NOT
* IN
* BETWEEN
* LIKE
* Wildcards
* UPDATE
* DELETE
* TRUNCATE
---
## **1. AND, OR, NOT**

**AND:** Combines two conditions – both must be true.

**OR:** Either condition can be true.

**NOT:** Negates a condition (reverses true/false).

> These logical operators allow complex filters in WHERE clauses.

#### Real-World Example

    A store manager wants:

    Products with price > $50 AND stock < 10 (low stock expensive items).

    Customers from 'USA' OR 'Canada' (either country).

    Employees NOT in the 'Sales' department.

--- 
#### SQL Syntax

```sql 

    SELECT column1, column2 
    FROM table
    WHERE condition1 AND condition2;

    ------------------------------------------------------

    SELECT * 
    FROM table
    WHERE condition1 OR condition2;

    ------------------------------------------------------

    SELECT * 
    FROM table
    WHERE NOT condition;
```


#### Examples

```sql 

    -- AND: Both must be true
    SELECT * 
    FROM Products 
    WHERE Price > 100 AND Stock < 20;

    ------------------------------------------------------

    -- OR: At least one true
    SELECT * 
    FROM Customers 
    WHERE Country = 'USA' OR Country = 'Canada';

    ------------------------------------------------------

    -- NOT: Opposite
    SELECT * 
    FROM Employees 
    WHERE NOT Department = 'Sales';

    ------------------------------------------------------

    -- Combining all three
    SELECT * 
    FROM Orders 
    WHERE (Status = 'Shipped' OR Status = 'Delivered') 
    AND NOT TotalAmount < 50;

```  
--- 

### Visual Table Illustration

**Table: Products**

| ProductID | Name     | Price | Stock |
|-----------|----------|-------|-------|
| 1         | Laptop   | 800   | 5     |
| 2         | Mouse    | 25    | 50    |
| 3         | Keyboard | 60    | 8     |
| 4         | Monitor  | 200   | 3     |

#### Query: 

```sql
    SELECT * 
    FROM Products 
    WHERE Price > 50 AND Stock < 10;
```    
#### Result:

| ProductID | Name     | Price | Stock |
|-----------|----------|-------|-------|
| 3         | Keyboard | 60    | 8     |
| 4         | Monitor  | 200   | 3     |

---
### Practice Questions
> Write a query to find employees with salary > 50000 AND department = 'IT'.

> Select orders where status = 'Pending' OR status = 'Processing'.

> Get products NOT in category 'Electronics'.

### Quiz
**Which operator returns true only if both conditions are true?**
1. OR
2. NOT
3. AND

---

### Summary for 1
**AND, OR, NOT** combine or negate conditions, enabling precise data filtering.

---
 
## **2. IN**

**IN checks if a value matches any value in a list or subquery.** It’s a shorter, cleaner alternative to multiple OR conditions.

#### Real-World Example
Find all employees **who work in** the **'Sales', 'Marketing', or 'IT' departments** – 

**instead of writing** 
> Dept = 'Sales' OR Dept = 'Marketing' OR Dept = 'IT', 

**Use** 

> Dept IN ('Sales','Marketing','IT').
---

#### SQL Syntax

```sql
    SELECT columns 
    FROM table
    WHERE column IN (value1, value2, ...);
```

#### Examples

```sql 

    -- With a list of values
    SELECT * 
    FROM Students 
    WHERE Grade IN ('A', 'B', 'C');

    ------------------------------------------------------

    -- Using NOT IN
    SELECT * 
    FROM Products 
    WHERE Category NOT IN ('Discontinued', 'Clearance');

    ------------------------------------------------------

    -- With subquery (advanced)
    SELECT * 
    FROM Orders 
    WHERE CustomerID IN (SELECT CustomerID 
                        FROM Customers 
                        WHERE Loyalty = 'Gold'
                        );

```

---

### Visual Table Illustration

#### Table: Employees

| EmpID | Name    | Dept      |
|-------|---------|-----------|
| 1     | Alice   | Sales     |
| 2     | Bob     | IT        |
| 3     | Charlie | Marketing |
| 4     | David   | HR        |

#### Query: 

```sql
    SELECT * 
    FROM Employees 
    WHERE Dept IN ('Sales', 'IT');
```

#### Result:

| EmpID | Name  | Dept  |
|-------|-------|-------|
| 1     | Alice | Sales |
| 2     | Bob   | IT    |

---

### Practice Questions
> Select all products with IDs (101, 105, 110).

> Find customers from 'NY', 'CA', or 'TX' using IN.

> Write a query using NOT IN to exclude a list of discontinued product IDs.

### Quiz
**IN (1,2,3) is equivalent to:**
1.  BETWEEN 1 AND 3
2.  =1 OR =2 OR =3
3.  AND 1,2,3

---

### Summary for 2
> IN tests membership in a set, making queries more readable.

---

## **3. BETWEEN**

**BETWEEN** selects values within a given range (inclusive of endpoints). **Works with numbers, dates, and text.**

#### Real-World Example
**Find employees** hired between Jan 1, 2020 and Dec 31, 2022 – HireDate BETWEEN '2020-01-01' AND '2022-12-31'.

#### SQL Syntax

```sql 
    SELECT columns 
    FROM table
    WHERE column BETWEEN lower_value AND upper_value;
```

#### Examples

```sql 

    -- Numeric range
    SELECT * 
    FROM Products 
    WHERE Price BETWEEN 50 AND 100;

    ------------------------------------------------------

    -- Date range
    SELECT * 
    FROM Orders 
    WHERE OrderDate BETWEEN '2025-01-01' AND '2025-01-31';

    ------------------------------------------------------

    -- Text range (alphabetical)
    SELECT * 
    FROM Employees 
    WHERE LastName BETWEEN 'A' AND 'M';

    ------------------------------------------------------

    -- NOT BETWEEN
    SELECT * 
    FROM Students 
    WHERE Age NOT BETWEEN 18 AND 22;

```
---

### Visual Table Illustration

#### Table: Products (Price column)

| ProductID | Name     | Price |
|-----------|----------|-------|
| 1         | Mouse    | 25    |
| 2         | Keyboard | 60    |
| 3         | Monitor  | 150   |
| 4         | Cable    | 10    |

#### Query: 

```sql
    SELECT * 
    FROM Products 
    WHERE Price BETWEEN 20 AND 100;
```

#### Result: 
> Mouse (25), Keyboard (60) – Cable (10) is below, Monitor (150) above.

### Practice Questions
> Get employees with salary between 40000 and 60000.

> Find orders placed in Q1 of 2025 (Jan 1 to Mar 31).

> Select products with price NOT between $5 and $20.

### Quiz
**Is BETWEEN inclusive of endpoints?**
1. Yes
2. No
3. Depends on database

---

### Summary for 3

> BETWEEN is a clean way to filter ranges, inclusive of both ends.

---

## **4. LIKE**

**LIKE performs pattern matching on text strings using two wildcard characters:**

**% (percent)** – matches zero or more characters

**_ (underscore)** – matches exactly one character

#### Real-World Example
Search for customers whose 

> names start with ‘J’ – Name LIKE 'J%'. 

OR 

> find emails ending with ‘@gmail.com’ – Email LIKE '%@gmail.com'.

#### SQL Syntax

```sql 
    SELECT columns 
    FROM table
    WHERE column LIKE pattern; 
```    

#### Examples

```sql
    -- Starts with 'A'
    SELECT * 
    FROM Employees 
    WHERE Name LIKE 'A%';

    ------------------------------------------------------

    -- Ends with 'son'
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '%son';

    ------------------------------------------------------

    -- Contains 'ar' anywhere
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '%ar%';

    ------------------------------------------------------

    -- Second letter is 'a' (e.g., 'Da', 'Ma')
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '_a%';

    ------------------------------------------------------

    -- Exactly 5 letters
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '_____';

    ------------------------------------------------------

    -- Starts with 'J' and ends with 'n'
    SELECT * 
    FROM Employees 
    WHERE Name LIKE 'J%n';

```
---

### Visual Table Illustration

#### Table: Employees

| Name    |
|---------|
| Alice   |
| Bob     |
| Charlie |
| David   |
| Aaron   |

#### Query: 
```sql 
    SELECT * 
    FROM Employees 
    WHERE Name LIKE 'A%';
```    
#### Result: 

> Alice, Aaron

#### Query: 
```sql
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '_a%';
```    
#### Result Explain:
> David (second letter 'a') – also Charlie? No, Charlie starts with C, second letter 'h', not 'a'.


---

### Practice Questions
> Find all products whose name contains 'Pro'.

> Get customers with phone numbers starting with '555'.

> Find employees whose email ends with '@company.com'.

### Quiz
*Which wildcard matches exactly one character?*
1. %
2. `*`
3. _

---
### Summary for 4
> LIKE with % and _ enables flexible text pattern searches.

---
## **5. Wildcards**

**Wildcards are special characters used with LIKE (and sometimes other operators) to represent unknown characters.** The two primary wildcards in SQL are:

> % – any number of characters (including zero)

>_ – exactly one character

Some databases have additional wildcards 
> (e.g., [a-f] range in SQL Server, or ESCAPE for custom escape).

#### Real-World Example
    An HR system needs to find employees with second name containing 'an' – WHERE Name LIKE '_%an%'. 
    OR 
    find product codes of pattern 'A_ _' where underscores represent digits.

#### Syntax (Same as LIKE)

```sql 
    SELECT ... 
    WHERE column LIKE 'pattern_with_%_and_underscores';
```    
#### Examples

```sql
    -- Starts with any single character, then 'at' (e.g., 'cat', 'bat')
    SELECT * 
    FROM Words 
    WHERE Word LIKE '_at';

    ------------------------------------------------------

    -- Contains at least one 'e'
    SELECT * 
    FROM Names 
    WHERE Name LIKE '%e%';

    ------------------------------------------------------

    -- Exactly 3 characters long
    SELECT * 
    FROM Codes 
    WHERE Code LIKE '___';

    ------------------------------------------------------

    -- Starts with 'M' and ends with 'e', any length in between
    SELECT * 
    FROM Employees 
    WHERE Name LIKE 'M%e';

    ------------------------------------------------------

    -- Escape special characters (e.g., find strings with %)
    SELECT * 
    FROM Logs 
    WHERE Message LIKE '%\%%' ESCAPE '\';

```
---
### Visual Table Illustration

#### Table: Products (ProductCode)

| Code  | Name     |
|-------|----------|
| AB123 | Laptop   |
| A12B  | Mouse    |
| B1C3  | Keyboard |
| AB    | Cable    |

#### Query: 
```sql
    SELECT * 
    FROM Products 
    WHERE Code LIKE 'A%';
```    

**Result:** AB123, A12B (both start with A)

#### Query: 
```sql
    SELECT * 
    FROM Products 
    WHERE Code LIKE '__%'
```
 (at least 2 chars) – all except none? Actually AB (2 chars) is included.

> Better example: LIKE '___' (exactly 3) – none in this set.


---

### Practice Questions
> Write a pattern to find strings that have exactly 5 characters, starting with 'X' and ending with 'Y'.

> Find all email addresses with 'gmail' in the domain (e.g., %@gmail.com).

> (Advanced) Find product codes containing underscore (_) as a literal character.

### Quiz
**What does the pattern '%a%' match?**
1. Strings starting with a
2. Strings ending with a
3. Strings containing at least one 'a'

---

### Summary for 5
> Wildcards % and _ give pattern-matching superpowers to LIKE. Remember to escape literal wildcards when needed.

---

## **6. UPDATE**

**UPDATE changes existing data in one or more rows.** Use WHERE to specify which rows; without WHERE, all rows are updated – be careful!

#### Real-World Example
A product price increases from $20 to $25. 

```sql
    UPDATE Products 
    SET Price = 25 
    WHERE ProductID = 101;
```    

#### Syntax

```sql 
    UPDATE table_name
    SET column1 = value1, column2 = value2, ...
    WHERE condition;
```    

#### Examples

```sql 

    -- Update a single column for one row
    UPDATE Employees 
    SET Salary = 65000 
    WHERE EmpID = 5;

    ------------------------------------------------------

    -- Update multiple columns
    UPDATE Products 
    SET Price = 19.99, Stock = 100 
    WHERE ProductID = 200;

    ------------------------------------------------------

    -- Update all rows (no WHERE) – use with extreme caution
    UPDATE Employees 
    SET Active = 0;  -- marks everyone inactive

    ------------------------------------------------------

    -- Update using expression
    UPDATE Orders 
    SET Total = Total * 1.1 
    WHERE OrderDate < '2025-01-01';

``` 
---

### Visual Table Illustration

#### Before UPDATE:

| EmpID | Name  | Salary |
|-------|-------|--------|
| 1     | Alice | 50000  |
| 2     | Bob   | 60000  |

#### UPDATE 

```sql
    UPDATE Employees 
    SET Salary = 55000 
    WHERE EmpID = 1;
```    

#### After UPDATE:

| EmpID | Name  | Salary |
|-------|-------|--------|
| 1     | Alice | 55000  |
| 2     | Bob   | 60000  |

---

### Practice Questions
> Increase the price of all products in category 'Electronics' by 10%.

> Change the department of employee with ID 15 to 'Marketing'.

> Set the status of orders older than 2024-01-01 to 'Archived'.
---
### Quiz
**What happens if you omit WHERE in an UPDATE statement?**
1. Syntax error
2. Only first row is updated
3. All rows are updated

--- 
### Summary for 6
> UPDATE modifies existing rows; always double‑check WHERE to avoid updating everything.

---

## 7. DELETE

**DELETE removes rows from a table.** Like UPDATE, you typically use WHERE to target specific rows. Without WHERE, all rows are deleted **(but table structure remains).**

#### Real-World Example
A customer asks to delete their account – DELETE FROM Customers WHERE CustomerID = 99;.


#### SQL Syntax

```sql
    DELETE FROM table_name 
    WHERE condition;
```    

#### Examples

```sql

    -- Delete a specific row
    DELETE FROM Employees 
    WHERE EmpID = 7;

    ------------------------------------------------------

    -- Delete rows meeting a condition
    DELETE FROM Orders 
    WHERE Status = 'Cancelled';

    ------------------------------------------------------

    -- Delete all rows from a table (structure stays)
    DELETE FROM Logs;   -- or DELETE FROM Logs WHERE 1=1;

    ------------------------------------------------------

    -- Delete based on another table (using subquery)
    DELETE FROM Products 
    WHERE CategoryID IN 
                (SELECT CategoryID 
                FROM Categories 
                WHERE IsDiscontinued = 1);

  ```
 
### Visual Table Illustration

#### Before DELETE:

| EmpID | Name  |
|-------|-------|
| 1     | Alice |
| 2     | Bob   |
| 3     | Carol |

```sql
    DELETE FROM Employees 
    WHERE Name = 'Bob';
```    

#### After DELETE:

| EmpID | Name  |
|-------|-------|
| 1     | Alice |
| 3     | Carol |

---

### Practice Questions
> Delete all products with stock = 0.

> Remove employees who left the company before 2020.

> Delete orders that are more than 5 years old.

### Quiz
*Which command removes rows but keeps the table structure?*
1. DROP TABLE
2. DELETE
3. TRUNCATE (also keeps structure, but we cover next)

---

### Summary for 7
> DELETE removes rows; always use a WHERE clause unless you really mean to empty the table.

---

## **8. TRUNCATE**

**TRUNCATE quickly removes all rows from a table, resetting storage and often auto-increment counters.**
>  It is faster than DELETE because it doesn’t log individual row deletions. However, you cannot use a WHERE clause – it’s all or nothing.

#### Real-World Example

A temporary staging table used for daily imports needs to be cleared before loading new data. 
```sql
TRUNCATE TABLE Staging;
```
 is much faster than 
 ```sql
 DELETE FROM Staging;
```
 .

#### SQL Syntax

```sql
    TRUNCATE TABLE table_name;
```    

#### Examples

```sql 

    -- Empty a table completely
    TRUNCATE TABLE TempOrders;

    ------------------------------------------------------

    -- Truncate multiple tables (some DBMS require separate statements)
    TRUNCATE TABLE Logs;
    TRUNCATE TABLE SessionData;

``` 
---
### Visual Table Illustration


#### Table: TempData (1000 rows)

**After** 
```sql
    TRUNCATE TABLE TempData;
```    
table still exists but has zero rows. The next inserted row may start with a reset identity value (depending on DB).

---

### Key Differences: DELETE vs TRUNCATE

| Feature                   | DELETE                           | TRUNCATE                                    |
|---------------------------|----------------------------------|---------------------------------------------|
| WHERE clause              | Yes                              | No                                          |
| Logs individual rows      | Yes                              | No (minimal logging)                        |
| Speed                     | Slower for large tables          | Very fast                                   |
| Triggers                  | Fire for each row                | Do not fire                                 |
| Identity reset            | Usually no                       | Yes (in many DBs)                           |
| Can be rolled back*       | Yes (in transaction)             | Yes (in transaction in some DBs, e.g., PostgreSQL) |

#### *In a transaction, both can be rolled back in most databases, but TRUNCATE is often a DDL command so behaviour varies.

---

### Practice Questions
> When would you choose TRUNCATE over DELETE?

> Write a command to remove all rows from a table named Archive.

> Can you truncate a table that has foreign key references? (Answer: generally no, unless cascading or disabling constraints.)

### Quiz
**Which statement is true about TRUNCATE?**
1. It can delete specific rows
2. It is faster than DELETE for removing all rows
3. It cannot be rolled back in any database

---

### Summary for 8
> TRUNCATE instantly empties a table; use it when you need to remove all rows without logging each deletion.

---

## Practice Questions (Comprehensive)
**Given a table Employees:**

| EmpID | Name  | Dept  | Salary | HireDate   |
|-------|-------|-------|--------|------------|
| 1     | John  | Sales | 55000  | 2020-06-15 |
| 2     | Mary  | IT    | 72000  | 2019-11-01 |
| 3     | Steve | Sales | 48000  | 2021-03-20 |
| 4     | Linda | HR    | 62000  | 2018-07-10 |
| 5     | James | IT    | 81000  | 2022-01-05 |

### Write queries for:

> Select employees in Sales or IT with salary >= 60000.

> Find employees whose name starts with 'J' and ends with any characters.

> Increase salary of all IT employees by 5%.

> Delete employees hired before 2020.

> Select employees whose salary is between 50000 and 70000.

> Using IN, get employees from 'Sales' and 'HR' departments.

> Select employees whose name contains 'a' as the second letter (e.g., 'James'? J a mes – yes).

> Empty the entire Employees table but keep its structure. Which command?

### Answers (for self-check):

```sql

    -- 1
    SELECT * 
    FROM Employees 
    WHERE (Dept = 'Sales' OR Dept = 'IT') AND Salary >= 60000;

    ------------------------------------------------------

    -- 2
    SELECT * 
    FROM Employees 
    WHERE Name LIKE 'J%';

    ------------------------------------------------------

    -- 3
    UPDATE Employees 
    SET Salary = Salary * 1.05 
    WHERE Dept = 'IT';

    ------------------------------------------------------

    -- 4
    DELETE 
    FROM Employees 
    WHERE HireDate < '2020-01-01';

    ------------------------------------------------------

    -- 5
    SELECT * 
    FROM Employees 
    WHERE Salary BETWEEN 50000 AND 70000;

    ------------------------------------------------------

    -- 6
    SELECT * 
    FROM Employees 
    WHERE Dept IN ('Sales', 'HR');

    ------------------------------------------------------

    -- 7
    SELECT * 
    FROM Employees 
    WHERE Name LIKE '_a%';  -- James has 'a' as second char (J a)

    ------------------------------------------------------

    -- 8
    TRUNCATE TABLE Employees;

```

### Quiz (Answers)

> AND

> =1 OR =2 OR =3

> Yes

> _

> Strings containing at least one 'a'

> All rows are updated

> DELETE (also TRUNCATE, but DELETE is more general answer)

> It is faster than DELETE for removing all rows
---
### Interview Questions

**Q: What is the difference between IN and BETWEEN?**

**A:** IN checks membership in a discrete list; BETWEEN checks if a value falls within a continuous range (inclusive).

---

**Q: Can you use LIKE with numbers?**

**A:** Yes, but numbers will be implicitly converted to strings. Better to use comparison operators unless you need pattern matching (e.g., phone numbers with dashes).

---

**Q: How would you update a column to be the result of a calculation from another table?**

**A:** Use a **JOIN** in the **UPDATE statement** (depending on DBMS). Example:
```sql
    UPDATE Orders 
    SET Orders.Total = (SELECT SUM(Amount) 
                        FROM OrderItems 
                        WHERE OrderItems.OrderID = Orders.OrderID);    
```
---
**Q: What happens if you run DELETE FROM table without a transaction? Can it be undone?**

**A:** In most databases, it’s immediately committed. Some (like PostgreSQL) can roll back if you wrap in BEGIN; DELETE ...; ROLLBACK;. Always back up or test first.

---

**Q: Why is TRUNCATE sometimes not allowed on tables with foreign keys?**

**A:** Because TRUNCATE resets the table and could orphan child rows. Use DELETE with a WHERE condition or cascade constraints.

---


# **Assignment**

### Scenario: 
> You manage a Library database with tables: Books, Members, and Loans.

#### Tasks:

1. Create the database and tables (or reuse from Phase 1 if available):



    ```sql 

    CREATE DATABASE Library;
    
    ------------------------------------------------------

    USE Library;   -- or \c Library

    ------------------------------------------------------

    CREATE TABLE Books (
        BookID INT PRIMARY KEY,
        Title VARCHAR(200),
        Author VARCHAR(100),
        Genre VARCHAR(50),
        YearPublished INT,
        Available BOOLEAN
    );

    ------------------------------------------------------

    CREATE TABLE Members (
        MemberID INT PRIMARY KEY,
        FullName VARCHAR(100),
        JoinDate DATE,
        Active BOOLEAN
    );

    ------------------------------------------------------

    CREATE TABLE Loans (
        LoanID INT PRIMARY KEY,
        BookID INT,
        MemberID INT,
        LoanDate DATE,
        ReturnDate DATE
    );

    ```
    
2. Insert sample data (at least 5 books, 3 members, 6 loans with some returns pending).


3. Write and execute the following queries (save them as a script):

    > List all books that are currently available (Available = TRUE) AND published after 2010.

    > Find members who joined in 2024 OR 2025 (use IN or BETWEEN).

    > Select books whose title contains 'SQL' or 'Database'.

    > Update the ReturnDate for a specific loan where BookID = 2 to today's date.

    > Delete any loans where the loan date is older than 3 years and the book has been returned (ReturnDate IS NOT NULL).

    > Use TRUNCATE to empty a temporary table (create one called TempAudit first).

    > Finally, show all books with a LIKE pattern to find authors whose last name starts with 'S' (e.g., 'Smith').

4. **Bonus:** Write a query using AND/OR and NOT to find books that are not in the 'Fiction' genre but are available, and published between 2000 and 2020.

---

## **Summary of Phase 2**

| Concept          | Purpose                                                       |
|------------------|---------------------------------------------------------------|
| AND / OR / NOT   | Combine or negate conditions for flexible filters             |
| IN               | Match any value in a list                                     |
| BETWEEN          | Filter within a range (inclusive)                             |
| LIKE             | Pattern matching with wildcards % and _                       |
| Wildcards        | % (any chars) and _ (one char)                                |
| UPDATE           | Modify existing rows                                          |
| DELETE           | Remove specific rows                                          |
| TRUNCATE         | Quickly delete all rows, resetting the table                  |

---


**Happy Learning! 😊**
 




