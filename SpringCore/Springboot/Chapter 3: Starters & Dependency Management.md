# Spring Boot — Chapter 3: Starters & Dependency Management

Now we move into one of the most practical Spring Boot topics:

> **Starters → Dependency Management → BOM → Parent POM**

You already know:

```text id="h8v3m1"
Spring Boot
   ↓
@SpringBootApplication
   ↓
Auto-Configuration
```

But auto-configuration needs dependencies to exist on the classpath.

So the next question is:

> **How does Spring Boot manage all these dependencies without us manually specifying hundreds of versions?**

---

# 1. Why do we need Starters?

Imagine we want to build a REST API.

We might need:

```text id="q2m7x4"
Spring MVC
Jackson
Tomcat
Spring Core
Validation
Logging
```

Without starters, we'd have to decide which individual dependencies to add.

That's tedious and creates version-compatibility problems.

Spring Boot provides **starter dependencies** to give you a convenient, supported set of dependencies for a particular capability. ([Home][1])

---

# 2. What is a Starter?

A starter is basically a **convenient dependency descriptor**.

For example:

```xml id="5zr42s"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
```

You are saying:

> "I am building a Spring MVC web application. Give me the normal dependencies needed for that."

You don't manually list every transitive dependency.

---

# 3. Important: Starter vs Library

This distinction is important.

### Starter

```text id="1k8f4z"
spring-boot-starter-webmvc
```

is a **dependency grouping/descriptor**.

### Library

Examples:

```text id="g5q7p2"
spring-webmvc
jackson-databind
tomcat
```

are actual libraries/modules used by your application.

So:

```text id="a3m9x6"
Starter
   ↓
Pulls in
   ↓
Multiple dependencies
```

---

# 4. Common Spring Boot Starters

For your roadmap, know these:

```text id="r7n2c5"
spring-boot-starter-webmvc
    ↓
Spring MVC REST applications

spring-boot-starter-jdbc
    ↓
JDBC applications

spring-boot-starter-security
    ↓
Spring Security

spring-boot-starter-validation
    ↓
Bean Validation

spring-boot-starter-test
    ↓
Testing

spring-boot-starter-security-oauth2-resource-server
    ↓
JWT / Bearer-token Resource Server
```

The exact starter set evolves with Spring Boot releases, so use the current Boot documentation rather than memorizing a frozen list. ([Home][1])

---

# 5. Why not add individual dependencies?

You technically can.

For example:

```xml id="y8m3q1"
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
</dependency>
```

Then:

```xml id="8q4m6d"
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

Then perhaps another dependency.

But now you are responsible for understanding compatibility among those versions.

A starter simplifies the common case.

---

# 6. Dependency Management

Now the more important concept.

Suppose you write:

```xml id="k5m2q9"
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
</dependency>
```

Notice:

```text id="v8c3p1"
No <version>
```

Why does Maven know which version to use?

Because Spring Boot provides **dependency management**.

Each Spring Boot release has a curated dependency set and manages compatible versions for many supported libraries. You normally don't specify their versions yourself. ([Home][1])

---

# 7. What is a BOM?

BOM means:

> **Bill of Materials**

Spring Boot publishes:

```text id="t4q8m3"
spring-boot-dependencies
```

This is a Maven BOM containing managed dependency versions.

Conceptually:

```text id="h1n6p9"
Spring Boot version
       ↓
spring-boot-dependencies
       ↓
Compatible library versions
```

This helps keep the dependency set consistent. ([Home][1])

---

# 8. Why is BOM Important?

Imagine your project uses:

```text id="s8r2c5"
Spring Framework
Jackson
Tomcat
JUnit
Logback
Hibernate
```

Instead of you independently choosing:

```text id="c2q7m1"
Spring = version A
Jackson = version B
Tomcat = version C
...
```

Boot provides a tested/curated version set.

So:

```text id="p9m4x6"
Spring Boot 4.1.x
        ↓
Compatible dependency versions
```

When you upgrade Boot, the managed dependency versions can move together consistently. ([Home][1])

---

# 9. What is `spring-boot-starter-parent`?

For Maven, many Boot projects use:

```xml id="w6c2q8"
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.0</version>
    <relativePath/>
</parent>
```

The starter parent provides convenient Maven defaults and plugin management, and it inherits Boot's dependency-management configuration. ([Home][2])

So conceptually:

```text id="x4m8n2"
spring-boot-starter-parent
          ↓
Maven defaults
+
Plugin management
+
Dependency management
```

---

# 10. A Typical Maven `pom.xml`

Example:

```xml id="j9s7v3"
<project>

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.0</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>employee-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>25</java.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

        </plugins>
    </build>

