# Spring Boot — Chapter 9: `@ConfigurationProperties` ⭐⭐⭐⭐⭐

This is one of the **most important Spring Boot topics**.

In real-world Spring Boot applications, you should **prefer `@ConfigurationProperties` over `@Value`** for application configuration.

We'll learn everything from scratch.

---

# 1. Why was `@ConfigurationProperties` introduced?

Imagine we have:

```yaml
app:
  employee:
    max-page-size: 100
    default-page-size: 20
    cache-enabled: true
    timeout: 30s
```

Using `@Value`:

```java
@Value("${app.employee.max-page-size}")
private int maxPageSize;

@Value("${app.employee.default-page-size}")
private int defaultPageSize;

@Value("${app.employee.cache-enabled}")
private boolean cacheEnabled;

@Value("${app.employee.timeout}")
private Duration timeout;
```

Problems:

* Too many `@Value` annotations
* Hard to maintain
* No grouping
* Difficult validation
* Not type-safe enough

Instead:

```java
@ConfigurationProperties(prefix = "app.employee")
public record EmployeeProperties(
        int maxPageSize,
        int defaultPageSize,
        boolean cacheEnabled,
        Duration timeout
) {}
```

Now everything is grouped into one object.

---

# 2. What is `@ConfigurationProperties`?

Definition:

> `@ConfigurationProperties` binds a group of external configuration properties to a strongly typed Java object.

Think:

```text
application.yaml
        │
        ▼
@ConfigurationProperties
        │
        ▼
Java Object
```

Instead of reading values individually, Spring binds an entire configuration section.

---

# 3. Example

### application.yaml

```yaml
app:
  employee:
    max-page-size: 100
    default-page-size: 20
    timeout: 30s
```

Java:

```java
@ConfigurationProperties(prefix = "app.employee")
public record EmployeeProperties(
        int maxPageSize,
        int defaultPageSize,
        Duration timeout
) {
}
```

Spring automatically binds:

```text
max-page-size
        ↓
maxPageSize

default-page-size
        ↓
defaultPageSize

timeout
        ↓
Duration
```

---

# 4. How does Binding work?

Suppose:

```yaml
app:
  employee:
    max-page-size: 100
```

Spring performs:

```text
application.yaml
        │
        ▼
Property Source
        │
        ▼
Binder
        │
        ▼
EmployeeProperties.maxPageSize = 100
```

This process is called **binding**.

---

# 5. What is the Binder?

Internally Spring Boot has a **Binder**.

Conceptually:

```text
Environment
      │
      ▼
Binder
      │
      ▼
ConfigurationProperties Object
```

The Binder:

* Reads properties
* Converts data types
* Creates objects
* Handles nested configuration
* Supports collections
* Performs relaxed binding

You usually never interact with it directly.

---

# 6. Registering Configuration Properties

Spring needs to know about the class.

There are three ways.

---

## Method 1 (Recommended)

```java
@ConfigurationPropertiesScan
@SpringBootApplication
public class EmployeeApplication {
}
```

Boot scans for every:

```java
@ConfigurationProperties
```

class.

---

## Method 2

```java
@EnableConfigurationProperties(
    EmployeeProperties.class
)
```

Useful when registering a few classes explicitly.

---

## Method 3

```java
@Component
@ConfigurationProperties(prefix="app.employee")
```

Works, but nowadays **`@ConfigurationPropertiesScan` is preferred** for application configuration classes.

---

# 7. Complete Example

### application.yaml

```yaml
app:
  employee:
    max-page-size: 100
    default-page-size: 20
```

Property class:

```java
@ConfigurationProperties(prefix = "app.employee")
public record EmployeeProperties(
        int maxPageSize,
        int defaultPageSize
) {
}
```

Application:

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class EmployeeApplication {
}
```

Now Spring creates:

```text
EmployeeProperties Bean
```

automatically.

---

# 8. Injecting the Properties

Anywhere:

```java
@Service
public class EmployeeService {

    private final EmployeeProperties properties;

    public EmployeeService(
            EmployeeProperties properties) {

        this.properties = properties;
    }
}
```

Just like any other Spring Bean.

---

# 9. Using the Values

```java
@Service
public class EmployeeService {

    private final EmployeeProperties properties;

