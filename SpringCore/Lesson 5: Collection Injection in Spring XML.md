Excellent! Now we're entering one of the most practical XML configuration topics.

# Lesson 5: Collection Injection in Spring XML

Spring can inject not only primitive values and objects, but also **collections** like:

* ✅ List
* ✅ Set
* ✅ Map
* ✅ Properties

This is useful when a bean needs multiple values or configurations.

---

# Example 1: Injecting a List

## Student.java

```java
package com.practice.springcore;

import java.util.List;

public class Student {

    private List<String> courses;

    public void setCourses(List<String> courses) {
        this.courses = courses;
    }

    public void display() {
        System.out.println("Courses:");
        for (String course : courses) {
            System.out.println(course);
        }
    }
}
```

---

## applicationContext.xml

```xml
<bean id="student"
      class="com.practice.springcore.Student">

    <property name="courses">

        <list>
            <value>Java</value>
            <value>Spring</value>
            <value>Hibernate</value>
            <value>Redis</value>
        </list>

    </property>

</bean>
```

---

## Main

```java
ApplicationContext context =
        new ClassPathXmlApplicationContext("applicationContext.xml");

Student student = context.getBean("student", Student.class);

student.display();
```

### Output

```text
Courses:
Java
Spring
Hibernate
Redis
```

---

# Internally Spring Does

```java
Student student = new Student();

List<String> list = new ArrayList<>();

list.add("Java");
list.add("Spring");
list.add("Hibernate");
list.add("Redis");

student.setCourses(list);
```

Spring writes this code for you.

---

# Example 2: Injecting a Set

## Employee.java

```java
package com.practice.springcore;

import java.util.Set;

public class Employee {

    private Set<String> skills;

    public void setSkills(Set<String> skills) {
        this.skills = skills;
    }

    public void display() {
        System.out.println(skills);
    }
}
```

---

## XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee">

    <property name="skills">

        <set>
            <value>Java</value>
            <value>Spring</value>
            <value>SQL</value>
            <value>Redis</value>
        </set>

    </property>

</bean>
```

### Output

```text
[Java, Spring, SQL, Redis]
```

### Why use Set?

A `Set` stores **unique** values.

If you add:

```xml
<value>Java</value>
<value>Java</value>
```

Only one `"Java"` is stored.

---

# Example 3: Injecting a Map

Suppose an employee has ID cards.

## Employee.java

```java
package com.practice.springcore;

import java.util.Map;

public class Employee {

    private Map<String, String> idCards;

    public void setIdCards(Map<String, String> idCards) {
        this.idCards = idCards;
    }

    public void display() {

        for (Map.Entry<String, String> entry : idCards.entrySet()) {

            System.out.println(entry.getKey() + " : " + entry.getValue());

        }

    }

}
```

---

## XML

```xml
<bean id="employee"
      class="com.practice.springcore.Employee">

    <property name="idCards">

        <map>

            <entry key="Office" value="EMP101"/>

            <entry key="Parking" value="P202"/>

            <entry key="Library" value="L303"/>

        </map>

    </property>

</bean>
```

### Output

```text
Office : EMP101
Parking : P202
Library : L303
```

---

# Example 4: Injecting Properties

`Properties` is commonly used for configuration values.

## DatabaseConfig.java

```java
package com.practice.springcore;

import java.util.Properties;

public class DatabaseConfig {

    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    public void display() {
        System.out.println(properties);
    }
}
```

---

## XML

```xml
<bean id="dbConfig"
      class="com.practice.springcore.DatabaseConfig">

    <property name="properties">

        <props>

            <prop key="url">jdbc:mysql://localhost:3306/test</prop>

            <prop key="username">root</prop>

            <prop key="password">root123</prop>

        </props>

    </property>

</bean>
```

### Output

```text
{url=jdbc:mysql://localhost:3306/test,
 username=root,
 password=root123}
```

---

# Collection Summary

| Collection | XML Tag   | Java Type    |
| ---------- | --------- | ------------ |
| List       | `<list>`  | `List`       |
| Set        | `<set>`   | `Set`        |
| Map        | `<map>`   | `Map`        |
| Properties | `<props>` | `Properties` |

---

# Nested Beans (Advanced XML)

Spring also allows you to define a bean inside another bean.

Example:

```xml
<bean id="car"
      class="com.practice.springcore.Car">

    <property name="engine">

        <bean class="com.practice.springcore.Engine"/>

    </property>

</bean>
```

This creates an `Engine` bean only for this specific `Car` bean.

---

# Interview Questions

### Q1. Can Spring inject collections?

**Answer:**
Yes. Spring can inject Java collections such as `List`, `Set`, `Map`, and `Properties` using XML configuration.

---

### Q2. What XML tags are used for collection injection?

* `<list>`
* `<set>`
* `<map>`
* `<props>`

---

### Q3. When would you use collection injection?

For example:

* A list of supported languages.
* A set of user roles.
* A map of configuration keys and values.
* Database or application settings.

---

# Practice Exercise

Create a class:

```java
public class Company {

    private String name;

    private List<String> departments;

    private Set<String> technologies;

    private Map<String, String> offices;

    // setters

