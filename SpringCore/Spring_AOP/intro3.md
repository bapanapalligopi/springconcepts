Excellent! Now we're moving from **theory to coding**. This is where most interview questions come from.

# Chapter 3: Types of Advice in Spring AOP

We'll cover:

```
1. @Before
2. @After
3. @AfterReturning
4. @AfterThrowing
5. @Around ⭐⭐⭐⭐⭐ (Most Important)
```

Don't worry about memorizing them. We'll understand **why each one exists**.

---

# Before We Start

We'll use this service throughout.

```java
@Service
public class EmployeeService {

    public void saveEmployee() {

        System.out.println("Employee Saved");

    }

    public void updateEmployee() {

        System.out.println("Employee Updated");

    }

}
```

Suppose the controller calls

```java
employeeService.saveEmployee();
```

We want to log information automatically.

---

# 1. @Before Advice

---

# Why?

Imagine your company has a requirement.

> Before executing any service method,
> check whether the user is authenticated.

Or

> Before every method,
> log the method name.

Without AOP

```java
public void saveEmployee() {

    log();

    System.out.println("Employee Saved");

}
```

Now imagine

```
500 methods
```

Every method starts with

```java
log();
```

Huge duplication.

---

# What?

`@Before` executes **before** the target method.

Think of it like this.

```
Method Call

↓

@Before

↓

Business Method
```

---

# How?

Suppose we create an Aspect.

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.demo.service.*.*(..))")
    public void log() {

        System.out.println("Method Started");

    }

}
```

Employee Service

```java
@Service
public class EmployeeService {

    public void saveEmployee() {

        System.out.println("Employee Saved");

    }

}
```

Output

```
Method Started

Employee Saved
```

Notice

Spring executed logging

BEFORE

the service method.

---

# Internal Working

Without AOP

```
Controller

↓

EmployeeService

↓

saveEmployee()
```

With AOP

```
Controller

↓

Proxy

↓

@Before

↓

saveEmployee()

↓

Return
```

The proxy executes the advice first.

---

# Where is it used?

Real projects use `@Before` for:

### Logging

```
Method Started
```

### Authentication

```
Is User Logged In?
```

### Authorization

```
Does User Have ADMIN Role?
```

### Input Validation

```
Validate Request
```

---

# Interview Question

**Q. When does @Before execute?**

Answer

> Immediately before the target method executes.

---

# 2. @After Advice

---

# Why?

Suppose your manager says

```
After every method,

print

Method Finished
```

Question

Should this happen

even if

the method throws an exception?

Yes.

---

# What?

`@After`

executes

AFTER

the method

regardless of

success or failure.

---

Visualization

```
saveEmployee()

↓

Exception?

↓

No Matter What

↓

@After Executes
```

---

Example

```java
@After("execution(* com.demo.service.*.*(..))")
public void logEnd() {

    System.out.println("Method Finished");

}
```

Output

```
Employee Saved

Method Finished
```

---

Even if

```java
throw new RuntimeException();
```

Output

```
Exception

Method Finished
```

---

# Where?

Cleanup tasks

Example

```
Close File

Release Lock

Release Resource
```

---

# Difference

```
@Before

↓

Method

↓

@After
```

---

# 3. @AfterReturning

---

# Why?

Suppose

Logging should happen

ONLY

if the method succeeds.

If an exception occurs

don't log success.

---

# What?

`@AfterReturning`

runs

only after

successful completion.

---

Example

```java
@AfterReturning(
pointcut="execution(* com.demo.service.*.*(..))")
public void success() {

    System.out.println("Success");

}
```

Output

```
Employee Saved

Success
```

---

Suppose

```java
throw new RuntimeException();
```

Output

```
Exception
```

No

```
Success
```

---

# Where?

Real Projects

```
Audit Logs

Success Logs

Performance Metrics

Cache Update
```

---

# Difference

```
Method Success

↓

@AfterReturning

-----------------

Method Exception

↓

No Execution
```

---

# 4. @AfterThrowing

---

# Why?

Suppose

Every exception should be logged.

Instead of

```java
try{

}

catch(Exception e){

}
```

inside

500 methods,

we can centralize exception logging.

---

# What?

Runs

only

if

the method throws an exception.

---

Example

```java
@AfterThrowing(
pointcut="execution(* com.demo.service.*.*(..))")
public void exception() {

    System.out.println("Exception Occurred");

}
```

---

Output

```
RuntimeException

↓

