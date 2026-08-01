# Spring Boot — Chapter 8: Profiles ⭐⭐⭐⭐⭐

Now we solve a very common real-world problem:

> **How do I use the same Spring Boot application in DEV, TEST, and PROD without changing Java code?**

Spring Boot Profiles let you segregate configuration and activate only the configuration appropriate for the current environment. You can activate profiles with `spring.profiles.active`; if none is active, Boot uses the `default` profile unless you change that behavior. ([Home][1])

---

# 1. Why do we need Profiles?

Imagine your application has:

### DEV

```text id="w2j7m1"
Database:
localhost

Logging:
DEBUG

External API:
localhost:9000
```

### TEST

```text id="q9m4x8"
Database:
test database

Logging:
INFO

External API:
test service
```

### PROD

```text id="b5c8n2"
Database:
production database

Logging:
WARN

External API:
production service
```

We don't want:

```java id="k7p3v9"
if (environment.equals("prod")) {
    ...
}
```

all over the application.

Instead:

```text id="m4x8q2"
Same Java Code
      +
Different Profile
      ↓
Different Configuration
```

---

# 2. What is a Spring Profile?

A profile is a named environment/configuration group.

Examples:

```text id="c3n7p1"
dev
test
prod
```

Conceptually:

```text id="f8q2m5"
Profile = DEV
    ↓
Development configuration

Profile = PROD
    ↓
Production configuration
```

---

# 3. Profile-Specific Files

This is the most common structure:

```text id="r6m3x9"
application.properties

application-dev.properties

application-test.properties

application-prod.properties
```

Spring Boot uses the naming convention:

```text id="t7p4c2"
application-{profile}.properties
```

and the YAML equivalent:

```text id="j5x8m1"
application-{profile}.yaml
```

Profile-specific configuration overrides non-profile-specific configuration. ([Home][2])

---

# 4. Example

### `application.properties`

```properties id="n3q7k2"
spring.application.name=employee-api

server.port=8080

app.employee.max-page-size=100
```

This is common configuration.

Now:

### `application-dev.properties`

```properties id="a8m4v9"
server.port=8081

app.employee.max-page-size=20

logging.level.com.practice.employeeapi=DEBUG
```

### `application-prod.properties`

```properties id="p2x7c5"
server.port=8080

app.employee.max-page-size=100

logging.level.com.practice.employeeapi=INFO
```

---

# 5. Which Configuration Wins?

Suppose:

```properties id="h3k9m4"
# application.properties

server.port=8080
```

and:

```properties id="v7p2c8"
# application-dev.properties

server.port=8081
```

When `dev` is active:

```text id="q8m1x5"
application.properties
        ↓
8080

application-dev.properties
        ↓
8081
```

Final:

```text id="r4c7n2"
8081
```

Profile-specific properties override the non-profile-specific ones. ([Home][2])

---

# 6. How Do We Activate a Profile?

There are multiple ways.

The most common:

```properties id="s9x2m6"
spring.profiles.active=dev
```

Then Boot activates:

```text id="c5q8p1"
dev
```

and loads:

```text id="j6m3r9"
application-dev.properties
```

Boot allows `spring.profiles.active` to be supplied through the normal external-configuration mechanisms, including command-line arguments. ([Home][1])

---

# 7. Activate from Command Line

Instead of putting:

```properties id="m2v9c4"
spring.profiles.active=dev
```

inside the application file, you can run:

```bash id="y6n3q8"
java -jar employee-api.jar \
    --spring.profiles.active=dev
```

This is often better for deployments because the artifact stays unchanged.

---

# 8. Environment Variable

You can also use:

```bash id="p1x7m4"
export SPRING_PROFILES_ACTIVE=prod
```

Then start:

```bash id="h8q3n9"
java -jar employee-api.jar
```

Now:

```text id="w7m2c5"
SPRING_PROFILES_ACTIVE=prod
      ↓
prod profile active
      ↓
application-prod.properties
```

This fits very naturally with Docker and cloud deployments.

---

# 9. What Happens if No Profile Is Active?

Spring Boot uses a default profile named:

```text id="z5q8m3"
default
```

when no explicit profile is active. You can change the default profile or set it to another value. ([Home][1])

