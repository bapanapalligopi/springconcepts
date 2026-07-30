# Spring MVC — Chapter 10: Interceptors

We are continuing exactly where we stopped.

```text id="8u3r5w"
Spring MVC
│
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controllers & Mappings
├── ✅ @RequestBody / @ResponseBody
├── ✅ Model / ViewResolver
├── ✅ Data Binding / Validation
├── ✅ Exception Handling
│
└── ⏭️ Interceptors  ← Current
```

We'll use:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need Interceptors?

Suppose you have 100 controller methods.

Before every request, you want to:

```text
Check authentication
Log request
Record start time
Add tracing information
```

After every request, you want to:

```text
Log response
Calculate execution time
Clean up request data
```

You don't want this:

```java
@GetMapping("/employees")
public Employee getEmployees() {

    logRequest();

    checkSomething();

    // business logic

    logResponse();

    return employee;
}
```

repeated in every controller.

We've already seen a similar problem with **AOP**.

But Spring MVC has a mechanism specifically around **HTTP request processing**:

> **HandlerInterceptor**

---

# 2. What is an Interceptor?

An interceptor is a component that can execute logic:

```text
Before Controller
After Controller
After Complete Request
```

It sits around the controller execution inside the Spring MVC request flow.

Think:

```text id="nrcy1w"
HTTP Request
     ↓
Interceptor
     ↓
Controller
     ↓
Interceptor
     ↓
HTTP Response
```

---

# 3. Interceptor vs AOP

This is important because you already learned AOP.

### AOP

Usually focuses on:

```text
Method execution
```

Examples:

```text
@Transactional
@Cacheable
Logging Aspect
```

### MVC Interceptor

Focuses on:

```text
HTTP request processing
```

Examples:

```text
Request logging
Authentication checks
Request timing
Tracing
```

Think:

```text id="k4dq6j"
AOP
 ↓
Method-level cross-cutting concern

Interceptor
 ↓
Web request-level concern
```

---

# 4. What is HandlerInterceptor?

Spring provides:

```java
HandlerInterceptor
```

with three important lifecycle methods:

```java
preHandle()

postHandle()

afterCompletion()
```

These are the three you should know.

---

# 5. `preHandle()`

## Why?

You may want to run something **before the controller method executes**.

Example:

```text id="7as7m8"
Request
   ↓
Check Authentication
   ↓
Controller
```

---

## What?

`preHandle()` executes before the controller handler.

Example:

```java id="4u3n8r"
@Component
public class LoggingInterceptor
        implements HandlerInterceptor {

    @Override
    public boolean preHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler) {

        System.out.println(
                "Request: " + request.getRequestURI());

        return true;
    }
}
```

---

# 6. What does `return true` mean?

This is very important.

```java id="6p9t4k"
return true;
```

means:

> Continue processing the request.

Flow:

```text id="03h2i9"
preHandle()
   ↓
true
   ↓
Controller
```

---

If you return:

```java id="1hvp70"
return false;
```

Spring stops further processing.

The controller **will not execute**.

Flow:

```text id="w8r0zt"
Request
   ↓
preHandle()
   ↓
false
   ↓
STOP
```

This makes `preHandle()` useful for request checks.

---

# 7. Authentication Example

Imagine:

```text id="6hgt9k"
GET /admin/employees
```

Interceptor checks:

```text
Is user authenticated?
```

If yes:

```text
return true
```

Controller executes.

If no:

```text
return false
```

and the request is stopped/handled appropriately.

In modern applications, authentication is generally handled by **Spring Security**, not a custom interceptor, but this example helps you understand the mechanism.

---

# 8. `postHandle()`

## Why?

Suppose you want to run logic **after the controller method completes but before the final response is rendered**.

That's what `postHandle()` is for.

```java id="7b9q9c"
@Override
public void postHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        ModelAndView modelAndView) {

    System.out.println(
            "Controller completed");
}
```

---

# 9. Important Difference

`postHandle()` is called:

```text
After Controller

Before View Rendering
```

For traditional MVC:

```text id="75k6hc"
Controller
   ↓
postHandle()
   ↓
ViewResolver
   ↓
View
```

This is especially relevant to classic MVC.

---

# 10. `afterCompletion()`

## Why?

Sometimes you want something to happen after the **entire request has completed**.

For example:

* Cleanup
* Final logging
* Request timing
* Releasing request-specific resources

Example:

```java id="9d3d2u"
@Override
public void afterCompletion(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        Exception ex) {

    System.out.println(
            "Request completed");
}
```

---

# 11. Complete Interceptor Flow

This is the most important diagram:

```text id="j8x06p"
HTTP Request
     ↓
preHandle()
     ↓
Controller
     ↓
postHandle()
     ↓
View Rendering / Response Processing
     ↓
afterCompletion()
     ↓
HTTP Response Completed
```

---

# 12. What if an exception occurs?

Suppose:

```java id="y5jly0"
@GetMapping("/employees")
public Employee getEmployee() {

    throw new RuntimeException();
}
```

Simplified flow:

```text id="0l2h3a"
preHandle()
     ↓
Controller
     ↓
Exception
     ↓
Exception Handling
     ↓
afterCompletion()
```

`afterCompletion()` is useful for final cleanup because it is invoked after request processing has completed.

---

# 13. How do we register an Interceptor?

Creating the interceptor isn't enough.

Spring MVC needs to know:

> "Use this interceptor for requests."

Create a configuration class:

