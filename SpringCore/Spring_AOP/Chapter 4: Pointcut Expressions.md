Excellent! This is one of the **most important topics in Spring AOP**. If you understand **Pointcut Expressions**, you'll be able to read and write almost any AOP code.

We'll follow our learning structure:

> **Why → What → How → Where → Code → Internal Working → Interview Questions → Best Practices**

---

# Chapter 4: Pointcut Expressions

---

# 1. Why? (The Problem)

Suppose your project has the following service:

```java
@Service
public class EmployeeService {

    public void saveEmployee() {}

    public void updateEmployee() {}

    public void deleteEmployee() {}

}
```

And another service:

```java
@Service
public class DepartmentService {

    public void saveDepartment() {}

    public void updateDepartment() {}

}
```

Suppose you create a logging aspect.

Question:

**Should logging execute for every method?**

Maybe not.

You may want:

* Only `saveEmployee()`
* All methods in `EmployeeService`
* All methods in every service
* Only methods starting with `save`
* Only methods returning `String`

Spring needs a mechanism to **select** methods.

That's exactly what a **Pointcut Expression** does.

---

# 2. What?

A **Pointcut Expression** tells Spring:

> **"Apply this advice only to these methods."**

Think of it like a filter.

Without Pointcut

```text
All Methods

↓

Logging
```

With Pointcut

```text
All Methods

↓

Filter (Pointcut)

↓

Selected Methods

↓

Logging
```

---

# 3. How?

The most common pointcut expression uses the `execution()` designator.

Example:

```java
@Before("execution(* com.demo.service.EmployeeService.saveEmployee())")
public void log() {
    System.out.println("Logging...");
}
```

Let's break it down.

---

# Understanding `execution()`

General syntax:

```text
execution(modifiers? return-type package.class.method(parameters))
```

The most common format is:

```text
execution(return-type package.class.method(parameters))
```

Example:

```text
execution(* com.demo.service.EmployeeService.saveEmployee())
```

Let's decode it.

---

## Part 1: `execution`

```text
execution(...)
```

Means:

> "Intercept method execution."

Spring AOP works only with **method execution**.

---

## Part 2: `*` (Return Type)

```text
execution(* ...)
```

The first `*` means:

> Any return type.

Examples:

```java
public void saveEmployee()
```

```java
public String getEmployee()
```

```java
public int countEmployees()
```

All are matched.

If you write:

```text
execution(void ...)
```

Only methods returning `void` match.

---

## Part 3: Package

```text
com.demo.service
```

Means:

Look only inside this package.

---

## Part 4: Class Name

```text
EmployeeService
```

Means:

Only this class.

---

## Part 5: Method Name

```text
saveEmployee
```

Only this method.

---

## Part 6: Parameters

```text
()
```

Means:

Method with **no parameters**.

---

Complete meaning:

```text
execution(* com.demo.service.EmployeeService.saveEmployee())
```

↓

```text
Any return type

↓

EmployeeService

↓

saveEmployee()

↓

No parameters
```

---

# 4. Wildcards (`*`)

Wildcards make pointcuts flexible.

---

## `*` for Return Type

```text
execution(* ...)
```

Matches:

```java
void
String
Employee
int
boolean
```

Everything.

---

## `*` for Method Name

```text
execution(* com.demo.service.EmployeeService.*())
```

Means:

All methods.

Matches:

```java
saveEmployee()

updateEmployee()

deleteEmployee()

findEmployee()
```

---

## `save*`

```text
execution(* com.demo.service.EmployeeService.save*())
```

Matches:

```java
save()

saveEmployee()

saveData()

saveAll()
```

Does NOT match:

```java
updateEmployee()

deleteEmployee()
```

---

# `*Service`

Suppose you have:

```text
EmployeeService

DepartmentService

OrderService
```

Expression:

```text
execution(* com.demo.service.*Service.*(..))
```

Matches all service classes ending with `Service`.

---

# 5. Double Dot (`..`)

This is another commonly asked interview topic.

---

## Why?

Suppose methods have different parameters.

```java
saveEmployee()

saveEmployee(Employee e)

saveEmployee(Employee e, String role)
```

If you write:

```text
()
```

Only the first method matches.

We need something more flexible.

---

## `(..)`

Means:

**Any number of parameters.**

Matches:

```java
()

(Employee)

(Employee, String)

(int, String, boolean)
```

Everything.

---

Example:

```text
execution(* EmployeeService.*(..))
```

Matches every method in `EmployeeService` regardless of parameters.

---

## Package `..`

Suppose you have:

```text
com.demo.service

com.demo.service.employee

com.demo.service.department
```

Expression:

```text
execution(* com.demo.service..*.*(..))
```

The package `..` means:

Include all sub-packages.

---

# 6. Real Project Examples

## Example 1

All methods in EmployeeService

```java
@Before("execution(* com.demo.service.EmployeeService.*(..))")
```

---

## Example 2

Only save methods

