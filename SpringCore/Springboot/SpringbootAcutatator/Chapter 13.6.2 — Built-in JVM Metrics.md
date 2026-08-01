# Chapter 13.6.2 — Built-in JVM Metrics ⭐⭐⭐⭐⭐

In the previous lesson, we learned that **Micrometer automatically collects metrics**.

Now we'll study **what metrics Spring Boot collects from the JVM**.

These are among the **most frequently monitored metrics** in production.

---

# Chapter Roadmap

```text
Built-in JVM Metrics
│
├── 1. Heap Memory
├── 2. Non-Heap Memory
├── 3. Garbage Collection
├── 4. Thread Metrics
├── 5. Class Loading
├── 6. CPU Metrics
├── 7. Process Metrics
├── 8. Common JVM Metrics
├── 9. Enterprise Monitoring
├── 10. Best Practices
└── Interview Questions
```

---

# 1. JVM Metrics Overview

When Spring Boot starts, Micrometer automatically registers JVM-related metrics.

```text
Spring Boot

↓

Micrometer

↓

JVM Metrics

├── Memory
├── GC
├── Threads
├── Classes
├── CPU
└── Process
```

You don't need to write any code for these metrics.

---

# 2. Heap Memory

Heap Memory stores Java objects created during program execution.

Example:

```java
Employee emp = new Employee();
List<Employee> employees = new ArrayList<>();
```

Both objects are stored in the **Heap**.

Flow:

```text
Object Created

↓

Heap Memory

↓

Used by Application

↓

Garbage Collector Cleans Unused Objects
```

---

## Heap Metrics

Important metric names:

```text
jvm.memory.used

jvm.memory.max

jvm.memory.committed
```

Example request:

```http
GET /actuator/metrics/jvm.memory.used
```

Example response:

```json
{
  "name": "jvm.memory.used",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 152345600
    }
  ]
}
```

The value is in **bytes**.

---

# 3. Understanding Heap Metrics

Suppose JVM has:

```text
Maximum Heap

↓

2 GB
```

Current usage:

```text
700 MB
```

Committed:

```text
1 GB
```

Diagram:

```text
Maximum Heap (2 GB)

┌───────────────────────────┐
│                           │
│  Committed (1 GB)         │
│  ┌─────────────────────┐  │
│  │ Used (700 MB)       │  │
│  └─────────────────────┘  │
│                           │
└───────────────────────────┘
```

Definitions:

| Metric    | Meaning                   |
| --------- | ------------------------- |
| Used      | Memory currently occupied |
| Committed | Memory reserved by JVM    |
| Max       | Maximum allowed heap      |

---

# 4. Non-Heap Memory

Not everything is stored in the heap.

The JVM also uses **Non-Heap Memory**.

It stores things like:

```text
Class Metadata

↓

JIT Compiler Data

↓

Code Cache

↓

Metaspace
```

Metric:

```http
GET /actuator/metrics/jvm.memory.used
```

Filter by tag:

```text
area=nonheap
```

---

# 5. Garbage Collection Metrics

Garbage Collector removes unused objects.

Flow:

```text
Application Creates Objects

↓

Objects Become Unused

↓

Garbage Collector Runs

↓

Memory Reclaimed
```

Micrometer exposes metrics such as:

```text
jvm.gc.pause

jvm.gc.memory.allocated

jvm.gc.memory.promoted
```

These help identify GC-related performance issues.

---

# 6. GC Pause Time

When GC runs, application threads may pause briefly.

```text
Application Running

↓

GC Starts

↓

Pause

↓

GC Ends

↓

Application Continues
```

Metric:

```http
GET /actuator/metrics/jvm.gc.pause
```

High pause times may indicate memory pressure.

---

# 7. Thread Metrics

Every HTTP request is processed by a thread.

Example:

```text
HTTP Request

↓

Worker Thread

↓

Controller

↓

Service

↓

Repository

↓

Response
```

Metrics:

```text
jvm.threads.live

jvm.threads.daemon

jvm.threads.peak
```

---

## Example

```http
GET /actuator/metrics/jvm.threads.live
```

Response:

```json
{
  "measurements": [
    {
      "value": 38
    }
  ]
}
```

Meaning:

Currently **38 live threads** exist in the JVM.

---

# 8. Class Loading Metrics

Whenever a class is first used:

```text
Employee.class

↓

Loaded

↓

JVM
```

Metrics:

```text
jvm.classes.loaded

jvm.classes.unloaded
```

Useful for diagnosing class loader issues.

---

# 9. CPU Metrics

Micrometer also exposes CPU information.

Examples:

