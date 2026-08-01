# Spring Boot — Chapter 12.6: `@PreDestroy` ⭐⭐⭐⭐⭐

In the previous chapter, we learned **how a bean is initialized** using `@PostConstruct`.

Now we'll learn the opposite:

> **What happens when Spring Boot shuts down?**

This is where `@PreDestroy` comes in.

It is one of the most important lifecycle annotations for enterprise applications because resources must be cleaned up properly.

---

# Chapter Roadmap

```text id="7xk2ma"
@PreDestroy
│
├── 1. What is @PreDestroy?
├── 2. Why do we need it?
├── 3. Bean Destruction Lifecycle
├── 4. Shutdown Flow
├── 5. Complete Example
├── 6. Enterprise Examples
├── 7. What Happens Internally?
├── 8. Common Mistakes
├── 9. Best Practices
└── 10. Interview Questions
```

---

# 1. What is `@PreDestroy`?

`@PreDestroy` marks a method that Spring calls **just before destroying a bean**.

Think of it as the **cleanup phase** of a bean.

Example:

```java
@Service
public class CacheService {

    @PreDestroy
    public void cleanup() {

        System.out.println("Cleaning cache...");
    }
}
```

You never call this method yourself.

Spring automatically executes it during application shutdown.

---

# 2. Why Do We Need It?

Imagine your application uses:

* Database connections
* File streams
* Thread pools
* Kafka consumers
* Redis connections
* Scheduled tasks

When the application stops,

Should we simply exit?

No.

Everything must be cleaned properly.

Without cleanup:

```text
Application Stops

↓

Open Files

↓

Memory Leaks

↓

Incomplete Transactions

↓

Resource Leaks
```

With `@PreDestroy`:

```text
Application Stops

↓

Cleanup Resources

↓

Close Connections

↓

Shutdown Safely
```

---

# 3. Where Does `@PreDestroy` Execute?

Remember the complete lifecycle.

```text
Application Starts

↓

Bean Created

↓

Constructor

↓

Dependency Injection

↓

@PostConstruct

↓

Application Running

↓

Shutdown Signal

↓

@PreDestroy ← HERE

↓

Bean Destroyed

↓

Application Ends
```

---

# 4. What Triggers `@PreDestroy`?

Several situations can trigger it.

### Ctrl + C

```text
Ctrl + C

↓

Shutdown Hook

↓

Spring Context Closes

↓

@PreDestroy
```

---

### Docker Stop

```text
docker stop

↓

SIGTERM

↓

Spring Shutdown

↓

@PreDestroy
```

---

### Kubernetes Pod Termination

```text
Kubernetes

↓

SIGTERM

↓

Graceful Shutdown

↓

@PreDestroy
```

---

### IDE Stop Button

```text
Stop Application

↓

Spring Context Closed

↓

@PreDestroy
```

---

# 5. Simple Example

```java
@Service
public class ReportService {

    @PostConstruct
    public void init() {

        System.out.println("Service Started");
    }

    @PreDestroy
    public void destroy() {

        System.out.println("Service Destroyed");
    }
}
```

Output:

Startup:

```text
Service Started
```

Shutdown:

```text
Service Destroyed
```

---

# 6. Internal Flow

Suppose Spring receives:

```text
Shutdown Signal
```

Internally:

```text
Shutdown Signal

↓

Stop Accepting Requests

↓

Close ApplicationContext

↓

Destroy Beans

↓

Call @PreDestroy

↓

Release Memory

↓

JVM Exits
```

---

# 7. Enterprise Example – Thread Pool

Suppose:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);
```

If we don't stop it,

Threads remain alive until the JVM exits.

Better:

```java
@Service
public class ReportService {

    private final ExecutorService executor =
            Executors.newFixedThreadPool(10);

    @PreDestroy
    public void shutdown() {

        executor.shutdown();
    }
}
```

Now all threads stop gracefully.

---

# 8. Enterprise Example – File Handling

Suppose:

```java
FileWriter writer;
```

Open during startup.

Cleanup:

```java
@PreDestroy
public void closeFile() throws Exception {

    writer.close();
}
```

Without this:

```text
File

↓

Not Closed

↓

Resource Leak
```

---

# 9. Enterprise Example – Cache Flush

Suppose:

```text
Redis Cache

↓

Local Cache

↓

Buffered Data
```

Before shutdown:

```java
@PreDestroy
public void flush() {

    cache.flush();
}
```

Now no data is lost.

---

# 10. Enterprise Example – Kafka Consumer

Suppose:

```text
Kafka Consumer

↓

Reading Messages
```

Shutdown:

```java
@PreDestroy
public void stopConsumer() {

    consumer.close();
}
```

This commits offsets and disconnects cleanly.

---

# 11. Enterprise Example – Scheduler

```java
ScheduledExecutorService scheduler;
```

Shutdown:

```java
@PreDestroy
public void stopScheduler() {

    scheduler.shutdown();
}
```

Otherwise,

Background jobs may terminate abruptly.

---

# 12. What Happens Internally?

Spring keeps track of every singleton bean.

During shutdown:

```text
ApplicationContext

