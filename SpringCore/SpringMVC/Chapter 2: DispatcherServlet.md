Great. Let's continue with a fresh **Spring MVC** lesson.

# Chapter 2: DispatcherServlet — Complete Request Flow

We'll use your preferred format:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need DispatcherServlet?

Imagine your application has:

```text
EmployeeController
OrderController
ProductController
UserController
PaymentController
```

And clients send:

```text
GET /employees
GET /orders
GET /products
GET /users
GET /payments
```

Who decides **which controller should handle each request**?

We need a central component that receives requests and dispatches them to the correct handler.

That's the role of **DispatcherServlet**.

---

# 2. What is DispatcherServlet?

> **DispatcherServlet is the Front Controller of Spring MVC.**

It is a Servlet provided by Spring that acts as the central entry point for incoming HTTP requests.

Think:

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
Find correct handler
     ↓
Invoke controller
     ↓
Process response
```

The name itself gives you the idea:

**Dispatcher** → decides where the request should go.

---

# 3. What is the Front Controller Pattern?

Instead of having many entry points:

```text
Request
 ├── EmployeeServlet
 ├── OrderServlet
 ├── UserServlet
 └── ProductServlet
```

we have:

```text
                 Request
                    ↓
             DispatcherServlet
              /      |      \
             ↓       ↓       ↓
        Employee   Order   User
        Controller Controller Controller
```

So all requests enter through one central point.

That's the **Front Controller Pattern**.

---

# 4. How does a request reach DispatcherServlet?

Suppose a client sends:

```http
GET /employees/101
```

The request first reaches the **Servlet container**, such as Tomcat.

Conceptually:

```text
Browser / Postman
       ↓
HTTP Request
       ↓
Tomcat
       ↓
DispatcherServlet
```

Tomcat doesn't directly invoke your `EmployeeController` method.

It invokes the `DispatcherServlet`, and Spring MVC takes over from there.

---

# 5. Complete Request Flow

This is the flow you should understand very well:

```text
Client
  ↓
Tomcat / Servlet Container
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Handler/Controller Method
  ↓
HandlerAdapter
  ↓
Controller Method Executes
  ↓
Service
  ↓
Repository
  ↓
Database
  ↓
Controller Returns Result
  ↓
DispatcherServlet
  ↓
ViewResolver / HttpMessageConverter
  ↓
HTTP Response
```

Don't worry about every component yet. We'll study them one by one.

---

# 6. Step 1 — Client sends request

Example:

```http
GET /employees/101
```

The request contains things like:

```text
Method: GET
Path: /employees/101
Headers
Query Parameters
Body (if applicable)
```

---

# 7. Step 2 — Tomcat receives it

In a Spring Boot application, embedded Tomcat is commonly the servlet container.

Tomcat receives:

```text
GET /employees/101
```

and forwards the request to the registered `DispatcherServlet`.

---

# 8. Step 3 — DispatcherServlet receives it

Now Spring MVC takes control.

Conceptually:

```text
DispatcherServlet
       │
       ▼
"What controller should handle
 /employees/101?"
```

It doesn't directly search Java classes itself in a simplistic way; it uses Spring MVC infrastructure.

The first important component is:

# HandlerMapping

---

# 9. Step 4 — HandlerMapping

## Why?

Suppose you have:

```java
@GetMapping("/employees/{id}")
public Employee getEmployee(...)
```

and:

```java
@GetMapping("/orders/{id}")
public Order getOrder(...)
```

Spring needs to determine which method matches:

```text
/employees/101
```

That's the job of **HandlerMapping**.

---

## What?

> `HandlerMapping` maps an incoming HTTP request to the appropriate handler.

Conceptually:

```text
Request
/employees/101
      ↓
HandlerMapping
      ↓
EmployeeController.getEmployee()
```

We'll study `HandlerMapping` deeply in the next chapter.

---

# 10. Step 5 — HandlerAdapter

Suppose HandlerMapping found:

```java
EmployeeController.getEmployee()
```

But Spring still needs a mechanism to invoke that controller method.

That's where **HandlerAdapter** comes in.

Think:

```text
HandlerMapping
    ↓
"Here is the handler"
    ↓
HandlerAdapter
    ↓
"Okay, I'll invoke it"
```

We'll study this separately too.

---

# 11. Step 6 — Controller Method

Suppose:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(@PathVariable Integer id) {

        return service.getEmployee(id);
    }
}
```

For:

```http
GET /employees/101
```

Spring invokes:

```java
getEmployee(101)
```

---

# 12. Step 7 — Service

