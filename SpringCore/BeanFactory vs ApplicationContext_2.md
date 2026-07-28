Excellent. Now we'll move to the **most important topic in Spring Core**.

# Lesson 2: BeanFactory vs ApplicationContext

This is one of the most frequently asked Spring interview questions.

---

# What is an IoC Container?

An **IoC (Inversion of Control) Container** is responsible for:

* Creating objects (beans)
* Managing bean lifecycle
* Injecting dependencies
* Providing beans when requested

Spring provides two main IoC containers:

```
BeanFactory
      ↑
ApplicationContext
```

`ApplicationContext` extends `BeanFactory`, so it has all the features of `BeanFactory` plus many more.

---

# BeanFactory

`BeanFactory` is the **basic IoC container**.

Example:

```java
BeanFactory factory =
    new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));
```

> **Note:** `XmlBeanFactory` has been **deprecated and removed** in modern Spring versions (including Spring 7). You'll see it only in older tutorials.

### Characteristics

* Basic IoC container
* Creates beans **lazily**
* Suitable for lightweight applications
* Fewer enterprise features

---

# What is Lazy Initialization?

Suppose you have:

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"/>
```

With `BeanFactory`:

```
BeanFactory Created
        │
        ▼
Employee NOT created
        │
        ▼
getBean("employee")
        │
        ▼
Employee object created
```

The bean is created **only when you ask for it**.

---

# ApplicationContext

`ApplicationContext` is the **advanced IoC container**.

Example:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

This is what we are using.

---

### Characteristics

* Advanced IoC container
* Creates singleton beans eagerly (by default)
* Supports events
* Internationalization (i18n)
* AOP integration
* Resource loading
* Environment and profile support

---

# What is Eager Initialization?

Suppose:

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"/>
```

When this line executes:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

Spring immediately creates the bean.

```
ApplicationContext Created
        │
        ▼
Reads XML
        │
        ▼
Creates Employee
        │
        ▼
Stores in Container
```

Even if you never call:

```java
context.getBean("employee");
```

the bean is already created (for singleton beans).

---

# Example

### Employee.java

```java
package com.practice.springcore;

public class Employee {

    public Employee() {
        System.out.println("Employee Constructor Called");
    }

}
```

### Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

System.out.println("Context Loaded");
```

### Output

```
Employee Constructor Called
Context Loaded
```

Notice:

The constructor runs **before** you call `getBean()`.

---

# Verify It Yourself

Now add this line:

```java
Employee employee =
        context.getBean("employee", Employee.class);
```

### Output

```
Employee Constructor Called
Context Loaded
```

The constructor is **not called again** because Spring returns the same singleton bean.

---

# BeanFactory vs ApplicationContext

| Feature              | BeanFactory | ApplicationContext                  |
| -------------------- | ----------- | ----------------------------------- |
| IoC Container        | ✅           | ✅                                   |
| Dependency Injection | ✅           | ✅                                   |
| Lazy Initialization  | ✅           | ❌ (Singletons are eager by default) |
| Eager Initialization | ❌           | ✅                                   |
| AOP Support          | Limited     | ✅                                   |
| Event Handling       | ❌           | ✅                                   |
| Internationalization | ❌           | ✅                                   |
| BeanPostProcessor    | Basic       | ✅                                   |
| Annotation Support   | Limited     | ✅                                   |
| Used Today           | Rarely      | ✅ Almost always                     |

---

# Why Does Spring Boot Use ApplicationContext?

Spring Boot applications are enterprise applications.

They require:

* Events
* AOP
* Transactions
* Security
* Validation
* Internationalization

All of these are available through `ApplicationContext`.

---

# Interview Questions

### 1. What is BeanFactory?

**Answer:**

> BeanFactory is the basic IoC container in Spring. It manages bean creation and dependency injection, and creates beans lazily when they are first requested.

---

### 2. What is ApplicationContext?

**Answer:**

> ApplicationContext is an advanced IoC container that extends BeanFactory. It provides all BeanFactory features along with support for eager bean initialization, event handling, AOP integration, internationalization, and resource management.

---

### 3. Difference between BeanFactory and ApplicationContext?

The key differences are:

* BeanFactory creates beans lazily.
* ApplicationContext creates singleton beans eagerly by default.
* ApplicationContext offers additional enterprise features like AOP, events, and internationalization.
* Modern Spring applications and Spring Boot use ApplicationContext.

---

# Practice Exercise

Modify `Employee`:

```java
public class Employee {

    public Employee() {
        System.out.println("Constructor Called");
    }
}
```

Then in `App.java`:

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

System.out.println("Application Started");
```

**Question:** What do you think the output will be?

<details>
<summary>Answer</summary>

```
Constructor Called
Application Started
```

This demonstrates that `ApplicationContext` eagerly creates singleton beans during startup.

</details>

---

# Next Lesson: Dependency Injection (DI) with XML

This is the heart of Spring Core. We'll cover:

* Constructor Injection
* Setter Injection
* Injecting primitive values (`String`, `int`, etc.)
* Injecting object references
* Constructor Injection vs Setter Injection
* Real-world examples (e.g., `Car` → `Engine`, `Employee` → `Address`)
