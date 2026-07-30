# Spring Boot — Chapter 7: Externalized Configuration

Now we enter one of the **most important practical Spring Boot topics**:

```text
application.properties
application.yaml
Environment Variables
System Properties
Command-Line Arguments
@Value
Environment
@ConfigurationProperties
```

Spring Boot is designed so that the **same application code can run with different configuration in different environments**. It supports properties/YAML, environment variables, system properties, command-line arguments, and other property sources, with a defined precedence order. ([Home][1])

---

# 1. Why do we need Externalized Configuration?

Imagine this:

```java
String dbUrl =
    "jdbc:postgresql://localhost:5432/employee";
```

Works locally.

But production uses:

```text
postgres-prod.company.com
```

Do we change Java code?

**No.**

Instead:

```text
Java Code
    ↓
Configuration
    ↓
DEV / TEST / PROD
```

So:

```text
Same JAR
   +
Different configuration
   =
Different environment
```

That's the whole purpose.

---

# 2. What should be externalized?

Typical examples:

```text
Database URL
Database username
Database password
Server port
JWT configuration
External API URL
Feature flags
Timeouts
Logging levels
Application name
```

Avoid hardcoding environment-specific values directly into Java code.

---

# 3. `application.properties`

The simplest configuration file:

```properties id="asq1c8"
spring.application.name=employee-api

server.port=8080

app.employee.max-page-size=100

app.external.payment-url=https://payment.example.com
```

Then your application can read these values.

Spring Boot automatically loads `application.properties` and `application.yaml` from its supported config locations. ([Home][1])

---

# 4. YAML

Instead of:

```properties id="f2v8m1"
app.employee.max-page-size=100
app.external.payment-url=https://payment.example.com
```

you can use:

```yaml id="u5c9n2"
app:
  employee:
    max-page-size: 100

  external:
    payment-url: https://payment.example.com
```

YAML is especially useful when configuration becomes hierarchical.

Spring Boot converts YAML structures into properties in the `Environment`. ([Home][1])

---

# 5. Properties vs YAML

### Properties

```properties id="z1m7q4"
server.port=8080
spring.datasource.url=jdbc:h2:mem:test
```

### YAML

```yaml id="d4q8x2"
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:test
```

Both ultimately contribute properties to Spring's `Environment`.

A practical rule:

> Pick one format for the application and stay consistent. Spring Boot recommends doing so; when both formats exist at the same location, `.properties` takes precedence. ([Home][1])

---

# 6. What is `Environment`?

This is the core abstraction behind configuration.

Conceptually:

```text id="j8p3w6"
Properties
YAML
Environment Variables
System Properties
Command Line
        ↓
      Environment
        ↓
      Your Beans
```

You can access a property directly:

```java id="x3k7m2"
@Autowired
Environment environment;
```

and:

```java id="c9n5q4"
String value =
    environment.getProperty(
        "server.port"
    );
```

Spring Boot documents the `Environment` abstraction as one way to access externalized configuration. ([Home][1])

---

# 7. `@Value`

For a small number of properties:

```java id="q7m2x8"
@Component
public class AppInfo {

    @Value("${spring.application.name}")
    private String applicationName;

    @Value("${server.port}")
    private int port;
}
```

Then:

```text id="v4c8n1"
application.properties
        ↓
Environment
        ↓
@Value
        ↓
Java field
```

Spring Boot supports property injection through `@Value`. ([Home][1])

---

# 8. Default Value with `@Value`

You can do:

```java id="m5q9r2"
@Value("${app.timeout:30}")
private int timeout;
```

Meaning:

```text id="g3x8p6"
app.timeout exists?
      │
   YES → use it
   NO  → use 30
```

The standard placeholder syntax supports defaults with `:`. ([Home][1])

---

# 9. What if the Property Doesn't Exist?

Without a default:

```java id="n7p4c1"
@Value("${app.timeout}")
private int timeout;
```

If the property is missing, application startup can fail because Spring can't resolve the placeholder.

So for required configuration:

```text id="j2m8q6"
Missing
 ↓
Fail fast
```

For optional configuration:

```java id="k5x9v3"
@Value("${app.timeout:30}")
```

you can provide a default.

---

# 10. Environment Variables

Suppose we have:

```properties id="r6q2m8"
app.jwt.secret=abc123
```

Instead of putting the secret in the configuration file, production can supply it as an environment variable.

Spring Boot supports binding environment variables to configuration properties using its relaxed binding rules. For example, `app.jwt.secret` can map to an uppercase underscore-based environment variable. ([Home][1])

Conceptually:

```text id="b4m9q1"
APP_JWT_SECRET
      ↓
Spring Environment
      ↓
app.jwt.secret
```

---

# 11. Environment Variable Naming

A configuration key:

```text id="y8r3n5"
app.jwt.secret
```

becomes approximately:

```text id="j1m6x4"
APP_JWT_SECRET
```

For configuration keys, Spring Boot converts names to uppercase and uses underscores as delimiters when binding environment variables. ([Home][1])

For example:

```text id="o7q2p8"
server.port
        ↓
SERVER_PORT
```

---

# 12. Why Environment Variables Are Useful

Especially for:

```text id="s9c4m7"
Docker
Kubernetes
Cloud deployments
CI/CD
Production secrets
```

You can keep the application JAR unchanged:

```text id="m3x8q2"
employee-api.jar
```

and provide different environment configuration:

```text id="g4r7p1"
DEV:
DB_URL=localhost

PROD:
DB_URL=prod-db.internal
```

Same artifact, different environment.

---

# 13. Command-Line Arguments

You can override configuration when starting the application:

```bash id="c6m1x8"
java -jar employee-api.jar \
    --server.port=9090
```

Spring Boot converts command-line options beginning with `--` into properties and adds them to the `Environment`. By default, command-line properties have higher precedence than file-based configuration. ([Home][1])

---

# 14. Example of Override

Suppose:

```properties id="w2p6n9"
server.port=8080
```

Start:

```bash id="n5r3c7"
java -jar app.jar --server.port=9090
```

Result:

```text id="k8m4x1"
8080
   ↓
Command line says 9090
   ↓
9090 wins
```

This is one of the most important ideas behind configuration precedence.

---

# 15. Configuration Precedence

Spring Boot has a defined `PropertySource` order.

The exact full list is long, but the important sources for everyday development are:

```text id="p7v3m8"
application.properties / YAML
        ↓
Environment variables
        ↓
System properties
        ↓
Command-line arguments
```

Higher-precedence sources override lower-precedence ones. The current Boot documentation defines the complete ordered list, including test-specific sources and other mechanisms. ([Home][1])

For interviews, remember:

> **Later/higher-priority property sources override earlier ones.**

---

# 16. Practical Override Example

Suppose:

```properties id="s4n8c2"
app.message=Hello
```

Environment:

```text id="g2m7q5"
APP_MESSAGE=Production
```

Command line:

```bash id="r6p1x9"
java -jar app.jar --app.message=CLI
```

Result:

```text id="x8q3m6"
application.properties
    Hello

environment
    Production

command line
    CLI

Final value
    CLI
```

---

# 17. Where Does an External `application.properties` Go?

Spring Boot supports configuration files inside and outside the packaged application.

A common deployment model is:

```text id="u7m2c8"
app.jar
config/
   application.properties
```

Then you can change deployment configuration without rebuilding the JAR.

Spring Boot's config-data loading supports packaged and external locations, with external configuration able to override packaged configuration according to the defined precedence. ([Home][1])

---

# 18. Packaged vs External Configuration

Conceptually:

```text id="e8q4n2"
Inside JAR
application.properties
       ↓
Default configuration

Outside JAR
application.properties
       ↓
Deployment override
```

This is very useful for:

```text id="m7r3x9"
Docker
VM deployment
On-prem servers
Cloud environments
```

---

# 19. `application-{profile}.properties`

We'll cover profiles in the next chapter, but you should know this naming convention:

```text id="p2q8m5"
application.properties

application-dev.properties

application-test.properties

application-prod.properties
```

For example:

```properties id="c7n3x1"
# application-dev.properties

server.port=8081
```

and:

```properties id="h4m9q6"
# application-prod.properties

server.port=8080
```

When the appropriate profile is active, the profile-specific file can override the common configuration. ([Home][1])

---

# 20. Property Placeholder

You can refer to another property:

```properties id="q4x8m2"
app.name=Employee API

app.description=${app.name} Backend
```

Then:

```text id="r7p3n6"
app.description
      ↓
Employee API Backend
```

Spring Boot supports `${...}` property placeholders and optional defaults such as `${name:default}`. ([Home][1])

---

# 21. Why Canonical Kebab Case Matters

Prefer:

```properties id="n6m2q7"
app.employee.max-page-size=100
```

rather than:

```properties id="z8r4c1"
app.employee.maxPageSize=100
```

Spring Boot recommends canonical kebab-case property names, especially for placeholders and configuration metadata. ([Home][1])

This matters because relaxed binding allows several representations, particularly for environment variables.

---

# 22. `@Value` vs `@ConfigurationProperties`

This is the next major decision.

For a single property:

```java id="m3q7x1"
@Value("${app.timeout}")
private int timeout;
```

Fine.

But suppose you have:

```text id="j5c9p4"
app.payment.url
app.payment.timeout
app.payment.retry-count
app.payment.enabled
```

Using many `@Value` fields becomes messy.

That's where:

```text id="x8n2m6"
@ConfigurationProperties
```

comes in.

Spring Boot explicitly recommends `@ConfigurationProperties` for grouping your own related configuration into structured, type-safe objects. ([Home][1])

---

# 23. `@ConfigurationProperties` Example

Create:

```java id="v4m8q2"
@ConfigurationProperties(
    prefix = "app.payment"
)
public record PaymentProperties(
        String url,
        Duration timeout,
        int retryCount,
        boolean enabled
) {
}
```

Configuration:

```yaml id="q7c3m9"
app:
  payment:
    url: https://payment.example.com
    timeout: 5s
    retry-count: 3
    enabled: true
```

Now:

```text id="d5p9x4"
application.yaml
       ↓
@ConfigurationProperties
       ↓
PaymentProperties
```

Much cleaner.

---

# 24. Why `@ConfigurationProperties` Is Better for Groups

`@Value`:

```java id="f2n7m3"
@Value("${app.payment.url}")
private String url;

@Value("${app.payment.timeout}")
private Duration timeout;

@Value("${app.payment.retry-count}")
private int retryCount;
```

Becomes:

```java id="g6q1r8"
@ConfigurationProperties(
    prefix = "app.payment"
)
public record PaymentProperties(
    String url,
    Duration timeout,
    int retryCount
) {}
```

Benefits include:

```text id="p9m4x2"
Type-safe binding
Structured configuration
Metadata support
Relaxed binding
Cleaner code
```

Spring Boot documents these advantages in its `@ConfigurationProperties` guidance. ([Home][1])

---

# 25. `@Value` vs `@ConfigurationProperties`

Remember this:

| `@Value`                | `@ConfigurationProperties`     |
| ----------------------- | ------------------------------ |
| Small number of values  | Group of related properties    |
| Simple injection        | Structured configuration       |
| Supports SpEL           | Does not use SpEL for binding  |
| Limited relaxed binding | Strong relaxed binding         |
| No metadata support     | Configuration metadata support |

Spring's current Boot documentation explicitly describes these differences. ([Home][1])

---

# 26. Environment Example

You might have:

```yaml id="md4slq"
app:
  jwt:
    issuer: employee-api
    expiration: 15m
```

Then:

```java id="7p2n8m"
@ConfigurationProperties(
    prefix = "app.jwt"
)
public record JwtProperties(
        String issuer,
        Duration expiration
) {
}
```

Environment variable overrides could be supplied in the corresponding relaxed form, and the same object can be used consistently across your application. ([Home][1])

---

# 27. What is `Environment` Used For?

Sometimes you need dynamic lookup:

```java id="h8q3v1"
@Component
public class AppConfigReader {

    private final Environment environment;

    public AppConfigReader(
            Environment environment) {

        this.environment = environment;
    }

    public String getPaymentUrl() {

        return environment.getProperty(
            "app.payment.url"
        );
    }
}
```

This is useful when:

```text id="m4r7x2"
Dynamic property access
```

is actually required.

But don't use `Environment.getProperty()` everywhere just because it works.

For stable application configuration:

```text id="z8p2n6"
@ConfigurationProperties
```

is usually cleaner.

---

