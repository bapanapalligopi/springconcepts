Excellent. We are now entering one of the **most asked Spring interview topics**.

This chapter explains **when Spring commits and when Spring rolls back a transaction**.

---

# Chapter 8: Rollback Rules (`rollbackFor` & `noRollbackFor`)

As always, we'll learn:

> **Why → What → How → Internal Working → Real Examples → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Rollback Rules?

Suppose you're transferring **₹10,000**.

```java
@Transactional
public void transferMoney() {

    debit();

    credit();

}
```

What if:

* `credit()` throws an exception?

Should Spring:

* Commit?
* Rollback?

Spring needs rules to decide.

These are called **Rollback Rules**.

---

# 2. Default Rollback Behavior

By default, Spring **does not rollback for every exception**.

This surprises many developers.

Spring's default behavior is:

| Exception Type    | Rollback? |
| ----------------- | --------- |
| RuntimeException  | ✅ Yes     |
| Error             | ✅ Yes     |
| Checked Exception | ❌ No      |

This is one of the most common Spring interview questions.

---

# 3. Runtime Exception

Example:

```java
@Transactional
public void transferMoney() {

    debit();

    throw new RuntimeException("Payment Failed");

}
```

Flow:

```text
Begin Transaction

↓

Debit

↓

RuntimeException

↓

Rollback
```

Database returns to its previous state.

---

# 4. Checked Exception

Example:

```java
@Transactional
public void transferMoney() throws IOException {

    debit();

    throw new IOException("File Error");

}
```

Flow:

```text
Begin Transaction

↓

Debit

↓

IOException

↓

Commit ❗
```

Many beginners expect a rollback, but Spring commits by default because `IOException` is a **checked exception**.

---

# Why?

Historically, Spring assumes:

* **RuntimeException** → Programming or unexpected failure → Rollback.
* **Checked Exception** → Business or recoverable condition → Caller decides how to handle it.

---

# Exception Hierarchy

```text
Throwable
│
├── Error
│      └── OutOfMemoryError
│
└── Exception
       │
       ├── RuntimeException  ✅ Rollback
       │      ├── NullPointerException
       │      ├── ArithmeticException
       │      └── IllegalArgumentException
       │
       └── Checked Exception ❌ No Rollback
              ├── IOException
              ├── SQLException
              └── ClassNotFoundException
```

---

# 5. `rollbackFor`

Sometimes you want to rollback even for checked exceptions.

Example:

```java
@Transactional(
    rollbackFor = IOException.class
)
public void transferMoney() throws IOException {

    debit();

    throw new IOException();

}
```

Flow:

```text
IOException

↓

rollbackFor matches

↓

Rollback
```

Now the transaction is rolled back.

---

# Multiple Exceptions

```java
@Transactional(
    rollbackFor = {
        IOException.class,
        SQLException.class
    }
)
public void process() {

}
```

If either exception occurs, Spring rolls back.

---

# 6. `noRollbackFor`

Sometimes you want the opposite.

Suppose:

```java
RuntimeException
```

Normally causes rollback.

But you want to commit anyway.

```java
@Transactional(
    noRollbackFor = IllegalArgumentException.class
)
public void save() {

}
```

Now:

```text
IllegalArgumentException

↓

Commit
```

Even though it's a runtime exception.

---

# 7. Internal Working

Suppose:

```java
@Transactional(
    rollbackFor = Exception.class
)
```

Flow:

```text
Begin Transaction

↓

Execute Method

↓

Exception?

↓

Check rollbackFor

↓

Match?

│

├── Yes

│

▼

Rollback

│

└── No

▼

Commit
```

---

# 8. Real Banking Example

```java
@Transactional
public void transferMoney() {

    debit();

    credit();

    sendReceipt();

}
```

Scenario 1

```text
credit()

↓

RuntimeException

↓

Rollback
```

No money is transferred.

---

Scenario 2

```text
sendReceipt()

↓

IOException

↓

Commit (Default)
```

Money transfer succeeds, but sending the receipt fails.

Depending on business requirements, this may be acceptable.

---

Scenario 3

```java
@Transactional(
    rollbackFor = IOException.class
)
```

Now:

```text
sendReceipt()

↓

IOException

↓

Rollback
```

Money transfer is also undone.

---

# 9. Comparison

| Configuration | RuntimeException    | Checked Exception     |
| ------------- | ------------------- | --------------------- |
| Default       | Rollback            | Commit                |
| rollbackFor   | Rollback            | Rollback (if matched) |
| noRollbackFor | Commit (if matched) | Commit                |

---

# 10. Common Mistake

Many developers think:

```java
@Transactional
```

means:

> "Rollback for every exception."

This is **wrong**.

Default behavior:

```text
RuntimeException

↓

Rollback

----------------------

Checked Exception

↓

Commit
```

---

# Interview Questions

### Q1. What is the default rollback behavior of `@Transactional`?

Spring rolls back for:

* RuntimeException
* Error

It does **not** roll back for checked exceptions by default.

---

### Q2. How do you rollback for a checked exception?

```java
@Transactional(
    rollbackFor = IOException.class
)
```

---

### Q3. What does `noRollbackFor` do?

It tells Spring **not** to roll back for the specified exception, even if it would normally trigger a rollback.

---

### Q4. Difference between `rollbackFor` and `noRollbackFor`?

| rollbackFor                         | noRollbackFor                                |
| ----------------------------------- | -------------------------------------------- |
| Forces rollback                     | Prevents rollback                            |
| Usually used for checked exceptions | Usually used for specific runtime exceptions |

---

### Q5. Why doesn't Spring rollback for checked exceptions by default?

Because checked exceptions are traditionally considered recoverable or business-related conditions, while runtime exceptions usually indicate programming or system failures.

---

# Best Practices

* Keep the default behavior unless your business requirement says otherwise.
* Use `rollbackFor` only when checked exceptions should invalidate the transaction.
* Avoid `rollbackFor = Exception.class` unless you're certain every exception should roll back, because it may undo work for recoverable business conditions.
* Define custom business exceptions carefully (checked vs. unchecked) based on how you want transactions to behave.

---

# Summary

```text
@Transactional

↓

Exception?

│

├── RuntimeException

│      ↓

│   Rollback

│

├── Error

│      ↓

│   Rollback

│

└── Checked Exception

       ↓

    Commit

-------------------------

rollbackFor

↓

Force Rollback

-------------------------

noRollbackFor

↓

Force Commit
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
✅ Rollback Rules

⏭️ Next

readOnly ⭐⭐⭐⭐
        ↓
timeout
        ↓
Self Invocation Problem ⭐⭐⭐⭐⭐
        ↓
Complete Transaction Lifecycle
```

---

# 🎯 Interview Tip

A common interview question is:

> **"A `@Transactional` method throws an `IOException`. Will Spring roll back?"**

**Answer:**

**No, not by default.**

`IOException` is a **checked exception**, so Spring commits the transaction unless you configure:

```java
@Transactional(rollbackFor = IOException.class)
```

This is a small detail that interviewers often use to distinguish between developers who have used `@Transactional` and those who understand how it actually works.
