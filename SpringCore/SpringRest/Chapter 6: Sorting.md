# Spring REST — Chapter 6: Sorting

Now we continue with **Sorting**, which naturally follows Pagination.

We'll use:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Sorting?

Suppose:

```http
GET /employees
```

returns:

```text
Rahul
Amit
John
Priya
```

The client may want:

```text
Name ascending:

Amit
John
Priya
Rahul
```

or:

```text
Salary descending:

Priya   90000
Rahul   70000
John    60000
Amit    50000
```

The API needs a way to specify the desired ordering.

---

# 2. What is Sorting?

Sorting means:

> **Ordering the result set according to one or more fields.**

A common REST convention is:

```http
GET /employees?sort=name,asc
```

or:

```http
GET /employees?sort=salary,desc
```

---

# 3. Spring's `Sort`

When using Spring Data, Spring provides:

```java
Sort
```

Example:

```java
Sort sort =
        Sort.by("name").ascending();
```

Or:

```java
Sort sort =
        Sort.by("salary").descending();
```

---

# 4. Sorting with `Pageable`

We already learned:

```java
Pageable pageable =
        PageRequest.of(0, 20);
```

Now add sorting:

```java
Pageable pageable =
        PageRequest.of(
                0,
                20,
                Sort.by("name").ascending()
        );
```

Meaning:

```text
Page = 0
Size = 20
Sort = name ASC
```

---

# 5. REST Request

A client can request:

```http
GET /employees?page=0&size=20&sort=name,asc
```

Conceptually:

```text
page
 ↓
0

size
 ↓
20

sort
 ↓
name ASC
```

In Spring Data-based applications, the MVC layer can bind these request parameters into a `Pageable`.

---

# 6. Descending Sort

```http
GET /employees?sort=salary,desc
```

Equivalent conceptually to:

```java
Sort.by("salary").descending();
```

---

# 7. Multiple Sort Fields

This is important in real applications.

Suppose:

```text
salary
department
name
```

You may want:

```text
salary DESC
name ASC
```

Example:

```java
Sort sort = Sort.by(
        Sort.Order.desc("salary"),
        Sort.Order.asc("name")
);
```

Now the database ordering is conceptually:

```sql
ORDER BY salary DESC, name ASC
```

---

# 8. Why use multiple sort fields?

Suppose 100 employees have:

```text
salary = 60000
```

If you sort only by:

```text
salary ASC
```

their relative order may not be deterministic.

Add a second field:

```text
salary ASC
name ASC
```

Now employees with the same salary are ordered by name.

Even better for stable pagination is to use a unique tie-breaker such as an ID:

```text
salary ASC, id ASC
```

This becomes particularly useful when paging through large datasets.

---

# 9. Pagination + Sorting

In real APIs, these are usually combined.

Example:

```http
GET /employees?page=0&size=20&sort=name,asc
```

Flow:

```text
HTTP Request
     ↓
Pageable
     ├── page = 0
     ├── size = 20
     └── sort = name ASC
     ↓
Repository
     ↓
Database
     ↓
Page<Employee>
```

---

# 10. Controller Example

With Spring Data:

```java
@GetMapping("/employees")
public Page<EmployeeResponse> getEmployees(
        Pageable pageable) {

    return employeeService.getEmployees(pageable);
}
```

Client:

```http
GET /employees?page=0&size=20&sort=name,asc
```

Spring can construct the `Pageable` for you.

---

# 11. Multiple Sort Parameters

You can request multiple sort fields in the query.

For example:

```http
GET /employees?sort=department,asc&sort=name,asc
```

Conceptually:

```text
department ASC
name ASC
```

The exact parameter parsing is handled by Spring Data's pageable argument support.

---

# 12. Internal Flow

Let's say the client sends:

```http
GET /employees?page=1&size=10&sort=salary,desc
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
Pageable Argument Resolution
   ↓
Pageable
   ├── page = 1
   ├── size = 10
   └── sort = salary DESC
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
Page<Employee>
```

---

# 13. Where is Sorting Used?

Almost every list/search API can benefit from sorting:

```text
Employees
Orders
Products
Customers
Transactions
Invoices
Audit Logs
```

Examples:

```http
GET /orders?sort=createdAt,desc
```

```http
GET /products?sort=price,asc
```

```http
GET /employees?sort=name,asc
```

---

# 14. Sorting vs Pagination

They solve different problems.

### Pagination

Answers:

> **How many records should I return?**

```text
page=0
size=20
```

### Sorting

Answers:

> **In what order should I return them?**

```text
sort=name,asc
```

Together:

```text
Pagination → Which records?
Sorting    → What order?
```

---

# 15. Sorting vs Filtering

This distinction is also useful.

### Filtering

```http
GET /employees?department=IT
```

Means:

> Which records should be included?

### Sorting

```http
GET /employees?sort=name,asc
```

Means:

> In what order should the selected records appear?

### Pagination

```http
GET /employees?page=0&size=20
```

Means:

> Which portion of the result should I return?

So:

```text
Filter
  ↓
Sort
  ↓
Paginate
```

The actual database execution strategy may vary, but conceptually this is a useful way to think about the query.

---

# 16. Real API Example

Suppose the requirement is:

> Give me the first 20 IT employees, ordered by highest salary.

Request:

```http
GET /employees
    ?department=IT
    &page=0
    &size=20
    &sort=salary,desc
```

Conceptually:

```text
Filter
department = IT

↓

Sort
salary DESC

↓

Page
0

Size
20
```

This is a very typical enterprise API.

---

# 17. Security Concern: Sort Field

This is an important real-world consideration.

Don't blindly allow arbitrary client input to become database column names.

For example:

```http
GET /employees?sort=someSensitiveColumn
```

Your API should ideally maintain an **allowlist of sortable fields**.

For example:

```text
Allowed:
name
salary
createdAt

Not allowed:
password
internalSecret
```

The exact protection depends on your data-access implementation, but the principle is important:

> **Client-controlled sort fields should be validated.**

---

# 18. Interview Questions

### What is `Sort`?

> `Sort` is a Spring Data abstraction used to represent ordering criteria for query results.

### How do you sort ascending?

```java
Sort.by("name").ascending();
```

### How do you sort descending?

```java
Sort.by("salary").descending();
```

### Can we sort by multiple fields?

Yes.

```java
Sort.by(
    Sort.Order.desc("salary"),
    Sort.Order.asc("name")
);
```

### Can pagination and sorting be used together?

Yes. `Pageable` can contain both page information and sorting information.

### Why use a tie-breaker field?

To make ordering more deterministic when multiple records have the same value for the primary sort field. A unique field such as `id` is a common tie-breaker.

---

# 19. Best Practices

Use clear API parameters:

```http
?page=0&size=20&sort=name,asc
```

Set a reasonable maximum page size.

Allow only approved sort fields.

Use stable sorting for pagination, especially for changing datasets.

Avoid exposing internal database columns directly through sort parameters.

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
├── ✅ Global Error Handling
├── ✅ Pagination
├── ✅ Sorting
│
└── ⏭️ Next
      Filtering / Search
        ↓
      Complete REST API Design
        ↓
      REST Interview Revision
```

## Next topic: Filtering / Search

We'll build APIs such as:

```http
GET /employees?department=IT
```

and:

```http
GET /employees?name=rahul&minSalary=50000
```

Then we'll combine:

```text
Filtering + Sorting + Pagination
```

into a production-style REST endpoint.
