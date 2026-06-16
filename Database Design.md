# Phase 7: Database Design

* **Primary Keys**
* **Foreign Keys**
* **Constraints**
* **Normalization**
* **Relationships**
    * One-to-One
    * One-to-Many
    * Many-to-Many

---


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

## 4. Normalization

**(Normalization is the process of organizing database tables to reduce redundancy and dependency.)** 

> It involves splitting tables and defining relationships according to a set of normal forms (1NF, 2NF, 3NF, BCNF, etc.). 

**The goals:**

* Eliminate duplicate data.

* Avoid update, insertion, and deletion anomalies.

* Ensure data dependencies make sense.

#### Normal Forms (simplified):

* 1NF – Each column contains atomic (indivisible) values; no repeating groups.

* 2NF – 1NF + all non‑key columns depend on the whole primary key (no partial dependencies).

* 3NF – 2NF + no transitive dependencies (non‑key columns depend only on the primary key).


#### Real-World Example

**Unnormalized table (repeating groups):**

| OrderID | Customer | Products                  |
|---------|----------|---------------------------|
| 101     | Alice    | Laptop, Mouse, Keyboard   |

**Problems:** 

> cannot search for a single product efficiently; hard to add/remove products.


**Normalized (1NF):** 

> separate rows per product. But still redundant customer name.


**2NF & 3NF:** 

> split into Orders, Customers, OrderItems tables.

---

### Visual Table Illustration

#### Before Normalization (Denormalized)

| EmployeeID | Name  | Department | DeptPhone |
|------------|-------|------------|-----------|
| 1          | Alice | Sales      | 555-1000  |
| 2          | Bob   | IT         | 555-2000  |
| 3          | Carol | Sales      | 555-1000  |

**Redundancy**: department phone repeated for each employee in Sales.

#### After Normalization (3NF)

#### Employees

| EmpID | Name  | DeptID |
|-------|-------|--------|
| 1     | Alice | 10     |
| 2     | Bob   | 20     |
| 3     | Carol | 10     |

#### Departments

| DeptID | DeptName | Phone     |
|--------|----------|-----------|
| 10     | Sales    | 555-1000  |
| 20     | IT       | 555-2000  |

---

### Practice Questions

1. What anomaly occurs if you store customer address in every order row?

2. Convert a table Books (BookID, Title, AuthorID, AuthorAddress) into 3NF.

3. Why is 1NF necessary before 2NF?

---

### Quiz

**4. Normalization helps to:**

    a) Increase redundancy
    b) Reduce data duplication and anomalies
    c) Speed up all queries

---

### Summary for Normalization

> Normalization eliminates redundancy and improves data integrity. 

> Most practical designs aim for 3NF, but occasional denormalization may be used for performance.


----

## 5. Relationships: One‑to‑One, One‑to‑Many, Many‑to‑Many

**Relationships describe how tables connect via foreign keys.**

#### 5.1 One‑to‑One (1:1)

* One row in table A is linked to exactly one row in table B.

* Implemented by placing a foreign key in one table with a **UNIQUE** constraint (or merging tables, but splitting is useful for security or optional data).

* Example: **Users** and **UserProfiles** (each user has one profile).


#### 5.2 One‑to‑Many (1:N)

* One row in table A can be linked to **many rows** in table B.

* The **many side** contains the foreign key.

* This is the most common relationship.

* Example: One customer can have many orders; **Orders** table has **CustomerID** foreign key.

#### 5.3 Many‑to‑Many (M:N)

* Many rows in table A can relate to many rows in table B.

* Implemented using a junction table (also called associative or bridge table) that contains foreign keys to both tables.

* Example: **Students** and **Courses** – a student can take many **courses**, a **course** can have many **students**. Junction table: **Enrollments** with **StudentID** and **CourseID** as composite primary key.

#### Real-World Example

* 1:1 – **Employees** and **EmployeeDetails** (sensitive info like SSN, separate for security).

* 1:N – **Categories** to **Products** (one category, many products).

