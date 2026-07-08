# Phase 3: **Functions**

* **String Functions**
* **Numeric Functions**
* **Date Functions**
* **Aggregate Functions**
    * **COUNT**
    * **SUM**
    * **AVG**
    * **MIN**
    * **MAX**

## **Overview: SQL Functions**

**Functions in SQL are built‑in tools that perform operations on data.** They can transform text, calculate numbers, manipulate dates, or summarize entire datasets. Functions fall into two main categories:

**Scalar functions** – operate on a single value and return one result per row (e.g., UPPER(), ROUND()).

**Aggregate functions** – operate on a set of rows and return a single summary value (e.g., SUM(), AVG()).

---

### Real-World Example

#### A sales report needs:

> Customer names in uppercase (UPPER),

> Discounted price rounded to two decimals (ROUND),

> Orders from last month (DATE functions),

> Total sales per region (SUM and GROUP BY).

In this phase we first learn scalar functions (String, Numeric, Date), then the powerful aggregate functions.

---

## 1. **String Functions**

**String functions manipulate character data** – 
* change case, 
* extract parts, 
* find length, 
* concatenate, 
* etc.


### 1.1 Common String Functions

| Function                      | Description                         | Example (MySQL/PostgreSQL)          |
|-------------------------------|-------------------------------------|--------------------------------------|
| **UPPER**(str) / LOWER(str)       | Convert case                        | UPPER('sql') → 'SQL'                 |
| **CONCAT**(str1, str2, ...)       | Join strings                        | CONCAT('Mr. ', name)                 |
| **LENGTH**(str)                   | Number of characters                | LENGTH('hello') → 5                  |
| **SUBSTRING**(str, start, length) | Extract part                        | SUBSTRING('Database', 1, 4) → 'Data' |
| **TRIM**(str)                     | Remove leading/trailing spaces      | TRIM(' hi ') → 'hi'                  |
| **REPLACE**(str, old, new)        | Substitute substring                | REPLACE('cat', 'c', 'b') → 'bat'     |


### Real-World Example
   **Format customer names for a mailing label:** 
UPPER(last_name), CONCAT(first_name, ' ', last_name), and trim any extra spaces.

---

#### SQL Syntax (generic)

```sql 
    SELECT string_function(column) 
    FROM table;
```    

#### Examples

```sql 
    
    -- Uppercase and lowercase
    SELECT 
        UPPER(city) AS city_upper, 
        LOWER(email) AS email_lower 
    FROM Customers;

    -------------------------------------------------------

    -- Concatenate with separator (different DB syntax: CONCAT_WS in MySQL)
    SELECT 
        CONCAT(first_name, ' ', last_name) AS full_name 
    FROM Employees;

    -------------------------------------------------------

    -- Length of product names
    SELECT 
        product_name, 
        LENGTH(product_name) AS name_length 
    FROM Products;

    -------------------------------------------------------

    -- Extract first 3 characters of postal code
    SELECT 
        SUBSTRING(postal_code, 1, 3) AS area_code 
    FROM Addresses;

    -------------------------------------------------------

    -- Remove spaces and replace text
    SELECT 
        TRIM(comment) AS clean_comment, 
        REPLACE(phone, '-', '') AS phone_digits 
    FROM Feedback;

```
---

### Visual Table Illustration

#### Table: Users

| UserID | FirstName | LastName |
|--------|-----------|----------|
| 1      | John      | DOE      |
| 2      | Alice     | Smith    |

#### Query:


```sql
    SELECT 
        CONCAT(FirstName, ' ', UPPER(LastName)) AS FormattedName
    FROM Users;
```    

#### Result:

| FormattedName |
|---------------|
| John DOE      |
| Alice SMITH   |

---

#### Practice Questions

> Write a query that displays product names in lowercase.

> Show the first 5 characters of each email address.

> Concatenate address, city, and postal code with commas between.

#### Quiz

**1. Which function returns the number of characters in a string?**

    a) COUNT()

    b) LEN() / LENGTH()

    c) CHAR()

---

### Summary (String Functions)

> String functions help clean, format, and extract text. Use them in SELECT, WHERE, or ORDER BY.

---

## 2. **Numeric Functions**

