# Spring REST — Chapter 1: What is REST and Why Do We Need It?

We are starting the **Spring REST** module from fresh, at your **1.5–2 years experience level**.

We'll follow exactly the structure you requested:

> **Why → What → How → Where → Code → Internal Flow → Interview Questions → Best Practices**

---

# 1. Why do we need REST?

Suppose we have an Employee application.

A browser/mobile app wants employee data.

The client could communicate with our backend using many technologies:

```text
Client
  ↓
Backend
```

Historically, applications used things like:

* JSP
* SOAP
* XML-based web services
* RMI

But modern applications often have many different clients:

```text
Web Application
Mobile App
React App
Angular App
Another Microservice
Third-Party Client
```

They all need to communicate with the backend over HTTP.

We need a simple, standardized way to expose our backend functionality.

That's where **REST** comes in.

---

# 2. What is REST?

REST stands for:

> **Representational State Transfer**

It is an **architectural style** for designing networked applications and APIs.

Important interview point:

> **REST is not a framework.**

REST is an architectural style.

Spring MVC / Spring Boot provides the tools we use to **build REST APIs**.

So:

```text
REST
 ↓
Architectural Style

Spring MVC
 ↓
Web Framework

Spring Boot
 ↓
Simplifies Spring configuration
```

---

# 3. What is a REST API?

A REST API exposes application data and operations as **resources** through HTTP.

For example, an employee is a resource.

We might expose:

```text id="6h6x1q"
GET    /employees
GET    /employees/101
POST   /employees
PUT    /employees/101
PATCH  /employees/101
DELETE /employees/101
```

The important idea is:

> **The URL represents the resource, while the HTTP method represents the operation.**

---

# 4. What is a Resource?

This is a very important REST concept.

Suppose we have:

```text
Employee
Department
Order
Product
Customer
```

These are resources.

We represent them with URIs.

```text id="3utj0t"
Employee       → /employees
One employee   → /employees/101

Order          → /orders
One order      → /orders/5001
```

Think:

```text id="e4kzwq"
Resource
   ↓
URI
```

---

# 5. Why should URLs represent resources?

Bad REST-style design:

```text id="4w8w6o"
/getEmployee
/saveEmployee
/deleteEmployee
/updateEmployee
```

The operation is encoded in the URL.

Better:

```text id="6vjj3w"
/employees
/employees/101
```

Then HTTP methods tell us the action:

```text id="vtdbde"
GET    /employees/101
POST   /employees
PUT    /employees/101
DELETE /employees/101
```

This produces a much cleaner API.

---

# 6. HTTP Methods

These are extremely important for interviews.

| HTTP Method | Common REST Usage |
| ----------- | ----------------- |
| GET         | Read              |
| POST        | Create            |
| PUT         | Replace/update    |
| PATCH       | Partial update    |
| DELETE      | Delete            |

Let's understand them.

---

# 7. GET

Used to retrieve data.

```http
GET /employees
```

Meaning:

> Give me the employees.

One employee:

```http
GET /employees/101
```

Meaning:

> Give me employee 101.

GET is generally considered **safe** and **idempotent**.

We'll discuss those terms shortly.

---

# 8. POST

Used to create a new resource.

```http
POST /employees
```

Request body:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Server creates a new employee.

Example response:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

---

# 9. PUT

PUT is commonly used to replace the representation of an existing resource.

```http
PUT /employees/101
```

Body:

```json
{
  "name": "Rahul",
  "salary": 70000
}
```

Conceptually:

> Replace/update employee 101 with this representation.

---

# 10. PATCH

PATCH is generally used for a **partial update**.

Suppose only salary needs to change:

```http
PATCH /employees/101
```

Body:

```json
{
  "salary": 70000
}
```

Only the specified field is changed.

---

# 11. DELETE

Used to delete a resource.

```http
DELETE /employees/101
```

Meaning:

> Delete employee 101.

---

# 12. Complete CRUD REST Design

This is worth memorizing:

```text id="wtdb1m"
CREATE
POST   /employees

READ
GET    /employees
GET    /employees/{id}

UPDATE
PUT    /employees/{id}
PATCH  /employees/{id}

DELETE
DELETE /employees/{id}
```

---

# 13. What does Spring provide?

We've already learned the MVC infrastructure.

Now we can use Spring's REST annotations:

```java
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@RequestBody
@PathVariable
@RequestParam
```

So REST in Spring is built on the Spring MVC request-processing infrastructure we already learned.

---

# 14. Simple REST Controller

Let's create an Employee REST API.

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public List<Employee> getEmployees() {
        return employeeService.getAllEmployees();
    }

    @GetMapping("/{id}")
    public Employee getEmployee(
            @PathVariable Integer id) {

        return employeeService.getEmployee(id);
    }

    @PostMapping
    public Employee createEmployee(
            @RequestBody EmployeeRequest request) {

        return employeeService.create(request);
    }

    @PutMapping("/{id}")
    public Employee updateEmployee(
            @PathVariable Integer id,
            @RequestBody EmployeeRequest request) {

        return employeeService.update(id, request);
    }

    @DeleteMapping("/{id}")
    public void deleteEmployee(
            @PathVariable Integer id) {

        employeeService.delete(id);
    }
}
```

Notice something important:

We've already learned almost everything used here.

That's why learning MVC first was useful.

---

# 15. How does a REST request work internally?

Suppose the client sends:

```http
GET /employees/101
```

Flow:

```text id="4tqpwq"
Client
   ↓
