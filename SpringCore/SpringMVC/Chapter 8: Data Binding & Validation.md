# Spring MVC — Chapter 8: Data Binding & Validation

We've completed the basic request flow, controllers, mappings, `@RequestBody`, `Model`, and `ViewResolver`.

Now we move to a topic you will use **constantly in real Spring Boot applications**:

> **How does Spring take incoming request data, convert it into Java objects, and validate it?**

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Data Binding?

Suppose a browser submits a form:

```text
name=Rahul
salary=60000
department=IT
```

Your Java code wants:

```java
Employee employee
```

Do you want to manually do this?

```java
String name = request.getParameter("name");
String salary = request.getParameter("salary");

Employee employee = new Employee();

employee.setName(name);
employee.setSalary(Double.parseDouble(salary));
```

Imagine doing this for 20 fields.

That's a lot of boilerplate.

Spring provides **Data Binding**.

---

# 2. What is Data Binding?

> **Data Binding is the process of converting incoming request data into Java object properties.**

Conceptually:

```text
HTTP Request Data
       ↓
Spring Data Binder
       ↓
Java Object
```

Example:

```text
name=Rahul
salary=60000
```

becomes:

```java
Employee employee = new Employee();

employee.setName("Rahul");
employee.setSalary(60000.0);
```

Spring does the binding for you.

---

# 3. `@ModelAttribute`

For traditional Spring MVC form/query-style data binding, you'll commonly see:

```java
@ModelAttribute
```

Example:

```java
@PostMapping("/employees")
public String saveEmployee(
        @ModelAttribute Employee employee) {

    System.out.println(employee.getName());
    System.out.println(employee.getSalary());

    return "success";
}
```

Suppose the request sends:

```text
name=Rahul&salary=60000
```

Spring creates/populates:

```text
Employee
 ├── name = Rahul
 └── salary = 60000
```

---

# 4. How does `@ModelAttribute` work?

Flow:

```text
Request Parameters
       ↓
DataBinder
       ↓
Create Employee Object
       ↓
Find Matching Properties
       ↓
Convert Values
       ↓
Set Properties
       ↓
Controller Method
```

For:

```text
name=Rahul
salary=60000
```

Spring essentially performs the equivalent of:

```java
employee.setName("Rahul");
employee.setSalary(60000.0);
```

---

# 5. Where is `@ModelAttribute` used?

Mostly in traditional Spring MVC:

* HTML forms
* Query/form parameters
* Server-side rendered applications

Example:

```text
Browser Form
     ↓
@Controller
     ↓
@ModelAttribute
     ↓
Java Object
```

For JSON REST requests, we normally use:

```java
@RequestBody
```

instead.

---

# 6. `@ModelAttribute` vs `@RequestBody`

This is important.

### `@ModelAttribute`

Typically binds:

```text
form fields / request parameters
```

Example:

```text
name=Rahul&salary=60000
```

---

### `@RequestBody`

Typically binds:

```text
JSON/XML request body
```

Example:

```json
{
  "name": "Rahul",
  "salary": 60000
}
```

Comparison:

| `@ModelAttribute`         | `@RequestBody`                     |
| ------------------------- | ---------------------------------- |
| Form/query-style data     | Request body                       |
| Data binding              | Message conversion/deserialization |
| Common in traditional MVC | Common in REST APIs                |
| Uses Spring data binding  | Uses `HttpMessageConverter`        |

---

# 7. Why is Data Binding Not Enough?

Suppose the client sends:

```text
name=
salary=-1000
email=abc
```

Spring can bind the values.

But are they valid?

Obviously:

```text
name = empty ❌
salary = negative ❌
email = invalid ❌
```

So we need **Validation**.

---

# 8. What is Validation?

> **Validation checks whether incoming data satisfies the rules defined by the application.**

Examples:

```text
Name → Required

Age → Must be >= 18

Salary → Must be positive

Email → Must be valid

Password → Minimum 8 characters
```

Spring commonly integrates with **Jakarta Bean Validation**.

---

# 9. Validation Annotations

Common annotations you'll use:

```java
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

---

# 10. Example DTO

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

Now the rules are attached to the DTO.

---

# 11. Using `@Valid`

Controller:

```java
@PostMapping("/employees")
public String saveEmployee(
        @Valid @ModelAttribute EmployeeRequest request) {

    return "success";
}
```

`@Valid` tells Spring:

> Validate this object using its Bean Validation annotations.

Flow:

```text
Request
   ↓
Data Binding
   ↓
EmployeeRequest Object
   ↓
@Valid
   ↓
Validation
   ↓
Valid?
 ┌───────┴───────┐
 Yes             No
 ↓                ↓
Controller       Errors
continues
```

---

# 12. Validation with REST

This is much more important for your future Spring Boot work.

Suppose:

```java
public class EmployeeRequest {

    @NotBlank
    private String name;

    @Positive
    private Double salary;
}
```

Controller:

```java
@PostMapping("/employees")
public EmployeeResponse save(
        @Valid @RequestBody EmployeeRequest request) {

    return service.save(request);
}
```

Request:

```json
{
  "name": "",
  "salary": -5000
}
```

Validation fails **before the controller method successfully processes the request**.

This is exactly how you'll use validation in Spring Boot REST APIs.

---

# 13. `BindingResult`

For traditional MVC form binding, you may want to inspect validation errors directly.

Example:

```java
@PostMapping("/employees")
public String saveEmployee(
        @Valid @ModelAttribute EmployeeRequest request,
        BindingResult result) {

    if (result.hasErrors()) {
        return "employee-form";
    }

    service.save(request);

    return "success";
}
```

Now:

```text
Validation Error
       ↓
