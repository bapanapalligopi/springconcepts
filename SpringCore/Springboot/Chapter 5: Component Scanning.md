# Spring Boot — Chapter 5: Component Scanning

Now we go deeper into one of the things you've already been using without realizing it:

```java
@Service
@Repository
@RestController
@Component
```

The question is:

> **How does Spring find these classes and turn them into Spring Beans?**

The answer is **component scanning**.

Spring Framework's classpath-scanning mechanism detects candidate components based on stereotype annotations and registers corresponding bean definitions with the container. `@Repository`, `@Service`, and `@Controller` are specialized forms of `@Component`. ([Home][1])

---

# 1. What is Component Scanning?

Component scanning means:

> Spring scans specified packages, identifies classes that qualify as components, and registers them with the `ApplicationContext`.

Conceptually:

```text id="b7e3m1"
Package
   ↓
Component Scan
   ↓
Find candidate classes
   ↓
Register BeanDefinitions
   ↓
Create/manage Spring Beans
```

---

# 2. The Classes Spring Looks For

The common stereotypes are:

```text id="c8p2x6"
@Component
@Service
@Repository
@Controller
@Configuration
```

Spring's documentation describes `@Component` as the generic stereotype, with `@Repository`, `@Service`, and `@Controller` as specialized stereotypes. ([Home][1])

So:

```text id="y4m8q2"
@Component
    │
    ├── @Service
    ├── @Repository
    └── @Controller
```

Conceptually, they are all candidates for component scanning.

---

# 3. Example

Suppose we have:

```java id="s4k8r2"
@Service
public class EmployeeService {
}
```

and:

```java id="w7p3m1"
@Repository
public class EmployeeRepository {
}
```

and:

```java id="q9n4x6"
@RestController
public class EmployeeController {
}
```

Spring scans the relevant packages and discovers them.

Then:

```text id="m2v7c9"
EmployeeService
        ↓
Spring Bean

EmployeeRepository
        ↓
Spring Bean

EmployeeController
        ↓
Spring Bean
```

---

# 4. Who Starts Component Scanning?

In our Boot application:

```java id="1e6v9p"
@SpringBootApplication
```

includes:

```text id="v5q2m8"
@ComponentScan
```

So:

```text id="f8r3k1"
@SpringBootApplication
        ↓
@ComponentScan
        ↓
Component Discovery
```

The Spring Boot documentation explains that the package of the `@SpringBootApplication` class provides the default base package for component scanning. ([Home][2])

---

# 5. Why Does Package Location Matter?

Suppose:

```text id="a3k8q2"
com.example.employee
│
├── EmployeeApplication
│
├── controller
│   └── EmployeeController
│
├── service
│   └── EmployeeService
│
└── repository
    └── EmployeeRepository
```

Application class:

```java id="x7c3m1"
package com.example.employee;

@SpringBootApplication
public class EmployeeApplication {
}
```

Spring naturally scans under:

```text id="z6p2m4"
com.example.employee
```

So it finds:

```text id="l8q4n7"
com.example.employee.controller
com.example.employee.service
com.example.employee.repository
```

That's why keeping the application class in a root package is recommended. ([Home][2])

---

# 6. Bad Package Structure

Suppose:

```text id="r2x9m5"
com.example.app.main
    └── Application.java

com.example.service
    └── EmployeeService.java
```

The application class is below `com.example`, but the service is outside its default scan hierarchy.

Result:

```text id="t4k7q1"
EmployeeService
      ↓
Not discovered
      ↓
No Bean
```

Then you may see something like:

```text id="y6p8m3"
NoSuchBeanDefinitionException
```

or:

```text id="c1m4q9"
Parameter 0 of constructor ... required a bean
that could not be found
```

This is one of the most common causes of "Spring can't find my bean."

---

# 7. What Actually Happens During Scanning?

The simplified flow is:

```text id="j5n8c2"
Application Startup
      ↓
@ComponentScan
      ↓
Determine base packages
      ↓
Scan classpath
      ↓
Find candidate components
      ↓
Create BeanDefinitions
      ↓
Register BeanDefinitions
      ↓
ApplicationContext
      ↓
Bean creation
```

Spring's underlying scanning support includes `ClassPathScanningCandidateComponentProvider` and `ClassPathBeanDefinitionScanner`. ([Home][3])

You normally don't interact with these low-level classes, but knowing they exist helps explain the mechanism.

---

# 8. What is a BeanDefinition?

Before Spring creates an actual object, it can register metadata describing the bean.

Conceptually:

```text id="p3q7m9"
@Component EmployeeService
        ↓
BeanDefinition
        ↓
Spring knows:
    class = EmployeeService
    bean name = employeeService
    scope = singleton by default
    dependencies = ...
```

