Excellent. We are at the **last major topic in Spring JDBC** before moving to **Spring Transaction Management**.

Current progress:

```text
Spring Framework

✅ Spring Core
✅ Spring AOP

Spring JDBC
│
├── ✅ JdbcTemplate
├── ✅ CRUD Operations
├── ✅ RowMapper
├── ✅ BeanPropertyRowMapper
├── ✅ ResultSetExtractor
├── ✅ NamedParameterJdbcTemplate
├── ✅ Exception Translation
│
└── ⏭️ Batch Updates (Current)
```

---

# Chapter 9: Batch Updates in Spring JDBC

## 1. Why do we need Batch Updates?

Imagine you have to insert 10,000 employees.

Normal approach:

```java
for(Employee emp : employees){

    jdbcTemplate.update(
        sql,
        emp.getName(),
        emp.getSalary()
    );

}
```

What happens?

For every employee:

```
Java Application
        |
        |
        V
Create SQL Request
        |
        V
Database
        |
        V
Execute
        |
        V
Return
```

For 10,000 employees:

```
10,000 database calls
```

This is slow.

---

# Problem

Database communication is expensive.

The biggest cost is not SQL execution.

The biggest cost is:

```
Application
       |
       |
       V
Network
       |
       |
       V
Database
```

Every round trip consumes time.

---

# Solution: Batch Processing

Instead of:

```
Insert Employee 1
Insert Employee 2
Insert Employee 3
...
Insert Employee 10000
```

Send them together:

```
Insert Employee 1
Insert Employee 2
Insert Employee 3
...
Insert Employee 10000

        |
        V

Execute Batch
```

---

# 2. What is Batch Update?

### Definition

Batch update allows multiple SQL statements to be sent to the database and executed together as a batch.

Spring provides:

```java
batchUpdate()
```

---

# 3. Normal update vs Batch update

## Normal update

```java
jdbcTemplate.update(
    sql,
    value1,
    value2
);
```

One SQL execution.

---

## Batch update

```java
jdbcTemplate.batchUpdate(
    sql,
    batchData
);
```

Multiple executions grouped together.

---

# 4. Simple Example

Suppose:

Employee table:

```sql
CREATE TABLE employee(

id INT,

name VARCHAR(50),

salary DOUBLE

);
```

We have:

```java
List<Employee> employees;
```

---

SQL:

```java
String sql =
"INSERT INTO employee(name,salary) VALUES(?,?)";
```

---

Batch:

```java
jdbcTemplate.batchUpdate(
        sql,
        employees,
        employees.size(),
        (ps, employee) -> {

            ps.setString(
                1,
                employee.getName()
            );

            ps.setDouble(
                2,
                employee.getSalary()
            );
        }
);
```

---

# 5. Internal Working

Flow:

```
Employee List

     |
     V

JdbcTemplate.batchUpdate()

     |
     V

PreparedStatement

     |
     V

Add Batch

     |
     V

Execute Batch

     |
     V

Database
```

---

# 6. BatchPreparedStatementSetter

For complex batch operations, Spring provides:

```java
BatchPreparedStatementSetter
```

Interface.

---

Example:

```java
jdbcTemplate.batchUpdate(
        sql,
        new BatchPreparedStatementSetter(){

        @Override
        public void setValues(
             PreparedStatement ps,
             int i)
             throws SQLException {


            Employee emp =
                employees.get(i);


            ps.setString(
                1,
                emp.getName()
            );


            ps.setDouble(
                2,
                emp.getSalary()
            );

        }


        @Override
        public int getBatchSize(){

            return employees.size();

        }

});
```

---

# 7. How does BatchPreparedStatementSetter work?

Two methods:

---

## setValues()

Called for every item.

Example:

List:

```
Employee 1
Employee 2
Employee 3
```

Calls:

```
setValues(0)

setValues(1)

setValues(2)
```

---

## getBatchSize()

Tells Spring:

How many records are there?

Example:

```java
return employees.size();
```

---

