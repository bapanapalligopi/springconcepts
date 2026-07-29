Excellent! You're following the roadmap very well.

So far we've completed:

```text
✅ Spring Core
✅ Spring AOP

Spring JDBC
│
├── ✅ Why Spring JDBC
├── ✅ JdbcTemplate
├── ✅ CRUD Operations
├── ✅ RowMapper
│
└── ⏭️ BeanPropertyRowMapper (Current Topic)
```

---

# Chapter 5: BeanPropertyRowMapper

As always,

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# 1. Why? (The Problem)

Suppose your `EmployeeRowMapper` looks like this:

```java
public class EmployeeRowMapper implements RowMapper<Employee> {

    @Override
    public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {

        Employee employee = new Employee();

        employee.setId(rs.getInt("id"));
        employee.setName(rs.getString("name"));
        employee.setSalary(rs.getDouble("salary"));

        return employee;
    }
}
```

Looks good.

Now imagine you have these entities:

```text
Employee
Department
Customer
Product
Order
Address
Invoice
Payment
```

You now need:

```text
EmployeeRowMapper
DepartmentRowMapper
CustomerRowMapper
ProductRowMapper
OrderRowMapper
...
```

You'll end up writing the same mapping code repeatedly.

Spring thought:

> **"Can we generate this mapping automatically?"**

That's why **BeanPropertyRowMapper** was introduced.

---

# 2. What is BeanPropertyRowMapper?

### Definition

`BeanPropertyRowMapper` is Spring's built-in implementation of `RowMapper` that automatically maps database columns to Java bean properties.

Instead of writing:

```java
employee.setId(...);
employee.setName(...);
employee.setSalary(...);
```

Spring does it for you.

---

# 3. How Does It Work?

Suppose your table is:

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 60000  |

Java class

```java
public class Employee{

    private Integer id;

    private String name;

    private Double salary;

}
```

Notice

Database Column

```text
id
```

Java Property

```text
id
```

Same name.

Database

```text
salary
```

Java

```text
salary
```

Same name.

Spring automatically matches them.

---

# Internal Flow

```text
Database Row

↓

BeanPropertyRowMapper

↓

Reflection

↓

setId()

↓

setName()

↓

setSalary()

↓

Employee Object
```

Notice

Unlike your custom `RowMapper`, this uses **Java Reflection**.

---

# 4. Using BeanPropertyRowMapper

Instead of

```java
jdbcTemplate.query(

        sql,

        new EmployeeRowMapper());
```

Simply write

```java
jdbcTemplate.query(

        sql,

        new BeanPropertyRowMapper<>(Employee.class));
```

Done.

No custom mapper.

---

# 5. query() Example

```java
String sql =

"SELECT * FROM employee";

List<Employee> employees =

jdbcTemplate.query(

        sql,

        new BeanPropertyRowMapper<>(Employee.class));
```

Spring internally creates

```java
Employee employee = new Employee();
```

and automatically calls

```java
setId(...);

setName(...);

setSalary(...);
```

---

# 6. queryForObject()

```java
String sql =

"SELECT * FROM employee WHERE id=?";

Employee employee =

jdbcTemplate.queryForObject(

        sql,

        new BeanPropertyRowMapper<>(Employee.class),

        1);
```

Again,

no custom mapper needed.

---

# 7. How Does Spring Know Which Setter to Call?

Suppose

Database

```text
salary
```

Spring looks for

```java
setSalary(...)
```

Database

```text
name
```

Spring looks for

```java
setName(...)
```

It uses reflection to inspect the class and invoke the appropriate setter methods.

---

# 8. Important Naming Rule

The column names should match the Java property names.

Good

Database

```text
employee_name
```

Java

```java
private String employeeName;
```

Spring can usually map `employee_name` to `employeeName`.

Bad

Database

```text
emp_name
```

Java

```java
private String employeeName;
```

Spring cannot infer that `emp_name` corresponds to `employeeName`.

