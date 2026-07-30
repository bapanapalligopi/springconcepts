# Spring MVC — Chapter 5: Controller & Request Mapping

Now we start writing actual Spring MVC endpoints.

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Controller mappings?

Suppose a client sends:

```http
GET /employees
```

Spring needs to know:

> Which Java method should execute?

We need to map:

```text
HTTP Request
     ↓
Controller Method
```

That's what request-mapping annotations do.

---

# 2. What is a Controller?

A controller is the Spring MVC component that handles incoming web requests.

Example:

```java
@Controller
public class EmployeeController {

}
```

For REST APIs, we commonly use:

```java
@RestController
public class EmployeeController {

}
```

We'll study the difference between these carefully in the REST module.

For now:

```text
@Controller
   ↓
Traditional MVC / Views

@RestController
   ↓
REST / Response Body
```

---

# 3. `@RequestMapping`

This is the general-purpose request mapping annotation.

```java
@Controller
@RequestMapping("/employees")
public class EmployeeController {

    @RequestMapping("/list")
    public String list() {
        return "employees";
    }
}
```

The complete path is:

```text
/employees/list
```

---

# 4. Class-Level vs Method-Level Mapping

This is important.

```java
@RequestMapping("/employees")
public class EmployeeController {

    @RequestMapping("/list")
    public String list() {
        return "employees";
    }
}
```

Spring combines them:

```text
/employees
   +
/list
   =
/employees/list
```

This lets you avoid repeating common paths.

---

# 5. HTTP Methods

HTTP provides methods such as:

```text
GET
POST
PUT
PATCH
DELETE
```

Spring lets you map them directly.

```java
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

These are specialized forms of `@RequestMapping`.

---

# 6. `@GetMapping`

Used for retrieving data.

```java
@GetMapping("/employees")
public String getEmployees() {
    return "employees";
}
```

Request:

```http
GET /employees
```

matches this method.

---

# 7. `@PostMapping`

Used to submit/create data.

```java
@PostMapping("/employees")
public String saveEmployee() {
    return "Employee Saved";
}
```

Request:

```http
POST /employees
```

matches it.

---

# 8. `@PutMapping`

Typically used for updating/replacing an existing resource.

```java
@PutMapping("/employees/{id}")
public String updateEmployee(@PathVariable Integer id) {
    return "Employee " + id + " Updated";
}
```

Request:

```http
PUT /employees/101
```

---

# 9. `@PatchMapping`

Typically used for partial updates.

```java
@PatchMapping("/employees/{id}")
public String updateSalary(
        @PathVariable Integer id) {

    return "Salary Updated";
}
```

---

# 10. `@DeleteMapping`

Used for deletion.

```java
@DeleteMapping("/employees/{id}")
public String deleteEmployee(
        @PathVariable Integer id) {

    return "Employee Deleted";
}
```

Request:

```http
DELETE /employees/101
```

---

# 11. `@PathVariable`

## Why?

Suppose the URL contains:

```text
/employees/101
```

The `101` represents the employee ID.

We need to extract it from the URL.

That's what `@PathVariable` does.

---

## What?

> `@PathVariable` binds a URI path variable to a Java method parameter.

Example:

```java
@GetMapping("/employees/{id}")
public String getEmployee(
        @PathVariable Integer id) {

    return "Employee ID = " + id;
}
```

Request:

```http
GET /employees/101
```

Spring gives:

```java
id = 101
```

---

# 12. How does it work?

```text
GET /employees/101
        ↓
DispatcherServlet
        ↓
HandlerMapping
        ↓
/employees/{id} matches
        ↓
PathVariable Resolver
        ↓
id = 101
        ↓
Controller Method
```

This is part of the argument-resolution mechanism we discussed with `HandlerAdapter`.

---

# 13. Named Path Variable

You can explicitly specify the name:

```java
@GetMapping("/employees/{id}")
public String getEmployee(
        @PathVariable("id") Integer employeeId) {

    return "Employee ID = " + employeeId;
}
```

URL:

```text
/employees/101
```

Mapping:

```text
{id}
 ↓