Tomcat
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
HandlerAdapter
   ↓
EmployeeController
   ↓
EmployeeService
   ↓
EmployeeRepository
   ↓
Database
```

Database result:

```text id="03p0ir"
Employee(101, Rahul, 60000)
```

Then:

```text id="yvypf0"
Employee Object
      ↓
HttpMessageConverter
      ↓
Jackson
      ↓
JSON
      ↓
HTTP Response
```

Response:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

---

# 16. What is `HttpMessageConverter`?

We already touched this in MVC.

In REST applications, it is extremely important.

It converts between:

```text
HTTP Body ↔ Java Object
```

For example:

### Request

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

becomes:

```java
EmployeeRequest
```

And:

```java
EmployeeResponse
```

becomes:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

Jackson is commonly the JSON library used by Spring Boot.

---

# 17. What does `@RestController` do?

We saw earlier:

```java
@RestController
```

Conceptually:

```text
@RestController
     ≈
@Controller
+
@ResponseBody
```

Therefore:

```java
@GetMapping("/{id}")
public Employee getEmployee(...) {
    return employee;
}
```

means the returned `Employee` should be written into the HTTP response body rather than treated as a view name.

---

# 18. REST is Stateless

This is one of the **core REST principles**.

What does stateless mean?

> Each request should contain the information necessary for the server to understand and process it; the server should not depend on conversational client state stored between requests.

For example:

```text id="k26jv7"
Request 1
GET /employees/101

Request 2
GET /employees/102
```

Each request should be independently understandable.

---

# 19. Why is Statelessness Useful?

Because it makes horizontal scaling easier.

Suppose we have:

```text id="b3b6h0"
Load Balancer
      │
 ┌────┼────┐
 ↓    ↓    ↓
App1 App2 App3
```

A request can go to any instance.

There is no need for the server to remember which application instance handled the previous request.

This is particularly important in microservices and distributed systems.

---

# 20. HTTP Status Codes

REST APIs should communicate the result using HTTP status codes.

Important ones for your experience level:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

We'll study each one in detail in the next chapter.

For now, remember:

> **HTTP status code tells the client what happened to the request.**

---

# 21. What is Idempotency?

This is a very common REST interview question.

A request is **idempotent** if making the same request multiple times has the same intended effect as making it once.

Typically:

```text
GET     → Idempotent
PUT     → Idempotent
DELETE  → Idempotent
POST    → Not generally idempotent
PATCH   → Depends on operation
```

Example:

```http
PUT /employees/101
```

with:

```json
{
  "salary": 70000
}
```

Sending it once or multiple times should result in the same final representation.

But:

```http
POST /employees
```

may create a new employee each time, so repeating it can create multiple resources.

---

# 22. Where is REST Used?

REST APIs are everywhere in modern backend development:

```text
React
   ↓
Spring Boot REST API

Mobile App
   ↓
Spring Boot REST API

Another Microservice
   ↓
Spring Boot REST API

External Client
   ↓
Spring Boot REST API
```

REST is especially common for:

* Web applications
* Mobile backends
* Microservices
* Third-party integrations

---

# 23. Interview Questions

### What is REST?

> REST is an architectural style for designing networked applications using stateless interactions and resource-oriented representations, commonly over HTTP.

### Is REST a framework?

> No. REST is an architectural style.

### What is a REST resource?

> A business entity or concept exposed through a URI, such as `/employees/101`.

### Difference between PUT and PATCH?

> PUT is generally used to replace/update a resource representation, while PATCH is intended for partial updates.

### Is POST idempotent?

> No, not by default.

### Is GET idempotent?

> Yes, GET is defined as idempotent and safe under HTTP semantics.

### Why should REST APIs be stateless?

> Statelessness makes requests independently processable and helps with scalability, load balancing, and simpler server architecture.

---

# 24. Best Practices

Use resource-oriented URLs:

```text
/employees
/employees/101
```

Use HTTP methods correctly:

```text
GET    → Read
POST   → Create
PUT    → Replace/Update
PATCH  → Partial Update
DELETE → Delete
```

Return appropriate HTTP status codes.

Use DTOs for request/response contracts.

Keep business logic in the service layer.

Don't return database entities blindly from every API.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ Why REST
├── ✅ What is REST
├── ✅ Resources
├── ✅ HTTP Methods
├── ✅ Statelessness
├── ✅ Idempotency
├── ✅ Spring REST Controller
├── ✅ HttpMessageConverter
│
└── ⏭️ Next
     HTTP Status Codes ⭐⭐⭐⭐⭐
     ↓
     ResponseEntity
     ↓
     Request/Response DTOs
     ↓
     REST API Design
     ↓
     Validation + Exception Handling
```

## Next topic

We'll go deep into **HTTP Status Codes + `ResponseEntity`**, because these determine how your REST API communicates success, validation errors, authentication failures, missing resources, conflicts, and server errors to clients.
