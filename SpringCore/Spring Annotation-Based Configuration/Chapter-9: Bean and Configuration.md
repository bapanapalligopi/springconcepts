Excellent. This is one of the **most practical Spring Core topics**. Every Spring Boot project you work on will contain `@Configuration` and `@Bean`.

---

# Chapter 8: @Bean and @Configuration

Before learning these annotations, let's answer one important question.

We already know this:

```java
@Component
public class EmployeeService {

}
```

Spring creates the bean automatically.

So why do we need another annotation called `@Bean`?

---

# The Problem

Suppose you have written this class.

```java
@Component
public class EmployeeService {

}
```

Spring can create this bean because **you own this class**.

You can simply add

```java
@Component
```

No problem.

---

But imagine you want to use a third-party library.

Example:

```java
ObjectMapper mapper = new ObjectMapper();
```

or

```java
RestTemplate restTemplate = new RestTemplate();
```

or

```java
RedisTemplate redisTemplate = new RedisTemplate();
```

Question:

Can you modify these classes and add

```java
@Component
```

?

No.

Why?

Because these classes belong to external libraries.

You don't own their source code.

---

# So How Does Spring Manage Them?

Spring gives us another mechanism.

Instead of marking the class,

we tell Spring

> "Call this method and register whatever it returns as a bean."

This is done using

```java
@Bean
```

---

# Example

```java
@Configuration
public class AppConfig {

    @Bean
    public Employee employee() {

        return new Employee();

    }

}
```

What happens?

Application starts

↓

Spring reads

```java
@Configuration
```

↓

Finds

```java
@Bean
```

↓

Calls

```java
employee()
```

↓

Gets

```java
new Employee()
```

↓

Registers it as a Spring Bean.

---

# Visual Flow

```text
Application Starts

↓

@Configuration

↓

@Bean Method

↓

employee()

↓

new Employee()

↓

Bean Registered
```

---

# Complete Example

## Employee.java

```java
public class Employee {

    public Employee() {

        System.out.println("Employee Created");

    }

}
```

Notice

No

```java
@Component
```

---

## AppConfig.java

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public Employee employee() {

        return new Employee();

    }

}
```

---

## Main.java

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

Employee employee =
        context.getBean(Employee.class);
```

Output

```text
Employee Created
```

Spring manages `Employee` even though it doesn't have `@Component`.

---

# What is @Configuration?

`@Configuration` tells Spring

> "This class contains bean definitions."

Think of it as a configuration file written in Java instead of XML.

Earlier, in XML, we wrote:

```xml
<beans>

    <bean id="employee"
          class="com.demo.Employee"/>

</beans>
```

Now, using Java:

```java
@Configuration
public class AppConfig {

    @Bean
    public Employee employee() {

        return new Employee();

    }

}
```

Same result.

---

# XML vs Java Configuration

### XML

```xml
<bean id="employee"
      class="com.demo.Employee"/>
```

---

### Java Configuration

```java
@Bean
public Employee employee() {

    return new Employee();

}
```

Java configuration is easier to maintain, refactor, and provides compile-time checking.

---

# Real Project Example

Suppose you want to call another microservice.

You need

```java
RestTemplate restTemplate =
        new RestTemplate();
```

Instead of creating it everywhere

```java
new RestTemplate();
```

you create one bean.

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {

        return new RestTemplate();

    }

}
```

Now

```java
@Service
public class EmployeeService {

    @Autowired
    private RestTemplate restTemplate;

}
```

Spring injects the same singleton bean wherever it's needed.

---

# Another Example: Jackson ObjectMapper

Instead of

```java
ObjectMapper mapper =
        new ObjectMapper();
```

in multiple places,

create one bean.

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {

        return new ObjectMapper();

    }

}
```

Now every class can inject

```java
@Autowired
private ObjectMapper objectMapper;
```

---

# @Component vs @Bean

This is one of the most common interview questions.

| @Component                              | @Bean                                                        |
| --------------------------------------- | ------------------------------------------------------------ |
| Applied on a class                      | Applied on a method                                          |
| Used for your own classes               | Often used for third-party classes or custom object creation |
| Discovered using component scanning     | Explicitly declared in a `@Configuration` class              |
| Spring creates the object automatically | You create and return the object                             |

---

# Which One Should We Use?

Suppose

```java
public class EmployeeService {
}
```

This is your class.

Use

```java
@Service
```

or

```java
@Component
```

---

Suppose

```java
ObjectMapper
```

belongs to the Jackson library.

You cannot modify it.

Use

```java
@Bean
```

---

# Bean Name

```java
@Bean
public Employee employee() {

    return new Employee();

}
```

Default bean name

```text
employee
```

because the method name is

```text
employee()
```

---

Custom name

```java
@Bean("emp")
public Employee employee() {

    return new Employee();

}
```

Bean name becomes

```text
emp
```

---

# Can a @Bean Method Have Dependencies?

Yes.

Example

```java
@Configuration
public class AppConfig {

    @Bean
    public EmployeeService employeeService(EmployeeRepository repository) {

        return new EmployeeService(repository);

    }

}
```

Spring automatically injects `EmployeeRepository` into the `@Bean` method, just like constructor injection.

---

# Interview Questions

### 1. What is `@Bean`?

A method-level annotation that tells Spring to execute the method and register its return value as a bean in the IoC container.

---

### 2. What is `@Configuration`?

A class-level annotation that indicates the class contains one or more `@Bean` methods used to configure the Spring container.

---

### 3. Difference between `@Component` and `@Bean`?

**Answer:**

* `@Component` is used for classes that you own and can annotate directly.
* `@Bean` is used when you want to register objects manually, especially for third-party classes or when bean creation requires custom logic.

---

### 4. Where have you used `@Bean` in your project?

A good interview answer:

> "I have used `@Bean` to configure reusable objects such as `RestTemplate`, `ObjectMapper`, and other shared components. Since these are third-party classes, they cannot be annotated with `@Component`, so we register them using `@Bean` in a `@Configuration` class."

---

# Summary

```text
@Component
        │
        ▼
Annotate Your Class
        │
        ▼
Spring Creates Bean

----------------------------------

@Bean
        │
        ▼
Write Configuration Method
        │
        ▼
Return Object
        │
        ▼
Spring Registers Bean
```

---

# Common Interview Scenario

Suppose the interviewer asks:

> **Why don't we simply use `new ObjectMapper()` everywhere?**

A strong answer is:

* It creates unnecessary objects.
* Configuration becomes duplicated.
* A single shared bean is easier to configure and maintain.
* Spring can inject the same configured instance wherever it's needed.

---

## Next Topic

The next topic is **`@Value` and `application.properties`**, where you'll learn:

* What is `application.properties`?
* How to read properties using `@Value`
* Default values
* Injecting values into fields and constructors
* Real project examples like database URLs, API keys, Redis configuration, and server ports

This is another topic you'll use in almost every Spring Boot application.
