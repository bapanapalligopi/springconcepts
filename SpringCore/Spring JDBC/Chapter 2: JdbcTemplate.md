Excellent. Now we are entering the **heart of Spring JDBC**.

If Spring JDBC had only one class, it would be **`JdbcTemplate`**.

This topic is asked in almost every Spring interview.

As always, we'll learn it using:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# Chapter 2: JdbcTemplate

---

# 1. Why? (The Problem)

Let's recall Plain JDBC.

```java
Connection con = DriverManager.getConnection(...);

PreparedStatement ps = con.prepareStatement(sql);

ps.setString(1, "Rahul");

ps.executeUpdate();

ps.close();

con.close();
```

Imagine writing this code in:

* saveEmployee()
* updateEmployee()
* deleteEmployee()
* getEmployee()
* getAllEmployees()

Every method repeats the same steps.

Spring asked:

> **Can we write this common code only once?**

The answer is **JdbcTemplate**.

---

# 2. What is JdbcTemplate?

### Definition

`JdbcTemplate` is Spring's central class for performing JDBC operations.

It handles all repetitive JDBC tasks automatically.

You only provide:

* SQL
* Parameters
* Mapping logic (for SELECT)

Spring handles everything else.

---

## Think of it like this

Without JdbcTemplate

```text
Your Code

↓

Connection

↓

PreparedStatement

↓

Execute

↓

Close
```

With JdbcTemplate

```text
Your Code

↓

JdbcTemplate

↓

Database
```

---

# Why is it called a "Template"?

This is an interview question.

JdbcTemplate follows the **Template Method Design Pattern**.

---

## What is the Template Method Pattern?

Suppose every database operation follows the same steps.

```text
Open Connection

↓

Create Statement

↓

Execute SQL

↓

Close Resources
```

Only one thing changes:

The SQL.

Example 1

```sql
INSERT INTO employee ...
```

Example 2

```sql
UPDATE employee ...
```

Example 3

```sql
DELETE employee ...
```

Everything else remains the same.

Spring writes the common steps once.

You only provide the changing part.

That is why it's called **JdbcTemplate**.

---

# 3. How JdbcTemplate Works

Suppose you write:

```java
jdbcTemplate.update(sql);
```

Internally,

```text
jdbcTemplate

↓

Get Connection

↓

Create PreparedStatement

↓

Execute SQL

↓

Handle Exception

↓

Close Statement

↓

Close Connection

↓

Return Result
```

You never write these steps.

Spring does.

---

# 4. Where is JdbcTemplate Stored?

Spring creates it as a Bean.

```text
IoC Container

↓

JdbcTemplate Bean
```

Repository class

```java
@Repository
public class EmployeeRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

}
```

Spring injects it automatically.

---

# 5. Common Methods of JdbcTemplate

These are the methods every Java developer should know.

| Method             | Purpose                                     |
| ------------------ | ------------------------------------------- |
| `update()`         | INSERT, UPDATE, DELETE                      |
| `query()`          | Returns multiple rows                       |
| `queryForObject()` | Returns one row                             |
| `queryForList()`   | Returns list of values or maps              |
| `batchUpdate()`    | Execute multiple SQL statements efficiently |

For **2 years experience**, these are sufficient.

---

# 6. update()

## Why?

Whenever you modify data.

Examples:

* INSERT
* UPDATE
* DELETE

---

Example

```java
String sql =
"INSERT INTO employee(name,salary) VALUES(?,?)";

jdbcTemplate.update(sql, "Rahul", 60000);
```

Spring internally

```text
Open Connection

↓

Prepare Statement

↓

Set Rahul

↓

Set 60000

↓

Execute

↓

Close Connection
```

Return value

```java
int rowsAffected
```

Example

```java
int rows =
jdbcTemplate.update(sql, "Rahul", 60000);

System.out.println(rows);
```

Output

```text
1
```

Meaning

One row inserted.

---

# Update Example

```java
String sql =
"UPDATE employee SET salary=? WHERE id=?";

jdbcTemplate.update(sql, 80000, 1);
```

---

# Delete Example

