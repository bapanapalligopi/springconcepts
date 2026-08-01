# Chapter 13.4 — Spring Boot Actuator Health Endpoint ⭐⭐⭐⭐⭐

The **Health Endpoint** is the **most frequently used Actuator endpoint** in production.

If someone asks:

> **"Is your application running correctly?"**

The answer usually comes from:

```http
GET /actuator/health
```

Every major cloud platform (AWS, Azure, GCP), Kubernetes, Docker, Prometheus, and Load Balancers use this endpoint.

This is one of the **most important Spring Boot topics** for interviews and enterprise applications.

---

# Chapter Roadmap

```text
Health Endpoint
│
├── 1. What is Health Endpoint?
├── 2. Why Do We Need It?
├── 3. Internal Working
├── 4. Health Contributors
├── 5. Built-in Health Indicators
├── 6. Health Status
├── 7. Database Health
├── 8. Disk Space Health
├── 9. Redis/Kafka/RabbitMQ Health
├── 10. Custom Health Indicators
├── 11. Health Groups
├── 12. Kubernetes Readiness & Liveness
├── 13. Enterprise Architecture
├── 14. Best Practices
└── Interview Questions
```

---

# 1. What is the Health Endpoint?

The Health Endpoint tells whether your application is healthy.

Example:

```http
GET /actuator/health
```

Possible response:

```json
{
  "status": "UP"
}
```

Think of it as a **medical check-up** for your application.

---

# 2. Why Do We Need It?

Imagine an Employee Management System.

```text
Employee API

↓

Database

↓

Redis

↓

Kafka

↓

External Payment API
```

Just because the application is running doesn't mean it's healthy.

Example:

```text
Application Running

↓

Database Down

↓

Application Not Functional
```

The Health Endpoint checks these dependencies.

---

# 3. Basic Health Flow

```text
Monitoring Tool

↓

GET /actuator/health

↓

Spring Boot

↓

Check Components

↓

Generate Status

↓

Return JSON
```

---

# 4. First Example

Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Request:

```http
GET http://localhost:8080/actuator/health
```

Response:

```json
{
  "status": "UP"
}
```

---

# 5. What Happens Internally?

Spring Boot doesn't simply return `"UP"`.

It performs checks.

```text
Request

↓

HealthEndpoint

↓

HealthContributorRegistry

↓

HealthIndicator 1

↓

HealthIndicator 2

↓

HealthIndicator 3

↓

Combine Results

↓

Return Final Status
```

---

# 6. Health Contributors

A **Health Contributor** is a component that reports the health of one part of the system.

Examples:

```text
Database

↓

Disk Space

↓

Redis

↓

MongoDB

↓

RabbitMQ

↓

Kafka

↓

Custom Service
```

Each contributor independently reports its status.

---

# 7. Built-in Health Indicators

Spring Boot automatically creates health indicators when corresponding dependencies are present.

Examples:

| Dependency    | Health Indicator             |
| ------------- | ---------------------------- |
| DataSource    | DatabaseHealthIndicator      |
| Redis         | RedisHealthIndicator         |
| MongoDB       | MongoHealthIndicator         |
| Cassandra     | CassandraHealthIndicator     |
| RabbitMQ      | RabbitHealthIndicator        |
| Elasticsearch | ElasticsearchHealthIndicator |
| Disk          | DiskSpaceHealthIndicator     |
| Ping          | PingHealthIndicator          |

You don't create these manually.

---

# 8. Health Aggregation

Suppose:

```text
Database

↓

UP
```

Redis:

```text
UP
```

Disk:

```text
UP
```

Kafka:

```text
DOWN
```

Spring combines everything.

```text
Database

↓

UP

Redis

↓

UP

Kafka

↓

DOWN

↓

Overall Status

↓

DOWN
```

One critical failure can affect the overall health depending on the configured status aggregator.

---

# 9. Health Status Values

Common statuses:

```text
UP

DOWN

OUT_OF_SERVICE

UNKNOWN
```

Meaning:

| Status         | Meaning                    |
| -------------- | -------------------------- |
| UP             | Everything is healthy      |
| DOWN           | Component failure          |
| OUT_OF_SERVICE | Intentionally unavailable  |
| UNKNOWN        | State cannot be determined |

---

# 10. Database Health Example

Suppose MySQL is connected.

```text
Application

↓

Database

↓

Connection Successful

↓

Status = UP
```

Response:

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    }
  }
}
```

Now database crashes.

```text
Application

↓

Database

↓

Connection Failed
```

Response:

```json
{
  "status": "DOWN",
  "components": {
    "db": {
      "status": "DOWN"
    }
  }
}
```

---

# 11. Disk Space Health

Spring automatically checks disk space.

Example:

```text
Disk

↓

Free Space

↓

Enough?

↓

YES

↓

UP
```

If disk becomes critically full:

```text
Disk

↓

1 MB Free

↓

Threshold Crossed

↓

DOWN
```

This helps prevent applications from failing due to lack of storage.

---

# 12. Redis Health

Suppose Redis is running.

```text
Spring Boot

