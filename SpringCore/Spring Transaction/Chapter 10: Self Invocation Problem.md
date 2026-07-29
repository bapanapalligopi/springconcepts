Excellent. Let's complete Spring Transactions with the **most famous Spring interview question**.

> **Chapter 10: Self Invocation Problem (Why `@Transactional` Sometimes Doesn't Work)**

This topic combines everything you've learned so far:

* ✅ Spring Core (IoC)
* ✅ Dependency Injection
* ✅ Spring AOP
* ✅ Proxy
* ✅ `@Transactional`

---

# 1. Why do we need to understand Self Invocation?

Imagine you're working on a banking application.

You write:

```java
@Service
public class BankService {

    public void transferMoney() {

        debit();

        credit();

    }

    @Transactional
    public void debit() {

        // Update Account

    }

}
```

You expect:

```
transferMoney()

↓

debit()

↓

Transaction Starts
```

But when you test it,

**No transaction starts.**

Why?

This surprises almost every Spring developer at least once.

---

# 2. What is Self Invocation?

### Definition

> **Self Invocation happens when one method inside a class calls another method of the same class directly.**

Example:

```java
@Service
public class EmployeeService {

    public void methodA() {

        methodB();

    }

    @Transactional
    public void methodB() {

    }

}
```

Here,

```
methodA()

↓

methodB()
```

Both methods belong to the same object.

This is called **Self Invocation**.

---

# 3. Why doesn't `@Transactional` work?

Remember how Spring AOP works.

Spring does **not** inject your original object.

It injects a **Proxy**.

Example:

```
Client

↓

Spring Proxy

↓

EmployeeService
```

When the client calls:

```java
employeeService.methodB();
```

Flow:

```
Client

↓

Proxy

↓

Begin Transaction

↓

methodB()

↓

Commit
```

Everything works.

---

## But Self Invocation is different.

Inside the class:

```java
public void methodA(){

    methodB();

}
```

Flow:

```
methodA()

↓

methodB()
```

Notice:

The call never goes through the proxy.

It directly invokes the method on the same object.

Therefore,

Spring never gets a chance to:

* Begin Transaction
* Commit
* Rollback

---

# 4. Internal Working

## Normal Call

```
Client

↓

Spring Proxy

↓

@Transactional

↓

Real Method

↓

Commit
```

---

## Self Invocation

```
Client

↓

Real Object

↓

methodA()

↓

methodB()

(No Proxy)

↓

No Transaction
```

This is the core reason.

---

# 5. Real Example

```java
@Service
public class EmployeeService {

    public void processEmployee(){

        saveEmployee();

    }

    @Transactional
    public void saveEmployee(){

        repository.save(emp);

    }

}
```

Question:

Will `saveEmployee()` execute inside a transaction?

**No.**

Because

```
processEmployee()

↓

saveEmployee()

↓

Direct Call

↓

Proxy Skipped
```

---

# 6. How to Fix It?

There are several approaches.

---

## Solution 1 (Recommended): Move the transactional method to another service

```java
@Service
public class EmployeeService {

    @Autowired
    private SaveService saveService;

    public void processEmployee(){

        saveService.saveEmployee();

    }

}
```

```java
@Service
public class SaveService {

    @Transactional
    public void saveEmployee(){

    }

}
```

Flow:

```
EmployeeService

↓

Spring Proxy

↓

SaveService

↓

Transaction Starts
```

This is the cleanest and most maintainable solution.

---

## Solution 2: Inject the proxied bean into itself

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeService self;

    public void processEmployee(){

        self.saveEmployee();

    }

    @Transactional
    public void saveEmployee(){

    }

}
```

Now the call goes through the proxy.

This works, but it makes the code harder to understand and is generally less preferred than separating responsibilities.

---

## Solution 3: Use `AopContext.currentProxy()`

```java
((EmployeeService) AopContext.currentProxy())
        .saveEmployee();
