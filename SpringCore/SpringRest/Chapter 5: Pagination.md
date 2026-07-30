# Spring REST — Chapter 5: Pagination

Now we move into a feature you'll use constantly in real REST APIs:

> **Pagination**

We'll continue with your format:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Pagination?

Suppose your company has:

```text
5,000,000 employees
```

Client calls:

```http
GET /employees
```

Should the server return all 5 million employees?

Absolutely not.

Problems:

```text
Huge database query
        ↓
Huge memory usage
        ↓
Large network response
        ↓
Slow API
        ↓
Slow client
```

Instead, return a small portion.

For example:

```http
GET /employees?page=0&size=20
```

Meaning:

> Give me the first 20 employees.

Then:

```http
GET /employees?page=1&size=20
```

returns the next 20.

---

# 2. What is Pagination?

> **Pagination divides a large result set into smaller chunks called pages.**

For example:

```text
100 employees
```

with:

```text
size = 20
```

gives:

```text
Page 0 → Employees 1–20
Page 1 → Employees 21–40
Page 2 → Employees 41–60
Page 3 → Employees 61–80
Page 4 → Employees 81–100
```

---

# 3. Why is Pagination important?

It improves:

* API response time
* Database efficiency
* Memory usage
* Network usage
* User experience

This is especially important in production systems.

---

# 4. Basic REST API Design

A common design is:

```http
GET /employees?page=0&size=20
```

Here:

```text id="3ak5zq"
page = 0
size = 20
```

We can also support:

```http
GET /employees?page=2&size=50
```

Meaning:

> Return page 2 with 50 records.

---

# 5. What are Page and Size?

### Page

Which page do you want?

```text
page=0
```

means first page.

```text
page=1
```

means second page.

In many Spring Data APIs, pages are **zero-based**.

---

### Size

How many records should be returned?

```text
size=20
```

means:

> Maximum 20 records on this page.

---

# 6. Simple Controller

If you're handling pagination manually:

```java
@GetMapping("/employees")
public List<EmployeeResponse> getEmployees(
        @RequestParam int page,
        @RequestParam int size) {

    return service.getEmployees(page, size);
}
```

Request:

```http
GET /employees?page=0&size=20
```

Controller receives:

```text
page = 0
size = 20
```

---

# 7. How does the database paginate?

Conceptually, for SQL databases, you want something like:

```sql
SELECT *
FROM employee
LIMIT 20 OFFSET 0;
```

Next page:

```sql
SELECT *
FROM employee
LIMIT 20 OFFSET 20;
```

Next:

```sql
SELECT *
FROM employee
LIMIT 20 OFFSET 40;
```

The exact pagination syntax varies by database.

---

# 8. The Problem with Very Large OFFSET

Suppose:

```text
page = 100000
size = 20
```

That means a huge offset.

Some databases may need to scan/skip many rows before returning the requested page.

For very large datasets, **cursor/keyset pagination** can be more efficient.

For your current level, just know:

```text
Offset Pagination
→ Simple and common

Keyset/Cursor Pagination
→ Better for very large/high-throughput datasets
```

We'll revisit this in microservices and performance discussions.

---

# 9. Spring Data `Pageable`

Now we get into the Spring ecosystem.

When you use **Spring Data JPA**, you can use:

```java
Pageable
```

Example:

```java
@GetMapping("/employees")
public Page<EmployeeResponse> getEmployees(
        Pageable pageable) {

    return service.getEmployees(pageable);
}
```

Then the request:

```http
GET /employees?page=0&size=20
```

can be converted into a `Pageable`.

---

# 10. What is `Pageable`?

> `Pageable` is an abstraction representing pagination and sorting information.

It can contain:

```text
Page number
Page size
Sort information
```

Think:

```text
Pageable
├── page
├── size
└── sort
```

---

# 11. `PageRequest`

You can also create one manually:

```java
Pageable pageable =
        PageRequest.of(0, 20);
```

Meaning:

```text
Page = 0
Size = 20
```

With sorting:

```java
Pageable pageable =
        PageRequest.of(
            0,
            20,
            Sort.by("name").ascending()
        );
```

---

# 12. What is `Page<T>`?

Spring Data can return a:

```java
Page<Employee>
```

instead of only:

```java
List<Employee>
```

The `Page` contains not only employee data, but also pagination metadata.

Conceptually:

```text
Page<Employee>
├── content
├── page number
├── page size
├── total elements
├── total pages
├── first?
├── last?
└── hasNext?
```

This is extremely useful to clients.

---

# 13. Example Response

