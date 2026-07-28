Excellent! You're progressing exactly in the right order. **Bean Scope** is one of the most important Spring Core topics and is frequently asked in interviews.

---

# Lesson 6: Bean Scope in Spring

## What is Bean Scope?

**Bean scope defines how many instances of a bean Spring should create and how long they should live.**

Think of it like this:

> "When I ask Spring for a bean, should it return the same object every time or create a new one?"

---

# Types of Bean Scopes

There are **6 scopes** in Spring.

| Scope         | Description                      | Used In    |
| ------------- | -------------------------------- | ---------- |
| `singleton`   | One object per Spring Container  | Core & Web |
| `prototype`   | New object every request         | Core & Web |
| `request`     | One object per HTTP request      | Web only   |
| `session`     | One object per HTTP session      | Web only   |
| `application` | One object per ServletContext    | Web only   |
| `websocket`   | One object per WebSocket session | Web only   |

For Spring Core, focus mainly on:

* ✅ Singleton
* ✅ Prototype

The other scopes are used in Spring MVC/Spring Boot web applications.

---

# 1. Singleton Scope (Default)

This is the **default scope**.

## Employee.java

```java
package com.practice.springcore;

public class Employee {

    public Employee() {
        System.out.println("Employee Created");
    }

}
```

---

## XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"/>
```

Notice:

There is **no scope** specified.

Spring assumes:

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"
      scope="singleton"/>
```

---

## Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

Employee e1 = context.getBean("employee", Employee.class);

Employee e2 = context.getBean("employee", Employee.class);

System.out.println(e1 == e2);
```

---

## Output

```text
Employee Created
true
```

Only one object is created.

Both variables point to the same object.

```text
e1
 │
 ▼
Employee Object
 ▲
 │
e2
```

---

# Memory Representation

```text
Spring Container

Employee Bean
      │
      ▼
Employee@101
```

Whenever you call

```java
getBean("employee")
```

Spring returns

```text
Employee@101
```

---

# 2. Prototype Scope

Prototype means:

> Create a **new object every time** someone requests the bean.

---

## XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"
      scope="prototype"/>
```

---

## Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

Employee e1 = context.getBean("employee", Employee.class);

Employee e2 = context.getBean("employee", Employee.class);

System.out.println(e1 == e2);
```

---

## Output

```text
Employee Created
Employee Created
false
```

Spring created **two different objects**.

```text
e1
 │
 ▼
Employee@101

e2
 │
 ▼
Employee@205
```

---

# Singleton vs Prototype

## Singleton

```text
Container

Employee@101

↓

e1

↓

e2

↓

e3
```

Everyone gets the same object.

---

## Prototype

```text
getBean()

↓

Employee@101

getBean()

↓

Employee@205

getBean()

↓

Employee@301
```

Every call creates a new object.

---

# Real-World Example

### Singleton

Suitable for:

* Services
* Repositories
* Controllers
* Configuration classes

Example:

```text
EmployeeService

↓

One instance

↓

Used by all requests
```

Creating a new `EmployeeService` object for every request would waste memory.

---

### Prototype

Suitable for:

* Shopping Cart
* Report Generator
* File Processor
* Temporary objects

Each user needs their own object.

---

# Why Singleton is Default?

Because creating objects is expensive.

Suppose you have:

```text
1000 users
```

If every request creates:

```text
EmployeeService
```

then Spring would create:

```text
1000 EmployeeService objects
```

Instead,

Spring creates:

```text
1 EmployeeService
```

and everyone shares it.

This improves:

* Memory usage
* Performance
* Startup time

---

# Important Interview Question

### Q1. What is the default scope in Spring?

**Answer:**

The default bean scope in Spring is **singleton**.

Only one instance of the bean is created per Spring IoC container.

---

### Q2. Difference between Singleton and Prototype?

| Singleton          | Prototype                |
| ------------------ | ------------------------ |
| One object         | New object every request |
| Default scope      | Must be specified        |
| Shared instance    | Separate instances       |
| Better performance | More memory usage        |

---

### Q3. What happens if you call `getBean()` twice?

#### Singleton

```java
Employee e1 = context.getBean("employee");
Employee e2 = context.getBean("employee");
```

Result:

```java
e1 == e2
```

```text
true
```

---

#### Prototype

```java
Employee e1 = context.getBean("employee");
Employee e2 = context.getBean("employee");
```

Result:

```java
e1 == e2
```

```text
false
```

---

# Bean Lifecycle and Scope

For **singleton** beans:

```text
Container Starts
      │
Create Bean
      │
Use Bean
      │
Destroy Bean
```

Spring manages the **entire lifecycle**.

---

For **prototype** beans:

```text
getBean()

↓

Create Bean

↓

Return Bean

↓

Spring stops managing it
```

Spring creates the bean but **does not manage its destruction**. If the bean holds resources (files, sockets, database connections), your application is responsible for cleaning them up.

---

# Practice Exercise

Create this XML:

```xml
<bean id="student"
      class="com.practice.springcore.Student"
      scope="prototype"/>
```

Then run:

```java
Student s1 = context.getBean("student", Student.class);
Student s2 = context.getBean("student", Student.class);

System.out.println(s1 == s2);
```

### Predict the output before running it.

**Answer:**

```text
Student Created
Student Created
false
```

---

# What You've Learned So Far

* ✅ Spring Framework Basics
* ✅ IoC (Inversion of Control)
* ✅ Dependency Injection
* ✅ BeanFactory vs ApplicationContext
* ✅ XML Configuration
* ✅ Constructor Injection
* ✅ Setter Injection
* ✅ Collection Injection
* ✅ Bean Scopes

---

# Next Lesson (Very Important)

We'll learn **Bean Lifecycle**, where you'll understand:

* `init-method`
* `destroy-method`
* `InitializingBean`
* `DisposableBean`
* `BeanPostProcessor`

This is another popular interview topic and explains **what Spring does immediately after creating a bean and just before destroying it**. Once you understand the lifecycle in XML, annotations like `@PostConstruct` and `@PreDestroy` will be much easier to grasp.
