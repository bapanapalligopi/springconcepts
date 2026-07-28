Excellent. Now we start **annotation-based configuration** from the inside out. Forget memorizing annotations—we'll understand how Spring actually works internally.

---

# Chapter 1: `@Component` (In Depth)

Before we study `@Component`, we need to answer one question.

> **How does Spring know which classes it should manage?**

Suppose your project contains 1000 classes.

```text
com.practice

├── Employee
├── Student
├── Address
├── Engine
├── Car
├── MathUtil
├── StringUtil
├── FileHelper
├── ExcelExporter
├── EmployeeService
├── EmployeeRepository
└── ...
```

Should Spring create objects for **all** these classes?

Obviously not.

For example:

* `MathUtil` contains only static methods.
* `StringUtil` is just a utility class.
* `ExcelExporter` may be created only when exporting data.

Spring needs a way to know:

> "Which classes should become Spring beans?"

This is exactly why `@Component` exists.

---

# What is `@Component`?

`@Component` is a **marker annotation**.

It tells Spring:

> **This class should be managed by the Spring IoC Container.**

Example

```java
@Component
public class Employee {

}
```

This annotation doesn't create the object by itself.

It only adds **metadata** to the class.

---

# What is Metadata?

Metadata means

> **Data about data.**

Example

```java
@Component
public class Employee {

}
```

Here

```java
@Component
```

is metadata.

It describes something about the class.

Similarly,

```java
@Override
```

describes a method.

```java
@Deprecated
```

describes that an API should no longer be used.

Annotations don't execute code. They provide information that another framework (Spring, the compiler, etc.) can read.

---

# Where is Annotation Stored?

When Java compiles your class

```java
@Component
public class Employee {

}
```

it becomes

```text
Employee.class
```

The annotation information is stored in the compiled `.class` file (provided its retention policy allows it).

Spring later reads that information using Java Reflection.

---

# Reflection (Very Important)

Spring heavily relies on **Reflection**.

Reflection allows Java to inspect classes while the application is running.

Example

```java
Class<?> cls = Employee.class;
```

Spring can ask

```java
cls.isAnnotationPresent(Component.class);
```

If the answer is

```text
true
```

Spring knows

> Create a bean.

---

# Internal Working (Simplified)

Imagine your package

```text
com.practice
```

contains

```text
Employee

Student

Car

Address
```

Spring scans every class.

Conceptually, it does something like:

```java
for (Class<?> cls : classes) {

    if (cls.isAnnotationPresent(Component.class)) {

        createBean(cls);

    }

}
```

This isn't the actual Spring source code, but it accurately represents the idea.

---

# Bean Creation

Suppose

```java
@Component
public class Employee {

}
```

Spring internally performs something similar to

```java
Constructor<?> constructor =
        Employee.class.getDeclaredConstructor();

Employee employee =
        (Employee) constructor.newInstance();
```

Notice

You never write

```java
new Employee();
```

Reflection creates the object.

---

# Why Reflection?

Because Spring doesn't know your classes at compile time.

Imagine you're developing Spring Framework.

Tomorrow someone writes

```java
@Component
public class BankService {

}
```

Spring has never seen this class before.

Yet it still creates

```java
new BankService();
```

How?

Using Reflection.

Reflection allows frameworks to work with classes that weren't known when the framework itself was compiled.

---

# Bean Registration

After creating the object,

Spring registers it.

Conceptually

```java
Map<String, Object> singletonObjects =
        new HashMap<>();

singletonObjects.put("employee", employee);
```

Real Spring uses more sophisticated internal caches, but this captures the idea.

---

# Bean Name Generation

Suppose

```java
@Component
public class EmployeeService {

}
```

Default bean name becomes

```text
employeeService
```

Rule

```text
Class Name

↓

Lowercase first letter

↓

employeeService
```

Examples

| Class Name      | Bean Name       |
| --------------- | --------------- |
| Employee        | employee        |
| Student         | student         |
| EmployeeService | employeeService |
| OrderRepository | orderRepository |

---

# Custom Bean Name

You can override it.

