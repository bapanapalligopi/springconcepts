# Spring REST — Chapter 3: Request DTOs & Response DTOs

We now move to a **very important real-project topic**.

At this stage, don't just learn how to make an API work. Learn how to design an API properly.

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need DTOs?

Suppose you're using JPA later and have this entity:

```java
@Entity
public class Employee {

    @Id
    private Integer id;

    private String name;

    private Double salary;

    private String email;

    private String password;

    private String internalStatus;

    // getters/setters
}
```

Now your REST controller does this:

```java
@GetMapping("/{id}")
public Employee getEmployee(@PathVariable Integer id) {
    return employeeService.getEmployee(id);
}
```

The API might return:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000,
  "email": "rahul@example.com",
  "password": "secret",
  "internalStatus": "ACTIVE"
}
```

That's a **serious design problem**.

The client doesn't need:

```text
password
internalStatus
database-specific fields
```

You shouldn't expose your persistence model directly as your API contract.

That's one of the main reasons DTOs exist.

---

# 2. What is DTO?

DTO stands for:

> **Data Transfer Object**

A DTO is a Java object specifically designed to transfer data between application boundaries.

For REST APIs, you commonly have:

```text
Request DTO
Response DTO
```

Think:

```text
Client
   ↓
Request DTO
   ↓
Service
   ↓
Entity
   ↓
Database
```

and:

```text
Database
   ↓
Entity
   ↓
Response DTO
   ↓
Client
```

---

# 3. Request DTO

A Request DTO represents **what the client is allowed to send**.

Example:

```java
public class EmployeeRequest {

    private String name;

    private Double salary;

    private String email;

    // getters/setters
}
```

Controller:

```java
@PostMapping
public ResponseEntity<EmployeeResponse> create(
        @Valid @RequestBody EmployeeRequest request) {

    EmployeeResponse response =
            employeeService.create(request);

    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
}
```

The client sends:

```json
{
  "name": "Rahul",
  "salary": 60000,
  "email": "rahul@example.com"
}
```

Notice:

No password.

No database ID.

No internal fields.

---

# 4. Response DTO

A Response DTO represents **what your API chooses to expose**.

```java
public class EmployeeResponse {

    private Integer id;

    private String name;

    private Double salary;

    // getters/setters
}
```

Response:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000
}
```

The client receives only what it needs.

---

# 5. Entity vs DTO

This distinction is extremely important.

## Entity

Represents persistence/database data.

```java
@Entity
public class Employee {
    ...
}
```

## Request DTO

Represents incoming API data.

```java
public class EmployeeRequest {
    ...
}
```

## Response DTO

Represents outgoing API data.

```java
public class EmployeeResponse {
    ...
}
```

So:

```text
Entity
   ↓
Database model

DTO
   ↓
API model
```

---

# 6. Why not use Entity everywhere?

There are several reasons.

### 1. Security

Your entity may contain:

```text
password
internal flags
audit fields
```

You don't want them in the API response.

### 2. Loose Coupling

If your database/entity changes:

```java
private String internalCode;
```

your API contract shouldn't automatically change.

### 3. Different Request and Response Shapes

Create request:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Response:

```json
{
  "id": 101,
  "name": "Rahul",
  "salary": 60000,
  "createdAt": "..."
}
```

Same business object, different API representations.

### 4. Validation

Request DTO:

```java
@NotBlank
private String name;

@Positive
private Double salary;
```

You don't necessarily want these API-specific validation rules on the persistence entity.

---

# 7. How does the flow work?

Suppose the client sends:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Flow:

```text
JSON
 ↓
@RequestBody
 ↓
EmployeeRequest
 ↓
@Valid
 ↓
Service
 ↓
Employee Entity
 ↓
Repository
 ↓
Database
```

Then response:

```text
Database
 ↓
Employee Entity
 ↓
EmployeeResponse
 ↓
HttpMessageConverter
 ↓
JSON
 ↓
Client
```

---

# 8. Mapping DTO to Entity

We need to convert:

```text
EmployeeRequest
      ↓
Employee
```

Simple example:

```java
public Employee toEntity(EmployeeRequest request) {

    Employee employee = new Employee();

    employee.setName(request.getName());
    employee.setSalary(request.getSalary());
    employee.setEmail(request.getEmail());

    return employee;
}
```

Then:

```java
Employee employee = toEntity(request);

repository.save(employee);
```

---

# 9. Mapping Entity to Response DTO

After retrieving data:

```java
public EmployeeResponse toResponse(Employee employee) {

    EmployeeResponse response =
            new EmployeeResponse();

    response.setId(employee.getId());
    response.setName(employee.getName());
    response.setSalary(employee.getSalary());

    return response;
}
```

---

# 10. Where should mapping happen?

At your experience level, you have several choices.

For a simple project:

```text
Service
   ↓
Map DTO ↔ Entity
```

For larger projects:

```text
Mapper Class

EmployeeMapper
    ↓
toEntity()
toResponse()
```

