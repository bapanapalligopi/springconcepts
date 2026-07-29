Excellent. This is one of the **most commonly asked Spring JDBC interview questions**.

> **Difference between `RowMapper` and `ResultSetExtractor`**

Many developers memorize the table. Instead, let's understand **why Spring created `ResultSetExtractor`**.

As always:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# Chapter 6: ResultSetExtractor

---

# 1. Why? (The Problem)

Let's say we have this table.

### Employee Table

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 60000  |
| 2  | Amit  | 70000  |
| 3  | John  | 80000  |

You execute:

```sql
SELECT * FROM employee;
```

With `RowMapper`:

```java
List<Employee> employees =
jdbcTemplate.query(
        sql,
        new EmployeeRowMapper());
```

Spring internally does:

```text
Row 1
   ↓
mapRow()
   ↓
Employee 1

Row 2
   ↓
mapRow()
   ↓
Employee 2

Row 3
   ↓
mapRow()
   ↓
Employee 3

↓

List<Employee>
```

Everything is fine.

---

## But Consider a JOIN Query

Suppose we have:

### Employee

| id | name  | dept_id |
| -- | ----- | ------- |
| 1  | Rahul | 10      |

### Department

| id | department_name |
| -- | --------------- |
| 10 | IT              |

SQL:

```sql
SELECT
e.id,
e.name,
d.department_name
FROM employee e
JOIN department d
ON e.dept_id=d.id;
```

Result:

| id | name  | department_name |
| -- | ----- | --------------- |
| 1  | Rahul | IT              |

Now imagine you want this object:

```java
Employee

↓

Department

↓

EmployeeResponse
```

Or:

```java
Employee
    └── Department
```

This is **not a simple row-to-object mapping**.

Spring needed another approach.

Hence,

**ResultSetExtractor**.

---

# 2. What is ResultSetExtractor?

### Definition

`ResultSetExtractor` processes the **entire `ResultSet` at once**.

Unlike `RowMapper`, it is **not called for every row**.

Think of it like this:

```text
ResultSet
      │
      ▼
ResultSetExtractor
      │
      ▼
Any Object
```

---

# Difference

### RowMapper

```text
One Row

↓

One Object
```

---

### ResultSetExtractor

```text
Entire ResultSet

↓

One Object
```

or

```text
Entire ResultSet

↓

Custom Structure
```

---

# 3. Interface

```java
public interface ResultSetExtractor<T>{

    T extractData(ResultSet rs)
            throws SQLException;

}
```

Notice

No

```java
rowNum
```

because

Spring gives you

the entire ResultSet.

---

# 4. Example

Suppose

```sql
SELECT * FROM employee
```

Create

```java
public class EmployeeExtractor
implements ResultSetExtractor<List<Employee>>{
```

Implementation

```java
@Override
public List<Employee> extractData(
        ResultSet rs)
        throws SQLException {

    List<Employee> employees =
            new ArrayList<>();

    while(rs.next()){

        Employee employee =
                new Employee();

        employee.setId(rs.getInt("id"));
        employee.setName(rs.getString("name"));
        employee.setSalary(rs.getDouble("salary"));

        employees.add(employee);

    }

    return employees;
}
```

Notice

You wrote

```java
while(rs.next())
```

Yourself.

With `RowMapper`,

Spring writes it.

---

# 5. Internal Working

Suppose

```java
jdbcTemplate.query(

        sql,

        new EmployeeExtractor());
```

Flow

```text
JdbcTemplate

↓

Execute SQL

↓

ResultSet

↓

EmployeeExtractor

↓

while(rs.next())

↓

Employee

↓

Employee

↓

Employee

↓

List<Employee>

↓

Return
```

---

# 6. RowMapper Internal Flow

```text
JdbcTemplate

↓

ResultSet

↓

while(rs.next())

↓

mapRow()

↓

mapRow()

↓

mapRow()

↓

Return List
```

Notice

Spring writes