```

This requires additional configuration and tightly couples your code to Spring AOP.

It's rarely recommended for business code.

---

# 7. Why does Spring use a Proxy?

Spring wants to keep your business logic clean.

The proxy adds:

```
Transaction

Logging

Security

Caching

Metrics
```

without modifying your class.

That's why AOP exists.

---

# 8. Interview Scenario

Consider:

```java
@Service
public class BankService {

    public void transfer(){

        debit();

    }

    @Transactional
    public void debit(){

    }

}
```

Question:

Will `debit()` run in a transaction?

Answer:

**No.**

Because the call is a self invocation and bypasses the Spring proxy.

---

# 9. Internal Comparison

### Works

```
Client

↓

Proxy

↓

@Transactional

↓

Method
```

---

### Doesn't Work

```
Method A

↓

Method B

↓

Direct Call

↓

Proxy Bypassed

↓

No Transaction
```

---

# 10. Common Mistakes

### Mistake 1

```java
methodA(){

    methodB();

}
```

Expecting `methodB()` to be transactional.

❌ Wrong.

---

### Mistake 2

Putting every method in one service class.

Large service classes often increase the chance of self invocation problems.

---

### Mistake 3

Thinking `@Transactional` changes the method itself.

It doesn't.

It changes how the **proxy** invokes the method.

---

# 11. Best Practices

✅ Keep transaction boundaries at the service layer.

✅ Split business responsibilities into multiple services when appropriate.

✅ Let Spring-managed beans call each other through dependency injection.

❌ Don't rely on self-invocation to trigger transactional behavior.

---

# Interview Questions

### Q1. What is Self Invocation?

Self Invocation occurs when a method calls another method in the same class directly.

---

### Q2. Why doesn't `@Transactional` work during Self Invocation?

Because the method call bypasses the Spring AOP proxy, so Spring cannot start or manage the transaction.

---

### Q3. How do you solve the Self Invocation problem?

The preferred solution is to move the transactional method into another Spring-managed bean and call it through dependency injection.

---

### Q4. Is this limitation only for `@Transactional`?

No.

Any feature implemented using Spring's proxy-based AOP can be affected, including:

* `@Transactional`
* `@Async`
* `@Cacheable`
* Method security annotations (such as `@PreAuthorize`)

---

# Summary

```
Client

↓

Spring Proxy

↓

@Transactional

↓

Method

↓

Transaction Starts

-----------------------------

Same Class

↓

methodA()

↓

methodB()

↓

Proxy Skipped

↓

No Transaction
```

---

# 📍 Spring Transaction Module Status

```
Spring Transactions

✅ Why Transactions
✅ ACID
✅ JDBC Transactions
✅ PlatformTransactionManager
✅ @Transactional
✅ Propagation
✅ Isolation Levels
✅ Rollback Rules
✅ readOnly
✅ timeout
✅ Self Invocation

🎉 Module Completed
```

---

# 🎯 Final Interview Question

**Question:**

> "Why does `@Transactional` sometimes not work?"

**Complete Answer:**

`@Transactional` is implemented using **Spring AOP proxies**. A transaction starts only when the method is invoked **through the proxy**. If a method inside the same class directly calls another `@Transactional` method (self invocation), the proxy is bypassed, so Spring never gets a chance to begin or manage the transaction.

---

# 🚀 Next Module: Spring MVC (Most Exciting Part)

Now that we've completed Spring Transactions, we'll begin **Spring MVC**.

We'll follow the same style:

1. Why MVC?
2. Problems before MVC
3. MVC Architecture
4. DispatcherServlet (the heart of Spring MVC)
5. Complete HTTP request lifecycle
6. Controllers
7. View Resolution
8. Form Handling
9. Data Binding
10. Validation
11. Exception Handling
12. Interceptors
13. File Upload
14. Interview Questions

By the end of Spring MVC, you'll understand **exactly how a browser request reaches your controller**, which is the foundation for building REST APIs and Spring Boot applications.
