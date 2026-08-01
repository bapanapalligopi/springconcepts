# Spring Boot — Chapter 11: Spring Boot Testing ⭐⭐⭐⭐⭐

> **"If your application isn't tested, you don't know if it actually works."**

Testing is one of the most important skills in Spring Boot development. Enterprise applications often have **thousands of automated tests**.

In this chapter, you'll learn:

* Unit Testing
* Integration Testing
* Spring Test Context
* MockMvc
* Test Slices (`@WebMvcTest`, `@DataJpaTest`, etc.)
* Mockito
* Testcontainers
* Best practices

---

# 1. Why Do We Test?

Suppose we have:

```java
@Service
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }
}
```

How do we know it works?

By writing a test.

```java
@Test
void shouldAddNumbers() {

    CalculatorService service = new CalculatorService();

    assertEquals(5, service.add(2, 3));
}
```

Without tests:

```text
Write Code
      ↓
Deploy
      ↓
Hope it works ❌
```

With tests:

```text
Write Code
      ↓
Run Tests
      ↓
Deploy with confidence ✅
```

---

# 2. Types of Testing

```text
                Testing
                   │
    ┌──────────────┼──────────────┐
    ▼              ▼              ▼
 Unit Test   Integration Test   End-to-End Test
```

---

## Unit Test

Tests **one class** in isolation.

Example:

```text
EmployeeService
```

Dependencies:

```text
Repository

↓

Mock
```

Fast.

No database.

No Spring Boot startup.

---

## Integration Test

Tests multiple components together.

Example:

```text
Controller

↓

Service

↓

Repository

↓

Database
```

Starts Spring.

May use a real database.

---

## End-to-End Test

Tests the complete application.

```text
Browser

↓

REST API

↓

Database

↓

External Services
```

Usually slower.

---

# 3. Testing Pyramid

```text
          E2E
        (Few Tests)
            ▲
    Integration Tests
      (Some Tests)
            ▲
      Unit Tests
    (Many Thousands)
```

Why?

Unit tests are:

* Fast
* Cheap
* Easy

Integration tests are slower.

End-to-end tests are the slowest.

---

# 4. JUnit 5

Spring Boot uses **JUnit 5** by default.

Example:

```java
class CalculatorTest {

    @Test
    void shouldAddNumbers() {

        CalculatorService service = new CalculatorService();

        assertEquals(5, service.add(2,3));
    }
}
```

Common assertions:

```java
assertEquals()

assertTrue()

assertFalse()

assertNull()

assertNotNull()

assertThrows()
```

---

# 5. Testing Spring Beans

Suppose:

```java
@Service
public class EmployeeService {

}
```

Instead of:

```java
new EmployeeService();
```

Spring creates it.

Testing Spring Beans requires the Spring Test Context.

---

# 6. `@SpringBootTest`

The most powerful testing annotation.

```java
@SpringBootTest
class EmployeeApplicationTests {

}
```

Spring Boot starts:

```text
Application Context

↓

Beans

↓

AutoConfiguration

↓

Database

↓

Configuration
```

Almost identical to production.

---

# 7. What Happens Internally?

```text
JUnit

↓

Spring Test Runner

↓

ApplicationContext

↓

Dependency Injection

↓

Test Executes
```

Spring loads the application context before executing the tests.

---

# 8. Injecting Beans

Example:

```java
@SpringBootTest
class EmployeeServiceTest {

    @Autowired
    private EmployeeService service;

    @Test
    void shouldNotBeNull() {

        assertNotNull(service);
    }
}
```

Spring injects the real bean.

---

# 9. Why `@SpringBootTest` Can Be Slow

Imagine the application has:

```text
150 Beans

↓

Datasource

↓

Security

↓

Caching

↓

Messaging

↓

Actuator
```

Every test starts all of these.

Large applications may take several seconds to initialize.

That's why slice tests exist.

---

# 10. Slice Testing

Instead of loading everything:

```text
Entire Spring Boot

↓

Slow
```

Load only one layer.

```text
Controller Only

↓

Repository Only

↓

JSON Only
```

Much faster.

---

# 11. `@WebMvcTest`

Loads only:

