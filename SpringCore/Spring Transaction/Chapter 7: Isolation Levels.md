Excellent. Now we are learning one of the **most important database concepts** that every Java developer should know.

> **Transactions tell us *when* to commit.**
>
> **Isolation Levels tell us *how transactions interact with each other*.**

This topic is frequently asked in **Spring, Hibernate, SQL, and System Design interviews**.

---

# Chapter 7: Isolation Levels

As always, we'll learn:

> **Why → What → How → Problems → Isolation Levels → Internal Working → Spring → Interview Questions → Best Practices**

---

# 1. Why do we need Isolation Levels?

Suppose two users access the same bank account simultaneously.

Initial Balance:

```text
Account A = ₹50,000
```

### User A (Transaction T1)

Withdraws ₹10,000.

### User B (Transaction T2)

Checks the balance at the same time.

Question:

Should User B see:

```text
₹50,000

OR

₹40,000
```

What if T1 hasn't committed yet?

If User B sees ₹40,000 and later T1 rolls back, User B saw **incorrect data**.

This is why databases provide **Isolation Levels**.

---

# 2. What is Isolation?

### Definition

Isolation determines **how much one transaction can see the changes made by another transaction before it is committed.**

Think of it as:

> **"How isolated should one transaction be from another?"**

---

# 3. Problems Without Isolation

There are **three major concurrency problems**.

```text
1. Dirty Read

2. Non-Repeatable Read

3. Phantom Read
```

These problems determine which isolation level you need.

---

# 4. Dirty Read

## What is it?

A transaction reads **uncommitted data** from another transaction.

---

### Example

Initial Balance

```text
₹50,000
```

Transaction T1

```text
Begin

↓

Update Balance

₹40,000

↓

Not Committed
```

At the same time,

Transaction T2 reads

```text
₹40,000
```

Now,

T1 rolls back.

Actual Balance becomes

```text
₹50,000
```

T2 saw a value that **never actually existed**.

This is called a **Dirty Read**.

---

### Timeline

```text
T1                    T2

Begin

Update ₹40,000

                     Read ₹40,000 ❌

Rollback

Balance ₹50,000
```

---

# 5. Non-Repeatable Read

## What is it?

A transaction reads the **same row twice** but gets different values because another transaction committed an update in between.

---

### Example

T1

```text
Read Balance

₹50,000
```

Meanwhile,

T2

```text
Update Balance

₹60,000

Commit
```

Now T1 reads again

```text
₹60,000
```

Same row.

Different value.

This is called a **Non-Repeatable Read**.

---

### Timeline

```text
T1                     T2

Read ₹50,000

                      Update ₹60,000

                      Commit

Read ₹60,000 ❌
```

---

# 6. Phantom Read

## What is it?

A transaction executes the **same query twice** and gets different numbers of rows because another transaction inserted or deleted rows.

---

### Example

T1

```sql
SELECT *
FROM employee
WHERE salary > 50000;
```

Result

```text
5 Employees
```

Meanwhile,

T2

```sql
INSERT INTO employee
VALUES(...,70000);
```

Commit.

Now T1 executes the same query.

Result

```text
6 Employees
```

An extra row appeared.

This is called a **Phantom Read**.

---

### Timeline

```text
T1                         T2

SELECT

5 Rows

                         INSERT

                         Commit

SELECT

6 Rows ❌
```

---

# 7. Isolation Levels

Spring supports four standard isolation levels.

```text
READ_UNCOMMITTED

↓

READ_COMMITTED

↓

REPEATABLE_READ

↓

SERIALIZABLE
```

As isolation increases:

* Data consistency increases.
* Concurrency decreases.
* Performance usually decreases.

---

# 8. READ_UNCOMMITTED

Lowest isolation.

Allows:

* ✅ Dirty Read
* ✅ Non-Repeatable Read
* ✅ Phantom Read

```text
Performance ⭐⭐⭐⭐⭐

Consistency ⭐
```

Rarely used.

---

# 9. READ_COMMITTED

Prevents:

* ❌ Dirty Read

Still allows:

* ✅ Non-Repeatable Read
* ✅ Phantom Read

This is the default in databases like Oracle and PostgreSQL.

---

### Timeline

```text
T1

Update

(Not committed)

↓

T2 cannot read

↓

Commit

↓

T2 reads
```

---

# 10. REPEATABLE_READ

Prevents:

* ❌ Dirty Read
* ❌ Non-Repeatable Read

Still allows:

* ✅ Phantom Read

