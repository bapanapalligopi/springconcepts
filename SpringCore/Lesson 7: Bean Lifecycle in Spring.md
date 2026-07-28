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
## Spring Bean Life Cycle (XML Configuration)

The Spring Bean Life Cycle describes what happens to a bean from the moment it is created until it is destroyed.

### Bean Life Cycle Flow

```text
Container Started
       │
       ▼
Bean Instantiation (Constructor)
       │
       ▼
Setter Injection
       │
       ▼
init-method()
       │
       ▼
Bean Ready to Use
       │
       ▼
Application Running
       │
       ▼
destroy-method()
       │
       ▼
Container Closed
```

---

# Step 1: Create Bean Class

```java
package com.practice.sp.models;

public class Employee {

    private int id;
    private String name;

    public Employee() {
        System.out.println("1. Constructor called");
    }

    public void setId(int id) {
        System.out.println("2. Setter Injection for id");
        this.id = id;
    }

    public void setName(String name) {
        System.out.println("3. Setter Injection for name");
        this.name = name;
    }

    // Initialization method
    public void init() {
        System.out.println("4. init() method called");
    }

    // Destroy method
    public void destroy() {
        System.out.println("5. destroy() method called");
    }

    @Override
    public String toString() {
        return id + " " + name;
    }
}
```

---

# Step 2: XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="employee"
          class="com.practice.sp.models.Employee"
          init-method="init"
          destroy-method="destroy">

        <property name="id" value="101"/>
        <property name="name" value="Gopi"/>

    </bean>

</beans>
```

---

# Step 3: Main Class

```java
package com.practice.sp;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.practice.sp.models.Employee;

public class App {

    public static void main(String[] args) {

        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");

        Employee emp = context.getBean("employee", Employee.class);

        System.out.println(emp);

        // Mandatory for destroy-method
        context.close();
    }
}
```

---

# Output

```text
1. Constructor called
2. Setter Injection for id
3. Setter Injection for name
4. init() method called
101 Gopi
5. destroy() method called
```

---

# Execution Order

```text
Container Starts
       │
       ▼
Constructor
       │
       ▼
Setter Injection
       │
       ▼
init-method
       │
       ▼
Bean Ready
       │
       ▼
Business Logic
       │
       ▼
context.close()
       │
       ▼
destroy-method
```

---

# What Happens Internally?

When you execute:

```java
ClassPathXmlApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");
```

Spring performs these steps:

1. Reads `applicationContext.xml`
2. Creates the `Employee` object (constructor)
3. Injects properties (`id`, `name`)
4. Calls `init()`
5. Keeps the bean in the Spring container

When you call:

```java
context.close();
```

Spring:

1. Calls `destroy()`
2. Removes the bean
3. Shuts down the container

---

# Using Default Init and Destroy Methods

Instead of specifying `init-method` and `destroy-method` for every bean, you can configure them globally.

## XML

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd"

       default-init-method="init"
       default-destroy-method="destroy">

    <bean id="employee"
          class="com.practice.sp.models.Employee">

        <property name="id" value="101"/>
        <property name="name" value="Gopi"/>

    </bean>

</beans>
```

Now every bean that has `init()` and `destroy()` methods will use them automatically.

---

# Singleton Bean Life Cycle

```text
Application Starts
       │
       ▼
Constructor
       │
       ▼
Setter Injection
       │
       ▼
init()
       │
       ▼
Bean Used
       │
       ▼
Container Closed
       │
       ▼
destroy()
```

---

# Prototype Bean Life Cycle

If you configure:

```xml
<bean id="employee"
      class="com.practice.sp.models.Employee"
      scope="prototype"
      init-method="init"
      destroy-method="destroy"/>
```

Spring creates a new bean every time you request it.

```text
getBean()
   │
   ▼
Constructor
   │
   ▼
Setter Injection
   │
   ▼
init()
   │
   ▼
Bean Returned
```

**Important:** For **prototype** beans, Spring **does not call** the `destroy-method`. You are responsible for cleanup.

---

# Interview Questions

### 1. What are the stages of the Spring Bean Life Cycle?

1. Bean Instantiation
2. Dependency Injection
3. Initialization (`init-method`)
4. Bean Ready for Use
5. Destruction (`destroy-method`)

---

### 2. Which method is called first?

**Constructor**

---

### 3. Which method is called after dependency injection?

**`init-method`**

---

### 4. Which method is called when the container is closed?

**`destroy-method`**

---

### 5. Is `destroy-method` called for prototype beans?

**No.** Spring does not manage the complete lifecycle of prototype beans after creation.

---

# Quick Revision

| Stage            | Description                              |
| ---------------- | ---------------------------------------- |
| Constructor      | Creates the bean                         |
| Setter Injection | Injects dependencies/properties          |
| `init-method`    | Performs initialization logic            |
| Bean Ready       | Available for application use            |
| `destroy-method` | Performs cleanup before bean destruction |

This XML-based lifecycle (`init-method` and `destroy-method`) is commonly asked in interviews, especially when discussing the evolution from XML configuration to annotation-based configuration in Spring.
