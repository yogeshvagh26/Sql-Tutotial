# SQL Fundamentals

## 1. What is SQL?

SQL (Structured Query Language) is a standard programming language designed for managing and manipulating relational databases. It allows you to create, read, update, and delete data – commonly known as CRUD operations.

**Real-World Example**

Imagine a school’s student records system with thousands of student profiles. The office staff needs to find all students born after 2010, add a new student, or update a phone number. SQL lets them do this quickly using simple commands.

#### SQL Syntax (General)

```sql
-- Basic pattern
    SELECT columns 
    FROM table
    WHERE condition;
```



#### Examples :

#### Retrieve all student names: 

```sql
    SELECT name
    FROM students;
```    

#### Add a new student: 
```sql 
    INSERT INTO students (name, age) VALUES
    ('John', 15);
```

### Summary for 1

>     SQL is the language used to talk to databases. It’s everywhere – from bank systems to e‑commerce websites.

---

## 2. Databases, Tables, Rows, Columns

**Database:** A container that holds related data (like an Excel workbook).

**Table:** A collection of structured data inside a database (like a single sheet in Excel).

**Rows:** Individual records in a table (horizontal entries).

**Columns:** Attributes or fields that define what data is stored (vertical categories).

#### Real-World Example
An online store ***database*** has a Products ***table***. Each row is one product (e.g., “***iPhone 15***”), and columns are ***ProductID, Name, Price, Stock.***

---

#### Visual Table Illustration

#### Table: Employees

| EmployeeID | FirstName | LastName | Department |
|------------|-----------|----------|------------|
| 1          | Alice     | Smith    | Sales      |
| 2          | Bob       | Johnson  | IT         |

> EmployeeID, FirstName, LastName, Department are columns.

> Row 1: (1, Alice, Smith, Sales) – one record.

> The whole grid is the table, inside an Employees database.

---

#### SQL Syntax (No command – just concept)

Use `CREATE DATABASE`, `CREATE TABLE` to define these structures.


### Summary for 2
> **Databases** contain **tables**; tables have **rows (records)** and **columns (fields)**.

---

## 3. Installing and Setting Up a Database

**To practice SQL**, you need a **database management system (DBMS).** Popular free options: **SQLite** (lightweight, file‑based), **MySQL**, **PostgreSQL**, or online sandboxes like SQLiteOnline or DB Fiddle.

#### Real-World Example
A developer working locally installs MySQL, starts the server, and uses a client (like MySQL Workbench or command line) to run SQL commands.

#### Step-by-Step (For SQLite – simplest to start)

> Download SQLite from sqlite.org

    Unzip and place sqlite3.exe (Windows) or use terminal (macOS/Linux)

    Open terminal / command prompt

    Run sqlite3 mydatabase.db to create/open a database

> For MySQL (Windows/macOS)

    Download MySQL Installer or use brew install mysql (macOS)

    Start service: sudo systemctl start mysql (Linux) or net start mysql (Windows)

    Connect: mysql -u root -p

> Online alternative (no install)

    Go to sqliteonline.com – select “SQLite” and start typing.

### Summary for 3

>  You can install MySQL/PostgreSQL for full features or use SQLite / online tools for quick practice.

---

## 4. Creating Databases

**`CREATE DATABASE`** makes a **new empty database**. After creation, you can switch to it and start building tables.

#### Real-World Example
A new project for a library management system: first create a database named LibraryDB.

#### SQL Syntax

```sql 
    CREATE DATABASE database_name;
```
#### Examples

```sql
     CREATE DATABASE SchoolDB;

    CREATE DATABASE IF NOT EXISTS CompanyDB;   -- prevents error if DB exists
```


#### Visual Illustration
**Before:** No databases

**After** : **CREATE DATABASE SchoolDB;** → new container called SchoolDB (empty)

---

#### Practice Questions
> Write a command to create a database named OnlineStore.

> What happens if you run CREATE DATABASE SchoolDB twice?

---

#### Quiz (answers at end)
> Which SQL keyword is used to create a new database?
 1. NEW DB
 2. CREATE DATABASE
 3. MAKE DATABASE

---

### Summary for 4
> CREATE DATABASE initialises a new database container.

---

## 5. Data Types

**Data types** define what kind of value a column can hold **(numbers, text, dates, etc.)**. Choosing the right type saves space and prevents errors.