</project>
```

Notice the dependency declarations don't include versions.

That's because Boot manages the versions for supported dependencies. ([Home][1])

---

# 11. What if I Don't Want to Use the Parent?

You don't have to.

Suppose your organization already has its own parent POM.

You can still import Spring Boot's BOM:

```xml id="6nd0i4"
<dependencyManagement>
    <dependencies>

        <dependency>
            <groupId>
                org.springframework.boot
            </groupId>

            <artifactId>
                spring-boot-dependencies
            </artifactId>

            <version>4.1.0</version>

            <type>pom</type>
            <scope>import</scope>
        </dependency>

    </dependencies>
</dependencyManagement>
```

This gives you Boot's dependency management without inheriting the `spring-boot-starter-parent`. The official Maven documentation specifically describes this alternative. ([Home][2])

---

# 12. Parent vs BOM

This is a frequent interview question.

### `spring-boot-starter-parent`

Provides:

```text id="8f4k1m"
Maven defaults
Plugin management
Dependency management
```

### `spring-boot-dependencies`

Primarily provides:

```text id="c5n7q2"
Dependency version management
```

So:

```text id="z3m6x8"
Parent
   ↓
More Maven behavior

BOM
   ↓
Dependency versions
```

---

# 13. What if We Need a Different Version?

Boot provides recommended versions.

But sometimes an application needs to override one.

For example, conceptually:

```xml id="f1v8k3"
<properties>
    <some-library.version>...</some-library.version>
</properties>
```

or explicitly specify a dependency version where appropriate.

Spring Boot allows managed versions to be overridden, but you should understand the compatibility implications before doing so. ([Home][1])

The general principle:

> **Use Boot's managed version unless you have a reason to override it.**

---

# 14. Why Shouldn't We Randomly Override Versions?

Suppose Boot expects:

```text id="b9c5m1"
Library A = version 1
Library B = version 2
```

and you force:

```text id="m4x8q3"
Library A = version 7
```

You might introduce:

```text id="s2r6n9"
Binary incompatibility
Runtime errors
Unexpected behavior
Security issues
```

So don't override a dependency version just because Maven allows it.

---

# 15. What Happens During Maven Dependency Resolution?

Conceptually:

```text id="p6k3m8"
pom.xml
   ↓
Parent / BOM
   ↓
Dependency Management
   ↓
Starter
   ↓
Transitive Dependencies
   ↓
Resolved Dependency Graph
   ↓
Classpath
```

For example:

```text id="h1r7c2"
spring-boot-starter-webmvc
        ↓
Spring MVC
Jackson
Embedded servlet web infrastructure
...
```

The precise transitive set depends on the Boot version and starter. Boot's current build documentation provides the official starter and dependency-management information. ([Home][1])

---

# 16. How to See the Actual Dependencies

With Maven:

```bash id="b4n9q2"
mvn dependency:tree
```

This is a very useful real-world command.

You'll see something conceptually like:

```text id="a7x3m5"
spring-boot-starter-webmvc
├── spring-webmvc
├── spring-web
├── jackson-...
└── ...
```

This is how you debug:

```text id="m5q2v8"
"Why is this library present?"
```

or:

```text id="k8c4n1"
"Why do I have two versions?"
```

---

# 17. Dependency Conflict

Suppose:

```text id="m2v7p4"
Library A
 ↓
Jackson 1

Library B
 ↓
Jackson 2
```

Maven must resolve the dependency graph.

This is why understanding:

```text id="x9q3c6"
Direct dependency
Transitive dependency
Dependency management
Version mediation
```

is valuable.

Spring Boot reduces many common compatibility problems by managing a curated version set, but it doesn't eliminate Maven's dependency resolution rules.

---

# 18. Starter vs BOM

Another interview question.

### Starter

Answers:

> **What dependencies do I commonly need for this feature?**

Example:

```text id="u6m3x8"
spring-boot-starter-security
```

### BOM

Answers:

> **Which versions of these libraries should be used together?**

Example:

```text id="k4r9p1"
spring-boot-dependencies
```

So:

```text id="b8x2q5"
Starter → dependency grouping

BOM → version coordination
```

This distinction is extremely useful.

---

# 19. Dependency Management vs Dependency

Another common confusion.

### Dependency

```xml id="y2c7m4"
<dependencies>
    <dependency>
        ...
    </dependency>
