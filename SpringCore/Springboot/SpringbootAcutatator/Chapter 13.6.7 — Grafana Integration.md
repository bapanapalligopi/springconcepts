# Chapter 13.6.7 — Grafana Integration ⭐⭐⭐⭐⭐

# Complete Enterprise Dashboard Implementation

At this point, our monitoring pipeline looks like this:

```text
Spring Boot
     │
     ▼
Micrometer
     │
     ▼
Prometheus
     │
     ▼
Stores Time-Series Metrics
```

Now we need a way to **visualize** those metrics.

That is where **Grafana** comes in.

Grafana transforms thousands of numerical metrics into **beautiful dashboards, graphs, gauges, heatmaps, and alerts**.

This is one of the most widely used monitoring tools in the industry.

---

# Chapter Roadmap

```text
Grafana
│
├── 1. What is Grafana?
├── 2. Why Grafana?
├── 3. Architecture
├── 4. Installation
├── 5. Connect Prometheus
├── 6. Dashboards
├── 7. Panels
├── 8. Variables
├── 9. Alerts
├── 10. Enterprise Dashboard
├── 11. Best Practices
└── Interview Questions
```

---

# 1. What is Grafana?

Grafana is a **visualization platform**.

It **does not collect metrics**.

Instead, it reads metrics from data sources such as:

* Prometheus
* InfluxDB
* Elasticsearch
* Loki
* MySQL
* PostgreSQL

For Spring Boot, the most common data source is:

```text
Prometheus
```

---

# 2. Why Do We Need Grafana?

Suppose Prometheus stores:

```text
CPU

23

26

41

55

60

67

72
```

Reading numbers is difficult.

Grafana converts them into:

```text
CPU Usage

100% |                        *
 80% |                     *
 60% |                 *
 40% |             *
 20% |         *
  0% +---------------------------->
```

Visualization makes trends easy to understand.

---

# 3. Complete Architecture

```text
Users
   │
   ▼
Spring Boot

   │

Micrometer

   │

/actuator/prometheus

   │

Prometheus

   │

Time-Series Database

   │

Grafana

   │

Dashboards

   │

Operations Team
```

---

# 4. Install Grafana

Using Docker:

```bash
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  grafana/grafana
```

Open:

```text
http://localhost:3000
```

Default credentials:

```text
Username

admin

Password

admin
```

Grafana prompts you to change the password after the first login.

---

# 5. Connect Prometheus

Navigate:

```text
Connections

↓

Data Sources

↓

Add Data Source

↓

Prometheus
```

Enter:

```text
URL

↓

http://localhost:9090
```

Click:

```text
Save & Test
```

If successful:

```text
Prometheus Connected
```

---

# 6. Dashboard

A **dashboard** contains multiple panels.

Example:

```text
Employee API Dashboard

---------------------------------------

CPU Usage

Memory

HTTP Requests

Response Time

Database Connections

GC Pause

Thread Count

Error Rate

---------------------------------------
```

One dashboard gives a complete health overview.

---

# 7. Panels

Each graph is called a **Panel**.

Example:

```text
Dashboard

│

├── CPU Panel

├── Memory Panel

├── HTTP Panel

├── Database Panel

└── JVM Panel
```

Every panel executes a PromQL query.

---

# 8. CPU Panel

PromQL:

```promql
process_cpu_usage
```

Graph:

```text
CPU %

100 |

80 |

60 |      *

40 |   *

20 | *

0 +------------------
```

Operations teams immediately see CPU spikes.

---

# 9. Memory Panel

PromQL:

```promql
jvm_memory_used_bytes
```

Dashboard:

```text
Heap Memory

2 GB

████████░░

78%
```

---

# 10. HTTP Requests Panel

PromQL:

```promql
rate(http_server_requests_seconds_count[1m])
```

Dashboard:

```text
Requests/sec

450

470

510

495

530
```

Useful for traffic monitoring.

---

# 11. Response Time Panel

PromQL:

```promql
rate(http_server_requests_seconds_sum[1m])

/

rate(http_server_requests_seconds_count[1m])
```

Dashboard:

```text
Average Response Time

120 ms
```

This helps detect slow APIs.

---

# 12. Database Panel

PromQL:

```promql
hikaricp_connections_active
```

Dashboard:

```text
Connections

20 Max

8 Active

12 Idle
```

---

# 13. Error Rate Panel

PromQL:

```promql
rate(http_server_requests_seconds_count{
status=~"5.."
}[1m])
```

Dashboard:

```text
500 Errors

↓

12/min
```

If this suddenly increases:

Operations investigate immediately.

---

# 14. Dashboard Variables

Suppose there are three applications.

```text
Employee API

Order API

Payment API
```

Instead of creating three dashboards:

Use a variable.

```text
Application

↓

Employee API ▼
```

Now one dashboard works for all applications.

---

# 15. Alert Rules

