# Spring REST — Chapter 7: Filtering / Search

Now we'll add the third important capability to our REST list API:

```text
Pagination ✅
Sorting    ✅
Filtering  ← Current
```

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Filtering?

Suppose we have 1 million employees.

The client doesn't want all employees.

They want:

> "Give me only employees from the IT department."

Instead of:

```http
GET /employees
```

we can provide:

```http
GET /employees?department=IT
```

Now the server returns only matching employees.

---

# 2. What is Filtering?

Filtering means:

> **Selecting only the records that satisfy one or more conditions.**

Example:

```text
All Employees
      ↓
department = IT
      ↓
Matching Employees
```

Common filters:

```text
department
name
city
status
salary
createdDate
```

---

# 3. Simple Filtering with `@RequestParam`

```java
@GetMapping("/employees")
public List<EmployeeResponse> getEmployees(
        @RequestParam String department) {

    return employeeService.findByDepartment(department);
}
```

Request:

```http
GET /employees?department=IT
```

Spring binds:

```text
department = "IT"
```

---

# 4. Optional Filtering

Usually, you want the endpoint to work both ways:

```http
GET /employees
```

and:

```http
GET /employees?department=IT
```

So make the parameter optional:

```java
@GetMapping("/employees")
public List<EmployeeResponse> getEmployees(
        @RequestParam(required = false)
        String department) {

    return employeeService.search(department);
}
```

Now:

```text
department = IT
```

when provided, otherwise:

```text
department = null
```

---

# 5. Multiple Filters

Suppose the requirement is:

> Find IT employees earning more than ₹50,000.

Request:

```http
GET /employees?department=IT&minSalary=50000
```

Controller:

```java
@GetMapping("/employees")
public List<EmployeeResponse> search(
        @RequestParam(required = false) String department,
        @RequestParam(required = false) Double minSalary) {

    return employeeService.search(
            department,
            minSalary);
}
```

Now the service can apply:

```text
department = IT
AND
salary >= 50000
```

---

# 6. Search by Name

Suppose:

```http
GET /employees?name=rahul
```

The service might search employees whose name contains `"rahul"`.

For example:

```text
Rahul
Rahul Kumar
Rahul Sharma
```

A typical SQL query could use:

```sql
WHERE LOWER(name) LIKE LOWER(?)
```

with something like:

```text
%rahul%
```

The exact implementation belongs in the repository/data-access layer.

---

# 7. Where should Filtering Logic Live?

Don't do this:

```java
@GetMapping("/employees")
public List<Employee> search(...) {

    // SQL here ❌
}
```

Prefer:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

The controller receives filter parameters.

The service coordinates the business operation.

The repository constructs and executes the database query.

---

# 8. Filtering + Pagination + Sorting

This is where the pieces come together.

Suppose the requirement is:

> "Give me the first 20 IT employees, earning at least ₹50,000, ordered by salary descending."

Request:

```http
GET /employees
    ?department=IT
    &minSalary=50000
    &page=0
    &size=20
    &sort=salary,desc
```

Conceptually:

```text
Request
   ↓
Filtering
department = IT
salary >= ₹50,000
   ↓
Sorting
salary DESC
   ↓
Pagination
page 0
size 20
```

This pattern appears constantly in enterprise REST APIs.

---

# 9. A More Realistic Controller

With Spring Data:

```java
@GetMapping("/employees")
public Page<EmployeeResponse> search(
        @RequestParam(required = false)
        String department,

        @RequestParam(required = false)
        Double minSalary,

        @RequestParam(required = false)
        String name,

        Pageable pageable) {

    return employeeService.search(
            department,
            minSalary,
            name,
            pageable);
}
```

A request like:

```http
GET /employees?department=IT&minSalary=50000&page=0&size=20&sort=salary,desc
```

can be represented by:

```text
Filters
├── department = IT
├── minSalary = 50000
└── name = optional

Pageable
├── page = 0
├── size = 20
└── sort = salary DESC
```

---

# 10. How Does the Database Handle This?

Conceptually, the database query becomes something like:

```sql
SELECT *
FROM employee
WHERE department = ?
  AND salary >= ?
ORDER BY salary DESC
LIMIT ?
OFFSET ?;
```

The actual SQL depends on the persistence technology you're using.

The important idea is:

```text
Filter
   ↓
Sort
   ↓
Page
```

The database should ideally perform these operations rather than loading a huge dataset into Java and filtering it there.

---

# 11. Why Should Filtering Happen in the Database?

Bad approach:

```text
Database
   ↓
Load 1 million employees
   ↓
Java Memory
   ↓
Filter 10 employees
```

Problems:

* High memory usage
* Slow processing
* Large network/database transfer
* Poor scalability

Better:

```text
Database
   ↓
Filter
   ↓
Sort
   ↓
Pagination
   ↓
Return only needed rows
```

This is much more efficient.

---

# 12. Optional Filters and Dynamic Queries

Suppose all of these are optional:

```text
department
name
minSalary
maxSalary
status
city
```

You don't want 50 different repository methods:

```text
findByDepartment()
findByDepartmentAndName()
findByDepartmentAndSalary()
findByDepartmentAndNameAndSalary()
...
```