```java id="qbyhjj"
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoggingInterceptor interceptor;

    public WebConfig(LoggingInterceptor interceptor) {
        this.interceptor = interceptor;
    }

    @Override
    public void addInterceptors(
            InterceptorRegistry registry) {

        registry.addInterceptor(interceptor);
    }
}
```

Now Spring MVC registers it.

---

# 14. Excluding URLs

You may not want the interceptor applied to everything.

Example:

```java id="t6ic1x"
registry.addInterceptor(interceptor)
        .addPathPatterns("/employees/**")
        .excludePathPatterns("/employees/public/**");
```

Meaning:

```text id="gq7j2l"
/employees/**

      ↓
Interceptor applies

/employees/public/**

      ↓
Interceptor skipped
```

This is very useful in real projects.

---

# 15. Complete Example

### Interceptor

```java id="w1pt9b"
@Component
public class LoggingInterceptor
        implements HandlerInterceptor {

    @Override
    public boolean preHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler) {

        System.out.println(
                "START: "
                + request.getRequestURI());

        return true;
    }

    @Override
    public void postHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler,
            ModelAndView modelAndView) {

        System.out.println("Controller completed");
    }

    @Override
    public void afterCompletion(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler,
            Exception ex) {

        System.out.println(
                "END: "
                + request.getRequestURI());
    }
}
```

### Configuration

```java id="g9k1x6"
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private final LoggingInterceptor interceptor;

    public WebConfig(LoggingInterceptor interceptor) {
        this.interceptor = interceptor;
    }

    @Override
    public void addInterceptors(
            InterceptorRegistry registry) {

        registry.addInterceptor(interceptor);
    }
}
```

---

# 16. Internal Flow

Suppose the client calls:

```http id="zj9p0v"
GET /employees/101
```

Spring MVC flow becomes:

```text id="qg3qk7"
Client
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
preHandle()
  ↓
HandlerAdapter
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Controller Result
  ↓
postHandle()
  ↓
View / Response Processing
  ↓
afterCompletion()
  ↓
HTTP Response
```

Notice where the interceptor fits:

> It surrounds the **controller request processing**, not the entire Spring application.

---

# 17. Interceptor vs Filter

This is a very common interview question.

Both can execute before/after requests, but they operate at different levels.

### Servlet Filter

Works at the **Servlet container level**.

```text id="qmg8y6"
Client
 ↓
Filter
 ↓
DispatcherServlet
 ↓
Controller
```

### Spring MVC Interceptor

Works inside **Spring MVC**.

```text id="6hkrk5"
DispatcherServlet
 ↓
Interceptor
 ↓
Controller
```

So:

```text id="x6d7q3"
Filter
    ↓
Servlet level

Interceptor
    ↓
Spring MVC level
```

---

# 18. Interceptor vs AOP vs Filter

You should be able to distinguish these in interviews.

| Feature              | Filter    | Interceptor      | AOP                     |
| -------------------- | --------- | ---------------- | ----------------------- |
| Level                | Servlet   | Spring MVC       | Spring bean/method      |
| Before controller    | ✅         | ✅                | Sometimes               |
| After controller     | Via chain | ✅                | ✅                       |
| HTTP-specific        | ✅         | ✅                | ❌                       |
| Method-specific      | ❌         | Handler-oriented | ✅                       |
| Transactions         | ❌         | ❌                | ✅                       |
| Logging HTTP request | ✅         | ✅                | Possible, but not ideal |

---

# 19. Where is an Interceptor used?

Common examples:

```text
Request Logging
Request Timing
Correlation ID / Tracing
Locale Handling
User Context
Common Request Checks
```

For authentication/authorization, prefer **Spring Security** once you introduce it rather than building your own security system in an interceptor.

---

# 20. Interview Questions

### What is a Spring MVC Interceptor?

> A component that allows logic to run before and after controller request processing.

### What are the three main methods?

```text
preHandle()
postHandle()
afterCompletion()
```

### What does `preHandle()` returning `false` mean?

> Request processing is stopped and the controller is not invoked.

### Difference between `postHandle()` and `afterCompletion()`?

> `postHandle()` runs after the controller but before view rendering/response completion, while `afterCompletion()` runs after the request has fully completed.

### Interceptor vs Filter?

> Filter operates at the Servlet container level; Interceptor operates within Spring MVC after DispatcherServlet has received the request.

---

# 21. Best Practices

Use interceptors for **web-layer concerns**, such as request tracing or logging.

Don't put:

```text
Business Logic ❌
Database Logic ❌
Transaction Logic ❌
```

inside an interceptor.

Use:

```text
Filter
    → Servlet-level concerns

Interceptor
    → MVC request concerns

AOP
    → Spring bean/method concerns

Spring Security
    → Authentication/Authorization
```

---

# 📍 Where We Are

```text id="0n6r7k"
Spring MVC
│
├── ✅ DispatcherServlet
├── ✅ HandlerMapping
├── ✅ HandlerAdapter
├── ✅ Controllers
├── ✅ Request Mapping
├── ✅ PathVariable / RequestParam
├── ✅ RequestBody / ResponseBody
├── ✅ Model / ViewResolver
├── ✅ Data Binding
├── ✅ Validation
├── ✅ Exception Handling
├── ✅ Interceptors
│
└── ⏭️ Next
      File Upload
        ↓
      Spring MVC Revision
        ↓
      Spring REST
```

### Next topic: **File Upload with Spring MVC**

We'll learn `MultipartFile`, `multipart/form-data`, upload flow, validation, multiple file uploads, and the practical interview questions around file handling.