The controller should generally **not contain business logic**.

It calls:

```java
@Service
public class EmployeeService {

    public Employee getEmployee(Integer id) {
        return repository.findById(id);
    }
}
```

Flow:

```text
Controller
    ↓
Service
```

---

# 13. Step 8 — Repository

The service calls the data-access layer:

```java
@Repository
public class EmployeeRepository {

    public Employee findById(Integer id) {
        // DB operation
        return employee;
    }
}
```

Flow:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

---

# 14. Step 9 — Data returns

Suppose database returns:

```text
Employee
id = 101
name = Rahul
salary = 60000
```

Then:

```text
Database
   ↓
Repository
   ↓
Service
   ↓
Controller
```

The controller returns the result.

---

# 15. What happens after the controller returns?

This depends on what type of controller you're using.

There are two major cases.

## Traditional MVC

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    public String employee() {
        return "employee";
    }
}
```

`"employee"` is interpreted as a **view name**.

Then:

```text
DispatcherServlet
       ↓
ViewResolver
       ↓
employee.jsp / template
       ↓
HTTP Response
```

---

## REST Controller

```java
@RestController
public class EmployeeController {

    @GetMapping("/employee")
    public Employee employee() {
        return employee;
    }
}
```

Now Spring typically uses an **HttpMessageConverter** (commonly Jackson for JSON) to serialize the Java object into JSON.

Example:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

So:

```text
Employee Object
      ↓
HttpMessageConverter
      ↓
JSON
      ↓
HTTP Response
```

We'll study this deeply in the Spring REST module.

---

# 16. Complete Diagram

Keep this diagram in mind:

```text
                   HTTP REQUEST
                        │
                        ▼
                Servlet Container
                    (Tomcat)
                        │
                        ▼
                DispatcherServlet
                        │
                        ▼
                 HandlerMapping
                        │
                        ▼
                    Handler
                        │
                        ▼
                 HandlerAdapter
                        │
                        ▼
                 Controller Method
                        │
                        ▼
                     Service
                        │
                        ▼
                   Repository
                        │
                        ▼
                    Database
                        │
                        ▼
                 Return to Controller
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        Traditional MVC       REST API
              │                   │
              ▼                   ▼
         ViewResolver      HttpMessageConverter
              │                   │
              ▼                   ▼
             View                JSON
              │                   │
              └─────────┬─────────┘
                        ▼
                  HTTP RESPONSE
```

---

# 17. Where is DispatcherServlet useful?

You generally don't interact with it directly in your application code.

But understanding it is essential for:

* Debugging request routing
* Understanding controller mapping
* Understanding interceptors
* Understanding exception handling
* Understanding REST request processing
* Spring MVC interviews

---

# 18. What does Spring Boot do?

In modern Spring Boot applications, you generally don't manually configure the `DispatcherServlet`.

Boot auto-configures the MVC infrastructure and registers the servlet for you.

So you write:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

and Spring Boot sets up the web infrastructure.

This is why understanding Spring MVC first helps when we later study Spring Boot auto-configuration.

---

# 19. Interview Questions

### What is DispatcherServlet?

> DispatcherServlet is the Front Controller of Spring MVC. It receives incoming HTTP requests and coordinates request processing by finding the appropriate handler, invoking it, and processing the resulting response.

### What is the Front Controller pattern?

> A design pattern where a single centralized component receives and dispatches requests to the appropriate handlers.

### What does HandlerMapping do?

> It determines which handler/controller method should process an incoming request.

### What does HandlerAdapter do?

> It provides the mechanism for DispatcherServlet to invoke the selected handler.

### Does DispatcherServlet execute business logic?

**No.**

It coordinates request processing. Business logic belongs in the service layer.

---

# 20. Best Practices

Keep this architecture:

```text
Controller
    ↓
Service
    ↓
Repository
```

Avoid:

```text
Controller
    ↓
SQL queries ❌
```

and:

```text
Controller
    ↓
Complex business logic ❌
```

Controllers should stay relatively thin.

---

# 📍 Where We Are

```text
Spring MVC — Fresh Start

✅ Why Spring MVC
✅ MVC Pattern
✅ Front Controller Pattern
✅ DispatcherServlet — What & Why
✅ Complete Request Flow — High Level

⏭️ Next:
HandlerMapping ⭐⭐⭐⭐⭐
    ↓
How Spring maps:
GET /employees/101
        ↓
EmployeeController.getEmployee(101)
```

The next topic is **HandlerMapping**, where we'll go deeper into exactly how Spring decides which controller method should handle a URL.
