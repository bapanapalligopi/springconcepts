Excellent. 🔥 This is one of the **most frequently asked Spring interview topics**. If you understand this deeply, you'll be able to answer 90% of DI-related interview questions.

---

# Chapter 5: @Qualifier and @Primary (Internal Working)

Let's start with a simple example.

Suppose you have an interface.

```java
public interface EmployeeRepository {

    void save();

}
```

Now two implementations.

```java
@Repository
public class MySqlEmployeeRepository
        implements EmployeeRepository {

    @Override
    public void save() {
        System.out.println("Saving to MySQL");
    }

}
```

```java
@Repository
public class OracleEmployeeRepository
        implements EmployeeRepository {

    @Override
    public void save() {
        System.out.println("Saving to Oracle");
    }

}
```

---

## EmployeeService

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

}
```

Question:

Which repository should Spring inject?

```text
EmployeeRepository

↓

MySqlEmployeeRepository

↓

OracleEmployeeRepository
```

Both satisfy the type.

Spring is confused.

---

# What Happens Internally?

Let's understand Spring's dependency resolution algorithm.

When Spring reaches

```java
@Autowired
private EmployeeRepository repository;
```

It starts searching.

---

## Step 1

Read Field Type

Reflection returns

```text
EmployeeRepository.class
```

---

## Step 2

Search BeanFactory

Spring asks

```text
Give me all beans of type EmployeeRepository
```

BeanFactory returns

```text
mysqlEmployeeRepository

oracleEmployeeRepository
```

Spring now has

```text
List<EmployeeRepository>

↓

mysqlEmployeeRepository

oracleEmployeeRepository
```

---

## Step 3

Count Beans

Number of beans

```text
2
```

Question

Can Spring choose?

No.

Result

```text
NoUniqueBeanDefinitionException
```

Application startup fails.

---

# Exception

```text
No qualifying bean of type
EmployeeRepository

Expected single matching bean

Found 2
```

This is one of the most common Spring interview questions.

---

# Why Doesn't Spring Pick the First One?

Many beginners ask

> Why not inject the first bean?

Because

Imagine tomorrow

```text
Before

MySql

Oracle

----------------

Tomorrow

Oracle

MySql
```

The injection would suddenly change.

Spring must be deterministic.

It refuses to guess.

---

# Solution 1: @Qualifier

Suppose

```java
@Repository
public class MySqlEmployeeRepository
        implements EmployeeRepository {
}
```

Default bean name

```text
mySqlEmployeeRepository
```

Second bean

```text
oracleEmployeeRepository
```

Now

```java
@Service
public class EmployeeService {

    @Autowired

    @Qualifier("mySqlEmployeeRepository")

    private EmployeeRepository repository;

}
```

Spring now knows exactly which bean to inject.

---

# Internal Working

Spring performs

```text
Find EmployeeRepository Beans

↓

MySql

Oracle

↓

Read @Qualifier

↓

mySqlEmployeeRepository

↓

Inject MySql
```

No ambiguity.

---

# How Does Spring Read @Qualifier?

Reflection again.

Imagine

```java
Field field =
EmployeeService.class.getDeclaredField("repository");
```

Spring checks

```java
field.isAnnotationPresent(Qualifier.class)
```

Result

```text
true
```

Now Spring reads

```java
Qualifier qualifier =
field.getAnnotation(Qualifier.class);
```

Returns

```text
mySqlEmployeeRepository
```

Spring searches

```text
BeanFactory

↓

mySqlEmployeeRepository
```

Found.

Inject.

---

# Visual Flow

```text
@Autowired

↓

Field Type

↓

EmployeeRepository

↓

2 Beans Found

↓

Read @Qualifier

↓

mySqlEmployeeRepository

↓

Inject
```

---

# Solution 2: @Primary

Suppose

```java
@Repository

@Primary

public class MySqlEmployeeRepository
implements EmployeeRepository{

}
```

Second bean

```java
@Repository
public class OracleEmployeeRepository
implements EmployeeRepository{

}
```

EmployeeService

```java
@Autowired
private EmployeeRepository repository;
```

No qualifier.

Spring performs

```text
EmployeeRepository

↓

MySql

Primary

↓

Inject
```

---

# Internal Algorithm

Spring searches

```text
EmployeeRepository

↓

MySql

Primary = YES

↓

Oracle

Primary = NO
```

One primary bean exists.

Inject it.

---

# Difference

### @Qualifier

Developer explicitly chooses.

```java
@Qualifier("oracleEmployeeRepository")
```

Spring follows your instruction.

---

### @Primary

Bean itself says

```text
Choose me

