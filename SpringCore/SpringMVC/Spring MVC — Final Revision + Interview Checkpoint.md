# Spring MVC — Final Revision + Interview Checkpoint

We've finished the major **Spring MVC topics** for your 1.5–2 years experience level.

Before jumping to Spring REST, let's connect everything into one picture.

## 1. Complete Spring MVC Request Flow

Suppose the client sends:

```http
GET /employees/101
```

The overall flow is:

```text
Client
  ↓
Tomcat / Servlet Container
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
HandlerMethod
  ↓
HandlerAdapter
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
  ↓
Service
  ↓
Controller
  ↓
Response Processing
  ↓
HTTP Response
```

For a traditional MVC application:

```text
Controller
   ↓
Model + View Name
   ↓
ViewResolver
   ↓
View
   ↓
HTML Response
```

For a REST API:

```text
Controller
   ↓
Java Object
   ↓
HttpMessageConverter
   ↓
JSON
   ↓
HTTP Response
```

---

# 2. What You Should Know for Interviews

### DispatcherServlet

> Front Controller of Spring MVC that receives requests and coordinates request processing.

### HandlerMapping

> Determines which handler/controller method should process the request.

### HandlerAdapter

> Invokes the handler selected by HandlerMapping.

### `@RequestMapping`

> Maps HTTP requests to controllers or controller methods.

### `@PathVariable`

```text
/employees/101
```

Gets `101` from the URL path.

### `@RequestParam`

```text
/employees?id=101
```

Gets `101` from a query parameter.

### `@RequestBody`

```json
{
  "name": "Rahul"
}
```

Converts request body into a Java object.

### `@ResponseBody`

Converts the return value into the HTTP response body.

### `@RestController`

Conceptually:

```text
@Controller + @ResponseBody
```

### `Model`

Carries data from controller to a view.

### `ModelAndView`

Contains:

```text
Model + View
```

### `ViewResolver`

Converts a logical view name into the actual view.

### Validation

```java
@Valid
@NotBlank
@NotNull
@Positive
@Email
```

### Exception Handling

```text
@ExceptionHandler
@ControllerAdvice
@RestControllerAdvice
```

### Interceptor

Runs logic around MVC request processing:

```text
preHandle
postHandle
afterCompletion
```

### File Upload

```java
MultipartFile
```

with:

```text
multipart/form-data
```

---

# 3. Most Important Differences

## `@PathVariable` vs `@RequestParam`

```text
/employees/101
       ↓
@PathVariable
```

```text
/employees?id=101
       ↓
@RequestParam
```

---

## `@ModelAttribute` vs `@RequestBody`

```text
Form/query parameters
       ↓
@ModelAttribute
```

```text
JSON/XML request body
       ↓
@RequestBody
```

---

## `@Controller` vs `@RestController`

```text
@Controller
   ↓
View-oriented MVC
```

```text
@RestController
   ↓
REST response body
```

---

## HandlerMapping vs HandlerAdapter

```text
HandlerMapping
→ Which handler?
```

```text
HandlerAdapter
→ How do I invoke it?
```

---

## Filter vs Interceptor vs AOP

```text
Filter
→ Servlet/container level

Interceptor
→ Spring MVC request level

AOP
→ Spring bean/method level
```

---

# 4. One Real Example

Imagine:

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

Controller:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @PostMapping
    public EmployeeResponse create(
            @Valid @RequestBody EmployeeRequest request) {

        return employeeService.create(request);
    }
}
```

Flow:

```text
JSON Request
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
EmployeeRequest
    ↓
@Valid
    ↓
Service
    ↓
Repository
    ↓
Database
    ↓
EmployeeResponse
    ↓
HttpMessageConverter
    ↓
JSON Response
```

At this point, you're already using most of what we've learned.

---

# 5. Spring MVC Interview Questions You Should Be Able to Answer

Before moving ahead, make sure you can explain these without notes:

1. What is Spring MVC?
2. Why do we need DispatcherServlet?
3. What is Front Controller?
4. What does HandlerMapping do?
5. What does HandlerAdapter do?
6. Explain the complete request lifecycle.
7. Difference between `@PathVariable` and `@RequestParam`.
8. Difference between `@RequestBody` and `@ModelAttribute`.
9. Difference between `@Controller` and `@RestController`.
10. What is `HttpMessageConverter`?
11. How does JSON become a Java object?
12. How does a Java object become JSON?
13. What is `Model`?
14. What is `ViewResolver`?
15. Difference between `Model` and `ModelAndView`.
16. What is `@Valid`?
17. Difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`.
18. How does `@RestControllerAdvice` work?
19. Difference between Filter and Interceptor.
20. What is `MultipartFile`?

---

# 📍 Status

```text
Spring Core                  ✅
Spring AOP                   ✅
Spring JDBC                  ✅
Spring Transactions          ✅
Spring MVC                   ✅
```

## Next Module: Spring REST 🚀

Now we'll move from the **MVC framework** to building proper **REST APIs**.

We'll start fresh with:

> **Why REST → What is REST → REST principles → HTTP methods → HTTP status codes → Resource design**

Then we'll build:

```text
GET
POST
PUT
PATCH
DELETE
```

with DTOs, `ResponseEntity`, validation, exception handling, and production-style API design.

This is the part you'll use most often in a real **Spring Boot backend job**.
