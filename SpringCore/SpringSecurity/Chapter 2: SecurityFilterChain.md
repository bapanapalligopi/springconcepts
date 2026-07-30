# Spring Security — Chapter 2: `SecurityFilterChain`

Now we go from **Security theory → actual Spring Security configuration**.

You already know:

```text
HTTP Request
    ↓
Spring Security
    ↓
Authentication
    ↓
Authorization
    ↓
DispatcherServlet
    ↓
Controller
```

The component that controls this web-security processing is the **`SecurityFilterChain`**. Spring Security's servlet architecture uses `FilterChainProxy` to select the appropriate `SecurityFilterChain` for each request and then execute its security filters. ([Home][1])

---

# 1. Why do we need `SecurityFilterChain`?

Imagine our Employee API has:

```text id="wzqk7e"
/api/auth/login
/api/employees
/api/admin/reports
/api/public/departments
```

We need different security rules:

```text id="fry0bf"
/api/auth/** 
    → Public

/api/public/**
    → Public

/api/employees/**
    → Authenticated users

/api/admin/**
    → ADMIN only
```

We need somewhere to define these rules.

That's what our security configuration does through `SecurityFilterChain`.

---

# 2. What is `SecurityFilterChain`?

A `SecurityFilterChain` defines the security filters and configuration that apply to matching HTTP requests. `FilterChainProxy` chooses the applicable chain. ([Home][1])

Think:

```text id="7cc3n4"
HTTP Request
      ↓
FilterChainProxy
      ↓
Which SecurityFilterChain?
      ↓
Security Filters
      ↓
Authentication / Authorization
```

---

# 3. First Security Configuration

With modern Spring Security:

```java id="sec1"
package com.practice.employeeapi.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

This means:

> **Every HTTP request must be authenticated.**

Spring Security's current request-authorization API uses `authorizeHttpRequests`, and the documentation recommends declaring authorization rules when using `HttpSecurity`. ([Home][2])

---

# 4. Let's understand each line

## `@Configuration`

```java id="b3xj1e"
@Configuration
```

This tells Spring:

> This class contains bean configuration.

---

## `@Bean`

```java id="6fs4s9"
@Bean
SecurityFilterChain securityFilterChain(...)
```

Spring creates the `SecurityFilterChain` object and manages it as a bean.

---

## `HttpSecurity`

```java id="8psd67"
HttpSecurity http
```

`HttpSecurity` is the configuration object through which we configure web security.

Think:

```text id="6s4bni"
HttpSecurity
    ↓
