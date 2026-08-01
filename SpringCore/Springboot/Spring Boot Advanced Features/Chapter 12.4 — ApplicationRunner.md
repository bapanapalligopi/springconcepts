# Spring Boot — Chapter 12.4: `ApplicationRunner` ⭐⭐⭐⭐⭐

In the previous chapter, we learned about `CommandLineRunner`.

Now we'll learn **why Spring Boot introduced another interface called `ApplicationRunner`**, even though `CommandLineRunner` already existed.

This is a common interview question.

---

# Chapter Roadmap

```text
ApplicationRunner
│
├── 1. Why ApplicationRunner?
├── 2. What is ApplicationRunner?
├── 3. Execution Flow
├── 4. ApplicationArguments
├── 5. Option Arguments
├── 6. Non-Option Arguments
├── 7. Multiple Runners
├── 8. @Order
├── 9. Enterprise Examples
├── 10. ApplicationRunner vs CommandLineRunner
├── 11. Best Practices
└── 12. Interview Questions
```

---

# 1. Why was `ApplicationRunner` introduced?

Suppose you start your application like this:

```bash
java -jar employee-api.jar \
    --profile=dev \
    --cache=true \
    --debug
```

With `CommandLineRunner`:

```java
@Override
public void run(String... args) {

    for (String arg : args) {
        System.out.println(arg);
    }
}
```

Output:

```text
--profile=dev

--cache=true

--debug
```

Everything is just a **String**.

Now you must manually parse:

```text
"profile=dev"

↓

Split

↓

Extract Value

↓

Validate
```

Not convenient.

---

# 2. Enter `ApplicationRunner`

Instead of giving raw strings,

Spring Boot provides a helper object.

Interface:

```java
@FunctionalInterface
public interface ApplicationRunner {

    void run(
        ApplicationArguments args
    ) throws Exception;
}
```

Instead of:

```text
String[]
```

you get:

```text
ApplicationArguments
```

---

# 3. Lifecycle Position

`ApplicationRunner` executes in exactly the same place as `CommandLineRunner`.

```text
main()

↓

SpringApplication.run()

↓

Beans Created

↓

@PostConstruct

↓

ApplicationStartedEvent

↓

ApplicationRunner

↓

ApplicationReadyEvent
```

---

# 4. What is `ApplicationArguments`?

It is an object that understands command-line arguments.

Instead of manually parsing:

```text
"--profile=dev"
```

it provides helper methods.

Think:

```text
Raw Arguments

↓

ApplicationArguments

↓

Easy API
```

---

# 5. Creating an `ApplicationRunner`

Example:

```java
@Component
public class StartupRunner
        implements ApplicationRunner {

    @Override
    public void run(
            ApplicationArguments args) {

        System.out.println("Started");
    }
}
```

Spring automatically executes:

```text
run(ApplicationArguments)
```

during startup.

---

# 6. Option Arguments

Suppose:

```bash
java -jar app.jar \
    --profile=dev \
    --cache=true
```

Now:

```java
args.containsOption("profile");
```

returns:

```text
true
```

Retrieve value:

```java
args.getOptionValues("profile");
```

returns:

```text
["dev"]
```

Spring parses everything for you.

---

# 7. Multiple Values

Command:

```bash
java -jar app.jar \
    --role=ADMIN \
    --role=USER
```

Code:

```java
List<String> roles =
        args.getOptionValues("role");
```

Result:

```text
ADMIN

USER
```

Notice:

Spring automatically groups repeated options.

---

# 8. Non-Option Arguments

Suppose:

```bash
java -jar app.jar dev production backup
```

No `--`.

These are **non-option arguments**.

Read them:

```java
args.getNonOptionArgs();
```

Returns:

```text
dev

production

backup
```

---

# 9. Check if an Option Exists

Example:

```bash
java -jar app.jar --debug
```

Code:

```java
if (args.containsOption("debug")) {

    System.out.println("Debug Enabled");
}
```

Very clean.

No string parsing.

---

# 10. Getting All Option Names

Example:

```bash
--profile=dev

--cache=true

--debug
```

Code:

```java
args.getOptionNames();
```

Result:

```text
profile

cache

debug
```

---

# 11. Complete Example

Command:

```bash
java -jar employee.jar \
    --profile=prod \
    --cache=true
```

Runner:

```java
@Component
public class StartupRunner
        implements ApplicationRunner {

    @Override
    public void run(
            ApplicationArguments args) {

        if (args.containsOption("profile")) {

            System.out.println(
                args.getOptionValues("profile")
            );
        }

        if (args.containsOption("cache")) {

            System.out.println(
                args.getOptionValues("cache")
            );
        }
    }
}
```