BindingResult
       ↓
Return Form Page
```

---

# 14. Why is `BindingResult` useful?

Suppose the user enters:

```text
Name = ""
Salary = -100
```

Instead of throwing the user away with a generic error page, you can return to the form and display:

```text
Name is required

Salary must be positive
```

This is especially common in traditional server-side MVC applications.

---

# 15. What happens internally?

Suppose:

```java
@Valid @RequestBody EmployeeRequest request
```

Flow:

```text
HTTP JSON
    ↓
HttpMessageConverter
    ↓
EmployeeRequest
    ↓
Bean Validation
    ↓
@NotBlank / @Positive / @Email
    ↓
Validation Result
```

If invalid, Spring raises a validation-related exception such as:

```text
MethodArgumentNotValidException
```

for the common `@RequestBody` case.

Later, when we learn **Exception Handling**, we'll handle these errors globally using:

```java
@ControllerAdvice
```

or

```java
@RestControllerAdvice
```

---

# 16. Data Binding vs Validation

Don't mix these two concepts.

### Data Binding

```text
"Can I convert the incoming data into my Java object?"
```

Example:

```text
"60000" → Double 60000.0
```

### Validation

```text
"Is the resulting data valid?"
```

Example:

```text
60000 > 0 ✅
```

So:

```text
Request
   ↓
Data Binding
   ↓
Java Object
   ↓
Validation
   ↓
Business Logic
```

---

# 17. Type Conversion

Data often arrives as strings.

For example:

```text
id=101
```

HTTP parameters are textual, but your Java field might be:

```java
private Integer id;
```

Spring converts:

```text
"101"
 ↓
Integer 101
```

Similarly:

```text
"60000"
 ↓
Double 60000.0
```

This conversion is part of Spring's data-binding infrastructure.

---

# 18. Common Example

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

Controller:

```java
@PostMapping("/employees")
public EmployeeResponse save(
        @Valid @RequestBody EmployeeRequest request) {

    return employeeService.save(request);
}
```

Request:

```json
{
  "name": "",
  "salary": -100,
  "email": "abc"
}
```

Validation errors:

```text
Name is required
Salary must be positive
Invalid email
```

---

# 19. `@NotNull` vs `@NotBlank`

Very common interview question.

### `@NotNull`

Checks:

```text
value != null
```

But an empty string is allowed:

```text
""
```

---

### `@NotBlank`

For strings, checks that the value is not:

* `null`
* empty
* only whitespace

So:

```text
null       ❌
""         ❌
"   "      ❌
"Rahul"    ✅
```

---

# 20. `@NotEmpty`

Checks that the value isn't:

* `null`
* empty

For strings, collections, maps, etc.

For a string:

```text
""      ❌
"Rahul" ✅
```

But whitespace-only strings may still be considered non-empty.

So for user-entered strings, `@NotBlank` is often more appropriate.

---

# 21. `@Size`

Example:

```java
@Size(min = 3, max = 20)
private String name;
```

Meaning:

```text
3 <= length <= 20
```

---

# 22. `@Positive`

```java
@Positive
private Double salary;
```

Accepts:

```text
1000 ✅
50000 ✅
```

Rejects:

```text
0 ❌
-100 ❌
```

---

# 23. Where is Validation Used?

Almost everywhere in Spring Boot APIs:

```text
Registration
Login
Employee Creation
Order Creation
Payment Requests
Search Filters
File Metadata
```

It prevents invalid input from reaching business logic.

---

# 24. Interview Questions

### What is Data Binding?

> Data Binding is the process of mapping incoming request data to Java object properties and performing necessary type conversion.

### What is `@ModelAttribute`?

> It binds request parameters or form data to a model object and can also add that object to the MVC model.

### Difference between `@ModelAttribute` and `@RequestBody`?

> `@ModelAttribute` typically binds form/query parameters, while `@RequestBody` reads the request body and uses an `HttpMessageConverter` to deserialize it, such as JSON into a Java object.

### What does `@Valid` do?

> It triggers Bean Validation on the annotated object.

### Difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`?

> `@NotNull` only checks for null, `@NotEmpty` checks that the value is not null or empty, and `@NotBlank` additionally rejects whitespace-only strings.

---

# 25. Best Practices

For REST APIs, prefer:

```java
@Valid @RequestBody EmployeeRequest request
```

Use **DTOs** instead of exposing your database entity directly.

Put validation rules on request DTOs.

Don't depend only on frontend validation; server-side validation is mandatory.

Keep business validation separate from simple input validation.

For example:

```text
@NotBlank
```

is input validation.

But:

```text
"Employee salary cannot exceed the department budget"
```

is business logic and belongs in the service/domain layer.

---

# 📍 Where We Are

```text
Spring MVC
│
├── ✅ Why MVC
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controllers
├── ✅ Request Mapping
├── ✅ @PathVariable
├── ✅ @RequestParam
├── ✅ @RequestBody
├── ✅ @ResponseBody
├── ✅ Model
├── ✅ ModelAndView
├── ✅ ViewResolver
├── ✅ Data Binding
├── ✅ @ModelAttribute
├── ✅ Validation
│
└── ⏭️ Next
      Exception Handling ⭐⭐⭐⭐⭐
      ↓
      @ExceptionHandler
      ↓
      @ControllerAdvice
      ↓
      @RestControllerAdvice
      ↓
      Global Error Response
```

## Next topic

We'll learn **Spring MVC Exception Handling**, including how an exception travels from the controller through Spring MVC and how to build a **global exception handler**—a pattern you'll use constantly in Spring Boot REST applications.
