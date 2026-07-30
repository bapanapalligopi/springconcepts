# Spring REST — Chapter 4: Validation + Global Error Handling

Now we're going to combine several things you've already learned:

```text
@RequestBody
   ↓
DTO
   ↓
@Valid
   ↓
Validation
   ↓
@RestControllerAdvice
   ↓
Consistent Error Response
```

This is **very important for real Spring Boot projects and interviews**.

We'll use:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need validation?

Suppose our API creates an employee:

```http
POST /employees
```

Request:

```json
{
  "name": "",
  "salary": -5000,
  "email": "abc"
}
```

Without validation, this bad data can reach:

```text
Controller
   ↓
Service
   ↓
Database
```

We want invalid data to be rejected **before business logic executes**.

---

# 2. What is validation?

Validation checks whether incoming data satisfies predefined rules.

For example:

```text
name     → required
salary   → positive
email    → valid email
age      → minimum 18
```

We usually put these rules on the **Request DTO**.

---

# 3. Request DTO

```java
public class EmployeeRequest {

    @NotBlank(message = "Name is required")
    private String name;

    @Positive(message = "Salary must be positive")
    private Double salary;

    @Email(message = "Invalid email")
    private String email;

    // getters/setters
}
```

Then:

```java
@PostMapping
public EmployeeResponse create(
        @Valid @RequestBody EmployeeRequest request) {

    return employeeService.create(request);
}
```

---

# 4. How does validation work?

Suppose the client sends:

```json
{
  "name": "",
  "salary": -5000,
  "email": "abc"
}
```

Spring processes it roughly like this:

```text
JSON Request
    ↓
HttpMessageConverter
    ↓
EmployeeRequest Object
    ↓
@Valid
    ↓
Bean Validation
    ↓
Validation Errors
```

Since the request is invalid, the controller method does not proceed normally.

For the common `@RequestBody` case, Spring raises:

```text
MethodArgumentNotValidException
```

---

# 5. Why do we need Global Error Handling?

Suppose you have:

```text
EmployeeController
OrderController
PaymentController
ProductController
```

All of them can have validation errors.

Do we want:

```java
try/catch
```

inside every controller?

No.

We want one centralized place:

```java
@RestControllerAdvice
```

---

# 6. What is `@RestControllerAdvice`?

It provides **global exception handling for REST controllers**.

Think:

```text
EmployeeController ──┐
OrderController ─────┤
PaymentController ───┤
ProductController ───┤
                     ↓
            @RestControllerAdvice
                     ↓
             Error Response
```

This prevents duplicate exception-handling code.

---

# 7. Handle Validation Errors

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {

        ErrorResponse error =
                new ErrorResponse(
                        400,
                        "Validation failed");

        return ResponseEntity
                .badRequest()
                .body(error);
    }
}
```

This works, but we can make it more useful by returning **field-level errors**.

---

# 8. Better Error Response

Create:

```java
public class ErrorResponse {

    private int status;
    private String message;
    private Map<String, String> errors;

    // constructor/getters/setters
}
```

Then:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult()
          .getFieldErrors()
          .forEach(error ->
              errors.put(
                  error.getField(),
                  error.getDefaultMessage()
              )
          );

        ErrorResponse response =
                new ErrorResponse();

        response.setStatus(400);
        response.setMessage("Validation failed");
        response.setErrors(errors);

        return ResponseEntity
                .badRequest()
                .body(response);
    }
}
```

Now a request like:

```json
{
  "name": "",
  "salary": -5000,
  "email": "abc"
}
```

can produce:

```json
{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "name": "Name is required",
    "salary": "Salary must be positive",
    "email": "Invalid email"
  }
}
```

That's much more useful for frontend clients.

---

# 9. Complete Flow

```text
Client
  ↓
POST /employees
  ↓
JSON
  ↓
@RequestBody
  ↓
EmployeeRequest
  ↓
@Valid
  ↓
Validation
  ↓
Invalid
  ↓
MethodArgumentNotValidException
  ↓
@RestControllerAdvice
  ↓
@ExceptionHandler
  ↓
ErrorResponse DTO
  ↓
JSON
  ↓
HTTP 400
```