**Numeric functions perform mathematical operations on numbers** – 
* rounding 
* absolute values 
* modulo 
* etc.

---

#### Common Numeric Functions

| Function                | Description                                    | Example                          |
|-------------------------|------------------------------------------------|----------------------------------|
| **ROUND**(column, decimals) | Rounds to specified decimal places             | ROUND(15.678, 1) → 15.7          |
| **CEILING**(column) / CEIL  | Rounds up to nearest integer                   | CEIL(15.2) → 16                  |
| **FLOOR**(column)           | Rounds down to nearest integer                 | FLOOR(15.9) → 15                 |
| **ABS**(column)             | Absolute value                                 | ABS(-50) → 50                    |
| **MOD**(a, b) or a % b      | Remainder of division                          | MOD(10, 3) → 1                   |
| **POWER**(a, b)             | a raised to power b                            | POWER(2, 3) → 8                  |

---

### Real-World Example

**Calculate discounted price:** ROUND(price * 0.85, 2).

**Get absolute difference between target and actual sales:** ABS(target - actual).

---

#### SQL Syntax

```SQL
    SELECT numeric_function(column) 
    FROM table;
```                

#### Examples

```sql

    -- Round prices to 2 decimals
    SELECT 
        product_name, 
        ROUND(price, 2) AS rounded_price 
    FROM Products;

    -------------------------------------------------------

    -- Ceiling and floor for inventory bins
    SELECT 
        CEILING(quantity / 10) AS bins_needed 
    FROM Inventory;

    -------------------------------------------------------

    -- Absolute difference
    SELECT 
        employee_id, 
        ABS(salary - 50000) AS distance_from_50k 
    FROM Salaries;

    -------------------------------------------------------

    -- Even or odd using modulo
    SELECT 
        order_id, 
        MOD(order_id, 2) AS is_odd 
    FROM Orders;

```
---

### Visual Table Illustration

#### Table: Prices

| Product | Price  |
|---------|--------|
| Laptop  | 899.99 |
| Mouse   | 12.49  |

#### Query: 
```sql
    SELECT 
        Product, 
        ROUND(Price * 1.1, 0) AS PriceWithTax 
    FROM Prices;
```    

#### Result:

| Product | PriceWithTax |
|---------|--------------|
| Laptop  | 990          |
| Mouse   | 14           |

---

#### Practice Questions
 
> Show the ceiling and floor of each product’s price.

> Calculate 15% VAT on each amount, rounded to 2 decimals.

> Find the remainder when employee ID is divided by 5.

---

#### Quiz

**2. ROUND(123.456, 1) returns:**

    a) 123.5
    b) 123.46
    c) 123.4

---

### Summary (Numeric Functions)

> Numeric functions allow precise mathematical transformations directly in SQL.

---

## 3. Date Functions

**Date functions extract parts of dates, perform arithmetic, and format date values.** 

> Syntax varies between databases (MySQL, PostgreSQL, SQL Server). 

We’ll use standard SQL where possible and note differences.

---

#### Common Date Functions

| Function           | Description          | MySQL Example                                      | PostgreSQL Example                                      |
|--------------------|----------------------|----------------------------------------------------|----------------------------------------------------------|
| **CURRENT_DATE**       | Today's date         | CURRENT_DATE                                       | CURRENT_DATE                                             |
| **CURRENT_TIMESTAMP**  | Now                  | NOW()                                              | CURRENT_TIMESTAMP                                        |
| **YEAR**(date)         | Extract year         | YEAR(order_date)                                   | EXTRACT(YEAR FROM order_date)                            |
| **MONTH**(date)        | Extract month        | MONTH(order_date)                                  | EXTRACT(MONTH FROM order_date)                           |
| **DAY**(date)          | Extract day          | DAY(order_date)                                    | EXTRACT(DAY FROM order_date)                             |
| **DATEDIFF**(date1, date2) | Days between    | DATEDIFF('2025-12-31', '2025-01-01')               | ('2025-12-31' - '2025-01-01')                            |
| **DATE_ADD**(date, INTERVAL) | Add time       | DATE_ADD(hiredate, INTERVAL 1 YEAR)                | hiredate + INTERVAL '1 YEAR'                             | 


