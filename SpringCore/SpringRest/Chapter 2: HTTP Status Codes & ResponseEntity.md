# Spring REST — Chapter 2: HTTP Status Codes & `ResponseEntity`

Now we're going deeper into how a REST API communicates with the client.

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need HTTP Status Codes?

Suppose the client calls:

```http
GET /employees/101
```

The server returns:

```json
{
  "id": 101,
  "name": "Rahul"
}
```

The client also needs to know:

> Was the request successful?
> Was the employee missing?
> Was the request invalid?
> Did authentication fail?
> Did the server fail?

That's what **HTTP status codes** communicate.

---

# 2. What is an HTTP Status Code?

An HTTP status code is a three-digit number that tells the client the result of an HTTP request.

The main categories are:

```text
1xx → Informational
2xx → Success
3xx → Redirection
4xx → Client Error
5xx → Server Error
```

For your experience level, focus heavily on **2xx, 4xx, and 5xx**.

---

# 3. Most Important Status Codes

## `200 OK`

Request succeeded.

Example:

```http
GET /employees/101
```

Response:

```http
200 OK
```

```json
{
  "id": 101,
  "name": "Rahul"
}
```

Use `200` when the request completed successfully and you are returning a response body.

---

# 4. `201 Created`

Used when a new resource has been created successfully.

Example:

```http
POST /employees
```

Response:

```http
201 Created
```

```json
{
  "id": 101,
  "name": "Rahul"
}
```

This is more appropriate than always returning `200` for a successful create operation.

---

# 5. `204 No Content`

The request succeeded, but there is no response body.

Common example:

```http
DELETE /employees/101
```

Response:

```http
204 No Content
```

There is nothing to send back.

---

# 6. `400 Bad Request`

The request is invalid.

Examples:

```json
{
  "salary": "abc"
}
```

when salary must be numeric.

Or:

```json
{
  "name": ""
}
```

when the API requires a name.

Typically:

```http
400 Bad Request
```

---

# 7. `401 Unauthorized`

This means the request lacks valid authentication credentials.

For example:

```text
No valid JWT
Expired authentication
Missing authentication credentials
```

Response:

```http
401 Unauthorized
```

Important interview distinction:

> **401 is about authentication.**

---

# 8. `403 Forbidden`

The user is authenticated, but doesn't have permission.

Example:

```text
User is logged in
        ↓
Tries ADMIN API
        ↓
User is not ADMIN
        ↓
403 Forbidden
```

So:

```text
401 → Who are you?
403 → I know who you are, but you're not allowed.
```

This distinction is asked very often.

---

# 9. `404 Not Found`

Resource doesn't exist.

Example:

```http
GET /employees/99999
```

if employee 99999 does not exist.

Response:

```http
404 Not Found
```

This fits perfectly with our earlier:

```java
throw new EmployeeNotFoundException(...)
```

and:

```java
@ExceptionHandler(EmployeeNotFoundException.class)
```

---

# 10. `409 Conflict`

Used when the request conflicts with the current state of the resource.

Example:

```http
POST /employees
```

with an email that already exists.

Response:

```http
409 Conflict
```

Other examples:

* Duplicate unique value
* Resource state conflict
* Version conflict

---

# 11. `500 Internal Server Error`

Unexpected server-side failure.

Example:

```text
NullPointerException
Unexpected infrastructure failure
Unhandled exception
```

The client receives:

```http
500 Internal Server Error
```

Don't expose the actual stack trace to the client.

---

# 12. Quick Status Code Table

| Code | Meaning               | Typical Use                    |
| ---- | --------------------- | ------------------------------ |
| 200  | OK                    | Successful read/update         |
| 201  | Created               | Successful creation            |
| 204  | No Content            | Successful delete/no body      |
| 400  | Bad Request           | Invalid input                  |
| 401  | Unauthorized          | Authentication missing/invalid |
| 403  | Forbidden             | Authenticated but not allowed  |
| 404  | Not Found             | Resource missing               |
| 409  | Conflict              | State/duplicate conflict       |
| 500  | Internal Server Error | Unexpected server failure      |

---

# 13. Why do we need `ResponseEntity`?

Suppose you write:

```java
@GetMapping("/{id}")
public Employee getEmployee(
        @PathVariable Integer id) {

    return service.getEmployee(id);
}
```

Spring decides the response behavior for you.

But sometimes you want explicit control over:

* HTTP status
* Response body
* Headers

That's where:

```java
ResponseEntity<T>
```

comes in.

---

# 14. What is `ResponseEntity`?

> `ResponseEntity` represents the complete HTTP response, including its status code, headers, and body.

Think:

```text
ResponseEntity
   ├── Status
   ├── Headers
   └── Body
```

---

# 15. Simple Example

