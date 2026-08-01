# Spring Boot — Chapter 12: Spring Boot Advanced Features ⭐⭐⭐⭐⭐

This chapter covers the features that distinguish **good Spring Boot developers** from **experienced Spring Boot developers**.

These are the features commonly used in enterprise applications but often overlooked in beginner tutorials.

---

# Chapter Roadmap

```text
Spring Boot Advanced Features
│
├── 1. Application Lifecycle
├── 2. Application Events
├── 3. CommandLineRunner
├── 4. ApplicationRunner
├── 5. Startup Hooks
├── 6. Shutdown Hooks
├── 7. Lazy Initialization
├── 8. DevTools
├── 9. Banner Customization
├── 10. Graceful Shutdown
├── 11. Build Information
├── 12. Spring Boot Admin
├── 13. AOT (Ahead-of-Time)
├── 14. Native Images
├── 15. Best Practices
└── 16. Interview Questions
```

---

# 1. Spring Boot Application Lifecycle

Before learning advanced features, understand **when** they execute.

```text
Application Starts
        │
        ▼
main()
        │
        ▼
SpringApplication.run()
        │
        ▼
Create Environment
        │
        ▼
Read Configuration
        │
        ▼
Create ApplicationContext
        │
        ▼
Register Beans
        │
        ▼
Dependency Injection
        │
        ▼
@PostConstruct
        │
        ▼
ApplicationStartedEvent
        │
        ▼
CommandLineRunner
ApplicationRunner
        │
        ▼
ApplicationReadyEvent
        │
        ▼
Application Running
        │
        ▼
Shutdown Signal
        │
        ▼
@PreDestroy
        │
        ▼
Context Closed
```

This lifecycle is fundamental. Every advanced feature fits somewhere in it.

---

# 2. Application Events

Spring Boot publishes events during the application lifecycle.

Examples:

```text
ApplicationStartingEvent

↓

ApplicationEnvironmentPreparedEvent

↓

ApplicationContextInitializedEvent

↓

ApplicationPreparedEvent

↓

ApplicationStartedEvent

↓

ApplicationReadyEvent

↓

ApplicationFailedEvent
```

These events let you execute logic at specific stages of startup.

---

# 3. Listening to Events

Example:

```java
@Component
public class StartupListener {

    @EventListener
    public void handleReady(
            ApplicationReadyEvent event) {

        System.out.println("Application is ready.");
    }
}
```

Flow:

```text
Application Ready

↓

ApplicationReadyEvent

↓

@EventListener

↓

Your Code
```

---

# 4. Common Events

## ApplicationStartedEvent

Occurs after the application context has been refreshed, but **before** runners execute.

```text
Application Context Ready

↓

ApplicationStartedEvent
```

---

## ApplicationReadyEvent

Occurs after:

* Context initialized
* Runners finished

Now the application is ready to serve requests.

```text
Everything Initialized

↓

ApplicationReadyEvent

↓

Ready for Users
```

---

## ApplicationFailedEvent

Occurs if startup fails.

Useful for:

* Logging
* Alerts
* Cleanup

---

# 5. CommandLineRunner

One of the most commonly used startup hooks.

Example:

```java
@Component
public class DataLoader
        implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Loading initial data...");
    }
}
```

Flow:

```text
Spring Boot Startup

↓

CommandLineRunner

↓

Your Initialization Logic
```

Common use cases:

* Seed initial data
* Verify configuration
* Warm caches
* Run migrations (if not using Flyway/Liquibase)

---

# 6. Multiple CommandLineRunners

You can have several.

```java
@Component
@Order(1)
class FirstRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("First");
    }
}
```

```java
@Component
@Order(2)
class SecondRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Second");
    }
}
```

Execution:

```text
FirstRunner

↓

SecondRunner
```

---

# 7. ApplicationRunner

Very similar.

```java
@Component
public class StartupRunner
        implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {

        System.out.println("Started");
    }
}
```

Difference:

```text
CommandLineRunner

↓

String[] args

-------------------------

ApplicationRunner

↓

ApplicationArguments
```