--- 

### Real-World Example

**Find employees hired more than 5 years ago:** hiredate <= CURRENT_DATE - INTERVAL 5 YEAR.

**Extract month from order date to see seasonal trends.**

---

#### SQL Syntax (MySQL flavored, but adapt)

```sql
    SELECT date_function(date_column) 
    FROM table;
```    

#### Examples

```sql

    -- Current date
    SELECT CURRENT_DATE AS today;

    -------------------------------------------------------

    -- Extract year and month from order date
    SELECT 
        order_id, 
        YEAR(order_date) AS order_year, 
        MONTH(order_date) AS order_month 
    FROM Orders;

    -------------------------------------------------------

    -- Age of each employee in years (difference in days / 365.25)
    SELECT 
        name, 
        DATEDIFF(CURRENT_DATE, birth_date) / 365.25 AS age 
    FROM Employees;

    -------------------------------------------------------

    -- Orders placed in the last 30 days
    SELECT * 
    FROM Orders 
    WHERE order_date >= CURRENT_DATE - INTERVAL 30 DAY;

    -------------------------------------------------------

    -- Add 1 year to membership expiry
    UPDATE Members 
    SET expiry_date = DATE_ADD(expiry_date, INTERVAL 1 YEAR) 
    WHERE member_id = 101;
    
```
---

### Visual Table Illustration

#### Table: Events

| Event      | EventDate  |
|------------|------------|
| Conference | 2025-06-15 |
| Webinar    | 2025-07-01 |

#### Query: 

```sql
    SELECT 
        Event, 
        YEAR(EventDate) AS Year, 
        MONTH(EventDate) AS Month 
    FROM Events;
```    
#### Result:

| Event      | Year | Month |
|------------|------|-------|
| Conference | 2025 | 6     |
| Webinar    | 2025 | 7     |

---

### Practice Questions

> Show the current date and time.

> Extract the day of the week (or weekday number) from each order date.

> Find products that expire within the next 90 days.

---

### Quiz

**3. Which function returns the current date in most SQL databases?**

    a) GETDATE()

    b) CURRENT_DATE

    c) TODAY()

---

### Summary (Date Functions)
> Date functions help filter, group, and calculate based on time. Learn your DBMS’s specific functions.

---

## 4. **Aggregate Functions (COUNT, SUM, AVG, MIN, MAX)**

**Aggregate functions operate on multiple rows and return a single value per group (or whole table).** They ignore NULL values (except COUNT(*)).

---

### 4.1 **COUNT**

**COUNT()** returns the number of rows that match a condition.

**COUNT(*)** counts all rows, including NULLs.

**COUNT(column)** counts non‑NULL values in that column.

---

### Real-World Example

How many customers have placed an order? COUNT(DISTINCT customer_id).

---
#### SQL Syntax

```sql 

    SELECT COUNT(*) 
    FROM table 
    WHERE condition;

    -------------------------------------------------------

    SELECT COUNT(column) 
    FROM table;
    
```    

#### Examples

```sql

    -- Total number of employees
    SELECT 
        COUNT(*) AS total_employees 
    FROM Employees;

    -------------------------------------------------------

    -- Number of employees with a non‑NULL email
    SELECT 
        COUNT(email) AS emails_provided 
    FROM Employees;

    -------------------------------------------------------    

    -- How many distinct departments?
    SELECT 
        COUNT(DISTINCT department) AS unique_depts 
    FROM Employees;
    
```
---

### **4.2 SUM**

**SUM()** adds up all values in a numeric column.

### Real-World Example

```sql
    SUM(amount) 
    FROM sales 
    WHERE quarter = 1
```    
---

#### SQL Syntax

```sql    
    SELECT 
        SUM(column) 
    FROM table 
    WHERE condition;
```    
#### Examples

```sql

    -- Total inventory value
    SELECT 
        SUM(price * stock) AS total_value 
    FROM Products;

    -------------------------------------------------------

    -- Total salary cost for IT department
    SELECT SUM(salary) 
    FROM Employees 
    WHERE department = 'IT';

```
---

### 4.3 **AVG**

**AVG()** calculates the arithmetic mean of a numeric column.