employeeId
```

---

# 14. Multiple Path Variables

```java
@GetMapping("/departments/{deptId}/employees/{empId}")
public String getEmployee(
        @PathVariable Integer deptId,
        @PathVariable Integer empId) {

    return "Department = " + deptId
            + ", Employee = " + empId;
}
```

Request:

```text
/departments/10/employees/101
```

Spring resolves:

```text
deptId = 10
empId  = 101
```

---

# 15. `@RequestParam`

Now consider:

```text
/employees?id=101
```

The `id` is not part of the path.

It's a **query parameter**.

We use:

```java
@RequestParam
```

Example:

```java
@GetMapping("/employees")
public String getEmployee(
        @RequestParam Integer id) {

    return "Employee ID = " + id;
}
```

Request:

```http
GET /employees?id=101
```

Spring gives:

```java
id = 101
```

---

# 16. PathVariable vs RequestParam

This is a very common interview question.

### PathVariable

```text
/employees/101
```

```java
@PathVariable
```

The value is part of the URL path.

### RequestParam

```text
/employees?id=101
```

```java
@RequestParam
```

The value is a query parameter.

---

## Comparison

| `@PathVariable`               | `@RequestParam`                |
| ----------------------------- | ------------------------------ |
| `/employees/101`              | `/employees?id=101`            |
| Part of URL path              | Query parameter                |
| Usually identifies a resource | Often used for filters/options |

---

# 17. Optional Request Parameter

Suppose:

```text
/employees?department=IT
```

But `department` may not always be provided.

```java
@GetMapping("/employees")
public String getEmployees(
        @RequestParam(required = false)
        String department) {

    return department;
}
```

If omitted:

```text
department = null
```

You can also provide a default:

```java
@RequestParam(
    defaultValue = "ALL"
)
String department
```

---

# 18. Multiple Query Parameters

```java
@GetMapping("/employees")
public String search(
        @RequestParam String department,
        @RequestParam Integer minSalary) {

    return department + " " + minSalary;
}
```

Request:

```text
/employees?department=IT&minSalary=50000
```

Spring resolves:

```text
department = IT
minSalary = 50000
```

---

# 19. `@RequestMapping` vs Specialized Annotations

This:

```java
@RequestMapping(
    value = "/employees",
    method = RequestMethod.GET
)
```

is equivalent in intention to:

```java
@GetMapping("/employees")
```

Similarly:

```java
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

are specialized forms.

For modern Spring code, specialized mappings are generally clearer.

---

# 20. Complete Controller Example

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public String getAllEmployees() {
        return "All Employees";
    }

    @GetMapping("/{id}")
    public String getEmployee(
            @PathVariable Integer id) {

        return "Employee = " + id;
    }

    @PostMapping
    public String saveEmployee() {
        return "Employee Saved";
    }

    @PutMapping("/{id}")
    public String updateEmployee(
            @PathVariable Integer id) {

        return "Employee Updated = " + id;
    }

    @DeleteMapping("/{id}")
    public String deleteEmployee(
            @PathVariable Integer id) {

        return "Employee Deleted = " + id;
    }
}
```

Mappings become:

```text
GET     /employees
GET     /employees/{id}
POST    /employees
PUT     /employees/{id}
DELETE  /employees/{id}
```

This is already close to a real REST controller.

---

# 21. Internal Request Flow

For:

```http
GET /employees/101
```

the flow is:

```text
Client
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
RequestMappingHandlerMapping
  ↓
Find:
GET /employees/{id}
  ↓
RequestMappingHandlerAdapter
  ↓
Resolve @PathVariable
  ↓
getEmployee(101)
```

This connects everything we've learned so far.

---

# 22. Where is this used?

Every Spring MVC application uses request mappings.

Typical controller design:

```text
EmployeeController
    ↓
/employees

OrderController
    ↓
/orders

ProductController
    ↓
/products
```

This mapping layer defines your application's HTTP interface.

---

# 23. Interview Questions

### What is `@RequestMapping`?

> `@RequestMapping` maps HTTP requests to controller classes or methods based on path and other request conditions such as HTTP method, parameters, headers, consumes, and produces.

### Difference between `@GetMapping` and `@RequestMapping`?

> `@GetMapping` is a specialized shortcut for mapping HTTP GET requests, while `@RequestMapping` is more general.

### What is `@PathVariable`?

> It binds a variable embedded in the URI path to a method parameter.

### What is `@RequestParam`?

> It binds a query parameter from the request URL to a method parameter.

### Difference between `/employees/{id}` and `/employees?id=101`?

> The first uses a path variable; the second uses a query parameter.

---

# 24. Best Practices

Use meaningful resource-oriented URLs:

```text
GET    /employees
GET    /employees/101
POST   /employees
PUT    /employees/101
DELETE /employees/101
```

Avoid URLs like:

```text
/getEmployee
/saveEmployee
/deleteEmployee
```

for REST-style APIs; the HTTP method should convey the operation.

Keep controller methods thin:

```text
Controller
    ↓
Service
    ↓
Repository
```

Don't put database queries directly in controllers.

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
├── ✅ Controller
├── ✅ Request Mapping
├── ✅ @GetMapping
├── ✅ @PostMapping
├── ✅ @PutMapping
├── ✅ @PatchMapping
├── ✅ @DeleteMapping
├── ✅ @PathVariable
├── ✅ @RequestParam
│
└── ⏭️ Next:
     @RequestBody ⭐⭐⭐⭐⭐
     @ResponseBody
     Model
     ModelAndView
     ViewResolver
```

The **next topic is `@RequestBody`**, where we'll learn how Spring converts incoming JSON into Java objects. This is the bridge between classic Spring MVC and the REST APIs you'll build in Spring Boot.