```java
String sql =
"DELETE FROM employee WHERE id=?";

jdbcTemplate.update(sql, 5);
```

Notice

Same method.

Different SQL.

---

# 7. queryForObject()

Suppose you want one employee.

```sql
SELECT * FROM employee WHERE id=1
```

Only one row.

Use

```java
queryForObject()
```

Example

```java
Employee employee =
jdbcTemplate.queryForObject(
        sql,
        rowMapper,
        1);
```

Returns

```java
Employee
```

---

# 8. query()

Suppose

```sql
SELECT * FROM employee
```

Returns

```text
100 Employees
```

Use

```java
query()
```

Example

```java
List<Employee> employees =
jdbcTemplate.query(sql, rowMapper);
```

Return

```java
List<Employee>
```

---

# Difference

```text
queryForObject()

↓

One Row

↓

Employee
```

---

```text
query()

↓

Multiple Rows

↓

List<Employee>
```

---

# 9. queryForList()

Suppose

```sql
SELECT name FROM employee
```

Returns

```text
Rahul

Amit

John
```

Use

```java
List<String> names =
jdbcTemplate.queryForList(
        sql,
        String.class);
```

---

# 10. batchUpdate()

Suppose

Need to insert

```text
1000 Employees
```

Wrong

```java
update()

update()

update()

update()
```

1000 times.

Better

```java
batchUpdate()
```

Spring sends them efficiently to the database.

---

# Internal Working

Suppose

```java
jdbcTemplate.update(sql);
```

Internally

```text
EmployeeRepository

↓

JdbcTemplate

↓

DataSource

↓

Connection

↓

PreparedStatement

↓

Execute SQL

↓

Handle SQLException

↓

Close Resources

↓

Return
```

Notice

You never touch

```java
Connection
```

directly.

---

# Real Project

Repository

```java
@Repository
public class EmployeeRepository {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public void save(Employee employee) {

        String sql =
        "INSERT INTO employee(name,salary) VALUES(?,?)";

        jdbcTemplate.update(
                sql,
                employee.getName(),
                employee.getSalary());

    }

}
```

Service

```java
@Service
public class EmployeeService {

    @Autowired
    EmployeeRepository repository;

    public void save(Employee employee) {

        repository.save(employee);

    }

}
```

Controller

```java
@PostMapping
public String save() {

    service.save(employee);

    return "Saved";
}
```

Clean architecture.

---

# Interview Questions

### Q1. Why is it called JdbcTemplate?

Because it implements the **Template Method Design Pattern**. Spring provides the common JDBC workflow, and the developer supplies only the variable parts like SQL and mapping.

---

### Q2. Does JdbcTemplate replace JDBC?

No.

It is built **on top of JDBC**.

Internally, it still uses:

* `Connection`
* `PreparedStatement`
* `ResultSet`

---

### Q3. Which method is used for INSERT?

```java
update()
```

---

### Q4. Which method returns multiple rows?

```java
query()
```

---

### Q5. Which method returns one row?

```java
queryForObject()
```

---

### Q6. Which method is used for bulk operations?

```java
batchUpdate()
```

---

# Best Practices

* Always use parameter placeholders (`?`) instead of building SQL with string concatenation.
* Keep SQL inside the repository layer.
* Reuse `JdbcTemplate`; let Spring manage it as a singleton bean.
* Use `queryForObject()` only when you expect exactly one row. If no row or multiple rows are returned, Spring throws an exception.

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
│
└── Next →
      CRUD Operations
          ↓
      RowMapper
          ↓
      BeanPropertyRowMapper
          ↓
      NamedParameterJdbcTemplate
          ↓
      Exception Translation
```

---

## 🚀 Next Topic: **CRUD Operations with JdbcTemplate**

Now we'll build a **complete Employee Management application** using MySQL and Spring JDBC.

We'll implement, step by step:

* Create the `employee` table
* Configure the `DataSource`
* Insert an employee
* Update an employee
* Delete an employee
* Find an employee by ID
* Find all employees
* Understand what happens internally for each operation

By the end, you'll know how a real Spring application uses `JdbcTemplate` in production and be ready for common interview questions on Spring JDBC.
