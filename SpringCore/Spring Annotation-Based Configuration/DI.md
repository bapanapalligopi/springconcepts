Perfect. We'll continue exactly where we stopped, but at the **1.5–2 years experience level**.

So far you've learned:

* ✅ IoC
* ✅ Dependency Injection
* ✅ XML Configuration
* ✅ Annotation Configuration
* ✅ `@Component`
* ✅ `@ComponentScan`
* ✅ `@Autowired`
* ✅ `@Qualifier`
* ✅ `@Primary`

The next topic is one of the **most frequently asked interview questions**.

---

# Constructor Injection vs Setter Injection vs Field Injection

Before learning the differences, let's understand **why multiple injection types exist**.

Suppose you have:

```java
@Service
public class EmployeeService {

    private EmployeeRepository repository;

}
```

The question is:

**How can Spring put the `EmployeeRepository` object into `EmployeeService`?**

There are only **three ways**.

```
EmployeeService
       │
       ▼
1. Constructor
2. Setter Method
3. Field
```

Let's study each one.

---

# 1. Field Injection

This is probably what you've seen most often.

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

Spring creates the object first.

```
EmployeeService

repository = null
```

Then Spring injects the dependency.

```
EmployeeService

repository = EmployeeRepository Object
```

### Advantages

* Very little code
* Easy to understand
* Common in older projects

### Disadvantages

You cannot create the object properly without Spring.

```java
EmployeeService service = new EmployeeService();
```

Now

```java
service.repository
```

is

```
null
```

If you call

```java
service.repository.save();
```

you get

```
NullPointerException
```

Another issue:

The dependency is hidden.

When someone looks at

```java
public class EmployeeService {
}
```

they don't immediately know what dependencies it needs.

---

# 2. Setter Injection

Example

```java
@Service
public class EmployeeService {

    private EmployeeRepository repository;

    @Autowired
    public void setRepository(EmployeeRepository repository) {
        this.repository = repository;
    }
}
```

Flow

```
Spring creates EmployeeService

↓

Calls setRepository()

↓

Dependency injected
```

### Advantages

* Dependency can be changed later.
* Useful for **optional dependencies**.

Example:

```java
@Autowired(required = false)
public void setEmailService(EmailService emailService) {
    this.emailService = emailService;
}
```

If no `EmailService` bean exists, Spring can simply skip this injection (because `required = false`).

### Disadvantages

Someone can later change the dependency.

```java
service.setRepository(null);
```

Now your object becomes invalid.

---

# 3. Constructor Injection ⭐ (Recommended)

Example

```java
@Repository
public class EmployeeRepository {

}
```

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

}
```

Notice two things.

The field is

```java
final
```

which means

```
Once assigned

↓

Cannot change
```

Also,

without `EmployeeRepository`

you **cannot create** `EmployeeService`.

```java
new EmployeeService();
```

Compilation error.

Because Java forces you to pass

```java
EmployeeRepository
```

This guarantees the object is always in a valid state.

---

# Spring 4.3+

If a class has **only one constructor**, you don't even need `@Autowired`.

```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

}
```

Spring automatically uses that constructor.

If there are multiple constructors, then use `@Autowired` to tell Spring which one to use.

Example

```java
public EmployeeService() {

}

@Autowired
public EmployeeService(EmployeeRepository repository) {
    this.repository = repository;
}
```

---

# Real Project Example

Imagine an Order Service.

```java
@Service
public class OrderService {

    private final OrderRepository repository;
    private final PaymentService paymentService;
    private final EmailService emailService;

    public OrderService(OrderRepository repository,
                        PaymentService paymentService,
                        EmailService emailService) {

        this.repository = repository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
}
```

Just by looking at the constructor, you immediately know the class depends on:

* OrderRepository
* PaymentService
* EmailService

This makes the code easier to understand and maintain.

---

# Comparison

| Feature                | Field | Setter    | Constructor        |
| ---------------------- | ----- | --------- | ------------------ |
| Easy to write          | ✅     | ✅         | Slightly more code |
| Dependency mandatory   | ❌     | ❌         | ✅                  |
| Can use `final` fields | ❌     | ❌         | ✅                  |
| Easy for unit testing  | ❌     | Moderate  | ✅                  |
| Spring recommendation  | ❌     | Sometimes | ✅                  |

---

# Which One Should You Use?

### Use Constructor Injection

When the dependency is **required**.

Example:

```java
UserService

↓

UserRepository
```

Without `UserRepository`, `UserService` cannot work.

Constructor injection is the best choice.

---

### Use Setter Injection

When the dependency is **optional**.

Example:

```
NotificationService

↓

SMS Service (optional)
```

The application can still run without it.

---

### Avoid Field Injection

It still works and you'll see it in many existing codebases, but for new code constructor injection is generally preferred because it makes dependencies explicit, supports immutable (`final`) fields, and is easier to test.

---

# Interview Questions

### 1. Which injection type does Spring recommend?

**Answer:** Constructor Injection.

---

### 2. Why is Constructor Injection preferred?

Because:

* Mandatory dependencies are guaranteed.
* Supports immutable (`final`) fields.
* Easier to unit test.
* Dependencies are clearly visible.

---

### 3. When do you use Setter Injection?

For optional dependencies or when the dependency may need to change after object creation.

---

### 4. Is `@Autowired` mandatory on constructors?

No.

From **Spring 4.3 onwards**, if there is only **one constructor**, Spring automatically uses it.

---

# Real Interview Answer (1.5 Years Experience)

**Interviewer:** *Which dependency injection type have you used in your project?*

**Good Answer:**

> "We primarily use constructor injection because it is the recommended approach in Spring. It makes dependencies mandatory, allows us to use `final` fields, improves testability, and makes the class dependencies explicit. I've seen field injection in older code, but for new development we prefer constructor injection."

This is exactly the kind of answer expected from someone with **1.5–2 years of Spring Boot experience**.

---

## Next Topic

The next Spring Core topic is **Bean Scope**, where we'll learn:

* Singleton (default scope)
* Prototype
* Request Scope
* Session Scope
* Application Scope
* Why Spring beans are singleton by default
* Singleton bean with a prototype dependency (a common interview question)
* Real-world examples of each scope

This is another topic that appears frequently in Spring Boot interviews.
