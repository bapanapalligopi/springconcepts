Great question! Let's talk about **disabling test cases** in **JUnit**, which is something you'll often do during development or debugging.

---

### ❓ What is "Disabling" a Test?

Disabling a test means **preventing it from running** during test execution. It’s like temporarily commenting it out — but cleaner and more controlled.

---

### 🔧 Why Disable Test Cases?

Here are some common reasons:

| Reason | Explanation |
|--------|-------------|
| 🔧 **Still in development** | The test isn’t finished yet. |
| 🧪 **Flaky test** | It sometimes fails randomly; you want to investigate without blocking builds. |
| 🚫 **Feature not ready** | The test is for a feature that hasn’t been implemented. |
| ⚙️ **Setup dependency** | External service or data isn’t available. |

---

### ✅ How to Disable Tests in JUnit

#### 👉 JUnit 5 (Jupiter)

Use the `@Disabled` annotation.

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class MyTests {

    @Test
    @Disabled("Not implemented yet")
    void testNotReady() {
        // This will be skipped
    }
}
```

- You can optionally add a string explanation: `@Disabled("reason")`.
- This works on **both methods and classes**.

#### 👉 Disable an Entire Class

```java
@Disabled("Feature under development")
class PaymentTests {
    // All test methods inside will be skipped
}
```

---

### 🧪 Output Behavior

When JUnit runs:
- It will **report the test as skipped**, not failed.
- Most IDEs/test runners will show a yellow mark or “skipped” label.

---

### 📌 JUnit 4 Equivalent

Use `@Ignore` from `org.junit.Ignore`:

```java
import org.junit.Ignore;
import org.junit.Test;

@Ignore("Not working yet")
public class MyOldTests {
    @Test
    public void testFeature() {
        // will not run
    }
}
```

---

### ✅ Summary

| Feature | JUnit 5 | JUnit 4 |
|---------|---------|---------|
| Annotation | `@Disabled` | `@Ignore` |
| Can disable method? | ✅ Yes | ✅ Yes |
| Can disable class? | ✅ Yes | ✅ Yes |
| Optional reason | ✅ Yes | ✅ Yes |

---

Want me to show how to conditionally disable tests (like based on environment or OS)?
