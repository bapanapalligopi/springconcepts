# Spring Boot — Chapter 1: Why Spring Boot?

We now start **Spring Boot from fresh**.

As before:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

Spring Boot's goal is to make Spring application setup and deployment much easier through conventions, auto-configuration, starters, embedded servers, and externalized configuration.

---

# 1. Why was Spring Boot created?

You already know Spring Core.

Before Spring Boot, a traditional Spring application could involve a lot of configuration.

For example:

```text id="q2n1s4"
Application
   ↓
Spring configuration
   ↓
DataSource configuration
   ↓
Transaction configuration
   ↓
MVC configuration
   ↓
JSON configuration
   ↓
Server/deployment configuration
```

There were several things developers repeatedly had to configure.

Spring Boot came to reduce this setup and provide a more opinionated starting point.

---

# 2. What problem does Spring Boot solve?

Imagine you want a simple REST API.

Without Boot, you may need to think separately about:

```text id="3z08d7"
Spring dependencies
Spring MVC
JSON conversion
Servlet configuration
Web server
Database configuration
Logging
Environment configuration
Packaging
```

With Spring Boot, much of the common setup is automatically configured based on:

```text id="0em5va"
Dependencies
+
Application configuration
+
Spring Boot conventions
```

Spring Boot's auto-configuration is designed to configure your application based on the dependencies present on the classpath and other configuration conditions.

---

# 3. What is Spring Boot?

A good interview definition:

> **Spring Boot is a framework built on top of the Spring Framework that simplifies application development by providing conventions, auto-configuration, starter dependencies, embedded servers, and production-oriented features.**

Important:

```text id="l5b1s3"
Spring Boot
    ↓
Does NOT replace Spring
```

Instead:

```text id="4p1f6z"
Spring Boot
    ↓
Simplifies using Spring
```

---

# 4. Spring Framework vs Spring Boot

This is one of the most common interview questions.

## Spring Framework

Provides:

```text id="0s8z6a"
IoC
Dependency Injection
AOP
MVC
Transactions
JDBC
Security integration
etc.
```

## Spring Boot

Provides:

```text id="16q7h5"
Auto-configuration
Starters
Embedded server support
Externalized configuration
Actuator
Boot testing support
Production conventions
```

So:

```text id="k7f4x1"
Spring Framework
       +
Spring Boot conveniences
       ↓
Easy Spring application development
```

---

# 5. Why couldn't we just use Spring Framework?

We could.

Spring Boot is **not mandatory**.

You can still build traditional Spring applications.

But Boot removes a lot of repetitive setup.

Think:

```text id="x3s2a9"
Traditional Spring
     ↓
"You configure a lot."

Spring Boot
     ↓
"Spring configures common things for you."
```

---

# 6. The Biggest Idea: Convention over Configuration

This is an important Boot concept.

Instead of asking you to configure every tiny detail, Spring Boot has sensible defaults.

For example, if Boot detects:

```text id="fy2g7p"
Spring MVC on classpath
```

it can configure common MVC infrastructure.

If it detects:

```text id="v8n4k2"
DataSource + JDBC
```

it can configure JDBC infrastructure.

If it detects:

```text id="j6r1m9"
Web application
```

it can configure an embedded servlet environment.

The exact configuration depends on the classpath and properties. Spring Boot's auto-configuration documentation explains this conditional behavior.

---

# 7. What is Auto-Configuration?

This is probably the **most important Spring Boot topic**.

Suppose your Maven dependencies contain:

```xml id="i7w3ad"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

Spring Boot sees the relevant web dependencies.

It can automatically configure a large portion of the web application infrastructure.

Conceptually:

```text id="o0uwab"
Classpath
   ↓
What libraries are present?
   ↓
Spring Boot Conditions
   ↓
Auto-Configuration
   ↓
Beans
```

Spring Boot's auto-configuration is conditional and backs away where you provide your own configuration for the relevant area.

---

# 8. Example of "Backs Away"

Suppose Boot sees a database driver and auto-configures a `DataSource`.

Then you explicitly define:

```java id="z2bq5m"
@Bean
DataSource myDataSource() {
    ...
}
```

Boot can generally back away from its automatic configuration for that component because you provided your own bean/configuration.

This is an important Boot philosophy:

> **Auto-configure when appropriate, but let the developer override it.**

---

# 9. What are Starters?

Instead of manually selecting many dependencies:

```text id="9d7t2m"
Spring MVC
Jackson
Tomcat
Validation
Logging
...
```

Spring Boot provides **starter dependencies**.

For example:

```xml id="6q2f8w"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

