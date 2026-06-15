# Phase 7: Database Design

### Overview

**Database design is the blueprint for organizing data efficiently and ensuring data integrity.** This phase covers the fundamental building blocks:

* **Primary Keys** – uniquely identify each row in a table.

* **Foreign Keys** – link tables together.

* **Constraints** – enforce rules on data.

* **Normalization** – process to reduce redundancy and dependency.

* **Relationships** – how tables connect (one‑to‑one, one‑to‑many, many‑to‑many).

> Proper design prevents anomalies, saves storage, and makes queries faster and more reliable.


---

## 1. Primary Keys

A **Primary Key** is a column (or a set of columns) that uniquely identifies each row in a table. Every table should have a primary key.

**Characteristics :**

* **Unique** – no two rows can have the same primary key value.

* **NOT NULL** – every row must have a value.

* **Immutable** – ideally, primary key values never change.

* Only one primary key per table (but it can consist of multiple columns – composite primary key).


#### Real-World Example

In a **Students** table, **StudentID** is the natural primary key because each student has a unique ID. You can guarantee that searching for **StudentID = 12345** returns exactly one row.

---

#### SQL Syntax

```sql 

    -- Single column primary key
    CREATE TABLE Students (
        StudentID INT PRIMARY KEY,
        Name VARCHAR(100)
    );

    -------------------------------------------------------

    -- Composite primary key (multiple columns)
    CREATE TABLE Enrollments (
        StudentID INT,
        CourseID INT,
        EnrollmentDate DATE,
        PRIMARY KEY (StudentID, CourseID)
    );
    
    -------------------------------------------------------

    -- Adding primary key after table creation
    ALTER TABLE Products 
    ADD PRIMARY KEY (ProductID);

```

#### Examples

**Example 1: Auto‑increment primary key (MySQL, SQLite)**

```sql

    CREATE TABLE Orders (
        OrderID INT AUTO_INCREMENT PRIMARY KEY,
        OrderDate DATE
    );

```

**Example 2: Using a natural key (e.g., email)**

```sql
    CREATE TABLE Users (
        Email VARCHAR(100) PRIMARY KEY,
        Name VARCHAR(100)
    );
```

**Example 3: Composite key**

```sql
    CREATE TABLE TeamMembers (
        TeamID INT,
        MemberID INT,
        Role VARCHAR(50),
        PRIMARY KEY (TeamID, MemberID)
    );
```

---
### Visual Table Illustration

#### Table: Books (Primary Key: ISBN)

| ISBN               | Title              | Author |
|--------------------|--------------------|--------|
| 978-3-16-148410-0  | SQL Fundamentals   | Smith  |
| 978-0-13-235088-4  | Database Design    | Jones  |

> Each ISBN is unique and cannot be NULL.

---
### Practice Questions

1. Why is a primary key important?

2. Can a primary key contain NULL values?

3. Write a **CREATE TABLE** statement for **Employees** with **EmployeeID** as the primary key.

---

### Quiz

**1. How many primary keys can a table have?**

    a) 0
    b) 1
    c) As many as needed

---

### Summary for Primary Keys

> A primary key **uniquely identifies** each row. It is essential for data integrity and efficient queries.

---

## 2. Foreign Keys

A Foreign Key is a **column (or set of columns) in one table that refers to the Primary Key of another table.** 

> It creates a link between the two tables and enforces referential integrity: 

> you cannot insert a value in the foreign key column that does not exist in the referenced primary key column.

Foreign keys can be **NULL** (meaning no relationship) unless defined as **NOT NULL**.

#### Real-World Example

In an **Orders table**, **CustomerID** is a foreign key referencing the Customers table’s **CustomerID**. This ensures every order is linked to a valid customer.

---

#### SQL Syntax

```sql

    -- Inline foreign key
    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT REFERENCES Customers(CustomerID),
        OrderDate DATE
    );

    -------------------------------------------------------

    -- Separate constraint (more explicit)
    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT,
        OrderDate DATE,
        FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
    );

    -------------------------------------------------------

    -- Add foreign key after table creation
    ALTER TABLE Orders
    ADD FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID);

```

#### Examples

#### Example 1: One foreign key

```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        CategoryID INT,
        FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
    );
```

#### Example 2: Composite foreign key

```sql
    CREATE TABLE OrderItems (
        OrderID INT,
        ProductID INT,
        Quantity INT,
        PRIMARY KEY (OrderID, ProductID),
        FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
        FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
    );
```

