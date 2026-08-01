# Chapter 13.6.3 — HTTP Request Metrics ⭐⭐⭐⭐⭐

This is one of the **most important metrics** in any Spring Boot application.

If someone asks:

> **"How fast is your API?"**

or

> **"Which endpoint is slow?"**

or

> **"How many requests are failing?"**

The answers come from **HTTP Request Metrics**.

In enterprise applications, these metrics are continuously collected by **Prometheus**, visualized in **Grafana**, and used to trigger alerts.

---

# Chapter Roadmap

```text
HTTP Request Metrics
│
├── 1. What are HTTP Metrics?
├── 2. http.server.requests
├── 3. Internal Working
├── 4. Request Lifecycle
├── 5. Important Tags
├── 6. Response Time
├── 7. Status Code Metrics
├── 8. Percentiles (P95, P99)
├── 9. Enterprise Dashboard
├── 10. Best Practices
└── Interview Questions
```

---

# 1. What are HTTP Request Metrics?

Whenever a client sends a request:

```http
GET /employees
```

Spring Boot records information like:

* How many requests?
* How long did they take?
* Did they succeed?
* Which endpoint?
* Which HTTP method?
* Which status code?

These measurements are automatically collected by **Micrometer**.

---

# 2. The `http.server.requests` Metric

The most important HTTP metric is:

```text
http.server.requests
```

It stores timing and metadata for every request.

Example:

```text
GET /employees

↓

Processing Time

↓

180 ms

↓

Status = 200

↓

Metric Recorded
```

---

# 3. Complete Request Flow

```text
Client

↓

GET /employees

↓

Tomcat

↓

Spring Security Filter Chain

↓

DispatcherServlet

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Response

↓

Micrometer Records Metrics

↓

Meter Registry

↓

Prometheus

↓

Grafana
```

Notice:

Metrics are recorded **after the request completes**, so the total processing time is known.

---

# 4. Internal Working

Suppose this controller exists:

```java
@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @GetMapping
    public List<Employee> getEmployees() {

        return service.findAll();
    }
}
```

Flow:

```text
Request Starts

↓

Timer Starts

↓

Controller Executes

↓

Service Executes

↓

Database Query

↓

Response Returned

↓

Timer Stops

↓

Metric Saved
```

---

# 5. Metric Example

Request:

```http
GET /actuator/metrics/http.server.requests
```

Example response (simplified):

```json
{
  "name": "http.server.requests",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 1520
    }
  ]
}
```

This means:

```text
Total Requests

↓

1520
```

---

# 6. Metric Tags

Micrometer doesn't only count requests.

It also stores **tags**.

Example:

| Tag       | Example    |
| --------- | ---------- |
| method    | GET        |
| uri       | /employees |
| status    | 200        |
| outcome   | SUCCESS    |
| exception | None       |

Example:

```text
http.server.requests

↓

method=GET

↓

uri=/employees

↓

status=200

↓

time=120ms
```

Tags allow you to filter metrics.

---

# 7. Why Tags Matter

Suppose your application has:

```text
GET /employees

POST /employees

DELETE /employees/{id}
```

Instead of one combined metric:

```text
Requests = 10,000
```

Micrometer stores:

```text
GET

↓

6000

POST

↓

3000

DELETE

↓

1000
```

Now you know which APIs are heavily used.

---

# 8. Response Time (Latency)

The most important measurement is **latency**.

Example:

```text
Request Starts

↓

120 ms

↓

Response Returned
```

Metric:

```text
Response Time = 120 ms
```

Why important?

Fast API:

```text
80 ms
```

Slow API:

```text
3200 ms
```

This immediately tells you which endpoints need optimization.

---

# 9. Status Code Metrics

Micrometer records HTTP status codes.

Example:

```text
GET /employees

↓

200 OK
```

Another request:

```text
GET /employees/999

↓

404
```

Another:

```text
POST /employees

↓

500
```

Dashboard:

```text
200 → 95%

404 → 3%

500 → 2%
```

A sudden increase in **500 errors** is a strong signal that something is wrong.

---

# 10. Outcomes

Micrometer groups status codes into outcomes.

| Status | Outcome      |
| ------ | ------------ |
| 2xx    | SUCCESS      |
| 3xx    | REDIRECTION  |
| 4xx    | CLIENT_ERROR |
| 5xx    | SERVER_ERROR |

Example:

```text
Request

↓

Status = 500

↓

Outcome

↓

SERVER_ERROR
```

---

# 11. Request Duration Statistics

Micrometer collects more than the average time.

It can calculate:

```text
Minimum

Average

Maximum

Count

Total Time
```

Example:

```text
Requests

↓

100

↓

Average = 95 ms

↓

Max = 480 ms

↓

Min = 30 ms
```

---

# 12. Percentiles (P95, P99)

Average response time can be misleading.

Example:

```text
99 Requests

↓

100 ms

1 Request

↓

5 seconds
```

Average:

```text
149 ms
```

Looks acceptable.

But one user waited **5 seconds**.

Percentiles solve this.

---

## P95

95% of requests finished within this time.

Example:

```text
P95 = 220 ms
```

Meaning:

```text
95%

↓

≤ 220 ms
```

---

## P99

```text
P99 = 700 ms
```

Meaning:

```text
99%

↓

≤ 700 ms
```

Enterprise systems monitor **P95** and **P99** much more than averages.

---

# 13. Enterprise Dashboard

Example Grafana panel:

```text
API Performance Dashboard

Requests/sec       120

Average Time       90 ms

P95                180 ms

P99                420 ms

Error Rate         0.4%

500 Errors         12

404 Errors         38
```

This provides a much clearer picture than simply checking if the application is "UP".

---

# 14. Slow Endpoint Detection

Suppose:

```text
GET /employees

↓

80 ms
```

```text
POST /employees

↓

150 ms
```

```text
GET /reports

↓

5 sec
```

Dashboard:

```text
Fast APIs

↓

Employees

Slow API

↓

Reports
```

Now developers know where to focus optimization efforts.

---

# 15. Enterprise Monitoring Flow

```text
User Requests

↓

Spring Boot

↓

Micrometer Timer

↓

Record Duration

↓

Store Tags

↓

Meter Registry

↓

Prometheus

↓

Grafana Dashboard

↓

Alert Manager
```

---

# 16. Common Mistakes

### Mistake 1

Looking only at average response time.

Always monitor:

* P95
* P99
* Maximum latency

---

### Mistake 2

Ignoring HTTP 500 metrics.

A healthy application should have **very few** server errors.

---

### Mistake 3

Ignoring slow endpoints because overall CPU usage looks normal.

Latency problems are not always caused by CPU.

They could be due to:

* Slow database queries
* External APIs
* Network delays
* Thread pool exhaustion

---

# 17. Best Practices

```text
✅ Monitor response time

✅ Monitor error rates

✅ Watch P95 and P99 latency

✅ Monitor request count

✅ Monitor status codes

✅ Track endpoint-specific metrics

❌ Don't rely only on averages

❌ Don't ignore increasing latency
```

---

# 18. Interview Questions

### Which metric records HTTP requests?

```text
http.server.requests
```

---

### What information does it contain?

* Request count
* Duration
* HTTP method
* URI
* Status code
* Outcome
* Exception (if any)

---

### What is latency?

The total time taken to process a request and return a response.

---

### What is P95?

95% of requests completed within that response time.

---

### Why are tags important?

They let you filter and analyze metrics by:

* Endpoint
* Method
* Status
* Outcome
* Exception

---

# 19. Complete Internal Flow

```text
HTTP Request

↓

Micrometer Timer Starts

↓

Spring MVC

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Response Generated

↓

Timer Stops

↓

Tags Added

↓

Metric Stored

↓

Prometheus Scrapes Metrics

↓

Grafana Dashboard
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.6.1 Metrics & Micrometer
├── ✅ 13.6.2 JVM Metrics
├── ✅ 13.6.3 HTTP Request Metrics ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.6.4 Database Connection Pool Metrics ⭐⭐⭐⭐⭐
       ↓
       HikariCP Metrics
       ↓
       Active Connections
       ↓
       Idle Connections
       ↓
       Pending Connections
       ↓
       Pool Exhaustion
       ↓
       Enterprise Database Monitoring
```

## Next Chapter (13.6.4) — Database Connection Pool Metrics

We'll dive into **HikariCP**, the default connection pool in Spring Boot, and learn:

* How connection pools work
* Why pooling is essential
* Active, idle, and pending connections
* Pool exhaustion
* Connection leaks
* HikariCP metrics exposed through Micrometer
* Real production tuning
* Enterprise monitoring dashboards

This is another **high-frequency interview topic** because database performance is often the bottleneck in enterprise applications.