# 8. Example Flow

Input:

```
employees = [

 Employee(Rahul,60000),

 Employee(Amit,70000),

 Employee(John,80000)

]
```

Spring does:

```
getBatchSize()

        |
        V

3

        |
        V


setValues(0)

INSERT Rahul


setValues(1)

INSERT Amit


setValues(2)

INSERT John


        |
        V

executeBatch()
```

---

# 9. Return Value of batchUpdate()

Example:

```java
int[] result =
jdbcTemplate.batchUpdate(sql,data);
```

Returns:

```
[
1,
1,
1
]
```

Meaning:

```
Employee 1 inserted

Employee 2 inserted

Employee 3 inserted
```

---

# 10. Real Enterprise Use Cases

Batch updates are commonly used for:

## 1. Bulk Import

Example:

```
CSV File

50000 employees

        |
        V

Batch Insert

        |
        V

Database
```

---

## 2. Data Migration

Moving data:

```
Old Database

        |

Batch Process

        |

New Database
```

---

## 3. Bulk Update

Example:

Increase salary of 10,000 employees.

---

# 11. Batch Update vs Transaction

Important interview point.

Batch update:

```
Improves performance
```

Transaction:

```
Maintains consistency
```

They solve different problems.

Example:

Batch:

```
Insert 100 employees
```

Transaction:

```
Either all 100 insert

OR

Rollback everything
```

---

# 12. Exception Handling in Batch

Suppose:

```
Employee 1  ✓

Employee 2  ✓

Employee 3  ❌

Employee 4  ?
```

What happens depends on transaction configuration.

Without transaction:

Some records may already be inserted.

With transaction:

Everything can rollback.

This connects to our next topic:

# Spring Transaction Management

---

# Interview Questions

## Q1. Why use batchUpdate()?

Answer:

To reduce database round trips and improve performance when executing large numbers of similar SQL operations.

---

## Q2. Difference between update() and batchUpdate()?

| update()                  | batchUpdate()            |
| ------------------------- | ------------------------ |
| Single SQL execution      | Multiple SQL executions  |
| More database calls       | Fewer database calls     |
| Good for small operations | Good for bulk operations |

---

## Q3. Does batchUpdate automatically provide rollback?

No.

Rollback is controlled by transaction management.

---

## Q4. What is BatchPreparedStatementSetter?

A Spring interface used to provide values for each statement in a batch operation.

---

## Q5. Why is PreparedStatement used in batch processing?

Because:

* Better performance
* SQL is compiled once
* Prevents SQL injection

---

# Best Practices

✅ Use batch processing for thousands of records.

✅ Choose reasonable batch sizes.

Example:

Instead of:

```
1 million records
```

one batch:

```
1000 records per batch
```

is usually better.

✅ Combine batch updates with transactions.

✅ Monitor memory usage for very large imports.

---

# Summary

```
Without Batch:

Employee 1 → DB
Employee 2 → DB
Employee 3 → DB
Employee 4 → DB


With Batch:

Employee 1
Employee 2
Employee 3
Employee 4

        |
        V

       DB
```

---

# 📍 Spring JDBC Completed

```
Spring JDBC

✅ JdbcTemplate
✅ CRUD
✅ RowMapper
✅ BeanPropertyRowMapper
✅ ResultSetExtractor
✅ NamedParameterJdbcTemplate
✅ Exception Translation
✅ Batch Updates
```

---

# Next Major Module 🚀

## Spring Transaction Management

This is extremely important for a **2-year Spring Boot developer**.

We'll cover:

```
Why Transactions?

ACID Properties

Transaction Boundaries

Spring Transaction Architecture

PlatformTransactionManager

@Transactional

Propagation

Isolation Levels

Rollback Rules

Checked vs Unchecked Exception Rollback

Transaction Proxy Internals

Real Banking Example

Interview Questions
```

This topic connects:

**Spring JDBC + Spring AOP + Spring Boot**

because `@Transactional` internally works using **Spring AOP proxies**.