Exception Occurred
```

---

No exception?

Advice never executes.

---

# Where?

Real Projects

```
Error Logging

Rollback Notification

Alert Email

Monitoring
```

---

# 5. @Around ⭐⭐⭐⭐⭐

This is the most important advice.

Almost every interview asks this.

---

# Why?

Suppose you want

```
Log Before

↓

Execute Method

↓

Measure Time

↓

Log After
```

Question

Can

`@Before`

do all this?

No.

Can

`@After`

do all this?

No.

Need something

that surrounds

the method.

---

# What?

`@Around`

executes

BEFORE

AND

AFTER

the method.

It even decides

whether

the method should execute.

---

Visualization

```
@Before Logic

↓

Business Method

↓

After Logic
```

Everything inside

one advice.

---

Example

```java
@Around("execution(* com.demo.service.*.*(..))")
public Object log(ProceedingJoinPoint joinPoint)
throws Throwable {

    System.out.println("Before");

    Object result = joinPoint.proceed();

    System.out.println("After");

    return result;

}
```

Output

```
Before

Employee Saved

After
```

---

# What is

```java
joinPoint.proceed();
```

This is very important.

```
Without proceed()

↓

Business Method Never Executes
```

Only

```java
joinPoint.proceed();
```

calls

the actual service method.

---

Flow

```
Controller

↓

Proxy

↓

Before Logic

↓

joinPoint.proceed()

↓

EmployeeService

↓

After Logic

↓

Return
```

---

# Where?

Most common use cases

```
Performance Monitoring

Logging

Transaction Management

Caching

Security

Retry Logic
```

---

Example

Measure execution time

```java
long start = System.currentTimeMillis();

joinPoint.proceed();

long end = System.currentTimeMillis();

System.out.println(end-start);
```

Very common interview example.

---

# Comparison Table

| Advice            | Executes When                                  |
| ----------------- | ---------------------------------------------- |
| `@Before`         | Before method                                  |
| `@After`          | After method (success or exception)            |
| `@AfterReturning` | Only after successful execution                |
| `@AfterThrowing`  | Only when an exception occurs                  |
| `@Around`         | Before and after; can control method execution |

---

# Complete Timeline

```
Controller

↓

Proxy

↓

@Before

↓

@Around (Before Part)

↓

Business Method

↓

@AfterReturning (if success)

↓

@AfterThrowing (if exception)

↓

@After

↓

@Around (After Part)

↓

Return
```

> **Note:** The exact order of `@After`, `@AfterReturning`, and the "after" part of `@Around` depends on the advice implementation and how aspects are ordered. The timeline above is a conceptual view to help you understand their responsibilities. We'll discuss ordering in more detail later.

---

# Interview Questions

### Q1. Which Advice is used most?

**Answer:**

`@Around`

because it can execute logic before and after the method and even control whether the target method executes.

---

### Q2. Difference between `@After` and `@AfterReturning`?

`@After`

```
Runs Always

↓

Success

OR

Exception
```

`@AfterReturning`

```
Runs

ONLY

after

successful execution.
```

---

### Q3. Difference between `@Before` and `@Around`?

`@Before`

* Runs only before the method.
* Cannot control the method execution.

`@Around`

* Runs before and after the method.
* Can decide whether to call `joinPoint.proceed()`.
* Can modify the return value or handle exceptions.

---

### Q4. What happens if `joinPoint.proceed()` is not called?

The target method is **never executed**. This is why `@Around` is powerful—it has complete control over whether the original method runs.

---

# Best Practices

* Use `@Before` for validation or authorization.
* Use `@AfterReturning` for audit logs or cache updates after successful execution.
* Use `@AfterThrowing` for centralized exception logging.
* Use `@Around` for execution time measurement, retries, transactions, and advanced logging.
* Avoid putting business logic inside aspects; keep them focused on cross-cutting concerns.

---

# Where We Are

```
Spring AOP

✅ Why AOP
✅ Core Concepts
✅ Types of Advice

Next →

Pointcut Expressions
        ↓
execution()
within()
bean()
Wildcards (*, ..)
&& || !
```

## Next Chapter (Very Important)

We'll learn **Pointcut Expressions** in depth.

You'll understand questions like:

* Why do we write `execution(* com.demo.service.*.*(..))`?
* What does each `*` mean?
* What does `..` mean?
* How do we target only `save*()` methods?
* How do we intercept methods with specific parameters?
* How do real projects write pointcuts?

This is one of the most practical parts of Spring AOP because it determines **exactly which methods your advice applies to**.