---

# 8. Why ApplicationRunner?

Suppose:

```bash
java -jar app.jar --mode=dev --cache=true
```

ApplicationRunner:

```java
args.containsOption("mode");

args.getOptionValues("mode");
```

Much easier than parsing `String[]`.

---

# 9. `@PostConstruct`

Executed immediately after dependency injection.

```java
@Service
public class EmployeeService {

    @PostConstruct
    public void init() {

        System.out.println("Initialized");
    }
}
```

Flow:

```text
Bean Created

↓

Dependencies Injected

↓

@PostConstruct

↓

Bean Ready
```

Use it for bean-specific initialization.

---

# 10. `@PreDestroy`

Runs before bean destruction.

```java
@Service
public class CacheService {

    @PreDestroy
    public void cleanup() {

        System.out.println("Closing cache");
    }
}
```

Useful for:

* Closing files
* Releasing resources
* Flushing caches
* Disconnecting external systems

---

# 11. Lazy Initialization

Normally:

```text
Application Starts

↓

Every Singleton Bean Created
```

With lazy initialization:

```java
@Lazy
@Service
public class ReportService {

}
```

Flow:

```text
Application Starts

↓

Bean NOT Created

↓

First Request

↓

Bean Created
```

Advantages:

* Faster startup
* Lower initial memory usage

Disadvantages:

* First request may be slower
* Errors may appear later

---

# 12. Global Lazy Initialization

Enable globally:

```properties
spring.main.lazy-initialization=true
```

Now almost all beans are created only when needed.

Usually **not recommended** for most production applications because startup-time failures may be delayed until runtime.

---

# 13. DevTools

Development-only dependency.

```xml
<dependency>
    <groupId>
        org.springframework.boot
    </groupId>

    <artifactId>
        spring-boot-devtools
    </artifactId>

    <scope>
        runtime
    </scope>
</dependency>
```

Features:

* Automatic restart
* Live reload
* Development property defaults

---

# 14. How DevTools Works

```text
Code Change

↓

Restart Triggered

↓

Application Restarts

↓

Browser Refresh
```

Much faster than manually stopping and starting the application.

---

# 15. Banner Customization

Default:

```text
:: Spring Boot ::
```

Custom:

Create:

```text
src/main/resources/banner.txt
```

Example:

```text
==================================
 EMPLOYEE MANAGEMENT SYSTEM
==================================
```

On startup:

```text
EMPLOYEE MANAGEMENT SYSTEM
Spring Boot 3.x
```

---

# 16. Disable Banner

```properties
spring.main.banner-mode=off
```

Or:

```java
SpringApplication app =
        new SpringApplication(App.class);

app.setBannerMode(Banner.Mode.OFF);

app.run(args);
```

---

# 17. Graceful Shutdown

Without graceful shutdown:

```text
Request Processing

↓

Server Stops Immediately

↓

Request Lost
```

With graceful shutdown:

```text
Shutdown Signal

↓

Stop Accepting New Requests

↓

Finish Existing Requests

↓

Shutdown
```

Enable:

```properties
server.shutdown=graceful
```

Very important for Kubernetes and cloud deployments.

---

# 18. Shutdown Timeout

Configure:

```properties
spring.lifecycle.timeout-per-shutdown-phase=30s
```

Meaning:

```text
Shutdown

↓

Wait 30 Seconds

↓

Force Stop
```

---

# 19. Build Information

Generate build metadata.

Example:

```text
Version

Build Time

Artifact

Group

Git Commit
```

Available via:

```text
/actuator/info
```

Useful for verifying deployed versions.

---

# 20. Spring Boot Admin

Spring Boot Admin is a separate application that monitors multiple Spring Boot services.

Architecture:

```text
Employee API

Inventory API

Order API

Payment API

        │

        ▼

Spring Boot Admin

        │

        ▼

Web Dashboard
```

Features:

* Health
* Metrics
* Logs
* JVM info
* Notifications

Very useful in microservice environments.

