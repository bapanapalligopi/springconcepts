Excellent! Welcome to **Spring JDBC**.

This module is very important because it connects your Spring knowledge with databases. We'll continue with our teaching style:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# Chapter 1: Why Spring JDBC? (Problems with Plain JDBC)

---

# 1. Why Do We Need JDBC?

Imagine you're building an Employee Management System.

The application flow is:

```text
Client
   │
   ▼
Spring Boot Application
   │
   ▼
MySQL Database
```

Question:

**Can Java directly understand MySQL?**

No.

Java and databases speak different languages.

Java uses:

```java
Employee employee = new Employee();
```

MySQL understands:

```sql
SELECT * FROM employee;
```

We need a bridge.

That bridge is **JDBC (Java Database Connectivity).**

---

## What is JDBC?

**Definition:**

> JDBC is a Java API that allows Java applications to communicate with relational databases.

Think of it like a translator.

```text
Java
   │
   ▼
JDBC API
   │
   ▼
JDBC Driver
   │
   ▼
MySQL
```

Without JDBC,

Java cannot execute SQL.

---

# 2. How Plain JDBC Works

Suppose you want to insert an employee.

The steps are:

```text
Load Driver
      │
      ▼
Create Connection
      │
      ▼
Create Statement
      │
      ▼
Execute SQL
      │
      ▼
Process Result
      │
      ▼
Close Resources
```

Every database operation follows these steps.

---

# Example: Plain JDBC

```java
public void saveEmployee() throws Exception {

    // 1. Load Driver
    Class.forName("com.mysql.cj.jdbc.Driver");

    // 2. Create Connection
    Connection con =
        DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/company",
            "root",
            "password");

    // 3. Create Statement
    PreparedStatement ps =
        con.prepareStatement(
            "INSERT INTO employee(name) VALUES(?)");

    // 4. Set Values
    ps.setString(1, "Rahul");

    // 5. Execute
    ps.executeUpdate();

    // 6. Close
    ps.close();
    con.close();
}
```

At first glance, this seems okay.

But imagine writing this for **100 database methods**.

---

# 3. Problems with Plain JDBC

Let's identify the pain points one by one.

---

## Problem 1: Boilerplate Code

Every method repeats:

```java
Connection connection = ...

PreparedStatement statement = ...

statement.execute...

statement.close();

connection.close();
```

The same code appears everywhere.

---

## Problem 2: Manual Resource Management

Suppose you forget:

```java
connection.close();
```

What happens?

The connection stays open.

This causes a **connection leak**.

Over time:

```text
Application
     │
     ▼
Many Open Connections
     │
     ▼
Database Connection Pool Full
     │
     ▼
Application Stops Serving Requests
```

This is a real production issue.

---

## Problem 3: Exception Handling

Look at the code.

```java
try {

    ...

} catch(SQLException e) {

    ...

} finally {

    connection.close();

}
```

Every method repeats the same exception handling.

---

## Problem 4: Manual Object Mapping

Suppose the database returns:

| ID | Name  | Salary |
| -- | ----- | ------ |
| 1  | Rahul | 60000  |

You manually write:

```java
Employee employee = new Employee();

employee.setId(rs.getInt("id"));

employee.setName(rs.getString("name"));

employee.setSalary(rs.getDouble("salary"));
```

For every query.

Imagine doing this for 50 tables.

---

## Problem 5: Checked Exceptions

JDBC throws:

```java
SQLException
```

Every method either handles it or declares:

```java
throws SQLException
```

Your service layer becomes cluttered with database-specific exceptions.

---

## Problem 6: Connection Management

Who opens the connection?

You.

Who closes it?

You.

If you forget, the application may eventually run out of available database connections.

---

# Summary of JDBC Problems

```text
Plain JDBC

❌ Too much boilerplate
❌ Manual Connection Management
❌ Manual Exception Handling
❌ Manual Result Mapping
❌ Resource Leaks
❌ Verbose Code
```

---

# 4. Enter Spring JDBC

Spring says:

> "You focus on SQL and business logic. I'll handle everything else."

Instead of writing 30–40 lines,

you write:

```java
jdbcTemplate.update(sql, values);
```