    public EmployeeService(
            EmployeeProperties properties) {

        this.properties = properties;
    }

    public int getMaxPageSize() {

        return properties.maxPageSize();
    }
}
```

No `@Value` anywhere.

---

# 10. Relaxed Binding

One of Boot's best features.

Configuration:

```yaml
max-page-size
```

binds to:

```java
maxPageSize
```

Also works with:

```text
max_page_size
MAX_PAGE_SIZE
maxPageSize
```

All bind correctly.

Conceptually:

```text
Configuration
      │
      ▼
Relaxed Binding
      │
      ▼
Java Field
```

This is why Boot recommends kebab-case in configuration.

---

# 11. Nested Configuration

Suppose:

```yaml
app:

  employee:

    cache:
      enabled: true
      ttl: 10m
```

Java:

```java
@ConfigurationProperties(prefix="app.employee")
public record EmployeeProperties(

        Cache cache

) {

    public record Cache(
            boolean enabled,
            Duration ttl
    ) {}
}
```

Spring binds:

```text
cache.enabled
        ↓
cache.enabled()

cache.ttl
        ↓
cache.ttl()
```

Nested configuration becomes very clean.

---

# 12. Lists

YAML:

```yaml
app:

  employee:

    departments:
      - HR
      - Finance
      - IT
```

Java:

```java
@ConfigurationProperties(prefix="app.employee")
public record EmployeeProperties(

        List<String> departments

) {
}
```

Result:

```text
departments

↓

["HR","Finance","IT"]
```

---

# 13. Maps

Configuration:

```yaml
app:

  employee:

    permissions:

      admin: ALL

      user: READ
```

Java:

```java
@ConfigurationProperties(prefix="app.employee")
public record EmployeeProperties(

        Map<String,String> permissions

) {}
```

Result:

```text
admin
↓

ALL
```

---

# 14. Supported Types

Binder supports many Java types.

Examples:

```text
String
int
long
boolean
double
Duration
DataSize
List
Set
Map
Enum
LocalDate
LocalDateTime
URI
URL
InetAddress
```

You rarely need manual conversion.

---

# 15. Duration

Instead of:

```yaml
timeout: 30
```

Use:

```yaml
timeout: 30s
```

Java:

```java
Duration timeout
```

Supported examples:

```text
5s
30s
10m
1h
2d
```

Spring converts automatically.

---

# 16. DataSize

Instead of:

```yaml
max-upload: 1048576
```

Use:

```yaml
max-upload: 10MB
```

Java:

```java
DataSize maxUpload;
```

Spring converts:

```text
10MB
↓

DataSize
```

---

# 17. Enum Binding

Enum:

```java
public enum EnvironmentType {

    DEV,
    TEST,
    PROD
}
```

Configuration:

```yaml
environment: PROD
```

Property:

```java
EnvironmentType environment
```

Spring converts automatically.

---

# 18. Validation

One of the biggest advantages.

Suppose:

```yaml
app:

  employee:

    max-page-size: -5
```

Not valid.

Property class:

```java
@ConfigurationProperties(prefix="app.employee")
@Validated
public record EmployeeProperties(

        @Min(1)
        @Max(500)

        int maxPageSize

) {}
```

If:

```text
-5
```

Application startup fails immediately.

This is called **fail fast**.

---

# 19. Why Validation Matters

Without validation:

```text
Wrong value

↓

Application starts

↓

Runtime bugs
```

With validation:

```text
Wrong value

↓

Startup Failure

↓

Fix immediately
```

Much safer.

---

# 20. Constructor Binding

Old versions required:

```java
@ConstructorBinding
```

Modern Spring Boot:

For records and single constructors, constructor binding is automatic.

Example:

```java
public record EmployeeProperties(
        int pageSize
) {}
```

Nothing else needed.

---

# 21. Immutable Configuration

Records are perfect.

```java
public record JwtProperties(

        String secret,

        Duration expiration

) {}
```

Benefits:

```text
Immutable
Thread-safe
Cleaner
Less boilerplate
```

Today this is the recommended style.

---

# 22. Configuration Metadata

When using:

```java
@ConfigurationProperties
```

your IDE can provide:

* Auto-completion
* Documentation
* Property suggestions

This improves developer experience significantly.

---

# 23. `@Value` vs `@ConfigurationProperties`

| @Value               | @ConfigurationProperties           |
| -------------------- | ---------------------------------- |
| One property         | Many related properties            |
| Scattered            | Grouped                            |
| Limited validation   | Full Bean Validation               |
| String placeholder   | Strong type binding                |
| Good for small cases | Best for application configuration |

---

# 24. Real JWT Example

```yaml
app:

  jwt:

    issuer: employee-api

    secret: my-secret

    expiration: 15m

    refresh-expiration: 7d
