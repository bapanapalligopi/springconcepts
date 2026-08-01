# Chapter 13.6.6 — Prometheus Integration ⭐⭐⭐⭐⭐

# Complete Enterprise Implementation

Up to this point, we've learned how Spring Boot collects metrics using **Micrometer**.

But those metrics are still **inside the application**.

Now the question becomes:

> **How does a monitoring server collect those metrics?**

The answer is:

> **Prometheus**

This chapter covers the **complete production setup** used in modern Spring Boot microservices.

---

# Chapter Roadmap

```text
Prometheus Integration
│
├── 1. What is Prometheus?
├── 2. Why Prometheus?
├── 3. Prometheus Architecture
├── 4. Micrometer Prometheus Registry
├── 5. Add Dependency
├── 6. Configure Actuator
├── 7. /actuator/prometheus
├── 8. Prometheus Scraping
├── 9. prometheus.yml
├── 10. Docker Setup
├── 11. PromQL Basics
├── 12. Enterprise Architecture
├── 13. Best Practices
└── Interview Questions
```

---

# 1. What is Prometheus?

**Prometheus** is an **open-source monitoring system**.

It continuously collects metrics from applications.

Instead of asking:

> "How is my application performing right now?"

Prometheus stores:

* Current metrics
* Historical metrics
* Trends
* Time-series data

---

## Example

```text
10:00 AM

CPU = 25%

↓

10:10 AM

CPU = 40%

↓

10:20 AM

CPU = 60%

↓

10:30 AM

CPU = 95%
```

Now you can analyze performance over time.

---

# 2. Why Prometheus?

Without Prometheus:

```text
Developer

↓

GET /actuator/metrics

↓

Manual Monitoring
```

Impossible in production.

With Prometheus:

```text
Spring Boot

↓

Prometheus

↓

Stores Metrics

↓

Grafana Dashboard

↓

Alerts
```

Everything becomes automatic.

---

# 3. Enterprise Architecture

```text
                   Users
                      │
                      ▼
              Spring Boot App
                      │
             /actuator/prometheus
                      │
          (HTTP Metrics Endpoint)
                      │
                      ▼
               Prometheus Server
                      │
          Stores Time-Series Data
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
    Grafana                    Alertmanager
        │                           │
   Dashboards                  Email/Slack Alerts
```

---

# 4. How Prometheus Works

Prometheus uses a **Pull Model**.

It periodically requests metrics from your application.

```text
Every 15 Seconds

↓

Prometheus

↓

GET /actuator/prometheus

↓

Spring Boot

↓

Return Metrics

↓

Store Metrics
```

Notice:

Spring Boot **does not push metrics**.

Prometheus **pulls** them.

---

# 5. Add Dependency

### Maven

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

This adds the Prometheus registry.

Without it:

```text
/actuator/prometheus

↓

404 Not Found
```

---

# 6. Configure Actuator

Expose the endpoint.

```properties
management.endpoints.web.exposure.include=health,info,prometheus
```

Now Spring Boot exposes:

```text
/actuator/prometheus
```

---

# 7. Prometheus Endpoint

Open:

```http
GET http://localhost:8080/actuator/prometheus
```

Response:

```text
# HELP jvm_memory_used_bytes
# TYPE jvm_memory_used_bytes gauge

jvm_memory_used_bytes{area="heap"} 7.48E7

process_cpu_usage 0.18

system_cpu_usage 0.43

http_server_requests_seconds_count 523

hikaricp_connections_active 5
```

Notice:

This is **not JSON**.

Prometheus expects a special **text exposition format**.

---

# 8. Internal Flow

```text
Spring Boot

↓

Micrometer

↓

Meter Registry

↓

Prometheus Registry

↓

Convert Metrics

↓

Text Format

↓

/actuator/prometheus
```

Micrometer converts internal meters into the Prometheus format.

---

# 9. Install Prometheus (Docker)

```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

Prometheus UI:

```text
http://localhost:9090
```

---

# 10. Configure `prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "employee-api"

    metrics_path: "/actuator/prometheus"

    static_configs:
      - targets:
          - "localhost:8080"
```

Explanation:

| Property        | Meaning                           |
| --------------- | --------------------------------- |
| scrape_interval | Collect metrics every 15 seconds  |
| job_name        | Logical name of the application   |
| metrics_path    | Endpoint to scrape                |
| targets         | Spring Boot application addresses |

---

# 11. Scraping Process

```text
15 Seconds

↓

Prometheus

↓

localhost:8080

↓

GET /actuator/prometheus

↓

Receive Metrics

↓

Store Database

↓

Repeat Forever
```

This happens automatically.

---

# 12. Time-Series Database

Prometheus stores every metric with:

```text
Metric Name

+

Timestamp

+