```java
@Before("execution(* com.demo.service.*.save*(..))")
```

Matches:

```java
saveEmployee()

saveOrder()

saveDepartment()
```

---

## Example 3

Only methods returning `String`

```java
execution(String com.demo.service.*.*(..))
```

---

## Example 4

Only methods with one String parameter

```java
execution(* com.demo.service.*.*(String))
```

Matches:

```java
findEmployee(String id)
```

---

## Example 5

Only methods with two parameters

```java
execution(* com.demo.service.*.*(*,*))
```

Matches:

```java
save(Employee e, String role)
```

---

# 7. Reusable Pointcuts

Instead of writing the same expression multiple times:

```java
@Before("execution(* com.demo.service.*.*(..))")
```

```java
@After("execution(* com.demo.service.*.*(..))")
```

Create a reusable pointcut.

```java
@Pointcut("execution(* com.demo.service.*.*(..))")
public void serviceMethods() {}
```

Then use it:

```java
@Before("serviceMethods()")
public void beforeLog() {}

@After("serviceMethods()")
public void afterLog() {}
```

This improves readability and maintenance.

---

# 8. Internal Flow

Suppose:

```java
employeeService.saveEmployee();
```

Spring flow:

```text
Application Starts
        │
        ▼
Find @Aspect
        │
        ▼
Read Pointcut
        │
        ▼
Does saveEmployee() Match?
        │
     Yes
        │
        ▼
Execute Advice
        │
        ▼
Call Business Method
```

If the method doesn't match the pointcut, Spring directly invokes the business method without running the advice.

---

# 9. Where Is This Used?

Real-world examples:

### Logging

```text
execution(* com.company.service.*.*(..))
```

Log all service methods.

---

### Security

```text
execution(* com.company.admin.*.*(..))
```

Protect admin APIs.

---

### Transactions

```text
execution(* com.company.repository.*.*(..))
```

Apply transaction management to repository methods.

---

### Performance Monitoring

```text
execution(* com.company.payment.*.*(..))
```

Measure payment processing time.

---

# 10. Commonly Used Pointcut Designators

Besides `execution()`, Spring AOP also supports:

| Designator      | Purpose                                            |
| --------------- | -------------------------------------------------- |
| `execution()`   | Match method execution (**most common**)           |
| `within()`      | Match all methods inside a class or package        |
| `bean()`        | Match by Spring bean name                          |
| `@annotation()` | Match methods annotated with a specific annotation |
| `args()`        | Match methods based on argument types              |

For **1.5–2 years of experience**, you should primarily master:

* `execution()` ⭐⭐⭐⭐⭐
* `@Pointcut` ⭐⭐⭐⭐
* Basic understanding of `within()` and `@annotation()`

---

# Interview Questions

### Q1. What is a Pointcut?

A pointcut is an expression that selects the join points where advice should be applied.

---

### Q2. What does `execution(* com.demo.service.*.*(..))` mean?

It matches all methods in all classes inside the `com.demo.service` package, regardless of return type or parameters.

Let's break it down:

| Part               | Meaning                           |
| ------------------ | --------------------------------- |
| `*`                | Any return type                   |
| `com.demo.service` | Package                           |
| `*`                | Any class                         |
| `*`                | Any method                        |
| `(..)`             | Any number and type of parameters |

---

### Q3. Difference between `*` and `..`?

| Symbol | Meaning                                       |
| ------ | --------------------------------------------- |
| `*`    | Matches one name (class, method, return type) |
| `..`   | Matches multiple parameters or sub-packages   |

Examples:

```text
EmployeeService.*
```

→ All methods in `EmployeeService`.

```text
(..)
```

→ Any parameters.

```text
service..
```

→ Package and all its sub-packages.

---

### Q4. Why use `@Pointcut`?

To define a reusable pointcut expression so multiple advice methods can share the same filter, making the code cleaner and easier to maintain.

---

# Best Practices

* Use `@Pointcut` for reusable expressions.
* Keep pointcuts specific enough to avoid intercepting unintended methods.
* Avoid overly broad expressions like `execution(* *(..))` unless you intentionally want to intercept every method.

---

# 📍 Where We Are

```text
Spring AOP

✅ Why AOP
✅ Core Concepts
✅ Types of Advice
✅ Pointcut Expressions

Next →

Spring AOP Proxy
├── Why does Spring create a Proxy?
├── JDK Dynamic Proxy
├── CGLIB Proxy
├── How Spring chooses between them
├── Internal working
├── Interview questions
```

## The Next Chapter

The next topic is **Proxies**, which is one of the most frequently asked interview topics.

We'll answer questions like:

* What exactly is a proxy?
* Why doesn't Spring call the target object directly?
* What's the difference between **JDK Dynamic Proxy** and **CGLIB Proxy**?
* How does Spring decide which proxy type to use?
* How does a method call travel through the proxy to the target object?

Understanding proxies is the key to understanding how Spring AOP works internally.