↓

Bean A

↓

Bean B

↓

Bean C

↓

Destroy Bean

↓

Call @PreDestroy

↓

Remove Bean
```

Spring destroys beans in an order that respects dependencies where possible.

---

# 13. Bean Destruction Flow

```text
Application Running

↓

Shutdown Signal

↓

Stop New Requests

↓

Finish Existing Requests

↓

Destroy Singleton Beans

↓

@PreDestroy

↓

Close Resources

↓

Application Ends
```

---

# 14. `@PostConstruct` vs `@PreDestroy`

| `@PostConstruct`           | `@PreDestroy`           |
| -------------------------- | ----------------------- |
| Startup                    | Shutdown                |
| Initialization             | Cleanup                 |
| After dependency injection | Before bean destruction |
| Prepare resources          | Release resources       |

Memory trick:

```text
@PostConstruct

↓

START

----------------

@PreDestroy

↓

END
```

---

# 15. Common Mistakes

### Heavy Processing

Bad:

```java
@PreDestroy
public void cleanup() {

    Thread.sleep(600000);
}
```

Application shutdown takes 10 minutes.

---

### Throwing Exceptions

```java
@PreDestroy
public void cleanup() {

    throw new RuntimeException();
}
```

Cleanup methods should handle their own exceptions where practical so shutdown can continue gracefully.

---

### Starting New Work

Wrong:

```java
@PreDestroy
public void cleanup() {

    sendNewEmails();
}
```

Shutdown is for **finishing**, not starting new business work.

---

# 16. What Should Go Inside `@PreDestroy`?

Good examples:

```text
Close Database Resources

↓

Close Files

↓

Shutdown Thread Pools

↓

Stop Kafka Consumers

↓

Disconnect Redis

↓

Flush Cache

↓

Release Memory
```

Not:

```text
Process Orders

↓

Generate Reports

↓

Business Logic
```

---

# 17. Best Practices

```text
✅ Release resources

✅ Close streams

✅ Stop executors

✅ Disconnect external systems

✅ Keep cleanup fast

✅ Handle exceptions gracefully

❌ Don't start new business operations

❌ Don't block shutdown for a long time

❌ Don't ignore resource cleanup
```

---

# 18. Interview Questions

### What is `@PreDestroy`?

> It is a lifecycle annotation that marks a method to be executed before a Spring-managed bean is destroyed.

---

### When is it executed?

> During application shutdown, after the application context begins closing and before the bean is removed.

---

### Typical use cases?

* Closing files
* Releasing resources
* Stopping thread pools
* Disconnecting message brokers
* Flushing caches

---

### Is it called for every bean?

> It is called for Spring-managed beans that participate in the container lifecycle. Prototype-scoped beans are generally not destroyed automatically by the container, so their `@PreDestroy` methods are not invoked by Spring.

---

### Can it have parameters?

No.

```java
@PreDestroy
public void cleanup() {

}
```

---

# 19. Complete Bean Lifecycle

```text
Spring Starts

↓

Create Bean

↓

Constructor

↓

Dependency Injection

↓

@PostConstruct

↓

Bean Ready

↓

Application Running

↓

Shutdown Signal

↓

@PreDestroy

↓

Bean Destroyed

↓

JVM Exit
```

---

# 20. Enterprise Shutdown Flow

Imagine an Employee Management System.

```text
Employee API Running

↓

SIGTERM Received

↓

Stop Accepting Requests

↓

Finish Active Requests

↓

Close Database Resources

↓

Shutdown Kafka Consumer

↓

Flush Redis Cache

↓

Stop Scheduler

↓

@PreDestroy

↓

Close ApplicationContext

↓

Application Stops
```

This graceful shutdown sequence helps prevent data loss and resource leaks.

---

# 📍 Where We Are

```text
Spring Boot Advanced Features
│
├── ✅ 12.1 Application Lifecycle
├── ✅ 12.2 Application Events
├── ✅ 12.3 CommandLineRunner
├── ✅ 12.4 ApplicationRunner
├── ✅ 12.5 @PostConstruct
├── ✅ 12.6 @PreDestroy ⭐⭐⭐⭐⭐
│
└── ⏭️ 12.7 Lazy Initialization ⭐⭐⭐⭐⭐
       ↓
       @Lazy
       ↓
       Bean Creation Strategy
       ↓
       Startup Performance
       ↓
       Global Lazy Initialization
       ↓
       Enterprise Use Cases
```

In the next chapter, we'll dive into **Lazy Initialization (`@Lazy`)**, including how Spring creates beans eagerly by default, how lazy loading changes that behavior, its impact on startup performance, and when it should (and shouldn't) be used in production.