A paginated API might return something conceptually like:

```json
{
  "content": [
    {
      "id": 101,
      "name": "Rahul"
    },
    {
      "id": 102,
      "name": "Amit"
    }
  ],
  "page": {
    "number": 0,
    "size": 20,
    "totalElements": 125,
    "totalPages": 7
  }
}
```

The exact JSON shape depends on how you expose `Page` or define your own response DTO.

---

# 14. Why is `Page` better than `List`?

Suppose you return:

```java
List<Employee>
```

The client sees employee data.

But it doesn't automatically know:

```text
How many total records?
How many pages?
Is there another page?
```

`Page` gives you this information.

---

# 15. Internal Flow with Spring Data

When you later use Spring Data JPA:

```text
GET /employees?page=0&size=20
       ↓
Spring MVC
       ↓
Pageable
       ↓
Service
       ↓
Repository
       ↓
Database
       ↓
Page<Employee>
       ↓
JSON Response
```

The repository may look like:

```java
Page<Employee> findAll(Pageable pageable);
```

Spring Data generates the appropriate query/pagination behavior.

We'll learn this properly when we reach **Spring Data JPA**.

---

# 16. Pagination + Sorting

Pagination and sorting usually go together.

Example:

```http
GET /employees?page=0&size=20&sort=name,asc
```

Meaning:

```text
Page     = 0
Size     = 20
Sort     = name ascending
```

Descending:

```http
GET /employees?page=0&size=20&sort=salary,desc
```

---

# 17. Why should sorting be explicit?

Suppose the database has:

```text
Rahul
Amit
John
```

Without an explicit ordering, you shouldn't assume the database will always return rows in the same order.

For stable pagination, use a deterministic sort.

For example:

```text
salary ASC, id ASC
```

Using a unique tie-breaker such as `id` is especially useful when many rows have the same sort value.

---

# 18. Filtering + Pagination

Real APIs often combine:

```http
GET /employees
    ?department=IT
    &page=0
    &size=20
    &sort=name,asc
```

Meaning:

```text
Filter → department = IT
Page   → 0
Size   → 20
Sort   → name ASC
```

Flow:

```text
Request
  ↓
Filter
  ↓
Sort
  ↓
Pagination
  ↓
Database
```

We'll cover filtering more deeply after sorting.

---

# 19. Where is Pagination Used?

Almost everywhere:

```text
Employee List
Order History
Products
Customers
Transactions
Audit Logs
Search Results
Reports
```

Example:

```http
GET /orders?page=0&size=50
```

is much better than:

```http
GET /orders
```

returning millions of records.

---

# 20. Interview Questions

### What is pagination?

> Pagination divides a large result set into smaller pages to reduce database, memory, and network overhead.

### What does `page=0&size=20` mean?

> Return the first page with up to 20 records. In Spring Data, page numbering is zero-based.

### What is `Pageable`?

> `Pageable` is an abstraction representing page number, page size, and sorting information.

### What is `Page<T>`?

> It represents a page of data along with pagination metadata such as total elements and total pages.

### Difference between `Page` and `List`?

> `List` contains only the records, while `Page` also contains pagination metadata.

### Why shouldn't we return millions of records from one API?

> It increases database load, memory consumption, network payload size, and response time.

---

# 21. Best Practices

For production APIs:

```text
Always define a reasonable default page size.
```

Don't allow a client to request:

```http
size=10000000
```

without a limit.

For example, you might enforce:

```text
default size = 20
maximum size = 100
```

Also:

* Sort deterministically.
* Index columns commonly used for filtering/sorting.
* Avoid huge offsets for very large datasets.
* Consider cursor/keyset pagination when offset pagination becomes inefficient.

---

# 22. Important Distinction

For your current roadmap:

```text
Spring MVC
→ Handles HTTP request parameters

Spring Data
→ Provides Pageable, Page, repository pagination, etc.
```

So when we use:

```java
Pageable pageable
```

in a typical Spring Boot project, we're combining **Spring MVC's parameter binding** with **Spring Data's pagination abstraction**.

You'll see this much more clearly when we reach JPA.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ REST Fundamentals
├── ✅ Resources & URIs
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
│
└── ⏭️ Next
      Sorting
        ↓
      Filtering / Search
        ↓
      REST API Design
        ↓
      Spring MVC/REST Revision
```

## Next: Sorting

We'll learn:

```http
GET /employees?sort=name,asc
```

and:

```http
GET /employees?sort=salary,desc
```

including **why sorting is important for pagination**, how Spring handles `Sort`, multiple sort fields, and the common interview questions.