---

# 21. Ahead-of-Time (AOT)

Traditional Spring:

```text
Application Starts

↓

Reflection

↓

Classpath Scanning

↓

Runtime Processing
```

AOT moves much of this work to build time.

Benefits:

* Faster startup
* Lower memory
* Better native image support

---

# 22. Native Images

Spring Boot supports building native executables using GraalVM.

Traditional:

```text
Java

↓

JVM

↓

Application
```

Native:

```text
Java

↓

Native Binary

↓

Application
```

Advantages:

* Very fast startup
* Lower memory footprint
* Excellent for serverless and containers

Trade-offs:

* Longer build time
* More complex build process
* Some reflection-based libraries require extra configuration

---

# 23. Startup Performance Tips

```text
✅ Remove unused starters

✅ Use constructor injection

✅ Avoid expensive @PostConstruct work

✅ Enable lazy initialization only when appropriate

✅ Use AOT for faster startup

✅ Keep auto-configuration lean

✅ Profile startup with ApplicationStartup when investigating slow startup
```

---

# 24. Enterprise Startup Flow

```text
Application Starts

↓

Read Configuration

↓

Create Beans

↓

Database Connected

↓

Redis Connected

↓

Kafka Connected

↓

Security Initialized

↓

CommandLineRunner

↓

Cache Warm-up

↓

ApplicationReadyEvent

↓

Load Balancer Marks Service Healthy

↓

Production Traffic Begins
```

---

# 25. Interview Questions

### What is `CommandLineRunner`?

> A callback interface whose `run()` method is executed after the Spring application context has been initialized.

---

### Difference between `CommandLineRunner` and `ApplicationRunner`?

| CommandLineRunner         | ApplicationRunner               |
| ------------------------- | ------------------------------- |
| Receives `String... args` | Receives `ApplicationArguments` |
| Simpler                   | Easier option parsing           |

---

### What is `@PostConstruct`?

> A lifecycle callback executed after dependency injection and before the bean is used.

---

### What is `@PreDestroy`?

> A lifecycle callback executed before a bean is destroyed, typically used for cleanup.

---

### What is lazy initialization?

> Delaying bean creation until the bean is first requested instead of creating it during application startup.

---

### What is graceful shutdown?

> A shutdown process where the server stops accepting new requests but allows in-flight requests to complete before terminating.

---

### What is AOT?

> Ahead-of-Time processing that shifts some runtime work to build time, improving startup performance and enabling native image generation.

---

### What is Spring Boot Admin?

> A monitoring dashboard for managing and observing multiple Spring Boot applications.

---

# 26. Best Practices

```text
✅ Use CommandLineRunner only for lightweight startup tasks

✅ Use ApplicationReadyEvent when the application must be fully ready

✅ Keep @PostConstruct fast

✅ Always clean resources in @PreDestroy

✅ Enable graceful shutdown in production

✅ Use DevTools only during development

✅ Use Spring Boot Admin for many services

✅ Consider AOT/native images for fast-starting workloads

❌ Don't perform heavy business logic during startup

❌ Don't block startup unnecessarily
```

---

# 📍 Spring Boot Course Status

```text
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server
├── ✅ Chapter 7 — Externalized Configuration
├── ✅ Chapter 8 — Profiles
├── ✅ Chapter 9 — @ConfigurationProperties
├── ✅ Chapter 10 — Actuator
├── ✅ Chapter 11 — Testing
├── ✅ Chapter 12 — Advanced Features
│
└── 🎉 Spring Boot Complete
```

## 🎓 What You've Completed

You now have a complete learning path covering:

* ✅ Spring Core
* ✅ Spring MVC
* ✅ REST API Best Practices
* ✅ Spring Security
* ✅ Spring Boot

The next logical step is not another theory chapter—it's building a **production-grade Employee Management System** from scratch, where you'll apply all of these concepts together (JWT authentication, Spring Security, JPA, Validation, Exception Handling, Testing, Actuator, Docker, PostgreSQL, and deployment), just like a real enterprise application.