Then Spring creates the actual object during application context initialization as appropriate.

So:

```text id="w8m2c5"
Component
   ↓
BeanDefinition
   ↓
Bean
```

---

# 9. Component Scan vs Bean Creation

These are not exactly the same thing.

### Scanning

Finds the class:

```text id="m7x3p8"
"EmployeeService exists."
```

### Bean registration

Creates/registers metadata:

```text id="c4n9q1"
"employeeService bean definition exists."
```

### Bean creation

Spring eventually creates the object:

```text id="h6p2m7"
new EmployeeService(...)
```

while resolving its dependencies and applying relevant lifecycle/proxy infrastructure.

So:

```text id="k9v4x2"
Scan
 ↓
Register
 ↓
Instantiate
 ↓
Initialize
```

---

# 10. Why `@Service` Rather Than `@Component`?

Technically:

```java id="w5m1r8"
@Component
public class EmployeeService {
}
```

can work.

But:

```java id="q8x4p3"
@Service
public class EmployeeService {
}
```

communicates the role of the class more clearly.

Likewise:

```text id="f2n7m4"
@Repository
 → persistence/data access

@Service
 → business/service layer

@Controller / @RestController
 → presentation/web layer
```

Spring explicitly defines these as specialized stereotypes for these roles. ([Home][1])

---

# 11. Does `@RestController` Work With Component Scanning?

Yes.

`@RestController` is a specialized controller stereotype and therefore participates in component scanning.

Conceptually:

```text id="s9q2k7"
@RestController
      ↓
@Controller
      ↓
@Component
      ↓
Component Scan
      ↓
Spring Bean
```

So Spring can discover it automatically.

---

# 12. Does `@Configuration` Get Scanned?

Yes.

For example:

```java id="o4m6x9"
@Configuration
public class SecurityConfig {
}
```

`@Configuration` is also a component stereotype and can be discovered through component scanning. ([Home][1])

This is why our earlier classes like:

```text id="p7q3m1"
SecurityConfig
CorsConfig
JwtConfig
```

can be found automatically.

---

# 13. What if a Class Has No `@Component`?

Suppose:

```java id="j8c3n5"
public class EmployeeFormatter {
}
```

There is no stereotype.

Component scanning won't normally discover it as a component.

You can instead register it explicitly:

```java id="y4m9q2"
@Configuration
public class AppConfig {

    @Bean
    EmployeeFormatter employeeFormatter() {
        return new EmployeeFormatter();
    }
}
```

Now:

```text id="k3v7p1"
@Bean
   ↓
BeanDefinition
   ↓
Spring Bean
```

So you have two common mechanisms:

```text id="q6m2x8"
@Component scanning
        OR
@Bean configuration
```

---

# 14. Component Scanning vs `@Bean`

This is a common interview question.

### Component scanning

```java id="j9r3m6"
@Service
public class EmployeeService {
}
```

Spring discovers the class.

### `@Bean`

```java id="n7c2q5"
@Bean
EmployeeService employeeService() {
    return new EmployeeService();
}
```

You explicitly tell Spring what object to register.

So:

```text id="s5m8x4"
@Component
 ↓
automatic discovery

@Bean
 ↓
explicit registration
```

---

# 15. When Should You Use `@Bean`?

Typical use cases:

```text id="h2q7m9"
Third-party class
Custom configuration
Multiple implementations
Complex object creation
Conditional custom setup
```

For example, you can't normally put:

```java id="v9m4x1"
@Service
```

on a third-party library class you don't own.

Instead:

```java id="k6p3n8"
@Bean
SomeThirdPartyClient client() {
    return new SomeThirdPartyClient(...);
}
```

---

# 16. `@ComponentScan` Manually

You can explicitly configure component scanning:

```java id="z3m8q5"
@Configuration
@ComponentScan(
    basePackages = "com.example.employee"
)
public class AppConfig {
}
```

Spring Framework supports specifying packages, patterns, include filters, and exclude filters through `@ComponentScan`. ([Home][1])

In a typical Boot application, however:

```java id="f7q2m9"
@SpringBootApplication
```

already provides the usual component scan, so you usually don't need another `@ComponentScan`.

---

# 17. `scanBasePackages`

Boot provides:

```java id="x8n3c6"
@SpringBootApplication(
    scanBasePackages = {
        "com.example.employee",
        "com.example.shared"
    }
)
```

This tells the component scanner where to search.

You can also use:

```java id="t4m7q1"
scanBasePackageClasses = {
    EmployeeApplication.class,
    SharedMarker.class
}
```

