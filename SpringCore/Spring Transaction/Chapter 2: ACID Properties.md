Excellent. Now we move to the **foundation of every database transaction**.

> **If you understand ACID, you will understand why transactions exist.**

This is one of the **most frequently asked interview topics**.

---

# Chapter 2: ACID Properties

As always, we'll learn:

> **Why → What → How → Where → Internal Working → Real Examples → Interview Questions → Best Practices**

---

# 1. Why do we need ACID?

Suppose you're transferring **₹10,000** from Account A to Account B.

The application executes:

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

Question:

How can the database guarantee that:

* Money is never lost?
* Data is always correct?
* Two users don't corrupt each other's data?
* Data survives a server crash?

The answer is:

> **ACID Properties**

These are four rules every transaction follows.

---

# What is ACID?

```text
A → Atomicity

C → Consistency

I → Isolation

D → Durability
```

Think of ACID as the **quality rules** for transactions.

---

# 2. Atomicity

## Why?

Imagine a bank transfer.

Steps:

```text
Debit ₹10,000 from Account A

↓

Credit ₹10,000 to Account B
```

Suppose:

Debit succeeds.

Credit fails.

What should happen?

Should only one operation remain?

No.

Both should be undone.

---

## What is Atomicity?

Atomicity means:

> **Either every operation succeeds or every operation fails.**

Nothing in between.

---

### Example

Without Atomicity

```text
Debit ₹10,000        ✓

Credit ₹10,000       ❌
```

Result

| Account | Balance |
| ------- | ------: |
| A       | ₹40,000 |
| B       | ₹20,000 |

₹10,000 disappeared.

---

With Atomicity

```text
Debit ₹10,000

↓

Credit ₹10,000

↓

Failure

↓

Rollback
```

Result

| Account | Balance |
| ------- | ------: |
| A       | ₹50,000 |
| B       | ₹20,000 |

Database remains correct.

---

## Internal Flow

```text
Begin Transaction

↓

Debit

↓

Credit

↓

Exception?

     │

Yes

↓

Rollback

──────────────

No

↓

Commit
```

---

# 3. Consistency

## Why?

Suppose a rule says:

```text
Account Balance

Cannot be Negative
```

Before transaction

```text
Account A

₹50,000
```

Transaction tries

```text
Withdraw

₹60,000
```

Balance becomes

```text
-₹10,000
```

This violates the business rule.

---

## What is Consistency?

Consistency means:

> **A transaction moves the database from one valid state to another valid state.**

Rules (constraints) must never be broken.

Examples:

* Primary Key
* Foreign Key
* Unique Key
* Check Constraints
* Business Rules

---

### Example

Before

```text
Stock = 10
```

Sell

```text
15 Items
```

Stock becomes

```text
-5
```

Not allowed.

Database rejects it.

Consistency is preserved.

---

# 4. Isolation

## Why?

Suppose two users transfer money simultaneously.

### User A

Withdraw ₹5,000

### User B

Withdraw ₹10,000

Both read:

```text
Balance = ₹20,000
```

Without isolation

Both update independently.

Final balance becomes incorrect.

---

## What is Isolation?

Isolation means:

> **Transactions should not interfere with each other while executing.**

Each transaction should behave as if it is running alone.

---

### Example

Without Isolation

```text
Transaction A

Reads ₹20,000

----------------

Transaction B

Reads ₹20,000

----------------

Both Update

↓

Wrong Balance
```

---

With Isolation

```text
Transaction A

Locks Data

↓

Commit

↓

Transaction B Starts
```

Correct result.

---

## Internal View

```text
Transaction A

        │

        ▼

Database Lock

        │

        ▼

Transaction B Waits

        │

        ▼

Commit

        │

        ▼

Transaction B Continues
```

---

# 5. Durability

## Why?

Suppose:

Transaction completed successfully.

Database says:

```text
Commit Successful
```

Immediately after that,

the server crashes.

Question:

Should committed data disappear?

No.

---

## What is Durability?

Durability means:

> **Once a transaction is committed, its changes are permanent, even after a crash.**

---

### Example

```text
Transfer ₹10,000

↓

Commit

↓

Power Failure

↓

Restart Database
```

Money is still transferred.

---

## How?

The database writes committed changes to persistent storage (such as transaction logs and disk).

---

# Complete ACID Flow

```text
Start Transaction

↓

Atomicity
(All or Nothing)

↓

Consistency
(Valid Data)

↓

Isolation
(No Interference)

↓

Commit

↓

Durability
(Permanent Storage)
```

---

# Easy Way to Remember

| Letter | Meaning     | Simple Memory                         |
| ------ | ----------- | ------------------------------------- |
| A      | Atomicity   | All or Nothing                        |
| C      | Consistency | Rules are never broken                |
| I      | Isolation   | Transactions don't disturb each other |
| D      | Durability  | Committed data never disappears       |

---

# Real Banking Example

Transfer ₹10,000

```text
Atomicity

Debit + Credit

OR

Rollback

-------------------

Consistency

Money Total remains correct

-------------------

Isolation

Other users cannot see half-completed transfer

-------------------

Durability

After Commit

Money remains transferred even if power fails
```

---

# Interview Questions

## Q1. What is ACID?

ACID stands for:

* Atomicity
* Consistency
* Isolation
* Durability

These properties ensure reliable and correct transaction processing.

---

## Q2. Which ACID property ensures "all or nothing"?

**Atomicity**

---

## Q3. Which ACID property prevents partial updates?

**Atomicity**

---

## Q4. Which ACID property ensures business rules are maintained?

**Consistency**

---

## Q5. Which ACID property prevents two transactions from interfering?

**Isolation**

---

## Q6. Which ACID property ensures data survives a crash?

**Durability**

---

## Q7. Can Spring implement ACID by itself?

**No.**

Spring manages transactions, but the actual ACID guarantees are provided by the underlying database (such as MySQL, PostgreSQL, or Oracle). Spring coordinates transaction boundaries; the database enforces ACID.

---

# Best Practices

* Keep transactions short to reduce locking.
* Commit only after all business validations succeed.
* Don't perform slow network calls inside transactions.
* Understand that ACID depends on both the database and how transactions are configured.

---

# Summary

```text
ACID

A → Atomicity
    All operations succeed or all rollback.

C → Consistency
    Database remains in a valid state.

I → Isolation
    Transactions do not interfere.

D → Durability
    Committed data is permanent.
```

---

# 📍 Where We Are

```text
Spring Transaction Management

✅ Why Transactions
✅ What is Transaction
✅ Commit
✅ Rollback
✅ ACID Properties

⏭️ Next

JDBC Transactions
    ↓
Auto Commit
    ↓
commit()
    ↓
rollback()
    ↓
PlatformTransactionManager
    ↓
@Transactional
```

---

## 🎯 Interview Tip

A very common interview question is:

> **"Does `@Transactional` itself guarantee ACID?"**

A strong answer is:

> **No.** `@Transactional` tells Spring where a transaction begins and ends. The **database** is responsible for enforcing ACID properties, while Spring coordinates transaction management through a transaction manager.
