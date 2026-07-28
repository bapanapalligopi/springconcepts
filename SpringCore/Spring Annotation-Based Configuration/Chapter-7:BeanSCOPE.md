Great. Let's continue with the next Spring Core topic that is **very frequently asked in interviews**.

# Chapter 6: Bean Scope

---

# What is Bean Scope?

Before understanding bean scope, let's answer one question.

Suppose you have:

```java
@Component
public class Employee {

}
```

Now,

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

Employee e1 = context.getBean(Employee.class);

Employee e2 = context.getBean(Employee.class);
```

Question:

How many objects are created?

```text
1 ?
2 ?
```

This depends on the **Bean Scope**.

---

# Definition

**Bean Scope determines how many instances of a bean Spring creates and how long those instances live.**

In simple words,

> Scope tells Spring **how many objects to create** and **when to create them**.

---

# Types of Bean Scope

There are six scopes.

```text
1. Singleton ⭐⭐⭐⭐⭐
2. Prototype ⭐⭐⭐⭐⭐
3. Request
4. Session
5. Application
6. WebSocket
```

For a **1.5–2 years experienced developer**, focus mainly on:

* Singleton
* Prototype
* Request
* Session

---

# 1. Singleton Scope (Default)

This is the default scope in Spring.

Example:

```java
@Component
public class Employee {

}
```

This is equivalent to:

```java
@Component
@Scope("singleton")
public class Employee {

}
```

---

## Example

```java
@Component
public class Employee {

    public Employee() {
        System.out.println("Employee Object Created");
    }

}
```

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

Employee e1 = context.getBean(Employee.class);

Employee e2 = context.getBean(Employee.class);

System.out.println(e1);

System.out.println(e2);
```

Output

```text
Employee Object Created

com.demo.Employee@4ab

com.demo.Employee@4ab
```

Notice

Same memory address.

Only one object exists.

---

# Visualization

```text
ApplicationContext

        │

        ▼

Employee Bean

        │

        ▼

Employee@101

      ▲      ▲

      │      │

     e1     e2
```

Both variables refer to the **same object**.

---

# Why Singleton?

Imagine a Spring Boot application.

You have

```text
EmployeeService

↓

EmployeeRepository

↓

ProductService

↓

OrderService
```

Suppose Spring creates a new object every time.

```text
1000 Requests

↓

1000 EmployeeService Objects

↓

1000 Repository Objects
```

Huge memory consumption.

Instead,

Spring creates

```text
One EmployeeService

One Repository

One ProductService
```

and shares them.

---

# Real Example

Suppose

```java
@Service
public class EmailService {

}
```

Every user sends emails using the same service.

There is no need to create a new `EmailService` object for every request.

Singleton is the perfect choice.

---

# 2. Prototype Scope

Prototype means:

> **Every request gets a new object.**

Example

```java
@Component
@Scope("prototype")
public class Employee {

}
```

Now

```java
Employee e1 = context.getBean(Employee.class);

Employee e2 = context.getBean(Employee.class);
```

Output

```text
Employee@101

Employee@205
```

Different objects.

---

# Visualization

```text
getBean()

↓

Employee@101

----------------

getBean()

↓

Employee@205

----------------

getBean()

↓

Employee@307
```

Every call creates a new object.

---

# When Do We Use Prototype?

Imagine

```text
Shopping Cart
```

Every customer should have their own cart.

Creating one shared cart for all users would be incorrect.

Other examples:

* File processing jobs
* Temporary report generation
* PDF generation objects

---

# Singleton vs Prototype

| Singleton           | Prototype                          |
| ------------------- | ---------------------------------- |
| One object          | New object every request           |
| Default scope       | Must specify `@Scope("prototype")` |
| Shared              | Not shared                         |
| Better memory usage | More memory usage                  |

---

# Request Scope

This is used in **Spring MVC / Spring Boot Web Applications**.

```java
@Component
@RequestScope
public class RequestData {

}
```

One object is created **for each HTTP request**.

Example:

```text
User A

↓

RequestData@101

-----------------

User B

↓

RequestData@202
```

Each request gets its own instance.

---

# Session Scope

Used when data should live for an entire user session.

```java
@Component
@SessionScope
public class UserSession {

}
```

Example:

User A logs in.

```text
User A

↓

Session Object@100
```

The same object is used for all requests from User A until they log out or the session expires.

User B logs in.

```text
User B

↓

Session Object@205
```

Different user, different session object.

---

# Interview Question

## Why is Singleton the default scope?

Because most Spring beans are **stateless services**.

Example:

```java
@Service
public class EmployeeService {

}
```

The service doesn't store user-specific data.

One object can safely serve multiple requests, reducing memory usage and improving performance.

---

# Important Rule

Singleton beans should generally be **stateless**.

Good Example:

```java
@Service
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }

}
```

No instance variables.

Safe for multiple users.

---

Bad Example:

```java
@Service
public class EmployeeService {

    private String currentUser;

}
```

Imagine:

```text
User A

currentUser = Rahul

-----------------

User B

currentUser = Priya
```

Both users share the same singleton object.

The value gets overwritten.

This can cause incorrect behavior in a multi-user application.

---

# Interview Questions

### 1. What is the default scope in Spring?

**Answer:** Singleton.

---

### 2. Difference between Singleton and Prototype?

**Singleton:** One object per Spring container.

**Prototype:** A new object is created every time the bean is requested from the container.

---

### 3. When would you use Prototype scope?

When each usage needs a separate instance, such as report generation, temporary processing objects, or other stateful components.

---

### 4. Can Singleton beans be used in a multi-threaded application?

Yes, **provided they are stateless or properly synchronized**. Stateless singleton services are the standard approach in Spring Boot applications.

---

# Real Project Example

Consider this application:

```text
Employee Management System

↓

EmployeeController

↓

EmployeeService

↓

EmployeeRepository
```

All three are typically:

```text
Singleton

↓

One object each

↓

Shared by all requests
```

This is how most Spring Boot applications are built.

---

# Next Topic (Very Common Interview Question)

We'll now learn:

## **Bean Lifecycle**

You'll learn:

* How a Spring bean is created
* `@PostConstruct`
* `@PreDestroy`
* Initialization methods
* Destruction methods
* The complete bean lifecycle

Interviewers often ask:

> **"Explain the lifecycle of a Spring Bean."**

This is one of the most common Spring Core interview questions for 1–3 years of experience.
