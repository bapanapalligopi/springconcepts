# Chapter 13.2 — Adding Spring Boot Actuator ⭐⭐⭐⭐⭐

In the previous chapter, we learned **what Actuator is**.

Now we'll actually **add it to a Spring Boot application** and understand **how it works internally**.

This chapter is very important because **everything else in Actuator depends on it**.

---

# Chapter Roadmap

```text
Adding Spring Boot Actuator
│
├── 1. Adding Dependency
├── 2. Auto Configuration
├── 3. Internal Working
├── 4. Endpoint Registration
├── 5. Default Endpoints
├── 6. First Project
├── 7. Verify Installation
├── 8. Enterprise Flow
├── 9. Common Mistakes
├── 10. Best Practices
└── Interview Questions
```

---

# 1. What Happens Without Actuator?

Suppose we have this project:

```text
Employee Management System

├── Controller
├── Service
├── Repository
└── Spring Boot
```

Application starts.

Available APIs:

```text
GET /employees

POST /employees

PUT /employees/{id}
```

Now try:

```http
GET /actuator/health
```

Response:

```text
404 Not Found
```

Why?

Because **Actuator isn't part of the project yet**.

---

# 2. Adding the Dependency

For Maven:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

That's it.

No additional configuration is required for basic functionality.

---

# 3. What Does This Dependency Bring?

This starter includes:

```text
Spring Boot Actuator

↓

Health Endpoint

↓

Metrics Support

↓

Info Endpoint

↓

Environment Endpoint

↓

Beans Endpoint

↓

Micrometer Integration

↓

Monitoring Infrastructure
```

Instead of adding many libraries individually, the starter brings compatible dependencies automatically.

---

# 4. What Happens During Startup?

Application starts.

```text
Spring Boot Starts

↓

Read pom.xml

↓

Find Actuator Starter

↓

Load Auto Configuration

↓

Register Actuator Beans

↓

Register Endpoints

↓

Application Ready
```

Notice:

You **never create** a `HealthEndpoint` bean manually.

Spring Boot does it for you.

---

# 5. Internal Auto Configuration

Spring Boot uses **Auto Configuration**.

Internally:

```text
spring.factories (Boot 2.x)
/AutoConfiguration.imports (Boot 3.x)

↓

Find Actuator AutoConfiguration

↓

Create Endpoint Beans

↓

Expose Endpoints
```

This is why Actuator works almost magically.

---

# 6. Bean Registration

Suppose you add the dependency.

Spring creates beans similar to:

```text
HealthEndpoint

InfoEndpoint

MetricsEndpoint

BeansEndpoint

EnvironmentEndpoint

MappingsEndpoint

LoggersEndpoint
```

These are normal Spring beans managed by the ApplicationContext.

---

# 7. Endpoint Registration Flow

```text
Application Starts

↓

ApplicationContext Created

↓

Actuator AutoConfiguration

↓

Create Endpoint Beans

↓

Map HTTP URLs

↓

Endpoints Ready
```

Each endpoint gets mapped under:

```text
/actuator
```

---

# 8. First Working Project

Project structure:

```text
employee-api
│
├── controller
├── service
├── repository
├── EmployeeApplication.java
├── pom.xml
└── application.properties
```

Dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Application:

```java
@SpringBootApplication
public class EmployeeApplication {

    public static void main(String[] args) {

        SpringApplication.run(EmployeeApplication.class, args);
    }
}
```

Run the application.

No additional code is needed.

---

# 9. First Actuator Endpoint

Open the browser or Postman.

```
GET http://localhost:8080/actuator
```

Typical response (simplified):

```json
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator"
    },
    "health": {
      "href": "http://localhost:8080/actuator/health"
    }
  }
}
```

This is called the **Actuator Root Endpoint**.

It lists available endpoints.

---

# 10. Understanding the Root Endpoint

Think of it like an API index.

```text
/actuator

↓

Shows Available Endpoints

↓

health

↓

info

↓

metrics

↓

beans

↓

env
```

