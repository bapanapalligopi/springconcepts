Excellent. Now we're entering one of the **most important Spring interview topics**.

Many developers know how to write:

```java
@Before(...)
public void log() {}
```

But when the interviewer asks:

> **"How does Spring execute this method without modifying your EmployeeService?"**

many struggle.

The answer is **Proxy**.

We'll follow our learning framework:

> **Why → What → How → Where → Code → Internal Working → Interview Questions → Best Practices**

---

# Chapter 5: Spring Proxy (Heart of Spring AOP)

---

# 1. Why? (The Problem)

Suppose you have this service.

```java
@Service
public class EmployeeService {

    public void saveEmployee() {
        System.out.println("Employee Saved");
    }
}
```

Controller

```java
@RestController
public class EmployeeController {

    @Autowired
    private EmployeeService service;

    public void save() {
        service.saveEmployee();
    }
}
```

Everything looks simple.

---

Now suppose you add logging.

Question:

Where should Spring execute this?

```
Method Started
```

Before

```java
saveEmployee();
```

Your code is

```java
public void saveEmployee() {

    System.out.println("Employee Saved");

}
```

There is no logging.

So...

**Who prints "Method Started"?**

---

### Possible Solution 1

Modify your method.

```java
public void saveEmployee() {

    System.out.println("Method Started");

    System.out.println("Employee Saved");

}
```

Problem:

Business logic and logging become mixed.

Not good.

---

### Solution 2

Spring creates another object.

Instead of

```
Controller

↓

EmployeeService
```

Spring changes it into

```
Controller

↓

Proxy

↓

EmployeeService
```

The proxy handles logging.

---

# 2. What is a Proxy?

Simple definition:

> A **Proxy** is an object that stands between the caller and the actual object.

Think of it as a middleman.

---

### Real-Life Analogy

Suppose you visit a bank.

```
You

↓

Reception

↓

Bank Manager
```

You don't directly meet the manager.

The receptionist:

* Verifies your identity
* Checks your appointment
* Directs you

The receptionist is the **proxy**.

The manager is the **target object**.

---

### Spring Example

```
Controller

↓

Proxy

↓

EmployeeService
```

The proxy:

* Logs
* Checks security
* Starts transactions
* Calls the real method

---

# 3. How Does Proxy Work?

Without AOP

```
Controller

↓

EmployeeService

↓

saveEmployee()
```

---

With AOP

```
Controller

↓

Spring Proxy

↓

@Before Advice

↓

EmployeeService.saveEmployee()

↓

@After Advice

↓

Return
```

Notice:

The controller doesn't know the proxy exists.

It simply calls

```java
service.saveEmployee();
```

Spring secretly replaces the original bean with a proxy bean.

---

# Internal Working

Application starts.

```
Spring Starts

↓

Finds @Service

↓

Creates EmployeeService

↓

Finds @Aspect

↓

Creates LoggingAspect

↓

Creates Proxy

↓

Stores Proxy in IoC Container

↓

Application Ready
```

Notice carefully.

Spring **does not store the original EmployeeService** in the IoC container for injection.

Instead,

```
IoC Container

↓

EmployeeService Proxy
```

Whenever you write

```java
@Autowired
private EmployeeService service;
```

Spring injects

```
EmployeeService Proxy
```

NOT

```
EmployeeService
```

This is an extremely common interview question.

---

# Method Execution Flow

Suppose

```java
service.saveEmployee();
```

Flow

```
Controller

↓

EmployeeService Proxy

↓

Execute @Before

↓

Execute EmployeeService

↓

Execute @AfterReturning

↓

Execute @After

↓

Return
```

Every method goes through the proxy.

---

# 4. Types of Proxy

Spring mainly uses two proxy mechanisms.

```
1. JDK Dynamic Proxy

2. CGLIB Proxy
```

These are interview favorites.

Let's understand both.

---

# JDK Dynamic Proxy

## Why?

Suppose

```java
public interface EmployeeService {

    void saveEmployee();

}
```

Implementation

```java
@Service
public class EmployeeServiceImpl
        implements EmployeeService {

    @Override
    public void saveEmployee() {

        System.out.println("Saved");

    }

}
```

Since an interface exists,

Java already provides a way to create a proxy.

Spring uses

```
JDK Dynamic Proxy
```

---

Visualization

```
Controller

↓

JDK Proxy

↓

EmployeeServiceImpl
```

---

### Rule

If the target bean implements an interface,

Spring can use

```
JDK Dynamic Proxy
```

---

# CGLIB Proxy

Suppose

```java
@Service
public class EmployeeService {

    public void saveEmployee() {

    }

}
```

No interface.

Question

How can Spring create a proxy?

Answer

It creates a subclass.

```
EmployeeService

↓

EmployeeService$$SpringProxy
```

