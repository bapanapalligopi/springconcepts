# Spring Boot — Chapter 10: Spring Boot Actuator ⭐⭐⭐⭐⭐

This is one of the **most important production features** in Spring Boot.

Up to now, we've built an application.

Now the question becomes:

> **How do we monitor, manage, and troubleshoot a running application in production?**

That's exactly what **Spring Boot Actuator** provides.

---

# 1. What is Spring Boot Actuator?

Spring Boot Actuator adds **production-ready features** to your application.

It exposes information such as:

* Application health
* Metrics
* Environment
* Beans
* Configuration
* Loggers
* Thread dumps
* Heap dumps
* Mappings

Think of it as the **control panel** of your application.

```text
                    Spring Boot Application
                             │
         ┌───────────────────┼────────────────────┐
         ▼                   ▼                    ▼
     Business APIs      Actuator APIs       Monitoring Tools
      /api/**          /actuator/**       Prometheus, Grafana
```

---

# 2. Why Do We Need Actuator?

Imagine your application is running in production.

Users complain:

> "The application is slow."

Questions you might ask:

* Is it alive?
* Is the database reachable?
* Is memory almost full?
* How many requests are coming in?
* Which endpoints exist?
* Which beans are loaded?

Without Actuator:

```text
No visibility
```

With Actuator:

```text
Application
      │
      ▼
Health
Metrics
Beans
Mappings
Environment
Thread Dump
Heap Dump
```

---

# 3. Adding Actuator

Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>

    <artifactId>
        spring-boot-starter-actuator
    </artifactId>
</dependency>
```

Once added, Spring Boot auto-configures Actuator endpoints.

---

# 4. Default Endpoint

Run application.

Visit:

```text
http://localhost:8080/actuator
```

You'll see links to exposed endpoints (depending on configuration).

Example:

```json
{
   "_links": {
      "self": "...",
      "health": "...",
      "health-path": "..."
   }
}
```

---

# 5. Health Endpoint

Most important endpoint:

```text
GET

/actuator/health
```

Example:

```json
{
    "status": "UP"
}
```

Meaning:

```text
Application

↓

Healthy
```

---

# 6. Possible Health Status

Common values:

```text
UP

DOWN

OUT_OF_SERVICE

UNKNOWN
```

Meaning:

```text
UP
↓

Application working

DOWN
↓

Critical problem

OUT_OF_SERVICE
↓

Intentionally unavailable

UNKNOWN
↓

Cannot determine state
```

---

# 7. Health Contributors

Health isn't only about the application.

Actuator checks many components.

Example:

```text
Application

↓

Database

↓

Disk Space

↓

Redis

↓

RabbitMQ

↓

Custom Checks
```

Overall health depends on these contributors.

---

# 8. Database Health Example

Suppose PostgreSQL is running.

Health:

```json
{
   "status":"UP"
}
```

Now database goes down.

Health:

```json
{
   "status":"DOWN"
}
```

This allows monitoring systems to detect problems immediately.

---

# 9. Showing Health Details

By default:

```json
{
   "status":"UP"
}
```

You can configure:

```properties
management.endpoint.health.show-details=always
```

Now response becomes:

```json
{
   "status":"UP",

   "components":{

      "db":{

         "status":"UP"
      },

      "diskSpace":{

         "status":"UP"
      }

   }
}
```

Useful during development.

In production, be careful because detailed health information may reveal infrastructure details.

---

# 10. Actuator Endpoint Architecture

```text
Application
      │
      ▼
Actuator
      │
 ┌────┼─────┐
 ▼    ▼     ▼
Health Metrics Info
```

Every endpoint is implemented as a Spring-managed endpoint.

---

# 11. Info Endpoint

URL:

```text
/actuator/info
```

Initially:

```json
{}
```

Add:

```properties
management.info.env.enabled=true

info.app.name=Employee API

