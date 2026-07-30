# Spring MVC — Chapter 4: `HandlerAdapter`

We now know:

```text
HTTP Request
    ↓
DispatcherServlet
    ↓
HandlerMapping
    ↓
HandlerMethod
```

The next question is:

> **Once HandlerMapping finds the controller method, who actually invokes it?**

That's the job of **HandlerAdapter**.

We'll follow your format:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need HandlerAdapter?

Imagine Spring MVC supports different kinds of handlers.

Historically, a handler might be:

```text
Controller object
HttpRequestHandler
Servlet
Annotated controller method
```

These different handlers don't necessarily have the same invocation style.

Spring needed a common mechanism that says:

> "I know how to invoke this particular type of handler."

That's why `HandlerAdapter` exists.

---

# 2. What is HandlerAdapter?

### Simple definition

> **HandlerAdapter is a Spring MVC component that invokes the handler selected by HandlerMapping.**

Remember the distinction:

```text
HandlerMapping
    ↓
Which handler?

HandlerAdapter
    ↓
How do I invoke it?
```

That's the key interview answer.

---

# 3. Real Example

Suppose we have:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(
            @PathVariable Integer id) {

        return service.getEmployee(id);
    }
}
```

Request:

```http
GET /employees/101
```

HandlerMapping finds something conceptually like:

```text
EmployeeController.getEmployee()
```

Now DispatcherServlet asks:

> Which HandlerAdapter can invoke this handler?

For annotation-based controllers, the important adapter is:

```text
RequestMappingHandlerAdapter
```

---

# 4. Complete Flow

```text
Client
   ↓
GET /employees/101
   ↓
Tomcat
   ↓
DispatcherServlet
   ↓
RequestMappingHandlerMapping
   ↓
HandlerMethod
   ↓
RequestMappingHandlerAdapter
   ↓
Controller Method
   ↓
EmployeeService
   ↓
Repository
```

So:

```text
HandlerMapping
    = Find

HandlerAdapter
    = Invoke
```

---

# 5. How does HandlerAdapter invoke the method?

This is where Spring MVC becomes powerful.

Suppose your controller method is:

```java
@GetMapping("/{id}")
public Employee getEmployee(
        @PathVariable Integer id) {
    return service.getEmployee(id);
}
```

The method has:

```java
@PathVariable Integer id
```

Who provides the value `101`?

Your method doesn't explicitly parse the HTTP request.

Spring MVC does it through its **argument resolution infrastructure**.

Conceptually:

```text
HTTP Request
     ↓
/employees/101
     ↓
PathVariable resolver
     ↓
id = 101
     ↓
Invoke getEmployee(101)
```

---

# 6. HandlerAdapter does more than "call the method"

For annotation-based controllers, `RequestMappingHandlerAdapter` coordinates things such as:

* Resolving method arguments
* Invoking the controller method
* Processing return values

For example:

```java
@GetMapping("/employees")
public Employee getEmployee(
        @RequestParam Integer id) {
}
```

Spring needs to extract:

```text
?id=101
```

and provide:

```java
id = 101
```

before invoking the method.

---

# 7. Example with Different Parameters

### `@PathVariable`

```java
@GetMapping("/employees/{id}")
public Employee getEmployee(
        @PathVariable Integer id) {
    ...
}
```

Request:

```text
/employees/101
```

Spring resolves:

```text
id → 101
```

---

### `@RequestParam`

```java
@GetMapping("/employees")
public Employee getEmployee(
        @RequestParam Integer id) {
    ...
}
```

Request:

```text
/employees?id=101
```

Spring resolves:

```text
id → 101
```

---

### `@RequestBody`

```java
@PostMapping("/employees")
public Employee save(
        @RequestBody Employee employee) {
    ...
}
```

Request body:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Spring converts the HTTP body into an `Employee` object using its message conversion infrastructure.

Then the adapter invokes:

```java
save(employee);
```

---

# 8. Internal Flow

For:

```http
POST /employees
```

with JSON:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Flow:

```text
Client
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
EmployeeController.save()
  ↓
RequestMappingHandlerAdapter
  ↓
Resolve @RequestBody
  ↓
HttpMessageConverter
  ↓
JSON → Employee
  ↓