Grafana can trigger alerts.

Example:

```text
CPU

>

90%
```

For

```text
5 Minutes
```

Then:

```text
Alert
```

---

Example:

```text
CPU

30%

↓

45%

↓

62%

↓

91%

↓

95%

↓

ALERT
```

---

# 16. Alert Flow

```text
Grafana

↓

Alert Rule

↓

Email

Slack

Microsoft Teams

PagerDuty
```

Operations teams receive notifications automatically.

---

# 17. Enterprise Dashboard

Typical production dashboard:

```text
--------------------------------------------------

Application Health

CPU                 35%

Memory              62%

Heap                1.2 GB

Threads             48

GC Pause            10 ms

Requests/sec        420

P95                 180 ms

Database Active     9

Database Idle       11

500 Errors          0

Uptime              15 Days

--------------------------------------------------
```

This gives a real-time operational view of the application.

---

# 18. Enterprise Monitoring Flow

```text
Spring Boot

↓

Micrometer

↓

Prometheus

↓

Store Metrics

↓

Grafana

↓

Dashboard

↓

Alert

↓

Operations Team
```

---

# 19. Common Mistakes

### Mistake 1

Thinking Grafana stores metrics.

It doesn't.

Prometheus stores the metrics.

Grafana visualizes them.

---

### Mistake 2

Creating too many dashboards.

Better:

```text
Few Dashboards

↓

Many Panels
```

Instead of:

```text
100 Dashboards
```

---

### Mistake 3

Ignoring alerts.

Dashboards are useful only if someone is actively watching them.

Critical metrics should trigger automatic alerts.

---

# 20. Best Practices

```text
✅ Use Prometheus as data source

✅ Group related panels

✅ Monitor JVM

✅ Monitor HTTP latency

✅ Monitor DB pool

✅ Monitor Error Rate

✅ Create Alerts

❌ Don't create unnecessary panels

❌ Don't ignore P95 latency
```

---

# 21. Interview Questions

### What is Grafana?

A visualization platform used to display metrics from monitoring systems such as Prometheus.

---

### Does Grafana store metrics?

No.

Prometheus stores metrics.

Grafana queries and visualizes them.

---

### What is a Panel?

A single visualization (graph, gauge, table, heatmap, etc.) on a dashboard.

---

### What is a Dashboard?

A collection of related panels used to monitor an application or system.

---

### Can Grafana generate alerts?

Yes.

It can notify users via Email, Slack, Microsoft Teams, PagerDuty, and other integrations.

---

### Which query language is used?

PromQL (executed against Prometheus).

---

# 22. Complete Production Architecture

```text
                    Users
                      │
                      ▼
               Spring Boot API
                      │
                Micrometer
                      │
         /actuator/prometheus
                      │
              Prometheus Server
                      │
         Time-Series Database
                      │
               Grafana Server
                      │
      ┌───────────────┴────────────────┐
      ▼                                ▼
 Dashboards                      Alert Rules
      │                                │
      └──────────────┬─────────────────┘
                     ▼
              Operations Team
```

---

# 23. Real Production Workflow

Imagine CPU usage spikes because of heavy traffic.

```text
Users Increase

↓

HTTP Requests Increase

↓

CPU Usage = 96%

↓

Prometheus Scrapes

↓

Grafana Updates Dashboard

↓

Alert Rule Matches

↓

Slack Notification

↓

DevOps Engineer Investigates

↓

Issue Resolved
```

No one had to manually log in and inspect the server.

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ Health Endpoint
├── ✅ Info Endpoint
├── ✅ Metrics
├── ✅ JVM Metrics
├── ✅ HTTP Metrics
├── ✅ HikariCP Metrics
├── ✅ Custom Metrics
├── ✅ Prometheus
├── ✅ Grafana ⭐⭐⭐⭐⭐
│
└── ⏭️ Chapter 13.7 — Logging & Loggers Endpoint ⭐⭐⭐⭐
       ↓
       Logging Levels
       ↓
       Runtime Log Level Changes
       ↓
       /actuator/loggers
       ↓
       Logger Hierarchy
       ↓
       Production Debugging
```

# 🎉 Metrics Module Completed

You have now completed one of the most valuable Spring Boot Actuator modules:

* ✅ Micrometer fundamentals
* ✅ JVM metrics
* ✅ HTTP metrics
* ✅ Database (HikariCP) metrics
* ✅ Custom metrics
* ✅ Prometheus integration
* ✅ Grafana dashboards

These concepts are widely used in enterprise Spring Boot applications and are common topics in senior developer interviews.

## Next Chapter (13.7): Logging & Loggers Endpoint

We'll learn:

* How Spring Boot logging works
* Logger hierarchy
* Log levels (`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`)
* The `/actuator/loggers` endpoint
* Changing log levels at runtime
* Production debugging techniques
* Logging best practices
* Enterprise logging architecture with centralized log management
