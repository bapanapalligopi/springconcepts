Excellent! Now we are entering one of the **most important Spring JDBC interview topics**.

> **RowMapper** is asked in almost every Spring JDBC interview.

As always, we'll learn using:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# Chapter 4: RowMapper

---

# 1. Why? (The Problem)

Suppose you execute this SQL:

```sql
SELECT id, name, salary
FROM employee
WHERE id = 1;
```

Database returns:

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 65000  |

But your Java application expects:

```java
Employee employee = new Employee();
```

Question:

**How does Spring know that:**

* `id` → `employee.setId()`
* `name` → `employee.setName()`
* `salary` → `employee.setSalary()`

It doesn't.

Someone has to perform this conversion.

That is exactly why **RowMapper** exists.

---

# 2. What is RowMapper?

### Definition

A **RowMapper** is an interface that converts **one row** of a `ResultSet` into **one Java object**.

Think of it as a translator.

```text
Database Row
      │
      ▼
 RowMapper
      │
      ▼
Employee Object
```

---

## Real-Life Analogy

Imagine a passport office.

Passport Data:

```text
Name : Rahul

Age : 25

City : Hyderabad
```

Your Java object is

```java
Person person = new Person();
```

Someone must copy each field into the object.

That "someone" is the RowMapper.

---

# 3. Where is RowMapper Used?

Only in **SELECT** queries.

Not in:

* INSERT
* UPDATE
* DELETE

Example:

```java
jdbcTemplate.query(...)
```

or

```java
jdbcTemplate.queryForObject(...)
```

Both need RowMapper because they read data.

---

# 4. RowMapper Interface

Spring provides this interface:

```java
public interface RowMapper<T> {

    T mapRow(ResultSet rs, int rowNum)
            throws SQLException;

}
```

Let's understand it.

---

## Generic `<T>`

Suppose

```java
Employee
```

Then

```java
RowMapper<Employee>
```

means

> This mapper converts a row into an `Employee`.

---

## mapRow()

This method is called **once for every row**.

Parameters:

```java
ResultSet rs
```

Contains one row.

---

```java
int rowNum
```

Current row number.

Usually,

you won't use `rowNum`.

---

Return

```java
Employee
```

---

# 5. Writing Our Own RowMapper

Employee class

```java
public class Employee {

    private Integer id;

    private String name;

    private Double salary;

    // getters and setters

}
```

Now

Create

```java
public class EmployeeRowMapper
        implements RowMapper<Employee> {
```

Implement

```java
@Override
public Employee mapRow(
        ResultSet rs,
        int rowNum)
        throws SQLException {

    Employee employee = new Employee();

    employee.setId(
            rs.getInt("id"));

    employee.setName(
            rs.getString("name"));

    employee.setSalary(
            rs.getDouble("salary"));

    return employee;
}
```

Done.

---

# Why Each Line?

```java
Employee employee = new Employee();
```

Create an empty object.

---

```java
rs.getInt("id");
```

Read column

```text
id
```

from ResultSet.

---

```java
employee.setId(...)
```

Copy database value into Java object.

---

Finally

```java
return employee;
```

Give it back to Spring.

---

# 6. Internal Working

Suppose

```java
jdbcTemplate.query(
        sql,
        new EmployeeRowMapper());
```

Database returns

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 60000  |
| 2  | Amit  | 70000  |
| 3  | John  | 80000  |

Flow

```text
JdbcTemplate
      │
      ▼
Execute SQL
      │
      ▼
ResultSet
      │
      ▼
Row 1
      │
      ▼
mapRow()
      │
      ▼
Employee Object
      │
      ▼
Add to List
      │
      ▼
Row 2
      │
      ▼
mapRow()
      │
      ▼
Employee Object
      │
      ▼
Add to List
      │
      ▼
Row 3
      │
      ▼
mapRow()
      │
      ▼
Employee Object
      │
      ▼
Return List<Employee>
```

Notice:

`mapRow()` executes **once per row**.

---

# 7. query() Example

```java
String sql =
"SELECT * FROM employee";

List<Employee> employees =
jdbcTemplate.query(
        sql,
        new EmployeeRowMapper());
```

Spring internally does

```text
Row 1 → Employee

Row 2 → Employee

Row 3 → Employee

↓

List<Employee>
```

---

# 8. queryForObject()

Suppose

```sql
SELECT *
FROM employee
WHERE id=1;
```

Only one row.

```java
Employee employee =
jdbcTemplate.queryForObject(

        sql,

        new EmployeeRowMapper(),

        1);
```

Flow

```text
One Row

↓

mapRow()

↓

Employee
```

---

# 9. Visual Understanding

```text
               Database

+----------------------------+
| id | name  | salary        |
+----------------------------+
| 1  | Rahul | 60000         |
| 2  | Amit  | 70000         |
+----------------------------+
            │
            ▼
      ResultSet
            │
            ▼
 EmployeeRowMapper
            │
            ▼
Employee(id=1,name=Rahul,salary=60000)

Employee(id=2,name=Amit,salary=70000)
            │
            ▼
List<Employee>
```

---

# 10. Advantages

Without RowMapper

Every query

```java
Employee e = new Employee();

e.setId(...);

e.setName(...);

e.setSalary(...);
```

Again

Again

Again

Repeated.

With RowMapper

One class.

Reuse everywhere.

---

# Real Project Structure

```text
repository
│
├── EmployeeRepository.java
├── EmployeeRowMapper.java
```

Repository

```java
@Repository
public class EmployeeRepository {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public List<Employee> findAll(){

        return jdbcTemplate.query(

                "SELECT * FROM employee",

                new EmployeeRowMapper());

    }

}
```

Very clean.

---

# Interview Questions

## Q1. Why do we need RowMapper?

To convert each row of a `ResultSet` into a Java object.

---

## Q2. When is RowMapper used?

Only for **SELECT** operations.

---

## Q3. How many times is `mapRow()` called?

Once for **every row** returned by the query.

Example:

10 rows

↓

`mapRow()` executes **10 times**.

---

## Q4. Can we reuse RowMapper?

Yes.

One `EmployeeRowMapper` can be reused across multiple repository methods.

---

## Q5. Does RowMapper create the ResultSet?

No.

`JdbcTemplate` creates and manages the `ResultSet`.

RowMapper only converts a row into an object.

---

# Common Mistake

Many beginners think:

```text
RowMapper

↓

Converts entire ResultSet
```

❌ Wrong.

Correct:

```text
One Row

↓

One Object
```

`JdbcTemplate` loops through the `ResultSet` and calls `mapRow()` for each row.

---

# Best Practices

* Keep `RowMapper` classes focused on one entity.
* Reuse the same mapper wherever possible.
* Use meaningful column names in SQL.
* Avoid putting business logic inside `mapRow()`.

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
│
└── Next →
      BeanPropertyRowMapper
           ↓
      ResultSetExtractor
           ↓
      NamedParameterJdbcTemplate
           ↓
      Exception Translation
```

---

# 🎯 Before We Move to BeanPropertyRowMapper

### Small Interview Question

Suppose the database returns:

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 60000  |
| 2  | Amit  | 70000  |

How many times will Spring call:

```java
mapRow(ResultSet rs, int rowNum)
```

And what will `jdbcTemplate.query()` finally return?

Try answering this yourself first. This is a classic interview question, and your answer will help confirm you've understood how `RowMapper` works before we move to `BeanPropertyRowMapper`.
