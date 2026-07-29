Excellent! We're almost done with **Spring JDBC**.

The next topic is one of Spring's biggest strengths and a very common interview topic.

# Chapter 8: Exception Translation

As always, we'll learn:

> **Why → What → How → Where → Internal Working → Interview Questions → Best Practices**

---

# 1. Why? (The Problem)

Suppose you're using plain JDBC.

```java
try {

    Connection con = DriverManager.getConnection(...);

    PreparedStatement ps =
            con.prepareStatement(sql);

    ps.executeUpdate();

} catch (SQLException e) {

    e.printStackTrace();

}
```

Everything throws:

```java
SQLException
```

Now suppose your application supports multiple databases.

* MySQL
* Oracle
* PostgreSQL

Each database throws different error codes.

Example:

### MySQL

```text
Error Code : 1062

Duplicate Entry
```

### Oracle

```text
ORA-00001

Unique Constraint Violated
```

### PostgreSQL

```text
23505

Duplicate Key Value
```

Question:

How can your Java application write one catch block for all databases?

Impossible.

Spring solved this problem.

---

# 2. What is Exception Translation?

### Definition

Spring converts database-specific exceptions (like `SQLException`) into a **consistent hierarchy of unchecked exceptions**.

Instead of dealing with:

```java
SQLException
```

You deal with:

```java
DataAccessException
```

---

# Visual

```text
MySQL SQLException
          │
Oracle SQLException
          │
PostgreSQL SQLException
          │
          ▼
Spring Exception Translator
          │
          ▼
DataAccessException
```

Your application only understands:

```java
DataAccessException
```

No database-specific code.

---

# 3. Why is DataAccessException Runtime Exception?

Look at JDBC.

```java
public void save(){

    jdbcTemplate.update(sql);

}
```

Notice

No

```java
throws SQLException
```

Why?

Because

```java
DataAccessException
```

extends

```java
RuntimeException
```

Hierarchy

```text
Throwable
    │
    ▼
Exception
    │
    ▼
RuntimeException
    │
    ▼
DataAccessException
```

Spring intentionally made it **unchecked**.

---

# 4. Benefits

Without Spring

```java
public void save()
        throws SQLException
```

Every service method

Every controller

Must handle

```java
SQLException
```

Very messy.

---

With Spring

```java
public void save(){

    jdbcTemplate.update(sql);

}
```

No checked exceptions.

Cleaner code.

---

# 5. Internal Working

Suppose

```java
jdbcTemplate.update(sql);
```

Database throws

```text
Duplicate Key Error
```

Flow

```text
Database

↓

SQLException

↓

SQLExceptionTranslator

↓

DuplicateKeyException

↓

DataAccessException

↓

Your Application
```

Notice

Spring intercepts

```java
SQLException
```

before your application sees it.

---

# 6. Exception Hierarchy

The most important hierarchy:

```text
Throwable
    │
    ▼
RuntimeException
    │
    ▼
DataAccessException
        │
        ├── DuplicateKeyException
        │
        ├── DataIntegrityViolationException
        │
        ├── EmptyResultDataAccessException
        │
        ├── IncorrectResultSizeDataAccessException
        │
        ├── CannotAcquireLockException
        │
        └── DeadlockLoserDataAccessException
```

For a **2-year developer**, these are enough.

---

# 7. Common Exceptions

## DuplicateKeyException

Suppose

```sql
INSERT INTO employee(id,name)

VALUES(1,'Rahul');
```

ID already exists.

Spring throws

```java
DuplicateKeyException
```

instead of

```java
SQLException
```

---

## EmptyResultDataAccessException

Suppose

```java
jdbcTemplate.queryForObject(...)
```

Expected

One Employee

Database

Returns

Zero rows.

Spring throws

```java
EmptyResultDataAccessException
```

---

## IncorrectResultSizeDataAccessException

Suppose

```java
queryForObject()
```