Instead of remembering every URL, clients can discover them dynamically.

---

# 11. Default Exposed Endpoints

In Spring Boot 3.x, **not every endpoint is exposed over HTTP by default**.

Typically:

```text
HTTP Exposed

↓

health

↓

info
```

Many other endpoints exist as beans but are **not exposed** unless configured.

Example:

```http
GET /actuator/beans
```

may return:

```text
404 Not Found
```

or not be available over HTTP until exposure is configured.

We'll learn how to expose additional endpoints later.

---

# 12. Internal Architecture

```text
             Spring Boot
                  │
                  ▼
       Auto Configuration
                  │
                  ▼
         Actuator Module
                  │
     ┌────────────┼────────────┐
     │            │            │
 Health      Metrics       Beans
     │            │            │
     └────────────┼────────────┘
                  │
                  ▼
          HTTP Endpoints
                  │
                  ▼
      /actuator/health
      /actuator/info
```

---

# 13. Enterprise Architecture

```text
                   Users
                     │
                     ▼
                Employee API
                     │
         ┌───────────┴───────────┐
         │                       │
         ▼                       ▼
 Business APIs             Actuator APIs
         │                       │
         ▼                       ▼
   /employees             /actuator/health
                           /actuator/metrics
```

Notice:

Business APIs and monitoring APIs are separate.

---

# 14. Common Mistakes

### Mistake 1

Adding:

```xml
spring-boot-actuator
```

instead of:

```xml
spring-boot-starter-actuator
```

Always use the **starter**.

---

### Mistake 2

Expecting every endpoint to work immediately.

Only a small set is exposed by default.

---

### Mistake 3

Thinking Actuator replaces your APIs.

It doesn't.

Your APIs:

```text
/employees
/departments
```

Actuator APIs:

```text
/actuator/health
/actuator/metrics
```

Different purposes.

---

# 15. Best Practices

```text
✅ Use spring-boot-starter-actuator

✅ Keep Actuator enabled in production

✅ Secure Actuator endpoints

✅ Expose only required endpoints

❌ Don't expose sensitive endpoints publicly

❌ Don't disable health monitoring in production
```

---

# 16. Interview Questions

### How do you add Actuator?

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

### Do we create endpoint beans manually?

No.

Spring Boot auto-configures them.

---

### Which endpoint is the entry point?

```
/actuator
```

It provides links to exposed endpoints.

---

### Are all endpoints exposed by default?

No.

Only selected endpoints (such as `health`) are exposed by default. Others must be explicitly exposed through configuration.

---

### Does adding Actuator affect business APIs?

No.

It adds separate monitoring and management endpoints.

---

# 17. Complete Startup Flow

```text
Application Starts

↓

Read Dependencies

↓

Detect Actuator Starter

↓

Load Auto Configuration

↓

Create Endpoint Beans

↓

Register HTTP Endpoints

↓

Application Ready

↓

GET /actuator

↓

Discover Available Endpoints
```

---

# 18. Real Enterprise Flow

Imagine a production deployment.

```text
Spring Boot Application

↓

Actuator Starts

↓

Health Endpoint Registered

↓

Monitoring Tool Calls

↓

Application Healthy?

↓

YES

↓

Keep Running

↓

NO

↓

Raise Alert
```

This is the foundation of enterprise monitoring.

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.1 Introduction
├── ✅ 13.2 Adding Actuator ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.3 Actuator Endpoints Overview ⭐⭐⭐⭐⭐
       ↓
       Endpoint Categories
       ↓
       Read vs Write Endpoints
       ↓
       Web vs JMX Exposure
       ↓
       Endpoint IDs
       ↓
       Internal Endpoint Architecture
```

## Next Chapter (13.3)

We'll explore **every built-in Actuator endpoint**, understand what each one does, how endpoints are categorized (health, metrics, info, env, beans, mappings, loggers, etc.), which are enabled by default, and how Spring internally discovers and exposes them. This chapter forms the basis for all remaining Actuator topics.
