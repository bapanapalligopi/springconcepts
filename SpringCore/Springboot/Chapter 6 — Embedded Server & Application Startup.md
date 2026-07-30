# Spring Boot — Chapter 6: Embedded Server & Application Startup

Now we trace what happens when you run:

```java
SpringApplication.run(
    EmployeeApplication.class,
    args
);
```

This chapter connects everything we've learned:

```text
@SpringBootApplication
        ↓
Component Scanning
        ↓
Auto-Configuration
        ↓
ApplicationContext
        ↓
Embedded Server
        ↓
Application Ready
```

---

# 1. Why Embedded Server?

Traditional Java web applications were commonly packaged as WAR files and deployed into an externally managed servlet container such as Tomcat.

Spring Boot supports a different model:

```text
Your application
    +
Dependencies
    +
Embedded servlet server
    ↓
Executable JAR
```

Then:

```bash
java -jar employee-api.jar
```

starts the application.

For current Spring Boot MVC applications, Boot's servlet web support provides an embedded servlet environment, with Tomcat as the common default when using the standard web starter. The exact server can be changed depending on dependencies/configuration.

---

# 2. What Does "Embedded Tomcat" Mean?

It does **not** mean Tomcat is literally compiled into your Java classes.

It means the application includes the required Tomcat libraries and starts the servlet container from within the application process.

Conceptually:

```text
java -jar employee-api.jar
        ↓
Java process
        ↓
Spring Boot
        ↓
Embedded Tomcat
        ↓
HTTP port 8080
```

So you don't need to separately install and deploy to an external Tomcat server for the usual executable-JAR model.

---

# 3. Traditional Deployment vs Spring Boot

## Traditional

```text
WAR
 ↓
External Tomcat
 ↓
Deploy WAR
 ↓
Application
```

## Spring Boot

```text
Executable JAR
 ↓
java -jar
 ↓
Embedded Tomcat
 ↓
Application
```

This is one reason Boot applications are convenient to package and deploy.

---

# 4. What Happens During Startup?

Let's start here:

```java
public static void main(String[] args) {

    SpringApplication.run(
            EmployeeApplication.class,
            args
    );
}
```

High-level flow:

```text
main()
  ↓
SpringApplication.run()
  ↓
Create SpringApplication
  ↓
Prepare Environment
  ↓
Create ApplicationContext
  ↓
Apply configuration
  ↓
Component scanning
  ↓
Auto-configuration
  ↓
Bean creation
  ↓
Start embedded server
  ↓
Application ready
```

The exact internal implementation contains many more steps and lifecycle callbacks, but this is the right conceptual model.

---

# 5. What is `SpringApplication`?