Configure security
```

---

## `authorizeHttpRequests`

```java id="h2v5ku"
.authorizeHttpRequests(...)
```

This configures **authorization rules for HTTP requests**. ([Home][2])

---

## `anyRequest()`

```java id="4rsbuz"
.anyRequest()
```

Means:

> Any request that hasn't already been matched by an earlier rule.

---

## `authenticated()`

```java id="xohv0h"
.authenticated()
```

Means:

> The request requires an authenticated user.

---

# 5. Important: Authorization Rules Are Evaluated in Order

Suppose:

```java id="1v3t97"
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/public/**").permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);
```

Spring evaluates the rules in the order they are declared. The first matching authorization rule is used. ([Home][2])

So:

```text id="n26cjn"
GET /public/about

       ↓

/public/** matches

       ↓

permitAll()
```

It doesn't continue looking for another rule.

This is extremely important.

---

# 6. `permitAll()`

```java id="4eaz6e"
.requestMatchers("/public/**")
    .permitAll()
```

Means:

> Anyone can access these endpoints; no authorization is required. ([Home][2])

Example:

```text id="j1v3s5"
/public/about
/public/departments
/api/auth/login
```

could be public.

---

# 7. `authenticated()`

```java id="00jqq1"
.requestMatchers("/employees/**")
    .authenticated()
```

Means:

> The user must be authenticated.

The user doesn't necessarily need a particular role.

```text id="g3p6y9"
USER       → Allowed
ADMIN      → Allowed
Unauthenticated → Rejected
```

---

# 8. `hasRole()`

Suppose only admins can delete employees.

```java id="o8q0kl"
.requestMatchers("/api/admin/**")
    .hasRole("ADMIN")
```

This means:

> The authenticated user needs the `ADMIN` role.

A key detail: `hasRole("ADMIN")` normally checks for the authority `ROLE_ADMIN`; you don't include the `ROLE_` prefix in the `hasRole` argument. ([Home][2])

So:

```java id="if3xj6"
.hasRole("ADMIN")
```

corresponds conceptually to:

```text id="2o7kks"
ROLE_ADMIN
```

---

# 9. `hasAuthority()`

This is slightly different:

```java id="1jmz3n"
.requestMatchers("/reports/**")
    .hasAuthority("report:read")
```

Here Spring checks for the exact `GrantedAuthority` value:

```text id="zfjw7j"
report:read
```

`hasAuthority` matches the authority value directly, while `hasRole` is a convenience that applies the role prefix. ([Home][2])

---

# 10. Role vs Authority

This distinction will become very important later.

### Role

```text id="q4q4gs"
ROLE_ADMIN
ROLE_USER
```

Usually represents a broad user category.

### Authority

```text id="0v9y2h"
employee:read
employee:create
employee:update
employee:delete
```

Usually represents a more specific permission.

Think:

```text id="doowhv"
Role
 ↓
Broad responsibility

Authority
 ↓
Specific permission
```

We'll dedicate a complete chapter to this.

---

# 11. Our Employee API Security Rules

Let's design:

```text id="h9ag0q"
/api/auth/** 
    → public

/api/public/**
    → public

GET /api/employees/**
    → authenticated

POST /api/employees/**
    → ADMIN

PUT /api/employees/**
    → ADMIN

PATCH /api/employees/**
    → ADMIN

DELETE /api/employees/**
    → ADMIN
```

Modern Spring Security configuration:

```java id="sec2"
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth

                .requestMatchers(
                        "/api/auth/**",
                        "/api/public/**"
                ).permitAll()

                .requestMatchers(
                        org.springframework.http.HttpMethod.GET,
                        "/api/employees/**"
                ).authenticated()

                .requestMatchers(
                        org.springframework.http.HttpMethod.POST,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                .requestMatchers(
                        org.springframework.http.HttpMethod.PUT,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                .requestMatchers(
                        org.springframework.http.HttpMethod.PATCH,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                .requestMatchers(
                        org.springframework.http.HttpMethod.DELETE,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                .anyRequest().denyAll()
            );

        return http.build();
    }
}
```

---

# 12. Why `anyRequest().denyAll()`?

This is a useful defensive strategy.

Instead of:

```java id="2igx5v"
.anyRequest().authenticated()
```

we can say:

```java id="4h7y3j"
.anyRequest().denyAll()
```

Meaning:

> Anything we forgot to explicitly authorize is rejected.

Spring Security's documentation specifically shows `anyRequest().denyAll()` as a useful strategy when you don't want an endpoint accidentally exposed because you forgot to add an authorization rule. ([Home][2])

For a security-sensitive application, this **deny-by-default** approach is often a good choice.

---

# 13. What Happens Internally?

Request:

```http id="rq4w0a"
GET /api/employees/101
```

Flow:

```text id="q8zv4d"
Client
   ↓
Servlet Container
   ↓
FilterChainProxy
   ↓
Matching SecurityFilterChain
   ↓
Security Filters
   ↓
Authorization Decision
   ↓
Is user authenticated?
   ↓
YES
   ↓
DispatcherServlet
   ↓
EmployeeController
```

Spring's servlet architecture places security filters before normal servlet processing, and `FilterChainProxy` selects the appropriate chain. ([Home][1])

---

# 14. What if User Isn't Authenticated?

Request:

```http id="n4w4u4"
GET /api/employees/101
```

but there is no valid authentication.

Conceptually:

```text id="4y7yrc"
Request
 ↓
SecurityFilterChain
 ↓
Authentication Required
 ↓
Not Authenticated
 ↓
Request Rejected
```

The controller is not reached.

For an HTTP API, this will ultimately become an authentication failure response, commonly `401 Unauthorized`, depending on the configured authentication entry point.

---

# 15. What if User is Authenticated but Isn't Admin?

Suppose:

```text id="p6h7th"
Rahul
ROLE_USER
```

calls:

```http id="n27k8v"
DELETE /api/employees/101
```

The rule says:

```java id="5kgh4x"
.hasRole("ADMIN")
```

But Rahul only has:

```text id="v4e7rb"
ROLE_USER
```

So:

```text id="1dj7j7"
Authenticated ✅
       ↓
Authorization ❌
       ↓
403 Forbidden
```

---

# 16. `requestMatchers()` vs `securityMatcher()`

This is slightly more advanced, but important.

### `requestMatchers()`

Used inside:

```java id="q7zg84"
authorizeHttpRequests(...)
```

to define **authorization rules** for specific requests.

Example:

```java id="5azp1x"
.requestMatchers("/admin/**").hasRole("ADMIN")
```

### `securityMatcher()`

Used to determine **which entire `SecurityFilterChain` applies** to a request. ([Home][2])

For example:

```java id="kzj8b7"
http.securityMatcher("/api/**");
```

means:

> This particular security chain is intended for `/api/**`.

So:

```text id="xvblio"
securityMatcher
   ↓
Which SecurityFilterChain?

requestMatchers
   ↓
Which authorization rule inside that chain?
```

This distinction becomes useful when an application has **multiple security filter chains**.

For your 1.5-year level, understand it conceptually. We don't need multiple chains yet.

---

# 17. Multiple SecurityFilterChains

Spring Security can have more than one chain.

For example:

```text id="vdfh4n"
SecurityFilterChain #1
/api/**
    ↓
API Security

SecurityFilterChain #2
/admin-ui/**
    ↓
Admin UI Security
```

`FilterChainProxy` checks the chains in order and uses the first matching chain. ([Home][1])

This is an advanced capability. You only need the concept right now.

---

# 18. Why don't we manually create the filters?

You generally should not write:

```java
new AuthenticationFilter()
new AuthorizationFilter()
new SomethingElse()
```

in your application.

Spring Security builds and manages the filter infrastructure from your configuration.

Your job is primarily:

```text id="0qc6to"
Configure
     ↓
Spring builds security infrastructure
```

---

# 19. Important SecurityFilterChain Concept

A `SecurityFilterChain` isn't itself "the filter."

It's a definition of:

```text id="m6y8ip"
Which request?
     ↓
Which Spring Security filters?
     ↓
What security configuration?
```

Spring's `FilterChainProxy` uses it to determine which security filters should run. ([Home][1])

---

# 20. First Practical Configuration

For now, start simple.

```java id="sec3"
@Configuration
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth

                .requestMatchers("/api/public/**")
                    .permitAll()

                .requestMatchers("/api/employees/**")
                    .authenticated()

                .anyRequest()
                    .denyAll()
            );

        return http.build();
    }
}
```

This teaches three fundamental concepts:

```text id="9p4tkw"
permitAll()
authenticated()
denyAll()
```

---

# 21. What hasn't been configured yet?

You might be wondering:

> "If everything requires authentication, where are the users?"

Good question.

We haven't configured the authentication mechanism yet.

That's the next stage.

For example, later we can add:

```text id="udq4zj"
UserDetailsService
PasswordEncoder
AuthenticationProvider
```

and then:

```text id="6q1d3q"
Username
Password
   ↓
Authentication
```

Eventually, JWT:

```text id="j1s3y3"
JWT
   ↓
Authentication
   ↓
Authorization
```

---

# 22. Interview Questions

### What is `SecurityFilterChain`?

> It defines the Spring Security filters and security processing that apply to matching HTTP requests.

### What is `FilterChainProxy`?

> It is Spring Security's central servlet-filter infrastructure that selects the appropriate `SecurityFilterChain` and invokes its filters. ([Home][1])

### What does `permitAll()` do?

> Allows a request without requiring authorization.

### What does `authenticated()` do?

> Requires the request to have an authenticated user.

### What does `denyAll()` do?

> Rejects the request regardless of authentication.

### Difference between `hasRole()` and `hasAuthority()`?

> `hasRole("ADMIN")` is a role-oriented shortcut that normally checks for `ROLE_ADMIN`; `hasAuthority("employee:delete")` checks the exact authority string. ([Home][2])

### Difference between `requestMatchers()` and `securityMatcher()`?

> `securityMatcher()` selects which entire security filter chain applies; `requestMatchers()` defines authorization rules within a chain. ([Home][2])

---

# 23. Best Practices

For your level, remember these:

```text id="5x9x7n"
✅ Keep authorization rules explicit
✅ Prefer deny-by-default for sensitive APIs
✅ Put specific rules before broad rules
✅ Don't implement authentication inside controllers
✅ Don't expose security checks through duplicated controller code
✅ Understand the difference between role and authority
```

And remember:

> **Security should reject unauthorized requests before they reach your business logic.**

---

# 📍 Where We Are

```text id="z3f5jv"
Spring Security
│
├── ✅ Chapter 1
│     Why Security?
│     Authentication vs Authorization
│
├── ✅ Chapter 2
│     SecurityFilterChain
│     FilterChainProxy
│     HttpSecurity
│     requestMatchers()
│     permitAll()
│     authenticated()
│     hasRole()
│     hasAuthority()
│     denyAll()
│
└── ⏭️ Chapter 3
      Authentication
      ↓
      Authentication object
      ↓
      Principal
      ↓
      SecurityContext
      ↓
      SecurityContextHolder
```

Next we'll learn **Authentication in depth**: what an `Authentication` object actually contains, how the authenticated user is represented, and how Spring makes the current user available to the rest of the application.

[1]: https://docs.spring.io/spring-security/reference/7.0/servlet/architecture.html?utm_source=chatgpt.com "Architecture :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com "Authorize HttpServletRequests :: Spring Security"