This is the default isolation level in MySQL InnoDB.

---

# 11. SERIALIZABLE

Highest isolation.

Prevents:

* ❌ Dirty Read
* ❌ Non-Repeatable Read
* ❌ Phantom Read

Transactions execute as if they run one after another.

```text
T1

↓

Finish

↓

T2 Starts
```

Most consistent.

Slowest.

---

# 12. Comparison Table

| Isolation Level  | Dirty Read | Non-Repeatable Read | Phantom Read |
| ---------------- | ---------- | ------------------- | ------------ |
| READ_UNCOMMITTED | ✅          | ✅                   | ✅            |
| READ_COMMITTED   | ❌          | ✅                   | ✅            |
| REPEATABLE_READ  | ❌          | ❌                   | ✅            |
| SERIALIZABLE     | ❌          | ❌                   | ❌            |

---

# 13. Using Isolation in Spring

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void transferMoney(){

}
```

Other options:

```java
Isolation.READ_UNCOMMITTED

Isolation.READ_COMMITTED

Isolation.REPEATABLE_READ

Isolation.SERIALIZABLE

Isolation.DEFAULT
```

`Isolation.DEFAULT` means:

Use the database's default isolation level.

---

# 14. Internal Working

```text
@Transactional

↓

Spring Proxy

↓

PlatformTransactionManager

↓

Set Isolation Level

↓

Begin Transaction

↓

Execute SQL

↓

Commit
```

The actual isolation behavior is enforced by the **database**, not by Spring.

---

# 15. Real World Usage

| Scenario               | Recommended Isolation                                     |
| ---------------------- | --------------------------------------------------------- |
| Employee CRUD          | READ_COMMITTED                                            |
| Banking                | REPEATABLE_READ or SERIALIZABLE (depends on requirements) |
| Reports                | READ_COMMITTED                                            |
| Financial Year Closing | SERIALIZABLE                                              |

---

# Interview Questions

### Q1. What is an isolation level?

An isolation level defines how one transaction is isolated from the changes made by other concurrent transactions.

---

### Q2. Which isolation level prevents Dirty Reads?

```text
READ_COMMITTED
```

and all higher levels.

---

### Q3. Which isolation level prevents all three concurrency problems?

```text
SERIALIZABLE
```

---

### Q4. What is the default isolation level in Spring?

Spring's default is:

```text
Isolation.DEFAULT
```

This means Spring uses **the default isolation level of the underlying database**.

Examples:

* MySQL (InnoDB): `REPEATABLE_READ`
* PostgreSQL: `READ_COMMITTED`
* Oracle: `READ_COMMITTED`

---

### Q5. Which is the fastest isolation level?

```text
READ_UNCOMMITTED
```

It provides the highest concurrency but the weakest consistency.

---

# Best Practices

* Use the **database default** (`Isolation.DEFAULT`) unless you have a specific business requirement.
* Avoid `SERIALIZABLE` unless strict consistency is essential, as it can reduce concurrency.
* Understand your database's default isolation level before changing it.
* Choose higher isolation levels only when needed because they can increase locking and reduce throughput.

---

# Summary

```text
READ_UNCOMMITTED

Allows Everything

-------------------------

READ_COMMITTED

✔ Prevents Dirty Read

-------------------------

REPEATABLE_READ

✔ Prevents Dirty Read

✔ Prevents Non-Repeatable Read

-------------------------

SERIALIZABLE

✔ Prevents Dirty Read

✔ Prevents Non-Repeatable Read

✔ Prevents Phantom Read
```

---

# 📍 Where We Are

```text
Spring Transaction Management

✅ Why Transactions
✅ ACID Properties
✅ JDBC Transactions
✅ PlatformTransactionManager
✅ @Transactional
✅ Propagation
✅ Isolation Levels

⏭️ Next

Rollback Rules ⭐⭐⭐⭐⭐
        ↓
readOnly
        ↓
timeout
        ↓
Self Invocation Problem
        ↓
Complete Transaction Flow
```

---

## 🎯 Interview Tip

A very common interview question is:

> **"What is the difference between `READ_COMMITTED` and `REPEATABLE_READ`?"**

A good answer is:

* **READ_COMMITTED** prevents **Dirty Reads**, but the same row can still change between two reads in the same transaction (Non-Repeatable Read is possible).
* **REPEATABLE_READ** prevents both **Dirty Reads** and **Non-Repeatable Reads**, ensuring repeated reads of the same row return the same value during a transaction. Phantom Reads may still be possible depending on the database implementation.