A starter is essentially a convenient dependency descriptor that brings together a set of dependencies commonly needed for a particular feature.

---

# 10. Common Starters

You'll see things such as:

```text id="m8v2g6"
spring-boot-starter-webmvc
spring-boot-starter-jdbc
spring-boot-starter-security
spring-boot-starter-validation
spring-boot-starter-test
```

and database-specific/data starters in the broader Boot ecosystem.

Think:

```text id="k0x6f1"
Starter
   ↓
"Give me the dependencies usually needed for this feature."
```

---

# 11. Embedded Server

Traditional applications were often deployed into an externally managed application server.

Spring Boot commonly lets you run a web application as a self-contained application with an embedded servlet server.

Conceptually:

```text id="x2k9p4"
Your JAR
 ├── Application
 ├── Dependencies
 └── Embedded Server
```

Then:

```bash id="w6m4a2"
java -jar application.jar
```

and the application starts.

This is one of the major practical conveniences of Spring Boot.

---

# 12. `@SpringBootApplication`

You'll soon see:

```java id="e1j7n5"
@SpringBootApplication
public class EmployeeApplication {

    public static void main(String[] args) {

        SpringApplication.run(
                EmployeeApplication.class,
                args
        );
    }
}
```

This annotation is extremely important.

At a high level, it combines several Spring features, most notably:

```text id="o4k9v2"
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

The exact annotation composition is worth knowing for interviews.

We'll study it in Chapter 2.

---

# 13. `SpringApplication.run()`

This line:

```java id="v3r7m8"
SpringApplication.run(
    EmployeeApplication.class,
    args
);
```

starts the Spring Boot application.

Conceptually:

```text id="n5q2c8"
main()
  ↓
SpringApplication.run()
  ↓
Create ApplicationContext
  ↓
Apply Boot configuration
  ↓
Component scanning
  ↓
Auto-configuration
  ↓
Create beans
  ↓
Start server if web app
  ↓
Application ready
```

This is the startup pipeline we'll study later.

---

# 14. Where does Spring Core still fit?

Everything you learned earlier is still happening.

For example:

```text id="7p8w2n"
Spring Core
   ↓
IoC Container
   ↓
Dependency Injection
```

Boot doesn't remove that.

It simplifies configuration around it.

So:

```text id="e3b6z1"
Spring Boot
   ↓
uses Spring Framework
   ↓
uses IoC / DI / AOP / MVC / Transactions / etc.
```

---

# 15. Spring Boot Doesn't Replace Spring MVC

You learned Spring MVC:

```text id="ym4t7q"
DispatcherServlet
HandlerMapping
Controller
MessageConverter
```

With Boot:

```text id="u5p2r9"
Spring Boot
   ↓
Auto-configures common Spring MVC infrastructure
```

You still use:

```java id="f7m3k1"
@RestController
@GetMapping
@PostMapping
@RequestBody
```

The difference is that Boot removes much of the manual setup.

---

# 16. Spring Boot Doesn't Replace Spring Security

Same idea.

You learned:

```text id="r4p1n7"
SecurityFilterChain
UserDetailsService
JwtDecoder
Authorization
```

Boot helps bring these pieces together through dependency management and auto-configuration.

You still use Spring Security.

---

# 17. A Simple Architecture

```text id="q2m8x5"
                Spring Boot
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
  Spring Core    Spring MVC    Spring Security
       │             │             │
       ▼             ▼             ▼
    IoC / DI    REST APIs      Authentication
                                Authorization
```

And underneath:

```text id="s7v3p9"
Spring Boot
   ↓
Spring Framework
   ↓
Your Application
```

---

# 18. What Makes Boot "Opinionated"?

Spring Boot gives sensible defaults.

For example:

```text id="c8r2x6"
"Use common setup unless
you tell me otherwise."
```

This is what people mean by **opinionated defaults**.

But Boot is not a closed box.

You can override many defaults.

So:

```text id="w9q4n1"
Boot
├── Defaults ✅
└── Customization ✅
```

---

# 19. Production Features

Boot also provides features aimed at production applications.

One important example is:

```text id="m6x2c8"
Spring Boot Actuator
```

It provides endpoints and infrastructure for monitoring and management.

We'll cover:

```text id="v5p8j3"
/actuator/health
/actuator/info
/actuator/metrics
```

later.

---

# 20. Externalized Configuration

Instead of hardcoding:

```java id="z3q9v6"
String databaseUrl =
    "jdbc:postgresql://...";
