# Spring MVC — Chapter 6: `@RequestBody` and `@ResponseBody`

This is an important bridge between **Spring MVC** and the **REST APIs** you'll build later with Spring Boot.

We'll use:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need `@RequestBody`?

Suppose a client sends this HTTP request:

```http
POST /employees
Content-Type: application/json
```

Body:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Your Java method needs an `Employee` object:

```java
public Employee saveEmployee(Employee employee) {
    ...
}
```

But the HTTP request body is **JSON**, not a Java object.

Something needs to convert:

```text
JSON
  ↓
Employee Java Object
```

That's where `@RequestBody` comes in.

---

# 2. What is `@RequestBody`?

`@RequestBody` tells Spring:

> **Read the HTTP request body and convert it into the Java object expected by this method parameter.**

Example:

```java
@PostMapping("/employees")
public Employee saveEmployee(
        @RequestBody Employee employee) {

    return employeeService.save(employee);
}
```

The client sends:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Spring creates something conceptually like:

```java
Employee employee = new Employee();

employee.setName("Rahul");
employee.setSalary(60000);
```

and passes it to:

```java
saveEmployee(employee);
```

---

# 3. How does Spring convert JSON to Java?

This is very important.

Spring MVC uses an **`HttpMessageConverter`**.

For JSON, Spring Boot commonly configures Jackson.

Conceptually:

```text
HTTP Request Body
      ↓
HttpMessageConverter
      ↓
Jackson
      ↓
Employee Object
      ↓
Controller Method
```

So the controller does **not** manually parse JSON.

You don't need to write:

```java
ObjectMapper mapper = new ObjectMapper();

Employee employee =
        mapper.readValue(json, Employee.class);
```

Spring handles that.

---

# 4. Complete Request Flow

Suppose:

```http
POST /employees
Content-Type: application/json
```

with:

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
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
HandlerAdapter
  ↓
@RequestBody
  ↓
HttpMessageConverter
  ↓
Jackson
  ↓
Employee Object
  ↓
Controller Method
  ↓
Service
  ↓
Repository
```

This connects directly to the concepts we've already learned.

---

# 5. Simple Example

## Employee

```java
public class Employee {

    private Integer id;
    private String name;
    private Double salary;

    // getters and setters
}
```

## Controller

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @PostMapping
    public Employee saveEmployee(
            @RequestBody Employee employee) {

        System.out.println(employee.getName());
        System.out.println(employee.getSalary());

        return employee;
    }
}
```

Request:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Spring creates:

```text
Employee
 ├── name = Rahul
 └── salary = 60000
```

---

# 6. What does `Content-Type` mean?

This header is important:

```http
Content-Type: application/json
```

It tells the server:

> "The request body is JSON."

Spring uses this information when selecting the appropriate message converter.

For example:

```text
application/json
      ↓
Jackson-based converter
```

Other content types can exist:

```text
application/xml
text/plain
multipart/form-data
```

We'll cover file upload and multipart later.

---

# 7. What if the JSON field doesn't exist in Java?

Suppose client sends:

```json
{
  "name": "Rahul",
  "salary": 60000,
  "city": "Hyderabad"
}
```

but your class only has:

```java
private String name;
private Double salary;
```

What happens depends on Jackson configuration, but by default in many Spring Boot setups, unknown JSON properties are not treated as errors.

The `city` value may simply be ignored.

Don't rely on that behavior blindly; API contracts should be intentional.

---

# 8. What if the JSON type is wrong?

Suppose:

```json
{
  "name": "Rahul",
  "salary": "abc"
}
```

but Java expects:

```java
private Double salary;
```

Spring/Jackson cannot normally convert `"abc"` into a `Double`.

The request can fail during deserialization before your controller method executes.

This is one reason validation and proper exception handling are important.

---

# 9. `@ResponseBody`

Now let's look at the opposite direction.

We know:

```text
Request Body

JSON
 ↓
Java Object
```

`@RequestBody` handles the incoming side.

What about:

```text
Java Object
 ↓
JSON Response
```

That's where `@ResponseBody` comes in.

---

# 10. What is `@ResponseBody`?

It tells Spring:

> **Write the return value directly into the HTTP response body instead of treating it as a view name.**

Example:

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    @ResponseBody
    public Employee getEmployee() {

        return new Employee(101, "Rahul", 60000);
    }
}
```

Spring serializes the returned Java object to JSON.

Response:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

---

# 11. Why do we use `@RestController`?

Writing this:

```java
@Controller
@ResponseBody
public class EmployeeController {
}
```

is repetitive.

Spring provides:

```java
@RestController
public class EmployeeController {
}
```

Conceptually:

```text
@RestController
      ≈
