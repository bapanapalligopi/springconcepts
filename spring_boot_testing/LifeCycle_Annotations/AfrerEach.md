Sure! Let's go over **`@AfterEach`** in JUnit in a clear and structured way, covering what it is, why it’s useful, its pros and cons, and a practical example.

---

### 🔹 What is `@AfterEach`?

- `@AfterEach` is a JUnit 5 annotation.
- It marks a method that runs **after each test method** in a test class.
- It's commonly used to **clean up or reset resources** used during a test.

---

### ✅ Purpose of `@AfterEach`

- To **clean up** resources (e.g., closing files, clearing data structures, resetting mocks).
- To **ensure test isolation** by resetting the environment after each test.
- To avoid repeated cleanup logic inside every test method.

---

### ✅ Advantages

1. **Cleaner tests** – no repetitive cleanup code.
2. **Improves isolation** – ensures that one test doesn’t affect another.
3. **Automated teardown** – especially useful when using shared resources.

---

### ❌ Disadvantages

1. **Overhead** – if the cleanup is heavy, it adds execution time to every test.
2. **Overuse** – unnecessary use can lead to complex or hard-to-maintain code.
3. **Not for one-time teardown** – if you only need cleanup once after all tests, use `@AfterAll` instead.

---

### 🧪 Example

```java
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class DatabaseTest {

    private FakeDatabaseConnection db;

    @BeforeEach
    void setup() {
        db = new FakeDatabaseConnection();
        db.connect();
    }

    @Test
    void testInsertData() {
        db.insert("Test Data");
        assert db.hasData("Test Data");
    }

    @Test
    void testDeleteData() {
        db.insert("ToDelete");
        db.delete("ToDelete");
        assert !db.hasData("ToDelete");
    }

    @AfterEach
    void teardown() {
        db.disconnect(); // Clean up after each test
    }
}
```

---

### 📌 When to Use `@AfterEach`

Use it when:
- You open a resource (e.g., file, DB, network) in `@BeforeEach` or test methods.
- You need to **reset shared variables** or test data after each test.
- You're using **mocking frameworks** that require cleanup/reset after test execution.

---

Would you like to see how `@AfterEach` works in combination with mocking frameworks like Mockito or with parameterized tests?