If nobody specifies

@Qualifier
```

---

# What If Both Exist?

Suppose

```java
@Repository

@Primary
public class MySqlRepository{}
```

```java
@Autowired

@Qualifier("oracleRepository")

private EmployeeRepository repository;
```

Question

Which wins?

Answer

```text
@Qualifier
```

Always.

Why?

Because

Developer explicitly requested

```text
oracleRepository
```

Spring respects explicit configuration over defaults.

---

# Complete Resolution Algorithm

This is almost exactly what Spring does.

```text
Need EmployeeRepository

↓

Find Matching Beans

↓

0 Beans ?

↓

NoSuchBeanDefinitionException

↓

1 Bean ?

↓

Inject

↓

Multiple Beans ?

↓

Check @Qualifier

↓

Found ?

↓

Inject

↓

No Qualifier

↓

Check @Primary

↓

One Primary ?

↓

Inject

↓

Still Multiple ?

↓

Throw Exception
```

This flow is extremely important.

---

# Real Example

Imagine

```text
PaymentGateway

↓

PhonePe

↓

GooglePay

↓

Paytm

↓

Stripe
```

Your service

```java
@Autowired
private PaymentGateway gateway;
```

Spring finds

```text
4 Beans
```

Impossible.

Now

```java
@Qualifier("stripeGateway")
```

Problem solved.

---

# Bean Names

Default bean names

```java
@Repository
public class MySqlEmployeeRepository{
}
```

↓

```text
mySqlEmployeeRepository
```

---

```java
@Repository
public class OracleEmployeeRepository{
}
```

↓

```text
oracleEmployeeRepository
```

---

Custom bean

```java
@Repository("mysql")
```

Now bean name

```text
mysql
```

Qualifier becomes

```java
@Qualifier("mysql")
```

---

# Interview Questions

## Question 1

Why does Spring throw

```text
NoUniqueBeanDefinitionException
```

Answer

Because multiple beans match the required type and Spring cannot determine which one should be injected.

---

## Question 2

Difference between

```java
@Primary
```

and

```java
@Qualifier
```

| @Primary                      | @Qualifier                 |
| ----------------------------- | -------------------------- |
| Applied on bean               | Applied at injection point |
| Default choice                | Explicit choice            |
| Used when no qualifier exists | Overrides @Primary         |

---

## Question 3

Priority Order

Spring resolves dependencies in this order:

```text
1. @Qualifier

↓

2. @Primary

↓

3. Single Bean

↓

4. Exception
```

---

# Internal Classes Involved

When Spring processes

```java
@Autowired
@Qualifier("mysql")
private EmployeeRepository repository;
```

The flow is:

```text
AutowiredAnnotationBeanPostProcessor

↓

DependencyDescriptor

↓

DefaultListableBeanFactory

↓

findAutowireCandidates()

↓

QualifierAnnotationAutowireCandidateResolver

↓

Resolve Bean

↓

Reflection Injection
```

These are real Spring Framework classes.

---

# Spring Source Code Flow (Simplified)

```java
Field field = repository;

DependencyDescriptor descriptor =
new DependencyDescriptor(field);

beanFactory.resolveDependency(descriptor);
```

Inside

```java
resolveDependency()
```

Spring performs

```text
Find Candidate Beans

↓

Filter by Type

↓

Apply Qualifier

↓

Apply Primary

↓

Choose Bean

↓

Return Bean
```

This is why Spring's DI is so flexible.

---

# Summary

```text
@Autowired
        │
        ▼
Read Field Type
        │
        ▼
Find Candidate Beans
        │
        ▼
One Bean?
        │
   Yes ─────────► Inject
        │
        ▼
Multiple Beans
        │
        ▼
Check @Qualifier
        │
   Found ───────► Inject
        │
        ▼
Check @Primary
        │
   Found ───────► Inject
        │
        ▼
Throw NoUniqueBeanDefinitionException
```

---

# Next Topic (One of the Most Important)

Now comes a topic that even experienced developers often struggle with:

## Constructor Injection vs Setter Injection vs Field Injection

We'll cover:

* How Spring chooses a constructor
* Why Spring recommends constructor injection
* What happens internally during constructor injection
* Circular dependency issues
* Why field injection can make unit testing harder
* Best practices in real-world Spring Boot applications
* Spring 4.3+ behavior (why `@Autowired` on a single constructor is optional)

This topic is extremely common in Spring Boot interviews because it tests not only your knowledge of dependency injection but also your understanding of good software design principles.
