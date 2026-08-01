# Chapter 13.3 — Spring Boot Actuator Endpoints ⭐⭐⭐⭐⭐

In the previous chapter, we learned how to **add Spring Boot Actuator**.

Now we'll study the **heart of Actuator**:

> **Endpoints**

Every monitoring tool—such as Prometheus, Grafana, Kubernetes, AWS, or Azure—interacts with your application through these endpoints.

Understanding them thoroughly is essential for enterprise development.

---

# Chapter Roadmap

```text id="7r1kpd"
Actuator Endpoints
│
├── 1. What is an Endpoint?
├── 2. Endpoint Categories
├── 3. Built-in Endpoints
├── 4. Read vs Write Endpoints
├── 5. HTTP vs JMX Exposure
├── 6. Endpoint IDs
├── 7. Internal Working
├── 8. Enterprise Architecture
├── 9. Common Mistakes
├── 10. Best Practices
└── Interview Questions
```

---

# 1. What is an Actuator Endpoint?

An **endpoint** is a URL exposed by Spring Boot that provides **management or monitoring information**.

Example:

```text
GET /actuator/health
```

Unlike your business API:

```text
GET /employees
```

which returns business data,

Actuator endpoints return **application data**.

---

## Business API vs Actuator API

```text
Client
   │
   ├──────────────► /employees
   │                     │
   │                     ▼
   │             Employee Data
   │
   └──────────────► /actuator/health
                         │
                         ▼
                Application Health
```

Different purpose.

---

# 2. Types of Endpoints

Spring Boot provides many endpoints.

```text
Health

Info

Metrics

Beans

Environment

Mappings

Loggers

Conditions

Config Props

Caches

Threaddump

Heapdump

Shutdown
```

Each endpoint answers a specific question.

---

# 3. Endpoint Categories

```text
                   Actuator
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
Monitoring        Diagnostics      Management
     │                 │                 │
Health          Beans            Loggers
Metrics         Env              Shutdown
Info            Mappings         Caches
```

---

# 4. Most Important Endpoints

| Endpoint                | Purpose                          |
| ----------------------- | -------------------------------- |
| `/actuator/health`      | Application health               |
| `/actuator/info`        | Application information          |
| `/actuator/metrics`     | Performance metrics              |
| `/actuator/beans`       | All Spring beans                 |
| `/actuator/env`         | Environment properties           |
| `/actuator/mappings`    | All HTTP mappings                |
| `/actuator/loggers`     | View/change logging levels       |
| `/actuator/caches`      | Cache information                |
| `/actuator/threaddump`  | Running threads                  |
| `/actuator/heapdump`    | JVM heap dump                    |
| `/actuator/configprops` | `@ConfigurationProperties` beans |

These are the endpoints you'll encounter most often.

---

# 5. Endpoint IDs

Every endpoint has an **ID**.

Examples:

```text
health

info

metrics

beans

env

mappings
```

Spring combines the base path and endpoint ID.

```
Base Path
↓

/actuator

+

Endpoint ID
↓

health

=

/actuator/health
```

---

# 6. Base Path

By default:

```text
/actuator
```

Examples:

```text
/actuator/health

/actuator/info

/actuator/metrics

/actuator/beans
```

You can change it.

Example:

```properties
management.endpoints.web.base-path=/manage
```

Now:

```text
/manage/health

/manage/info

/manage/metrics
```

---

# 7. Read vs Write Endpoints

Most endpoints are **read-only**.

Example:

```text
GET /actuator/health
```

returns information.

Some endpoints support **write operations**.

Example:

```text
POST /actuator/shutdown
```

This can stop the application (when enabled).

---

## Comparison

| Read       | Write                     |
| ---------- | ------------------------- |
| GET        | POST                      |
| Safe       | Changes application state |
| Monitoring | Management                |

---

# 8. HTTP vs JMX Exposure

Actuator endpoints can be exposed through:

```text
Actuator
│
├── HTTP
│      │
│      └── /actuator/health
│
└── JMX
       │
       └── MBeans
```

Most modern applications use **HTTP**.

Some enterprise systems still use **JMX** for JVM management.

---

# 9. Internal Architecture

```text
                Spring Boot
                     │
                     ▼
           Endpoint Discoverer
                     │
      ┌──────────────┼──────────────┐
      │              │              │
 HealthEndpoint  MetricsEndpoint  BeansEndpoint
      │              │              │
      └──────────────┼──────────────┘
                     ▼
             Endpoint Mapping
                     ▼
          /actuator/<endpoint>
```

