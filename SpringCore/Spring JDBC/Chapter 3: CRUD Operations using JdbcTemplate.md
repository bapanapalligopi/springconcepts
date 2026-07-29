Excellent. Now we'll build a **real Spring JDBC application**.

This is one of the most important topics for a **2-year Java/Spring Boot developer** because almost every CRUD application follows this architecture.

As promised, we'll follow:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# Chapter 3: CRUD Operations using JdbcTemplate

---

# 1. Why CRUD?

Almost every enterprise application performs these operations:

```text
Employee Management

Create Employee

Read Employee

Update Employee

Delete Employee
```

These are called **CRUD** operations.

---

# 2. Project Architecture

We'll build this application.

```text
Client
   │
   ▼
EmployeeController
   │
   ▼
EmployeeService
   │
   ▼
EmployeeRepository
   │
   ▼
JdbcTemplate
   │
   ▼
DataSource
   │
   ▼
MySQL Database
```

Notice:

Every layer has one responsibility.

| Layer        | Responsibility                |
| ------------ | ----------------------------- |
| Controller   | Handles HTTP requests         |
| Service      | Business Logic                |
| Repository   | Database Operations           |
| JdbcTemplate | Executes SQL                  |
| DataSource   | Provides database connections |

---

# 3. Database Table

Suppose we have

```sql
CREATE TABLE employee(

    id INT PRIMARY KEY AUTO_INCREMENT,

    name VARCHAR(100),

    salary DOUBLE

);
```

---

# 4. Employee Class

```java
public class Employee {

    private Integer id;

    private String name;

    private Double salary;

    // getters and setters
}
```

This represents one row from the database.

---

# 5. Configure DataSource

## Why?

`JdbcTemplate` needs a database connection.

Instead of manually creating:

```java
DriverManager.getConnection(...)
```

Spring uses a **DataSource**.

---

### application.properties

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/company

spring.datasource.username=root

spring.datasource.password=root

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

Spring Boot automatically creates:

```text
DataSource Bean

↓

JdbcTemplate Bean
```

No configuration class is required.

---

# 6. Repository

```java
@Repository
public class EmployeeRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

}
```

Question:

Who creates JdbcTemplate?

Answer:

Spring IoC Container.

---

# 7. CREATE Operation

## Why?

Insert a new employee.

SQL

```sql
INSERT INTO employee(name,salary)

VALUES(?,?)
```

Repository

```java
@Repository
public class EmployeeRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public int save(Employee employee) {

        String sql =
        "INSERT INTO employee(name,salary) VALUES(?,?)";

        return jdbcTemplate.update(
                sql,
                employee.getName(),
                employee.getSalary());

    }

}
```

---

## Internal Working

```text
save(employee)

↓

JdbcTemplate.update()

↓

DataSource

↓

Connection

↓

PreparedStatement

↓

Set Parameters

↓

Execute SQL

↓

Close Resources

↓

Return Rows Affected
```

---

# 8. UPDATE Operation

Suppose salary changes.

SQL

```sql
UPDATE employee

SET salary=?

WHERE id=?
```

Repository

```java
public int update(Employee employee){

    String sql=

    "UPDATE employee SET salary=? WHERE id=?";

    return jdbcTemplate.update(

            sql,

            employee.getSalary(),

            employee.getId());

}
```

Returns

```text
1
```

Meaning

One row updated.

---

# 9. DELETE Operation

SQL

```sql
DELETE FROM employee

WHERE id=?
```

Repository

```java
public int delete(Integer id){

    String sql=

    "DELETE FROM employee WHERE id=?";

    return jdbcTemplate.update(sql,id);

}
```

Again

`update()` is used.

Because

INSERT

UPDATE

DELETE

all modify data.

---

# 10. READ One Employee

SQL

```sql
SELECT *

FROM employee

WHERE id=?
```

Question

Which JdbcTemplate method?

Answer

```java
queryForObject()
```

Repository

