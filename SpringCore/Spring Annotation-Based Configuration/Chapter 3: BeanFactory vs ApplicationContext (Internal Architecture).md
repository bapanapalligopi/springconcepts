Excellent. Now we're going to learn **how Spring is architected internally**. This is where many developers stop, but if you understand this, you'll be able to explain **how Spring works under the hood**, not just how to use it.

---

# Chapter 3: BeanFactory vs ApplicationContext (Internal Architecture)

Most interview answers are like this:

> **BeanFactory → Lazy loading**
>
> **ApplicationContext → Eager loading**

This is **correct but incomplete**. That's only one difference.

To really understand Spring, we need to know **how these interfaces are related**.

---

# Spring Container Architecture

Look at this hierarchy.

```text
                 BeanFactory (Interface)
                        ▲
                        │
          ConfigurableBeanFactory
                        ▲
                        │
       ListableBeanFactory
                        ▲
                        │
      ConfigurableListableBeanFactory
                        ▲
                        │
        DefaultListableBeanFactory
```

And for `ApplicationContext`:

```text
                 BeanFactory
                       ▲
                       │
             ApplicationContext
                       ▲
                       │
       ConfigurableApplicationContext
                       ▲
                       │
    AbstractApplicationContext
                       ▲
                       │
 GenericApplicationContext
                       ▲
                       │
AnnotationConfigApplicationContext
```

Notice something important:

> **ApplicationContext is also a BeanFactory.**

This is because

```java
public interface ApplicationContext
        extends BeanFactory {
}
```

So whenever you use

```java
ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

you are actually using a much more powerful version of `BeanFactory`.

---

# What is BeanFactory?

Think of `BeanFactory` as the **basic IoC container**.

Its responsibilities are:

* Store `BeanDefinition`
* Create beans
* Inject dependencies
* Return beans

Nothing more.

Conceptually:

```text
BeanFactory

↓

BeanDefinition

↓

Object Creation

↓

getBean()
```

---

# Internal Structure

Imagine you write

```java
Employee employee =
        context.getBean(Employee.class);
```

Internally, `BeanFactory` does something like:

```java
BeanDefinition definition =
        beanDefinitionMap.get("employee");

Object object =
        createBean(definition);

return object;
```

The actual implementation is much more sophisticated, but this is the core idea.

---

# Why Was BeanFactory Introduced?

Back in 2003, memory was limited.

Imagine a project with

```text
500 Beans
```

Creating all 500 objects at startup could be expensive.

So Spring introduced **lazy initialization**.

Flow:

```text
Application Starts

↓

BeanDefinitions Loaded

↓

No Objects Created

↓

getBean()

↓

Create Object

↓

Return Object
```

Only the requested bean is created.

---

# Example

Suppose

```java
@Component
public class Employee {
    public Employee() {
        System.out.println("Employee Created");
    }
}
```

Using `BeanFactory`:

```java
BeanFactory factory = ...
```

Application starts.

Output:

```text
Nothing
```

Now

```java
factory.getBean(Employee.class);
```

Output:

```text
Employee Created
```

The object is created only when requested.

---

# Problem with BeanFactory

Imagine a web application.

First user opens the application.

```text
User Request

↓

Need EmployeeService

↓

Create EmployeeService

↓

Need Repository

↓

Create Repository

↓

Need DataSource

↓

Create DataSource
```

The first request becomes slower because Spring has to create multiple beans.

---

# ApplicationContext Solves This

`ApplicationContext` creates singleton beans during startup.

Flow:

```text
Application Starts

↓

Read BeanDefinitions

↓

Create Singleton Beans

↓

Dependency Injection

↓

@PostConstruct

↓

Application Ready
```

When a request comes:

```java
context.getBean(Employee.class);
```

Spring simply returns the already-created object.

---

# Internal Cache

Spring maintains an internal cache for singleton beans.

Conceptually:

```text
Singleton Cache

employee

↓

Employee@102

repository

↓

Repository@205

service

↓

Service@309
```

When you call

```java
context.getBean(Employee.class);
```

Spring first checks the singleton cache.

```text
Found?

↓

Yes

↓

Return Object
```

No new object is created.

---

# How Does Spring Decide?

Suppose

```java
@Component
public class Employee {
}
```

By default

```java
@Scope("singleton")
```

is implied.

During startup

Spring sees

```text
Scope

↓

Singleton

↓

