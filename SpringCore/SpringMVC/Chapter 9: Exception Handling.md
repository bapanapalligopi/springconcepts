# Spring MVC — Chapter 9: Exception Handling

Now we learn one of the **most important practical Spring MVC topics**.

In real Spring Boot projects, exceptions are expected. The important part is:

> **How do we handle them cleanly and return a proper response to the client?**

We'll use your preferred structure:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Exception Handling?

Suppose we have:

```java
@GetMapping("/employees/{id}")
public Employee getEmployee(@PathVariable Integer id) {

    return employeeService.getEmployee(id);
}
```

What happens if employee `101` doesn't exist?

The service might throw:

```java
throw new EmployeeNotFoundException("Employee not found");
```

If we don't handle it properly, the client may receive an ugly generic error response.

We want something meaningful:

```json
{
  "status": 404,
  "message": "Employee not found",
  "timestamp": "..."
}
```

So exception handling gives us a way to:

* Handle failures
* Return proper HTTP status codes
* Create consistent error responses
* Avoid duplicate `try-catch` blocks
* Keep controllers clean

---

# 2. What is Exception Handling in Spring MVC?

Spring MVC provides several mechanisms.

The important ones for your experience level are:

```text
@ExceptionHandler
@ControllerAdvice
@RestControllerAdvice
```

Think of them as:

```text
@ExceptionHandler
    ↓
Handle specific exception

@ControllerAdvice
    ↓
Handle exceptions globally for controllers

@RestControllerAdvice
    ↓
Global REST exception handling
```

---

# 3. First: `@ExceptionHandler`

## Why?

Suppose one controller has:

```java
@GetMapping("/{id}")
public Employee getEmployee(
        @PathVariable Integer id) {

    return service.getEmployee(id);
}
```

and `EmployeeNotFoundException` can be thrown.

You can handle it inside the same controller.

---

## What?

`@ExceptionHandler` tells Spring:

> **If this controller throws this exception, use this method to handle it.**

Example:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping("/{id}")
    public Employee getEmployee(
            @PathVariable Integer id) {

        return service.getEmployee(id);
    }

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<String> handleEmployeeNotFound(
            EmployeeNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ex.getMessage());
    }
}
```

Now if:

```java
throw new EmployeeNotFoundException("Employee not found");
```

occurs:

```text
Controller Method
      ↓
Exception
      ↓
@ExceptionHandler
      ↓
404 Response
```

---

# 4. Why isn't `try-catch` enough?

You could write:

```java
@GetMapping("/{id}")
public ResponseEntity<?> getEmployee(
        @PathVariable Integer id) {

    try {

        return ResponseEntity.ok(
                service.getEmployee(id));

    } catch (EmployeeNotFoundException e) {

        return ResponseEntity
                .status(404)
                .body(e.getMessage());
    }
}
```

This works.

But imagine:

```text
50 Controllers
500 Methods
```

You end up with:

```text
try-catch
try-catch
try-catch
try-catch
...
```

Lots of duplicated error-handling code.

Spring provides **global exception handling** to solve this.

---

# 5. `@ControllerAdvice`

## What?

`@ControllerAdvice` lets you define exception-handling logic that applies across multiple controllers.

Example:

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<String> handleEmployeeNotFound(
            EmployeeNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(ex.getMessage());
    }
}
```

Now you don't need the `@ExceptionHandler` inside every controller.

---

# 6. How does it work?

Suppose:

```text
EmployeeController
OrderController
PaymentController
```

Any of them throws:

```text
EmployeeNotFoundException
```

Spring can route the exception to the global advice.

Flow:

```text
Controller
    ↓
Exception
    ↓
DispatcherServlet Exception Handling
    ↓
@ControllerAdvice
    ↓
@ExceptionHandler
    ↓
HTTP Response
```

---

# 7. `@RestControllerAdvice`

For REST APIs, you'll commonly see:

```java
@RestControllerAdvice
```

instead of:

```java
@ControllerAdvice
```

Conceptually:

```text
@RestControllerAdvice
      ≈
@ControllerAdvice
+
@ResponseBody semantics
```

This is convenient for JSON error responses.

---

# 8. Real REST Example

Suppose:

```java
public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

Global handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleEmployeeNotFound(
            EmployeeNotFoundException ex) {

        ErrorResponse error = new ErrorResponse(
                404,
                ex.getMessage());

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }
}
```

---

# 9. ErrorResponse DTO

```java
public class ErrorResponse {

    private int status;
    private String message;

    public ErrorResponse(int status,
                         String message) {
        this.status = status;
        this.message = message;
    }

    // getters/setters
}
```

Client receives:

```json
{
  "status": 404,
  "message": "Employee not found"
}
```

This is much cleaner.

---

# 10. Internal Flow

Let's trace:

```http
GET /employees/101
```

Suppose employee doesn't exist.

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
HandlerMapping
     ↓
Controller
     ↓
Service
     ↓
EmployeeNotFoundException
     ↓
Exception Resolution
     ↓
@RestControllerAdvice
     ↓
@ExceptionHandler
     ↓
ErrorResponse
     ↓
JSON
     ↓
HTTP 404
```

This is the important mental model.

---

# 11. Handling Different Exceptions

One handler can handle one exception:

```java
@ExceptionHandler(EmployeeNotFoundException.class)
public ResponseEntity<ErrorResponse> handleEmployeeNotFound(...) {
    ...
}
```

Another can handle:

```java
@ExceptionHandler(IllegalArgumentException.class)
```

Another:

```java
@ExceptionHandler(DataAccessException.class)
```

You can centralize different error types in one class.

---

# 12. Generic Exception Handler

You can also catch unexpected exceptions:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handleGenericException(
        Exception ex) {

    ErrorResponse error =
            new ErrorResponse(
                    500,
                    "Internal server error");

    return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
}
```

But be careful:

> Don't expose internal exception details or stack traces to API clients.

---

# 13. Validation Exceptions

This is very important because we just learned validation.

Suppose:

```java
@PostMapping
public EmployeeResponse create(
        @Valid @RequestBody EmployeeRequest request) {

    return service.create(request);
}
```

and input is invalid.

Spring may throw:

```text
MethodArgumentNotValidException
```

Your global handler can handle it:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException ex) {

    ...
}
```

Then return something like:

```json
{
  "status": 400,
  "message": "Validation failed"
}
```

A more useful API often returns field-level errors:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "name": "Name is required",
    "salary": "Salary must be positive"
  }
}
```

---

# 14. `ResponseEntity`

You'll see this constantly.

```java
ResponseEntity
```

allows you to control:

* HTTP status
* Headers
* Response body

Example:

```java
return ResponseEntity
        .status(HttpStatus.NOT_FOUND)
        .body(error);
```

Or:

```java
return ResponseEntity.ok(employee);
```

---

# 15. Where should Exceptions be handled?

A useful architecture is:

```text
Controller
    ↓
Service
    ↓
Repository
```

Exceptions can originate anywhere:

```text
Repository
   ↓
Service
   ↓
Controller
```

You usually allow them to propagate until your **central exception handler** converts them into the appropriate API response.

That keeps business code cleaner.

---

# 16. Custom Exceptions

Real projects usually define application-specific exceptions.

Example:

```java
public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

Service:

```java
public Employee getEmployee(Integer id) {

    Employee employee =
            repository.findById(id);

    if (employee == null) {
        throw new EmployeeNotFoundException(
                "Employee not found: " + id);
    }

    return employee;
}
```

Global handler:

```java
@ExceptionHandler(EmployeeNotFoundException.class)
public ResponseEntity<ErrorResponse>
handleEmployeeNotFound(
        EmployeeNotFoundException ex) {

    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(
                new ErrorResponse(
                    404,
                    ex.getMessage()
                )
            );
}
```

This gives a clean separation:

```text
Service
  ↓
