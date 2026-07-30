# Spring Security — Chapter 8: URL Authorization

We now know **authentication**, **roles**, and **authorities**. This chapter puts them together at the HTTP endpoint level.

> **Why → What → How → Where → Internal Flow → Complete Code → Interview Questions → Best Practices**

---

# 1. Why do we need URL Authorization?

Our Employee API has:

```text
GET    /api/employees
GET    /api/employees/101
POST   /api/employees
PUT    /api/employees/101
PATCH  /api/employees/101
DELETE /api/employees/101
```

We may want:

```text
USER
 ├── GET

ADMIN
 ├── GET
 ├── POST
 ├── PUT
 ├── PATCH
 └── DELETE
```

Authentication only tells us:

```text
"Rahul is logged in."
```

Authorization tells us:

```text
"Rahul is allowed to GET,
but not DELETE."
```

---

# 2. What is URL Authorization?

URL authorization means applying authorization rules based on the incoming HTTP request.

For example:

```java
.requestMatchers(HttpMethod.GET, "/api/employees/**")
    .hasAnyRole("USER", "ADMIN")
```

This means:

```text
GET /api/employees/**
        ↓
USER or ADMIN
```

Spring Security's `authorizeHttpRequests` DSL is the standard mechanism for configuring these request authorization rules. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

---

# 3. Our Employee API Security Policy

Let's define it clearly first.

```text
/api/auth/**

    Public


/api/public/**

    Public


GET /api/employees/**

    USER or ADMIN


POST /api/employees/**

    ADMIN


PUT /api/employees/**

    ADMIN


PATCH /api/employees/**

    ADMIN


DELETE /api/employees/**

    ADMIN
```

Everything else:

```text
DENY
```

This is a **deny-by-default** strategy.

---

# 4. Complete `SecurityFilterChain`

```java
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth

                // Public endpoints
                .requestMatchers(
                        "/api/auth/**",
                        "/api/public/**"
                ).permitAll()

                // Read employees
                .requestMatchers(
                        HttpMethod.GET,
                        "/api/employees/**"
                ).hasAnyRole("USER", "ADMIN")

                // Create employees
                .requestMatchers(
                        HttpMethod.POST,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                // Full update
                .requestMatchers(
                        HttpMethod.PUT,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                // Partial update
                .requestMatchers(
                        HttpMethod.PATCH,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                // Delete
                .requestMatchers(
                        HttpMethod.DELETE,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                // Deny anything not explicitly allowed
                .anyRequest()
                .denyAll()
            );

        return http.build();
    }
}
```

Imports:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;
```

---

# 5. Why specify `HttpMethod`?

Consider:

```java
.requestMatchers("/api/employees/**")
    .hasRole("ADMIN")