For example:

```properties id="m7c2x9"
spring.profiles.default=dev
```

Then:

```text id="f4q8n1"
No active profile
      ↓
Default profile = dev
      ↓
dev configuration
```

---

# 10. `spring.profiles.active` vs `spring.profiles.default`

### `spring.profiles.active`

> Which profiles should actually be active?

```properties id="k9m3p7"
spring.profiles.active=prod
```

### `spring.profiles.default`

> Which profile should be used when no profile is active?

```properties id="r2x8c4"
spring.profiles.default=dev
```

So:

```text id="j6q1m8"
active
  ↓
Explicit choice

default
  ↓
Fallback
```

---

# 11. Can Multiple Profiles Be Active?

Yes.

For example:

```properties id="n8p3v6"
spring.profiles.active=prod,metrics
```

Now both are active:

```text id="a5m7x2"
prod
metrics
```

Spring Boot supports multiple active profiles. ([Home][1])

---

# 12. What If Two Profiles Define the Same Property?

Suppose:

```text id="q4m8n1"
spring.profiles.active=prod,metrics
```

and both define:

```properties id="x7p2c5"
logging.level.root=...
```

Spring Boot uses a **last-wins** strategy among active profiles in the relevant profile-specific configuration. So `metrics` can override the value from `prod` if both contribute that property. ([Home][2])

Think:

```text id="m3q8v5"
prod
  ↓
value A

metrics
  ↓
value B

final
  ↓
B
```

---

# 13. Profile-Specific Beans

Profiles are not only for properties.

You can put:

```java id="8j1q4t"
@Profile("dev")
```

on a configuration or bean.

Example:

```java id="v6m3x8"
@Configuration
@Profile("dev")
public class DevConfiguration {

    @Bean
    public EmailService emailService() {
        return new FakeEmailService();
    }
}
```

This bean exists only when:

```text id="q5p8n2"
dev
```

is active.

Spring Framework's `@Profile` annotation controls whether a configuration component is registered based on active profiles. ([docs.spring.io](https://docs.spring.io/spring-framework/reference/core/beans/environment.html?utm_source=chatgpt.com))

---

# 14. Production Implementation

Maybe in DEV:

```java id="n7c3p9"
@Profile("dev")
@Bean
EmailClient emailClient() {
    return new FakeEmailClient();
}
```

and in PROD:

```java id="k2m8x5"
@Profile("prod")
@Bean
EmailClient emailClient() {
    return new RealEmailClient();
}
```

Then:

```text id="g4q7m1"
dev
 ↓
FakeEmailClient

prod
 ↓
RealEmailClient
```

Same application codebase.

Different bean implementation.

---

# 15. Profile Expressions

You can use profile expressions in `@Profile`.

For example:

```java id="s8m2v4"
@Profile("prod & !test")
```

Meaning:

```text id="y3q7n1"
prod active
AND
test not active
```

You can also use OR-style expressions such as:

```java id="f5m8c2"
@Profile("dev | test")
```

Then the bean is active for either profile.

This is useful, but don't create complicated profile expressions unless there's a real need.

---

# 16. Profile Groups

Now an advanced but useful Boot feature.

Suppose:

```text id="u9p4x6"
Production requires:

prod
metrics
security
```

Instead of:

```properties id="w5m3n8"
spring.profiles.active=prod,metrics,security
```

you can define a profile group.

For example:

```properties id="r7q2c9"
spring.profiles.group.production=prod,metrics,security
```

Then:

```properties id="m3x8v1"
spring.profiles.active=production
```

activates:

```text id="k6p2q7"
production
   ↓
prod
metrics
security
```

Spring Boot supports profile groups specifically to provide a convenient logical name for a set of profiles. ([Home][1])

---

# 17. Why Profile Groups Are Useful

Imagine:

```text id="z4n7c2"
production
  ├── prod-db
  ├── prod-monitoring
  └── prod-security
```

Deployment only needs:

```text id="p8m3q6"
SPRING_PROFILES_ACTIVE=production
```

instead of knowing every internal profile.

This is cleaner for larger applications.

---

# 18. Important Restriction

