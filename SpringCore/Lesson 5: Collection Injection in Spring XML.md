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