```java
@GetMapping("/{id}")
public ResponseEntity<Employee> getEmployee(
        @PathVariable Integer id) {

    Employee employee = service.getEmployee(id);

    return ResponseEntity.ok(employee);
}
```

This returns:

```http
200 OK
```

with:

```json
{
  "id": 101,
  "name": "Rahul"
}
```

---

# 16. Returning `201 Created`

```java
@PostMapping
public ResponseEntity<Employee> create(
        @RequestBody EmployeeRequest request) {

    Employee employee = service.create(request);

    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(employee);
}
```

Response:

```http
201 Created
```

---

# 17. Returning `204 No Content`

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(
        @PathVariable Integer id) {

    service.delete(id);

    return ResponseEntity.noContent().build();
}
```

Response:

```http
204 No Content
```

---

# 18. Returning `404 Not Found`

You can do:

```java
@GetMapping("/{id}")
public ResponseEntity<Employee> getEmployee(
        @PathVariable Integer id) {

    Employee employee = service.find(id);

    if (employee == null) {
        return ResponseEntity.notFound().build();
    }

    return ResponseEntity.ok(employee);
}
```

But in real applications, it's often cleaner to throw a custom exception:

```java
throw new EmployeeNotFoundException(
    "Employee not found: " + id
);
```

and let `@RestControllerAdvice` handle it.

That keeps the controller cleaner.

---

# 19. Returning Headers

`ResponseEntity` can also configure headers.

Example:

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("X-Request-Id", "12345")
        .body(employee);
```

Response:

```http
201 Created
X-Request-Id: 12345
```

---

# 20. Why not just return the object?

You can.

```java
public Employee getEmployee() {
    return employee;
}
```

This is perfectly valid.

Use `ResponseEntity` when you need explicit control over the response.

For example:

```text
Need custom status?
        ↓
ResponseEntity

Need custom headers?
        ↓
ResponseEntity

Need conditional responses?
        ↓
ResponseEntity
```

Don't wrap every response in `ResponseEntity` merely because you can.

---

# 21. Complete REST Controller

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public ResponseEntity<EmployeeResponse> getEmployee(
            @PathVariable Integer id) {

        EmployeeResponse employee =
                service.getEmployee(id);

        return ResponseEntity.ok(employee);
    }

    @PostMapping
    public ResponseEntity<EmployeeResponse> create(
            @Valid @RequestBody EmployeeRequest request) {

        EmployeeResponse employee =
                service.create(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(employee);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @PathVariable Integer id) {

        service.delete(id);

        return ResponseEntity
                .noContent()
                .build();
    }
}
```

This is a realistic controller structure.

---

# 22. Internal Flow

For:

```http
POST /employees
```

the flow is:

```text
Client
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
HandlerAdapter
   ↓
@RequestBody
   ↓
Validation
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
ResponseEntity
   ↓
HttpMessageConverter
   ↓
JSON
   ↓
HTTP Response
```

---

# 23. Common Interview Questions

### What is `ResponseEntity`?

> `ResponseEntity` represents the full HTTP response and allows explicit control over status code, headers, and body.

### Difference between returning an object and `ResponseEntity<Object>`?

> Returning an object lets Spring determine the response details based on the normal MVC processing, while `ResponseEntity` gives the developer explicit control over status, headers, and body.

### Difference between 401 and 403?

> `401` means authentication is missing or invalid; `403` means the user is authenticated but doesn't have sufficient permission.

### When should we return 201?

> Typically after successfully creating a new resource.

### When should we return 204?

> When the operation succeeds and there is intentionally no response body, such as a successful delete.

### What status should be used when an employee doesn't exist?

Typically:

```text
404 Not Found
```

---

# 24. Best Practices

Use semantic status codes rather than returning `200` for every outcome.

Prefer:

```text
POST success     → 201
GET success      → 200
DELETE success   → 204
Invalid request  → 400
Missing resource → 404
Conflict         → 409
Unexpected error → 500
```

For authentication/authorization:

```text
Authentication failure → 401
Authorization failure  → 403
```

Don't return internal exception details to clients.

Use a consistent error response structure across the API.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ Why REST
├── ✅ REST Architecture
├── ✅ Resources
├── ✅ HTTP Methods
├── ✅ Statelessness
├── ✅ Idempotency
├── ✅ HttpMessageConverter
├── ✅ HTTP Status Codes
├── ✅ ResponseEntity
│
└── ⏭️ Next
      REST Request/Response DTOs ⭐⭐⭐⭐⭐
      ↓
      Validation
      ↓
      Global Error Response
      ↓
      REST API Design
      ↓
      Pagination / Sorting / Filtering
```

## Next topic

We'll build **proper Request DTOs and Response DTOs** and understand **why you generally shouldn't expose your JPA/entity objects directly through REST APIs**. This is a very practical topic for your experience level and comes up frequently in code reviews and interviews.