```text
system.cpu.usage

process.cpu.usage
```

Difference:

| Metric            | Meaning                               |
| ----------------- | ------------------------------------- |
| system.cpu.usage  | CPU usage of the entire machine       |
| process.cpu.usage | CPU usage of your Spring Boot process |

Example:

```http
GET /actuator/metrics/process.cpu.usage
```

---

# 10. Process Metrics

Process metrics describe the running JVM process.

Examples:

```text
process.uptime

process.start.time

process.files.open

process.files.max
```

Example:

```http
GET /actuator/metrics/process.uptime
```

This tells how long the application has been running.

---

# 11. Enterprise Monitoring Dashboard

```text
                  Grafana Dashboard

Heap Usage        ████████░░ 78%

CPU Usage         ██████░░░░ 55%

GC Pause          120 ms

Live Threads      62

HTTP Requests     450/sec

Uptime            12 days
```

Operations teams monitor these dashboards continuously.

---

# 12. JVM Metrics Flow

```text
Spring Boot

↓

Micrometer

↓

Read JVM

↓

Memory

↓

Threads

↓

GC

↓

CPU

↓

Meter Registry

↓

/actuator/metrics

↓

Prometheus

↓

Grafana
```

---

# 13. Common JVM Metrics

| Metric                    | Description           |
| ------------------------- | --------------------- |
| `jvm.memory.used`         | Used memory           |
| `jvm.memory.max`          | Maximum heap          |
| `jvm.memory.committed`    | Reserved memory       |
| `jvm.gc.pause`            | GC pause duration     |
| `jvm.gc.memory.allocated` | Allocated memory      |
| `jvm.threads.live`        | Live thread count     |
| `jvm.threads.peak`        | Peak thread count     |
| `jvm.classes.loaded`      | Loaded classes        |
| `process.cpu.usage`       | JVM process CPU usage |
| `system.cpu.usage`        | System CPU usage      |
| `process.uptime`          | Application uptime    |

These are the metrics you'll see most often.

---

# 14. Common Mistakes

### Mistake 1

Confusing:

```text
system.cpu.usage
```

with

```text
process.cpu.usage
```

System CPU measures the whole machine.

Process CPU measures only your application.

---

### Mistake 2

Treating memory values as MB.

Most memory metrics are reported in **bytes**.

Convert before interpreting.

---

### Mistake 3

Ignoring GC metrics.

Frequent or long GC pauses may explain slow response times.

---

# 15. Best Practices

```text
✅ Monitor heap usage

✅ Watch GC pause times

✅ Monitor live threads

✅ Track process uptime

✅ Alert on high CPU usage

❌ Don't rely on a single metric

❌ Always correlate CPU, memory, and GC metrics
```

---

# 16. Interview Questions

### Does Spring Boot expose JVM metrics automatically?

Yes.

Micrometer automatically registers common JVM metrics.

---

### Which metric shows heap usage?

```text
jvm.memory.used
```

---

### Which metric shows live thread count?

```text
jvm.threads.live
```

---

### Which metric shows process uptime?

```text
process.uptime
```

---

### Difference between `system.cpu.usage` and `process.cpu.usage`?

* `system.cpu.usage` → CPU usage of the entire machine.
* `process.cpu.usage` → CPU usage of the Spring Boot JVM process.

---

# 17. Complete Internal Flow

```text
JVM Running

↓

Objects Created

↓

Threads Running

↓

GC Executed

↓

CPU Used

↓

Micrometer Collects Metrics

↓

Meter Registry

↓

/actuator/metrics

↓

Prometheus

↓

Grafana Dashboard
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.1 Introduction
├── ✅ 13.2 Adding Actuator
├── ✅ 13.3 Endpoints
├── ✅ 13.4 Health Endpoint
├── ✅ 13.5 Info Endpoint
├── ✅ 13.6.1 Metrics & Micrometer
├── ✅ 13.6.2 Built-in JVM Metrics ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.6.3 HTTP Request Metrics ⭐⭐⭐⭐⭐
       ↓
       http.server.requests
       ↓
       Request Count
       ↓
       Response Time
       ↓
       Status Codes
       ↓
       URI Tags
       ↓
       Performance Analysis
```

## Next Chapter (13.6.3) — HTTP Request Metrics

We'll explore one of the most valuable metrics in production:

* `http.server.requests`
* Request count
* Response time (latency)
* Success vs error rates
* Status code analysis (200, 404, 500)
* URI and method tags
* Percentiles (P95, P99)
* SLA/SLO monitoring
* Enterprise performance dashboards

This is a core topic for diagnosing API performance issues in Spring Boot applications.