```java
public Employee findById(Integer id){

    String sql=

    "SELECT * FROM employee WHERE id=?";

    return jdbcTemplate.queryForObject(

            sql,

            new EmployeeRowMapper(),

            id);

}
```

Notice

Need

```java
EmployeeRowMapper
```

because SQL returns

```text
Rows

↓

Java Object
```

Someone must convert it.

---

# 11. READ All Employees

SQL

```sql
SELECT *

FROM employee
```

Repository

```java
public List<Employee> findAll(){

    String sql=

    "SELECT * FROM employee";

    return jdbcTemplate.query(

            sql,

            new EmployeeRowMapper());

}
```

Return

```java
List<Employee>
```

---

# 12. What is RowMapper?

This is the next important topic.

For now,

think of it as

```text
Database Row

↓

Employee Object
```

Example

Database

| id | name  | salary |
| -- | ----- | ------ |
| 1  | Rahul | 50000  |

↓

Java

```java
Employee employee

↓

id=1

↓

name="Rahul"

↓

salary=50000
```

---

# Internal Flow

Suppose

```java
findById(1)
```

Flow

```text
Controller

↓

Service

↓

Repository

↓

JdbcTemplate

↓

Execute SQL

↓

ResultSet

↓

RowMapper

↓

Employee Object

↓

Return
```

---

# Service Layer

```java
@Service
public class EmployeeService {

    @Autowired

    EmployeeRepository repository;

    public Employee getEmployee(Integer id){

        return repository.findById(id);

    }

}
```

---

# Controller

```java
@RestController

@RequestMapping("/employees")

public class EmployeeController {

    @Autowired

    EmployeeService service;

    @GetMapping("/{id}")

    public Employee find(

            @PathVariable Integer id){

        return service.getEmployee(id);

    }

}
```

---

# Complete Flow

Suppose

```text
GET /employees/1
```

Flow

```text
Browser

↓

Controller

↓

Service

↓

Repository

↓

JdbcTemplate

↓

Database

↓

ResultSet

↓

RowMapper

↓

Employee

↓

JSON Response
```

---

# Why is RowMapper Needed?

Question

Database returns

```text
Rows
```

Java wants

```java
Employee
```

Who converts?

RowMapper.

Without RowMapper

Spring has no idea how to convert

```text
ID

NAME

SALARY
```

into

```java
Employee
```

---

# Interview Questions

### Q1. Which JdbcTemplate method is used for INSERT?

```java
update()
```

---

### Q2. Which method is used for UPDATE?

```java
update()
```

---

### Q3. Which method is used for DELETE?

```java
update()
```

---

### Q4. Which method returns one object?

```java
queryForObject()
```

---

### Q5. Which method returns multiple objects?

```java
query()
```

---

### Q6. Why do we need RowMapper?

To convert each row of a `ResultSet` into a Java object.

---

### Q7. Does JdbcTemplate automatically create connections?

No. It obtains connections from the configured `DataSource`, executes the SQL, and ensures resources are released afterward.

---

# Best Practices

* Use parameter placeholders (`?`) to avoid SQL injection.
* Keep SQL in the repository layer.
* Return domain objects (`Employee`), not `ResultSet`.
* Let `JdbcTemplate` manage resources.
* Separate business logic (service) from data access (repository).

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
│
└── ⏭️ Next
      RowMapper
          ↓
      BeanPropertyRowMapper
          ↓
      ResultSetExtractor
          ↓
      NamedParameterJdbcTemplate
          ↓
      Exception Translation
```

## 🚀 Next Topic: **RowMapper (Deep Dive)**

This is one of the **most frequently asked Spring JDBC interview topics**.

We'll answer:

* Why is `RowMapper` needed?
* How does `JdbcTemplate` use it internally?
* How do you write a custom `RowMapper`?
* What is the difference between `RowMapper`, `BeanPropertyRowMapper`, and `ResultSetExtractor`?
* When should you use each one in a real project?

Understanding `RowMapper` is the key to understanding how query results become Java objects.
