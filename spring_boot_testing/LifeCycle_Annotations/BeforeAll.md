![image](https://github.com/user-attachments/assets/f41a2074-a68e-477f-b77b-ddfa267d652e)

---

### 🔹 What is `@BeforeAll`?

- `@BeforeAll` is a JUnit 5 annotation.
- It marks a method that runs **once before all test methods** in a test class.
- It’s used to perform **one-time setup** (like initializing resources).

---

### ✅ Purpose of `@BeforeAll`

- To **set up shared resources** (e.g., database connection, mock server, configuration).
- It’s useful when the setup is **expensive or time-consuming**, and you don’t want to repeat it before every test.

---

### ✅ Advantages

| Benefit | Explanation |
|--------|-------------|
| 🔁 One-time execution | Runs just once, saving time compared to `@BeforeEach`. |
| ✅ Shared setup | Useful for initializing things used across multiple tests. |
| 🚀 Performance | Improves test performance by avoiding repeated setup. |

---

### ❌ Disadvantages

| Limitation | Why it matters |
|------------|----------------|
| ❗ Must be `static` | Unless you're using `@TestInstance(Lifecycle.PER_CLASS)` |
| 🔄 Shared state risk | If not used carefully, it can lead to state leakage between tests |
| 🧪 Not ideal for isolated tests | Because all tests rely on the same setup |

---

### 🧪 Example

```java
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;

@TestInstance(TestInstance.Lifecycle.PER_CLASS) // Allows non-static @BeforeAll
class ServerTest {

    private MockServer server;

    @BeforeAll
    void startServer() {
        server = new MockServer();
        server.start();
        System.out.println("Server started");
    }

    @Test
    void testServerConnection() {
        assert server.isRunning();
    }

    @Test
    void testServerResponse() {
        assert server.getResponse().equals("OK");
    }
}
```

---

### ⚙️ When to Use `@BeforeAll`

Use it when:
- You need to set up **shared state or resource** once for all tests.
- Setup is **time-consuming** (e.g., spinning up containers, connecting to a test DB).
- You're doing **integration testing** and want to avoid repeated setup.

---

Would you like to see an example with `@BeforeAll` + database setup or test containers (like PostgreSQL or MongoDB)?