For more complex search APIs, common approaches include:

* Dynamic query construction
* Specifications
* Criteria API
* Query builders
* QueryDSL

We'll properly learn **Specifications** when we reach Spring Data JPA.

For your current REST level, understand the problem and the concept.

---

# 13. Why not put every filter into the URL path?

Compare:

```http
GET /employees/IT/Hyderabad/50000
```

with:

```http
GET /employees?department=IT&city=Hyderabad&minSalary=50000
```

The second is generally more natural for optional filtering criteria.

A useful rule:

```text
Resource identity
    ↓
Path variable

Optional filtering criteria
    ↓
Query parameters
```

So:

```http
GET /employees/101
```

identifies one employee.

Whereas:

```http
GET /employees?department=IT
```

filters a collection.

---

# 14. Filtering vs PathVariable

This distinction is frequently asked.

### PathVariable

```http
GET /employees/101
```

Means:

> Give me **employee 101**.

### Filter

```http
GET /employees?department=IT
```

Means:

> Give me **employees matching this condition**.

So:

```text
PathVariable
→ identifies a resource

Query parameter
→ filters/modifies the collection query
```

---

# 15. Filtering by Date

A very common real-world requirement:

```http
GET /orders?from=2026-01-01&to=2026-01-31
```

Controller:

```java
@GetMapping("/orders")
public Page<OrderResponse> searchOrders(
        @RequestParam(required = false)
        LocalDate from,

        @RequestParam(required = false)
        LocalDate to,

        Pageable pageable) {

    return service.search(from, to, pageable);
}
```

Spring can convert suitable request parameter strings into Java types such as `LocalDate`.

---

# 16. Filtering by Status

Example:

```http
GET /orders?status=COMPLETED
```

Java:

```java
@GetMapping("/orders")
public Page<OrderResponse> getOrders(
        @RequestParam(required = false)
        OrderStatus status,
        Pageable pageable) {

    return service.findOrders(status, pageable);
}
```

Using an enum is often safer than passing arbitrary strings through the application.

---

# 17. Internal Flow

Let's take:

```http
GET /employees
?department=IT
&minSalary=50000
&page=0
&size=20
&sort=salary,desc
```

Flow:

```text
Client
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
HandlerAdapter
   ↓
@RequestParam + Pageable
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   │
   ├── Filter
   ├── Sort
   └── Pagination
   ↓
Page<Employee>
   ↓
Response DTO
   ↓
JSON
   ↓
Client
```

---

# 18. Security: Don't Allow Arbitrary Filters Blindly

Suppose you expose:

```http
GET /employees?sort=password
```

or arbitrary field names.

That's not a good API design.

Maintain an allowlist:

```text
Allowed filters:
department
status
city
minSalary
maxSalary

Allowed sorting:
name
salary
createdAt
```

Reject unsupported fields.

This also prevents exposing internal database design unnecessarily.

---

# 19. Interview Questions

### What is filtering in REST?

> Filtering means restricting a resource collection based on client-provided criteria, commonly represented through query parameters.

### Where should filtering logic be implemented?

> Request parameters are handled by the controller, business coordination belongs in the service, and the actual database filtering should be performed in the repository/data-access layer.

### Why shouldn't we load all records and filter in Java?

> It wastes memory, increases network transfer, increases processing time, and doesn't scale well.

### PathVariable vs filtering query parameter?

> A path variable identifies a specific resource, while a query parameter commonly filters or modifies a collection query.

### Can filtering be combined with pagination and sorting?

> Yes. In fact, production APIs commonly support all three together.

---

# 20. Best Practices

A good list endpoint should support controlled combinations such as:

```http
GET /employees
    ?department=IT
    &status=ACTIVE
    &page=0
    &size=20
    &sort=name,asc
```

Keep:

```text
Controller → request parameters
Service    → business coordination
Repository → database query
```

Use database-side filtering, sorting, and pagination.

Validate filter values and keep an allowlist for fields that can be filtered or sorted.

Don't expose internal database columns as public API parameters.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ REST Fundamentals
├── ✅ Resources & URI Design
├── ✅ HTTP Methods
├── ✅ Statelessness
├── ✅ Idempotency
├── ✅ HTTP Status Codes
├── ✅ ResponseEntity
├── ✅ Request DTOs
├── ✅ Response DTOs
├── ✅ Validation
├── ✅ Global Exception Handling
├── ✅ Pagination
├── ✅ Sorting
├── ✅ Filtering
│
└── ⏭️ Next
      Complete REST API Design
      ↓
      REST Best Practices
      ↓
      Spring REST Interview Revision
      ↓
      Spring Boot
```

### Next topic

We'll design a **complete production-style Employee REST API** combining everything we've learned:

```text
GET    /employees
GET    /employees/{id}
POST   /employees
PUT    /employees/{id}
PATCH  /employees/{id}
DELETE /employees/{id}

+ DTOs
+ Validation
+ Error Handling
+ Pagination
+ Sorting
+ Filtering
+ ResponseEntity
```

That will serve as our **Spring REST capstone** before moving into **Spring Boot**.