That subclass overrides methods and inserts AOP logic.

This is called

```
CGLIB Proxy
```

---

Visualization

```
Controller

↓

CGLIB Proxy

↓

EmployeeService
```

---

### Rule

No interface?

Spring uses

```
CGLIB
```

---

# JDK vs CGLIB

| JDK Dynamic Proxy                          | CGLIB Proxy                            |
| ------------------------------------------ | -------------------------------------- |
| Uses interfaces                            | Uses inheritance (subclassing)         |
| Requires an interface                      | No interface required                  |
| Proxies interface methods                  | Proxies class methods                  |
| Creates a proxy implementing the interface | Creates a subclass of the target class |

---

# Which One Does Spring Boot Use?

This is a very common interview question.

### Spring Framework

Historically:

```
Interface Present

↓

JDK Proxy

Else

↓

CGLIB
```

### Spring Boot

By default,

**Spring Boot configures class-based (CGLIB) proxies** for many AOP use cases, even if an interface exists, unless you explicitly configure interface-based proxies.

For interviews, remember:

* **Spring Framework concept:** JDK proxy for interfaces, CGLIB otherwise.
* **Spring Boot default:** CGLIB (class-based proxies) is commonly the default configuration.

---

# Real Project Example

Suppose

```
Order Service

↓

@Transactional
```

Question

Who starts the transaction?

Not your service.

The proxy.

Flow

```
Proxy

↓

Start Transaction

↓

OrderService.save()

↓

Commit

↓

Return
```

Exactly the same idea applies to:

* `@Cacheable`
* `@Async`
* `@Retryable`
* Method security annotations
* Custom AOP aspects

---

# Important Limitation

Consider this class:

```java
@Service
public class EmployeeService {

    public void saveEmployee() {
        updateEmployee();   // Internal call
    }

    public void updateEmployee() {
        System.out.println("Updated");
    }
}
```

Suppose

```java
@Before(...)
```

is applied on

```
updateEmployee()
```

Question

Will it execute?

Answer

**No**, not in this scenario.

Why?

Because

```
saveEmployee()

↓

updateEmployee()
```

is an **internal method call**.

The call never goes through the proxy.

Only external calls go through the proxy.

This is one of the most asked Spring AOP interview questions.

---

# Complete Architecture

```
                 IoC Container
                       │
       ┌───────────────┴───────────────┐
       │                               │
 LoggingAspect                  EmployeeService
       │                               │
       └───────────────┬───────────────┘
                       │
                 Spring Creates
                     Proxy
                       │
                 EmployeeServiceProxy
                       │
                Injected Everywhere
                       │
                  Controller Calls
                       │
        Before → Target → After → Return
```

---

# Interview Questions

### Q1. What is a Proxy?

A proxy is an object created by Spring that sits between the client and the target object to intercept method calls and execute cross-cutting concerns such as logging, transactions, or security.

---

### Q2. Why does Spring use proxies?

To apply additional behavior **without modifying the original business class**.

---

### Q3. Difference between JDK Proxy and CGLIB?

* **JDK Dynamic Proxy**: Works with interfaces and creates a proxy that implements those interfaces.
* **CGLIB Proxy**: Creates a subclass of the target class, so it works even when no interface is present.

---

### Q4. Does Spring inject the original bean?

No.

Spring injects the **proxy object**, which delegates calls to the original bean after applying any configured advice.

---

### Q5. Why doesn't AOP work on self-invocation?

Because a method calling another method within the same class bypasses the proxy. Since the proxy isn't involved, the advice doesn't execute.

---

# Best Practices

* Prefer **constructor injection** so your dependencies are clear and immutable.
* Avoid relying on AOP for internal method calls.
* Use interfaces when they make sense architecturally, not just to force JDK proxies.
* Use AOP for infrastructure concerns (logging, security, transactions), not business rules.

---

# 📍 Where We Are

```
Spring AOP

✅ Why AOP
✅ Core Concepts
✅ Advice
✅ Pointcuts
✅ Proxy
✅ JDK Dynamic Proxy
✅ CGLIB Proxy

Next →

Complete Spring AOP Project
        ↓
Logging Aspect
        ↓
Execution Time Aspect
        ↓
Exception Logging
        ↓
Interview-Level Project
```

---

## Before We Move On

We've now completed **the theory of Spring AOP**.

The next step is to build a **complete Spring Boot AOP project** from scratch. We'll create:

* `EmployeeController`
* `EmployeeService`
* `EmployeeRepository`
* `LoggingAspect`
* `ExecutionTimeAspect`

This will show you how all the concepts—`@Aspect`, `@Pointcut`, `@Before`, `@AfterReturning`, `@Around`, proxies, and pointcuts—work together in a real application. That's exactly the kind of understanding interviewers look for from a **2-year Spring Boot developer**.