```

That would mean **all HTTP methods** matching that path require ADMIN.

But maybe GET should be available to USER.

So we make the rules more precise:

```java
.requestMatchers(
    HttpMethod.GET,
    "/api/employees/**"
).hasAnyRole("USER", "ADMIN")
```

and:

```java
.requestMatchers(
    HttpMethod.DELETE,
    "/api/employees/**"
).hasRole("ADMIN")
```

Now authorization depends on both:

```text
HTTP Method
+
URL
```

---

# 6. How does rule matching work?

Suppose:

```http
GET /api/employees/101
```

Spring evaluates the configured authorization rules.

It finds:

```java
.requestMatchers(
    HttpMethod.GET,
    "/api/employees/**"
)
```

Then it checks:

```text
Does user have ROLE_USER or ROLE_ADMIN?
```

If yes:

```text
Authorization ✅
```

The request proceeds.

If no:

```text
Authorization ❌
```

The request is rejected.

---

# 7. Rule Ordering

This is **very important**.

Spring Security evaluates authorization rules in the order they're declared; the first matching rule is used. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

Suppose you write:

```java
.authorizeHttpRequests(auth -> auth
    .anyRequest().permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
)
```

This is wrong.

Why?

The first rule:

```java
.anyRequest().permitAll()
```

matches everything.

The `/admin/**` rule never gets a chance.

Correct:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**")
        .hasRole("ADMIN")
    .anyRequest()
        .denyAll()
)
```

Specific rules should come before broad rules.

---

# 8. `permitAll()`

```java
.requestMatchers("/api/auth/**")
    .permitAll()
```

Means:

```text
No authentication required.
```

Typical examples:

```text
/api/auth/login
/api/auth/register
/api/public/**
```

One important point:

> `permitAll()` doesn't mean "security filters don't run." It means the authorization decision permits that request. Spring Security can still run other security infrastructure around the request. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

---

# 9. `authenticated()`

```java
.requestMatchers("/api/profile/**")
    .authenticated()
```

Means:

```text
USER ✅
ADMIN ✅
Unauthenticated ❌
```

It doesn't care which role the user has.

---

# 10. `hasRole()`

```java
.requestMatchers("/api/admin/**")
    .hasRole("ADMIN")
```

Requires:

```text
ROLE_ADMIN
```

So:

```text
ROLE_ADMIN → ✅
ROLE_USER  → ❌
```

---

# 11. `hasAnyRole()`

```java
.hasAnyRole("USER", "ADMIN")
```

Means:

```text
ROLE_USER OR ROLE_ADMIN
```

So:

```text
USER  → ✅
ADMIN → ✅
MANAGER → ❌
```

unless MANAGER is explicitly included.

---

# 12. `hasAuthority()`

For exact permission:

```java
.hasAuthority("employee:delete")
```

requires:

```text
employee:delete
```

This is useful when authorization needs to be finer-grained than roles.

For example:

```text
HR
    employee:read
    employee:update

Manager
    employee:read

Admin
    employee:read
    employee:create
    employee:update
    employee:delete
```

---

# 13. `hasAnyAuthority()`

```java
.hasAnyAuthority(
    "employee:read",
    "employee:admin"
)
```

At least one must match.

---

# 14. Real Employee Scenarios

### Scenario 1

```text
Rahul
ROLE_USER
```

Request:

```http
GET /api/employees/101
```

Result:

```text
✅ Allowed
```

---

### Scenario 2

```text
Rahul
ROLE_USER
```

Request:

```http
POST /api/employees
```

Result:

```text
❌ 403 Forbidden
```

Authentication succeeded, authorization failed.

---

### Scenario 3

```text
Admin
ROLE_ADMIN
```

Request:

```http
DELETE /api/employees/101
```

Result:

```text
✅ Allowed
```

---

### Scenario 4

No authenticated user:

```http
GET /api/employees/101
```

Result:

```text
❌ Authentication required
```

For an API, the configured authentication entry point normally turns that into a `401 Unauthorized` response.

---

# 15. Complete Request Flow

Let's trace the USER deleting an employee.

```text
DELETE /api/employees/101
        ↓
SecurityFilterChain
        ↓
Authentication
        ↓
Rahul
        ↓
ROLE_USER
        ↓
Authorization Rule
        ↓
DELETE requires ROLE_ADMIN
        ↓
Mismatch
        ↓
Access Denied
        ↓
403
```

The request does **not** reach:

```text
DispatcherServlet
Controller
Service
Repository
```

That is the security benefit.

---

# 16. `requestMatchers()` With Multiple Paths

You can match multiple paths:

```java
.requestMatchers(
        "/css/**",
        "/js/**",
        "/images/**"
).permitAll()
```

or:

```java
.requestMatchers(
        "/api/auth/**",
        "/api/public/**"
).permitAll()
```

---

# 17. Exact Match vs Pattern Match

For example:

```java
.requestMatchers("/api/employees")
```

matches the collection endpoint.

Whereas:

```java
.requestMatchers("/api/employees/**")
```

covers nested paths such as:

```text
/api/employees/101
/api/employees/101/report
```

Choose the pattern deliberately.

---

# 18. URL Authorization vs Method Security

We now have two possible places to enforce security.

## URL-level

```java
.requestMatchers(
    HttpMethod.DELETE,
    "/api/employees/**"
).hasRole("ADMIN")
```

This protects HTTP endpoints.

---

## Method-level

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
}
```

This protects the business method itself.

So:

```text
URL Security
    ↓
HTTP boundary

Method Security
    ↓
Business operation
```

Method security will be our next chapter.

---

# 19. Why use Method Security if URL Security already exists?

Imagine:

```text
Controller A
     ↓
deleteEmployee()

Controller B
     ↓
deleteEmployee()

Scheduler
     ↓
deleteEmployee()
```

URL security protects only HTTP endpoints.

But:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id)
```

protects the method regardless of which application component calls it through the method-security mechanism.

That provides a stronger business-layer boundary.

Spring Security provides method authorization using annotations such as `@PreAuthorize`, enabled with `@EnableMethodSecurity`. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html?utm_source=chatgpt.com))

---

# 20. Common Mistake: Only Frontend Security

Never assume:

```text
Hide Delete Button
    ↓
Security ✅
```

A malicious client can still call:

```http
DELETE /api/employees/101
```

directly.

Authorization must be enforced on the **server**.

---

# 21. Common Mistake: Authentication = Authorization

Suppose:

```text
Rahul logged in ✅
```

That doesn't mean:

```text
Rahul can delete employees ✅
```

You need both:

```text
Authentication
      ↓
Who?

Authorization
      ↓
Allowed to do what?
```

---

# 22. Common Mistake: `ROLE_` Prefix

Correct:

```java
.hasRole("ADMIN")
```

because the authority is:

```text
ROLE_ADMIN
```

Incorrect:

```java
.hasRole("ROLE_ADMIN")
```

when using the normal role prefix.

If you want exact matching:

```java
.hasAuthority("ROLE_ADMIN")
```

---

# 23. Interview Questions

### How does Spring authorize an HTTP request?

> Spring Security evaluates the request against the authorization rules configured through `authorizeHttpRequests`. The first matching rule determines the authorization requirement. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

### What does `anyRequest().denyAll()` achieve?

> It makes unconfigured endpoints inaccessible by default, which is a useful deny-by-default strategy.

### Why should specific matchers come before `anyRequest()`?

> Because the first matching rule is applied. `anyRequest()` matches everything not already matched, so placing it too early can make later rules unreachable. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

### Authentication succeeds but `hasRole("ADMIN")` fails. What happens?

> The request is authenticated but not authorized, so the result is typically `403 Forbidden`.

### Can authorization be based on HTTP method?

> Yes. `requestMatchers(HttpMethod.GET, "/...")` and similar matchers allow method-specific authorization.

### URL security vs method security?

> URL security protects HTTP endpoints; method security protects application methods. They address different boundaries and can be used together.

---

# 24. Best Practices

For your experience level:

```text
✅ Define explicit authorization rules
✅ Put specific rules before broad rules
✅ Consider deny-by-default
✅ Use HttpMethod-specific rules
✅ Keep public endpoints explicitly public
✅ Don't rely on frontend restrictions
✅ Use method security for sensitive business operations
✅ Use roles for coarse access
✅ Use authorities for fine-grained access
```

A strong design for our Employee API is:

```text
Public
  ↓
/api/auth/**

USER + ADMIN
  ↓
GET /api/employees/**

ADMIN
  ↓
POST / PUT / PATCH / DELETE
```

---

# 📍 Where We Are

```text
Spring Security
│
├── ✅ Chapter 1
│   Why Security?
│
├── ✅ Chapter 2
│   SecurityFilterChain
│
├── ✅ Chapter 3
│   Authentication / SecurityContext
│
├── ✅ Chapter 4
│   UserDetails / UserDetailsService
│
├── ✅ Chapter 5
│   PasswordEncoder / BCrypt
│
├── ✅ Chapter 6
│   AuthenticationManager
│   AuthenticationProvider
│   DaoAuthenticationProvider
│
├── ✅ Chapter 7
│   Roles / Authorities
│
├── ✅ Chapter 8
│   URL Authorization
│
└── ⏭️ Chapter 9
      Method Security ⭐⭐⭐⭐⭐
      ↓
      @EnableMethodSecurity
      ↓
      @PreAuthorize
      ↓
      @PostAuthorize
      ↓
      @PreFilter / @PostFilter
      ↓
      Method-level authorization
```

Next is **Method Security**, where we'll protect the actual `EmployeeService` methods and understand why `@PreAuthorize` is another important use of Spring AOP/proxies—connecting directly back to the Spring AOP module you learned earlier.