```text
Controller

↓

DispatcherServlet

↓

MVC Configuration
```

Not loaded:

```text
Repository

Service

Database
```

Example:

```java
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

}
```

---

# 12. MockMvc

`MockMvc` simulates HTTP requests without starting a real server.

Example:

```java
@Autowired
private MockMvc mockMvc;
```

Test:

```java
mockMvc.perform(get("/employees"))
       .andExpect(status().isOk());
```

Flow:

```text
Mock HTTP Request

↓

DispatcherServlet

↓

Controller

↓

Response
```

Very fast.

---

# 13. Example Controller Test

Controller:

```java
@RestController
@RequestMapping("/employees")
class EmployeeController {

    @GetMapping
    public String hello() {
        return "Hello";
    }
}
```

Test:

```java
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    void shouldReturnHello() throws Exception {

        mockMvc.perform(get("/employees"))
               .andExpect(status().isOk())
               .andExpect(content().string("Hello"));
    }
}
```

---

# 14. What If Controller Uses a Service?

Controller:

```java
@RestController
class EmployeeController {

    private final EmployeeService service;

    EmployeeController(EmployeeService service) {
        this.service = service;
    }
}
```

`@WebMvcTest` **does not create** `EmployeeService`.

We mock it.

```java
@MockBean
private EmployeeService service;
```

Then:

```java
when(service.getAll())
        .thenReturn(List.of(...));
```

Spring injects the mock into the controller.

---

# 15. Mockito

Mockito creates fake objects.

Example:

```java
EmployeeRepository repository =
    mock(EmployeeRepository.class);
```

Instead of real database:

```text
Repository

↓

Mock
```

---

# 16. Mock Behavior

Example:

```java
when(repository.findById(1L))
        .thenReturn(Optional.of(employee));
```

Now:

```java
repository.findById(1L);
```

returns:

```text
employee
```

without accessing a database.

---

# 17. Verifying Calls

Mockito can verify interactions.

```java
verify(repository)
        .save(employee);
```

Meaning:

```text
Was save() called?
```

Useful for service-layer testing.

---

# 18. Unit Testing Service Layer

```java
class EmployeeServiceTest {

    EmployeeRepository repository =
        mock(EmployeeRepository.class);

    EmployeeService service =
        new EmployeeService(repository);

    @Test
    void shouldFindEmployee() {

        Employee employee =
            new Employee(1L, "John");

        when(repository.findById(1L))
                .thenReturn(Optional.of(employee));

        Employee result =
            service.findById(1L);

        assertEquals("John", result.getName());
    }
}
```

Notice:

No Spring.

No Boot.

Very fast.

---

# 19. `@DataJpaTest`

Tests repositories.

Loads:

```text
JPA

Hibernate

Repositories

EntityManager
```

Not loaded:

```text
Controllers

Security

Services
```

Example:

```java
@DataJpaTest
class EmployeeRepositoryTest {

}
```

Usually uses an in-memory database unless configured otherwise.

---

# 20. Example Repository Test

```java
@DataJpaTest
class EmployeeRepositoryTest {

    @Autowired
    EmployeeRepository repository;

    @Test
    void shouldSaveEmployee() {

        Employee employee =
            new Employee(null, "John");

        Employee saved =
            repository.save(employee);

        assertNotNull(saved.getId());
    }
}
```

---

# 21. `@JsonTest`

Tests JSON serialization.

Loads:

```text
Jackson

JSON Configuration
```

Useful for:

```text
ObjectMapper

Serialization

Deserialization
```

---

# 22. `@RestClientTest`

Tests REST clients like:

```java
RestTemplate

WebClient

RestClient
```

without calling real external services.

---

# 23. Testcontainers

Enterprise applications usually don't test against H2 alone.

Instead:

```text
Docker

↓

PostgreSQL Container

↓

Real Database
```

Example:

```java
@Testcontainers
@SpringBootTest
class EmployeeApplicationTests {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16");
}
```

Now your tests run against a real PostgreSQL instance in Docker.

Benefits:

* Same database engine as production
* More reliable than in-memory substitutes
* Easy cleanup after tests

---

# 24. Testing REST APIs