* M:N – **Doctors** and **Patients** (many doctors see many patients; junction table Appointments).

---

#### SQL Syntax

* **One‑to‑Many (foreign key on many side)**

    ```sql

        CREATE TABLE Orders (
            OrderID INT PRIMARY KEY,
            CustomerID INT REFERENCES Customers(CustomerID)
        );

    ```

* **One‑to‑One (foreign key + UNIQUE)**

    ```sql
    
        CREATE TABLE UserProfiles (
            ProfileID INT PRIMARY KEY,
            UserID INT UNIQUE REFERENCES Users(UserID),
            Bio TEXT
        );
        
    ```

* **Many‑to‑Many (junction table)**

    ```sql

        CREATE TABLE Students (
            StudentID INT PRIMARY KEY,
            Name VARCHAR(100)
        );

        CREATE TABLE Courses (
            CourseID INT PRIMARY KEY,
            Title VARCHAR(100)
        );

        CREATE TABLE Enrollments (
            StudentID INT,
            CourseID INT,
            EnrollmentDate DATE,
            PRIMARY KEY (StudentID, CourseID),
            FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
            FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
        );

    ```

---

### Visual Table Illustration

* **One‑to‑Many**

    ----
    Categories (1) ──┬── Products (many)

    | CategoryID | Name         |
    |------------|--------------|
    | 1          | Electronics  |

    | ProductID | Name   | CategoryID |
    |-----------|--------|------------|
    | 101       | Laptop | 1          |
    | 102       | Mouse  | 1          |

* **Many‑to‑Many**

    Students ──┬── Enrollments ──┬── Courses


    **Students:**

    | StudentID | Name  |
    |-----------|-------|
    | 1         | Alice |
    | 2         | Bob   |

    **Courses:**

    | CourseID | Title   |
    |----------|---------|
    | 10       | Math    |
    | 20       | Science |


    **Enrollments:**

    | StudentID | CourseID |
    |-----------|----------|
    | 1         | 10       |
    | 1         | 20       |
    | 2         | 10       |

    > Alice takes Math and Science; Bob takes Math only.


---

#### Practice Questions

1. Identify the relationship between Authors and Books (one author can write many books; a book has one author). → One‑to‑many.

2. Design tables for Users and Roles where a user can have many roles and a role can belong to many users.


3. When would you use a one‑to‑one relationship instead of merging tables?

#### Quiz

**5. A many‑to‑many relationship requires:**

    a) A foreign key in one table
    b) A junction table
    c) Two primary keys in one table

---

### Summary for Relationships

* 1:1 – foreign key + unique constraint.

* 1:N – foreign key on the “many” side.

* M:N – junction table with two foreign keys.

> Understanding relationships is key to designing a normalized database.

---

## Practice Questions (Comprehensive)

Given the following business rules, design a normalized database schema (tables, columns, keys, constraints, relationships):

1. A Customer has a unique ID, name, and email.

2. A Product has a unique ID, name, price (must be >= 0), and a category (each product belongs to exactly one category).

3. A Category has a unique ID and name.

4. A customer can place many Orders; each order has a unique ID, order date, and total amount.

5. An order can contain many products; each product can appear in many orders. For each product in an order, we store the quantity purchased.

6. Each order belongs to exactly one customer.

> Write the CREATE TABLE statements including all constraints (primary keys, foreign keys, CHECK, NOT NULL where appropriate).

#### Answer (self‑check):

```sql

    CREATE TABLE Customers (
        CustomerID INT PRIMARY KEY,
        Name VARCHAR(100) NOT NULL,
        Email VARCHAR(100) UNIQUE NOT NULL
    );

    CREATE TABLE Categories (
        CategoryID INT PRIMARY KEY,
        Name VARCHAR(50) NOT NULL
    );

    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        Name VARCHAR(100) NOT NULL,
        Price DECIMAL(10,2) CHECK (Price >= 0),
        CategoryID INT NOT NULL,
        FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
    );

    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerID INT NOT NULL,
        OrderDate DATE NOT NULL,
        TotalAmount DECIMAL(10,2) CHECK (TotalAmount >= 0),
        FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
    );

    CREATE TABLE OrderItems (
        OrderID INT,
        ProductID INT,
        Quantity INT CHECK (Quantity > 0),
        PRIMARY KEY (OrderID, ProductID),
        FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE,
        FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
    );

```