#### Example 3: With ON DELETE / ON UPDATE actions

```sql
    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT,
        FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
            ON DELETE CASCADE
            ON UPDATE CASCADE
    );
```

### Visual Table Illustration

#### Customers (parent)

| CustomerID | Name  |
|------------|-------|
| 1          | Alice |
| 2          | Bob   |


#### Orders (child, foreign key CustomerID)

| OrderID | CustomerID | Amount |
|---------|------------|--------|
| 101     | 1          | 250    |
| 102     | 2          | 100    |
| 103     | 1          | 150    |

> Attempting to insert CustomerID = 99 (non‑existent) would fail.

---

## Practice Questions

1. What happens if you try to delete a customer who has orders, if the foreign key has no ON **DELETE** clause?

2. Write a **CREATE TABLE** for Reviews where **ProductID** references **Products(ProductID)**.

3. Can a foreign key reference a non‑primary key column? (Answer: It must reference a unique column, usually primary key.)

--

### Quiz

**2. A foreign key ensures:**

    a) Data is sorted
    b) Referential integrity between tables
    c) Automatic indexing


---

### Summary for Foreign Keys

> Foreign keys maintain links between tables and prevent orphaned records. They are the foundation of relational database integrity.

---

## 3. Constraints

**Constraints** are **rules applied to table columns** to enforce data integrity. 

> SQL provides several types:

* **NOT NULL** – column cannot have NULL values.

* **UNIQUE** – all values in the column must be different.

* **PRIMARY KEY** – combination of NOT NULL and UNIQUE.

* **FOREIGN KEY** – links tables (covered above).

* **CHECK** – validates data against a condition.

* **DEFAULT** – provides a default value when none is supplied.

> Constraints can be defined at column level or table level.


#### Real-World Example

> In an **Employees** table:

* **EmployeeID** must be **UNIQUE** and **NOT NULL (PRIMARY KEY).**

* **Email** must be **UNIQUE**.

* **Salary** must be **>= 0 (CHECK)**.

* **HireDate** should default to **current date**.

---

#### SQL Syntax

```sql
    CREATE TABLE table_name (
        column1 datatype CONSTRAINT constraint_name PRIMARY KEY,
        column2 datatype NOT NULL,
        column3 datatype UNIQUE,
        column4 datatype CHECK (condition),
        column5 datatype DEFAULT default_value,
        CONSTRAINT fk_name FOREIGN KEY (column) REFERENCES other_table(column)
    );
```

#### Examples

#### Example 1: NOT NULL and UNIQUE

```sql
    CREATE TABLE Users (
        UserID INT PRIMARY KEY,
        Username VARCHAR(50) NOT NULL UNIQUE,
        Email VARCHAR(100) NOT NULL UNIQUE
    );
```    

#### Example 2: CHECK constraint

```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        Price DECIMAL(10,2) CHECK (Price >= 0),
        Stock INT CHECK (Stock >= 0)
    );
```

#### Example 3: DEFAULT constraint

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderDate DATE DEFAULT CURRENT_DATE,
    Status VARCHAR(20) DEFAULT 'Pending'
);
```

#### Example 4: Multiple constraints on one column

```sql
    CREATE TABLE Employees (
        EmpID INT PRIMARY KEY,
        Age INT NOT NULL CHECK (Age >= 18 AND Age <= 65)
    );
```

### Visual Table Illustration

#### Table: Accounts with constraints

| Column    | Constraint                |
|-----------|---------------------------|
| AccountID | PRIMARY KEY               |
| Balance   | CHECK (Balance >= 0)      |
| Currency  | DEFAULT 'USD'             |
| Email     | UNIQUE, NOT NULL          |

> Attempting to insert a negative balance or duplicate email would be rejected.

---

### Practice Questions

1. Write a CHECK constraint to ensure OrderAmount is greater than 0.

2. Create a table Students with StudentID primary key, Email unique and not null, and Age between 5 and 100.

3. What is the difference between UNIQUE and PRIMARY KEY?

---

### Quiz

**3. Which constraint ensures a column cannot have empty (NULL) values?**

    a) UNIQUE
    b) NOT NULL
    c) CHECK

---

### Summary for Constraints

> Constraints enforce business rules at the database level, preventing invalid data insertion and maintaining consistency.

---
