# Spring MVC — Chapter 7: `Model`, `ModelAndView`, and `ViewResolver`

Now we complete the **traditional MVC side** of Spring MVC before moving deeper into validation and exception handling.

We’ll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need `Model`?

Suppose your controller gets employee data from the service:

```java
Employee employee = employeeService.getEmployee(101);
```

Now you want to send that data to an HTML/JSP page.

The controller needs a way to pass:

```text
Java Object
   ↓
View
```

That's what the **Model** is used for.

---

# 2. What is `Model`?

`Model` is an object that allows the controller to pass data to the view.

Example:

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    public String getEmployee(Model model) {

        Employee employee =
                employeeService.getEmployee(101);

        model.addAttribute("employee", employee);

        return "employee";
    }
}
```

Here:

```java
model.addAttribute("employee", employee);
```

means:

> Put the `employee` object into the model so the view can access it.

---

# 3. How does it work?

Suppose:

```java
Employee employee =
    new Employee(101, "Rahul", 60000);
```

Controller:

```java
model.addAttribute("employee", employee);
```

Conceptually:

```text
Model

employee → Employee(101, Rahul, 60000)
```

Then:

```java
return "employee";
```

tells Spring:

> Find the view named `employee`.

---

# 4. Complete Flow

```text
Browser
   ↓
DispatcherServlet
   ↓
Controller
   ↓
Service
   ↓
Employee Object
   ↓
Model
   ↓
ViewResolver
   ↓
employee.jsp
   ↓
HTML Response
```

This is classic Spring MVC.

---

# 5. Example with JSP

Suppose the controller returns:

```java
return "employee";
```

A configured `ViewResolver` might translate that into:

```text
/WEB-INF/views/employee.jsp
```

Then the JSP could access the model attribute.

For example:

```jsp
<h1>${employee.name}</h1>
<p>${employee.salary}</p>
```

The page might display:

```text
Rahul
60000
```

---

# 6. What is `ViewResolver`?

## Why?

The controller returns:

```java
return "employee";
```

But `"employee"` isn't a file path.

Spring needs to determine:

> Which actual view should `"employee"` refer to?

That's the job of **ViewResolver**.

---

## What?

> `ViewResolver` maps a logical view name returned by a controller to an actual view resource.

Conceptually:

```text
"employee"
    ↓
ViewResolver
    ↓
/WEB-INF/views/employee.jsp
```

---

# 7. Why use a logical view name?

Instead of hardcoding:

```java
return "/WEB-INF/views/employee.jsp";
```

you write:

```java
return "employee";
```

The view resolver handles the actual location.

This keeps controller code independent of the physical view location.

---

# 8. Example

Suppose configuration is conceptually:

```text
Prefix:
 /WEB-INF/views/

Suffix:
 .jsp
```

Controller:

```java
return "employee";
```

Spring resolves:

```text
/WEB-INF/views/
      +
employee
      +
.jsp

= /WEB-INF/views/employee.jsp
```

---

# 9. `ModelAndView`

Now suppose you want to return:

* the model data
* the view name

together.

Spring provides:

```java
ModelAndView
```

Example:

```java
@GetMapping("/employee")
public ModelAndView getEmployee() {

    Employee employee =
            employeeService.getEmployee(101);

    ModelAndView mav =
            new ModelAndView("employee");

    mav.addObject("employee", employee);

    return mav;
}
```

This contains:

```text
ModelAndView
 ├── View Name → employee
 └── Model → employee object
```

---

# 10. `Model` vs `ModelAndView`

This is a common interview question.

### Using `Model`

```java
@GetMapping("/employee")
public String getEmployee(Model model) {

    model.addAttribute("employee", employee);

    return "employee";
}
```

Here:

```text
Model → data
String → view name
```

---

### Using `ModelAndView`

```java
@GetMapping("/employee")
public ModelAndView getEmployee() {

    ModelAndView mav =
            new ModelAndView("employee");

    mav.addObject("employee", employee);

    return mav;
}
```

Here:

```text
ModelAndView
   ↓
Contains both
   ├── Model
   └── View
```

---

# 11. Which approach is commonly preferred?

In modern Spring MVC code, many developers prefer:

```java
public String getEmployee(Model model)
```

because it cleanly separates:

```text
Model → Data

