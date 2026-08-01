# Spring Boot — Chapter 12.2: Application Events ⭐⭐⭐⭐⭐

One of the most underrated Spring Boot topics.

Many developers know **how to create REST APIs**, but very few understand **Spring's Event System**, even though it's widely used in enterprise applications.

---

# Chapter Roadmap

```text
Application Events
│
├── 1. What are Events?
├── 2. Why do we need Events?
├── 3. Publisher & Listener
├── 4. Event Flow
├── 5. Built-in Spring Boot Events
├── 6. @EventListener
├── 7. Publishing Custom Events
├── 8. Synchronous vs Asynchronous Events
├── 9. Transaction Events
├── 10. Best Practices
├── 11. Enterprise Examples
└── 12. Interview Questions
```

---

# 1. What is an Event?

An **event** is simply:

> **A notification that something important has happened.**

Real-life examples:

```text
Door Bell Rings
        │
        ▼
Someone came
```

```text
Fire Alarm
        │
        ▼
Fire detected
```

```text
Order Created
        │
        ▼
Notify warehouse
```

In Spring:

```text
Employee Created

↓

Event Published

↓

Interested Components React
```

Notice something important:

The component that creates the employee **doesn't need to know** who is listening.

---

# 2. Why Do We Need Events?

Suppose we create an employee.

Without events:

```text
EmployeeService

↓

Save Employee

↓

Send Email

↓

Write Audit Log

↓

Clear Cache

↓

Update Search Index

↓

Notify HR

↓

Publish Kafka Message
```

One service is doing **everything**.

Problems:

* Huge class
* Tight coupling
* Hard to maintain
* Hard to test

---

# 3. With Events

Instead:

```text
EmployeeService

↓

Save Employee

↓

Publish EmployeeCreatedEvent
```

Then:

```text
                EmployeeCreatedEvent
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
 Send Email      Write Audit      Clear Cache
                                       │
                                       ▼
                                Notify HR
                                       │
                                       ▼
                                Publish Kafka
```

Now every component is independent.

This is called **event-driven architecture**.

---

# 4. Important Principle

The publisher **does not know** who listens.

Publisher:

```text
EmployeeService

↓

Publish Event

↓

Done
```

It doesn't know whether:

* 0 listeners exist
* 2 listeners exist
* 100 listeners exist

That's the beauty of events.

---

# 5. Spring Event Architecture

```text
                  ApplicationContext
                         │
          (acts as Event Publisher)
                         │
                         ▼
                 Publish Event
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
  Listener A       Listener B      Listener C
```

The `ApplicationContext` is also an **ApplicationEventPublisher**.

---

# 6. Built-in Spring Boot Events

Spring Boot publishes many lifecycle events automatically.

```text
ApplicationStartingEvent

↓

ApplicationEnvironmentPreparedEvent

↓

ApplicationContextInitializedEvent

↓

ApplicationPreparedEvent

↓

ApplicationStartedEvent

↓

ApplicationReadyEvent

↓

ApplicationFailedEvent
```

These happen during application startup.

---

# 7. Which Event Happens When?

```text
main()

↓

ApplicationStartingEvent

↓

Read Configuration

↓

ApplicationEnvironmentPreparedEvent

↓

Create Context

↓

ApplicationPreparedEvent

↓

Create Beans

↓

ApplicationStartedEvent

↓

CommandLineRunner

↓

ApplicationReadyEvent

↓

Application Running
```

If startup fails:

```text
ApplicationFailedEvent
```

---

# 8. Listening to an Event

The easiest way:

```java
@Component
public class StartupListener {

    @EventListener
    public void onReady(
            ApplicationReadyEvent event) {

        System.out.println("Application is ready.");
    }
}
```

Flow:

```text
ApplicationReadyEvent

↓

ApplicationContext

↓

@EventListener

↓

Your Method Executes
```

---

# 9. What Happens Internally?

Suppose Spring publishes:

```text
ApplicationReadyEvent
```

Internally:

```text
ApplicationContext

↓

Looks for every @EventListener

↓

Checks Event Type

↓

Invokes Matching Methods
```

Only listeners interested in that event are called.

---

# 10. Multiple Listeners

You can have:

```java
@Component
class EmailListener {

    @EventListener
    void handle(ApplicationReadyEvent e) {
    }
}
```

```java
@Component
class CacheListener {

    @EventListener
    void handle(ApplicationReadyEvent e) {
    }
}
```

```java
@Component
class MetricsListener {

    @EventListener
    void handle(ApplicationReadyEvent e) {
    }
}
```

Execution:

```text
ApplicationReadyEvent

↓

EmailListener

↓

CacheListener

↓

MetricsListener
```

Every listener receives the same event.

---

# 11. Custom Events

Events aren't limited to Spring Boot lifecycle.

You can create your own.

Example:

```java
public record EmployeeCreatedEvent(
        Long employeeId,
        String email
) {
}
```

Modern Spring doesn't require extending `ApplicationEvent`.

Any object can be published as an event.

---

# 12. Publishing an Event

Inject:

```java
private final ApplicationEventPublisher publisher;
```

Example:

```java
@Service
public class EmployeeService {

    private final ApplicationEventPublisher publisher;

    public EmployeeService(
            ApplicationEventPublisher publisher) {

        this.publisher = publisher;
    }

    public void createEmployee() {

        // Save employee...

        publisher.publishEvent(
            new EmployeeCreatedEvent(
                1L,
                "john@example.com"
            )
        );
    }
}
```

