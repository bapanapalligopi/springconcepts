Excellent. Now we are entering the **most confusing topic in Spring Transactions**, but once you understand the logic, it's actually straightforward.

> **Propagation decides how a transaction should behave when one `@Transactional` method calls another `@Transactional` method.**

---

# Chapter 6: Transaction Propagation

As always, we'll learn using:

> **Why → What → How → Where → Internal Working → Real Examples → Interview Questions → Best Practices**

---

# 1. Why do we need Propagation?

Suppose we have two services.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {

        paymentService.makePayment();

    }

}
```

```java
@Service
public class PaymentService {

    @Transactional
    public void makePayment() {

    }

}
```

Question:

When `placeOrder()` calls `makePayment()`:

* Should both use the **same transaction**?
* Should a **new transaction** be created?
* What if there is **no transaction**?

Spring needs a rule.

That rule is called **Propagation**.

---

# 2. What is Transaction Propagation?

### Definition

Transaction propagation defines **how a transactional method should behave when it is called by another transactional method.**

Think of it as:

> **"If a transaction already exists, what should I do?"**

---

# Example

```text
OrderService.placeOrder()

↓

PaymentService.makePayment()
```

Should PaymentService:

```text
Join Existing Transaction?

OR

Create New Transaction?

OR

Run Without Transaction?
```

Propagation answers these questions.

---

# 3. Types of Propagation

Spring provides seven propagation types.

```text
Propagation

├── REQUIRED ⭐⭐⭐⭐⭐
├── REQUIRES_NEW ⭐⭐⭐⭐⭐
├── SUPPORTS ⭐⭐⭐
├── MANDATORY ⭐⭐
├── NOT_SUPPORTED ⭐⭐
├── NEVER ⭐⭐
└── NESTED ⭐⭐⭐
```

For a **2-year developer**, focus mainly on:

* REQUIRED
* REQUIRES_NEW
* SUPPORTS

Know the remaining four conceptually.

---

# 4. Propagation.REQUIRED (Default)

## What?

```java
@Transactional(propagation = Propagation.REQUIRED)
```

This is the **default** propagation.

Rule:

```text
Transaction Exists?

YES → Join It

NO → Create New One
```

---

## Example

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {

        paymentService.makePayment();

    }

}
```

```java
@Service
public class PaymentService {

    @Transactional
    public void makePayment() {

    }

}
```

Flow:

```text
placeOrder()

↓

Transaction T1

↓

makePayment()

↓

Join T1
```

Only **one transaction** exists.

---

## Internal Working

```text
Client

↓

Proxy

↓

Begin Transaction T1

↓

OrderService

↓

PaymentService

↓

Same Transaction T1

↓

Commit
```

---

## Real Example

```text
Place Order

↓

Reduce Stock

↓

Make Payment

↓

Save Order

↓

Commit
```

Everything succeeds or everything rolls back.

This is why `REQUIRED` is the default.

---

# 5. Propagation.REQUIRES_NEW

## What?

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

Rule:

```text
Always Create New Transaction
```

Even if another transaction already exists.

---

## Example

```java
@Transactional
public void placeOrder(){

    paymentService.makePayment();

}
```

```java
@Transactional(
propagation = Propagation.REQUIRES_NEW)
public void makePayment(){

}
```

Flow

```text
Transaction T1

↓

Suspend T1

↓

Create T2

↓

Execute Payment

↓

Commit T2

↓

Resume T1
```

Notice:

Two separate transactions.

---

## Internal Flow

```text
Client

↓

T1 Started

↓

Order Service

↓

Suspend T1

↓

Start T2

↓

Payment Service

↓

Commit T2

↓

Resume T1

↓

Commit T1
```

---

## Real Example

Imagine an audit log.

```text
Place Order

↓

Save Order

↓

Write Audit Log

↓

Commit
```

Even if the order later fails,

the audit log should still be saved.

Use:

```text
REQUIRES_NEW
```

for the audit operation.

---

# 6. Propagation.SUPPORTS

Rule:

```text
Transaction Exists?

YES → Join It

NO → Execute Without Transaction
```

Unlike REQUIRED,

it **does not create a new transaction**.

---

Example

