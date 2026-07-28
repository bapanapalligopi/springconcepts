Excellent! Now we're entering a topic that you'll use **every day** in Spring Boot development.

# Chapter 9: `@Value` and `application.properties`

This is one of the **most common interview and project topics**.

---

# Why Do We Need `application.properties`?

Imagine you're developing an Employee Management application.

You connect to MySQL like this:

```java
String url = "jdbc:mysql://localhost:3306/employeedb";
String username = "root";
String password = "root123";
```

Question:

Suppose tomorrow your application moves to another server.

New database:

```text
URL      : jdbc:mysql://192.168.1.100:3306/employeedb
Username : admin
Password : admin123
```

Should you open the Java file and change the code?

**No.**

Why?

* Every environment (Development, Testing, Production) has different configurations.
* Changing source code for configuration is bad practice.

So Spring provides a separate configuration file.

---

# application.properties

Example

```properties
server.port=8080

spring.datasource.url=jdbc:mysql://localhost:3306/employeedb

spring.datasource.username=root

spring.datasource.password=root123
```

Notice

No Java code changed.

Only the configuration file changes.

---

# Real Project

You may have three environments.

```text
Development

↓

application-dev.properties

----------------------

Testing

↓

application-test.properties

----------------------

Production

↓

application-prod.properties
```

Each environment has different values.

---

# Reading Values using `@Value`

Suppose

```properties
company.name=Infosys
```

You want this value inside a class.

```java
@Component
public class EmployeeService {

    @Value("${company.name}")
    private String companyName;

    public void print() {

        System.out.println(companyName);

    }

}
```

Output

```text
Infosys
```

Spring reads the property and injects it.

---

# How Does It Work?

Imagine this property file.

```properties
company.name=Infosys

company.location=Hyderabad
```

Spring loads it into memory.

Think of it like a map:

```text
company.name     → Infosys

company.location → Hyderabad
```

When Spring sees

```java
@Value("${company.name}")
```

it looks up the key:

```text
company.name
```

Finds

```text
Infosys
```

Injects it into the field.

---

# Example

## application.properties

```properties
employee.id=101

employee.name=Rahul

employee.salary=50000
```

---

## EmployeeService

```java
@Component
public class EmployeeService {

    @Value("${employee.id}")
    private int id;

    @Value("${employee.name}")
    private String name;

    @Value("${employee.salary}")
    private double salary;

    public void display() {

        System.out.println(id);

        System.out.println(name);

        System.out.println(salary);

    }

}
```

Output

```text
101

Rahul

50000
```

---

# Default Value

Suppose the property doesn't exist.

```java
@Value("${company.city}")
private String city;
```

Spring throws an exception during startup.

Instead

```java
@Value("${company.city:Hyderabad}")
private String city;
```

Output

```text
Hyderabad
```

If the property exists, Spring uses it.

Otherwise,

Spring uses the default value.

---

# Injecting Primitive Types

Properties

```properties
employee.age=25

employee.active=true
```

Java

```java
@Value("${employee.age}")
private int age;

@Value("${employee.active}")
private boolean active;
```

Spring automatically converts String values into the appropriate Java types.

---

# Constructor Injection with `@Value`

Instead of field injection,

you can inject through the constructor.

```java
@Component
public class EmployeeService {

    private final String company;

    public EmployeeService(
            @Value("${company.name}") String company) {

        this.company = company;

    }

}
```

This works well with constructor injection and immutable fields.

---

# Real Project Example

Imagine you're using Redis.

```properties
spring.data.redis.host=localhost

spring.data.redis.port=6379
```

Spring Boot reads these properties and configures the Redis connection automatically.

Similarly,

```properties
server.port=9090
```

starts your application on port **9090** instead of the default **8080**.

---

# Common Properties

```properties
server.port=8080

spring.application.name=Employee-Service

spring.datasource.url=jdbc:mysql://localhost:3306/employeedb

spring.datasource.username=root

spring.datasource.password=root123

spring.jpa.hibernate.ddl-auto=update

spring.jpa.show-sql=true
```

You'll use these almost every day in Spring Boot projects.

---

# `application.properties` vs Hardcoded Values

### Hardcoded

```java
String company = "Infosys";
```

Problems:

* Requires code changes for configuration changes.
* Different environments need different builds.
* Difficult to maintain.

---

### Property File

```properties
company.name=Infosys
```

```java
@Value("${company.name}")
private String company;
```

Advantages:

* No code changes.
* Easy to maintain.
* Environment-specific configurations.
* Standard Spring Boot practice.

---

# Interview Questions

### 1. What is `@Value`?

`@Value` is used to inject values from configuration files, system properties, environment variables, or expressions into Spring-managed beans.

---

### 2. Why do we use `application.properties`?

To keep configuration separate from source code, making applications easier to configure across different environments.

---

### 3. Can `@Value` inject only Strings?

No.

Spring automatically converts values to compatible types such as:

* `int`
* `long`
* `double`
* `boolean`
* `String`

---

### 4. How do you provide a default value?

```java
@Value("${company.city:Hyderabad}")
```

If `company.city` is missing,

Spring injects

```text
Hyderabad
```

---

### 5. Where have you used `@Value` in your project?

A good interview answer:

> "I have used `@Value` to read configuration values such as API URLs, feature flags, timeout values, file paths, and application-specific properties from `application.properties`."

---

# Important Note (Spring Boot)

For **a few properties**, `@Value` is perfect.

But suppose you have many related properties.

Example:

```properties
employee.id=101
employee.name=Rahul
employee.salary=50000
employee.department=IT
employee.city=Hyderabad
```

Using five `@Value` annotations works, but it becomes difficult to maintain.

Spring Boot provides a better approach:

```java
@ConfigurationProperties(prefix = "employee")
```

This binds all related properties to a single Java class.

👉 For **1.5–2 years experience**, interviewers often ask the difference between `@Value` and `@ConfigurationProperties`. We'll cover that when we move into Spring Boot.

---

# Summary

```text
application.properties
        │
        ▼
company.name=Infosys
        │
        ▼
@Value("${company.name}")
        │
        ▼
company = "Infosys"
```

---

## We've Covered in Spring Core

✅ IoC Container
✅ Dependency Injection
✅ XML Configuration
✅ Annotation Configuration
✅ `@Component`
✅ `@ComponentScan`
✅ `@Autowired`
✅ `@Qualifier`
✅ `@Primary`
✅ Constructor vs Setter vs Field Injection
✅ Bean Scope
✅ Bean Lifecycle (`@PostConstruct`, `@PreDestroy`)
✅ `@Bean` and `@Configuration`
✅ `@Value`

---

## Next Topic

The next topic is **Spring Profiles (`@Profile`)**.

You'll learn:

* What are Profiles?
* Why do we need different configurations for Dev, Test, and Prod?
* How Spring selects the active profile
* Real project examples
* Interview questions

After that, we'll move into **Spring AOP**, which is another major interview topic for Spring developers.
