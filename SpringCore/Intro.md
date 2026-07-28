Excellent choice. If you understand **Spring Core with XML configuration**, then annotations (`@Component`, `@Autowired`, etc.) will make much more sense because you'll know what Spring is doing behind the scenes.

We'll learn this like a classroom course. **Lesson 1** focuses on the foundation.

---

# Spring Core Course

## Module 1: Introduction to Spring Core

Topics we'll cover:

* ✅ What is Spring?
* ✅ Why was Spring introduced?
* ✅ What is Spring Core?
* ✅ What is IoC (Inversion of Control)?
* ✅ What is Dependency Injection?
* ✅ What is a Spring Container?
* ✅ First XML Configuration Project

---

# 1. Why was Spring introduced?

Before Spring, Java developers used plain Java objects and created dependencies manually.

Example:

```java
public class Engine {

    public void start() {
        System.out.println("Engine Started");
    }

}
```

```java
public class Car {

    Engine engine = new Engine();

    public void drive() {
        engine.start();
        System.out.println("Car Running");
    }

}
```

Here,

```text
Car
 │
 ▼
Engine
```

The `Car` class **creates** the `Engine` object.

### What's the problem?

* Tight coupling
* Difficult to test
* Hard to replace implementations

Suppose tomorrow you want to replace:

```text
PetrolEngine
```

with

```text
DieselEngine
```

You must modify the `Car` class.

This violates the **Open/Closed Principle**.

---

# 2. What is Spring?

Spring is a lightweight framework that **creates, configures, and manages objects** for you.

Instead of:

```java
Engine engine = new Engine();
```

Spring creates the object.

---

# 3. Spring Core

Spring Core is the **foundation module** of the Spring Framework.

It provides:

* IoC Container
* Dependency Injection
* Bean Management
* Bean Lifecycle

Without Spring Core, none of the other Spring modules work.

---

# 4. What is IoC (Inversion of Control)?

Normally, your application controls object creation.

Example:

```java
Employee employee = new Employee();
```

Here,

**Your code controls object creation.**

With Spring:

```text
Spring Container
      │
Creates Employee
      │
Gives it to your application
```

The control of object creation is transferred from your code to the Spring container.

This is called **Inversion of Control (IoC)**.

### Interview Definition

> IoC is a design principle in which the responsibility for creating and managing objects is transferred from the application code to the Spring IoC container.

---

# 5. What is Dependency Injection?

Consider these classes:

```java
public class Engine {

}
```

```java
public class Car {

    Engine engine;

}
```

A `Car` depends on an `Engine`.

Without Spring:

```java
Engine engine = new Engine();

Car car = new Car(engine);
```

With Spring:

```text
Spring Container

Creates Engine

↓

Creates Car

↓

Injects Engine into Car
```

This is called **Dependency Injection (DI)**.

### Interview Definition

> Dependency Injection is the process of supplying the required dependencies to an object instead of the object creating them itself.

---

# 6. What is a Spring Bean?

A bean is simply an object managed by the Spring container.

Example:

Without Spring:

```java
Employee employee = new Employee();
```

Not a bean.

With Spring:

```xml
<bean id="employee" class="com.practice.Employee"/>
```

Now the `Employee` object is created and managed by Spring.

---

# 7. What is the Spring Container?

The Spring Container is responsible for:

* Creating objects
* Injecting dependencies
* Managing bean lifecycle
* Managing bean scope

Think of it as an **object factory with extra features**.

```text
Application

↓

Spring Container

↓

Employee Bean

↓

Engine Bean

↓

Car Bean
```

---

# 8. XML Configuration

Before annotations existed, Spring applications were configured using XML.

Example:

```xml
<beans>

    <bean id="employee"
          class="com.practice.Employee"/>

</beans>
```

Spring reads this file.

Creates an object.

Stores it in memory.

Returns it whenever requested.

---

# First Spring XML Project

## Step 1

Create a Maven project.

---

## Step 2

`pom.xml`

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.39</version>
    </dependency>

</dependencies>
```

---

## Step 3

Create:

```
src/main/java

    com.practice.springcore

        Employee.java
```

```java
package com.practice.springcore;

public class Employee {

    public void display() {

        System.out.println("Employee Bean Created");

    }

}
```

---

## Step 4

Create:

```
src/main/resources

applicationContext.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employee"
          class="com.practice.springcore.Employee"/>

</beans>
```

---

## Step 5

Create Main Class

```java
package com.practice.springcore;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

    public static void main(String[] args) {

        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");

        Employee employee =
                context.getBean("employee", Employee.class);

        employee.display();

    }

}
```

---

## Output

```
Employee Bean Created
```

---

# What Happened Internally?

When you write:

```java
ApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");
```

Spring performs these steps:

```text
Reads applicationContext.xml
          │
          ▼