Business error

Global Handler
  ↓
HTTP response
```

---

# 17. Exception Handling vs Business Logic

Don't do this:

```java
try {
    // huge business logic
} catch (...) {
    // HTTP response logic
}
```

Instead:

```text
Business Layer
    ↓
Throw meaningful exception
    ↓
Global Exception Handler
    ↓
Convert to HTTP response
```

This keeps the service layer independent of HTTP concerns.

---

# 18. `@ExceptionHandler` vs `@ControllerAdvice`

Very common interview question.

### `@ExceptionHandler`

Usually handles an exception for a particular controller.

```text
One Controller
```

### `@ControllerAdvice`

Provides cross-controller exception handling.

```text
Many Controllers
```

---

# 19. `@ControllerAdvice` vs `@RestControllerAdvice`

### `@ControllerAdvice`

Used for shared MVC/controller concerns; if returning a response body, you may use `@ResponseBody` on the handler method.

### `@RestControllerAdvice`

Designed for REST APIs and applies response-body semantics automatically.

For modern Spring Boot REST APIs:

```java
@RestControllerAdvice
```

is commonly the convenient choice.

---

# 20. Interview Questions

### What is `@ExceptionHandler`?

> It identifies a method that handles a particular exception thrown during controller request processing.

### Why use `@ControllerAdvice`?

> To centralize controller-related exception handling across multiple controllers instead of duplicating handlers in every controller.

### What is `@RestControllerAdvice`?

> A specialized form of controller advice intended for REST APIs, combining controller-advice behavior with response-body semantics.

### Why use custom exceptions?

> To represent application-specific failure conditions clearly and allow the exception handler to return appropriate HTTP responses.

### Why shouldn't we return stack traces to clients?

> Stack traces may expose internal implementation details, database information, file paths, or security-sensitive information.

---

# 21. Best Practices

Use a consistent error response:

```json
{
  "status": 404,
  "message": "Employee not found",
  "timestamp": "...",
  "path": "/employees/101"
}
```

Use appropriate HTTP statuses:

```text
400 → Invalid request
401 → Unauthenticated
403 → Forbidden
404 → Resource not found
409 → Conflict
500 → Unexpected server error
```

Don't expose:

```text
Stack Trace ❌
SQL Query ❌
Database Credentials ❌
Internal Class Names ❌
```

Keep the global handler focused on **translating exceptions into API responses**, not performing business logic.

---

# 22. Complete Architecture

```text
                 HTTP Request
                      ↓
               DispatcherServlet
                      ↓
                  Controller
                      ↓
                   Service
                      ↓
                 Repository
                      ↓
                  Database

        Exception can occur anywhere
                      ↓
              Exception Propagation
                      ↓
             @RestControllerAdvice
                      ↓
               @ExceptionHandler
                      ↓
              ErrorResponse DTO
                      ↓
                   JSON
                      ↓
                HTTP Response
```

---

# 📍 Where We Are

```text
Spring MVC
│
├── ✅ Why MVC
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controller & Mapping
├── ✅ @PathVariable
├── ✅ @RequestParam
├── ✅ @RequestBody
├── ✅ @ResponseBody
├── ✅ Model
├── ✅ ModelAndView
├── ✅ ViewResolver
├── ✅ Data Binding
├── ✅ Validation
├── ✅ @ExceptionHandler
├── ✅ @ControllerAdvice
├── ✅ @RestControllerAdvice
│
└── ⏭️ Next
      Interceptors
        ↓
      File Upload
        ↓
      Complete Spring MVC Revision
        ↓
      Spring REST
```

### Next topic: **Spring MVC Interceptors**

We'll learn why applications need processing **before and after controller execution**, how `HandlerInterceptor` works, and where interceptors are useful for logging, authentication checks, timing, and request tracing.
