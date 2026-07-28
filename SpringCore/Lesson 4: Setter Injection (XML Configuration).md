Awesome! You're learning the right way. Before annotations, every Spring developer should understand **Setter Injection**.

---

# Lesson 4: Setter Injection (XML Configuration)

## What is Setter Injection?

Instead of injecting dependencies through the constructor, Spring injects them by calling **setter methods**.

Without Spring:

```java
Engine engine = new Engine();

Car car = new Car();
car.setEngine(engine);
```

With Spring:

```text
Spring Container

Creates Engine
      │
      ▼
Creates Car
      │
      ▼
Calls setEngine(engine)
```

---

# Example 1: Object Injection

## Step 1: Engine.java

```java
package com.practice.springcore;

public class Engine {

    public Engine() {
        System.out.println("Engine Created");
    }

    public void start() {
        System.out.println("Engine Started");
    }

}
```

---

## Step 2: Car.java

Notice there is **NO constructor**.

```java
package com.practice.springcore;

public class Car {

    private Engine engine;

    public void setEngine(Engine engine) {
        this.engine = engine;
        System.out.println("Engine Injected");
    }

    public void drive() {
        engine.start();
        System.out.println("Car Running");
    }

}
```

---

## Step 3: XML Configuration

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="engine"
          class="com.practice.springcore.Engine"/>

    <bean id="car"
          class="com.practice.springcore.Car">

        <property name="engine" ref="engine"/>

    </bean>

</beans>
```

### What does this mean?

```xml
<property name="engine" ref="engine"/>
```

Spring looks for:

```java
setEngine(...)
```

and calls:

```java
car.setEngine(engine);
```

---

## Step 4: Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

Car car = context.getBean("car", Car.class);

car.drive();
```

### Output

```
Engine Created
Engine Injected
Engine Started
Car Running
```

---

# What Happens Internally?

Spring performs something like:

```java
Engine engine = new Engine();

Car car = new Car();

car.setEngine(engine);
```

You never write this code.

Spring does it.

---

# Example 2: Primitive Injection

## Employee.java

```java
package com.practice.springcore;

public class Employee {

    private int id;
    private String name;

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void display() {
        System.out.println(id + " " + name);
    }

}
```

---

## XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee">

    <property name="id" value="101"/>

    <property name="name" value="Rahul"/>

</bean>
```

Spring internally performs:

```java
Employee emp = new Employee();

emp.setId(101);

emp.setName("Rahul");
```

---

# Mixing Primitive and Object Injection

```java
public class Employee {

    private int id;
    private String name;
    private Address address;

    // setters
}
```

XML

```xml
<bean id="address"
      class="com.practice.springcore.Address"/>

<bean id="employee"
      class="com.practice.springcore.Employee">

    <property name="id" value="101"/>

    <property name="name" value="Rahul"/>

    <property name="address" ref="address"/>

</bean>
```

Spring calls:

```java
emp.setId(101);

emp.setName("Rahul");

emp.setAddress(address);
```

---

# Constructor Injection vs Setter Injection

## Constructor Injection

```java
public Car(Engine engine)
```

Spring:

```java
new Car(engine);
```

---

## Setter Injection

```java
public void setEngine(Engine engine)
```

Spring:

```java
Car car = new Car();

car.setEngine(engine);
```

---

# Which is Better?

### Constructor Injection

Advantages:

* Mandatory dependencies
* Immutable objects
* Better for testing
* Recommended in modern Spring

---

### Setter Injection

Advantages:

* Optional dependencies
* Easy to modify after object creation
* Good for configuration values

---

# Interview Question

### Constructor vs Setter Injection

**Answer:**

| Constructor Injection        | Setter Injection                    |
| ---------------------------- | ----------------------------------- |
| Dependency is mandatory      | Dependency is optional              |
| Object is fully initialized  | Object can be partially initialized |
| Better for immutable classes | Better when dependencies may change |
| Recommended in modern Spring | Used when flexibility is needed     |

For example, a `Car` cannot function without an `Engine`, so constructor injection is appropriate. An optional `Color` property could be set using setter injection.

---

# XML Tags Learned So Far

## Bean

```xml
<bean id="employee"
      class="com.practice.Employee"/>
```

---

## Constructor Injection

```xml
<constructor-arg value="101"/>

<constructor-arg ref="address"/>
```

---

## Setter Injection

```xml
<property name="name" value="Rahul"/>

<property name="address" ref="address"/>
```

---

# Homework

Create these classes yourself:

```
Employee
     │
     ▼
Department
```

`Department.java`

```java
public class Department {

    public void display() {
        System.out.println("IT Department");
    }

}
```

`Employee.java`

```java
public class Employee {

    private int id;
    private String name;
    private Department department;

    // Create setters

    public void display() {
        System.out.println(id);
        System.out.println(name);
        department.display();
    }

}
```

Write the XML using **setter injection** for:

* `id`
* `name`
* `department`

---

# Next Lesson (Very Important)

We'll learn **Collection Injection**, where Spring injects:

* `List`
* `Set`
* `Map`
* `Properties`

This is commonly used in enterprise applications and is a popular interview topic because it demonstrates how Spring can inject more than just simple objects.
