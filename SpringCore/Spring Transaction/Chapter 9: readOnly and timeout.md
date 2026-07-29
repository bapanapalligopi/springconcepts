Excellent. We are now learning two attributes that are widely used in production applications:

* **`readOnly`**
* **`timeout`**

Although they look simple, interviewers often ask them because they relate to **performance optimization**.

---

# Chapter 9: `readOnly` and `timeout`

As always, we'll follow:

> **Why → What → How → Internal Working → Real Examples → Interview Questions → Best Practices**

---

# Part 1: `readOnly`

---

# 1. Why do we need `readOnly`?

Suppose your application has a method that only fetches employee details.

```java
public List<Employee> getEmployees() {

    return employeeRepository.findAll();

}
```

Question:

Does this method modify the database?

**No.**

Then why start a full read-write transaction?

It adds unnecessary overhead.

Spring provides:

```java
@Transactional(readOnly = true)
```

---

# 2. What is `readOnly`?

Definition:

> **`readOnly=true` tells Spring that this transaction is intended only for reading data, not modifying it.**

It is a **hint** to Spring and the underlying persistence technology (such as Hibernate). The exact optimization depends on the transaction manager and database.

---

# Example

```java
@Transactional(readOnly = true)
public List<Employee> getEmployees() {

    return repository.findAll();

}
```

This indicates that the method is performing only read operations.

---

# 3. Internal Working

```text
Client

↓

Transaction Proxy

↓

Begin Transaction

↓

Read Only = true

↓

Execute SELECT

↓

Commit
```

No insert/update/delete is expected.

---

# 4. What optimizations can happen?

Depending on the persistence provider:

* Hibernate may reduce dirty checking work.
* Some databases may optimize the transaction.
* Spring can communicate that the transaction is read-only to the transaction manager.

**Important:** It is not a guarantee that writes are impossible. The behavior depends on the underlying technology.

---

# 5. Real Example

```java
@Service
public class EmployeeService {

    @Transactional(readOnly = true)
    public Employee getEmployee(int id){

        return repository.findById(id);

    }

}
```

Perfect use case.

---

# 6. Wrong Usage

```java
@Transactional(readOnly = true)
public void saveEmployee(){

    repository.save(emp);

}
```

This is incorrect.

Some databases may still allow it, while some providers may reject it or behave unexpectedly. You should never rely on writes inside a read-only transaction.

---

# Best Practice

Use `readOnly=true` only for:

* SELECT
* Search
* Reports
* Dashboard
* Analytics
* View operations

Avoid it for:

* INSERT
* UPDATE
* DELETE

---

# Part 2: `timeout`

---

# 1. Why do we need `timeout`?

Imagine:

```java
@Transactional
public void generateReport(){

    // Takes 15 minutes

}
```

During this time:

* Database locks may be held.
* Other users may wait.
* Performance suffers.

We need a way to stop transactions that run too long.

---

# 2. What is `timeout`?

Definition:

> **`timeout` specifies the maximum number of seconds a transaction is allowed to run before Spring marks it for rollback.**

Example:

```java
@Transactional(timeout = 30)
public void processSalary(){

}
```

Meaning:

Maximum execution time:

```text
30 Seconds
```

---

# 3. Internal Working

```text
Begin Transaction

↓

Start Timer

↓

Execute SQL

↓

Finished Within Timeout?

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

# 4. Real Example

Suppose:

```java
@Transactional(timeout = 5)
public void generateReport(){

    Thread.sleep(10000);

}
```

Execution time:

```text
10 Seconds
```

Timeout:

```text
5 Seconds
```

Result:

```text
Rollback
```

Spring throws a transaction timeout exception (the exact exception depends on the transaction manager).

---

# 5. Combining Attributes

```java
@Transactional(
    readOnly = true,
    timeout = 20
)
public List<Employee> getEmployees(){

}
```

Meaning:

* Read-only transaction
* Maximum execution time: 20 seconds

---

# Real Production Example

```java
@Service
public class EmployeeService{

    @Transactional(
        readOnly = true,
        timeout = 10
    )
    public List<Employee> searchEmployee(){

        return repository.findAll();

    }

}
```

Advantages:

* Optimized for reading.
* Prevents long-running transactions.

---

# Comparison

| Attribute       | Purpose                                              |
| --------------- | ---------------------------------------------------- |
| `readOnly=true` | Indicates the transaction is intended only for reads |
| `timeout=30`    | Limits transaction execution time to 30 seconds      |

---

# Common Mistakes

### Mistake 1

```java
@Transactional(readOnly = true)
public void saveEmployee(){

    repository.save(emp);

}
```

❌ Wrong.

---

### Mistake 2

```java
@Transactional(timeout = 2)
```

When the method normally needs:

```text
10 Seconds
```

The transaction will repeatedly time out.

---

### Mistake 3

Not using `readOnly` for large reporting queries.

This may miss out on provider-specific optimizations.

---

# Interview Questions

## Q1. What is `readOnly=true`?

It indicates that the transaction is intended only for reading data and allows Spring and the persistence provider to apply read-only optimizations where supported.

---

## Q2. Does `readOnly=true` completely prevent writes?

**Not always.**

It is primarily a hint. Whether writes are blocked depends on the transaction manager, ORM (e.g., Hibernate), and the database. You should not rely on it as a security mechanism.

---

## Q3. What is `timeout`?

It specifies the maximum duration (in seconds) that a transaction may execute before it is rolled back.

---

## Q4. Default timeout?

If you don't specify one:

```java
@Transactional
```

Spring uses the **default timeout of the underlying transaction manager**, if one is configured. Otherwise, there is typically **no explicit timeout** imposed by Spring.

---

## Q5. Can we combine attributes?

Yes.

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    timeout = 30,
    readOnly = true
)
```

---

# Best Practices

✅ Use `readOnly=true` for service methods that only read data.

✅ Configure reasonable timeouts for long-running business operations.

✅ Keep transactions as short as possible.

❌ Don't perform external API calls inside long-running transactions if you can avoid it.

❌ Don't use `readOnly=true` for methods that modify data.

---

# Summary

```text
@Transactional

↓

readOnly = true

↓

Read Operations

↓

Possible Optimizations

--------------------------------

@Transactional

↓

timeout = 30

↓

Transaction must finish

Within 30 Seconds

↓

Else Rollback
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
✅ readOnly
✅ timeout

⏭️ Next (Most Important)

⭐⭐⭐⭐⭐ Self Invocation Problem
        ↓
Why @Transactional Sometimes Doesn't Work
        ↓
Spring Proxy Limitation
        ↓
Real Interview Scenarios
```

---

# 🎯 Interview Tip

A very common interview question is:

> **"I added `@Transactional` to my method, but it didn't start a transaction. Why?"**

In many cases, the answer is **self-invocation**—a method inside the same class calls another `@Transactional` method directly, bypassing the Spring proxy. Because the proxy is skipped, Spring never gets the chance to start the transaction.

That is exactly what we'll cover in the next chapter, and it's one of the most valuable Spring interview topics.
