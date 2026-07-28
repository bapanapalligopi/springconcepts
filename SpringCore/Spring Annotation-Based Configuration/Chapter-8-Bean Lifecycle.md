Excellent. This is one of the **top 10 Spring interview questions** for **1–3 years** experience.

> **"Explain the lifecycle of a Spring Bean."**

If you answer this properly with an example, it leaves a strong impression.

---

# Chapter 7: Spring Bean Lifecycle

Before learning the lifecycle, let's ask a question.

Suppose we have a bean.

```java
@Component
public class EmployeeService {

}
```

Question:

When does Spring create this object?

What happens after object creation?

When is it destroyed?

The complete journey of a bean from creation to destruction is called the **Bean Lifecycle**.

---

# Definition

**Bean Lifecycle** is the sequence of steps a bean goes through from the moment Spring creates it until it is destroyed.

Simply,

```text
Bean Creation

↓

Dependency Injection

↓

Initialization

↓

Ready to Use

↓

Destruction
```

---

# Complete Lifecycle

For interview purposes, remember this flow.

```text
Application Starts

        │
        ▼

Bean Created

        │
        ▼

Dependencies Injected

        │
        ▼

@PostConstruct

        │
        ▼

Bean Ready

        │
        ▼

Application Running

        │
        ▼

Application Shutdown

        │
        ▼

@PreDestroy

        │
        ▼

Bean Destroyed
```

This diagram alone answers most interview questions.

---

# Step 1: Bean Instantiation

Suppose

```java
@Component
public class EmployeeService {

    public EmployeeService() {
        System.out.println("Constructor Called");
    }

}
```

When Spring starts,

Output

```text
Constructor Called
```

The object is now created.

At this stage

```text
EmployeeService

↓

Object Exists
```

But it is **not fully initialized** yet.

---

# Step 2: Dependency Injection

Suppose

```java
@Repository
public class EmployeeRepository {

}
```

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

}
```

Spring now injects

```text
EmployeeRepository
```

into

```text
EmployeeService
```

Now

```text
repository != null
```

The bean has all required dependencies.

---

# Step 3: @PostConstruct

Now imagine your application needs to perform some initialization.

Example:

* Load configuration
* Open a file
* Initialize a cache
* Create a connection pool
* Print startup information

Spring provides

```java
@PostConstruct
```

Example

```java
import jakarta.annotation.PostConstruct;
import org.springframework.stereotype.Component;

@Component
public class EmployeeService {

    @PostConstruct
    public void init() {

        System.out.println("Initialization Logic");

    }

}
```

Output

```text
Initialization Logic
```

This method is called **once**, immediately after dependency injection.

---

# Real Project Example

Suppose your application uses Redis.

When the application starts

```java
@PostConstruct
public void loadCache(){

    System.out.println("Loading Employee Cache");

}
```

Output

```text
Loading Employee Cache
```

The cache is loaded before users start making requests.

---

# Why Not Put This in Constructor?

Suppose

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

    public EmployeeService() {

        repository.findAll();

    }

}
```

Will this work?

No.

Why?

Because during constructor execution

```text
repository

↓

null
```

Dependency injection hasn't happened yet.

Result

```text
NullPointerException
```

---

# Correct Way

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

    @PostConstruct
    public void init(){

        repository.findAll();

    }

}
```

Now

```text
Constructor

↓

Dependency Injection

↓

@PostConstruct
```

Repository is available.

---

# Step 4: Bean Ready

Now the bean is ready.

Spring stores it in the singleton cache.

Whenever another bean needs it

```java
@Autowired
private EmployeeService service;
```

Spring returns the already initialized bean.

---

# Step 5: Application Shutdown

When the application stops,

Spring starts destroying beans.

Before destruction,

Spring calls

```java
@PreDestroy
```

Example

```java
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class EmployeeService {

    @PreDestroy
    public void destroy(){

        System.out.println("Cleaning Resources");

    }

}
```

Output

```text
Cleaning Resources
```

---

# Real Example

Suppose

```java
@Component
public class FileManager {

    private FileInputStream stream;

    @PostConstruct
    public void openFile(){

        System.out.println("Opening File");

    }

    @PreDestroy
    public void closeFile(){

        System.out.println("Closing File");

    }

}
```

Output

```text
Opening File

...

Application Running

...

Closing File
```

---

# Complete Example

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class EmployeeService {

    public EmployeeService() {
        System.out.println("Constructor Called");
    }

    @PostConstruct
    public void init() {
        System.out.println("PostConstruct Called");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("PreDestroy Called");
    }

}
```

Output

```text
Constructor Called

PostConstruct Called

Application Running...

PreDestroy Called
```

---

# Timeline

```text
Application Starts

↓

Constructor

↓

@Autowired

↓

@PostConstruct

↓

Application Running

↓

Shutdown

↓

@PreDestroy
```

---

# Where Should We Write Logic?

## Constructor

Use for

* Simple object creation
* Assigning constructor parameters

Avoid

* Database calls
* Repository calls
* Business logic

---

## @PostConstruct

Use for

* Cache initialization
* Loading configuration
* Reading files
* Starting background tasks (if appropriate)
* Preparing resources

---

## @PreDestroy

Use for

* Closing files
* Closing sockets
* Releasing resources
* Cleaning up before shutdown

---

# Singleton vs Prototype

Interview Question:

Will `@PreDestroy` execute for Prototype beans?

Answer:

**No**, Spring creates prototype beans but **does not manage their complete lifecycle after handing them to the application**. Therefore, destruction callbacks like `@PreDestroy` are not automatically invoked for prototype-scoped beans.

---

# Interview Questions

## 1. What is Bean Lifecycle?

A bean lifecycle is the sequence of phases a Spring bean goes through, from creation, dependency injection, and initialization to destruction.

---

## 2. When is `@PostConstruct` executed?

After dependency injection is complete and before the bean is used.

---

## 3. When is `@PreDestroy` executed?

Just before Spring destroys the bean during application shutdown (typically for singleton beans).

---

## 4. Can we access an `@Autowired` bean inside the constructor?

No. Constructor injection is different, but if you're using **field injection**, `@Autowired` fields are injected **after** the constructor finishes. Therefore, field-injected dependencies are `null` inside the constructor.

Example:

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

    public EmployeeService() {
        // repository is null here
    }
}
```

---

## 5. Where have you used `@PostConstruct` in your project?

A good interview answer:

> "I have used `@PostConstruct` to initialize application data and load frequently accessed information into memory after the application starts. It ensures all dependencies are available before the initialization logic runs."

---

# Summary

```text
Spring Bean Lifecycle

        │
        ▼
Constructor
        │
        ▼
Dependency Injection
        │
        ▼
@PostConstruct
        │
        ▼
Bean Ready
        │
        ▼
Application Running
        │
        ▼
@PreDestroy
        │
        ▼
Bean Destroyed
```

---

# Next Topic

The next topic is another **very important interview question**:

## `@Bean` and `@Configuration`

You'll learn:

* Why do we need `@Bean` when we already have `@Component`?
* Difference between `@Bean` and `@Component`
* Real project examples
* When to use each
* How we configure third-party libraries like Redis, Jackson, and RestTemplate

This topic is essential because you'll see `@Bean` methods in almost every Spring Boot project.