Spring scans for endpoint beans and maps them automatically.

---

# 10. Endpoint Discovery

During startup:

```text
Application Starts

↓

Load Actuator

↓

Find Endpoint Beans

↓

Assign Endpoint IDs

↓

Register HTTP Mappings

↓

Application Ready
```

You don't write controller methods for these endpoints.

---

# 11. Endpoint Exposure

Creating an endpoint and exposing it are **different things**.

```text
Endpoint Bean Created

↓

Should It Be Exposed?

↓

YES

↓

Available Over HTTP

↓

NO

↓

Hidden
```

This is why some endpoints exist internally but are not accessible through the web.

---

# 12. Enterprise Architecture

```text
                    Monitoring Server
                           │
                           ▼
                  /actuator/health
                           │
                           ▼
                  Spring Boot App
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
      Database          Redis          Kafka
```

The monitoring server doesn't inspect your database directly.

It asks **your application**, which reports its status.

---

# 13. Example Responses

### Health

```json
{
  "status": "UP"
}
```

---

### Info

```json
{
  "app": {
    "name": "Employee API"
  }
}
```

---

### Metrics

```json
{
  "names": [
    "jvm.memory.used",
    "http.server.requests",
    "system.cpu.usage"
  ]
}
```

Notice:

Each endpoint returns a different type of information.

---

# 14. Common Mistakes

### Mistake 1

Assuming every endpoint is public.

Wrong.

Many endpoints are **not exposed by default**.

---

### Mistake 2

Confusing:

```text
/actuator/info
```

with

```text
/info
```

The first is an Actuator endpoint.

The second is just another application endpoint if you create one.

---

### Mistake 3

Exposing every endpoint in production.

Example:

```properties
management.endpoints.web.exposure.include=*
```

This is useful for local development, but in production it can expose sensitive information such as environment properties, bean definitions, and mappings if not properly secured.

---

# 15. Best Practices

```text
✅ Expose only required endpoints

✅ Secure management endpoints

✅ Monitor health and metrics

✅ Separate monitoring from business APIs

❌ Don't expose sensitive endpoints publicly

❌ Don't rely on defaults without understanding them
```

---

# 16. Interview Questions

### What is an Actuator endpoint?

> An Actuator endpoint is a management endpoint that exposes monitoring or operational information about a Spring Boot application.

---

### What is the default base path?

```text
/actuator
```

---

### Are all endpoints exposed over HTTP?

No.

Many exist internally but must be explicitly exposed.

---

### Can Actuator endpoints use JMX?

Yes.

Spring Boot supports both HTTP and JMX exposure.

---

### Which endpoint is most commonly used by monitoring tools?

```text
/actuator/health
```

because it indicates whether the application is healthy.

---

# 17. Complete Internal Flow

```text
Application Starts

↓

Actuator Auto Configuration

↓

Create Endpoint Beans

↓

Assign Endpoint IDs

↓

Apply Exposure Rules

↓

Register HTTP Mappings

↓

Monitoring Tools Access Endpoints
```

---

# 18. Big Picture

```text
                  Spring Boot
                       │
                       ▼
                 Actuator Module
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
   Health           Metrics          Environment
      ▼                ▼                ▼
     HTTP             HTTP             HTTP
      ▼                ▼                ▼
 Prometheus       Grafana         Developers
```

This is the architecture used by most enterprise Spring Boot applications.

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.1 Introduction
├── ✅ 13.2 Adding Actuator
├── ✅ 13.3 Actuator Endpoints ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.4 Health Endpoint ⭐⭐⭐⭐⭐
       ↓
       Health Contributors
       ↓
       Database Health
       ↓
       Disk Space Health
       ↓
       Redis Health
       ↓
       Custom Health Indicators
       ↓
       Enterprise Monitoring
```

## Next Chapter (13.4) — **Health Endpoint**

This is the **most important Actuator endpoint**.

We'll cover:

* How `/actuator/health` works internally
* Health contributors
* Database health checks
* Disk space monitoring
* Redis, RabbitMQ, Kafka health
* Custom health indicators
* Health groups
* Liveness and readiness probes (Kubernetes)
* Enterprise production monitoring architecture

This is one of the highest-priority Spring Boot interview topics.