Value
```

Example:

| Time  | CPU |
| ----- | --- |
| 10:00 | 22% |
| 10:01 | 26% |
| 10:02 | 31% |
| 10:03 | 44% |

Unlike Actuator, Prometheus remembers history.

---

# 13. PromQL Basics

PromQL = Prometheus Query Language.

Examples:

### Current JVM Memory

```promql
jvm_memory_used_bytes
```

---

### CPU Usage

```promql
process_cpu_usage
```

---

### HTTP Requests

```promql
http_server_requests_seconds_count
```

---

### Requests Per Second

```promql
rate(http_server_requests_seconds_count[1m])
```

Meaning:

Average requests per second during the last minute.

---

# 14. Enterprise Monitoring Flow

```text
HTTP Request

↓

Spring Boot

↓

Micrometer

↓

Meter Registry

↓

/actuator/prometheus

↓

Prometheus Scrapes

↓

Stores History

↓

Grafana Dashboard

↓

Alertmanager

↓

Email / Slack / PagerDuty
```

---

# 15. Production Example

Suppose CPU suddenly increases.

```text
CPU

↓

30%

↓

45%

↓

70%

↓

92%

↓

98%
```

Prometheus detects:

```text
CPU > 90%

↓

Alert Rule

↓

Alertmanager

↓

Slack

↓

Operations Team
```

No one has to manually check dashboards.

---

# 16. Common Mistakes

### Mistake 1

Adding Actuator but forgetting:

```xml
micrometer-registry-prometheus
```

Result:

```text
/actuator/prometheus

↓

404
```

---

### Mistake 2

Not exposing the endpoint.

```properties
management.endpoints.web.exposure.include=*
```

or explicitly include `prometheus`.

---

### Mistake 3

Thinking Prometheus pushes metrics.

Wrong.

Prometheus **pulls** metrics.

---

### Mistake 4

Using `/actuator/metrics` instead of `/actuator/prometheus`.

| Endpoint               | Purpose                           |
| ---------------------- | --------------------------------- |
| `/actuator/metrics`    | Human/API exploration (JSON)      |
| `/actuator/prometheus` | Prometheus scraping (text format) |

---

# 17. Best Practices

```text
✅ Use Prometheus Registry

✅ Secure Actuator endpoints

✅ Scrape every 15–30 seconds

✅ Use meaningful job names

✅ Monitor JVM, HTTP, DB, and custom metrics

✅ Keep historical data

❌ Don't expose Actuator publicly

❌ Don't scrape too frequently without a need
```

---

# 18. Interview Questions

### What is Prometheus?

An open-source monitoring system and time-series database.

---

### Does Prometheus use Push or Pull?

**Pull**

Prometheus periodically scrapes application endpoints.

---

### Which dependency enables Prometheus?

```xml
io.micrometer:micrometer-registry-prometheus
```

---

### Which endpoint does Prometheus scrape?

```text
/actuator/prometheus
```

---

### What language does Prometheus use?

**PromQL**

---

### Difference between `/actuator/metrics` and `/actuator/prometheus`?

| `/actuator/metrics` | `/actuator/prometheus` |
| ------------------- | ---------------------- |
| JSON                | Prometheus text format |
| Human-readable      | Scrape format          |
| Explore metrics     | Monitoring systems     |

---

# 19. Complete Enterprise Architecture

```text
                 Users
                   │
                   ▼
            Spring Boot API
                   │
            Micrometer Metrics
                   │
         /actuator/prometheus
                   │
          Prometheus Scrapes
                   │
      Time-Series Database
                   │
      ┌────────────┴────────────┐
      ▼                         ▼
   Grafana                 Alertmanager
      │                         │
 Dashboards          Slack / Email / SMS
      │                         │
      └────────── Operations Team
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ Health Endpoint
├── ✅ Info Endpoint
├── ✅ Metrics Endpoint
├── ✅ JVM Metrics
├── ✅ HTTP Metrics
├── ✅ HikariCP Metrics
├── ✅ Custom Metrics
├── ✅ Prometheus Integration ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.6.7 Grafana Integration ⭐⭐⭐⭐⭐
       ↓
       Dashboards
       ↓
       Panels
       ↓
       Variables
       ↓
       Alerts
       ↓
       Enterprise Monitoring
```

# Next Chapter (13.6.7) — Grafana Integration

We'll complete the observability stack by learning:

* Installing Grafana
* Connecting Grafana to Prometheus
* Building dashboards
* Creating panels for JVM, CPU, memory, HTTP, and HikariCP metrics
* Dashboard variables
* Alert rules
* Enterprise dashboard design
* Real production monitoring examples

At the end of the next chapter, you'll understand the full **Spring Boot → Micrometer → Prometheus → Grafana** monitoring pipeline used in production systems.