```java
while(rs.next())
```

---

# 7. ResultSetExtractor Flow

```text
JdbcTemplate

↓

ResultSet

↓

extractData()

↓

Developer writes

while(rs.next())

↓

Return
```

You control everything.

---

# 8. When Should We Use ResultSetExtractor?

### Case 1

Nested Objects

Example

```java
Employee

↓

Department
```

---

### Case 2

Parent-Child Mapping

Suppose

Order

↓

Order Items

Need

```java
Order

↓

List<OrderItem>
```

One SQL

Many rows

Need one object.

`RowMapper`

Not suitable.

`ResultSetExtractor`

Perfect.

---

### Case 3

Complex Reports

Dashboard

Analytics

Summary

Custom DTOs

---

# 9. Example

Imagine

Database

| Order | Item     |
| ----- | -------- |
| 1     | Laptop   |
| 1     | Mouse    |
| 1     | Keyboard |

Need

```java
Order

↓

items

Laptop

Mouse

Keyboard
```

One Order.

Three rows.

`RowMapper`

Would create

```text
Order

Order

Order
```

Wrong.

`ResultSetExtractor`

Creates

```text
One Order

↓

Three Items
```

Correct.

---

# 10. Comparison

| Feature   | RowMapper          | ResultSetExtractor |
| --------- | ------------------ | ------------------ |
| Processes | One row            | Entire ResultSet   |
| Method    | `mapRow()`         | `extractData()`    |
| Loop      | Spring             | Developer          |
| Best For  | Simple entities    | Complex mappings   |
| Returns   | One object per row | Any object         |

---

# Visual Comparison

## RowMapper

```text
Row1

↓

Employee

Row2

↓

Employee

Row3

↓

Employee

↓

List<Employee>
```

---

## ResultSetExtractor

```text
Entire ResultSet

↓

Loop

↓

Custom Logic

↓

Return Any Object
```

---

# Interview Questions

## Q1. What is the difference between RowMapper and ResultSetExtractor?

**RowMapper** maps one row at a time and Spring loops through the `ResultSet`.

**ResultSetExtractor** receives the entire `ResultSet`, and the developer is responsible for iterating through it.

---

## Q2. Which one is used more often?

**RowMapper.**

For 90% of CRUD applications,

`RowMapper`

or

`BeanPropertyRowMapper`

is sufficient.

---

## Q3. When should we use ResultSetExtractor?

* Complex joins
* Parent-child relationships
* Nested DTOs
* Reports
* Custom aggregations

---

## Q4. Does ResultSetExtractor replace RowMapper?

No.

They solve different problems.

---

## Q5. Which is simpler?

`RowMapper`

because Spring handles the iteration.

---

# Best Practices

* Use `RowMapper` or `BeanPropertyRowMapper` for standard CRUD operations.
* Use `ResultSetExtractor` only when the result cannot be represented as "one row = one object".
* Keep extraction logic focused on mapping; avoid mixing business logic.

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
│
└── ⏭️ Next
      NamedParameterJdbcTemplate
           ↓
      Exception Translation
           ↓
      Batch Updates
```

---

# Interview Cheat Code

A concise answer that impresses interviewers:

| If the question is...                                           | Answer               |
| --------------------------------------------------------------- | -------------------- |
| "One database row maps to one Java object?"                     | `RowMapper`          |
| "Need to process the entire ResultSet or build nested objects?" | `ResultSetExtractor` |

---

## 🚀 Next Topic: NamedParameterJdbcTemplate

This is another practical Spring JDBC feature. We'll cover:

* **Why was `NamedParameterJdbcTemplate` introduced?**
* **Problems with positional `?` parameters**
* **Named parameters (`:id`, `:name`)**
* **MapSqlParameterSource**
* **BeanPropertySqlParameterSource**
* **Real project examples**
* **Interview questions and best practices**

This topic is commonly used in enterprise applications because it makes SQL more readable and less error-prone.
