# Chapter 13.6.5 — Custom Metrics with Micrometer ⭐⭐⭐⭐⭐

# (Complete Enterprise Implementation)

Until now, we've seen **built-in metrics** that Spring Boot provides automatically.

But enterprise applications also need **business metrics**.

Examples:

* How many orders were placed today?
* How many users logged in?
* How many payment failures occurred?
* What is the average order processing time?
* How many active shopping carts exist?

Spring Boot **cannot know your business**, so you create **custom metrics** using **Micrometer**.

This is one of the **most frequently asked senior Spring Boot interview topics**.

---

# Chapter Roadmap

```text
Custom Metrics
│
├── 1. Why Custom Metrics?
├── 2. Meter Types
│     ├── Counter
│     ├── Gauge
│     ├── Timer
│     ├── Distribution Summary
│     └── Long Task Timer
├── 3. MeterRegistry
├── 4. Counter Implementation
├── 5. Gauge Implementation
├── 6. Timer Implementation
├── 7. Distribution Summary
├── 8. Long Task Timer
├── 9. Tags
├── 10. Best Practices
└── Interview Questions
```

---

# 1. Why Custom Metrics?

Suppose you own an e-commerce application.

Spring Boot already tells you:

```text
CPU = 40%

Memory = 2 GB

HTTP Requests = 500/sec
```

Useful?

Yes.

But management asks:

```text
How many orders succeeded today?
```

Spring Boot doesn't know.

You create your own metric.

---

# 2. Micrometer Meter Types

Micrometer provides five major meter types.

```text
Meter
│
├── Counter
├── Gauge
├── Timer
├── Distribution Summary
└── Long Task Timer
```

Each measures something different.

---

# 3. MeterRegistry

Every custom metric is registered inside the **MeterRegistry**.

```text
Application

↓

MeterRegistry

↓

Counter

↓

Gauge

↓

Timer

↓

Expose to Actuator
```

Inject it like any Spring bean.

```java
@Service
public class OrderService {

    private final MeterRegistry meterRegistry;

    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
}
```

---

# PART 1 — Counter ⭐⭐⭐⭐⭐

## What is Counter?

A Counter **only increases**.

Examples:

* Orders
* Logins
* Registrations
* Payments
* Emails Sent

---

### Flow

```text
Order Created

↓

Counter++

↓

Order Created

↓

Counter++

↓

Order Created

↓

Counter++

↓

Total = 3
```

---

## Counter Implementation

```java
@Service
public class OrderService {

    private final Counter orderCounter;

    public OrderService(MeterRegistry registry) {

        this.orderCounter = Counter.builder("orders.created")
                .description("Total Orders Created")
                .register(registry);
    }

    public void createOrder() {

        // Business Logic

        orderCounter.increment();

    }

}
```

---

## View Metric

```
GET /actuator/metrics/orders.created
```

Example:

```json
{
  "name":"orders.created",
  "measurements":[
    {
      "statistic":"COUNT",
      "value":523
    }
  ]
}
```

523 orders created.

---

# PART 2 — Gauge ⭐⭐⭐⭐⭐

Counter measures totals.

Gauge measures **current values**.

Examples:

* Active Users
* Queue Size
* Shopping Cart Count
* Online Sessions

---

### Gauge Flow

```text
Users Online

↓

25

↓

40

↓

18

↓

33
```

Gauge goes **up and down**.

---

## Gauge Implementation

```java
@Service
public class UserService {

    private final AtomicInteger activeUsers =
            new AtomicInteger(0);

    public UserService(MeterRegistry registry){

        Gauge.builder(
                "users.active",
                activeUsers,
                AtomicInteger::get
        ).register(registry);

    }

    public void login(){

        activeUsers.incrementAndGet();

    }

    public void logout(){

        activeUsers.decrementAndGet();

    }

}
```

---

View:

```
GET /actuator/metrics/users.active
```

Example:

```json
{
  "value":31
}
```

31 users currently active.

---

# PART 3 — Timer ⭐⭐⭐⭐⭐

One of the most important metrics.

Measures:

* Execution time
* Response time
* Processing time

---

Example

```text
Payment Started

↓

3.2 sec

↓

Payment Completed
```

---

## Timer Implementation

```java
@Service
public class PaymentService {

    private final Timer paymentTimer;

    public PaymentService(MeterRegistry registry){

        paymentTimer = Timer.builder("payment.processing")
                .description("Payment Processing Time")
                .register(registry);

    }

    public void processPayment(){

        paymentTimer.record(() -> {

            // Business Logic

            try{
                Thread.sleep(250);
            }catch(Exception ignored){}

        });

    }

}
```

---

View

```
GET /actuator/metrics/payment.processing
```

Response

```json
{
  "count":2500,
  "totalTime":540,
  "max":1.2
}
```

---

# PART 4 — Distribution Summary ⭐⭐⭐⭐