Create Immediately
```

If instead

```java
@Component
@Scope("prototype")
public class Employee {
}
```

Spring stores only the `BeanDefinition`.

Objects are created each time `getBean()` is called.

---

# BeanFactory Internals

The real implementation used by Spring is

```text
DefaultListableBeanFactory
```

This class is extremely important.

Responsibilities:

* Store BeanDefinitions
* Register BeanDefinitions
* Create Beans
* Resolve Dependencies
* Store Singleton Objects

Everything revolves around it.

---

# BeanDefinitionRegistry

Another important interface is

```text
BeanDefinitionRegistry
```

Its job is simple.

Register BeanDefinitions.

Conceptually:

```java
registerBeanDefinition(
    "employee",
    employeeDefinition
);
```

Now the container knows:

> "There is a bean called `employee`."

No object exists yet.

---

# DefaultListableBeanFactory

This class combines multiple responsibilities.

```text
DefaultListableBeanFactory

↓

BeanDefinitionRegistry

↓

BeanFactory

↓

Singleton Registry

↓

Dependency Resolver
```

It is the core implementation behind most Spring containers.

---

# ApplicationContext Internals

`ApplicationContext` builds on top of `BeanFactory` and adds enterprise features.

Additional capabilities include:

* Internationalization (i18n)
* Event publishing
* Resource loading
* Environment and property resolution
* Automatic registration of BeanPostProcessors
* Automatic invocation of lifecycle callbacks

So conceptually:

```text
ApplicationContext

↓

BeanFactory

+

Events

+

Resources

+

Environment

+

MessageSource

+

Lifecycle Management
```

---

# Startup Sequence

Let's trace what happens when you create an `ApplicationContext`.

```java
ApplicationContext context =
new AnnotationConfigApplicationContext(AppConfig.class);
```

Step 1

```text
Read Configuration
```

↓

Step 2

```text
Component Scan
```

↓

Step 3

```text
Create BeanDefinitions
```

↓

Step 4

```text
Register BeanDefinitions
```

↓

Step 5

```text
Instantiate Singleton Beans
```

↓

Step 6

```text
Inject Dependencies
```

↓

Step 7

```text
Execute BeanPostProcessors
```

↓

Step 8

```text
Call @PostConstruct
```

↓

Step 9

```text
Store Bean in Singleton Cache
```

↓

Application Ready.

---

# Why Does Spring Need BeanPostProcessor?

Suppose

```java
@Service
public class EmployeeService {

}
```

How does Spring process

```java
@Autowired

@Value

@PostConstruct

@PreDestroy
```

These annotations don't work by themselves.

Spring has special classes called **BeanPostProcessors** that inspect beans after creation and before they're fully initialized, applying behaviors like dependency injection and lifecycle callbacks.

We'll study them in depth later because they are the foundation of many Spring features.

---

# BeanFactory vs ApplicationContext

| Feature                  | BeanFactory                             | ApplicationContext                   |
| ------------------------ | --------------------------------------- | ------------------------------------ |
| IoC Container            | ✅                                       | ✅                                    |
| Bean Creation            | Lazy by default                         | Eager for singleton beans by default |
| Dependency Injection     | ✅                                       | ✅                                    |
| BeanPostProcessors       | Must typically be registered explicitly | Automatically detected and applied   |
| Events                   | ❌                                       | ✅                                    |
| Internationalization     | ❌                                       | ✅                                    |
| Resource Loading         | Limited                                 | ✅                                    |
| Environment & Properties | Limited                                 | ✅                                    |

---

# Interview Questions

### 1. Is ApplicationContext a BeanFactory?

Yes.

`ApplicationContext` extends `BeanFactory` and provides all of its functionality along with additional enterprise features.

---

### 2. Which implementation does Spring commonly use?

`DefaultListableBeanFactory` is the core implementation responsible for managing bean definitions, creating beans, resolving dependencies, and maintaining singleton instances.

---

### 3. Why don't we use BeanFactory directly anymore?

Because `ApplicationContext` provides all `BeanFactory` features plus additional capabilities such as automatic BeanPostProcessor registration, event publishing, resource loading, environment abstraction, and better integration with modern Spring applications.

---

# This Leads to the Most Important Topic

Everything we've learned so far builds up to one question:

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

}
```

How does Spring actually inject `repository`?

It isn't Java syntax.

It isn't the JVM.

It isn't the compiler.

So **who sees `@Autowired`?**

The answer is:

```text
Bean Created

↓

AutowiredAnnotationBeanPostProcessor

↓

Reflection

↓

Find @Autowired Fields

↓

Resolve Dependency

↓

Inject Object

↓

Bean Ready
```

This is the internal mechanism behind dependency injection.

---

## Next Chapter (One of the Most Important in Spring)

We'll spend significant time on **`@Autowired` Internals**, covering:

* How `AutowiredAnnotationBeanPostProcessor` works
* Field injection vs constructor injection vs setter injection
* Dependency resolution by type
* How `@Qualifier` resolves ambiguity
* How `@Primary` works
* Circular dependencies
* Why constructor injection is generally preferred in modern Spring applications
* The exact algorithm Spring uses to resolve dependencies

Once you understand that chapter, you'll know **how dependency injection really works**, not just how to write `@Autowired`.