#### Real-World Example

An **age** column should be an **integer (whole number)**, not text, **so you can calculate average age.**

#### Common SQL Data Types

| Category    | Type            | Description                            |
|-------------|-----------------|----------------------------------------|
| Numeric     | INT, INTEGER    | Whole numbers                          |
| Numeric     | DECIMAL(10,2)   | Exact decimal (e.g., price)            |
| Text        | VARCHAR(n)      | Variable text, max n characters        |
| Text        | TEXT            | Long text                              |
| Date/Time   | DATE            | 'YYYY-MM-DD'                           |
| Boolean     | BOOLEAN         | TRUE / FALSE                           |


#### SQL Syntax (used in CREATE TABLE)

```sql 
    column_name DATATYPE
```

#### Multiple Examples

```sql 
    EmployeeID INT,
    Name VARCHAR(100),
    Salary DECIMAL(8,2),
    HireDate DATE,
    IsActive BOOLEAN
```    
---

### Visual Table Illustration
**Table:** Products with data types

| Column    | Data Type    | Example value |
|-----------|--------------|---------------|
| ProductID | INT          | 101           |
| Name      | VARCHAR(50)  | 'Laptop'      |
| Price     | DECIMAL(6,2) | 899.99        |
| InStock   | BOOLEAN      | TRUE          |

---

#### Practice Questions

> Which data type would you use for a person’s email address?

> Choose a type for a product’s weight with two decimals.

---

#### Quiz
What data type is best for storing '2025-12-31'?
1.  TEXT
2. DATE
3. VARCHAR

---

### Summary for 5
> **Data types** enforce the kind of data stored – match the column to its real‑world meaning.

---

## 6. Creating Tables

**CREATE TABLE** defines a new table inside a database, specifying column names and their data types. Optionally, you can add constraints like PRIMARY KEY.

#### Real-World Example
After creating **UniversityDB**, you need a Students table to hold student information.

#### SQL Syntax
    
```sql

    CREATE TABLE table_name (
        column1 datatype constraint,
        column2 datatype constraint,
        ...
    );
```    

#### Examples

```sql

    CREATE TABLE Students (
        StudentID INT PRIMARY KEY,
        FirstName VARCHAR(50),
        LastName VARCHAR(50),
        EnrollmentDate DATE
    );

    --------------------------------------------------------

    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        Name VARCHAR(100) NOT NULL,
        Price DECIMAL(8,2)
    );

```
---

#### Visual Table Illustration (after creation)

**Table:** Employees (empty, just structure)

| EmpID (INT) | Name (VARCHAR) | Salary (DECIMAL) | Dept (VARCHAR) |
|-------------|----------------|------------------|----------------|

(no rows yet)   

#### Practice Questions
> Create a table Movies with columns: MovieID (INT), Title (VARCHAR(100)), ReleaseYear (INT).

> Add a Rating column with type DECIMAL(3,1).

#### Quiz

**Which SQL command removes a table?**
1. DELETE TABLE
2. DROP TABLE
3. REMOVE TABLE

---

### Summary for 6

> CREATE TABLE builds the skeleton of your data storage.



## 7. INSERT

**INSERT INTO** adds new **rows (records)** to an **existing table**. You can **insert one row or multiple rows at once**.

#### Real-World Example

A new employee joins the company – you add their details to the Employees table.

#### SQL Syntax

```sql 
    INSERT INTO table_name (col1, col2, ...) VALUES (val1, val2, ...);

    INSERT INTO table_name VALUES (val1, val2, ...);  -- order must match all columns

```

#### Examples

```sql 

    -- Insert one row
    INSERT INTO Students (StudentID, FirstName, LastName, EnrollmentDate) 
    VALUES (1, 'Emma', 'Jones', '2025-01-15');

    ------------------------------------

    -- Insert multiple rows
    INSERT INTO Products (ProductID, Name, Price) VALUES
    (101, 'Mouse', 25.99),
    (102, 'Keyboard', 45.50);

    ------------------------------------

    -- Insert without column list (order matters)
    INSERT INTO Employees VALUES (101, 'Alice', 55000, 'Sales');

```
---
#### Visual Table Illustration
**Before** INSERT (Employees table empty)