`spring.profiles.active`, `spring.profiles.default`, and profile grouping configuration should be defined in **non-profile-specific documents**. They cannot be used inside a document that is itself activated only for a profile. ([Home][1])

So don't do something like:

```properties id="j9m2x4"
# application-prod.properties

spring.profiles.active=test
```

That's conceptually wrong.

Instead, profile activation belongs in the general configuration/deployment environment.

---

# 19. Modern YAML Example

You can keep configuration in one `application.yaml` with multiple documents and conditionally activate documents by profile.

For example:

```yaml id="c8r4m1"
spring:
  application:
    name: employee-api

---
spring:
  config:
    activate:
      on-profile: dev

server:
  port: 8081

---
spring:
  config:
    activate:
      on-profile: prod

server:
  port: 8080
```

Spring Boot supports multi-document YAML and property files, with activation properties such as `spring.config.activate.on-profile`. ([Home][2])

This is an alternative to having separate:

```text
application-dev.yaml
application-prod.yaml
```

files.

---

# 20. Which Approach Should You Use?

For your current level, I'd recommend:

```text id="w3q7m9"
application.yaml
application-dev.yaml
application-test.yaml
application-prod.yaml
```

Why?

Because it's easy to understand:

```text id="k5x2n8"
common configuration
        +
environment-specific configuration
```

For larger systems, multi-document configuration and profile groups can be useful.

---

# 21. Our Employee API Structure

Let's design it properly.

```text id="j7m3c8"
src/main/resources/
│
├── application.yaml
├── application-dev.yaml
├── application-test.yaml
└── application-prod.yaml
```

### `application.yaml`

```yaml id="q2x8m6"
spring:
  application:
    name: employee-api

app:
  employee:
    max-page-size: 100
```

### `application-dev.yaml`

```yaml id="k9p4v1"
server:
  port: 8081

spring:
  datasource:
    url: jdbc:h2:mem:employee_dev
    username: sa
    password:

logging:
  level:
    com.practice.employeeapi: DEBUG
```

### `application-test.yaml`

```yaml id="f7m2c5"
spring:
  datasource:
    url: jdbc:h2:mem:employee_test
    username: sa
    password:

logging:
  level:
    com.practice.employeeapi: INFO
```

### `application-prod.yaml`

```yaml id="n4q8x2"
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

logging:
  level:
    com.practice.employeeapi: INFO
```

Notice something important:

We aren't putting:

```text id="s5m9c3"
production password
```

directly into `application-prod.yaml`.

We're using environment variables.

---

# 22. Running DEV

```bash id="b7x3m8"
java -jar employee-api.jar \
    --spring.profiles.active=dev
```

Result:

```text id="q4m2n9"
application.yaml
       +
application-dev.yaml
       ↓
DEV environment
```

---

# 23. Running TEST

```bash id="p8c4v1"
java -jar employee-api.jar \
    --spring.profiles.active=test
```

Result:

```text id="r3m7x5"
application.yaml
       +
application-test.yaml
       ↓
TEST environment
```

---

# 24. Running PROD

```bash id="z6n2q8"
SPRING_PROFILES_ACTIVE=prod \
DB_URL='jdbc:postgresql://prod-db/employee' \
DB_USERNAME='employee_user' \
DB_PASSWORD='secret' \
java -jar employee-api.jar
```

Result:

```text id="x7m3c9"
application.yaml
       +
application-prod.yaml
       +
Environment Variables
       ↓
PRODUCTION
```

This is exactly the kind of deployment separation you want.

---

# 25. Profile vs Environment Variable

Don't confuse these.

### Profile

```text id="d9q4m2"
Which environment/configuration set?
```

Example:

```text
prod
```

### Environment variable

```text id="p1x8v5"
What is the actual value?
```

Example:

```text
DB_PASSWORD=...
```

So:

```text id="m5c7n3"
Profile
 ↓
Select configuration

Environment variable
 ↓
Supply configuration value
```

They work together.

---

# 26. Profiles vs `@ConditionalOnProperty`

You now know both.

### Profile

```java id="x7p2m9"
@Profile("prod")
```

means:

> Activate this component in the `prod` profile.

### Property condition

