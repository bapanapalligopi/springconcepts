# Spring Boot — Chapter 12.3: `CommandLineRunner` ⭐⭐⭐⭐⭐

`CommandLineRunner` is one of the most commonly used startup features in Spring Boot.

Many developers know **how to use it**, but few understand **when it runs, why it exists, and how it fits into the application lifecycle**.

---

# Chapter Roadmap

```text
CommandLineRunner
│
├── 1. What is CommandLineRunner?
├── 2. Why do we need it?
├── 3. Execution Flow
├── 4. Lifecycle Position
├── 5. Creating a Runner
├── 6. Multiple Runners
├── 7. @Order
├── 8. Common Enterprise Use Cases
├── 9. Command-line Arguments
├── 10. Best Practices
├── 11. CommandLineRunner vs ApplicationRunner
└── 12. Interview Questions
```

---

# 1. What is `CommandLineRunner`?

`CommandLineRunner` is a **callback interface**.

Spring Boot automatically calls its `run()` method **once**, immediately after the application has started.

Interface:

```java
@FunctionalInterface
public interface CommandLineRunner {

    void run(String... args) throws Exception;
}
```

The important point:

> **You never call `run()` yourself. Spring Boot calls it automatically.**

---

# 2. Why Do We Need It?

Imagine an Employee Management application.

When the application starts, you want to:

* Load default roles
* Create an admin user
* Validate configuration
* Warm the cache
* Preload lookup data

Without `CommandLineRunner`:

```text
Application Started

↓

Someone manually runs initialization
```

This is error-prone.

With `CommandLineRunner`:

```text
Application Started

↓

Spring automatically executes run()

↓

Initialization completes

↓

Application ready
```

---

# 3. Where Does It Execute?

Remember the lifecycle?

```text
main()

↓

SpringApplication.run()

↓

Environment

↓

ApplicationContext

↓

Beans Created

↓

@PostConstruct

↓

ApplicationStartedEvent

↓

CommandLineRunner  ← HERE

↓

ApplicationReadyEvent

↓

Application Running
```

Notice:

It runs **after all beans have been created and injected**, but **before** `ApplicationReadyEvent`.

---

# 4. Internal Flow

Suppose your application contains:

```java
@Component
class DataLoader implements CommandLineRunner
```

Internally:

```text
Spring Boot Starts

↓

ApplicationContext Created

↓

Find all CommandLineRunner Beans

↓

Sort by @Order

↓

Execute run()

↓

ApplicationReadyEvent
```

Spring automatically discovers every `CommandLineRunner` bean.

---

# 5. Creating a Simple Runner

Example:

```java
@Component
public class StartupRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Application started successfully.");
    }
}
```

Output:

```text
Application started successfully.
```

This executes once during startup.

---

# 6. Complete Enterprise Example

Suppose we need default roles.

```java
@Component
public class RoleInitializer
        implements CommandLineRunner {

    private final RoleRepository repository;

    public RoleInitializer(RoleRepository repository) {
        this.repository = repository;
    }

    @Override
    public void run(String... args) {

        if (repository.count() == 0) {

            repository.save(new Role("ADMIN"));
            repository.save(new Role("USER"));
        }
    }
}
```

Flow:

```text
Application Starts

↓

Database Connected

↓

RoleInitializer

↓

Insert Default Roles
```

Every startup ensures required roles exist.

---

# 7. Another Example – Cache Warm-up

Instead of loading data on the first request:

```text
User Login

↓

Load Cache

↓

Slow Response
```

Load during startup:

```java
@Component
public class CacheLoader
        implements CommandLineRunner {

    @Override
    public void run(String... args) {

        cacheService.loadDepartments();
        cacheService.loadCountries();
    }
}
```

Now:

```text
Application Starts

↓

Load Cache

↓

Ready

↓

Fast User Requests
```

---

# 8. Multiple `CommandLineRunner`s

Spring supports multiple runners.

Example:

```java
@Component
class FirstRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("First");
    }
}
```

```java
@Component
class SecondRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Second");
    }
}
```

Without ordering, execution order should not be relied upon.

---

# 9. Controlling Order with `@Order`

Example:

```java
@Component
@Order(1)
class DatabaseRunner
        implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Database initialized");
    }
}
```

```java
@Component
@Order(2)
class CacheRunner
        implements CommandLineRunner {

    @Override
    public void run(String... args) {

        System.out.println("Cache initialized");
    }
}
```

Execution:

```text
DatabaseRunner

↓

CacheRunner
```

Lower number = higher priority.

---

# 10. Passing Command-line Arguments

Run application:

```bash
java -jar employee-api.jar dev cache=true
```

Runner:

```java
@Component
public class StartupRunner
        implements CommandLineRunner {

    @Override
    public void run(String... args) {

        for (String arg : args) {
            System.out.println(arg);
        }
    }
}
```

