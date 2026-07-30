# Spring MVC — Chapter 3: `HandlerMapping`

We now know:

```text
HTTP Request
    ↓
DispatcherServlet
```

The next question is:

> **How does DispatcherServlet know which controller method should handle the request?**

That's the job of **HandlerMapping**.

We'll use your preferred structure:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need HandlerMapping?

Imagine we have three controllers:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(@PathVariable Integer id) {
        // ...
    }
}
```

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/{id}")
    public Order getOrder(@PathVariable Integer id) {
        // ...
    }
}
```

```java
@RestController
@RequestMapping("/departments")
public class DepartmentController {

    @GetMapping("/{id}")
    public Department getDepartment(@PathVariable Integer id) {
        // ...
    }
}
```

Now the request arrives:

```http
GET /employees/101
```

DispatcherServlet receives it.

But it needs to answer:

```text
Which controller?
Which method?
```

We need a mechanism to map:

```text
HTTP Request
      ↓
Controller Method
```

That is **HandlerMapping**.

---

# 2. What is HandlerMapping?

### Simple definition

> **HandlerMapping is a Spring MVC component that maps an incoming HTTP request to the appropriate handler.**

In normal controller-based applications, the handler is usually a controller method.

Conceptually:

```text
GET /employees/101

        ↓

HandlerMapping

        ↓

EmployeeController.getEmployee(101)
```

---

# 3. Important terminology

Spring MVC uses the word **Handler**.

A handler represents the code that should process the request.

For our level, think:

```text
Handler ≈ Controller method
```

For example:

```java
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable Integer id)
```

This method is a handler for matching requests.

---

# 4. How does Spring know the mapping?

Through annotations such as:

```java
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@DeleteMapping
```

Example:

```java
@GetMapping("/employees/{id}")
```

This provides metadata saying:

> "When a GET request matches `/employees/{id}`, this method can handle it."

Spring discovers and registers these mappings during application startup.

---

# 5. Startup vs Request Time

This distinction is important.

## During startup

Spring scans controllers and discovers mappings.

Conceptually:

```text
EmployeeController
    ↓
@GetMapping("/employees/{id}")
    ↓
Register Mapping
```

It builds an internal mapping structure.

Conceptually:

```text
GET /employees/{id}
        ↓
EmployeeController.getEmployee()
```

---

## During a request

Request arrives:

```text
GET /employees/101
```

DispatcherServlet asks the mapping infrastructure:

> "Which handler matches this request?"

Spring finds:

```text
EmployeeController.getEmployee()
```

Then the next MVC component helps invoke it.

---

# 6. How does `@RequestMapping` work?

Suppose:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(@PathVariable Integer id) {
        // ...
    }
}
```

There are two mapping levels.

### Class level

```java
@RequestMapping("/employees")
```

### Method level

```java
@GetMapping("/{id}")
```

They combine to form:

```text
/employees/{id}
```

So:

```http
GET /employees/101
```

matches the method.

---

# 7. `@GetMapping` is a Specialized `@RequestMapping`

These are related:

```java
@GetMapping("/employees")
```

is essentially a convenient form of:

```java
@RequestMapping(
    path = "/employees",
    method = RequestMethod.GET
)
```

Similarly:

```java
@PostMapping
@PutMapping
@DeleteMapping
@PatchMapping
```

are specialized mappings for specific HTTP methods.

---

# 8. Matching More Than URL

Handler mapping doesn't only consider the path.

A mapping can include:

### HTTP method

```java
@GetMapping
@PostMapping
```

### Path

```java
"/employees/{id}"
```

### Request parameters

```java
@GetMapping(
    value = "/employees",
    params = "type=permanent"
)
```

### Headers

```java
@GetMapping(
    value = "/employees",
    headers = "X-API-VERSION=1"
)
```

### Content type

```java
@PostMapping(
    value = "/employees",
    consumes = "application/json"
)
```

### Response type

```java
@GetMapping(
    value = "/employees",
    produces = "application/json"
)
```

At your experience level, understand the concept; you don't need to memorize every mapping combination.

---

# 9. Real Example

Suppose:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public List<Employee> getAll() {
        // ...
    }

    @GetMapping("/{id}")
    public Employee getById(
            @PathVariable Integer id) {
        // ...
    }

    @PostMapping
    public Employee save(
            @RequestBody Employee employee) {
        // ...
    }
}
```

Spring knows:

```text
GET  /employees
    ↓
getAll()

GET  /employees/{id}
    ↓
getById()

POST /employees
    ↓
save()
```

---

# 10. What happens for `GET /employees/101`?

Complete flow:

```text
Client
  ↓
GET /employees/101
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Match "/employees/{id}"
  ↓
EmployeeController.getById()
  ↓
HandlerAdapter
  ↓
Method executes
```

Notice:

**HandlerMapping finds the method.**

It doesn't normally perform the actual method invocation itself.

That's where **HandlerAdapter** comes in.

---

# 11. HandlerMapping vs HandlerAdapter

This distinction is extremely important.

### HandlerMapping

Answers:

> **"Which handler should process this request?"**

### HandlerAdapter

Answers:

> **"How should I invoke that handler?"**

Think:

```text
Request
  ↓
HandlerMapping
  ↓
"Use EmployeeController.getById()"
  ↓
HandlerAdapter
  ↓
"Okay, I'll invoke it."
```

We'll study HandlerAdapter next.

---

# 12. Common HandlerMapping

In modern annotation-based Spring MVC applications, the important one to know is:

```text
RequestMappingHandlerMapping
```

It handles controller methods mapped using:

```java
@RequestMapping
@GetMapping
@PostMapping
...
```

For your experience level, **know this name**.

You don't need to memorize all of Spring's older HandlerMapping implementations.

---

# 13. Internal Architecture

Conceptually:

```text
                  DispatcherServlet
                         │
                         ▼
              RequestMappingHandlerMapping
                         │
                         ▼
                 Find matching mapping
                         │
                         ▼
                 HandlerMethod
                         │
                         ▼
              HandlerAdapter
                         │
                         ▼
                 Controller Method
```

`HandlerMethod` is an important concept too.

It represents information about the controller method Spring matched.

Think of it as:

```text
Controller Instance
        +
Method
        +
Method Metadata
```

---

# 14. Where is HandlerMapping used?

Every time a Spring MVC request needs to be routed to a controller.

Examples:

```text
GET /employees
POST /employees
GET /employees/101
PUT /employees/101
DELETE /employees/101
```

Spring uses its mapping infrastructure to find the correct handler.

---

# 15. Common Mistake

Don't think:

> "DispatcherServlet scans all controller classes for every request."

That's not the best mental model.

Spring discovers and registers mappings during application initialization, and then uses that mapping infrastructure to resolve requests.

This makes request routing efficient.

---

# 16. Interview Questions

### Q1. What is HandlerMapping?

> HandlerMapping is a Spring MVC component responsible for mapping an incoming HTTP request to the appropriate handler, typically a controller method.

### Q2. Which HandlerMapping is commonly used with `@RequestMapping`?

> `RequestMappingHandlerMapping`.

### Q3. Does HandlerMapping invoke the controller method?

> No. It identifies the appropriate handler. A `HandlerAdapter` is then responsible for invoking the handler.

### Q4. What is the difference between `@RequestMapping` and `@GetMapping`?

> `@GetMapping` is a specialized shortcut for `@RequestMapping` configured for the HTTP GET method.

### Q5. Can mapping depend on more than URL?

> Yes. It can also use HTTP method, request parameters, headers, `consumes`, and `produces` conditions.

---

# 17. Best Practices

Prefer specific mappings:

```java
@GetMapping("/{id}")
```

instead of excessively broad mappings.

Keep related mappings together:

```java
@RequestMapping("/employees")
```

at the controller level.

Use the correct HTTP method:

```text
GET     → Read
POST    → Create
PUT     → Replace/Update
PATCH   → Partial Update
DELETE  → Delete
```

---

# One Diagram to Remember

```text
HTTP Request
     │
     ▼
DispatcherServlet
     │
     ▼
HandlerMapping
     │
     │ "Which method?"
     ▼
HandlerMethod
     │
     ▼
HandlerAdapter
     │
     │ "How do I invoke it?"
     ▼
Controller Method
```

---

# 📍 Where We Are

```text
Spring MVC
│
├── ✅ Why MVC?
├── ✅ MVC Pattern
├── ✅ Front Controller
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
│
└── ⏭️ Next: HandlerAdapter ⭐⭐⭐⭐
       ↓
   How Spring actually invokes
   the controller method
```

The next topic is **HandlerAdapter**, and we'll use a concrete request like:

```http
GET /employees/101
```

to trace exactly how Spring goes from the `HandlerMethod` returned by `HandlerMapping` to the actual Java method invocation.
