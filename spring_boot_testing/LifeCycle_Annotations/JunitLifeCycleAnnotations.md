The image outlines **JUnit Lifecycle Annotations**, which are super useful for managing setup and cleanup tasks when writing tests. Here’s a brief breakdown of each:

---

### 🔄 What are Lifecycle Annotations?

JUnit provides these annotations to define **what should happen before or after** your tests run. They help you avoid repeating setup and teardown code across multiple test methods.

---

### 🧪 Lifecycle Annotations Explained

| Annotation       | When it Runs | Purpose |
|------------------|--------------|---------|
| **`@BeforeEach`** | Before **every test method** | Use to set up test data or state that is needed before each test. |
| **`@AfterEach`**  | After **every test method**  | Use to clean up or reset things after each test. |
| **`@BeforeAll`**  | Once **before all tests** in the class | Initialize expensive/shared resources (like DB connection). Must be `static`. |
| **`@AfterAll`**   | Once **after all tests** in the class  | Clean up shared resources (like closing DB connection). Must be `static`. |

---

### ✅ Example (JUnit 5)

```java
import org.junit.jupiter.api.*;

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class ExampleTest {

    @BeforeAll
    void setupAll() {
        System.out.println("Before all tests");
    }

    @BeforeEach
    void setup() {
        System.out.println("Before each test");
    }

    @Test
    void testOne() {
        System.out.println("Running test one");
    }

    @Test
    void testTwo() {
        System.out.println("Running test two");
    }

    @AfterEach
    void teardown() {
        System.out.println("After each test");
    }

    @AfterAll
    void teardownAll() {
        System.out.println("After all tests");
    }
}
```

---

### 🧠 Quick Tip

- `@BeforeEach` and `@AfterEach` are great for **isolated, repeatable tests**.
- `@BeforeAll` and `@AfterAll` are ideal for **one-time setup/teardown**, like connecting to a database or starting a server.

---

Let me know if you want a visual flow of the execution order or a real-world testing use case!
