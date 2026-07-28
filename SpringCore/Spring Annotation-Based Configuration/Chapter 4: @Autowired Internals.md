Excellent! 🔥 Now we reach the **most important topic in Spring Framework**.

I can confidently say that **70% of Spring interview questions revolve around Dependency Injection and `@Autowired`**.

Most developers know **how to use** `@Autowired`.

Very few know **how Spring actually implements it internally**.

Today we'll learn it from Spring's source code perspective.

---

# Chapter 4: @Autowired Internals

---

# First Question

Consider this code.

```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

}
```

Question:

Who assigns

```java
repository = new EmployeeRepository();
```

Did Java do it?

❌ No

Did JVM do it?

❌ No

Did Compiler do it?

❌ No

Then who?

Answer:

**Spring Framework**

Now the next question is

> **How does Spring know this field needs injection?**

---

# Let's Start From Application Startup

Suppose

```java
@ComponentScan("com.practice")
```

Spring starts.

```text
Application Starts
        │
        ▼
Component Scan
        │
        ▼
Create BeanDefinitions
        │
        ▼
Create Objects
```

Suppose Spring creates

```java
EmployeeService service =
        new EmployeeService();
```

Current object

```text
EmployeeService

repository = null
```

Notice

Nothing has been injected.

---

# Why?

Because

```java
new EmployeeService()
```

calls only the constructor.

Java doesn't know anything about

```java
@Autowired
```

Java simply allocates memory.

```text
Heap

EmployeeService

repository = null
```

---

# Then How Does Injection Happen?

Spring has a special class.

Its name is

```text
AutowiredAnnotationBeanPostProcessor
```

Remember this name.

It is one of the most important classes in Spring.

---

# What is BeanPostProcessor?

Suppose Spring creates

```java
EmployeeService service =
        new EmployeeService();
```

Immediately after creation,

Spring says

```text
Wait...

Before using this bean,

Let me inspect it.
```

This inspection is done by

```text
BeanPostProcessor
```

Think of it like an airport security check.

```text
Passenger

↓

Security Check

↓

Board Flight
```

Similarly

```text
Bean Created

↓

BeanPostProcessor

↓

Bean Ready
```

---

# Internal Flow

```text
new EmployeeService()

↓

BeanPostProcessor

↓

Check @Autowired

↓

Inject Dependency

↓

Ready
```

---

# What Does BeanPostProcessor Do?

Imagine Spring code.

```java
Object bean = new EmployeeService();

process(bean);
```

Inside

```java
process(bean);
```

Spring does something like

```java
Class<?> cls = bean.getClass();
```

Suppose

```java
EmployeeService.class
```

Now it scans every field.

```java
for(Field field : cls.getDeclaredFields()){

}
```

Suppose fields

```java
private EmployeeRepository repository;

private String name;
```

Spring checks

```java
field.isAnnotationPresent(Autowired.class)
```

For

```java
repository
```

Result

```text
true
```

For

```java
name
```

Result

```text
false
```

Only annotated fields are processed.

---

# Reflection Again

Spring now has

```java
Field field
```

representing

```java
private EmployeeRepository repository;
```

Question

Can Java normally access

```java
private
```

field?

No.

But Reflection can.

Spring performs

```java
field.setAccessible(true);
```

Now private restriction is bypassed.

---

# Dependency Resolution

Spring now asks

```text
What is the field type?
```

Reflection returns

```text
EmployeeRepository.class
```

Now Spring searches

```text
Singleton Cache

↓

EmployeeRepository ?
```

Suppose

```text
EmployeeRepository@205
```

exists.

Spring gets it.

---

# Injection

Reflection performs

```java
field.set(bean, repositoryObject);
```

Equivalent to

```java
service.repository = repositoryObject;
```

even though

```java
repository
```

is private.

Reflection makes it possible.

---

# Final Object

Before

```text
EmployeeService

repository = null
```

After

```text
EmployeeService

repository

↓

EmployeeRepository@205
```

Injection completed.

---

# Complete Internal Algorithm

Spring roughly performs

```java
EmployeeService service =
        new EmployeeService();

for(Field field : service.getClass().getDeclaredFields()){

    if(field.hasAnnotation(Autowired.class)){

        Object dependency =
            beanFactory.getBean(field.getType());

        field.setAccessible(true);

        field.set(service, dependency);

    }

}
```

The real implementation is much more complex, but this is the core idea.

---

# Visual Flow

```text
EmployeeService

↓

Constructor

↓

repository = null

↓

BeanPostProcessor

↓

Reflection

↓

Find @Autowired

↓

Search BeanFactory

↓

EmployeeRepository Found

↓

Reflection Injection

↓

Bean Ready
```

---

# But Wait...

How does

