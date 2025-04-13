It looks like you're referring to JUnit's `assertThrowsExactly`, but with a small typo (`assertThrowsxaclty`). Here's a quick guide to using `assertThrowsExactly` in JUnit.

---

### ✅ `assertThrowsExactly` in JUnit

In **JUnit 5**, the method you're probably looking for is:

```java
org.junit.jupiter.api.Assertions.assertThrowsExactly
```

It is used to assert that a specific **exception type** is thrown — **exactly** that type, not a subclass.

---

### 🔍 Syntax

```java
assertThrowsExactly(
    Class<T> expectedType,
    Executable executable
)
```

- `expectedType`: The exact class of the exception you expect.
- `executable`: A lambda or method reference that throws the exception.

---

### 📌 Example

```java
import static org.junit.jupiter.api.Assertions.assertThrowsExactly;
import org.junit.jupiter.api.Test;

class MyTest {
    @Test
    void testThrowsExactly() {
        assertThrowsExactly(
            IllegalArgumentException.class,
            () -> {
                throw new IllegalArgumentException("Invalid arg");
            }
        );
    }
}
```

This will fail if:
- A different type of exception is thrown (e.g., a subclass like `NumberFormatException`)
- No exception is thrown

---

### ⚠️ Difference from `assertThrows`

- `assertThrows()` passes if the exception is **of the given type or a subclass**.
- `assertThrowsExactly()` passes **only if the exception is exactly of the given type**.

---

Want a cheat sheet or helper for exception testing? Or do you have a specific test you're writing?