Measures values instead of durations.

Example:

```text
Order Amount

↓

500

↓

2000

↓

7500

↓

1200
```

Useful for:

* File Size
* Order Amount
* Upload Size
* Payload Size

---

Implementation

```java
@Service
public class OrderService {

    private final DistributionSummary summary;

    public OrderService(MeterRegistry registry){

        summary = DistributionSummary
                .builder("order.amount")
                .register(registry);

    }

    public void createOrder(double amount){

        summary.record(amount);

    }

}
```

---

Metrics

```
Average Order

Maximum Order

Total Orders
```

---

# PART 5 — Long Task Timer ⭐⭐⭐⭐

Measures tasks lasting minutes or hours.

Examples

```text
Report Generation

↓

8 Minutes
```

```text
Video Encoding

↓

20 Minutes
```

```text
Database Migration

↓

15 Minutes
```

---

Implementation

```java
@Service
public class ReportService {

    private final LongTaskTimer timer;

    public ReportService(MeterRegistry registry){

        timer = LongTaskTimer
                .builder("report.generation")
                .register(registry);

    }

    public void generate(){

        LongTaskTimer.Sample sample =
                timer.start();

        try{

            // Long Running Task

        }finally{

            sample.stop();

        }

    }

}
```

---

# Tags ⭐⭐⭐⭐⭐

Without tags

```text
Orders = 1000
```

Not useful.

With tags

```text
Orders

↓

Country = India

↓

Payment = UPI
```

Example

```java
Counter.builder("orders.created")
       .tag("country","India")
       .tag("payment","UPI")
       .register(registry);
```

Now Grafana can filter.

```
Orders

↓

Country

↓

Payment Type

↓

Device

↓

Version
```

---

# Enterprise Example

Suppose

```text
Amazon
```

Metrics

```text
Orders Created

Orders Cancelled

Payments Failed

Cart Size

Users Online

Search Count

Coupon Usage

Inventory Remaining
```

These are all **business metrics**.

---

# Naming Best Practices

Good

```
orders.created

payment.success

payment.failed

users.active

inventory.stock
```

Bad

```
metric1

test

counter

abc
```

Metric names should clearly describe what they measure.

---

# Common Mistakes

### Mistake 1

Using Counter for current values.

Wrong.

Counter never decreases.

Use Gauge.

---

### Mistake 2

Using Gauge for totals.

Wrong.

Use Counter.

---

### Mistake 3

Creating too many tags.

Example

```
UserId

OrderId

Email

Phone
```

Terrible.

High-cardinality tags create millions of unique time series, increasing memory usage and reducing monitoring performance.

Use stable, low-cardinality tags like:

* Region
* Payment type
* HTTP method
* Status

---

# Best Practices

```
✅ Counter for totals

✅ Gauge for current values

✅ Timer for execution time

✅ DistributionSummary for values

✅ LongTaskTimer for long jobs

✅ Meaningful metric names

✅ Low-cardinality tags

❌ Never use UserId as a tag

❌ Don't create duplicate metrics
```

---

# Interview Questions

### Which class registers metrics?

```
MeterRegistry
```

---

### Difference between Counter and Gauge?

| Counter        | Gauge            |
| -------------- | ---------------- |
| Only increases | Goes up and down |
| Total Orders   | Active Users     |

---

### Which meter measures execution time?

```
Timer
```

---

### Which meter measures payload size?

```
DistributionSummary
```

---

### Which meter measures long-running tasks?

```
LongTaskTimer
```

---

# Complete Architecture

```text
Business Event

↓

Counter / Gauge / Timer

↓

MeterRegistry

↓

Micrometer

↓

Actuator

↓

Prometheus

↓

Grafana

↓

Dashboard
```

---

# 📍 Where We Are

```text
Spring Boot Actuator
│
├── ✅ Metrics & Micrometer
├── ✅ JVM Metrics
├── ✅ HTTP Metrics
├── ✅ HikariCP Metrics
├── ✅ Custom Metrics ⭐⭐⭐⭐⭐
│
└── ⏭️ 13.6.6 Prometheus Integration ⭐⭐⭐⭐⭐
       ↓
       Prometheus Registry
       ↓
       /actuator/prometheus
       ↓
       Scraping
       ↓
       PromQL
       ↓
       Enterprise Monitoring
```

# Next Chapter (13.6.6) — Prometheus Integration

We'll build a **complete production monitoring stack**:

* Adding the Prometheus Micrometer registry dependency
* Exposing the `/actuator/prometheus` endpoint
* Understanding the Prometheus text exposition format
* How Prometheus scrapes Spring Boot applications
* Configuring `prometheus.yml`
* PromQL basics
* Running Prometheus locally with Docker
* Complete Spring Boot + Prometheus implementation
* Production architecture with Prometheus, Grafana, and Alertmanager

This chapter is where Actuator metrics become part of a real-world observability pipeline.
