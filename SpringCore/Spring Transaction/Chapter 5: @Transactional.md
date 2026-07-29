Excellent. Now we are learning the **most important annotation in Spring**.

If you understand this chapter well, you'll understand **80% of Spring Transaction Management**.

---

# Chapter 5: `@Transactional`

As always,

> **Why → What → How → Where → Internal Working → Attributes → Real Examples → Interview Questions → Best Practices**

---

# 1. Why do we need `@Transactional`?

Let's revisit plain JDBC.

```java
Connection con = null;

try {

    con = dataSource.getConnection();

    con.setAutoCommit(false);

    debit();

    credit();

    con.commit();

} catch (Exception e) {

    con.rollback();

}
```

Problems:

* Boilerplate code
* Easy to forget `commit()`
* Easy to forget `rollback()`
* Hard to maintain
* Transaction logic mixed with business logic

Spring introduced:

```java
@Transactional
```

Now you only write:

```java
@Transactional
public void transferMoney() {

    debit();

    credit();

}
```

Everything else is handled by Spring.

---

# 2. What is `@Transactional`?

### Definition

`@Transactional` is a Spring annotation that tells Spring:

> "Execute this method inside a database transaction."

Spring automatically:

* Begins the transaction
* Executes the method
* Commits if successful
* Rolls back if required

---

# 3. Where can we use it?

## Method Level (Most Common)

```java
@Service
public class EmployeeService {

    @Transactional
    public void saveEmployee() {

    }

}
```

Only this method runs inside a transaction.

---

## Class Level

```java
@Service
@Transactional
public class EmployeeService {

    public void save(){}

    public void update(){}

    public void delete(){}

}
```

Every public method becomes transactional.

---

# Which is preferred?

Usually:

```java
@Transactional
public void saveEmployee(){}
```

because different methods often need different transaction settings.

---

# 4. Internal Working

Suppose

```java
@Transactional
public void transferMoney(){

    debit();

    credit();

}
```

Internally Spring does this:

```text
Client

↓

Transaction Proxy (AOP)

↓

PlatformTransactionManager

↓

Begin Transaction

↓

transferMoney()

↓

Success?

│

├── Yes

│

▼

Commit

│

└── No

▼

Rollback
```

Notice:

Your method never calls

```java
commit();
```

or

```java
rollback();
```

Spring does.

---

# 5. Step-by-Step Flow

Imagine the client calls:

```java
bankService.transferMoney();
```

Flow:

```text
Client

↓

Proxy

↓

Begin Transaction

↓

Actual Service Method

↓

Return?

↓

Commit

↓

Return Result
```

If an exception occurs:

```text
Client

↓

Proxy

↓

Begin Transaction

↓

Service Method

↓

Exception

↓

Rollback

↓

Throw Exception
```

---

# 6. Does `@Transactional` create a transaction?

Not exactly.

It tells Spring:

> "Use PlatformTransactionManager to manage a transaction."

So internally:

```text
@Transactional

↓

Transaction Proxy

↓

PlatformTransactionManager

↓

Database Transaction
```

---

# 7. Important Rule

Spring Transaction works only through **Proxy**.

Remember AOP?

```text
Client

↓

Proxy

↓

Service
```

If the call bypasses the proxy,

there is **no transaction**.

This leads to one of Spring's most famous interview questions:

**Self Invocation Problem**

We'll study it later.

---

# 8. Example

```java
@Service
public class EmployeeService {

    @Transactional
    public void addEmployee() {

        repository.save(emp);

        repository.updateSalary(emp);

    }

}
```

Suppose:

```text
save()

✓

updateSalary()

❌
```

Spring automatically:

```text
Rollback

↓

save() undone
```

Database remains consistent.

---

# 9. Internal Comparison

Without Spring

```text
Application

↓

Connection

↓

setAutoCommit(false)

↓

Business Logic

↓

Commit

↓

Rollback
```

---

With Spring