info.app.version=1.0
```

Response:

```json
{
   "app":{

      "name":"Employee API",

      "version":"1.0"
   }
}
```

Useful for build and version information.

---

# 12. Metrics Endpoint

Endpoint:

```text
/actuator/metrics
```

Example response:

```text
jvm.memory.used

jvm.gc.pause

system.cpu.usage

http.server.requests

process.uptime
```

These are metric names.

To inspect one:

```text
/actuator/metrics/jvm.memory.used
```

---

# 13. Common Metrics

```text
JVM Memory

Heap Memory

CPU Usage

GC Time

Thread Count

HTTP Requests

Disk Usage

Datasource Metrics

Tomcat Metrics
```

These metrics help diagnose application performance.

---

# 14. HTTP Metrics

One of the most useful metrics.

Example:

```text
http.server.requests
```

Contains:

* Request count
* Response time
* Status codes
* URI
* Method

Example:

```text
GET /employees

↓

Average = 50 ms

Count = 12,500
```

---

# 15. Beans Endpoint

Endpoint:

```text
/actuator/beans
```

Shows every Spring Bean.

Example:

```text
EmployeeController

EmployeeService

EmployeeRepository

DispatcherServlet

DataSource

JwtAuthenticationFilter
```

Very useful while debugging.

---

# 16. Environment Endpoint

Endpoint:

```text
/actuator/env
```

Shows:

```text
application.yaml

Environment Variables

System Properties

Profiles

Configuration Sources
```

This helps determine **where a property came from**.

Sensitive values are sanitized by default.

---

# 17. Mappings Endpoint

Endpoint:

```text
/actuator/mappings
```

Shows every request mapping.

Example:

```text
GET

/api/employees

↓

EmployeeController
```

Very useful when debugging routing problems.

---

# 18. Conditions Endpoint

Endpoint:

```text
/actuator/conditions
```

Shows:

```text
Why AutoConfiguration happened

Why it didn't happen
```

Example:

```text
DataSourceAutoConfiguration

↓

Matched
```

or

```text
RedisAutoConfiguration

↓

Did not match
```

This is extremely helpful when troubleshooting Boot auto-configuration.

---

# 19. ConfigProps Endpoint

Endpoint:

```text
/actuator/configprops
```

Shows every:

```text
@ConfigurationProperties
```

bean.

Example:

```text
JwtProperties

EmployeeProperties

DatasourceProperties
```

Great for verifying configuration binding.

---

# 20. Loggers Endpoint

Endpoint:

```text
/actuator/loggers
```

Allows viewing logger levels.

Example:

```text
ROOT

↓

INFO
```

You can also change logger levels at runtime (if enabled and authorized), which is very useful for temporary production debugging.

---

# 21. Thread Dump

Endpoint:

```text
/actuator/threaddump
```

Shows JVM threads.

Useful when investigating:

* Deadlocks
* High CPU usage
* Hanging requests

---

# 22. Heap Dump

Endpoint:

```text
/actuator/heapdump
```

Provides a heap dump that can be analyzed with JVM tools.

Useful for:

* Memory leaks
* OutOfMemoryError investigations

Be careful—heap dumps may contain sensitive application data and are typically restricted.

---

# 23. Scheduled Tasks

Endpoint:

```text
/actuator/scheduledtasks
```

Lists every scheduled task.

Example:

```text
Cleanup Job

Runs every hour
```

Useful when applications use `@Scheduled`.

---

# 24. Exposing Endpoints

By default, only a small set of endpoints is exposed over HTTP.

Expose more:

```properties
management.endpoints.web.exposure.include=*
```

or:

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

Never expose everything in production unless you fully understand the security implications.

---

# 25. Securing Actuator

Bad:

```text
Internet

↓

/actuator/env
```

Anyone can access it.

Good:

```text
Internet

↓

Spring Security

↓

Authenticated User

↓