Output:

```text
dev

cache=true
```

The arguments are available as `String... args`.

---

# 11. Typical Enterprise Use Cases

## Data Seeding

```text
Application Starts

↓

Insert Default Roles

↓

Insert Default Permissions
```

---

## Cache Initialization

```text
Application Starts

↓

Load Departments

↓

Load Countries

↓

Load Configuration
```

---

## Configuration Validation

```java
@Override
public void run(String... args) {

    if (jwtSecret == null) {

        throw new IllegalStateException(
                "JWT Secret Missing");
    }
}
```

Fail fast if required configuration is missing.

---

## External Connection Check

```text
Application Starts

↓

Connect Redis

↓

Connect Kafka

↓

Verify External APIs
```

Detect problems before serving users.

---

# 12. Common Mistakes

### Mistake 1

Heavy work:

```java
run() {

    Thread.sleep(300000);
}
```

Startup takes 5 minutes.

Bad.

---

### Mistake 2

Running business logic.

```text
Create Employees

↓

Process Orders
```

Business operations should happen in services/controllers, not startup runners.

---

### Mistake 3

Ignoring failures.

If initialization is critical:

```java
throw new IllegalStateException(...);
```

Fail startup rather than running with invalid state.

---

# 13. `CommandLineRunner` vs `@PostConstruct`

`@PostConstruct`

```text
Bean Created

↓

Dependencies Injected

↓

@PostConstruct
```

Runs **once for each bean**.

---

`CommandLineRunner`

```text
All Beans Ready

↓

Application Started

↓

run()
```

Runs after the entire application context is initialized.

| `@PostConstruct`    | `CommandLineRunner`       |
| ------------------- | ------------------------- |
| Bean lifecycle      | Application lifecycle     |
| Runs per bean       | Runs once per runner      |
| Bean initialization | Application startup tasks |

---

# 14. `CommandLineRunner` vs `ApplicationRunner`

| CommandLineRunner            | ApplicationRunner                 |
| ---------------------------- | --------------------------------- |
| `String... args`             | `ApplicationArguments`            |
| Simpler                      | Rich argument parsing             |
| Good for basic startup tasks | Better for option-based arguments |

Example:

```java
@Component
class Runner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {

        if (args.containsOption("debug")) {
            System.out.println("Debug mode");
        }
    }
}
```

If you need named options like `--debug` or `--profile=dev`, `ApplicationRunner` is more convenient.

---

# 15. Enterprise Startup Sequence

```text
Application Starts

↓

Environment Ready

↓

ApplicationContext Created

↓

Beans Created

↓

@PostConstruct

↓

ApplicationStartedEvent

↓

RoleInitializer

↓

CacheLoader

↓

ConfigurationValidator

↓

ApplicationReadyEvent

↓

Users Can Access APIs
```

---

# 16. Best Practices

```text
✅ Keep runners short

✅ Use them for startup initialization

✅ Use @Order when execution order matters

✅ Validate critical configuration

✅ Warm caches if needed

✅ Seed reference data only when necessary

❌ Don't perform long-running work

❌ Don't put normal business logic in runners

❌ Don't assume runner execution order without @Order
```

---

# 17. Interview Questions

### What is `CommandLineRunner`?

> A Spring Boot callback interface whose `run()` method is automatically executed once after the application context has been initialized.

---

### When does it execute?

> After all beans are created and dependency injection is complete, but before `ApplicationReadyEvent` is published.

---

### Can there be multiple `CommandLineRunner`s?

> Yes. Spring detects all `CommandLineRunner` beans and executes them. If ordering matters, use `@Order`.

---

### What is the purpose of `@Order`?

> It defines the execution order when multiple runners exist. Lower values execute first.

---

### Typical use cases?

* Initial data seeding
* Cache warm-up
* Configuration validation
* Startup initialization
* External dependency verification

---

### When should you avoid using `CommandLineRunner`?

> Avoid putting long-running tasks or normal request-processing business logic inside it. It should be reserved for startup-related initialization.

---

# 18. Complete Execution Flow

```text
main()

↓

SpringApplication.run()

↓

Environment

↓

ApplicationContext

↓

Bean Creation

↓

Dependency Injection

↓

@PostConstruct

↓

ApplicationStartedEvent

↓

Find CommandLineRunner Beans

↓

Sort by @Order

↓

Execute run()

↓

ApplicationReadyEvent

↓

Application Running
```

---

# 📍 Next Topic

## **Chapter 12.4 — `ApplicationRunner` ⭐⭐⭐⭐**

We'll cover:

* Why `ApplicationRunner` was introduced
* `ApplicationArguments`
* Parsing command-line options
* `ApplicationRunner` vs `CommandLineRunner`
* Enterprise examples
* Internal execution flow
* Best practices
* Interview questions