**After:**
```sql
    INSERT INTO Employees (EmpID, Name, Salary, Dept) 
    VALUES (1, 'John Doe', 60000, 'IT');
```    

| EmpID | Name     | Salary | Dept |
|-------|----------|--------|------|
| 1     | John Doe | 60000  | IT   |

---

#### Practice Questions
> Insert a new movie: The Matrix, release year 1999.

> Insert three students in one command.

#### Quiz
**What happens if you omit column names in INSERT?**
1. Error always
2. Values must match all columns in table order
3. First column is skipped

---

### Summary for 7

> INSERT adds data rows into a table.

---

### 8. SELECT

**SELECT retrieves data from one or more tables.** It’s the most frequently used SQL command.

#### Real-World Example
A manager wants to see all employees and their salaries – 
```sql 
    SELECT *
    FROM Employees;
```    

#### SQL Syntax

```sql 

    SELECT
        column1,
        column2
    FROM table_name;

    ------------------------------------

    SELECT *
    FROM table_name;   -- all columns

```


#### Examples

```sql

    -- All columns
    SELECT * FROM Students;

    ------------------------------------

    -- Specific columns
    SELECT 
        FirstName, 
        LastName 
    FROM Students;

    ------------------------------------

    -- With calculation
    SELECT 
        Name, 
        Price, 
        Price * 1.1 AS PriceWithTax 
    FROM Products;

```
---

#### Visual Table Illustration

#### Table: Employees (data)


| EmpID | Name  | Dept  | Salary |
|-------|-------|-------|--------|
| 1     | Alice | Sales | 50000  |
| 2     | Bob   | IT    | 70000  |

---

```sql
    SELECT 
        Name, 
        Salary 
    FROM Employees; produces:
```

| Name  | Salary |
|-------|--------|
| Alice | 50000  |
| Bob   | 70000  |

---

#### Practice Questions
> Write a query to get ProductID and Price from Products.

> How would you see all columns of the Movies table?

--- 

#### Quiz
**Which symbol means “all columns”?**
1. ALL
2. %
3. `*`

---

### Summary for 8
**SELECT reads** and **displays** data from tables.

---

## 9. WHERE

**WHERE filters rows based on a condition.** Only rows that satisfy the condition are returned.

#### Real-World Example

> Find all employees who earn more than $50,000 – WHERE Salary > 50000.

#### SQL Syntax

```sql
    SELECT columns 
    FROM table 
    WHERE condition;
```    
#### Comparison Operators

    =, <> or !=, >, <, >=, <=, BETWEEN, LIKE, IN, AND, OR, NOT.

#### Examples    

```sql 

    -- Exact match
    SELECT * 
    FROM Students 
    WHERE StudentID = 5;

    ------------------------------------

    -- Greater than
    SELECT 
        Name, 
        Salary 
    FROM Employees 
    WHERE Salary > 60000;

    ------------------------------------

    -- Text condition
    SELECT * 
    FROM Products 
    WHERE Name = 'Laptop';

    ------------------------------------

    -- Pattern matching
    SELECT * 
    FROM Customers 
    WHERE City LIKE 'New%';  -- starts with New

    ------------------------------------

    -- Multiple conditions
    SELECT * 
    FROM Orders 
    WHERE Total > 100 AND Status = 'Shipped';        

```

#### Visual Table Illustration
**Table: Products**

| ProductID | Name     | Price |
|-----------|----------|-------|
| 1         | Mouse    | 25    |
| 2         | Keyboard | 45    |
| 3         | Monitor  | 150   |

**Query**: 

```sql
    SELECT * 
    FROM Products 
    WHERE Price > 40; 
```
> returns rows 2 and 3.

---

#### Practice Questions
> List all employees with department = 'Sales'.

> Find movies released after 2010.

> Get products with price between 20 and 100.

---

#### Quiz
**Which query selects students named 'John'?**
1. SELECT * WHERE Name = 'John'
2. SELECT * FROM Students WHERE Name = 'John'
3. SELECT Students WHERE Name IS John

---

### Summary for 9

> WHERE filters data based on conditions.

---

## 10. ORDER BY

**ORDER BY sorts the result set by one or more columns, in ascending (ASC) or descending (DESC) order.** 

>Ascending is default.

#### Real-World Example

Show products from cheapest to most expensive – ORDER BY Price ASC.

#### SQL Syntax