String → View Name
```

`ModelAndView` is still valid and important to understand, especially for interviews and legacy Spring MVC applications.

---

# 12. Traditional MVC Example

Let's build a complete example.

### Controller

```java
@Controller
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeService service;

    public EmployeeController(EmployeeService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public String getEmployee(
            @PathVariable Integer id,
            Model model) {

        Employee employee =
                service.getEmployee(id);

        model.addAttribute(
                "employee",
                employee
        );

        return "employee";
    }
}
```

---

### Flow

Request:

```http
GET /employees/101
```

Flow:

```text
DispatcherServlet
      ↓
HandlerMapping
      ↓
Controller
      ↓
Service
      ↓
Employee
      ↓
Model
      ↓
return "employee"
      ↓
ViewResolver
      ↓
employee.jsp
      ↓
HTML Response
```

---

# 13. REST vs Traditional MVC

Now you can see the difference much more clearly.

## Traditional MVC

```java
@Controller
public class EmployeeController {

    @GetMapping("/employee")
    public String employee(Model model) {

        model.addAttribute("employee", employee);

        return "employee";
    }
}
```

Flow:

```text
Controller
   ↓
Model
   ↓
ViewResolver
   ↓
JSP / Thymeleaf
   ↓
HTML
```

---

## REST

```java
@RestController
public class EmployeeController {

    @GetMapping("/employee")
    public Employee employee() {

        return employee;
    }
}
```

Flow:

```text
Controller
   ↓
Employee Object
   ↓
HttpMessageConverter
   ↓
JSON
   ↓
HTTP Response
```

This is why REST APIs don't usually use `Model` and `ViewResolver`.

---

# 14. Internal Request Flow

Traditional MVC:

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
HandlerMapping
     ↓
HandlerAdapter
     ↓
Controller
     ↓
Model + View Name
     ↓
ViewResolver
     ↓
View
     ↓
HTTP Response
```

REST:

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
HandlerMapping
     ↓
HandlerAdapter
     ↓
Controller
     ↓
Return Object
     ↓
HttpMessageConverter
     ↓
JSON
     ↓
HTTP Response
```

---

# 15. Where are `Model` and `ViewResolver` used?

They matter primarily in **server-side rendered web applications**, such as:

* JSP applications
* Thymeleaf applications
* Internal enterprise web portals
* Legacy Spring MVC applications

For a modern Spring Boot REST microservice, you will usually work more with:

```text
@RestController
@RequestBody
ResponseEntity
HttpMessageConverter
DTO
```

rather than `Model` and `ViewResolver`.

---

# 16. Interview Questions

### Q1. What is `Model`?

> `Model` is used by a Spring MVC controller to pass data from the controller to the view.

### Q2. What is `ViewResolver`?

> `ViewResolver` converts the logical view name returned by a controller into an actual view resource.

### Q3. What is `ModelAndView`?

> `ModelAndView` is a container that holds both model data and the logical view name.

### Q4. Difference between `Model` and `ModelAndView`?

> With `Model`, the model data and view name are returned separately; with `ModelAndView`, both are packaged into one object.

### Q5. Do REST APIs use `ViewResolver`?

Normally, no. REST controllers typically return response data, which Spring serializes using `HttpMessageConverter`.

---

# 17. Best Practices

For traditional MVC:

```text
Controller
   ↓
Service
   ↓
Model
   ↓
View
```

Keep controllers thin and avoid putting business logic into them.

For REST APIs, prefer DTOs instead of exposing internal domain/entity classes unnecessarily.

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
├── ✅ @PathVariable
├── ✅ @RequestParam
├── ✅ @RequestBody
├── ✅ @ResponseBody
├── ✅ Model
├── ✅ ModelAndView
├── ✅ ViewResolver
│
└── ⏭️ Next
     Data Binding & Validation ⭐⭐⭐⭐⭐
     ↓
     @ModelAttribute
     ↓
     @Valid
     ↓
     Bean Validation
     ↓
     BindingResult
```

## Next topic: Data Binding

We'll learn how Spring automatically converts incoming request data into Java objects using `@ModelAttribute`, and then how **`@Valid` and validation annotations** validate that data.

That topic is extremely important because it's used constantly in real Spring Boot applications.