```

Boot supports configuration outside your Java code through:

```text id="p7k2m4"
application.properties
application.yaml
Environment variables
Command-line arguments
```

Spring Boot's external configuration system lets the same application code run in different environments with different configuration values.

This becomes very important when we discuss:

```text id="x8m1q5"
dev
test
prod
```

---

# 21. Why is Boot so popular?

Because developers can move quickly from:

```text id="o7n3q2"
Project creation
```

to:

```text id="h4m8x1"
Running REST API
```

without manually configuring every infrastructure component.

The practical workflow becomes:

```text id="j6q2w9"
Choose dependencies
      ↓
Create Boot application
      ↓
Write business code
      ↓
Configure only what differs
      ↓
Run
```

---

# 22. A Real Example

Imagine creating this REST API:

```java id="9p5m2c"
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public List<Employee> getEmployees() {
        return service.findAll();
    }
}
```

You don't manually configure:

```text id="d7r3k9"
DispatcherServlet bean
JSON converter
Embedded server
ApplicationContext startup
```

Boot handles much of the standard infrastructure automatically.

That's the practical value.

---

# 23. What Spring Boot Does NOT Magically Do

This is a good interview point.

Boot does **not** mean:

```text id="j8m4v2"
"Write no configuration ever."
```

You still need to configure things such as:

```text id="f2q7n6"
Database credentials
Security policies
Business-specific beans
External service URLs
Profiles
Application-specific settings
```

Boot simply eliminates **unnecessary repetitive configuration**.

---

# 24. Interview Questions

### What is Spring Boot?

> Spring Boot is built on Spring Framework and simplifies application development through conventions, auto-configuration, starters, embedded servers, externalized configuration, and production-oriented features.

### Spring vs Spring Boot?

> Spring provides the core framework capabilities; Spring Boot simplifies how those capabilities are configured and deployed.

### What is auto-configuration?

> Spring Boot automatically configures common application infrastructure based on the classpath, existing beans, and configuration conditions.

### What is a starter?

> A starter is a convenient dependency descriptor that groups dependencies commonly needed for a feature.

### Does Spring Boot replace Spring?

> No. Spring Boot builds on the Spring Framework.

### What is an embedded server?

> It allows the application to package and start a servlet server as part of the application rather than requiring a separately deployed server in the traditional model.

### Why is Spring Boot called opinionated?

> Because it provides sensible defaults and conventions so common applications require less explicit configuration.

---

# 25. Best Practices

For your level:

```text id="c5m8q2"
✅ Understand what Boot auto-configures
✅ Don't blindly add configuration for everything
✅ Understand starters
✅ Keep application configuration externalized
✅ Override Boot defaults deliberately
✅ Don't treat auto-configuration as magic
✅ Know how to debug auto-configuration
```

The strongest Boot developers know **both**:

```text id="m2v7p5"
"What Boot gives me automatically"
```

and:

```text id="x8n3q1"
"How to override it when my application needs something different."
```

---

# 26. Mental Model

Remember:

```text id="w4k9p2"
Spring Framework
│
├── IoC / DI
├── AOP
├── MVC
├── Transactions
├── Security
└── Other Spring capabilities

        +
        
Spring Boot
│
├── Auto-Configuration
├── Starters
├── Embedded Server
├── External Configuration
├── Actuator
└── Boot conventions

        ↓

Production-ready Spring application
```

---

# 📍 Where We Are

```text id="b8m2x6"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
│      ↓
│   Spring vs Spring Boot
│   Auto-configuration concept
│   Starters
│   Embedded server
│   Opinionated defaults
│   External configuration concept
│
└── ⏭️ Chapter 2 — @SpringBootApplication ⭐⭐⭐⭐⭐
       ↓
       @Configuration
       ↓
       @EnableAutoConfiguration
       ↓
       @ComponentScan
       ↓
       SpringApplication.run()
       ↓
       Complete startup flow
```

Next we'll **open `@SpringBootApplication` and understand exactly what those three annotations do**, then trace what happens from `main()` until your beans and embedded server are running.