### Real-World Example

Average order value: **AVG(total_amount) FROM Orders.**

---

#### SQL Syntax

```sql
    SELECT AVG(column) 
    FROM table;
```    

#### Examples

```sql

    -- Average product price
    SELECT 
        AVG(price) AS avg_price 
    FROM Products;
    
    -------------------------------------------------------

    -- Average salary excluding NULLs
    SELECT AVG(salary) 
    FROM Employees;

```


### 4.4 **MIN and MAX**


**MIN()** returns the smallest value; **MAX()** returns the largest.

### Real-World Example

Earliest and latest order dates: **MIN(order_date), MAX(order_date)**.

---

#### SQL Syntax

```sql
    SELECT 
        MIN(column), 
        MAX(column) 
    FROM table; 
```    

#### Examples

```sql

    -- Cheapest and most expensive product
    SELECT 
        MIN(price) AS cheapest, 
        MAX(price) AS most_expensive 
    FROM Products;
    
    -------------------------------------------------------

    -- First and last hire date
    SELECT 
        MIN(hire_date) AS first_hire, 
        MAX(hire_date) AS last_hire 
    FROM Employees;

```

---

### Visual Table Illustration

#### Table: Sales

| SaleID | Amount | Region |
|--------|--------|--------|
| 1      | 100    | North  |
| 2      | 200    | South  |
| 3      | 150    | North  |
| 4      | NULL   | East   |

### Queries and results:

```sql

    SELECT COUNT(*) FROM Sales; → 4

    -------------------------------------------------------

    SELECT COUNT(Amount) FROM Sales; → 3 (ignores NULL)

    -------------------------------------------------------

    SELECT SUM(Amount) FROM Sales; → 450

    -------------------------------------------------------

    SELECT AVG(Amount) FROM Sales; → 150 (450/3)

    -------------------------------------------------------

    SELECT MIN(Amount) FROM Sales; → 100

    -------------------------------------------------------

    SELECT MAX(Amount) FROM Sales; → 200
    
```
---

#### Practice Questions (Aggregates)
 
> Count how many orders have a total > $500.

> Calculate the total quantity sold for product ID 5.

> Find the average rating from the Reviews table.

> Get the minimum and maximum price among active products.

> Count distinct customer IDs that have made a purchase.

---

### Quiz
 

**4. AVG(column) ignores:**

    a) Zero values
    b) NULL values
    c) Negative values


**5. COUNT(*) counts:**

    a) Only non‑NULL rows
    b) All rows regardless of NULLs
    c) Distinct values

---

### Summary (Aggregate Functions)

> Aggregates summarize data: count, sum, average, min, max. They are often used with GROUP BY (next phase) for grouped summaries.

---

### Practice Questions (Comprehensive)

> Use a table **Orders**:

| OrderID | Customer | OrderDate  | Amount | Status    |
|---------|----------|------------|--------|-----------|
| 1       | Alice    | 2025-01-10 | 250.00 | Shipped   |
| 2       | Bob      | 2025-02-15 | 99.99  | Pending   |
| 3       | Alice    | 2025-02-20 | 150.00 | Shipped   |
| 4       | Charlie  | 2025-03-01 | 75.50  | Cancelled |
| 5       | Bob      | 2025-03-10 | 300.00 | Shipped   |

#### Write queries:

> Convert all customer names to uppercase.

> Round each amount to the nearest whole number.

> Extract the month from OrderDate for each order.

> Calculate the total amount of all shipped orders.

> Find the average amount of orders placed in February 2025.

> Count how many orders are pending.

> Get the minimum and maximum order amount.

> Find the total number of distinct customers.

---

### Answers (self‑check):