`SpringApplication` is the central Bootstrap class used to launch a Spring application from `main()`. ([docs.spring.io](https://docs.spring.io/spring-boot/api/java/org/springframework/boot/SpringApplication.html))

Example:

```java
SpringApplication.run(
    EmployeeApplication.class,
    args
);
```

It determines how the application should be started and coordinates the Boot startup process.

---

# 6. How Does Boot Know This Is a Web Application?

Spring Boot examines the application's environment/classpath and determines the appropriate application type.

For a servlet web application, Boot creates a servlet-based `ApplicationContext`, specifically a web application context suitable for servlet applications.

Conceptually:

```text
Web dependencies present
        ↓
Web application detected
        ↓
Servlet WebApplicationContext
        ↓
Embedded servlet server
```

For a non-web application:

```text
No web application
        ↓
Regular ApplicationContext
```

So the dependencies you choose influence the kind of application Boot starts.

---

# 7. ApplicationContext

You already learned:

```text
ApplicationContext
    ↓
Spring IoC Container
```

During Boot startup:

```text
SpringApplication
       ↓
ApplicationContext
       ↓
Beans
```

The context manages things such as:

```text
Controllers
Services
Repositories
Configuration beans
Auto-configured infrastructure
```

---

# 8. What is `WebApplicationContext`?

For Spring MVC applications, the ApplicationContext is web-aware.

Conceptually:

```text
ApplicationContext
      ↓
WebApplicationContext
      ↓
Spring MVC infrastructure
```

This allows the application to integrate Spring's web components with the servlet environment.

---

# 9. Where Does `DispatcherServlet` Fit?

You learned Spring MVC earlier:

```text
Client
   ↓
DispatcherServlet
   ↓
Controller
```

With Boot, much of the setup of the `DispatcherServlet` and related web infrastructure is automatically configured.

So the full picture becomes:

```text
Browser / Client
      ↓
Embedded Tomcat
      ↓
DispatcherServlet
      ↓
Spring MVC
      ↓
Controller
```

Boot has simplified the setup; it hasn't removed the Spring MVC architecture.

---

# 10. Complete Web Request Flow

Suppose:

```http
GET /api/employees
```

Then:

```text
Client
  ↓
Port 8080
  ↓
Embedded Tomcat
  ↓
Servlet Filter Chain
  ↓
Spring Security Filters
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
EmployeeController
  ↓
EmployeeService
  ↓
Repository
  ↓
Database
```

This combines almost everything you've learned so far.

---

# 11. How Does the Server Know Which Port?

Default:

```text
8080
```

You can configure:

```properties
server.port=9090
```

Then:

```text
http://localhost:9090
```

The Boot configuration system maps `server.port` into the embedded server configuration.

---

# 12. Port 0

You can also use:

```properties
server.port=0
```

This tells the embedded server to choose an available random port.

This is particularly useful in tests where you don't want fixed-port conflicts.

Conceptually:

```text
Application
   ↓
server.port=0
   ↓
Operating system chooses available port
```

---

# 13. Context Path

You can configure:

```properties
server.servlet.context-path=/employee-api
```

Then:

```text
http://localhost:8080/employee-api/api/employees
```

instead of:

```text
http://localhost:8080/api/employees
```

This changes the application context path at the servlet-container level.

---

# 14. What Is the Default Context Path?

Normally:

```text
/
```

So:

```http
GET /api/employees
```

is directly handled at that application path.

You don't need to configure a context path unless your deployment architecture requires one.

---

# 15. How Is the Embedded Server Started?

Conceptually:

```text
SpringApplication
       ↓
ApplicationContext created
       ↓
Web environment detected
       ↓
Embedded web-server infrastructure configured
       ↓
Tomcat initialized
       ↓
Tomcat started
       ↓
Application ready
```

The exact Boot internals involve specialized web-server application-context classes and lifecycle management, but this is the practical architecture you need to understand.

---

# 16. What is `ServletWebServerApplicationContext`?

This is a useful interview-level internal class.

For servlet-based Boot applications, Spring Boot uses:

```text
ServletWebServerApplicationContext
```

Its job includes managing the embedded `WebServer` lifecycle along with the application context.

Conceptually:

```text
ServletWebServerApplicationContext
        ├── Spring Beans
        └── Embedded WebServer
```

This is one of the internal classes behind the "embedded server just starts" behavior.

---

# 17. Is Tomcat a Spring Bean?

The embedded web server is managed as part of Boot's web application lifecycle, but don't oversimplify it as:

> "Tomcat is just another `@Component`."

That's not the right mental model.

Think:

```text
Spring ApplicationContext
        ↓
Web Server Lifecycle
        ↓
Embedded Tomcat
```

Boot manages the embedded web-server infrastructure.

---

# 18. Why Can We Run `java -jar`?

Spring Boot's Maven/Gradle tooling can create an executable archive containing your application and the dependencies needed to run it.

So:

```bash
java -jar employee-api.jar
```

has enough information to start:

```text
JVM
 ↓
Boot application
 ↓
Spring
 ↓
Embedded server
```

The Spring Boot Maven plugin supports packaging applications as executable archives.

---

# 19. JAR vs WAR

This is a common interview question.

### JAR

Typical modern Boot deployment:

```text
application.jar
    ↓
java -jar
    ↓
Embedded server
```

### WAR

Traditional deployment:

```text
application.war
    ↓
External servlet container
```

Spring Boot still supports WAR deployment when your environment requires it, but executable JAR deployment is a very common approach.

---

# 20. Why JAR Is Popular for Microservices

Suppose:

```text
Employee Service
Order Service
Payment Service
```

Each can be:

```text
employee.jar
order.jar
payment.jar
```

Each process can contain:

```text
Application
+
Embedded server
+
Dependencies
```

Deployment becomes:

```text
Run employee.jar
Run order.jar
Run payment.jar
```

This fits naturally with containerized deployments.

---

# 21. What Happens When Port Is Already Used?

Suppose:

```text
App 1 → 8080
App 2 → 8080
```

The second application can fail to start because the operating system won't allow both processes to bind the same address/port combination.

You might see an error indicating that the port is already in use.

Change:

```properties
server.port=8081
```

or stop the other process.

---

# 22. Startup Failure

A very important Boot debugging principle:

Not every startup failure is caused by Tomcat.

Examples:

```text
BeanCreationException
Dependency conflict
Configuration error
Database configuration error
Port already in use
Invalid property
Missing class
Auto-configuration condition
```

So:

```text
Application won't start
```

doesn't mean:

```text
Tomcat problem
```

Always inspect the root cause in the startup logs.

---

# 23. Startup Events

Spring Boot publishes lifecycle events during startup.

You may encounter events such as:

```text
ApplicationStartingEvent
ApplicationEnvironmentPreparedEvent
ApplicationContextInitializedEvent
ApplicationPreparedEvent
ApplicationStartedEvent
ApplicationReadyEvent
ApplicationFailedEvent
```

You don't need to memorize all of them yet.

The important one for application readiness is:

```text
ApplicationReadyEvent
```

Conceptually:

```text
Beans ready
+
Server started
+
Application ready
```

---

# 24. `CommandLineRunner`

You can execute code after the application starts using:

```java
@Bean
CommandLineRunner runner() {

    return args -> {
        System.out.println(
            "Application started"
        );
    };
}
```

This runs during startup after the application context has been prepared.

You'll learn `CommandLineRunner` in more detail later.

---

# 25. `ApplicationRunner`

There is a similar interface:

```java
ApplicationRunner
```

Difference:

```text
CommandLineRunner
    ↓
String[] style arguments

ApplicationRunner
    ↓
Parsed ApplicationArguments
```

For now:

```text
Both
 ↓
Run startup logic
```

---

# 26. Application Startup Timeline

A useful simplified diagram:

```text
java -jar employee-api.jar
          ↓
       JVM starts
          ↓
  SpringApplication.run()
          ↓
Environment prepared
          ↓
ApplicationContext created
          ↓
@SpringBootApplication processed
          ↓
Component scanning
          ↓
Auto-configuration
          ↓
Beans instantiated
          ↓
Bean initialization/lifecycle
          ↓
Embedded Tomcat starts
          ↓
ApplicationStartedEvent
          ↓
Startup runners
          ↓
ApplicationReadyEvent
          ↓
READY ✅
```

The exact ordering among individual lifecycle callbacks is more detailed, but this is the correct high-level picture.

---

# 27. `ApplicationContext` vs Embedded Server

Don't confuse these.

### ApplicationContext

Manages Spring:

```text
Beans
Dependency Injection
Configuration
Lifecycle
```

### Embedded Server

Handles network/web serving:

```text
HTTP
TCP socket
Servlet execution
```

Conceptually:

```text
Spring
ApplicationContext
      │
      └──── Embedded Server
```

---

# 28. Why Does Spring Boot Need an Embedded Server at All?

Because Spring MVC is servlet-based.

A servlet application needs a servlet container to receive HTTP requests and invoke servlet processing.

With Boot:

```text
Embedded Tomcat
      ↓
Servlet infrastructure
      ↓
DispatcherServlet
      ↓
Spring MVC
```

Boot simply packages and starts the container for you.

---

# 29. Can We Replace Tomcat?

Yes.

Spring Boot supports different embedded servlet containers depending on the application's dependencies and configuration.

The important architectural point is:

```text
Boot Web Application
       ↓
Embedded Servlet Container
       ↓
Tomcat / another supported container
```

Tomcat is common, but the architecture is not fundamentally tied to Tomcat.

---

# 30. Interview Questions

### Why does Spring Boot use embedded Tomcat?

> It allows the application to run as a self-contained executable application without requiring a separately installed external servlet container.

### What happens when `SpringApplication.run()` executes?

> Boot prepares the environment, creates and configures the application context, performs component scanning and auto-configuration, initializes beans, starts the embedded web server for a web application, and eventually signals that the application is ready.

### JAR vs WAR?

> An executable JAR commonly includes the application and embedded server and runs with `java -jar`; a WAR is conventionally deployed into an external servlet container, though Spring Boot supports both deployment models.

### What is `ServletWebServerApplicationContext`?

> It is a Boot application-context implementation for servlet web applications that manages the Spring application context together with the embedded web server lifecycle.

### Where does DispatcherServlet run?

> Inside the servlet container, which in the executable JAR model is the embedded servlet server.

### Can we change the port?

Yes:

```properties
server.port=9090
```

### Why should the application class be in the root package?

> So component scanning naturally discovers components in its subpackages.

---

# 31. Best Practices

```text
✅ Prefer executable JARs for modern service deployments
✅ Keep server configuration externalized
✅ Don't hardcode deployment-specific ports in Java
✅ Know how to diagnose startup failures
✅ Keep the application class in the root package
✅ Use startup runners only for appropriate initialization logic
✅ Monitor ApplicationReadyEvent for readiness-related integration when needed
```

---

# 32. Complete Boot Startup Diagram

Keep this:

```text
                   java -jar app.jar
                          │
                          ▼
                SpringApplication.run()
                          │
                          ▼
                 Prepare Environment
                          │
                          ▼
                Create ApplicationContext
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
      Component Scanning        Auto-Configuration
             │                         │
             └────────────┬────────────┘
                          ▼
                     Bean Creation
                          │
                          ▼
                 Web Application Setup
                          │
                          ▼
                 Embedded Tomcat
                          │
                          ▼
                 DispatcherServlet
                          │
                          ▼
                   Application Ready
```

---

# 📍 Where We Are

```text id="px8m4q"
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters & Dependency Management
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server & Startup
│
└── ⏭️ Chapter 7 — Application Configuration ⭐⭐⭐⭐⭐
       ↓
       application.properties
       ↓
       application.yaml
       ↓
       Property sources
       ↓
       Environment
       ↓
       @Value
       ↓
       Configuration precedence
       ↓
       Environment variables
       ↓
       Command-line arguments
```

Next we'll get into **Spring Boot configuration**, which is one of the most important practical topics: how `application.properties`/YAML, environment variables, command-line arguments, and `@Value` work together, and which configuration wins when multiple sources provide the same property.
