# Spring Boot — Chapter 12.5: `@PostConstruct` ⭐⭐⭐⭐⭐

`@PostConstruct` is one of the most important bean lifecycle annotations in Spring.

Many developers use it without fully understanding **when it executes**, **why it exists**, and **how it differs from constructors and `CommandLineRunner`**.

---

# Chapter Roadmap

```text id="40u7da"
@PostConstruct
│
├── 1. What is @PostConstruct?
├── 2. Why do we need it?
├── 3. Bean Creation Lifecycle
├── 4. Constructor vs @PostConstruct
├── 5. Dependency Injection Timing
├── 6. Complete Execution Flow
├── 7. Enterprise Examples
├── 8. Common Mistakes
├── 9. Best Practices
└── 10. Interview Questions
```

---

# 1. What is `@PostConstruct`?

`@PostConstruct` marks a method that Spring calls **once** after:

* The bean is created
* All dependencies are injected

but **before** the bean is used.

Example:

```java
@Service
public class EmployeeService {

    @PostConstruct
    public void init() {
        System.out.println("EmployeeService initialized");
    }
}
```

Spring automatically calls:

```java
init();
```

You **never call it yourself**.

---

# 2. Why Do We Need It?

Imagine this service:

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

Suppose we want to:

* Validate configuration
* Load cache
* Read reference data
* Initialize expensive objects

We need a place **after** dependency injection.

That's exactly what `@PostConstruct` provides.

---

# 3. Bean Lifecycle

Understanding the lifecycle is the key.

```text
Spring Starts
      │
      ▼
Find Bean
      │
      ▼
Call Constructor
      │
      ▼
Inject Dependencies
      │
      ▼
@PostConstruct   ← HERE
      │
      ▼
Bean Ready
      │
      ▼
Application Uses Bean
```

Notice:

`@PostConstruct` runs **after injection**, not before.

---

# 4. Example

```java
@Repository
public class EmployeeRepository {

}
```

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

    @PostConstruct
    public void init() {

        System.out.println(repository);
    }
}
```

Output:

```text
com.example.EmployeeRepository@6d311334
```

The repository already exists.

---

# 5. What Happens Internally?

Suppose Spring creates:

```java
@Service
class EmployeeService
```

Internally:

```text
Allocate Memory

↓

Call Constructor

↓

Inject Repository

↓

Inject Other Dependencies

↓

Call @PostConstruct

↓

Bean Ready
```

This order is guaranteed.

---

# 6. Constructor vs `@PostConstruct`

Many beginners confuse these.

Constructor:

```java
public EmployeeService() {

}
```

Runs immediately when the object is created.

At this moment:

```text
Dependencies

↓

Not Fully Injected Yet
```

---

`@PostConstruct`

```java
@PostConstruct
public void init() {

}
```

Runs after:

```text
Constructor

↓

Dependency Injection

↓

@PostConstruct
```

---

# 7. Why Not Do Everything in the Constructor?

Suppose:

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {

        this.repository = repository;

        repository.findAll();
    }
}
```

Although constructor injection ensures `repository` is available, constructors should ideally be used only to establish object state, not perform heavy initialization or business logic.

Better:

```java
@PostConstruct
public void init() {

    repository.findAll();
}
```

This keeps construction and initialization separate.

---

# 8. Complete Execution Example

```java
@Service
public class EmployeeService {

    public EmployeeService() {

        System.out.println("Constructor");
    }

    @PostConstruct
    public void init() {

        System.out.println("Post Construct");
    }
}
```

Output:

```text
Constructor

Post Construct
```

Execution:

```text
Constructor

↓

Dependency Injection

↓

@PostConstruct
```

---

# 9. Enterprise Example – Cache Loading

Suppose departments rarely change.

Instead of:

```text
User Login

↓

Query Database

↓

Load Departments
```

Load once:

```java
@Service
public class DepartmentCache {

    @PostConstruct
    public void loadCache() {

        System.out.println("Loading departments...");
    }
}
```

Startup:

```text
Application Starts

↓

@PostConstruct

↓

Load Cache

↓

Ready
```

---

# 10. Enterprise Example – Configuration Validation

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @PostConstruct
    public void validate() {

        if (secret == null || secret.isBlank()) {
            throw new IllegalStateException(
                "JWT secret is missing");
        }
    }
}
```

Application fails immediately if configuration is invalid.

---

# 11. Enterprise Example – Precomputing Data

```java
@Service
public class CurrencyService {