```text
Application

↓

@Transactional

↓

Proxy

↓

PlatformTransactionManager

↓

Business Logic

↓

Commit / Rollback
```

Much cleaner.

---

# 10. Important Attributes

`@Transactional` has several attributes.

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    readOnly = false,
    timeout = 30,
    rollbackFor = Exception.class
)
```

We'll study each one in detail.

| Attribute     | Purpose                             |
| ------------- | ----------------------------------- |
| propagation   | How transactions interact           |
| isolation     | Concurrency behavior                |
| readOnly      | Optimize read-only operations       |
| timeout       | Maximum execution time              |
| rollbackFor   | Exceptions that trigger rollback    |
| noRollbackFor | Exceptions that should not rollback |

---

# 11. Default Behaviour

Suppose:

```java
@Transactional
public void saveEmployee(){

}
```

Default settings:

```text
Propagation = REQUIRED

Isolation = Database Default

ReadOnly = false

Timeout = Default

Rollback = RuntimeException & Error
```

Remember this for interviews.

---

# 12. Common Mistake

Many beginners think:

```java
@Transactional
private void save(){ }
```

works.

It doesn't.

Spring's proxy-based transaction management applies to **public methods** that are invoked through the proxy.

---

# 13. Where Should We Use It?

Recommended:

```text
Controller

↓

Service   ✅

↓

Repository
```

Transaction boundary should be at the **Service Layer**.

Reason:

One business operation may involve multiple repository calls.

Example:

```text
Transfer Money

↓

Debit Repository

↓

Credit Repository

↓

Notification Repository
```

All belong to one transaction.

---

# Interview Questions

## Q1. What is `@Transactional`?

It is a Spring annotation that declaratively manages transactions by automatically beginning, committing, or rolling back a transaction around a method.

---

## Q2. Which layer should use `@Transactional`?

Service Layer.

---

## Q3. Does `@Transactional` work without AOP?

No.

Spring uses AOP proxies to implement declarative transaction management.

---

## Q4. Who actually starts the transaction?

```text
PlatformTransactionManager
```

---

## Q5. Can we use `@Transactional` on a class?

Yes.

All eligible public methods of that class become transactional by default.

---

## Q6. Does `@Transactional` work on private methods?

No.

Private methods are not intercepted by Spring's proxy-based transaction management.

---

# Best Practices

* Place `@Transactional` on **service methods**, not controllers.
* Keep transactions short and focused.
* Avoid long-running operations (such as external API calls) inside transactions when possible.
* Use method-level annotations unless the same behavior is needed for all methods in a class.

---

# Summary

```text
@Transactional

↓

Spring AOP Proxy

↓

PlatformTransactionManager

↓

Begin Transaction

↓

Execute Business Method

↓

Success?

│

├── Yes

│

▼

Commit

│

└── No

▼

Rollback
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

⏭️ Next

Propagation ⭐⭐⭐⭐⭐
        ↓
Isolation Levels
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

# 🎯 Interview Tip

One of the most common interview questions is:

> **"What happens internally when you call a `@Transactional` method?"**

A complete answer is:

1. The client calls the Spring-managed bean.
2. The **AOP proxy** intercepts the call.
3. The proxy asks the **PlatformTransactionManager** to start a transaction.
4. The target method executes.
5. If the method completes successfully, the transaction is committed.
6. If a rollback-triggering exception occurs, the transaction is rolled back.
7. The proxy returns the result (or rethrows the exception).

---

## 🚀 Next Chapter: Transaction Propagation (Very Important)

This is one of the **hardest and most frequently asked** Spring interview topics.

We'll cover:

* Why propagation is needed
* Every propagation type (`REQUIRED`, `REQUIRES_NEW`, `SUPPORTS`, `MANDATORY`, `NOT_SUPPORTED`, `NEVER`, `NESTED`)
* Real banking and order-processing examples
* Internal working diagrams
* Common interview scenarios
* Best practices for each propagation mode