Actuator
```

In production:

* Secure Actuator with Spring Security
* Limit exposed endpoints
* Restrict administrative endpoints to trusted users or networks

---

# 26. Management Port

Actuator can run on a different port.

Example:

```properties
management.server.port=9091
```

Application:

```text
8080
```

Actuator:

```text
9091
```

Useful for operational separation.

---

# 27. Prometheus Integration

One of the most common enterprise setups.

Application:

```text
Spring Boot

↓

/actuator/prometheus

↓

Prometheus

↓

Grafana

↓

Dashboard
```

Prometheus periodically scrapes metrics, and Grafana visualizes them.

---

# 28. Custom Health Indicator

Suppose we want to check an external payment service.

```java
@Component
public class PaymentHealthIndicator
        implements HealthIndicator {

    @Override
    public Health health() {

        boolean available = checkPaymentService();

        if (available) {
            return Health.up().build();
        }

        return Health.down()
                .withDetail("service", "Payment API unavailable")
                .build();
    }

    private boolean checkPaymentService() {
        // call external service or perform a lightweight check
        return true;
    }
}
```

Now:

```text
/actuator/health
```

includes:

```text
payment

↓

UP
```

This is how you add application-specific health checks.

---

# 29. Liveness and Readiness

Very important for Kubernetes.

**Liveness**

> Is the application process alive?

**Readiness**

> Is the application ready to receive traffic?

```text
Kubernetes

↓

Liveness Probe

↓

Application Alive?

↓

Readiness Probe

↓

Ready for Requests?
```

Spring Boot Actuator supports health groups that can be used for liveness and readiness probes.

---

# 30. Production Architecture

```text
                    Spring Boot
                          │
                    Business APIs
                          │
         ┌────────────────┼────────────────┐
         ▼                ▼                ▼
     /api/**        /actuator/**      Metrics
         │                │                │
         ▼                ▼                ▼
      Clients        Prometheus       Grafana
                          │
                          ▼
                   Operations Team
```

---

# 31. Common Interview Questions

### What is Spring Boot Actuator?

> Spring Boot Actuator provides production-ready features such as health checks, metrics, environment information, mappings, logging management, and monitoring endpoints.

---

### What is `/actuator/health`?

It reports the health status of the application and its registered health contributors.

---

### What is `/actuator/metrics`?

It exposes application and JVM metrics such as memory usage, CPU usage, request metrics, and garbage collection statistics.

---

### What is `/actuator/prometheus`?

It exposes metrics in a format that Prometheus can scrape.

---

### Why is Actuator important?

Because it provides operational visibility into a running application without modifying business code.

---

### Why shouldn't we expose every endpoint?

Some endpoints reveal sensitive operational details. They should be secured and only the required endpoints should be exposed.

---

### What is a custom `HealthIndicator`?

A Spring component that contributes application-specific health information to the health endpoint.

---

# 32. Best Practices

```text
✅ Add Actuator to every production application

✅ Expose only required endpoints

✅ Secure Actuator with Spring Security

✅ Use Prometheus for metrics collection

✅ Use Grafana for visualization

✅ Create custom HealthIndicators for critical dependencies

✅ Monitor HTTP request metrics

✅ Use readiness and liveness probes for Kubernetes

❌ Never expose all endpoints publicly

❌ Never expose sensitive operational endpoints without authentication
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
├── ✅ Chapter 9 — @ConfigurationProperties
├── ✅ Chapter 10 — Spring Boot Actuator ⭐⭐⭐⭐⭐
│
└── ⏭️ Chapter 11 — Spring Boot Testing ⭐⭐⭐⭐⭐
       ↓
       Unit Testing
       ↓
       Integration Testing
       ↓
       @SpringBootTest
       ↓
       @WebMvcTest
       ↓
       @DataJpaTest
       ↓
       MockMvc
       ↓
       Testcontainers
       ↓
       Best Testing Practices
```

The next chapter covers **Spring Boot Testing**, where we'll build a complete testing strategy from unit tests to full integration tests, including `@SpringBootTest`, `MockMvc`, slice testing (`@WebMvcTest`, `@DataJpaTest`), Mockito, and Testcontainers with practical enterprise examples.