Example:

```java
@Component
public class EmployeeMapper {

    public Employee toEntity(EmployeeRequest request) {
        ...
    }

    public EmployeeResponse toResponse(Employee employee) {
        ...
    }
}
```

This keeps mapping logic out of your service.

---

# 11. Complete Example

## Request DTO

```java
public class EmployeeRequest {

    @NotBlank
    private String name;

    @Positive
    private Double salary;

    @Email
    private String email;

    // getters/setters
}
```

## Response DTO

```java
public class EmployeeResponse {

    private Integer id;

    private String name;

    private Double salary;

    private String email;

    // getters/setters
}
```

## Controller

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeService service;

    public EmployeeController(EmployeeService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<EmployeeResponse> create(
            @Valid @RequestBody EmployeeRequest request) {

        EmployeeResponse response =
                service.create(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(response);
    }
}
```

## Service

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

    public EmployeeResponse create(
            EmployeeRequest request) {

        Employee employee = new Employee();

        employee.setName(request.getName());
        employee.setSalary(request.getSalary());
        employee.setEmail(request.getEmail());

        Employee saved =
                repository.save(employee);

        EmployeeResponse response =
                new EmployeeResponse();

        response.setId(saved.getId());
        response.setName(saved.getName());
        response.setSalary(saved.getSalary());
        response.setEmail(saved.getEmail());

        return response;
    }
}
```

This is a clean layered design.

---

# 12. Real Project Architecture

```text
Client
   ↓
JSON
   ↓
EmployeeRequest DTO
   ↓
Controller
   ↓
Service
   ↓
Employee Entity
   ↓
Repository
   ↓
Database
   ↓
Employee Entity
   ↓
EmployeeResponse DTO
   ↓
JSON
   ↓
Client
```

This separation becomes especially valuable once we introduce JPA/Hibernate.

---

# 13. DTO and Validation

One major benefit is that the Request DTO can define API-specific validation:

```java
public class EmployeeRequest {

    @NotBlank(message = "Name is required")
    private String name;

    @Positive(message = "Salary must be positive")
    private Double salary;

    @Email(message = "Invalid email")
    private String email;
}
```

Then:

```java
@Valid @RequestBody EmployeeRequest request
```

Validation happens before business logic.

---

# 14. What if the client sends extra fields?

Suppose the request contains:

```json
{
  "name": "Rahul",
  "salary": 60000,
  "password": "secret"
}
```

If `EmployeeRequest` doesn't define `password`, the handling depends on your Jackson configuration.

The important architectural principle is:

> Don't create a DTO containing fields just because the client might send them.

Define the contract intentionally.

---

# 15. DTO vs Entity Interview Question

**Interviewer:**

> Why don't you return entities directly from your REST controllers?

Good answer:

> "We use DTOs to separate our API contract from the persistence model. This prevents exposing internal or sensitive fields, reduces coupling between the API and database schema, allows different request and response structures, and gives us a clean place for API-specific validation."

That's a strong 1.5–2 year answer.

---

# 16. Common Interview Questions

### What is DTO?

> A Data Transfer Object is an object designed to transfer data across application boundaries without necessarily representing the persistence model.

### Why use Request DTO?

> To control what the client is allowed to send and apply request-specific validation.

### Why use Response DTO?

> To control what the API exposes to clients and avoid leaking internal entity fields.

### Entity vs DTO?

> Entity is primarily a persistence model; DTO is an API/data-transfer model.

### Where should DTO mapping happen?

> Commonly in the service or a dedicated mapper component, depending on project size and complexity.

---

# 17. Best Practices

For your level, follow this:

```text
REST Controller
      ↓
Request DTO
      ↓
Service
      ↓
Entity
      ↓
Repository
```

and:

```text
Repository
      ↓
Entity
      ↓
Service
      ↓
Response DTO
      ↓
REST Controller
```

Avoid:

```text
Controller
   ↓
Entity directly exposed ❌
```

Also avoid putting database/JPA logic inside DTOs.

Keep DTOs simple.

---

# 📍 Where We Are

```text
Spring REST
│
├── ✅ Why REST
├── ✅ REST principles
├── ✅ Resources
├── ✅ HTTP Methods
├── ✅ Statelessness
├── ✅ Idempotency
├── ✅ HTTP Status Codes
├── ✅ ResponseEntity
├── ✅ Request/Response DTOs
│
└── ⏭️ Next
      Validation + Global Error Response
      ↓
      REST API Design
      ↓
      Pagination
      ↓
      Sorting
      ↓
      Filtering
```

## Next topic

We'll combine the concepts we've learned into **production-style REST validation and global error handling**, including:

```text
@Valid
   ↓
Validation Failure
   ↓
@RestControllerAdvice
   ↓
Field-level errors
   ↓
Consistent API error response
```

After that, we'll move into **pagination, sorting, and filtering**, which are very common in real Spring Boot APIs.