```sql 
    SELECT columns 
    FROM table 
    ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
``` 

#### Examples

```sql 

    -- Sort by name A→Z
    SELECT * 
    FROM Products 
    ORDER BY Name;

    ------------------------------------

    -- Highest salary first
    SELECT 
        Name, 
        Salary 
    FROM Employees 
    ORDER BY Salary DESC;

    ------------------------------------

    -- Sort by department, then by name
    SELECT * 
    FROM Employees 
    ORDER BY Dept ASC, Name ASC;

```
--- 

#### Visual Table Illustration
**Unsorted Employees**

| Name  | Salary |
|-------|--------|
| Alice | 50000  |
| Bob   | 70000  |


```sql 
    SELECT 
        Name, 
        Salary 
    FROM Employees 
    ORDER BY Salary DESC;
```

| Name  | Salary |
|-------|--------|
| Bob   | 70000  |
| Alice | 50000  |

---

### Practice Questions
> Write a query to list students by last name alphabetically.

> Show products ordered by price descending, then by name ascending.

#### Quiz
**ORDER BY default order is:**
1. DESC
2. ASC
3. No default

---

### Summary for 10
> ORDER BY sorts query results.


---

## 11. LIMIT

**LIMIT restricts the number of rows returned by a query.** Very useful for large tables or to show top N records.

#### Real-World Example
Show the three most expensive products – combine **ORDER BY Price DESC and LIMIT 3**.

#### SQL Syntax

```sql 

    SELECT columns 
    FROM table 
    ORDER BY column 
    LIMIT count;

    -- Some DBMS use `TOP` or `FETCH FIRST`, but LIMIT is standard in MySQL, SQLite, PostgreSQL

```

#### Examples 

```sql 

    -- First 5 rows
    SELECT * 
    FROM Employees 
    LIMIT 5;

    ------------------------------------

    -- Cheapest 3 products
    SELECT * 
    FROM Products 
    ORDER BY Price ASC 
    LIMIT 3;

    ------------------------------------
    
    -- With offset (skip first 2, then show 4)
    SELECT * 
    FROM Orders 
    LIMIT 4 OFFSET 2;

```
---

#### Visual Table Illustration
**Table: Students**

| ID | Name  |
|----|-------|
| 1  | Anna  |
| 2  | Ben   |
| 3  | Carla |
| 4  | David |

```sql
    SELECT * 
    FROM Students 
    LIMIT 2; 
```
> returns only Anna and Ben.

---

### Practice Questions
> Get the top 10 highest paid employees.

> Show 5 products after skipping the first 3 (pagination).

#### Quiz
**LIMIT 10 returns:**
1. 10 random rows
2. First 10 rows
3. Last 10 rows

---

### Summary for 11
> LIMIT caps the number of rows in the result.

---

## 12. DISTINCT

**DISTINCT removes duplicate rows from the result, keeping only unique values for the selected column(s).**

#### Real-World Example
Find all unique departments in the company – 

```sql 
    SELECT DISTINCT Dept 
    FROM Employees;
```    

#### SQL Syntax

```sql 
    SELECT DISTINCT column1, 
        column2 
    FROM table;
```

#### Examples

```sql 

    -- Unique cities
    SELECT DISTINCT City 
    FROM Customers;

    ------------------------------------

    -- Unique combinations of dept and job title
    SELECT DISTINCT Dept, 
        JobTitle 
    FROM Employees;

    ------------------------------------

    -- Count unique values
    SELECT COUNT(DISTINCT Category) 
    FROM Products;

```

#### Visual Table Illustration
**Table: Orders (CustomerID column)**

| OrderID | CustomerID |
|---------|------------|
| 101     | 5          |
| 102     | 3          |
| 103     | 5          |
| 104     | 2          |

```sql
    SELECT DISTINCT CustomerID 
    FROM Orders; 
```
> returns unique IDs: 5, 3, 2.

---

### Practice Questions
> List all distinct product categories from Products table.

> Write a query to see unique (City, State) pairs.

#### Quiz
**SELECT DISTINCT Name, Age from a table gives:**
1. Unique names only
2. Unique combinations of name and age
3.  Distinct ages only

---

### Summary for 12
> DISTINCT eliminates duplicate rows.

---

## Practice Questions (Comprehensive)

> Create a database named SalesDB.