```java
@Transactional(propagation = Propagation.SUPPORTS)
public List<Employee> getEmployees(){

}
```

If called from:

```text
Transaction Exists

↓

Join
```

If called directly:

```text
No Transaction

↓

Execute Normally
```

No transaction is created.

Usually used for **read operations**.

---

# 7. Comparison

| Propagation  | Existing Transaction            | No Transaction |
| ------------ | ------------------------------- | -------------- |
| REQUIRED     | Join                            | Create         |
| REQUIRES_NEW | Suspend existing and create new | Create         |
| SUPPORTS     | Join                            | No transaction |

These three cover most real-world use cases.

---

# 8. Remaining Propagation Types

## MANDATORY

Rule:

```text
Transaction Exists?

YES → Join

NO → Throw Exception
```

Use when a method **must** always run inside a transaction.

---

## NOT_SUPPORTED

Rule:

```text
Transaction Exists?

↓

Suspend It

↓

Execute Without Transaction
```

Useful for long-running, non-database work that shouldn't hold database locks.

---

## NEVER

Rule:

```text
Transaction Exists?

↓

Throw Exception
```

Ensures the method is never called within a transaction.

---

## NESTED

Rule:

```text
Outer Transaction

↓

Nested Transaction (Savepoint)

↓

Failure?

↓

Rollback to Savepoint
```

Works using database savepoints and depends on the transaction manager and database support.

---

# Visual Summary

```text
REQUIRED

T1
 └── Service A
      └── Service B (Same T1)

-------------------------

REQUIRES_NEW

T1
 └── Service A

Suspend T1

T2
 └── Service B

Resume T1

-------------------------

SUPPORTS

T1 Exists?
     │
 Yes │ No
     │
 Join│ No Transaction
```

---

# Real Banking Example

```text
Transfer Money

↓

Debit Account

↓

Credit Account

↓

Audit Log
```

Recommended:

```text
Transfer

↓

REQUIRED

↓

Debit

↓

Credit

↓

Audit

↓

REQUIRES_NEW
```

Even if the transfer fails,

the audit record can still be committed independently.

---

# Interview Questions

### Q1. What is propagation?

Propagation defines how a transactional method behaves when another transactional method calls it.

---

### Q2. What is the default propagation?

```text
Propagation.REQUIRED
```

---

### Q3. Difference between REQUIRED and REQUIRES_NEW?

| REQUIRED                              | REQUIRES_NEW                                 |
| ------------------------------------- | -------------------------------------------- |
| Joins existing transaction if present | Always starts a new transaction              |
| One transaction                       | Two independent transactions                 |
| Default choice                        | Used for independent work like audit logging |

---

### Q4. When do we use SUPPORTS?

For methods that can run either inside a transaction or without one, commonly read-only operations.

---

### Q5. When do we use REQUIRES_NEW?

When the inner operation must commit or roll back independently of the outer transaction.

---

# Best Practices

* ✅ Use **REQUIRED** for most business operations.
* ✅ Use **REQUIRES_NEW** sparingly for independent tasks (audit logs, notifications persisted to DB).
* ✅ Use **SUPPORTS** for read-only methods that don't require a transaction.
* ❌ Avoid changing propagation unless you have a clear business requirement.

---

# Summary

```text
REQUIRED

Existing Transaction?
      │
 Yes  │  No
      │
 Join │ Create New

---------------------------------

REQUIRES_NEW

Always Create New

Suspend Existing

Resume Later

---------------------------------

SUPPORTS

Existing Transaction?
      │
 Yes  │  No
      │
 Join │ Execute Without Transaction
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

⏭️ Next

Isolation Levels ⭐⭐⭐⭐⭐
        ↓
Rollback Rules
        ↓
ReadOnly
        ↓
Timeout
        ↓
Self Invocation Problem
```

---

## 🎯 Interview Tip

A common interview scenario is:

> **"OrderService is `@Transactional(REQUIRED)` and calls AuditService with `@Transactional(REQUIRES_NEW)`. If the order transaction rolls back, what happens to the audit record?"**

**Answer:**

The audit operation runs in its **own transaction (T2)**. If T2 commits successfully, the audit record remains committed **even if the outer order transaction (T1) later rolls back**. This independence is the primary purpose of `REQUIRES_NEW`.