The Spring Boot API specifically recommends `scanBasePackageClasses()` as a type-safe alternative to string-based package names. ([Home][4])

---

# 18. Why Is `scanBasePackageClasses` Better?

Instead of:

```java id="n9p5x2"
scanBasePackages = "com.company.shared"
```

you can define a marker class:

```java id="c3r8m6"
package com.company.shared;

public class SharedPackageMarker {
}
```

Then:

```java id="u7k2q4"
@SpringBootApplication(
    scanBasePackageClasses = {
        EmployeeApplication.class,
        SharedPackageMarker.class
    }
)
```

If the package is later renamed, the compiler helps you catch the change.

---

# 19. Component Scanning Filters

By default, component scanning detects the common stereotype annotations.

Spring allows you to customize the scan using:

```text id="p8m4q1"
includeFilters
excludeFilters
```

The official Spring Framework documentation describes filter-based customization of `@ComponentScan`. ([Home][1])

For your level, understand the idea:

```text id="g2v7n5"
Scan everything matching normal stereotypes

OR

Customize:
    include certain classes
    exclude certain classes
```

You don't need to memorize all filter types yet.

---

# 20. Why Would We Exclude a Component?

Imagine:

```text id="h9x2m6"
Two implementations
```

and you only want one registered automatically.

Or:

```text id="q5c8r3"
A legacy component
```

should not be picked up.

Then exclude filters can control what component scanning detects.

In practice, cleaner package structure is usually preferable to elaborate scanning filters.

---

# 21. Common Error: Bean Not Found

Suppose:

```java id="s7m3x8"
@RestController
public class EmployeeController {

    private final EmployeeService service;

    public EmployeeController(
            EmployeeService service) {
        this.service = service;
    }
}
```

but:

```java id="n2q6v4"
EmployeeService
```

has no:

```java id="k5r9p1"
@Service
```

or:

```java id="c8m2x7"
@Bean
```

Then Spring doesn't know how to create it.

Result:

```text id="e1v7q9"
No qualifying bean of type EmployeeService
```

---

# 22. Another Common Error: Wrong Package

Suppose:

```text id="b6m3p8"
com.company.app
    └── Application

com.company.service
    └── EmployeeService
```

If the scan only covers:

```text id="x7q2n9"
com.company.app
```

then:

```text id="m5c8v1"
EmployeeService
```

is outside the scan.

So Spring doesn't find it.

Fix the package structure or explicitly configure the scan.

---

# 23. Another Common Error: Multiple Beans

This is different.

Suppose:

```java id="r8m2q5"
@Service
class EmployeeServiceImplA
```

and:

```java id="q3v7c1"
@Service
class EmployeeServiceImplB
```

both implement:

```java id="n6p4x9"
EmployeeService
```

Now Spring finds **two candidates**.

Controller:

```java id="c7m1q8"
public EmployeeController(
        EmployeeService service) {
}
```

Spring asks:

```text id="w4x8p2"
Which EmployeeService?
      ↓
A or B?
```

Result can be:

```text id="j9m3q6"
NoUniqueBeanDefinitionException
```

Component scanning worked perfectly.

The problem is **ambiguity**, not scanning.

---

# 24. How Do We Fix Multiple Beans?

One option:

```java id="f5q2m8"
@Primary
@Service
public class EmployeeServiceImplA
        implements EmployeeService {
}
```

Then Spring prefers that bean for unqualified injection.

Or:

```java id="z8n4p1"
@Qualifier("employeeServiceImplB")
```

to explicitly choose one.

This connects directly to the Spring Core DI topics you already learned.

---

# 25. Component Scanning + Dependency Injection

Now put everything together:

```text id="d3m7x9"
@ComponentScan
      ↓
Find EmployeeService
      ↓
Create/Register Bean
      ↓
Find EmployeeController
      ↓
Create/Register Bean
      ↓
Controller needs EmployeeService
      ↓
Dependency Injection
      ↓
EmployeeService injected
```

So:

```text id="q5v8n2"
Scanning
  +
IoC
  +
Dependency Injection
```

are closely connected.

---

# 26. Component Scanning vs Auto-Configuration

This should now be crystal clear.

```text id="k8m3x7"
@ComponentScan
        ↓
Find my application's components

@EnableAutoConfiguration
        ↓
Apply Boot's conditional infrastructure
```

Example:

```text id="y4q1p9"
@Service EmployeeService
       ↓
Component Scan

DataSource auto-configuration
       ↓
Auto-Configuration
```

---

# 27. Is Every Bean Created Through Component Scanning?

**No.**

Beans can come from:

```text id="s2m8q4"
@Component
@Service
@Repository
@Controller

@Bean

@Import

Auto-Configuration
```

