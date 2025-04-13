![image](https://github.com/user-attachments/assets/b7e3d819-a8c4-4eee-a631-d71f2833b6a9)


![image](https://github.com/user-attachments/assets/685f0c84-2d5e-4a7e-9262-01c1f17db233)


Sure! Let’s explain **`@Nested` vs without @Nested** — briefly, simply, and clearly, with examples so you can **see the difference** at a glance. 👇

---

## 🔹 Without `@Nested`

All tests are in **one flat structure**, even if they're testing different scenarios.

### ✅ Example (without `@Nested`):

```java
class LoginServiceTest {

    LoginService service = new LoginService();

    @Test
    void loginWithValidCredentials() {
        service.login("user", "pass");
        assert service.isLoggedIn();
    }

    @Test
    void loginWithInvalidCredentials() {
        service.login("user", "wrong");
        assert !service.isLoggedIn();
    }

    @Test
    void guestCannotAccessDashboard() {
        service.login("guest", "guest");
        assert !service.canAccessDashboard();
    }
}
```

### ⚠️ Problem:
- All tests are mixed together.
- Harder to **organize** or understand which tests belong to which context.

---

## 🔸 With `@Nested`

You **group related tests together** using inner classes — each with its **own setup** if needed.

### ✅ Example (with `@Nested`):

```java
class LoginServiceTest {

    LoginService service;

    @BeforeEach
    void setup() {
        service = new LoginService();
    }

    @Nested
    class WhenUserLogsIn {

        @Test
        void withValidCredentials() {
            service.login("user", "pass");
            assert service.isLoggedIn();
        }

        @Test
        void withInvalidCredentials() {
            service.login("user", "wrong");
            assert !service.isLoggedIn();
        }
    }

    @Nested
    class WhenGuestLogsIn {

        @Test
        void cannotAccessDashboard() {
            service.login("guest", "guest");
            assert !service.canAccessDashboard();
        }
    }
}
```

---

## ✅ Benefits of Using `@Nested`

| Feature | Without `@Nested` | With `@Nested` |
|--------|--------------------|----------------|
| 🧠 Readability | All tests together | Organized by scenario |
| 🔁 Setup control | One setup for all | Separate setup per group |
| 📂 Grouping | Manual (via method names) | Logical and automatic |
| 📋 Test report clarity | Flat list | Nested view per group |

---

## 🧠 Summary

| Aspect | Without `@Nested` | With `@Nested` |
|--------|--------------------|----------------|
| Structure | Flat | Hierarchical |
| Readability | Lower | Higher |
| Use case | Simple test classes | Complex test scenarios with context |

---

Want me to show how it looks in a test report (IDE) or combine with other annotations like `@BeforeEach` inside `@Nested`?


Great topic! Let's dig into **`@Nested`** in **JUnit 5**, which is super useful for organizing and grouping related test cases in a clean, readable way. Here's the full breakdown 👇

---

## 🔹 What is `@Nested`?

- `@Nested` is a **JUnit 5 annotation** that allows you to define **inner test classes** within a test class.
- It's used to **group related tests** together — especially when they share a **common setup context**.

---

## ✅ Purpose of `@Nested`

- Organize tests **logically** into sections.
- Create **hierarchies** of test scenarios (like “when user is logged in” vs “when user is not logged in”).
- Share specific setup logic for a subset of tests using `@BeforeEach`/`@AfterEach`.

---

## 🧪 Basic Example

```java
import org.junit.jupiter.api.*;

class UserServiceTest {

    UserService userService;

    @BeforeEach
    void setup() {
        userService = new UserService();
    }

    @Nested
    class WhenUserIsAdmin {

        @BeforeEach
        void setupAdmin() {
            userService.loginAs("admin");
        }

        @Test
        void canAccessAdminPanel() {
            Assertions.assertTrue(userService.canAccessAdminPanel());
        }

        @Test
        void canDeleteUsers() {
            Assertions.assertTrue(userService.canDeleteUsers());
        }
    }

    @Nested
    class WhenUserIsGuest {

        @BeforeEach
        void setupGuest() {
            userService.loginAs("guest");
        }

        @Test
        void cannotAccessAdminPanel() {
            Assertions.assertFalse(userService.canAccessAdminPanel());
        }
    }
}
```

---

## 💡 Explanation

- Outer class setup (`@BeforeEach`) runs before **every test**, including nested ones.
- Inner `@Nested` classes can have their **own setup and test methods**.
- You can use `@BeforeEach`, `@AfterEach`, `@Test`, etc., inside the nested class as usual.

---

## ✅ Advantages

| Benefit | Why it’s helpful |
|--------|------------------|
| 🗂️ Logical grouping | Cleanly groups tests for different scenarios or contexts. |
| 🔁 Separate setup | Allows custom setup for each group of tests. |
| 📖 Better readability | Makes test structure easy to read and follow. |
| ✅ IDE/Test report friendly | Most IDEs and test runners show nested structure clearly. |

---

## ❌ Limitations

| Limitation | Notes |
|------------|-------|
| No further nesting | You **can’t nest inside a nested** class (only one level deep). |
| Can get bulky | Too many nested classes may complicate the test file. |
| Works with JUnit 5 only | Not available in JUnit 4. |

---

## 🎯 When to Use `@Nested`

- You have multiple test scenarios with **different setup logic**.
- You want to **group related test cases** (e.g., “valid inputs” vs “invalid inputs”).
- You're testing **stateful services** where user roles, states, or conditions matter.

---

## 🧪 Real-World Use Case: Form Validation

```java
class FormValidatorTest {

    FormValidator validator;

    @BeforeEach
    void init() {
        validator = new FormValidator();
    }

    @Nested
    class WhenInputIsEmpty {

        @Test
        void shouldReturnError() {
            Assertions.assertFalse(validator.validate(""));
        }
    }

    @Nested
    class WhenInputIsValid {

        @Test
        void shouldReturnTrue() {
            Assertions.assertTrue(validator.validate("Valid Input"));
        }
    }
}
```

---

Want to see `@Nested` combined with `@ParameterizedTest` or more complex stateful setups? Let me know!