```java
beanFactory.getBean(EmployeeRepository.class)
```

find the correct bean?

This is called **Dependency Resolution**.

Spring follows a resolution algorithm.

Let's understand it.

---

# Dependency Resolution Algorithm

Suppose

```java
@Autowired
private EmployeeRepository repository;
```

Step 1

Look at field type.

```text
EmployeeRepository
```

Step 2

Search BeanFactory.

```text
EmployeeRepository
```

Step 3

Found exactly one bean.

```text
EmployeeRepository@205
```

Step 4

Inject it.

Done.

---

# What If Bean Doesn't Exist?

Suppose

```java
@Autowired
private EmployeeRepository repository;
```

No bean exists.

Spring searches

```text
BeanFactory

↓

Nothing
```

Result

```text
NoSuchBeanDefinitionException
```

Application startup fails.

---

# What If Two Beans Exist?

Suppose

```java
@Repository
class MySqlRepository{}
```

```java
@Repository
class OracleRepository{}
```

Both implement

```java
EmployeeRepository
```

Now Spring searches

```text
EmployeeRepository

↓

MySqlRepository

↓

OracleRepository
```

Question

Which one?

Spring doesn't know.

Result

```text
NoUniqueBeanDefinitionException
```

This is why

```java
@Qualifier
```

exists.

We'll study that next.

---

# Constructor Injection

Now consider

```java
@Service
public class EmployeeService {

    private EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository){

        this.repository = repository;

    }

}
```

Question

Does Spring use Reflection to set the field?

No.

Instead,

Spring first resolves

```text
EmployeeRepository
```

Then directly calls

```java
new EmployeeService(repository);
```

No need for

```java
field.set(...)
```

This is one reason constructor injection is preferred.

---

# Field Injection vs Constructor Injection

### Field Injection

```java
@Autowired
private EmployeeRepository repository;
```

Process

```text
Create Object

↓

Reflection

↓

Inject Field
```

---

### Constructor Injection

```java
public EmployeeService(EmployeeRepository repository)
```

Process

```text
Resolve Dependency

↓

Call Constructor

↓

Object Ready
```

No reflection is needed to inject the dependency after construction.

---

# Why Constructor Injection is Recommended

Spring team recommends constructor injection because:

✅ Dependencies are mandatory

```java
public EmployeeService(EmployeeRepository repository)
```

The object cannot exist without its dependency.

---

Immutable fields

```java
private final EmployeeRepository repository;
```

Possible only with constructor injection.

---

Easy unit testing

```java
EmployeeRepository repo = mock(...);

EmployeeService service =
        new EmployeeService(repo);
```

No Spring required.

---

No reflection-based field modification after construction.

---

# Interview Questions

## Q1. Who processes `@Autowired`?

**Answer:**

`AutowiredAnnotationBeanPostProcessor` processes `@Autowired` annotations after a bean is instantiated. It uses reflection to detect annotated fields, methods, or constructors, resolves the required dependency from the Spring container, and injects it.

---

## Q2. Does Java understand `@Autowired`?

No.

Java simply stores annotation metadata in the compiled class file.

Spring reads that metadata at runtime using Reflection.

---

## Q3. Does `@Autowired` work without Spring?

No.

Without a Spring container, the annotation has no effect.

Example:

```java
EmployeeService service = new EmployeeService();

System.out.println(service.repository);
```

Output

```text
null
```

because no Spring container processed the annotation.

---

## Q4. Which API does Spring use to inject private fields?

Java Reflection.

Specifically, it makes the field accessible and assigns the dependency programmatically.

---

# Complete Picture So Far

```text
@Component
        │
        ▼
Component Scan
        │
        ▼
BeanDefinition
        │
        ▼
BeanFactory
        │
        ▼
Create Object
        │
        ▼
BeanPostProcessor
        │
        ▼
AutowiredAnnotationBeanPostProcessor
        │
        ▼
Reflection
        │
        ▼
Dependency Resolution
        │
        ▼
Inject Dependency
        │
        ▼
@PostConstruct
        │
        ▼
Bean Ready
```

---

# Next Chapter (One of the Most Interesting)

Now comes the question every Spring developer eventually faces:

```java
@Repository
class MySqlEmployeeRepository implements EmployeeRepository {}
```

```java
@Repository
class OracleEmployeeRepository implements EmployeeRepository {}
```

```java
@Autowired
private EmployeeRepository repository;
```

There are **two beans of the same type**.

How does Spring decide which one to inject?

This leads us to:

* `@Qualifier`
* `@Primary`
* Dependency resolution precedence
* Internal bean selection algorithm
* How Spring resolves ambiguity

This topic is asked in almost every Spring Boot interview because it demonstrates whether you understand dependency injection beyond the basics.