Invoke save(employee)
```

This is why we don't manually write:

```java
ObjectMapper.readValue(...)
```

inside every controller.

Spring MVC handles it.

---

# 9. What about the return value?

Suppose:

```java
@GetMapping("/{id}")
public Employee getEmployee(...) {
    return employee;
}
```

After the controller returns:

```text
Employee Object
     ↓
Return Value Handling
     ↓
HttpMessageConverter
     ↓
JSON
     ↓
HTTP Response
```

For a `@RestController`, this is typically how the response becomes JSON.

For traditional MVC:

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    public String employee() {
        return "employee";
    }
}
```

the return value can be treated as a **view name** and later processed through view resolution.

---

# 10. HandlerAdapter vs DispatcherServlet

This is another common interview question.

### DispatcherServlet

Coordinates the **entire request processing pipeline**.

It asks:

```text
Which handler?
Which adapter?
Invoke it
Process result
Return response
```

### HandlerAdapter

Handles the **actual invocation of the selected handler**.

Think:

```text
DispatcherServlet
    ↓
Coordinator

HandlerAdapter
    ↓
Invoker
```

---

# 11. Common HandlerAdapter

For annotation-driven controllers:

```text
RequestMappingHandlerAdapter
```

is the one you should remember.

Interview answer:

> `RequestMappingHandlerAdapter` is the HandlerAdapter used to invoke controller methods mapped using `@RequestMapping` and its composed annotations such as `@GetMapping` and `@PostMapping`.

---

# 12. Why can't DispatcherServlet invoke every handler directly?

Because Spring MVC supports different handler types.

Instead of putting:

```text
if handler is type A → do A
if handler is type B → do B
if handler is type C → do C
```

inside DispatcherServlet, Spring uses the **Adapter Pattern**.

This follows the **Adapter Design Pattern**.

That's why the component is called:

```text
HandlerAdapter
```

---

# 13. Design Pattern Connection

This is useful for interviews.

Spring MVC uses:

### Front Controller Pattern

```text
DispatcherServlet
```

### Adapter Pattern

```text
HandlerAdapter
```

So:

```text
Front Controller
      ↓
DispatcherServlet

Adapter
      ↓
HandlerAdapter
```

---

# 14. Complete Spring MVC Request Lifecycle

Now let's connect everything we've learned:

```text
HTTP Request
      ↓
Servlet Container (Tomcat)
      ↓
DispatcherServlet
      ↓
HandlerMapping
      ↓
HandlerMethod
      ↓
HandlerAdapter
      ↓
Argument Resolution
      ↓
Controller Method
      ↓
Service
      ↓
Repository
      ↓
Database
      ↓
Controller Return Value
      ↓
Return Value Handling
      ↓
View / HttpMessageConverter
      ↓
HTTP Response
```

This is becoming the central Spring MVC diagram you should memorize.

---

# 15. Interview Questions

### Q1. What is HandlerAdapter?

> HandlerAdapter is a Spring MVC component that invokes the handler selected by HandlerMapping and handles the mechanics required to call it.

### Q2. Difference between HandlerMapping and HandlerAdapter?

> HandlerMapping finds **which handler** should process the request. HandlerAdapter determines **how to invoke that handler**.

### Q3. What HandlerAdapter is commonly used for annotation-based controllers?

> `RequestMappingHandlerAdapter`.

### Q4. Why does Spring use HandlerAdapter?

> To support different types of handlers through a common invocation mechanism, following the Adapter design pattern.

### Q5. Does HandlerAdapter find the controller?

No.

That's the responsibility of **HandlerMapping**.

---

# 16. Best Practices

At your level, you don't configure `HandlerAdapter` manually in normal Spring Boot applications.

Understand its role so you can debug the MVC pipeline and answer interview questions.

Focus on:

```text
DispatcherServlet
    ↓
HandlerMapping
    ↓
HandlerAdapter
    ↓
Controller
```

That's enough for your experience level.

---

# 📍 Where We Are

```text
Spring MVC
│
├── ✅ Why MVC
├── ✅ MVC Pattern
├── ✅ Front Controller
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
│
└── ⏭️ Next
     Controller & Request Mapping ⭐⭐⭐⭐⭐
       ↓
     @RequestMapping
     @GetMapping
     @PostMapping
     @PutMapping
     @DeleteMapping
     @PatchMapping
     @PathVariable
     @RequestParam
```

The next topic is **Controller and Request Mapping**, where we'll start writing actual Spring MVC endpoints and understand exactly how URLs, HTTP methods, path variables, and request parameters map to Java methods.