Output:

```text
prod

true
```

---

# 12. Multiple `ApplicationRunner`s

Spring supports multiple runners.

Example:

```java
@Component
@Order(1)
class DatabaseRunner
        implements ApplicationRunner {

    @Override
    public void run(
            ApplicationArguments args) {

    }
}
```

```java
@Component
@Order(2)
class CacheRunner
        implements ApplicationRunner {

    @Override
    public void run(
            ApplicationArguments args) {

    }
}
```

Execution:

```text
DatabaseRunner

↓

CacheRunner
```

---

# 13. Enterprise Use Cases

## Feature Flags

```bash
--feature-new-ui=true
```

```java
if (args.containsOption("feature-new-ui")) {

    enableFeature();
}
```

---

## Startup Mode

```bash
--mode=maintenance
```

```java
if (args.containsOption("mode")) {

    String mode =
        args.getOptionValues("mode").get(0);
}
```

---

## Cache Control

```bash
--load-cache=false
```

```java
if ("false".equals(
    args.getOptionValues("load-cache").get(0))) {

    return;
}
```

---

## Debug Mode

```bash
--debug
```

```java
if (args.containsOption("debug")) {

    enableVerboseLogging();
}
```

---

# 14. Internal Flow

```text
Application Starts

↓

ApplicationArguments Created

↓

Find ApplicationRunner Beans

↓

Sort by @Order

↓

Execute run()

↓

ApplicationReadyEvent
```

Very similar to `CommandLineRunner`.

---

# 15. `ApplicationRunner` vs `CommandLineRunner`

| Feature          | CommandLineRunner | ApplicationRunner      |
| ---------------- | ----------------- | ---------------------- |
| Parameter        | `String... args`  | `ApplicationArguments` |
| Parsing          | Manual            | Automatic              |
| Option handling  | Manual            | Built-in               |
| Non-option args  | Manual            | Built-in               |
| Repeated options | Manual            | Built-in               |
| Complexity       | Simpler           | Richer                 |

---

# 16. Which One Should You Choose?

### Use `CommandLineRunner`

When:

* Simple startup logic
* Arguments don't matter
* Quick initialization

Example:

```java
run(String... args)
```

---

### Use `ApplicationRunner`

When:

* Startup options matter
* Named arguments
* Configuration flags
* Multiple option values

Example:

```java
run(ApplicationArguments args)
```

---

# 17. Best Practices

```text
✅ Use ApplicationRunner when you need command-line options

✅ Use @Order when multiple runners exist

✅ Keep startup logic short

✅ Validate startup arguments

✅ Fail fast for invalid options

❌ Don't parse strings manually if ApplicationArguments can do it

❌ Don't execute long-running business workflows here
```

---

# 18. Interview Questions

### What is `ApplicationRunner`?

> A Spring Boot callback interface that executes after the application context has started and provides command-line arguments through the `ApplicationArguments` API.

---

### Why is it better than `CommandLineRunner`?

> It provides structured access to command-line arguments, including named options, repeated values, and non-option arguments, eliminating manual parsing.

---

### What is `ApplicationArguments`?

> A helper interface that represents parsed command-line arguments and provides methods such as `containsOption()`, `getOptionValues()`, and `getNonOptionArgs()`.

---

### When does it execute?

> After the application context has been initialized and before `ApplicationReadyEvent`.

---

### Can multiple `ApplicationRunner`s exist?

> Yes. Spring executes all registered runners. Use `@Order` to define execution order if necessary.

---

# 19. Complete Execution Flow

```text
Application Starts

↓

SpringApplication.run()

↓

ApplicationContext

↓

Bean Creation

↓

Dependency Injection

↓

@PostConstruct

↓

ApplicationStartedEvent

↓

ApplicationRunner

↓

ApplicationReadyEvent

↓

Application Running
```

---

# 📍 Where We Are

```text
Spring Boot Advanced Features
│
├── ✅ 12.1 Application Lifecycle
├── ✅ 12.2 Application Events
├── ✅ 12.3 CommandLineRunner
├── ✅ 12.4 ApplicationRunner
│
└── ⏭️ 12.5 @PostConstruct ⭐⭐⭐⭐⭐
```

The next chapter explains **`@PostConstruct`** in depth:

* Bean lifecycle internals
* Initialization sequence
* Dependency injection timing
* Common mistakes
* `@PostConstruct` vs constructors
* `@PostConstruct` vs `CommandLineRunner`
* Enterprise use cases
* Complete lifecycle diagrams