```

Java:

```java
@ConfigurationProperties(prefix="app.jwt")
public record JwtProperties(

        String issuer,

        String secret,

        Duration expiration,

        Duration refreshExpiration

) {}
```

Anywhere:

```java
@Service
public class JwtService {

    private final JwtProperties properties;

    public JwtService(
            JwtProperties properties) {

        this.properties = properties;
    }
}
```

Very clean.

---

# 25. Employee API Example

```yaml
app:

  employee:

    pagination:

      default-page-size: 20

      max-page-size: 100

    cache:

      enabled: true

      ttl: 30m

    upload:

      max-size: 10MB
```

Java:

```java
@ConfigurationProperties(prefix="app.employee")
public record EmployeeProperties(

        Pagination pagination,

        Cache cache,

        Upload upload

) {

    public record Pagination(

            int defaultPageSize,

            int maxPageSize

    ) {}

    public record Cache(

            boolean enabled,

            Duration ttl

    ) {}

    public record Upload(

            DataSize maxSize

    ) {}
}
```

This is exactly how enterprise Spring Boot applications organize configuration.

---

# 26. Complete Binding Flow

```text
application.yaml
        │
        ▼
Property Sources
        │
        ▼
Environment
        │
        ▼
Binder
        │
        ▼
@ConfigurationProperties
        │
        ▼
EmployeeProperties Bean
        │
        ▼
Dependency Injection
        │
        ▼
EmployeeService
```

---

# 27. Common Mistakes

❌ Using `@Value` for 30 properties.

❌ Forgetting to register configuration properties.

❌ Hardcoding secrets.

❌ Not validating configuration.

❌ Using mutable configuration unnecessarily.

---

# 28. Interview Questions

### What is `@ConfigurationProperties`?

> It binds a group of external configuration properties to a strongly typed Java object.

---

### Why is it better than `@Value`?

* Type-safe
* Grouped configuration
* Supports validation
* Supports nested objects
* Easier maintenance

---

### What is relaxed binding?

> Spring Boot allows different naming styles in configuration (such as `max-page-size`) to bind to Java properties like `maxPageSize`.

---

### What is constructor binding?

> Binding configuration through the constructor. In modern Spring Boot, records and classes with a single constructor are bound automatically.

---

### Why use records?

* Immutable
* Less boilerplate
* Thread-safe
* Recommended for configuration

---

### What happens if validation fails?

Application startup fails with a configuration binding/validation exception, allowing you to detect invalid configuration immediately.

---

# 29. Best Practices

```text
✅ Prefer @ConfigurationProperties for application configuration

✅ Prefer records

✅ Group related configuration

✅ Validate configuration

✅ Use Duration instead of int milliseconds

✅ Use DataSize instead of raw bytes

✅ Keep secrets outside Git

✅ Use kebab-case in YAML

✅ Keep configuration immutable
```

---

# 📍 Where We Are

```text
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters & Dependency Management
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server & Startup
├── ✅ Chapter 7 — Externalized Configuration
├── ✅ Chapter 8 — Profiles
├── ✅ Chapter 9 — @ConfigurationProperties ⭐⭐⭐⭐⭐
│
└── ⏭️ Chapter 10 — Spring Boot Actuator ⭐⭐⭐⭐⭐
       ↓
       Health Endpoint
       ↓
       Info Endpoint
       ↓
       Metrics
       ↓
       Prometheus
       ↓
       Custom Health Indicators
       ↓
       Monitoring
       ↓
       Production Readiness
```

**Next Chapter (Actuator)** is where you'll learn how production teams monitor Spring Boot applications using health checks, metrics, readiness/liveness probes, Prometheus integration, and custom health indicators. This is heavily used in Kubernetes, Docker, cloud deployments, and enterprise production systems.