---

## Quiz (Answers)

1. b) 1

2. b) Referential integrity between tables

3. b) NOT NULL

4. b) Reduce data duplication and anomalies

5. b) A junction table


---

## Interview Questions
1. **What is the difference between a primary key and a unique key?**

    **Answer**

    Both enforce uniqueness, but a table can have only one primary key (which is also NOT NULL). Unique keys can be NULL (one NULL allowed in most DBMS), and there can be multiple unique keys per table.

2. **What are the three types of relationships? Give examples.**

    **Answer**

    One‑to‑one (person ↔ passport), one‑to‑many (customer ↔ orders), many‑to‑many (students ↔ courses).

3. **Explain the purpose of normalization. What is 3NF?**
    
    **Answer**

    Normalization reduces redundancy and anomalies. 3NF means the table is in 2NF and has no transitive dependencies (non‑key columns depend only on the primary key, not on other non‑key columns).

4. **Can a foreign key reference a unique column that is not a primary key?**

    **Answer**
    
    Yes, it can reference any column that has a UNIQUE constraint.

5. **What happens if you try to delete a parent row that has child rows in a foreign key relationship without ON DELETE CASCADE?**
    
    **Answer**
    
    The delete will fail (or will set child foreign keys to NULL if ON DELETE SET NULL is defined). By default, most DBMS prevent the deletion to maintain referential integrity.
    
6. **How do you implement a many‑to‑many relationship?**

    **Answer**

    Create a junction table that contains foreign keys to both related tables, and usually a composite primary key using both foreign keys.

---

## Assignment

#### Scenario: 

> You are designing a database for a library management system. The requirements:

* Each Book has a unique ISBN, title, publication year, and a category (e.g., Fiction, Non‑fiction).

* Each Author has a unique ID, name, and country. A book can have one or more authors; an author can write many books.

* Each Member has a unique ID, full name, email, and membership date.


* A member can borrow many books over time, but a specific copy of a book can only be borrowed by one member at a time. For simplicity, assume a book has multiple copies (each copy has a unique copy number). Track which copy is borrowed, the borrow date, due date, and return date.


* Library branch – each branch has an ID, name, and address. Books are stored at specific branches. A book title may be available at multiple branches.

#### Tasks:

1. Identify all entities (tables) and their attributes.

2. Identify primary keys for each table.

3. Identify all relationships (1:1, 1:N, M:N) and foreign keys.

4. Write the complete CREATE TABLE SQL script with all constraints (PRIMARY KEY, FOREIGN KEY, NOT NULL, CHECK, UNIQUE, DEFAULT where appropriate).

5. Insert sample data (at least 3 books with multiple authors, 2 members, 2 branches, and several borrow records).

6. Write three example queries:

    * List all books currently borrowed by a specific member.

    * Show the number of books per category available at a given branch.

    * Find members who have overdue books (return date > due date, or not returned yet and due date < today).

**Bonus:** Explain which normal form your design satisfies (should be at least 3NF). If any denormalization was done, justify why.

---

## Summary of Phase 7

| Concept            | Purpose                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Primary Key**        | Uniquely identifies each row.                                           |
| **Foreign Key**        | Links tables and enforces referential integrity.                        |
| **Constraints**        | Rules (NOT NULL, UNIQUE, CHECK, DEFAULT) for data integrity.            |
| **Normalization**      | Process to reduce redundancy and anomalies (1NF, 2NF, 3NF).             |
| **1:1 Relationship**  | One row matches one row (foreign key + UNIQUE).                         |
| **1:N Relationship**  | One row matches many rows (foreign key on many side).                   |
| **    **  | Many rows match many rows (junction table).                             |


