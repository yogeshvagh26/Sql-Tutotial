# Phase 9: Transactions & Security

* **Transactions**
* **COMMIT**
* **ROLLBACK**
* **ACID Properties**
* **User Permissions**
* **Security Best Practices**

---

### Overview

This phase covers two pillars of production databases:

* **Transactions** – ensuring data consistency when multiple operations are grouped together.

* **Security** – controlling who can access what, and protecting data from threats.

Together, they guarantee your database remains reliable, consistent, and safe even under concurrent access and malicious attempts.

---

## 1. Transactions

A **transaction** is a logical unit of work that consists of one or more SQL statements. It guarantees that either all operations succeed (commit) or none of them take effect (rollback). This all‑or‑nothing behaviour is essential for data integrity.

**Key properties (ACID – detailed later):**

* **Atomicity** – indivisible.

* **Consistency** – valid state.

* **Isolation** – concurrent transactions do not interfere.

* **Durability** – committed changes persist.

**Real-World Example**

> When you transfer money from account A to account B:

1. Debit A.

2. Credit B.

> If step 2 fails (e.g., B does not exist), step 1 must be undone. A transaction ensures both happen or neither.

---
#### SQL Syntax

```sql

    -- Start a transaction
    BEGIN;               -- or START TRANSACTION;

    -- Execute DML statements
    UPDATE ...;
    INSERT ...;

    -- If all OK
    COMMIT;

    -- If something fails
    ROLLBACK;

```
---

#### Examples

**Example 1: Simple money transfer**

```sql
    BEGIN;

    UPDATE Accounts 
    SET Balance = Balance - 500 
    WHERE AccountID = 101;

    UPDATE Accounts 
    SET Balance = Balance + 500 
    WHERE AccountID = 102;

    COMMIT;
```

**Example 2: With savepoint for partial rollback**

```sql

    BEGIN;

    INSERT INTO Orders (OrderID, CustomerID) VALUES 
    (1001, 5);

    SAVEPOINT before_items;
    
    INSERT INTO OrderItems (OrderID, ProductID, Quantity) VALUES 
    (1001, 10, 2);
    -- Oops, ProductID 10 is invalid
    
    ROLLBACK TO SAVEPOINT before_items;  -- undo only the item insert, keep the order

    COMMIT;  -- Now order exists but with no items (you can later add)

```

**Example 3: Transaction in stored procedure (MySQL)**

```sql
    DELIMITER //
    CREATE PROCEDURE Transfer(IN from_acc INT, IN to_acc INT, IN amount DECIMAL)
    BEGIN
        DECLARE EXIT HANDLER FOR SQLEXCEPTION
        BEGIN
            ROLLBACK;
            RESIGNAL;
        END;
        START TRANSACTION;
        UPDATE Accounts SET Balance = Balance - amount WHERE AccountID = from_acc;
        UPDATE Accounts SET Balance = Balance + amount WHERE AccountID = to_acc;
        COMMIT;
    END//
    DELIMITER ;
```

---

### Visual Table Illustration

Before transaction:
    
| AccountID | Balance |
|-----------|---------|
| 101       | 1000    |
| 102       | 500     |    

**Transaction:**

```sql
    BEGIN;
    UPDATE Accounts SET Balance = Balance - 200 WHERE AccountID = 101;
    UPDATE Accounts SET Balance = Balance + 200 WHERE AccountID = 102;
    COMMIT;
```

**After commit:**

| AccountID | Balance |
|-----------|---------|
| 101       | 800     |
| 102       | 700     |

> If ROLLBACK instead of COMMIT, balances remain as before.

---
#### Practice Questions

1. What happens if a transaction is left open without COMMIT or ROLLBACK?

2. Write a transaction to add a new customer and their first order.

3. Why is BEGIN needed before a series of operations?

#### Quiz

**1. A transaction ensures:**

    a) Data is always encrypted
    b) All operations succeed or none take effect
    c) Queries are faster

> Answer: b

---

## 2. COMMIT

**COMMIT makes all changes performed during the current transaction permanent and visible to other users**. Once committed, changes cannot be undone (except via a new compensating transaction). It also releases any locks held by the transaction and ends the transaction.

#### Real-World Example

After successfully updating both accounts in a transfer, you **COMMIT** to finalise the transaction. The customer now sees the updated balance.

#### SQL Syntax

```sql
    COMMIT;
    -- or
    COMMIT WORK;
```

#### Examples

**Example 1: Explicit commit after successful operations**

```sql
    BEGIN;

        DELETE FROM TempTable 
        WHERE Processed = 1;

        INSERT INTO AuditLog (Action, Timestamp) VALUES 
        ('Cleaned up temp data', NOW());

    COMMIT;
```

**Example 2: Auto‑commit (default in many clients)**

```sql

    -- Turn off auto‑commit to group statements
    SET autocommit = 0;

    BEGIN;

    -- ... operations ...
    COMMIT;

    SET autocommit = 1;  -- restore if needed

```

**Example 3: Commit after multiple inserts (batch)**

```sql

    BEGIN;

    INSERT INTO Sales (ProductID, Quantity) VALUES (1, 5);

    INSERT INTO Sales (ProductID, Quantity) VALUES (2, 3);

    INSERT INTO Sales (ProductID, Quantity) VALUES (3, 7);

    COMMIT;

```
---

### Visual Table Illustration

**State visibility timeline:**

| Time | Action    | Visible to others? |
|------|-----------|---------------------|
| T1   | BEGIN     | N/A                 |
| T2   | UPDATE    | No (transaction not committed) |
| T3   | UPDATE    | No                  |
| T4   | COMMIT    | Yes – changes permanent |

---

#### Practice Questions

1. What happens if you COMMIT after a DELETE without WHERE?

2. Can you ROLLBACK after COMMIT? Why?

3. Why is auto‑commit dangerous for multi‑step operations?

---

#### Quiz

**2. COMMIT makes changes:**

    a) Temporary
    b) Permanent and visible
    c) Reversible

> Answer: b

---




    













