There are other mechanisms too.

So don't say in an interview:

> "Spring creates beans only by scanning."

Instead say:

> "Component scanning is one major mechanism for registering beans; Spring can also register beans through `@Bean`, imports, auto-configuration, and other configuration mechanisms."

---

# 28. A Useful Startup Diagram

```text id="m9x3c7"
@SpringBootApplication
        │
        ▼
@ComponentScan
        │
        ▼
Find:
 ├── @Component
 ├── @Service
 ├── @Repository
 ├── @Controller
 └── @Configuration
        │
        ▼
BeanDefinitions
        │
        ▼
ApplicationContext
        │
        ▼
Dependency Injection
        │
        ▼
Ready Beans
```

At the same time:

```text id="f4p7n2"
@EnableAutoConfiguration
        │
        ▼
Conditional Auto-Configurations
        │
        ▼
Infrastructure Beans
```

Eventually they meet inside:

```text id="g8m1q5"
ApplicationContext
```

---

# 29. Interview Questions

### What is component scanning?

> It is Spring's mechanism for scanning configured packages, finding candidate components such as `@Component`, `@Service`, `@Repository`, `@Controller`, and `@Configuration`, and registering their bean definitions with the application context. ([Home][1])

### Why place the main class in the root package?

> Because `@SpringBootApplication` implicitly defines a default scan base package from the application's package, so placing it in the root makes subpackages discoverable. ([Home][2])

### `@Component` vs `@Service`?

> `@Service` is a specialized stereotype of `@Component`, intended to communicate that the class belongs to the service layer. ([Home][1])

### Component scanning vs `@Bean`?

> Component scanning discovers annotated classes automatically; `@Bean` explicitly registers an object returned by a configuration method.

### Why isn't my `@Service` found?

Check:

```text id="h5q2m7"
Package hierarchy
@ComponentScan base package
@Service annotation
Main application package
Multiple application contexts
Profile/conditional configuration
```

### What causes `NoUniqueBeanDefinitionException`?

> Multiple beans satisfy the same injection point and Spring cannot determine which one to use.

---

# 30. Best Practices

```text id="p7n3x8"
✅ Put @SpringBootApplication in the root package
✅ Keep package structure clean
✅ Use @Service/@Repository/@Controller appropriately
✅ Use @Bean for explicit/custom/third-party objects
✅ Prefer type-safe scanBasePackageClasses when custom scanning is needed
✅ Avoid unnecessary @ComponentScan declarations
✅ Use @Qualifier/@Primary when multiple implementations exist
❌ Don't put application classes in the default package
❌ Don't randomly expand component scanning to the whole classpath
```

Spring Boot specifically discourages the Java default package because component scanning and related scanning features can end up reading every class from every JAR. ([Home][2])

---

# 📍 Where We Are

```text id="k2q8m4"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
│
├── ✅ Chapter 2 — @SpringBootApplication
│
├── ✅ Chapter 3 — Starters & Dependency Management
│
├── ✅ Chapter 4 — Auto-Configuration
│
├── ✅ Chapter 5 — Component Scanning ⭐⭐⭐⭐⭐
│      ├── @Component
│      ├── @Service
│      ├── @Repository
│      ├── @Controller
│      ├── @Configuration
│      ├── @ComponentScan
│      ├── scanBasePackages
│      ├── scanBasePackageClasses
│      └── Bean discovery
│
└── ⏭️ Chapter 6 — Embedded Server & Application Startup
       ↓
       SpringApplication.run()
       ↓
       ApplicationContext
       ↓
       Embedded Tomcat
       ↓
       WebApplicationContext
       ↓
       Server startup
       ↓
       Port configuration
       ↓
       Context path
       ↓
       JAR vs WAR
```

Next we'll trace **exactly what happens when you run `SpringApplication.run()`**, including how the `ApplicationContext` is created, how Boot knows it's a web application, how the embedded server starts, and why you can simply run:

```bash
java -jar employee-api.jar
```

without installing Tomcat separately.

[1]: https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html?utm_source=chatgpt.com "Classpath Scanning and Managed Components :: Spring Framework"
[2]: https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html?utm_source=chatgpt.com "Structuring Your Code :: Spring Boot"
[3]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ClassPathBeanDefinitionScanner.html?utm_source=chatgpt.com "ClassPathBeanDefinitionScanner (Spring Framework 7.0.8 API)"
[4]: https://docs.spring.io/spring-boot/api/java/org/springframework/boot/autoconfigure/SpringBootApplication.html?utm_source=chatgpt.com "SpringBootApplication (Spring Boot 4.1.0 API)"