```java id="k3m8c1"
@ConditionalOnProperty(
    name = "feature.email.enabled",
    havingValue = "true"
)
```

means:

> Activate this configuration when this property matches.

So:

```text id="h8q4v2"
Profile
 ↓
Environment identity

Property condition
 ↓
Feature/configuration condition
```

They can complement each other.

---

# 27. Don't Create Profiles for Every Tiny Feature

Bad:

```text id="e3q8m4"
profile-a
profile-b
profile-c
profile-d
profile-e
profile-f
```

Profiles are primarily useful for environment/configuration segmentation.

For feature toggles, often prefer explicit configuration:

```properties id="s5m2n7"
feature.notifications.enabled=true
```

and then:

```java id="u9c4x1"
@ConditionalOnProperty(...)
```

Use the tool that matches the problem.

---

# 28. Interview Questions

### What is a Spring Profile?

> A named configuration/environment grouping that lets you selectively activate configuration and beans for different environments. ([Home][1])

### How do you activate a profile?

```properties id="j8x3m7"
spring.profiles.active=dev
```

or:

```bash id="n4q7c2"
java -jar app.jar --spring.profiles.active=dev
```

or through:

```text id="x5m9p1"
SPRING_PROFILES_ACTIVE
```

### What is `application-prod.properties`?

> Profile-specific configuration loaded when the `prod` profile is active. Profile-specific configuration overrides non-profile-specific configuration. ([Home][2])

### What is `spring.profiles.default`?

> The profile used when no explicit profile is active; Boot's default value is `default`. ([Home][1])

### Can multiple profiles be active?

Yes:

```properties id="w7m3x9"
spring.profiles.active=prod,metrics
```

### What is a profile group?

> A logical profile name that activates a set of profiles together. ([Home][1])

### `@Profile` vs `spring.profiles.active`?

> `spring.profiles.active` selects the active environment profiles; `@Profile` controls whether a particular bean/configuration is registered for those active profiles.

---

# 29. Best Practices

```text id="q4x7m2"
✅ Keep common configuration in application.yaml
✅ Use application-{profile}.yaml for environment differences
✅ Use environment variables for secrets
✅ Keep prod credentials outside source control
✅ Use profiles for environments/configuration groups
✅ Use profile groups when environments require multiple profiles
✅ Use @Profile for environment-specific beans
✅ Prefer property-based feature flags for individual features
❌ Don't hardcode prod secrets
❌ Don't scatter environment checks through business code
```

---

# 30. The Mental Model

Remember:

```text id="m7c2p9"
                 Spring Boot
                      │
              Active Profile
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
       dev           test          prod
        │             │             │
        ▼             ▼             ▼
application-dev   application-test   application-prod
     .yaml             .yaml              .yaml
        │               │                  │
        └───────────────┼──────────────────┘
                        ▼
                Spring Environment
                        │
                        ▼
             @Value / @ConfigurationProperties
                        │
                        ▼
                   Application
```

---

# 📍 Where We Are

```text id="k8m3q5"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters & Dependency Management
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server & Startup
├── ✅ Chapter 7 — Externalized Configuration
├── ✅ Chapter 8 — Profiles ⭐⭐⭐⭐⭐
│      ├── spring.profiles.active
│      ├── spring.profiles.default
│      ├── @Profile
│      ├── application-{profile}.yaml
│      ├── Multiple Profiles
│      └── Profile Groups
│
└── ⏭️ Chapter 9 — @ConfigurationProperties ⭐⭐⭐⭐⭐
       ↓
       Type-safe configuration
       ↓
       Binding
       ↓
       Relaxed binding
       ↓
       Validation
       ↓
       Nested configuration
       ↓
       Immutable records
       ↓
       Configuration metadata
```

Next is **`@ConfigurationProperties` in depth**, where we'll build a strongly typed configuration object for our Employee API and compare it directly with `@Value`, including validation and nested properties.

[1]: https://docs.spring.io/spring-boot/reference/features/profiles.html?utm_source=chatgpt.com "Profiles :: Spring Boot"
[2]: https://docs.spring.io/spring-boot/reference/features/external-config.html?utm_source=chatgpt.com "Externalized Configuration :: Spring Boot"
