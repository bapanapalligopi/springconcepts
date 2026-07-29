Excellent. You're learning in the correct sequence. Now we'll cover another important Spring JDBC topic.

# Chapter 7: NamedParameterJdbcTemplate

As always,

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# 1. Why? (The Problem)

Suppose you want to insert an employee.

Using `JdbcTemplate`:

```java
String sql =
"INSERT INTO employee(name, salary, department) VALUES (?, ?, ?)";

jdbcTemplate.update(
    sql,
    employee.getName(),
    employee.getSalary(),
    employee.getDepartment()
);
```

Looks fine.

Now imagine a larger query.

```java
String sql =
"UPDATE employee SET name=?, salary=?, department=?, city=?, email=? WHERE id=?";
```

Calling it:

```java
jdbcTemplate.update(
    sql,
    employee.getName(),
    employee.getSalary(),
    employee.getDepartment(),
    employee.getCity(),
    employee.getEmail(),
    employee.getId()
);
```

### Problem

What if you accidentally swap two parameters?

```java
jdbcTemplate.update(
    sql,
    employee.getSalary(),   // ❌ Wrong
    employee.getName(),     // ❌ Wrong
    employee.getDepartment(),
    employee.getCity(),
    employee.getEmail(),
    employee.getId()
);
```

The query executes, but your data becomes incorrect.

The compiler won't detect this because the parameter types are still valid.

---

# 2. What is NamedParameterJdbcTemplate?

Instead of using positional placeholders (`?`), you use **named placeholders**.

Instead of:

```sql
WHERE id = ?
```

You write:

```sql
WHERE id = :id
```

Instead of:

```sql
VALUES (?, ?, ?)
```

You write:

```sql
VALUES (:name, :salary, :department)
```

This makes SQL much easier to read and maintain.

---

# 3. How Does It Work?

### SQL

```sql
INSERT INTO employee(name, salary)
VALUES (:name, :salary)
```

Instead of passing values by position,

you pass them by name.

---

# 4. Using a Map

```java
Map<String, Object> params = new HashMap<>();

params.put("name", "Rahul");
params.put("salary", 65000);
```

Then:

```java
namedParameterJdbcTemplate.update(sql, params);
```

Spring matches:

```text
:name
      ↓
params.get("name")

:salary
      ↓
params.get("salary")
```

No need to remember parameter positions.

---

# 5. Using MapSqlParameterSource

Instead of a `Map`, Spring provides:

```java
MapSqlParameterSource params =
        new MapSqlParameterSource();

params.addValue("name", "Rahul");
params.addValue("salary", 65000);

namedParameterJdbcTemplate.update(sql, params);
```

Advantages:

* Cleaner API
* Chainable
* Better readability

---

# 6. BeanPropertySqlParameterSource

Suppose you already have:

```java
Employee employee = new Employee();

employee.setName("Rahul");
employee.setSalary(65000);
```

Instead of:

```java
params.addValue("name", employee.getName());
params.addValue("salary", employee.getSalary());
```

Simply do:

```java
BeanPropertySqlParameterSource params =
        new BeanPropertySqlParameterSource(employee);

namedParameterJdbcTemplate.update(sql, params);
```

Spring automatically maps:

```text
employee.getName()
        ↓
:name

employee.getSalary()
        ↓
:salary
```

---

# 7. Internal Working

Suppose:

```java
namedParameterJdbcTemplate.update(sql, params);
```

Internally:

```text
Repository
      │
      ▼
NamedParameterJdbcTemplate
      │
      ▼
Replace :name → ?
Replace :salary → ?
      │
      ▼
Create PreparedStatement
      │
      ▼
Bind Values
      │
      ▼
Execute SQL
      │
      ▼
Close Resources
```

Notice:

Internally it still uses **PreparedStatement**.

Named parameters are just a developer-friendly abstraction.

---

# 8. Example

SQL

```sql
UPDATE employee
SET salary = :salary
WHERE id = :id
```

Java

```java
String sql =
"UPDATE employee SET salary=:salary WHERE id=:id";

MapSqlParameterSource params =
        new MapSqlParameterSource()
            .addValue("salary", 80000)
            .addValue("id", 1);

namedParameterJdbcTemplate.update(sql, params);
```

Much easier to understand than:

```java
jdbcTemplate.update(sql, 80000, 1);
```

especially for queries with many parameters.

---

# 9. JdbcTemplate vs NamedParameterJdbcTemplate

| JdbcTemplate            | NamedParameterJdbcTemplate |
| ----------------------- | -------------------------- |
| Uses `?`                | Uses `:name`               |
| Positional parameters   | Named parameters           |
| Easier to mix up        | More readable              |
| Good for simple queries | Better for complex queries |

---

# 10. Where Is It Used?

Large enterprise applications often prefer `NamedParameterJdbcTemplate` because:

* Queries are easier to read.
* Less chance of parameter-order mistakes.
* Easier to maintain when SQL changes.

---

# Interview Questions

### Q1. Why was `NamedParameterJdbcTemplate` introduced?

To replace positional `?` parameters with named parameters, improving readability and reducing parameter-order mistakes.

---

### Q2. Does it replace `JdbcTemplate`?

No.

It is built on top of `JdbcTemplate`.

Internally, it still executes SQL using JDBC and `PreparedStatement`.

---

### Q3. What are the advantages?

* Readable SQL
* Safer parameter binding
* Easier maintenance
* Better for large queries

---

### Q4. Difference between `MapSqlParameterSource` and `BeanPropertySqlParameterSource`?

**MapSqlParameterSource**

* You manually add each parameter.

**BeanPropertySqlParameterSource**

* Spring automatically reads properties from a Java bean.

---

# Best Practices

* Use `JdbcTemplate` for simple CRUD with a few parameters.
* Prefer `NamedParameterJdbcTemplate` for long or complex SQL statements.
* Use `BeanPropertySqlParameterSource` when parameter names match your entity property names.
* Continue using parameter binding—never concatenate user input into SQL.

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
│
└── ⏭️ Next
      Exception Translation
           ↓
      DataAccessException
           ↓
      Batch Updates
           ↓
      Spring Transaction Management
```

---

## 🎯 Interview Tip

A common interview question is:

> **"Does NamedParameterJdbcTemplate improve database performance?"**

**Answer:**

**No.** It improves **developer productivity and code readability**, not database performance. Internally, Spring converts named parameters (like `:id`) into positional parameters (`?`) and uses a `PreparedStatement` to execute the SQL.

---

# 🚀 Next Topic: Exception Translation

This is one of Spring's biggest advantages over plain JDBC. We'll learn:

* Why Spring doesn't expose `SQLException` directly.
* What is `DataAccessException`.
* Why `DataAccessException` is unchecked.
* The exception hierarchy.
* How Spring translates vendor-specific exceptions (MySQL, Oracle, PostgreSQL) into a consistent exception model.
* Real interview questions and internal working.

This knowledge is very useful because it ties together JDBC, Spring's exception handling philosophy, and transaction management.
