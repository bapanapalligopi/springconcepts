Awesome! You're doing great. Now we'll learn **Bean Lifecycle**, which is one of the most important Spring Core concepts. Once you understand this, `@PostConstruct` and `@PreDestroy` in Spring Boot will be very easy.

---

# Lesson 7: Bean Lifecycle in Spring

## What is Bean Lifecycle?

The **Bean Lifecycle** describes **what happens to a bean from the moment Spring creates it until it is destroyed.**

Think of a bean as an employee joining and leaving a company.

```text
Employee

↓

Joining

↓

Working

↓

Leaving
```

Similarly, a Spring bean goes through several phases.

---

# Complete Bean Lifecycle

```text
                Spring Container

                      │
                      ▼
              Read XML Configuration
                      │
                      ▼
               Instantiate Bean
                      │
                      ▼
          Inject Dependencies (DI)
                      │
                      ▼
              init-method() Called
                      │
                      ▼
              Bean Ready to Use
                      │
          getBean() returns Bean
                      │
                      ▼
      Container Shuts Down
                      │
                      ▼
           destroy-method() Called
```

---

# Step 1: Bean Creation

Employee.java

```java
package com.practice.springcore;

public class Employee {

    public Employee() {
        System.out.println("Constructor Called");
    }
}
```

When Spring reads

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"/>
```

it internally does

```java
Employee emp = new Employee();
```

Output

```text
Constructor Called
```

---

# Step 2: Dependency Injection

Suppose

```java
public class Employee {

    private Address address;

    public void setAddress(Address address) {
        this.address = address;
    }

}
```

Spring performs

```java
Employee emp = new Employee();

emp.setAddress(address);
```

At this point

```text
Bean Created

↓

Dependencies Injected
```

---

# Step 3: Initialization

Sometimes after dependencies are injected, we want to perform some initialization.

Example:

* Open a file
* Validate data
* Load configuration
* Establish a database connection

Spring allows this using **init-method**.

---

# Example

Employee.java

```java
package com.practice.springcore;

public class Employee {

    public Employee() {
        System.out.println("Constructor Called");
    }

    public void init() {
        System.out.println("Init Method Called");
    }

}
```

---

XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"
      init-method="init"/>
```

---

Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

Output

```text
Constructor Called
Init Method Called
```

Notice

Spring automatically calls

```java
init();
```

You never call it.

---

# What Happens Internally?

Spring does something similar to

```java
Employee emp = new Employee();

emp.init();
```

---

# Step 4: Bean is Ready

Now

```java
Employee emp =
        context.getBean("employee", Employee.class);
```

Spring simply returns the already initialized bean.

---

# Step 5: Destroy Method

When the application closes,

Spring can call another method.

Example

```java
public void destroy() {

    System.out.println("Destroy Method Called");

}
```

XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee"
      init-method="init"
      destroy-method="destroy"/>
```

---

Main

```java
ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

Employee emp =
        context.getBean("employee", Employee.class);

context.close();
```

Output

```text
Constructor Called

Init Method Called

Destroy Method Called
```

Notice

If you don't call

```java
context.close();
```

the destroy method won't execute.

---

# Why use init-method?

Suppose you want to

```text
Connect Database

Open File

Read Configuration

Initialize Cache

Validate Data
```

This is the perfect place.

---

# Why use destroy-method?

Before bean destruction

```text
Close Database Connection

Close File

Release Socket

Clear Cache

Stop Threads
```

---

# Complete Flow

```text
Spring Starts
      │
      ▼
new Employee()
      │
      ▼
Dependency Injection
      │
      ▼
init()
      │
      ▼
Bean Ready
      │
      ▼
Application Running
      │
      ▼
context.close()
      │
      ▼
destroy()
```

---

# Singleton vs Prototype

### Singleton

```text
Create Bean

↓

Init

↓

Use

↓

Destroy
```

Spring manages the complete lifecycle.

---

### Prototype

```text
Create Bean

↓

Init

↓

Return Bean

↓

Spring Stops Managing It
```

The **destroy-method is NOT called automatically** for prototype beans.

This is a very common interview question.

---

# Interview Questions

### 1. What is Bean Lifecycle?

**Answer**

The Bean Lifecycle is the sequence of steps a bean goes through inside the Spring container, including instantiation, dependency injection, initialization, usage, and destruction.

---

### 2. What is init-method?

A custom method that Spring calls **after dependency injection** and **before the bean is used**.

---

### 3. What is destroy-method?

A custom method that Spring calls **before destroying a singleton bean** when the container shuts down.

---

### 4. Does destroy-method execute for prototype beans?

**Answer**

No.

Spring creates prototype beans but does not manage their complete lifecycle after returning them.

---

# XML Attributes Learned So Far

```xml
<bean id="employee"
      class="com.practice.Employee"
      scope="singleton"
      init-method="init"
      destroy-method="destroy"/>
```

Now you know what each attribute does:

* `id` → Bean name
* `class` → Bean class
* `scope` → Bean scope
* `init-method` → Initialization callback
* `destroy-method` → Destruction callback

---

# Behind the Scenes

For a singleton bean, Spring performs operations conceptually like this:

```java
Employee emp = new Employee();   // Constructor

emp.setAddress(address);         // Dependency Injection

emp.init();                      // Initialization

// Bean is now ready

...

emp.destroy();                   // Cleanup before shutdown
```

---

# Summary So Far

You've now covered the core XML-based Spring concepts:

* ✅ What is Spring Framework
* ✅ IoC (Inversion of Control)
* ✅ Dependency Injection (DI)
* ✅ BeanFactory vs ApplicationContext
* ✅ XML Bean Configuration
* ✅ Constructor Injection
* ✅ Setter Injection
* ✅ Collection Injection (`List`, `Set`, `Map`, `Properties`, `Array`)
* ✅ Bean Scopes (`singleton`, `prototype`)
* ✅ Bean Lifecycle (`init-method`, `destroy-method`)

---

# Next Lesson (Most Important Before Annotations)

We'll learn **Autowiring in XML**, where Spring automatically injects dependencies without explicitly writing `<property>` or `<constructor-arg>` for every bean.

You'll understand:

* `autowire="byName"`
* `autowire="byType"`
* `autowire="constructor"`
* `autowire="no"` (default)

Once you master XML autowiring, annotations like `@Autowired` will feel natural because you'll know exactly what Spring is doing behind the scenes.