# 28. JSON Configuration

Spring Boot also supports supplying configuration as JSON through:

```text id="c6m9q3"
SPRING_APPLICATION_JSON
```

Example:

```bash id="p7x2r8"
SPRING_APPLICATION_JSON='{"app":{"name":"Employee API"}}'
```

Boot parses this into the `Environment`. ([Home][1])

This is less common in day-to-day application code but useful to recognize.

---

# 29. External Configuration Files

You can specify configuration locations using:

```text id="y3m8q1"
spring.config.location
spring.config.additional-location
```

For example:

```bash id="f8n4c6"
java -jar app.jar \
  --spring.config.additional-location=optional:file:./config/
```

Spring Boot supports external config locations and profile-specific variants through these mechanisms. ([Home][1])

---

# 30. Configuration Architecture

This is the mental model:

```text id="r4x9m2"
                Configuration Sources
                        │
       ┌────────────────┼─────────────────┐
       ▼                ▼                 ▼
application.yaml   Environment      Command line
application.properties variables     arguments
       │                │                 │
       └────────────────┼─────────────────┘
                        ▼
                  Spring Environment
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
          @Value          @ConfigurationProperties
             │                     │
             └──────────┬──────────┘
                        ▼
                   Application
```

---

# 31. Important Security Rule

Never put production secrets into source-controlled files such as:

```properties
id="vytp2j"
app.jwt.secret=my-real-production-secret
spring.datasource.password=my-real-password
```

Instead:

```text id="b8m3q6"
Environment Variable
       OR
Secret Manager
       OR
Deployment Secret
```

Then:

```text id="n7p2c4"
Application
   ↓
Environment
   ↓
@ConfigurationProperties
```

This is especially important because we just implemented JWT security.

---

# 32. Interview Questions

### What is externalized configuration?

> It means keeping environment-specific configuration outside application code so the same application artifact can run with different settings across environments. Spring Boot supports files, environment variables, command-line arguments, system properties, and other property sources. ([Home][1])

### What is `application.properties`?

> A standard Spring Boot configuration file used to provide application properties.

### Properties vs YAML?

> Both provide configuration to the Spring `Environment`; YAML is particularly convenient for hierarchical configuration. ([Home][1])

### What is configuration precedence?

> When multiple property sources define the same key, the higher-precedence source wins. Command-line properties, for example, override file-based properties by default. ([Home][1])

### `@Value` vs `@ConfigurationProperties`?

> `@Value` is convenient for a small number of individual properties; `@ConfigurationProperties` is preferable for structured groups of related configuration because it provides type-safe binding, relaxed binding, and metadata support. ([Home][1])

### Why use environment variables?

> They allow deployment-specific configuration and secrets to be supplied without rebuilding or modifying the application artifact.

---

# 33. Best Practices

```text id="x8p4m2"
✅ Externalize environment-specific values
✅ Prefer @ConfigurationProperties for grouped configuration
✅ Use @Value for small/simple injections
✅ Keep secrets out of Git
✅ Use environment variables or secret managers
✅ Use canonical kebab-case property names
✅ Understand property precedence
✅ Keep one configuration format consistently where practical
```

---

# 📍 Where We Are

```text id="q5m8x2"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters & Dependency Management
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server & Startup
├── ✅ Chapter 7 — Externalized Configuration ⭐⭐⭐⭐⭐
│      ├── application.properties
│      ├── YAML
│      ├── Environment
│      ├── @Value
│      ├── Environment Variables
│      ├── Command-Line Arguments
│      ├── Property Precedence
│      └── @ConfigurationProperties
│
└── ⏭️ Chapter 8 — Profiles ⭐⭐⭐⭐⭐
       ↓
       dev / test / prod
       ↓
       application-{profile}.properties
       ↓
       spring.profiles.active
       ↓
       spring.profiles.default
       ↓
       Profile groups
       ↓
       Profile-specific beans
       ↓
       Environment-specific configuration
```

Next is **Spring Boot Profiles**, where we'll build a clean `dev → test → prod` configuration structure and understand exactly how profile activation and profile-specific properties work.

[1]: https://docs.spring.io/spring-boot/reference/features/external-config.html?utm_source=chatgpt.com "Externalized Configuration :: Spring Boot"