↓

PING Redis

↓

PONG

↓

UP
```

If Redis is unreachable:

```text
Spring Boot

↓

PING Redis

↓

Timeout

↓

DOWN
```

---

# 13. Kafka Health

Kafka health works similarly.

```text
Application

↓

Connect Kafka

↓

Success

↓

UP
```

Failure:

```text
Connection Failed

↓

DOWN
```

---

# 14. Health JSON Structure

Typical response:

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

Overall status comes first.

Component details follow.

---

# 15. Showing Component Details

By default, health details may be hidden.

Example:

```json
{
  "status": "UP"
}
```

Enable details:

```properties
management.endpoint.health.show-details=always
```

Now:

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    }
  }
}
```

In production, consider restricting details to authorized users instead of exposing them publicly.

---

# 16. Custom Health Indicator

Suppose your application depends on:

```text
Payroll Service
```

Spring doesn't know how to check it.

Create your own indicator.

```java
@Component
public class PayrollHealthIndicator
        implements HealthIndicator {

    @Override
    public Health health() {

        boolean serviceUp = true;

        if (serviceUp) {
            return Health.up()
                    .withDetail("Payroll", "Available")
                    .build();
        }

        return Health.down()
                .withDetail("Payroll", "Unavailable")
                .build();
    }
}
```

Now it becomes part of:

```text
/actuator/health
```

---

# 17. Health Groups

Sometimes different users need different health views.

Example:

```text
Health

├── Liveness
├── Readiness
└── Full Health
```

This allows Kubernetes to ask one question while administrators ask another.

We'll revisit this when discussing Kubernetes.

---

# 18. Kubernetes Integration

Kubernetes uses two important probes.

### Liveness Probe

Question:

```text
Is the application alive?
```

If NO:

```text
Restart Pod
```

---

### Readiness Probe

Question:

```text
Can this application receive traffic?
```

If NO:

```text
Remove From Load Balancer
```

Flow:

```text
Kubernetes

↓

/actuator/health/liveness

↓

Alive?

↓

YES

↓

Keep Running

-------------------

/actuator/health/readiness

↓

Ready?

↓

YES

↓

Receive Traffic
```

---

# 19. Enterprise Monitoring Flow

```text
Prometheus

↓

GET /actuator/health

↓

Spring Boot

↓

Database Check

↓

Redis Check

↓

Kafka Check

↓

Disk Check

↓

Aggregate Status

↓

Return JSON

↓

Grafana Dashboard

↓

Alert Manager
```

---

# 20. Common Mistakes

### Mistake 1

Assuming `"UP"` means every business feature works.

It only means configured health contributors report healthy.

---

### Mistake 2

Exposing detailed health information publicly.

Avoid:

```properties
management.endpoint.health.show-details=always
```

for public, unauthenticated access in production.

---

### Mistake 3

Ignoring custom dependencies.

If your application depends on an external payroll or billing service, create a custom `HealthIndicator`.

---

# 21. Best Practices

```text
✅ Enable Health Endpoint in production

✅ Secure health details

✅ Add custom health indicators

✅ Monitor database connectivity

✅ Monitor disk space

✅ Use readiness and liveness probes

❌ Don't expose sensitive details publicly

❌ Don't ignore external service health
```

---

# 22. Interview Questions

### What is the Health Endpoint?

> The Health Endpoint reports the health status of a Spring Boot application and its monitored components.

---

### What URL is used?

```text
/actuator/health
```

---

### What interface is used for custom checks?

```java
HealthIndicator
```

---

### Common status values?

* `UP`
* `DOWN`
* `OUT_OF_SERVICE`
* `UNKNOWN`

---

### How do you always show health details?

```properties
management.endpoint.health.show-details=always
```

Use this carefully in production.

---

### What is the difference between liveness and readiness?

| Liveness          | Readiness                            |
| ----------------- | ------------------------------------ |
| Is the app alive? | Is the app ready to receive traffic? |
| Failed → Restart  | Failed → Remove from traffic         |

---

# 23. Complete Internal Flow

```text
GET /actuator/health

↓

HealthEndpoint

↓

HealthContributorRegistry

↓

Database Indicator

↓

Disk Indicator

↓

Redis Indicator

↓

Kafka Indicator

↓

Custom Indicators

↓

Status Aggregator

↓

JSON Response

↓

Monitoring Tool
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.1 Introduction
├── ✅ 13.2 Adding Actuator
├── ✅ 13.3 Actuator Endpoints
├── ✅ 13.4 Health Endpoint ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.5 Info Endpoint ⭐⭐⭐
       ↓
       Application Information
       ↓
       Build Information
       ↓
       Git Information
       ↓
       Custom Info
       ↓
       Enterprise Metadata
```

## Next Chapter (13.5) — **Info Endpoint**

We'll learn:

* What `/actuator/info` does
* How to display application name and version
* Build metadata (`build-info.properties`)
* Git metadata (`git.properties`)
* Custom information
* Enterprise release/version management
* Securely exposing application metadata
* Best practices for production environments