```sql

    -- 1
    SELECT UPPER(Customer) 
    FROM Orders;

    -------------------------------------------------------

    -- 2
    SELECT 
        OrderID, 
        ROUND(Amount, 0) 
    FROM Orders;

    -------------------------------------------------------

    -- 3
    SELECT 
        OrderID, 
        MONTH(OrderDate) 
    FROM Orders;  -- or EXTRACT

    -------------------------------------------------------

    -- 4
    SELECT 
        SUM(Amount) 
    FROM Orders 
    WHERE Status = 'Shipped';

    -------------------------------------------------------

    -- 5
    SELECT 
        AVG(Amount) 
    FROM Orders 
    WHERE OrderDate BETWEEN '2025-02-01' AND '2025-02-28';

    -------------------------------------------------------

    -- 6
    SELECT COUNT(*) 
    FROM Orders 
    WHERE Status = 'Pending';

    -------------------------------------------------------

    -- 7
    SELECT 
        MIN(Amount), 
        MAX(Amount) 
    FROM Orders;

    -------------------------------------------------------

    -- 8
    SELECT 
        COUNT(DISTINCT Customer) 
    FROM Orders;

```

---

## **Quiz (Answers)**


1. b) LEN() / LENGTH()

2. a) 123.5

3. b) CURRENT_DATE

4. b) NULL values

5. b) All rows regardless of NULLs

---

## **Interview Questions**

1. **Q: What is the difference between COUNT(*) and COUNT(column)?**
    
    **A:** 
    > COUNT(*) counts all rows in the result set, including rows with NULLs. 

    >COUNT(column) counts only rows where that specific column is NOT NULL.

    ----

2. **Q: Can aggregate functions be used in a WHERE clause?**

    **A:** 
    > No, that is what HAVING is for (with GROUP BY). 

    > WHERE filters rows before aggregation.

    ---

3. **Q: What happens if you use AVG() on an integer column? Will the result be integer or decimal?**

    **A:** 
    >It depends on the DBMS. Most return a decimal (e.g., PostgreSQL returns numeric, SQLite returns real). 
    
    > MySQL returns a decimal, but if all inputs are integers, the result may be a decimal. Cast if needed.

    ---
4. **Q: Give an example of using a string function inside an aggregate function.**

    **A:** 
    >   SELECT COUNT(DISTINCT UPPER(city)) FROM Customers; – counts unique city names case‑insensitively.

    ---
5. **Q: How do you get the number of days between two dates in SQL?**

    **A:** 
    >DATEDIFF(date1, date2) in MySQL, date1 - date2 in PostgreSQL, JULIANDAY(date1) - JULIANDAY(date2) in SQLite.

---

## **Assignment**

### Scenario: 
> You have an e‑commerce database. Tables: Products, Orders, OrderItems.

### Tasks:


1. **Create and populate the tables (use any DBMS or SQLiteOnline).**

    ```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        Name VARCHAR(100),
        Category VARCHAR(50),
        Price DECIMAL(8,2),
        Stock INT
    );
    
    -------------------------------------------------------

    CREATE TABLE Orders (
        OrderID INT PRIMARY KEY,
        CustomerEmail VARCHAR(100),
        OrderDate DATE,
        Status VARCHAR(20)
    );
    
    -------------------------------------------------------

    CREATE TABLE OrderItems (
        ItemID INT PRIMARY KEY,
        OrderID INT,
        ProductID INT,
        Quantity INT,
        UnitPrice DECIMAL(8,2)
    );  

    ```

2. **Insert at least 5 products, 3 orders, and 8 order items (each order has 2‑3 items).**

3. **Write SQL queries to answer:**

    > Show product names in uppercase, with their price rounded to zero decimals.

    > Extract the year and month from each order date.

    > Count the total number of orders.

    > Calculate the total revenue (sum of Quantity * UnitPrice) from all order items.

    > Find the average product price per category.

    > Get the most expensive product name and price.

    > Find the earliest and latest order date.

    > (Bonus) Count how many orders were placed in the last 30 days.

4. **Use string functions to clean customer email domains: extract the domain part (e.g., from 'john@gmail.com' get 'gmail.com').**

---

## **Summary of Phase 3**

| Function Category     | Purpose                                                                 |
|-----------------------|-------------------------------------------------------------------------|
| **String functions**      | Modify, format, or extract text                                         |
| **Numeric functions**     | Perform math operations (round, floor, etc.)                            |
| **Date functions**        | Work with dates – extract parts, add intervals                          |
| **Aggregate functions**   | Summarize many rows into one value: COUNT, SUM, AVG, MIN, MAX           |

---

<br/><br/><br/>
<center> <b>Happy Learning! 😊</b> </center>
