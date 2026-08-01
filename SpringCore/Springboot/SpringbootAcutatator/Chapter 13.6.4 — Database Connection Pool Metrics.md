# Chapter 13.6.4 — Database Connection Pool Metrics (HikariCP) ⭐⭐⭐⭐⭐

One of the **most critical performance components** in any Spring Boot application is the **database connection pool**.

Many production outages are **not caused by slow SQL queries**, but because **all database connections are already in use**.

Spring Boot uses **HikariCP** as its default connection pool, and Micrometer automatically exposes its metrics.

This chapter is extremely important for **Spring Boot interviews**, **performance tuning**, and **production monitoring**.

---

# Chapter Roadmap

```text
Database Connection Pool Metrics
│
├── 1. Why Connection Pools?
├── 2. What is HikariCP?
├── 3. Internal Working
├── 4. Pool Lifecycle
├── 5. HikariCP Metrics
├── 6. Active vs Idle Connections
├── 7. Pending Connections
├── 8. Pool Exhaustion
├── 9. Connection Leaks
├── 10. Enterprise Monitoring
├── 11. Best Practices
└── Interview Questions
```

---

# 1. Why Do We Need a Connection Pool?

Imagine every request creates a new database connection.

```text
Client Request

↓

Create DB Connection

↓

Execute Query

↓

Close Connection
```

Creating a database connection is **expensive** because it involves:

* Network communication
* Authentication
* Database session creation
* Resource allocation

If 1000 users connect simultaneously, the database spends a lot of time just creating and closing connections.

---

## Solution: Connection Pool

Instead of creating connections repeatedly:

```text
Application

↓

Connection Pool

┌───────────────┐
│ Conn-1        │
│ Conn-2        │
│ Conn-3        │
│ Conn-4        │
│ Conn-5        │
└───────────────┘

↓

Database
```

Connections are **reused**, which greatly improves performance.

---

# 2. What is HikariCP?

**HikariCP** is Spring Boot's default connection pool.

It is known for:

* High performance
* Low latency
* Low memory usage
* Fast connection acquisition

Spring Boot automatically configures HikariCP when using `spring-boot-starter-data-jpa` or JDBC.

---

# 3. Internal Working

Suppose the pool size is **5**.

```text
Pool

Conn-1

Conn-2

Conn-3

Conn-4

Conn-5
```

A request arrives.

```text
Request

↓

Borrow Conn-2

↓

Execute SQL

↓

Return Conn-2

↓

Pool
```

The connection is **not closed**.

It is returned to the pool for reuse.

---

# 4. Request Lifecycle

```text
HTTP Request

↓

Controller

↓

Service

↓

Repository

↓

Borrow Connection

↓

Execute SQL

↓

Return Connection

↓

Response
```

Borrow → Use → Return

That's the lifecycle.

---

# 5. HikariCP Metrics

Micrometer automatically exposes metrics like:

```text
hikaricp.connections

hikaricp.connections.active

hikaricp.connections.idle

hikaricp.connections.pending

hikaricp.connections.max

hikaricp.connections.min
```

These metrics help identify connection pool issues.

---

# 6. Active Connections

An **active connection** is currently executing work.

Example:

Pool size = 10

```text
Connections

10 Total

↓

4 Active

↓

6 Idle
```

Metric:

```http
GET /actuator/metrics/hikaricp.connections.active
```

Example response:

```json
{
  "name": "hikaricp.connections.active",
  "measurements": [
    {
      "statistic": "VALUE",
      "value": 4
    }
  ]
}
```

Meaning:

4 database connections are currently in use.

---

# 7. Idle Connections

Idle connections are available for immediate use.

Example:

```text
Total = 10

↓

Active = 3

↓

Idle = 7
```

Metric:

```http
GET /actuator/metrics/hikaricp.connections.idle
```

Healthy pools usually have some idle connections ready to serve new requests.

---

# 8. Pending Connections

What happens if all connections are busy?

Example:

Pool:

```text
10 Connections

↓

All Busy
```

New request:

```text
Request

↓

Wait

↓

Pending Queue
```

Metric:

```http
GET /actuator/metrics/hikaricp.connections.pending
```

A consistently high pending count indicates the application cannot obtain database connections fast enough.

---

# 9. Pool Exhaustion

Suppose:

Pool size:

```text
10
```

Current usage:

```text
10 Active

0 Idle

15 Waiting
```

Diagram:

```text
Pool

[XXXXXXXXXX]

All Busy

↓

New Requests Waiting

↓

Timeout
```

Eventually, requests fail with exceptions like:

```text
SQLTransientConnectionException

or

Connection is not available
```

This condition is called **pool exhaustion**.

---

# 10. Connection Leaks

A connection leak occurs when the application borrows a connection but never returns it.

Example (incorrect):

```java
Connection connection = dataSource.getConnection();

// Execute SQL

// Forgot to close connection
```

Correct approach:

```java
try (Connection connection = dataSource.getConnection()) {

    // Execute SQL

}
```

With Spring Data JPA or `JdbcTemplate`, Spring usually manages connections automatically.

Leaks commonly occur in manually written JDBC code.

---

# 11. Maximum Pool Size

Configuration example:

```properties
spring.datasource.hikari.maximum-pool-size=20
```

Meaning:

```text
Maximum Connections

↓

20
```

Choose this value carefully.

Too low:

```text
Waiting Requests
```

Too high:

```text
Database Overloaded
```

The optimal value depends on:

* Database capacity
* CPU cores
* Workload
* Query execution time

---

# 12. Enterprise Monitoring Dashboard

Example Grafana panel:

```text
Database Connections

Maximum          20

Active           8

Idle             12

Pending          0

Pool Usage       40%
```

If pending starts increasing:

```text
Pending

0

↓

5

↓

12

↓

25

↓

ALERT
```

Operations teams investigate immediately.

---

# 13. Enterprise Flow

```text
User Request

↓

Repository

↓

HikariCP

↓

Borrow Connection

↓

Execute SQL

↓

Return Connection

↓

Micrometer

↓

Meter Registry

↓

Prometheus

↓

Grafana Dashboard
```

---

# 14. Common Problems

### Problem 1

Too many active connections.

Possible causes:

* Slow SQL queries
* Long-running transactions
* High traffic

---

### Problem 2

Pending connections increasing.

Possible causes:

* Pool size too small
* Database overloaded
* Connection leak

---

### Problem 3

No idle connections.

```text
Idle = 0
```

This usually indicates the pool is under heavy load.

---

# 15. Common Mistakes

### Mistake 1

Increasing pool size without checking the database server.

More connections are **not always better**.

---

### Mistake 2

Ignoring connection leaks.

Leaked connections eventually exhaust the pool.

---

### Mistake 3

Monitoring only active connections.

Always monitor together:

* Active
* Idle
* Pending
* Maximum

---

# 16. Best Practices

```text
✅ Use HikariCP (default)

✅ Monitor active connections

✅ Monitor idle connections

✅ Alert on pending connections

✅ Use proper transaction boundaries

✅ Always close JDBC resources

❌ Don't set extremely high pool sizes

❌ Don't ignore connection leak warnings
```

---

# 17. Interview Questions

### What is HikariCP?

A high-performance JDBC connection pool used by Spring Boot by default.

---

### Why use a connection pool?

To reuse database connections instead of creating a new connection for every request, improving performance and reducing overhead.

---

### Difference between active and idle connections?

| Active                   | Idle                |
| ------------------------ | ------------------- |
| Currently executing work | Available for reuse |

---

### What is pool exhaustion?

When all connections are busy and new requests must wait or eventually time out.

---

### Which metrics are most important?

* `hikaricp.connections.active`
* `hikaricp.connections.idle`
* `hikaricp.connections.pending`
* `hikaricp.connections.max`

---

# 18. Complete Internal Flow

```text
HTTP Request

↓

Repository

↓

Borrow Connection

↓

HikariCP

↓

Execute SQL

↓

Return Connection

↓

Update Metrics

↓

Micrometer

↓

Prometheus

↓

Grafana

↓

Operations Dashboard
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ 13.6.1 Metrics & Micrometer
├── ✅ 13.6.2 JVM Metrics
├── ✅ 13.6.3 HTTP Request Metrics
├── ✅ 13.6.4 Database Connection Pool Metrics ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.6.5 Custom Metrics ⭐⭐⭐⭐⭐
       ↓
       Counter
       ↓
       Gauge
       ↓
       Timer
       ↓
       Distribution Summary
       ↓
       Long Task Timer
       ↓
       Real Enterprise Examples
```

## Next Chapter (13.6.5) — Custom Metrics

This is where you'll learn how to create your **own application metrics**.

We'll cover:

* `Counter`
* `Gauge`
* `Timer`
* `DistributionSummary`
* `LongTaskTimer`
* Business metrics (orders placed, payments processed, login failures)
* Best practices for naming and tagging metrics
* Complete Spring Boot implementations with real-world examples

This is one of the most practical Micrometer topics because you'll create metrics tailored to your application's business logic.
