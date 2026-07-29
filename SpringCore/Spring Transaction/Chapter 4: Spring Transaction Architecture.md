Excellent. Now we are starting the **heart of Spring Transaction Management**.

Everything you've learned so far comes together here:

* ✅ Spring Core (IoC, DI)
* ✅ Spring AOP (Proxy)
* ✅ Spring JDBC (JdbcTemplate)
* ✅ JDBC Transactions

Now we'll see **how Spring combines all of them**.

---

# Chapter 4: Spring Transaction Architecture

As always,

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# 1. Why did Spring introduce Transaction Management?

Let's recall plain JDBC.

```java
Connection con = null;

try{

    con = dataSource.getConnection();

    con.setAutoCommit(false);

    // SQL 1

    // SQL 2

    con.commit();

}catch(Exception e){

    con.rollback();

}
finally{

    con.close();

}
```

Imagine writing this code in:

* EmployeeService
* StudentService
* OrderService
* PaymentService
* ProductService

Every service repeats the same code.

Problems:

* Boilerplate code
* Easy to forget `commit()`
* Easy to forget `rollback()`
* Hard to maintain
* Business logic mixed with transaction logic

Spring asked:

> **Can transaction management be separated from business logic?**

Answer:

**Yes, using AOP.**

---

# 2. What is Spring Transaction Management?

Spring moves transaction handling out of your business code.

Instead of writing:

```java
con.setAutoCommit(false);

...

con.commit();

...

con.rollback();
```

You write only:

```java
@Transactional
public void transferMoney(){

    debit();

    credit();

}
```

Spring automatically:

* Starts the transaction
* Commits if successful
* Rolls back if an exception occurs
* Closes the connection

---

# 3. Architecture

```text
               Spring Container
                      │
                      │
                      ▼
          PlatformTransactionManager
                      │
                      │
                      ▼
             Transaction Proxy (AOP)
                      │
                      │
                      ▼
              Service Bean
                      │
                      ▼
              JdbcTemplate
                      │
                      ▼
                 Database
```

Everything revolves around the **Transaction Manager**.

---

# 4. What is PlatformTransactionManager?

It is the **central interface** responsible for managing transactions.

Definition:

> `PlatformTransactionManager` is Spring's abstraction for transaction management.

Think of it as the **manager** that controls:

* Begin Transaction
* Commit
* Rollback

---

## Interface

```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(
            TransactionDefinition definition);

    void commit(TransactionStatus status);

    void rollback(TransactionStatus status);

}
```

Notice

These methods look similar to JDBC.

JDBC

```java
commit();

rollback();
```

Spring

```java
commit(status);

rollback(status);
```

---

# 5. Why an Interface?

Spring supports many technologies.

Example:

```text
JDBC

JPA (Hibernate)

MyBatis

JTA

MongoDB (limited support)

...
```

Each has different transaction implementations.

Spring hides these differences.

Application only knows:

```text
PlatformTransactionManager
```

This follows the **Dependency Inversion Principle (DIP)**.

---

# 6. Implementations

Common implementations:

```text
PlatformTransactionManager
           │
           ├─────────────► DataSourceTransactionManager
           │
           ├─────────────► JpaTransactionManager
           │
           ├─────────────► HibernateTransactionManager
           │
           └─────────────► JtaTransactionManager
```

For Spring JDBC, the implementation is usually:

```text
DataSourceTransactionManager
```

---

# 7. Internal Working

Suppose

```java
@Transactional
public void transferMoney(){

    debit();

    credit();

}
```

Internally:

```text
Client

↓

Proxy

↓

PlatformTransactionManager

↓

getTransaction()

↓

transferMoney()

↓

Success?

│

├── Yes

│

▼

commit()

│

└── No

▼

rollback()
```

Notice

**Your method never calls `commit()` or `rollback()`.**

Spring does it.

---

# 8. Role of Spring AOP

Remember AOP?

Spring creates:

```text
Client

↓

Proxy

↓

Real Service
```

Now transaction management becomes:

```text
Client

↓

Transaction Proxy

↓

Begin Transaction

↓

Real Method

↓

Commit / Rollback
```

This is why understanding Spring AOP was important.

---

# 9. Real Example

Your service:

```java
@Service
public class BankService{

    @Transactional
    public void transferMoney(){

        debit();

        credit();

    }

}
```

You wrote only business logic.

Spring secretly adds:

```text
Begin Transaction

↓

debit()

↓

credit()

↓

Success?

↓

Commit

OR

Rollback
```

---

# 10. Internal Flow

```text
Client

↓

Proxy

↓

PlatformTransactionManager

↓

setAutoCommit(false)

↓

Execute SQL

↓

Execute SQL

↓

Success?

│

├── Yes

│

▼

commit()

│

└── No

▼

rollback()
```

Notice something important:

Spring still uses JDBC underneath.

It simply automates the transaction handling.

---

# 11. Where is PlatformTransactionManager Used?

Whenever Spring sees:

```java
@Transactional
```

It internally uses a `PlatformTransactionManager`.

You rarely call it directly in application code.

---

# Interview Questions

## Q1. What is PlatformTransactionManager?

It is Spring's central interface for transaction management. It provides methods to begin, commit, and roll back transactions.

---

## Q2. Why did Spring introduce it?

To provide a consistent transaction API across different persistence technologies such as JDBC, JPA, Hibernate, and JTA.

---

## Q3. Which implementation is used with Spring JDBC?

```text
DataSourceTransactionManager
```

---

## Q4. Does PlatformTransactionManager replace JDBC?

No.

It manages transactions but still relies on the underlying technology (like JDBC) to execute database operations.

---

## Q5. Who calls `commit()`?

Spring's transaction infrastructure calls `commit()` or `rollback()` automatically based on the outcome of the `@Transactional` method.

---

# Best Practices

* Never call `commit()` or `rollback()` manually inside an `@Transactional` method.
* Keep transaction boundaries in the **service layer**, not the controller or repository.
* Use Spring's transaction abstraction instead of working directly with JDBC transaction APIs in Spring applications.

---

# Summary

```text
Without Spring

Connection

↓

setAutoCommit(false)

↓

Business Logic

↓

commit()

↓

rollback()

----------------------------------------

With Spring

@Transactional

↓

Transaction Proxy

↓

PlatformTransactionManager

↓

Business Logic

↓

Commit / Rollback
```

---

# 📍 Where We Are

```text
Spring Transaction Management

✅ Why Transactions
✅ ACID Properties
✅ JDBC Transactions
✅ Spring Transaction Architecture
✅ PlatformTransactionManager

⏭️ Next

@Transactional ⭐⭐⭐⭐⭐
        ↓
Transaction Proxy
        ↓
Propagation
        ↓
Isolation Levels
        ↓
Rollback Rules
```

---

# 🎯 Interview Tip

One of the most common interview questions is:

> **"How does `@Transactional` work internally?"**

The complete answer is:

1. Spring creates an **AOP proxy** around the bean.
2. When a `@Transactional` method is called, the **proxy intercepts** the call.
3. The proxy asks the **PlatformTransactionManager** to begin a transaction.
4. The target method executes.
5. If it completes successfully, the proxy calls **commit()**.
6. If a rollback-triggering exception occurs, the proxy calls **rollback()**.
7. The result or exception is returned to the caller.

In the **next chapter**, we'll dive deep into `@Transactional` itself—its attributes, internal flow, common pitfalls (like self-invocation), and the interview questions that are asked most often.