Finds <bean> definitions
          │
          ▼
Creates Employee object
          │
          ▼
Stores it in the IoC Container
          │
          ▼
Returns ApplicationContext
```

Then:

```java
Employee employee = context.getBean("employee", Employee.class);
```

Spring searches its container for the bean with ID `employee` and returns the already-managed object.

---

# Important Interview Questions

1. What is Spring?
2. What is Spring Core?
3. What is IoC?
4. What is Dependency Injection?
5. What is a Spring Bean?
6. What is the Spring Container?
7. What is `ApplicationContext`?
8. What is `ClassPathXmlApplicationContext`?
9. What is the difference between `BeanFactory` and `ApplicationContext`? *(We'll cover this in the next lesson.)*

---

## Homework

Build the project exactly as shown and answer these questions:

1. What happens if you remove the `<bean>` from `applicationContext.xml` and call `getBean("employee")`?
2. What happens if you change the bean ID from `"employee"` to `"emp"` but still call `getBean("employee")`?
3. What happens if you create two `<bean>` definitions for the same `Employee` class with different IDs?

These exercises will help you understand how the Spring IoC container behaves before we move on to dependency injection in XML.
Your `pom.xml` looks mostly fine, but I have a few recommendations before we proceed.

---

# 1. You don't need all these dependencies

Since `spring-context` already depends on `spring-core`, `spring-beans`, and other required Spring modules, you can simplify your `pom.xml`.

A cleaner version is:

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>7.0.8</version>
    </dependency>

</dependencies>
```

Maven will automatically download:

* spring-core
* spring-beans
* spring-expression
* spring-jcl

---

# 2. Remove this property

You have:

```xml
<spring.version>5.3.39</spring.version>
```

but you're actually using:

```xml
7.0.8
```

So either remove it or change it to:

```xml
<spring.version>7.0.8</spring.version>
```

---

# 3. Java Version

You're compiling with Java 25.

```xml
<maven.compiler.source>25</maven.compiler.source>
<maven.compiler.target>25</maven.compiler.target>
```

That's okay.

For interview preparation, however, most companies currently use:

* Java 17
* Java 21

So later I recommend switching to Java 21 or 17, but Java 25 will work for learning.

---

# 4. Your XML

From the screenshot, the XML declaration should be

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

Notice the closing `?>`.

---

# 5. If STS still shows an error

Right click

```
Project
    ↓
Maven
    ↓
Update Project
```

or press

```
Alt + F5
```

If it still has an error

```
Project
    ↓
Clean
```

---

# 6. Verify your project structure

```
springframework
│
├── src
│   ├── main
│   │   ├── java
│   │   │     └── com.practice.springcore
│   │   │           ├── Employee.java
│   │   │           └── App.java
│   │   │
│   │   └── resources
│   │          └── applicationContext.xml
│   │
│   └── test
│
└── pom.xml
```

---

# 7. Let's write the first Spring program

### Employee.java

```java
package com.practice.springcore;

public class Employee {

    public Employee() {
        System.out.println("Employee Object Created");
    }

    public void display() {
        System.out.println("Welcome to Spring Core");
    }
}
```

---

### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employee"
          class="com.practice.springcore.Employee"/>

</beans>
```

---

### App.java

```java
package com.practice.springcore;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

    public static void main(String[] args) {

        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");

        Employee employee =
                context.getBean("employee", Employee.class);

        employee.display();

    }

}
```

Expected output:

```
Employee Object Created
Welcome to Spring Core
```

---

# How does Spring create the object?

This is the key concept.

When the following line executes:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

Spring performs these steps:

```
Step 1
Read applicationContext.xml
        │
        ▼
Step 2
Find all <bean> tags
        │
        ▼
Step 3
Load Employee.class
        │
        ▼
Step 4
Call new Employee()
        │
        ▼
Step 5
Store the object inside the IoC Container
        │
        ▼
Step 6
Return ApplicationContext
```

Then when you call:

```java
Employee emp = context.getBean("employee", Employee.class);
```

Spring **does not create a new object**. It returns the already-created bean (by default, beans are singletons).

---

## One important question before we continue

**Are you learning Spring Core for interviews, or do you want to understand how Spring works internally?**

The reason I ask is that I can teach it in two different ways:

1. **Interview-oriented** (fast, practical, enough for 2 years' experience).
2. **Framework-oriented** (deep dive into IoC container, BeanFactory, ApplicationContext internals, bean lifecycle, XML parsing, and how Spring actually works under the hood).

Given the questions you've been asking so far, I think you'd enjoy the second approach because it builds a much stronger understanding before moving to annotations and Spring Boot.
