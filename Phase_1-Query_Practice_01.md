#### 1. Create Database
```sql
    CREATE DATABASE company_db;
```


#### 2. Use the database
```sql
    USE company_db;
```

#### 3. Create Tables
```sql
    CREATE TABLE employees (
        employee_id INT,
        first_name VARCHAR(50),
        last_name VARCHAR(50),
        department VARCHAR(50),
        salary DECIMAL(10,2),
        hire_date DATE
    );
```

```sql
    CREATE TABLE products (
        product_id INT,
        product_name VARCHAR(100),
        category VARCHAR(50),
        price DECIMAL(10,2),
        stock_quantity INT
    );
```

```sql
    CREATE TABLE customers (
        customer_id INT,
        first_name VARCHAR(50),
        last_name VARCHAR(50),
        city VARCHAR(50),
        state VARCHAR(50),
        signup_date DATE
    );
```

#### 4. Insert Sample Data
```sql
    INSERT INTO employees (employee_id, first_name, last_name, department, salary, hire_date)
    VALUES
    (1, 'John', 'Smith', 'Sales', 65000, '2020-01-15'),
    (2, 'Emma', 'Johnson', 'Marketing', 72000, '2019-03-22'),
    (3, 'Michael', 'Brown', 'Sales', 58000, '2021-07-01'),
    (4, 'Sarah', 'Davis', 'IT', 85000, '2018-11-10'),
    (5, 'David', 'Miller', 'IT', 79000, '2020-06-19'),
    (6, 'Laura', 'Wilson', 'Marketing', 67000, '2022-02-28'),
    (7, 'James', 'Moore', 'Sales', 61000, '2021-09-14'),
    (8, 'Linda', 'Taylor', 'HR', 55000, '2020-12-01'),
    (9, 'Robert', 'Anderson', 'IT', 92000, '2017-05-05'),
    (10, 'Mary', 'Thomas', 'HR', 48000, '2023-01-20');
```

```sql
    INSERT INTO products (product_id, product_name, category, price, stock_quantity)
    VALUES
    (101, 'Laptop Pro', 'Electronics', 1200.00, 15),
    (102, 'Wireless Mouse', 'Accessories', 25.50, 200),
    (103, 'USB-C Hub', 'Accessories', 45.00, 0),
    (104, 'Monitor 27"', 'Electronics', 350.00, 8),
    (105, 'Desk Lamp', 'Furniture', 40.00, 45),
    (106, 'Gaming Keyboard', 'Electronics', 95.00, 12),
    (107, 'Office Chair', 'Furniture', 250.00, 5),
    (108, 'HDMI Cable', 'Accessories', 15.00, 150),
    (109, 'Smartphone X', 'Electronics', 999.00, 0),
    (110, 'Notebook Pack', 'Office', 12.00, 300);
```

```sql
    INSERT INTO customers (customer_id, first_name, last_name, city, state, signup_date)
    VALUES
    (201, 'Alice', 'Walker', 'New York', 'NY', '2022-05-10'),
    (202, 'Bob', 'Green', 'Los Angeles', 'CA', '2021-08-21'),
    (203, 'Charlie', 'Adams', 'Chicago', 'IL', '2023-02-14'),
    (204, 'Diana', 'Nelson', 'Houston', 'TX', '2020-11-01'),
    (205, 'Ethan', 'Carter', 'Phoenix', 'AZ', '2019-07-19'),
    (206, 'Fiona', 'Mitchell', 'New York', 'NY', '2022-12-05'),
    (207, 'George', 'Perez', 'Los Angeles', 'CA', '2023-06-30'),
    (208, 'Hannah', 'Roberts', 'Chicago', 'IL', '2021-04-17');
```
---