@Controller + @ResponseBody
```

So:

```java
@RestController
public class EmployeeController {

    @GetMapping("/employee")
    public Employee getEmployee() {
        return employee;
    }
}
```

The return value goes directly into the HTTP response body.

---

# 12. `@Controller` vs `@RestController`

This is a **very common interview question**.

### `@Controller`

Typically used for traditional Spring MVC applications where methods return view names:

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    public String employee() {

        return "employee";
    }
}
```

Spring interprets `"employee"` as a view name.

---

### `@RestController`

Used for REST APIs:

```java
@RestController
public class EmployeeController {

    @GetMapping("/employee")
    public Employee employee() {

        return employee;
    }
}
```

Spring writes the returned object to the HTTP response body.

---

# 13. `@RequestBody` vs `@ResponseBody`

| Annotation      | Direction       | Purpose                     |
| --------------- | --------------- | --------------------------- |
| `@RequestBody`  | Client → Server | JSON/body → Java object     |
| `@ResponseBody` | Server → Client | Java object → HTTP response |

Think:

```text
@RequestBody

JSON
 ↓
Java
```

```text
@ResponseBody

Java
 ↓
JSON
```

---

# 14. Complete REST Flow

Now combine everything:

```text
POST /employees
      │
      ▼
DispatcherServlet
      │
      ▼
HandlerMapping
      │
      ▼
HandlerAdapter
      │
      ▼
@RequestBody
      │
      ▼
Jackson / HttpMessageConverter
      │
      ▼
Employee Object
      │
      ▼
Controller
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
Employee Object
      │
      ▼
HttpMessageConverter
      │
      ▼
JSON
      │
      ▼
HTTP Response
```

This is the core request/response flow behind many Spring Boot REST APIs.

---

# 15. Where is `@RequestBody` used?

Almost every REST API uses it for create/update operations.

For example:

```java
@PostMapping
public Employee create(
        @RequestBody EmployeeRequest request) {
    ...
}
```

```java
@PutMapping("/{id}")
public Employee update(
        @PathVariable Integer id,
        @RequestBody EmployeeRequest request) {
    ...
}
```

---

# 16. DTO Example

In real projects, you often shouldn't directly expose your database entity as the API request object.

Instead:

```java
public class EmployeeRequest {

    private String name;
    private Double salary;

    // getters/setters
}
```

Controller:

```java
@PostMapping
public EmployeeResponse createEmployee(
        @RequestBody EmployeeRequest request) {

    return employeeService.create(request);
}
```

This becomes especially important when we learn:

* Validation
* DTOs
* JPA
* Security

---

# 17. Interview Questions

### Q1. What does `@RequestBody` do?

> It tells Spring MVC to read the HTTP request body and deserialize it into the Java object expected by the controller method.

### Q2. Which component converts JSON to Java?

> An appropriate `HttpMessageConverter`; in typical Spring Boot JSON applications, Jackson is commonly used underneath.

### Q3. What does `@ResponseBody` do?

> It tells Spring MVC to serialize the return value into the HTTP response body instead of resolving it as a view.

### Q4. Difference between `@Controller` and `@RestController`?

> `@RestController` combines `@Controller` and `@ResponseBody` semantics, making it convenient for REST APIs.

### Q5. What happens if request JSON cannot be converted?

> Deserialization fails before the controller method can successfully receive the object, and Spring generates an appropriate HTTP error response through its exception-handling infrastructure.

---

# 18. Best Practices

Use DTOs for request/response contracts:

```java
@RequestBody EmployeeRequest request
```

instead of exposing internal entities unnecessarily.

Validate incoming data rather than assuming the JSON is correct.

Keep controllers thin:

```text
Controller
   ↓
Service
   ↓
Repository
```

Don't manually parse JSON using `ObjectMapper` inside every controller.

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
├── ✅ Controller & Request Mapping
├── ✅ @GetMapping / @PostMapping / etc.
├── ✅ @PathVariable
├── ✅ @RequestParam
├── ✅ @RequestBody
├── ✅ @ResponseBody
│
└── ⏭️ Next
     Model / ModelAndView
     ViewResolver
     Data Binding
     Validation
     Exception Handling
     Interceptors
```

### Next topic

We'll now cover **`Model`, `ModelAndView`, and `ViewResolver`** so you understand the **traditional MVC side** of Spring MVC—not just REST APIs. This is important because it completes your understanding of why the framework is called **Model–View–Controller**.