Using `MockMvc`:

```java
mockMvc.perform(post("/employees")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {
              "name":"John"
            }
        """))
       .andExpect(status().isCreated());
```

This verifies both request handling and response status.

---

# 25. Common Testing Strategy

```text
Controller
      │
@WebMvcTest
      ▼
Service
      │
Mockito Unit Tests
      ▼
Repository
      │
@DataJpaTest
      ▼
Entire Application
      │
@SpringBootTest
      ▼
Testcontainers
```

Each layer uses the most appropriate test type.

---

# 26. Common Mistakes

❌ Using `@SpringBootTest` for every test

❌ Connecting unit tests to a real database

❌ Mocking everything in integration tests

❌ Writing only happy-path tests

❌ Sharing mutable state between tests

---

# 27. Best Practices

```text
✅ Write many unit tests

✅ Add integration tests for critical flows

✅ Use @WebMvcTest for controllers

✅ Use @DataJpaTest for repositories

✅ Use MockMvc for REST APIs

✅ Use Mockito to isolate dependencies

✅ Use Testcontainers for database integration

✅ Keep tests independent

✅ Name tests clearly (shouldCreateEmployee, shouldReturn404WhenEmployeeNotFound)

✅ Test success and failure scenarios
```

---

# 28. Interview Questions

### What is `@SpringBootTest`?

> It starts the full Spring Boot application context, allowing you to test the application as a whole.

---

### What is `@WebMvcTest`?

> A slice test that loads only the Spring MVC layer (controllers and MVC infrastructure), making controller tests fast.

---

### Why use `MockMvc`?

> To simulate HTTP requests and verify responses without starting an actual web server.

---

### What is Mockito?

> A mocking framework used to create fake implementations of dependencies for unit testing.

---

### What is `@DataJpaTest`?

> A slice test that loads only JPA-related components, making repository testing faster than loading the full application.

---

### Why use Testcontainers?

> It runs tests against real infrastructure (such as PostgreSQL or MySQL) inside Docker containers, providing production-like integration testing.

---

# 29. Complete Testing Flow

```text
                    Spring Boot Testing

                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
    Unit Tests      Integration Tests    End-to-End

        │                 │                 │
     Mockito        @SpringBootTest     Browser/API

        │
        ├─────────────┐
        ▼             ▼
   @WebMvcTest   @DataJpaTest

        │             │
     MockMvc      Test Database

        └─────────────┬─────────────┘
                      ▼
              Testcontainers
                      ▼
             Production-like Tests
```

---

# 📍 Where We Are

```text
Spring Boot
│
├── ✅ Chapter 1 — Why Spring Boot?
├── ✅ Chapter 2 — @SpringBootApplication
├── ✅ Chapter 3 — Starters & Dependency Management
├── ✅ Chapter 4 — Auto-Configuration
├── ✅ Chapter 5 — Component Scanning
├── ✅ Chapter 6 — Embedded Server & Startup
├── ✅ Chapter 7 — Externalized Configuration
├── ✅ Chapter 8 — Profiles
├── ✅ Chapter 9 — @ConfigurationProperties
├── ✅ Chapter 10 — Spring Boot Actuator
├── ✅ Chapter 11 — Spring Boot Testing ⭐⭐⭐⭐⭐
│
└── ⏭️ Chapter 12 — Spring Boot Advanced Features ⭐⭐⭐⭐⭐
       ↓
       Application Events
       ↓
       CommandLineRunner & ApplicationRunner
       ↓
       Banner Customization
       ↓
       Lazy Initialization
       ↓
       Startup Hooks
       ↓
       Graceful Shutdown
       ↓
       DevTools
       ↓
       AOT & Native Images
```

## 🎯 After Chapter 12

The Spring Boot course will be complete. Then the natural next step is to build a **complete enterprise Employee Management System** that combines:

* Spring Boot
* Spring Security (JWT)
* Spring Data JPA
* Validation
* Exception Handling
* REST API Best Practices
* Testing
* Actuator
* Docker
* PostgreSQL
* Redis (optional)
* Kafka (optional)

This capstone project ties together everything you've learned into a production-ready application.
