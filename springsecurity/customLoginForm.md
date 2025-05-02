Certainly! Let’s break down what **custom form login** means in **Spring Security** and how it works.

---

## 🔐 What Is "Form Login" in Spring Security?

Spring Security provides a **built-in login form** when you secure a web app. When a user tries to access a protected page, it automatically redirects them to a default login page at `/login`.

But in real-world apps, you usually want to:

* Customize the **look and feel** of the login form.
* Change the **login URL** and **login handling behavior**.
* Show custom error or logout messages.

This is called **custom form login**.

---

## ✅ How Does Custom Form Login Work?

You override Spring Security’s defaults by configuring:

* A **custom login page**
* A **custom login processing URL**
* What happens **after login** (success or failure)

---

## 🔧 Step-by-Step: How to Do Custom Form Login

### 1. **Security Configuration**

Override the `configure(HttpSecurity http)` method in a class extending `WebSecurityConfigurerAdapter`.

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/login", "/resources/**").permitAll() // allow access to login page
            .anyRequest().authenticated()
            .and()
        .formLogin()
            .loginPage("/login")               // Custom login page (GET)
            .loginProcessingUrl("/doLogin")    // Custom POST endpoint for login
            .defaultSuccessUrl("/home")        // Where to go on success
            .failureUrl("/login?error")        // Where to go on failure
            .permitAll();
}
```

---

### 2. **Custom Login Page (`/login`)**

You provide your own HTML or JSP page to collect username and password:

```html
<form action="/doLogin" method="post">
  <input type="text" name="username" placeholder="Username"/>
  <input type="password" name="password" placeholder="Password"/>
  <input type="submit" value="Login"/>
</form>
```

> The form’s `action` must match `loginProcessingUrl()` in your config.

---

### 3. **Spring Handles Login Automatically**

You **do not** need a controller for `/doLogin`. Spring Security handles it internally:

* It reads `username` and `password` from the form.
* Authenticates using your configured `UserDetailsService` or in-memory users.
* If valid: redirects to `/home`
* If invalid: redirects to `/login?error`

---

### 4. **SecurityContext and Session**

If login succeeds:

* Spring stores the user's info in a `SecurityContext` inside the session.
* Subsequent requests don’t need to log in again, as long as the session is valid.

---

## 🧠 Summary

| Feature              | Default Behavior           | Custom Form Login Does          |
| -------------------- | -------------------------- | ------------------------------- |
| Login Page           | Auto-generated at `/login` | You create your own at `/login` |
| Login POST URL       | `/login`                   | You define, e.g., `/doLogin`    |
| Success Redirect     | `/`                        | You define, e.g., `/home`       |
| Failure Redirect     | `/login?error`             | You can customize               |
| Authentication Logic | Spring internal            | Still handled by Spring         |

---

Great question — let’s clarify **where controllers fit in** during **custom form login with Spring Security**.

---

## 🔍 Where Are Controllers in Custom Form Login?

In a **custom form login setup**, you usually only need **one controller**:
🔹 A controller to **serve the login page** (`GET /login`) and possibly other pages like `home.jsp`.

### ✅ 1. Login Controller (Optional but Recommended)

```java
@Controller
public class LoginController {

    // Shows the custom login form (login.jsp or login.html)
    @GetMapping("/login")
    public String showLoginPage() {
        return "login"; // returns login.jsp or login.html
    }

    // Landing page after successful login
    @GetMapping("/home")
    public String homePage() {
        return "home"; // returns home.jsp or home.html
    }

    // Optional: Show access denied page
    @GetMapping("/access-denied")
    public String accessDenied() {
        return "denied"; // returns denied.jsp
    }
}
```

> 🔸 **Note:** You do **not** need a controller for the POST login (`/doLogin`).
> Spring Security intercepts it via `.loginProcessingUrl("/doLogin")` and handles it automatically.

---

## 🔄 What Spring Security Handles (No Controller Needed):

| URL (Path)      | Who Handles It?            | Notes                          |
| --------------- | -------------------------- | ------------------------------ |
| `POST /doLogin` | Spring Security internally | Processes login form           |
| `/logout`       | Spring Security internally | Handles logout if configured   |
| `/login?error`  | Handled by your login page | Shows error message (if added) |

---

## 🧱 Your Custom Controller + Spring Security Together

* Your **controller** serves the **login page** (`GET /login`) and post-login pages.
* **Spring Security** handles the **form submission** (`POST /doLogin`) and authentication logic.
* If credentials are valid: forwards to `/home` (your controller).
* If invalid: redirects back to `/login?error`.

---

## ✅ Example URL Flow

| Step | URL                      | What Happens                       |
| ---- | ------------------------ | ---------------------------------- |
| 1    | `/login`                 | Your controller returns login form |
| 2    | `/doLogin`               | Spring handles POST authentication |
| 3a   | Success → `/home`        | Controller returns home page       |
| 3b   | Failure → `/login?error` | Login page shows error             |

---

Sure! Here's a **simple and minimal JSP-based login setup** using **Spring Security** with a custom login form.

---

## ✅ 1. `login.jsp` – Simple Custom Login Page

Place this in: `src/main/webapp/WEB-INF/views/login.jsp`

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>

<h2>Login Page</h2>

<c:if test="${param.error != null}">
    <p style="color:red;">Invalid username or password!</p>
</c:if>

<c:if test="${param.logout != null}">
    <p style="color:green;">You have logged out.</p>
</c:if>

<form action="${pageContext.request.contextPath}/doLogin" method="post">
    <label>Username:</label>
    <input type="text" name="username" /><br/><br/>
    
    <label>Password:</label>
    <input type="password" name="password" /><br/><br/>
    
    <input type="submit" value="Login" />
</form>

</body>
</html>
```

---

## ✅ 2. `home.jsp` – After Login Page

Place this in: `src/main/webapp/WEB-INF/views/home.jsp`

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>

<h2>Welcome to the Home Page!</h2>
<p>You are logged in.</p>
<a href="<c:url value='/logout'/>">Logout</a>

</body>
</html>
```

---

## ✅ 3. Controller to Serve JSPs

```java
@Controller
public class LoginController {

    @GetMapping("/login")
    public String showLogin() {
        return "login"; // returns login.jsp
    }

    @GetMapping("/home")
    public String showHome() {
        return "home"; // returns home.jsp
    }
}
```

---

## ✅ 4. Security Config (Simplified)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("gopi")
            .password("{noop}gopi1234")
            .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/doLogin")
                .defaultSuccessUrl("/home", true)
                .failureUrl("/login?error=true")
                .permitAll()
                .and()
            .logout()
                .logoutSuccessUrl("/login?logout=true")
                .permitAll();
    }
}
```

---

That’s it! This is a clean and working setup using just:

* `login.jsp`
* `home.jsp`
* `LoginController`
* `SecurityConfig`

Here’s a full and **simple JSP setup** for your controller with three views: `login.jsp`, `home.jsp`, and `denied.jsp`.

---

## ✅ 1. `login.jsp` – Login Page

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>

<h2>Login Page</h2>

<c:if test="${param.error != null}">
    <p style="color:red;">Invalid username or password!</p>
</c:if>

<c:if test="${param.logout != null}">
    <p style="color:green;">You have successfully logged out.</p>
</c:if>

<form action="${pageContext.request.contextPath}/doLogin" method="post">
    <label>Username:</label>
    <input type="text" name="username" /><br/><br/>

    <label>Password:</label>
    <input type="password" name="password" /><br/><br/>

    <input type="submit" value="Login" />
</form>

</body>
</html>
```

---

## ✅ 2. `home.jsp` – Home Page After Successful Login

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>

<h2>Welcome, you are successfully logged in!</h2>

<a href="<c:url value='/logout'/>">Logout</a>

</body>
</html>
```

---

## ✅ 3. `denied.jsp` – Access Denied Page

```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Access Denied</title>
</head>
<body>

<h2>Access Denied</h2>
<p>You do not have permission to access this page.</p>

<a href="<c:url value='/login'/>">Go back to login</a>

</body>
</html>
```

---

## ✅ Place JSPs Here:

```plaintext
src/
└── main/
    └── webapp/
        └── WEB-INF/
            └── views/
                ├── login.jsp
                ├── home.jsp
                └── denied.jsp
```

---

Would you like me to generate the full project structure for download as a `.zip` file or Maven project?