Flow:

```text
Employee Saved

↓

publishEvent()

↓

ApplicationContext

↓

Listeners Execute
```

---

# 13. Listening to Custom Events

```java
@Component
public class EmailListener {

    @EventListener
    public void handle(
            EmployeeCreatedEvent event) {

        System.out.println(
            "Sending welcome email to "
            + event.email()
        );
    }
}
```

Whenever the event is published, this method executes.

---

# 14. Multiple Listeners Example

```text
EmployeeCreatedEvent
         │
         ▼
 ┌───────┼────────┬─────────────┐
 ▼       ▼        ▼             ▼
Email  Audit   Analytics    Cache
```

Publisher doesn't know about any of them.

Each listener focuses on one responsibility.

---

# 15. Synchronous Events

By default, Spring events are synchronous.

Flow:

```text
EmployeeService

↓

publishEvent()

↓

Listener A

↓

Listener B

↓

Listener C

↓

Return
```

The publisher waits until all listeners finish.

If one listener is slow, the publisher is delayed.

---

# 16. Asynchronous Events

Sometimes listeners shouldn't block the publisher.

Example:

```text
Employee Created

↓

Publish Event

↓

Return Immediately

↓

Background Thread

↓

Send Email
```

Enable async support:

```java
@EnableAsync
@Configuration
public class AsyncConfig {
}
```

Listener:

```java
@Async
@EventListener
public void handle(
        EmployeeCreatedEvent event) {

    // Send email
}
```

Now the listener runs in another thread.

---

# 17. Transactional Events

Imagine:

```text
Save Employee

↓

Publish Event

↓

Database Rollback
```

Oops!

Email already sent.

But employee doesn't exist.

Bad.

Spring provides:

```java
@TransactionalEventListener
```

Example:

```java
@Component
public class EmailListener {

    @TransactionalEventListener
    public void handle(
            EmployeeCreatedEvent event) {

        // Send email
    }
}
```

Now the listener executes **after a successful transaction commit** by default.

This avoids acting on data that later rolls back.

---

# 18. Enterprise Example

Employee registration:

```text
EmployeeService

↓

Save Employee

↓

Transaction Commit

↓

EmployeeCreatedEvent

↓

Send Welcome Email

↓

Write Audit Log

↓

Publish Kafka Event

↓

Refresh Cache

↓

Notify HR
```

This is how enterprise systems separate responsibilities.

---

# 19. Event Flow Diagram

```text
Client

↓

POST /employees

↓

EmployeeController

↓

EmployeeService

↓

Repository.save()

↓

publishEvent()

↓

ApplicationContext
        │
 ┌──────┼──────────────┐
 ▼      ▼              ▼
Email  Audit      Notification
Listener Listener   Listener
```

Notice:

The controller never knows about these listeners.

---

# 20. Common Mistakes

❌ Putting business logic in listeners that must happen before the main request completes.

❌ Using synchronous listeners for long-running work like sending emails.

❌ Publishing events before the transaction is committed when listeners depend on committed data.

❌ Creating circular event chains (Listener A publishes Event B, which republishes Event A).

---

# 21. Best Practices

```text
✅ Keep listeners focused on one responsibility

✅ Use events to reduce coupling

✅ Prefer @TransactionalEventListener for database-related events

✅ Use @Async for slow operations

✅ Keep event objects immutable (records are ideal)

✅ Give events meaningful names
   (EmployeeCreatedEvent, OrderPaidEvent)

❌ Don't replace normal method calls with events everywhere

❌ Don't put critical synchronous business workflows only in async listeners
```

---

# 22. Interview Questions

### What is an Application Event?

> An application event is a notification that something has happened within the application. Spring's event system allows publishers and listeners to communicate without being tightly coupled.

---

### What is `ApplicationEventPublisher`?

> It is the Spring component responsible for publishing events to all registered listeners.

---

### What is `@EventListener`?

> It marks a method that should be invoked automatically when a matching event is published.

---

### Can we create custom events?

> Yes. In modern Spring, any object (including a Java record) can be published as an event.

---

### Are Spring events synchronous?

> Yes, by default they are synchronous. They can be made asynchronous using `@Async` with async support enabled.

---

### What is `@TransactionalEventListener`?

> It delays event handling until a specified transaction phase, most commonly **after a successful transaction commit**, ensuring listeners act only on committed data.

---

# 23. Mental Model

Remember this picture:

```text
                   EmployeeService
                          │
                    Save Employee
                          │
                          ▼
                 publishEvent(event)
                          │
                          ▼
                ApplicationContext
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   EmailListener    AuditListener   CacheListener
        │                 │                 │
        ▼                 ▼                 ▼
 Send Email         Write Log       Refresh Cache
```

**The publisher doesn't know who is listening. The listeners don't know who published the event.**

This loose coupling is the biggest advantage of Spring's event mechanism.

---

# 📍 Next Topic

## **Chapter 12.3 — CommandLineRunner ⭐⭐⭐⭐⭐**

We'll cover:

* Why `CommandLineRunner` exists
* Complete execution flow
* Multiple runners and `@Order`
* Real enterprise use cases
* Data seeding
* Cache warming
* Startup validation
* `CommandLineRunner` vs `ApplicationRunner`
* Internal execution sequence
* Interview questions