> In SalesDB, create a table Orders with:
OrderID (INT PRIMARY KEY), CustomerName (VARCHAR), Amount (DECIMAL), OrderDate (DATE).

> Insert three orders: (1, 'Alice', 250.00, '2025-01-10'), (2, 'Bob', 99.99, '2025-01-11'), (3, 'Alice', 150.00, '2025-01-12').

> Write a query to select all orders where Amount > 100.

> Retrieve all orders sorted by OrderDate descending.

> Get the top 2 highest Amount orders.

> List all distinct customer names.

### Answers (for self-check):

```sql 
    -- 1-3
    CREATE DATABASE SalesDB;

    ------------------------------------

    USE SalesDB;  -- or \c SalesDB in psql
    
    ------------------------------------
    
    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY, 
        CustomerName VARCHAR(50), 
        Amount DECIMAL(8,2), 
        OrderDate DATE
    );

    ------------------------------------

    INSERT INTO Orders VALUES 
    (1, 'Alice', 250.00, '2025-01-10'), 
    (2, 'Bob', 99.99, '2025-01-11'), 
    (3, 'Alice', 150.00, '2025-01-12');

    ------------------------------------
    
    -- 4
    SELECT * 
    FROM Orders 
    WHERE Amount > 100;
    
    ------------------------------------

    -- 5
    SELECT * 
    FROM Orders 
    ORDER BY OrderDate DESC;
    
    ------------------------------------

    -- 6
    SELECT * 
    FROM Orders 
    ORDER BY Amount DESC 
    LIMIT 2;

    ------------------------------------

    -- 7
    SELECT DISTINCT CustomerName 
    FROM Orders;

```

---

#### Quiz (Answers)
> CREATE DATABASE

> DATE

> DROP TABLE

> Values must match all columns in table order

> `*`

> SELECT * FROM Students WHERE Name = 'John'

> ASC

> First 10 rows

> Unique combinations of name and age


---

### Interview Questions

Q: What is the difference between **DELETE** and **DROP**?

A: **DELETE removes rows** (can be rolled back), **DROP removes the entire table or database structure.**

--- 

**Q: Can you use WHERE with INSERT? Why?**

A: **No.** **INSERT does not filter existing rows;** it adds new ones. Conditions belong in UPDATE or DELETE.

--- 

**Q: How do you select the top 5 highest salaries, including ties?**

A: In MySQL/SQLite: 

```sql
    SELECT * 
    FROM Employees 
    ORDER BY Salary DESC LIMIT 5;
```


For ties, you might need window functions like DENSE_RANK().

--- 

**Q: What is the effect of SELECT DISTINCT `*`?**

A: It returns rows that are completely unique across all columns.

--- 

**Q: Why specify data types for columns?**

A: For data integrity, storage efficiency, and correct sorting/comparisons.


---

### Assignment
**Scenario:** You are building a small library management system.

### Tasks:

> Create a database named Library.

> Create a table Books with columns:

* BookID INT PRIMARY KEY

* Title VARCHAR(200) NOT NULL

* Author VARCHAR(100)

* YearPublished INT

* Genre VARCHAR(50)

* Available BOOLEAN

> Insert at least 5 realistic book records.

> Write queries to:
1. Show all books.
2. Show only Title and Author of books published after 2010.
3. List distinct genres.
4. Show books sorted by YearPublished descending.
5. Display the 3 most recent books.

> (Bonus) Insert another book, then display all books where Genre is 'Fiction' and Available is TRUE, ordered by Title.

---

### Summary of Phase 1

| Concept         | Purpose                                           |
|-----------------|---------------------------------------------------|
| **SQL**             | Language to communicate with databases            |
| **Database**        | Container of tables                               |
| **Table, Row, Column** | Structure: grid with records and fields         |
| **CREATE DATABASE** | Make a new database                               |
| **Data Types**      | Define column contents (INT, VARCHAR, DATE, ...)  |
| **CREATE TABLE**    | Define table schema                               |
| **INSERT**          | Add rows                                          |
| **SELECT**          | Retrieve data                                     |
| **WHERE**           | Filter rows                                       |
| **ORDER BY**        | Sort results                                      |
| **LIMIT**           | Restrict number of rows                           |
| **DISTINCT**        | Remove duplicates                                 |


---

> **Happy Learning! :)**