```java
@Component("empService")
public class EmployeeService {

}
```

Bean name

```text
empService
```

instead of

```text
employeeService
```

---

# What if Two Beans Have Same Name?

Example

```java
@Component("employee")
public class Employee {

}
```

```java
@Component("employee")
public class Student {

}
```

Spring starts

↓

Registers first bean

↓

Tries second bean

↓

Bean name already exists

↓

Throws an exception because two beans cannot share the same name.

---

# Does `@Component` Create Object Immediately?

Many beginners think

```java
@Component
public class Employee {

}
```

means

```java
new Employee();
```

Not exactly.

The annotation **only marks** the class.

The object is created later when the container processes it.

For singleton beans, creation typically happens during container startup (unless lazy initialization is enabled).

---

# Where Does `@ComponentScan` Fit?

Suppose you have

```java
@Component
public class Employee {
}
```

How does Spring find it?

Using

```java
@Configuration
@ComponentScan("com.practice")
public class AppConfig {
}
```

Flow

```text
Application Starts

↓

Read AppConfig

↓

@ComponentScan

↓

Scan Package

↓

Find @Component

↓

Create Bean

↓

Register Bean

↓

Ready
```

Without `@ComponentScan`, Spring doesn't automatically discover your `@Component` classes.

---

# XML vs Annotation

XML

```xml
<bean id="employee"
      class="com.practice.Employee"/>
```

Annotation

```java
@Component
public class Employee {
}
```

Both produce the same result.

Internally

```text
XML

↓

BeanDefinition

↓

IoC Container

---------------------

@Component

↓

BeanDefinition

↓

IoC Container
```

The configuration style changes.

The container remains the same.

---

# Common Misconceptions

### ❌ Misconception 1

`@Component` means object creation.

Wrong.

It marks the class as a candidate bean.

---

### ❌ Misconception 2

`@Component` performs dependency injection.

Wrong.

Dependency injection is handled by annotations like `@Autowired` (or XML configuration).

---

### ❌ Misconception 3

`@Component` scans packages.

Wrong.

`@ComponentScan` performs package scanning.

---

# Real Startup Flow

Imagine this project

```text
com.practice

config
    AppConfig

service
    EmployeeService

repository
    EmployeeRepository

controller
    EmployeeController
```

Startup sequence:

```text
main()

↓

AnnotationConfigApplicationContext

↓

Read AppConfig

↓

@ComponentScan

↓

Scan packages

↓

Reflection checks every class

↓

@Component ?

↓

YES

↓

Create BeanDefinition

↓

Instantiate Bean

↓

Store Singleton

↓

Container Ready
```

---

# Interview Questions

### 1. What is `@Component`?

A stereotype annotation that marks a class as a Spring-managed bean. During component scanning, Spring detects it, creates a bean definition, instantiates the bean (according to its scope), and registers it in the IoC container.

---

### 2. Does `@Component` create the object?

Not directly. It marks the class as a bean candidate. The Spring container creates the object when processing bean definitions.

---

### 3. How does Spring detect `@Component`?

Spring uses `@ComponentScan` to scan packages. It inspects compiled classes using Java Reflection, identifies classes annotated with `@Component` (or other stereotype annotations), and registers them as beans.

---

### 4. What is the default bean name?

The default bean name is the class name with the first letter converted to lowercase.

Example:

```text
EmployeeService

↓

employeeService
```

---

# Before Moving to `@Autowired`

There is one more foundational concept you should understand:

> **What exactly is a BeanDefinition?**

Many developers with 2–5 years of experience use Spring every day but cannot explain **how Spring converts a `@Component` into an actual bean**.

The missing link is **BeanDefinition**.

Once you understand `BeanDefinition`, the entire Spring startup process becomes clear:

```text
@Component
        ↓
Component Scan
        ↓
BeanDefinition Created
        ↓
BeanFactory Stores BeanDefinition
        ↓
Spring Instantiates Bean
        ↓
Dependency Injection
        ↓
Bean Ready
```

This is the level of understanding that helps you answer advanced Spring interview questions confidently, rather than just reciting annotation definitions.