Notice something important:

**The service method isn't executed when validation fails.**

That's exactly what we want.

---

# 10. Custom Business Exceptions

Validation isn't the only thing we need to handle.

Suppose:

```java
public EmployeeResponse getEmployee(Integer id) {

    Employee employee = repository.findById(id);

    if (employee == null) {
        throw new EmployeeNotFoundException(
                "Employee not found: " + id);
    }

    ...
}
```

Create:

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
@ExceptionHandler(EmployeeNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(
        EmployeeNotFoundException ex) {

    ErrorResponse response =
            new ErrorResponse(
                    404,
                    ex.getMessage());

    return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(response);
}
```

Response:

```json
{
  "status": 404,
  "message": "Employee not found: 101"
}
```

---

# 11. Different Exception → Different Status

A real application might have:

```text
Validation error
     ↓
400

Authentication failure
     ↓
401

Authorization failure
     ↓
403

Employee not found
     ↓
404

Duplicate employee/email
     ↓
409

Unexpected error
     ↓
500
```

This gives the client meaningful information.

---

# 12. One Global Handler

A practical structure:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse>
    handleValidation(...) {
        ...
    }

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse>
    handleEmployeeNotFound(...) {
        ...
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse>
    handleGenericException(...) {
        ...
    }
}
```

The last one is a fallback for unexpected exceptions.

But don't expose:

```text
stack trace
SQL
internal class names
file paths
```

to the client.

---

# 13. Where should validation happen?

There are two different kinds of validation.

### Input validation

Examples:

```java
@NotBlank
@Email
@Positive
@Size
```

Usually belongs on the DTO.

### Business validation

Example:

> Employee cannot be transferred to another department if salary processing is already finalized.

That's business logic.

It belongs in the **service/domain layer**, not just in annotations.

So:

```text
Request DTO
   ↓
Basic input validation
   ↓
Service
   ↓
Business validation
```

---

# 14. Common Validation Annotations

For your experience level, know these well:

```text
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Positive
@PositiveOrZero
@Email
@Pattern
```

Examples:

```java
@NotBlank
private String name;
```

```java
@Positive
private Double salary;
```

```java
@Size(min = 8, max = 20)
private String password;
```

```java
@Email
private String email;
```

---

# 15. What about `@Valid` vs `@Validated`?

This is worth knowing.

### `@Valid`

Standard Bean Validation trigger.

Common:

```java
@Valid @RequestBody EmployeeRequest request
```

### `@Validated`

Spring's validation annotation.

It is useful for features such as **validation groups** and is also commonly used for method-level validation scenarios.

At your level:

```text
@Valid       → Must know
@Validated   → Know the basic difference
```

---

# 16. Interview Questions

### Why should validation be done before the service layer?

> To reject malformed or invalid input early and prevent unnecessary business/database processing.

### What exception is commonly thrown for an invalid `@RequestBody` with `@Valid`?

> `MethodArgumentNotValidException`.

### Why use `@RestControllerAdvice`?

> To centralize REST exception handling across controllers and return consistent error responses.

### Difference between validation and business validation?

> Validation annotations check input constraints such as required fields or ranges; business validation checks application-specific rules and belongs in the business/service layer.

### Why use DTOs for validation?

> They allow API-specific validation rules without coupling the API contract directly to the persistence entity.

---

# 17. Best Practices

A good REST architecture is:

```text
Client
  ↓
Request DTO
  ↓
@Valid
  ↓
Controller
  ↓
Service
  ↓
Repository
```

Errors:

```text
Exception
  ↓
@RestControllerAdvice
  ↓
ErrorResponse
  ↓
HTTP Status
```

Use consistent error structures across the application.

Don't return different error formats from every controller.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ REST fundamentals
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
│
└── ⏭️ Next
      Pagination
      ↓
      Sorting
      ↓
      Filtering / Search
      ↓
      REST API Design
```

## Next topic: Pagination

We'll learn why returning 1 million database rows from:

```http
GET /employees
```

is a bad design, and how Spring APIs implement:

```http
GET /employees?page=0&size=20
```

including `Page`, `Pageable`, `PageRequest`, sorting, and the interview questions around pagination.