expects

One row.

Database

Returns

Five rows.

Spring throws

```java
IncorrectResultSizeDataAccessException
```

---

## DataIntegrityViolationException

Examples

* Foreign Key violation
* NOT NULL violation
* Unique constraint violation

---

# 8. Where is Exception Translation Used?

Every JdbcTemplate operation.

```java
update()

query()

queryForObject()

batchUpdate()
```

All use it internally.

---

# 9. Internal Flow

Suppose

```java
jdbcTemplate.queryForObject(...)
```

Flow

```text
Repository

↓

JdbcTemplate

↓

PreparedStatement

↓

Database

↓

SQLException

↓

SQLExceptionTranslator

↓

DataAccessException

↓

Repository

↓

Service

↓

Controller
```

Your service never sees

```java
SQLException
```

---

# 10. Why is this Useful?

Suppose today

Database

MySQL

Tomorrow

Company changes to Oracle.

Without Spring

Need to rewrite

every

```java
catch(SQLException)
```

With Spring

Nothing changes.

Still

```java
DataAccessException
```

This makes your code **database-independent**.

---

# Interview Questions

## Q1. What is Exception Translation?

Spring converts database-specific `SQLException`s into a consistent hierarchy of unchecked `DataAccessException`s.

---

## Q2. Why is DataAccessException unchecked?

To avoid forcing every layer of the application to handle or declare checked database exceptions, resulting in cleaner code.

---

## Q3. Does JdbcTemplate throw SQLException?

No.

It catches `SQLException` internally and translates it into `DataAccessException`.

---

## Q4. Which class performs the translation?

```java
SQLExceptionTranslator
```

Internally used by `JdbcTemplate`.

---

## Q5. Why is this useful?

It provides:

* Database independence
* Cleaner code
* Consistent exception hierarchy
* Better maintainability

---

# Best Practices

* Catch `DataAccessException` only if you can recover or add meaningful context. Otherwise, let it propagate to a global exception handler (for example, `@ControllerAdvice` in Spring MVC).
* Don't convert `DataAccessException` back into `SQLException`.
* Log the exception with sufficient context (SQL operation, entity, identifiers), but avoid exposing database details to API clients.

---

# Summary

```text
Without Spring

Database
     │
     ▼
SQLException
     │
     ▼
Application

----------------------------------

With Spring

Database
     │
     ▼
SQLException
     │
     ▼
SQLExceptionTranslator
     │
     ▼
DataAccessException
     │
     ▼
Application
```

---

# 📍 Where We Are

```text
Spring Framework

✅ Spring Core
✅ Spring AOP

Spring JDBC
│
├── ✅ Why Spring JDBC
├── ✅ JdbcTemplate
├── ✅ CRUD Operations
├── ✅ RowMapper
├── ✅ BeanPropertyRowMapper
├── ✅ ResultSetExtractor
├── ✅ NamedParameterJdbcTemplate
├── ✅ Exception Translation
│
└── ⏭️ Next
      Batch Updates
           ↓
      Spring Transaction Management
```

---

## 🎯 Important Interview Tip

One question interviewers frequently ask is:

> **"Why doesn't `JdbcTemplate` throw `SQLException`?"**

A strong answer is:

> "`JdbcTemplate` catches `SQLException` internally and uses `SQLExceptionTranslator` to convert it into Spring's unchecked `DataAccessException` hierarchy. This removes database-specific code from the application and simplifies exception handling."

---

## 🚀 Next Topic: Batch Updates

We'll learn:

* Why batch processing is needed.
* How `batchUpdate()` works internally.
* Performance benefits over multiple `update()` calls.
* `BatchPreparedStatementSetter`.
* Real-world use cases like importing thousands of employee records.
* Common interview questions and best practices.

After that, we'll have completed **Spring JDBC** and move on to **Spring Transaction Management**, which builds directly on everything you've learned so far.