Use SQL aliases:

```sql
SELECT emp_name AS employeeName
FROM employee;
```

Now Spring can map it correctly.

---

# 9. Internal Working

Suppose

```java
jdbcTemplate.query(

        sql,

        new BeanPropertyRowMapper<>(Employee.class));
```

Flow

```text
JdbcTemplate

↓

Execute SQL

↓

ResultSet

↓

Read Column Names

↓

Reflection

↓

Find Matching Setter

↓

Create Employee Object

↓

Call Setters

↓

Return Employee
```

---

# 10. Custom RowMapper vs BeanPropertyRowMapper

| Feature     | Custom RowMapper | BeanPropertyRowMapper |
| ----------- | ---------------- | --------------------- |
| Code        | More             | Less                  |
| Performance | Faster           | Slightly slower       |
| Reflection  | No               | Yes                   |
| Flexibility | High             | Medium                |
| Best for    | Complex mappings | Simple entities       |

---

# 11. When Should You Use Which?

### Use BeanPropertyRowMapper

When

* Column names match property names
* Simple CRUD
* Less code
* Standard entities

---

### Use Custom RowMapper

When

* Multiple tables are joined
* Column names differ significantly
* Complex object creation is required
* You need custom conversion logic

Example

```sql
SELECT

e.id,

e.name,

d.department_name

FROM employee e

JOIN department d
```

A custom `RowMapper` gives you complete control over how the data is mapped.

---

# Real Project

Repository

```java
@Repository
public class EmployeeRepository {

    @Autowired
    JdbcTemplate jdbcTemplate;

    public List<Employee> findAll(){

        String sql =

        "SELECT * FROM employee";

        return jdbcTemplate.query(

                sql,

                new BeanPropertyRowMapper<>(Employee.class));

    }

}
```

Notice

No `EmployeeRowMapper` class.

---

# Interview Questions

### Q1. What is BeanPropertyRowMapper?

A built-in `RowMapper` implementation that automatically maps database columns to Java bean properties.

---

### Q2. How does BeanPropertyRowMapper work internally?

It uses Java Reflection to create an object and invoke setter methods based on matching column names and property names.

---

### Q3. Is BeanPropertyRowMapper faster than a custom RowMapper?

No.

A custom `RowMapper` is generally a bit faster because it doesn't use reflection.

The performance difference is usually negligible for standard CRUD operations.

---

### Q4. When would you avoid BeanPropertyRowMapper?

When:

* Column names don't match property names.
* You need custom mapping logic.
* You're mapping data from complex joins.
* Performance is extremely critical.

---

### Q5. Does BeanPropertyRowMapper require getters and setters?

Yes.

It relies on standard JavaBean setter methods to populate the object.

---

# Best Practices

* Use `BeanPropertyRowMapper` for simple CRUD.
* Use SQL aliases (`AS`) when column names don't match property names.
* Prefer a custom `RowMapper` for complex mappings or performance-sensitive code.
* Keep entity property names clear and consistent with database columns.

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
│
└── ⏭️ Next
      ResultSetExtractor
           ↓
      NamedParameterJdbcTemplate
           ↓
      Exception Translation
           ↓
      Batch Updates
```

---

## 🎯 Before We Move On

Here's a common interview question:

**Why does Spring provide both `RowMapper` and `BeanPropertyRowMapper`? Why not only `BeanPropertyRowMapper`?**

A strong answer is:

> `BeanPropertyRowMapper` is excellent for simple entity mappings where database column names match Java property names. However, it uses reflection and is less flexible. For complex queries, joins, custom conversions, or performance-critical scenarios, a custom `RowMapper` provides better control and efficiency.

---

### Next Topic

We'll cover **ResultSetExtractor**, which answers another common interview question:

> **What is the difference between `RowMapper` and `ResultSetExtractor`?**

This distinction is frequently asked in Spring interviews because it tests whether you understand how Spring processes query results internally.