    // display()
}
```

Try writing the XML yourself using:

* `<list>` for departments.
* `<set>` for technologies.
* `<map>` for offices.

This is excellent practice because it combines everything you've learned about setter injection with collection types.

---

# What We've Covered So Far

✅ Spring Framework introduction
✅ IoC (Inversion of Control)
✅ Dependency Injection (DI)
✅ BeanFactory vs ApplicationContext
✅ XML Bean Configuration
✅ Constructor Injection
✅ Setter Injection
✅ Collection Injection

---

# Next Lesson (One of the Most Important)

We'll learn **Bean Scopes**, where you'll understand:

* Singleton (default scope)
* Prototype
* Request
* Session
* Application
* WebSocket

You'll also answer a common interview question: **"If I call `getBean()` twice, do I get the same object or a new one?"** Understanding bean scopes is essential before moving on to bean lifecycle and annotation-based configuration.
That's an excellent question. This is where many Spring learners get confused.

The short answer is:

* ✅ **Arrays are directly supported in Spring XML.**
* ⚠️ **Queues are not directly supported with a dedicated XML tag**, but you can inject a `Queue` using a bean or a utility collection.

Let's look at both.

---

# 1. Array Injection ✅

Suppose you have:

```java
package com.practice.springcore;

public class Student {

    private String[] courses;

    public void setCourses(String[] courses) {
        this.courses = courses;
    }

    public void display() {
        for (String course : courses) {
            System.out.println(course);
        }
    }
}
```

## XML

```xml
<bean id="student"
      class="com.practice.springcore.Student">

    <property name="courses">

        <array>
            <value>Java</value>
            <value>Spring</value>
            <value>Redis</value>
        </array>

    </property>

</bean>
```

### Output

```
Java
Spring
Redis
```

Internally, Spring does something like:

```java
String[] arr = {"Java", "Spring", "Redis"};

student.setCourses(arr);
```

---

# 2. Queue Injection ⚠️

Suppose:

```java
import java.util.Queue;

public class TaskManager {

    private Queue<String> tasks;

    public void setTasks(Queue<String> tasks) {
        this.tasks = tasks;
    }

    public void display() {
        System.out.println(tasks);
    }
}
```

There is **no XML tag like**:

```xml
<queue> ❌
```

Spring doesn't provide a `<queue>` element.

---

## Option 1 (Recommended)

Create the queue as a bean.

```xml
<bean id="taskQueue"
      class="java.util.LinkedList">

    <constructor-arg>
        <list>
            <value>Task1</value>
            <value>Task2</value>
            <value>Task3</value>
        </list>
    </constructor-arg>

</bean>

<bean id="taskManager"
      class="com.practice.springcore.TaskManager">

    <property name="tasks" ref="taskQueue"/>

</bean>
```

Here:

```text
LinkedList
        │
implements
        ▼
Queue
```

So Spring injects a `LinkedList` object into a `Queue` reference.

---

# 3. PriorityQueue

```xml
<bean id="queue"
      class="java.util.PriorityQueue">

    <constructor-arg>

        <list>
            <value>30</value>
            <value>10</value>
            <value>20</value>
        </list>

    </constructor-arg>

</bean>
```

Spring creates

```java
PriorityQueue<Integer> queue = new PriorityQueue<>();
```

---

# 4. Deque

```xml
<bean id="deque"
      class="java.util.ArrayDeque">

    <constructor-arg>

        <list>
            <value>A</value>
            <value>B</value>
            <value>C</value>
        </list>

    </constructor-arg>

</bean>
```

---

# Collections Supported by Spring XML

| Java Collection | XML Tag       | Supported Directly |
| --------------- | ------------- | ------------------ |
| List            | `<list>`      | ✅ Yes              |
| Set             | `<set>`       | ✅ Yes              |
| Map             | `<map>`       | ✅ Yes              |
| Properties      | `<props>`     | ✅ Yes              |
| Array           | `<array>`     | ✅ Yes              |
| Queue           | No direct tag | ⚠️ Use a bean      |
| Deque           | No direct tag | ⚠️ Use a bean      |
| PriorityQueue   | No direct tag | ⚠️ Use a bean      |

---

# Why doesn't Spring have `<queue>`?

Because `Queue` is an **interface**, not a concrete class.

```text
Queue
   ▲
   │
LinkedList
PriorityQueue
ArrayDeque
ConcurrentLinkedQueue
```

Spring cannot instantiate an interface directly. It needs a concrete implementation, such as `LinkedList` or `PriorityQueue`.

---

# Interview Question

**Q: Which collection types can be injected directly using Spring XML?**

**Answer:**

Spring XML provides dedicated tags for:

* `<list>`
* `<set>`
* `<map>`
* `<props>`
* `<array>`

For interfaces like `Queue` or `Deque`, there are no dedicated XML tags. Instead, you define a concrete implementation (such as `LinkedList` or `ArrayDeque`) as a bean and inject it into the interface reference.

---

## Next Topic

Now that you understand dependency injection and collections, the next important Spring Core topic is **Bean Scopes**. It's one of the most frequently asked interview topics and explains why calling `getBean()` sometimes returns the **same object** and other times a **new object**.
