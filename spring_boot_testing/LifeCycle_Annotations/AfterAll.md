Absolutely! Let’s now go over **`@AfterAll`** in JUnit — the final piece of the test lifecycle puzzle. Here's a clear breakdown of what it is, why it’s used, pros/cons, a simple example, and when to use it. 👇

---

### 🔹 What is `@AfterAll`?

- `@AfterAll` is a JUnit 5 annotation.
- It marks a method that should run **once after all test methods** in the test class have finished.
- It’s used to **clean up shared resources** initialized in `@BeforeAll`.

---

### ✅ Purpose of `@AfterAll`

- To **tear down or release shared resources** used across multiple tests.
- Helps prevent **memory leaks**, **resource locks**, or **leftover test state**.

---

### ✅ Advantages

| Benefit | Explanation |
|---------|-------------|
| ✅ One-time cleanup | Efficient for global resources (e.g., closing DB, stopping servers). |
| 🧹 Prevents leaks | Makes sure resources are released properly after all tests run. |
| 🧪 Complements `@BeforeAll` | Often used together for setup/teardown. |

---

### ❌ Disadvantages

| Limitation | Why it matters |
|------------|----------------|
| ❗ Must be `static` | Unless using `@TestInstance(Lifecycle.PER_CLASS)` |
| ⚠️ Order matters | If cleanup is too early or late, it may affect debugging |
| 🔁 Not for test-specific cleanup | Use `@AfterEach` for that instead |

---

### 🧪 Example

```java
import org.junit.jupiter.api.*;

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class DatabaseTests {

    private FakeDatabaseConnection db;

    @BeforeAll
    void setupDatabase() {
        db = new FakeDatabaseConnection();
        db.connect();
        System.out.println("Database connected");
    }

    @Test
    void testInsert() {
        db.insert("test1");
        assert db.hasData("test1");
    }

    @Test
    void testDelete() {
        db.insert("test2");
        db.delete("test2");
        assert !db.hasData("test2");
    }

    @AfterAll
    void closeDatabase() {
        db.disconnect();
        System.out.println("Database disconnected");
    }
}
```

---

### 📌 When to Use `@AfterAll`

Use it when:
- You opened a shared resource in `@BeforeAll` (e.g., DB, server, external system).
- You want to **log something once** at the end of the test run.
- You're doing **integration testing** or **performance testing** with shared state.

---

Would you like a full lifecycle diagram showing `@BeforeAll`, `@BeforeEach`, `@AfterEach`, and `@AfterAll` in action together?
