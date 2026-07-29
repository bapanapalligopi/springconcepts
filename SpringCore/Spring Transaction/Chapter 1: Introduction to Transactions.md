Perfect. Now we start one of the **most important topics in Spring Framework**.

This topic is asked in almost every interview for **2 years experienced Java/Spring Boot developers**.

---

# Module 4: Spring Transaction Management

## Chapter 1: Introduction to Transactions

As always, we'll learn using:

> **Why → What → How → Where → Internal Working → Real Example → Interview Questions → Best Practices**

---

# 1. Why do we need Transactions?

Let's understand with a real-world example.

Imagine you transfer **₹10,000** from Account A to Account B.

The application performs two SQL statements.

```sql
UPDATE account
SET balance = balance - 10000
WHERE id = 1;
```

```sql
UPDATE account
SET balance = balance + 10000
WHERE id = 2;
```

Looks simple.

---

## Scenario 1: Everything Works

Initial State

| Account | Balance |
| ------- | ------: |
| A       | ₹50,000 |
| B       | ₹20,000 |

After transfer

| Account | Balance |
| ------- | ------: |
| A       | ₹40,000 |
| B       | ₹30,000 |

Everything is correct.

---

## Scenario 2: Failure Without Transaction

Suppose the first query executes successfully.

```sql
UPDATE account
SET balance = balance - 10000
WHERE id = 1;
```

Now the server crashes before executing the second query.

Database becomes

| Account | Balance |
| ------- | ------: |
| A       | ₹40,000 |
| B       | ₹20,000 |

Question:

Where did ₹10,000 go?

**It is lost.**

The database is now inconsistent.

---

# Problem

Without transactions,

multiple SQL statements execute **independently**.

If one succeeds and another fails,

the database may contain **partial updates**.

This can lead to:

* Incorrect account balances
* Lost money
* Duplicate orders
* Missing inventory
* Data corruption

---

# Solution: Transaction

A transaction groups multiple database operations into **one logical unit of work**.

```text
Transfer Money

        │

        ▼

Debit Account A

        │

        ▼

Credit Account B

        │

        ▼

One Transaction
```

The rule is simple:

> **Either all operations succeed or none of them succeed.**

---

# 2. What is a Transaction?

### Definition

A **transaction** is a sequence of one or more database operations that are executed as a **single unit**.

The database guarantees:

```text
All Success

OR

All Fail
```

Nothing in between.

---

# Real Life Analogy

Imagine ordering a mobile phone online.

Steps:

```text
Payment

↓

Reduce Inventory

↓

Create Order

↓

Send Confirmation Email
```

If payment succeeds but inventory update fails,

should the order continue?

No.

Everything should roll back.

That's a transaction.

---

# 3. Transaction Flow

Without Transaction

```text
Debit Money

      ✓

Credit Money

      ❌

Database Inconsistent
```

---

With Transaction

```text
Start Transaction

↓

Debit Money

↓

Credit Money

↓

Commit
```

If any step fails

```text
Start Transaction

↓

Debit Money

↓

Credit Money ❌

↓

Rollback

↓

Database Restored
```

---

# 4. Commit

What is Commit?

Commit means:

> Permanently save all changes made during the transaction.

Example:

```java
connection.commit();
```

After commit,

changes are stored permanently.

They cannot be rolled back.

---

# 5. Rollback

Rollback means:

> Undo every operation performed in the current transaction.

Example

```java
connection.rollback();
```

Suppose:

```text
Debit ₹10,000

↓

Exception
```

Rollback restores

| Account | Balance |
| ------- | ------: |
| A       | ₹50,000 |
| B       | ₹20,000 |

As if nothing happened.

---

# 6. Internal Flow

```text
Application

      │

      ▼

Begin Transaction

      │

      ▼

SQL 1

      │

      ▼

SQL 2

      │

      ▼

SQL 3

      │

      ▼

Success?

 ┌──────────────┐
 │              │
Yes            No
 │              │
 ▼              ▼

Commit      Rollback
```

---

# 7. Where Are Transactions Used?

Transactions are used whenever multiple related operations must succeed or fail together.

Examples:

* Bank transfers
* Online shopping
* Ticket booking
* Inventory management
* Salary processing
* Student admission systems
* Payment gateways

---

# 8. Why Not Use Transactions Everywhere?

Transactions:

* Keep database locks longer.
* Reduce concurrency.
* Increase resource usage if they run for a long time.

So keep transactions:

* Small
* Fast
* Focused

---

# 9. Interview Questions

### Q1. What is a transaction?

A transaction is a group of one or more database operations executed as a single unit of work. Either all operations succeed or all are rolled back.

---

### Q2. Why do we need transactions?

To maintain data consistency and prevent partial updates when failures occur.

---

### Q3. What is Commit?

Commit permanently saves all changes made during a transaction.

---

### Q4. What is Rollback?

Rollback undoes all changes made during the current transaction.

---

### Q5. Give a real-world example.

A bank money transfer:

* Debit one account
* Credit another account

If one operation fails, both should be rolled back.

---

# Best Practices

* Keep transactions as short as possible.
* Avoid long-running business logic inside a transaction.
* Don't perform slow external calls (like REST APIs or email sending) inside a transaction unless necessary.
* Apply transactions at the **service layer**, where business operations are coordinated.

---

# Summary

```text
Without Transaction

SQL 1 ✓

SQL 2 ❌

↓

Partial Data

--------------------------------

With Transaction

Begin

↓

SQL 1

↓

SQL 2

↓

Success?

↓

Commit

OR

Rollback
```

---

# 📍 Where We Are

```text
Spring Transaction Management

✅ Why Transactions
✅ What is a Transaction?
✅ Commit
✅ Rollback

⏭️ Next:
ACID Properties
    ↓
Auto Commit
    ↓
JDBC Transactions
    ↓
PlatformTransactionManager
    ↓
@Transactional
```

---

## 🎯 Homework (Interview Scenario)

Consider this service:

```java
public void transferMoney() {

    debit(fromAccount, 10000);

    credit(toAccount, 10000);
}
```

Suppose `debit()` executes successfully, but `credit()` throws an exception.

**Question:**

1. What happens **without** a transaction?
2. What happens **with** a transaction?

If you can answer those two questions confidently, you've understood the core purpose of transactions. In the next chapter, we'll dive into **ACID Properties**, which are the foundation of every database transaction.