</dependencies>
```

means:

> I actually need this library in my application.

### Dependency Management

```xml id="j9p5x3"
<dependencyManagement>
```

means:

> Define/manage versions and dependency metadata.

It doesn't by itself mean:

> Put every dependency into my classpath.

This is an important Maven concept.

---

# 20. Why Does Boot Need Dependency Management?

Because Spring Boot itself depends on a coordinated ecosystem.

For example:

```text id="f6m3p8"
Spring Framework
Spring Security
Spring Data
Jackson
Tomcat
JUnit
Logging
```

Boot's curated dependency list helps keep compatible versions together. ([Home][1])

So:

```text id="w3q8n1"
Spring Boot version
        ↓
Curated dependency set
        ↓
Less version-management work
```

---

# 21. What About Spring Framework Version?

This is especially important.

Suppose you're using Spring Boot.

You normally should **not** write:

```xml id="q5m9v3"
<spring.version>...</spring.version>
```

to manually choose a Spring Framework version.

Spring Boot is associated with a corresponding Spring Framework baseline, and the official docs recommend not specifying the Spring Framework version separately. ([Home][1])

So:

```text id="v8m2c6"
Spring Boot
   ↓
Manages Spring Framework version
```

---

# 22. Real-World Example

Suppose your project needs:

```text id="h6q3m7"
REST API
Security
Validation
Testing
JDBC
```

You might simply declare:

```xml id="5d3m9w"
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

No individual Spring version declarations.

Boot's dependency management handles supported versions. ([Home][1])

---

# 23. Interview Questions

### What is a Spring Boot Starter?

> A starter is a convenient dependency descriptor that groups the dependencies commonly required for a particular Spring Boot capability. ([Home][1])

### What is a BOM?

> A Bill of Materials is a dependency-management POM that defines compatible versions for a coordinated set of dependencies.

### What is `spring-boot-dependencies`?

> It is Spring Boot's curated dependency-management BOM containing versions for supported Spring and third-party libraries. ([Home][1])

### Why don't we specify versions for every dependency?

> Spring Boot's dependency management supplies tested/curated versions for supported dependencies. ([Home][1])

### What does `spring-boot-starter-parent` provide?

> It provides convenient Maven defaults, plugin management, and inherited dependency-management configuration. ([Home][2])

### Parent vs BOM?

> The parent provides broader Maven build conventions and plugin management along with dependency management; the BOM primarily provides dependency version management.

### Can we override Boot's dependency versions?

> Yes, but it should be done deliberately because overriding the curated versions can introduce compatibility problems. ([Home][1])

---

# 24. Best Practices

```text id="e1x9q4"
✅ Prefer Spring Boot starters
✅ Let Boot manage versions when possible
✅ Understand the BOM
✅ Use dependency:tree when debugging
✅ Avoid unnecessary version overrides
✅ Keep dependencies minimal
✅ Remove dependencies you don't actually need
✅ Understand direct vs transitive dependencies
```

---

# 25. Mental Model

This is the one diagram to remember:

```text id="p8q4x2"
spring-boot-starter-parent
          │
          ├── Maven defaults
          ├── Plugin management
          └── Dependency management
                    │
                    ▼
          spring-boot-dependencies
                    │
                    ▼
             Managed Versions
                    │
                    ▼
              Your Starters
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      MVC        Security      JDBC
        │           │           │
        └───────────┼───────────┘
                    ▼
             Application
```

---

# 📍 Where We Are

```text id="o5v9m2"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
│
├── ✅ Chapter 2 — @SpringBootApplication
│
├── ✅ Chapter 3 — Starters & Dependency Management
│      ├── Starters
│      ├── BOM
│      ├── spring-boot-dependencies
│      ├── spring-boot-starter-parent
│      ├── Dependency Management
│      └── Dependency Tree
│
└── ⏭️ Chapter 4 — Auto-Configuration ⭐⭐⭐⭐⭐
       ↓
       What exactly happens?
       ↓
       @ConditionalOnClass
       ↓
       @ConditionalOnMissingBean
       ↓
       Auto-configuration classes
       ↓
       Why Boot creates some beans automatically
       ↓
       How to debug auto-configuration
       ↓
       How to override it
```

Next is **Spring Boot Auto-Configuration**, the most important Boot concept. We'll go beyond "Boot automatically configures things" and understand **how the conditional mechanism actually decides whether an auto-configuration should apply**.

[1]: https://docs.spring.io/spring-boot/reference/using/build-systems.html?utm_source=chatgpt.com "Build Systems :: Spring Boot"
[2]: https://docs.spring.io/spring-boot/maven-plugin/using.html?utm_source=chatgpt.com "Using the Plugin :: Spring Boot"