    @PostConstruct
    public void loadSupportedCurrencies() {

        // Read static data
    }
}
```

Instead of reading on every request.

---

# 12. Common Mistakes

### Mistake 1 – Long Running Work

```java
@PostConstruct
public void init() throws Exception {

    Thread.sleep(300000);
}
```

Application startup is delayed by 5 minutes.

Bad.

---

### Mistake 2 – Network Calls

```java
@PostConstruct
public void init() {

    paymentGateway.connect();
}
```

If the external service is unavailable, startup may fail or hang.

Use such operations carefully.

---

### Mistake 3 – Heavy Database Processing

```java
@PostConstruct
public void init() {

    repository.findAll();
}
```

Fine for small reference tables.

Not fine for millions of rows.

---

# 13. `@PostConstruct` vs `CommandLineRunner`

| `@PostConstruct`                    | `CommandLineRunner`                             |
| ----------------------------------- | ----------------------------------------------- |
| Bean lifecycle                      | Application lifecycle                           |
| Runs for each bean                  | Runs once per runner                            |
| Executes after dependency injection | Executes after the application context is ready |
| Bean initialization                 | Application initialization                      |

Flow:

```text
Bean Created

↓

@PostConstruct
```

vs

```text
Application Started

↓

CommandLineRunner
```

---

# 14. `@PostConstruct` vs Constructor

| Constructor              | `@PostConstruct`                                    |
| ------------------------ | --------------------------------------------------- |
| Object creation          | Bean initialization                                 |
| Runs first               | Runs after dependency injection                     |
| Establishes object state | Performs initialization using injected dependencies |
| Should avoid heavy logic | Suitable for lightweight initialization             |

---

# 15. Internal Spring Flow

```text
Create Bean

↓

Call Constructor

↓

Inject Dependencies

↓

BeanPostProcessor

↓

@PostConstruct

↓

Bean Available
```

`@PostConstruct` is invoked as part of Spring's bean initialization process.

---

# 16. Best Practices

```text
✅ Use for lightweight initialization

✅ Validate required configuration

✅ Initialize small caches

✅ Prepare immutable lookup data

✅ Keep methods fast

❌ Don't execute long-running tasks

❌ Don't load huge datasets

❌ Don't perform request-specific business logic

❌ Don't block application startup unnecessarily
```

---

# 17. Interview Questions

### What is `@PostConstruct`?

> It is a lifecycle annotation that marks a method to be executed once after dependency injection is complete and before the bean is used.

---

### When is it executed?

> After the constructor and dependency injection, during bean initialization.

---

### Can it have parameters?

No.

```java
@PostConstruct
public void init() {

}
```

The method must take no arguments.

---

### How many times is it called?

Once per bean instance.

---

### Can there be multiple `@PostConstruct` methods in one bean?

No. A bean should have only one `@PostConstruct` method.

---

### Should heavy business logic go inside `@PostConstruct`?

No.

Use it only for lightweight initialization.

---

# 18. Complete Lifecycle Diagram

```text
Spring Starts

↓

Create Bean

↓

Constructor

↓

Dependency Injection

↓

@PostConstruct

↓

Bean Ready

↓

Application Running

↓

@PreDestroy

↓

Bean Destroyed
```

---

# 19. Real Enterprise Flow

```text
EmployeeService Bean

↓

Constructor

↓

Inject Repository

↓

Inject JwtService

↓

Inject Cache

↓

@PostConstruct

↓

Validate Configuration

↓

Load Small Reference Data

↓

Service Ready

↓

Controller Uses Service
```

---

# 📍 Where We Are

```text
Spring Boot Advanced Features
│
├── ✅ 12.1 Application Lifecycle
├── ✅ 12.2 Application Events
├── ✅ 12.3 CommandLineRunner
├── ✅ 12.4 ApplicationRunner
├── ✅ 12.5 @PostConstruct ⭐⭐⭐⭐⭐
│
└── ⏭️ 12.6 @PreDestroy ⭐⭐⭐⭐⭐
       ↓
       Bean Destruction
       ↓
       Shutdown Process
       ↓
       Resource Cleanup
       ↓
       Database Connections
       ↓
       Thread Pools
       ↓
       Enterprise Shutdown Flow
```

In the next chapter, we'll explore **`@PreDestroy`**, the counterpart to `@PostConstruct`, and see exactly what happens when Spring Boot shuts down, how beans are destroyed, and how to safely clean up resources.
