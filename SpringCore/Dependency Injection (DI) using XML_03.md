Perfect! Now we're starting the **heart of Spring Core**. Once you master Dependency Injection (DI), annotations like `@Autowired` become much easier to understand.

# Lesson 3: Dependency Injection (DI) using XML

## What is a Dependency?

A dependency is an object that another object needs to perform its work.

Example:

```text
Car
 │
 ▼
Engine
```

Here:

* `Car` depends on `Engine`.
* Therefore, `Engine` is a dependency of `Car`.

---

## Without Spring

```java
public class Engine {

    public void start() {
        System.out.println("Engine Started");
    }
}
```

```java
public class Car {

    private Engine engine = new Engine();

    public void drive() {
        engine.start();
        System.out.println("Car Running");
    }
}
```

### Problem

The `Car` class creates the `Engine` itself.

```text
Car
 │
new Engine()
```

This creates **tight coupling**.

If tomorrow you want to use:

* PetrolEngine
* DieselEngine
* ElectricMotor

you must modify the `Car` class.

---

# With Spring

Instead of the `Car` creating the `Engine`, Spring creates both objects and injects the `Engine` into the `Car`.

```text
Spring Container

Creates Engine
       │
       ▼
Creates Car
       │
Injects Engine
```

This is **Dependency Injection (DI)**.

---

# Types of Dependency Injection

Spring supports three main types:

```text
Dependency Injection

├── Constructor Injection ⭐⭐⭐⭐⭐
├── Setter Injection ⭐⭐⭐⭐⭐
└── Field Injection (Annotations)
```

We'll start with **Constructor Injection**.

---

# Constructor Injection

## Step 1: Create `Engine`

```java
package com.practice.springcore;

public class Engine {

    public Engine() {
        System.out.println("Engine Object Created");
    }

    public void start() {
        System.out.println("Engine Started");
    }
}
```

---

## Step 2: Create `Car`

```java
package com.practice.springcore;

public class Car {

    private Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
        System.out.println("Car Object Created");
    }

    public void drive() {
        engine.start();
        System.out.println("Car is Running");
    }
}
```

Notice:

```java
public Car(Engine engine)
```

The constructor expects an `Engine`.

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

        <constructor-arg ref="engine"/>

    </bean>

</beans>
```

### What does `<constructor-arg ref="engine"/>` mean?

It tells Spring:

> "When creating the `Car`, pass the bean with ID `engine` to its constructor."

Equivalent Java code:

```java
Engine engine = new Engine();
Car car = new Car(engine);
```

But **Spring** does it for you.

---

## Step 4: Main Class

```java
package com.practice.springcore;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {

    public static void main(String[] args) {

        ApplicationContext context =
                new ClassPathXmlApplicationContext("applicationContext.xml");

        Car car = context.getBean("car", Car.class);

        car.drive();
    }
}
```

---

## Output

```text
Engine Object Created
Car Object Created
Engine Started
Car is Running
```

---

# What Happens Internally?

When you create the `ApplicationContext`:

```java
ApplicationContext context =
    new ClassPathXmlApplicationContext("applicationContext.xml");
```

Spring executes something conceptually like this:

```text
Read applicationContext.xml
        │
        ▼
Find bean "engine"
        │
        ▼
new Engine()
        │
        ▼
Find bean "car"
        │
        ▼
new Car(engine)
        │
        ▼
Store both beans in the IoC Container
```

Notice that **you never wrote `new Engine()` or `new Car()`**. Spring did it.

---

# Constructor Injection with Primitive Values

Spring can also inject simple values.

### Employee.java

```java
package com.practice.springcore;

public class Employee {

    private int id;
    private String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public void display() {
        System.out.println(id + " " + name);
    }
}
```

### XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee">

    <constructor-arg value="101"/>
    <constructor-arg value="Rahul"/>

</bean>
```

Equivalent Java:

```java
Employee emp = new Employee(101, "Rahul");
```

---

# Constructor Injection with Multiple Objects

```text
OrderService
      │
      ├── PaymentService
      └── NotificationService
```

Spring can inject both dependencies:

```xml
<constructor-arg ref="paymentService"/>
<constructor-arg ref="notificationService"/>
```

---

# Constructor Injection vs Setter Injection

| Constructor Injection                    | Setter Injection                    |
| ---------------------------------------- | ----------------------------------- |
| Dependency is mandatory                  | Dependency is optional              |
| Object is fully initialized when created | Object can be partially initialized |
| Preferred for required dependencies      | Good for optional configuration     |

A common interview answer is:

> Use **constructor injection** for mandatory dependencies because it ensures the object is created in a valid state. Use **setter injection** when a dependency is optional or may change after object creation.

---

# Practice Exercise

Create these classes:

```
Student
      │
      ▼
Address
```

### Address.java

```java
public class Address {

    public Address() {
        System.out.println("Address Created");
    }
}
```

### Student.java

```java
public class Student {

    private Address address;

    public Student(Address address) {
        this.address = address;
        System.out.println("Student Created");
    }
}
```

### XML

Try writing the XML yourself before looking at the answer:

<details>
<summary>Answer</summary>

```xml
<bean id="address"
      class="com.practice.springcore.Address"/>

<bean id="student"
      class="com.practice.springcore.Student">

    <constructor-arg ref="address"/>

</bean>
```

</details>

---

## Next Lesson

We'll cover **Setter Injection**, where Spring injects dependencies by calling setter methods instead of constructors. You'll also learn how Spring decides which setter to call and when setter injection is a better choice than constructor injection.
