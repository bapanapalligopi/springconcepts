Excellent! 🎉 You're learning Spring the same way it evolved historically. By the end of this course, you'll understand **what `@Autowired` actually does behind the scenes**, instead of just memorizing annotations.

# Lesson 8: Autowiring in XML

---

# What is Autowiring?

So far, we've manually injected dependencies.

Example:

```xml
<bean id="engine"
      class="com.practice.springcore.Engine"/>

<bean id="car"
      class="com.practice.springcore.Car">

    <property name="engine" ref="engine"/>

</bean>
```

Here we explicitly told Spring:

> "Inject the `engine` bean into the `Car`."

This works well, but imagine a project with:

* 200 beans
* 500 dependencies

Writing every `<property>` becomes tedious.

**Autowiring** tells Spring:

> **"You find the dependency and inject it automatically."**

---

# Types of Autowiring

Spring XML supports four modes:

| Mode          | Description                      | Used? |
| ------------- | -------------------------------- | ----- |
| `no`          | No automatic injection (default) | ✅     |
| `byName`      | Match bean id with property name | ⭐⭐⭐⭐  |
| `byType`      | Match by data type               | ⭐⭐⭐⭐⭐ |
| `constructor` | Constructor injection            | ⭐⭐⭐⭐  |

---

# 1. autowire="no"

Default mode.

```xml
<bean id="car"
      class="com.practice.springcore.Car"
      autowire="no"/>
```

Spring won't inject anything automatically.

You must write:

```xml
<property name="engine"
          ref="engine"/>
```

---

# 2. autowire="byName"

## Engine.java

```java
public class Engine {

    public void start() {
        System.out.println("Engine Started");
    }

}
```

---

## Car.java

```java
public class Car {

    private Engine engine;

    public void setEngine(Engine engine) {
        this.engine = engine;
    }

    public void drive() {
        engine.start();
    }

}
```

Notice:

Property name is

```java
engine
```

---

## XML

```xml
<bean id="engine"
      class="com.practice.springcore.Engine"/>

<bean id="car"
      class="com.practice.springcore.Car"
      autowire="byName"/>
```

No `<property>`!

Spring looks for:

```java
private Engine engine;
```

Then looks for a bean with id:

```text
engine
```

It finds:

```xml
<bean id="engine"/>
```

Then internally calls:

```java
car.setEngine(engine);
```

---

# What if the names don't match?

```xml
<bean id="myEngine"
      class="Engine"/>
```

Property:

```java
private Engine engine;
```

Spring searches for

```text
engine
```

It finds

```text
myEngine
```

❌ No match.

Injection fails.

---

# byName Rule

```text
Property Name
       │
       ▼
Bean ID
```

Both must be the same.

---

# 3. autowire="byType"

Instead of matching names,

Spring matches data types.

Car

```java
private Engine engine;
```

XML

```xml
<bean id="myEngine"
      class="com.practice.springcore.Engine"/>

<bean id="car"
      class="com.practice.springcore.Car"
      autowire="byType"/>
```

Even though

```text
myEngine
```

doesn't match

```text
engine
```

Spring sees

```text
Engine
```

↓

Finds bean of type

```text
Engine
```

↓

Injects it.

---

# What if there are two Engine beans?

```xml
<bean id="engine1"
      class="Engine"/>

<bean id="engine2"
      class="Engine"/>
```

Now Spring finds

```text
Engine

↓

engine1

↓

engine2
```

Which one?

It doesn't know.

Result:

```text
NoUniqueBeanDefinitionException
```

This is a common interview question.

---

# byType Rule

```text
Property Type

↓

Find Bean

↓

Inject
```

---

# 4. constructor Autowiring

Suppose

```java
public class Car {

    private Engine engine;

    public Car(Engine engine) {

        this.engine = engine;

    }

}
```

XML

```xml
<bean id="engine"
      class="Engine"/>

<bean id="car"
      class="Car"
      autowire="constructor"/>
```

Spring searches for

```text
Constructor

↓

Engine Parameter

↓

Engine Bean

↓

new Car(engine)
```

---

# Internal Working

Suppose

```java
public class Car {

    private Engine engine;

    public void setEngine(Engine engine){

        this.engine = engine;

    }

}
```

Spring internally performs

```java
Engine engine = new Engine();

Car car = new Car();

car.setEngine(engine);
```

Autowiring simply removes the need to explicitly write the `<property>` element.

---

# byName vs byType

| byName                           | byType                                         |
| -------------------------------- | ---------------------------------------------- |
| Matches bean ID                  | Matches bean class/type                        |
| Bean id must equal property name | Bean id doesn't matter                         |
| Faster lookup                    | More flexible                                  |
| Fails if names differ            | Fails if multiple beans of the same type exist |

---

# Interview Questions

## Q1. What is Autowiring?

**Answer**

Autowiring is a feature of Spring that automatically resolves and injects bean dependencies without requiring explicit `<property>` or `<constructor-arg>` configuration.

---

## Q2. What are the autowiring modes?

* no
* byName
* byType
* constructor

---

## Q3. Difference between byName and byType?

**byName**

* Matches the property name with the bean ID.

**byType**

* Matches the property type with the bean class.
* Throws an exception if multiple beans of the same type exist.

---

# How does this relate to `@Autowired`?

This is where everything connects.

### XML

```xml
<bean id="engine"
      class="Engine"/>

<bean id="car"
      class="Car"
      autowire="byType"/>
```

### Annotation-based configuration

```java
@Component
public class Car {

    @Autowired
    private Engine engine;

}
```

`@Autowired` behaves much like **autowire="byType"** by default.

If multiple beans of the same type exist, you use:

```java
@Autowired
@Qualifier("dieselEngine")
private Engine engine;
```

which is similar to explicitly choosing a bean in XML.

---

# Complete XML Journey

You've now mastered almost everything in XML configuration:

✅ Spring Framework Basics
✅ IoC Container
✅ Dependency Injection
✅ BeanFactory vs ApplicationContext
✅ Constructor Injection
✅ Setter Injection
✅ Collection Injection
✅ Bean Scope
✅ Bean Lifecycle
✅ XML Autowiring

---

# Next Topic (Very Important)

We'll now move to **Annotation-Based Configuration**, where you'll learn:

* `@Component`
* `@Service`
* `@Repository`
* `@Controller`
* `@Autowired`
* `@Qualifier`
* `@Primary`
* `@Value`
* `@ComponentScan`
* `@Configuration`
* `@Bean`

Since you now understand the XML equivalents, these annotations will be much easier to learn, and you'll also be able to answer interview questions like **"What is the difference between XML configuration and annotation-based configuration?"** confidently.