That's it.

Spring manages:

* Opening the connection
* Closing the connection
* Preparing the statement
* Executing SQL
* Translating exceptions
* Cleaning up resources

---

# Before vs After

### Plain JDBC

```text
Load Driver

↓

Open Connection

↓

Create Statement

↓

Execute SQL

↓

Handle Exception

↓

Close Statement

↓

Close Connection
```

---

### Spring JDBC

```text
Write SQL

↓

JdbcTemplate

↓

Done
```

---

# 5. What is JdbcTemplate?

**Definition:**

`JdbcTemplate` is Spring's central class for performing JDBC operations.

Think of it as a smart helper.

```text
Your Code

↓

JdbcTemplate

↓

JDBC

↓

Database
```

Instead of interacting directly with JDBC,

you interact with `JdbcTemplate`.

---

# 6. Internal Working of JdbcTemplate

Suppose you call:

```java
jdbcTemplate.update(sql);
```

Internally:

```text
Your Service
      │
      ▼
JdbcTemplate
      │
      ▼
Get Connection
      │
      ▼
Create PreparedStatement
      │
      ▼
Execute SQL
      │
      ▼
Handle SQLException
      │
      ▼
Close Resources
      │
      ▼
Return Result
```

Everything that you previously did manually is now handled by Spring.

---

# 7. Where Is Spring JDBC Used?

Even today, many enterprise applications use `JdbcTemplate` because:

* It offers fine-grained control over SQL.
* It's lightweight.
* It's fast.
* It's simpler than full ORM frameworks when queries are straightforward.

Typical use cases:

* Reporting applications
* Legacy systems
* Performance-critical SQL
* Applications with complex SQL queries

---

# 8. JDBC vs Spring JDBC

| Plain JDBC                          | Spring JDBC                    |
| ----------------------------------- | ------------------------------ |
| Manual connection management        | Automatic                      |
| Manual exception handling           | Exception translation          |
| Manual resource cleanup             | Automatic                      |
| Manual `PreparedStatement` creation | Handled by `JdbcTemplate`      |
| More boilerplate                    | Less boilerplate               |
| Verbose                             | Cleaner and easier to maintain |

---

# Interview Questions

### Q1. Why was Spring JDBC introduced?

To reduce JDBC boilerplate code, automatically manage resources, and provide consistent exception handling while still allowing developers to write SQL.

---

### Q2. What is `JdbcTemplate`?

It is Spring's core helper class that simplifies JDBC operations by managing connections, statements, exceptions, and resource cleanup.

---

### Q3. Does Spring JDBC replace JDBC?

No.

It is built **on top of JDBC**.

Internally, it still uses JDBC APIs like `Connection`, `PreparedStatement`, and `ResultSet`.

---

### Q4. What problems does `JdbcTemplate` solve?

* Resource management
* Connection handling
* Exception translation
* Reduced boilerplate code
* Simplified CRUD operations

---

# Best Practices

* Always use `PreparedStatement`-based methods (which `JdbcTemplate` does by default) instead of concatenating SQL strings.
* Keep SQL in the repository/DAO layer, not in controllers.
* Use `RowMapper` to map query results to Java objects (we'll learn this next).
* Don't catch every `SQLException` manually; let Spring translate exceptions into its `DataAccessException` hierarchy.

---

# 📍 Where We Are

```text
Spring Framework

✅ Spring Core
✅ Spring AOP

Spring JDBC
│
├── ✅ Why Spring JDBC?
├── ✅ Problems with Plain JDBC
└── ⏭️ Next: JdbcTemplate
      ├── What is JdbcTemplate?
      ├── CRUD Operations
      ├── update()
      ├── query()
      ├── queryForObject()
      └── Internal Working
```

## Next Lesson

We'll dive deep into **`JdbcTemplate`** itself:

* Why it is called a **Template**.
* How the **Template Method Design Pattern** is used.
* Every important method (`update`, `query`, `queryForObject`, `queryForList`).
* Real CRUD examples with MySQL.
* How it works internally.
* Common interview questions and best practices.

By the end of that lesson, you'll understand not just how to use `JdbcTemplate`, but why it's designed the way it is.
