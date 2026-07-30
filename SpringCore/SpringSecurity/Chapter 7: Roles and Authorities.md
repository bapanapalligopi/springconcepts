# Spring Security — Chapter 7: Roles and Authorities

Now we learn **Roles vs Authorities**, one of the most common Spring Security interview topics.

We already have the authentication flow:

```text id="8z4nq3"
Username + Password
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService
        ↓
PasswordEncoder
        ↓
Authentication
        ↓
SecurityContext
```

Now the question is:

> **After the user is authenticated, what is the user actually allowed to do?**

That's authorization.

---

# 1. Why do we need Roles and Authorities?

Suppose our Employee API has:

```text id="p7x2k9"
GET    /api/employees
POST   /api/employees
PUT    /api/employees/101
DELETE /api/employees/101
```

We don't want every authenticated user to have every permission.

Maybe:

```text id="m4q8c2"
USER
 ├── Read employees
 └── View employee details

ADMIN
 ├── Read
 ├── Create
 ├── Update
 └── Delete
```

We need a way to represent these permissions.

Spring Security uses:

```text id="a8v3n6"
GrantedAuthority
```

and roles are a common convention built on top of authorities.

---

# 2. What is a `GrantedAuthority`?

A `GrantedAuthority` represents a permission or privilege associated with an authenticated user.

Example:

```text id="d6k9r1"
ROLE_USER
ROLE_ADMIN

employee:read
employee:create
employee:update
employee:delete
```

Spring Security's authorization architecture uses `GrantedAuthority` values when making authorization decisions. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html?utm_source=chatgpt.com))

---

# 3. What is a Role?

A role is generally a **coarse-grained grouping** of permissions.

Examples:

```text id="r7c3m5"
ROLE_USER
ROLE_ADMIN
ROLE_MANAGER
```

Think:

```text id="n2v8q4"
Role
 ↓
Broad category of access
```

For example:

```text id="y6p4j1"
ROLE_ADMIN
```

might allow:

```text
employee:read
employee:create
employee:update
employee:delete
```

---

# 4. What is an Authority?

An authority is usually more specific.

For example:

```text id="k3m8q5"
employee:read
employee:create
employee:update
employee:delete
```

Think:

```text id="w9c2p7"
Authority
 ↓
Specific permission
```

---

# 5. Simple Comparison

```text id="j5r8m2"
ROLE_ADMIN
     ↓
Broad permission group

employee:delete
     ↓
Specific permission
```

You can think:

```text id="x4p7n1"
Role
  ↓
"What kind of user?"

Authority
  ↓
"What exactly can they do?"
```

This isn't an absolute technical rule, but it's a useful architectural way to design permissions.

---

# 6. The Important `ROLE_` Prefix

This is where beginners frequently get confused.

Suppose we write:

```java id="h8k3p5"
.roles("ADMIN")
```

Spring creates an authority conceptually equivalent to:

```text id="c6m9r2"
ROLE_ADMIN
```

Then:

```java id="q4v7n1"
.hasRole("ADMIN")
```

looks for:

```text id="a9p3k6"
ROLE_ADMIN
```

The `ROLE_` prefix is automatically applied by the role-based authorization API. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

So:

```java id="m7x2c8"
.hasRole("ADMIN")
```

not:

```java id="f5n9q4"
.hasRole("ROLE_ADMIN") // ❌
```

---

# 7. `hasRole()`

Example:

```java id="p8c3m6"
.requestMatchers("/admin/**")
    .hasRole("ADMIN")
```

Meaning:

> The current user must have the `ROLE_ADMIN` authority.

Spring's `hasRole` is a convenience around role-prefixed authorities. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

---

# 8. `hasAuthority()`

For exact permission matching:

```java id="x6r2v9"
.requestMatchers(HttpMethod.DELETE,
        "/api/employees/**")
    .hasAuthority("employee:delete")
```

Now Spring checks for exactly:

```text id="q5m8c1"
employee:delete
```

No `ROLE_` prefix is added by `hasAuthority`. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

---

# 9. Role Example

Create a user:

```java id="a4v7k2"
UserDetails admin =
        User.withUsername("admin")
                .password(
                    passwordEncoder.encode("admin123")
                )
                .roles("ADMIN")
                .build();
```

The user's authorities include:

```text id="r8m3q6"
ROLE_ADMIN
```

Therefore:

```java id="k5p9c2"
.hasRole("ADMIN")
```

matches.

---

# 10. Authority Example

Create a user with explicit authorities:

```java id="j2x6m8"
UserDetails user =
        User.withUsername("rahul")
                .password(
                    passwordEncoder.encode("password123")
                )
                .authorities(
                    "employee:read",
                    "employee:update"
                )
                .build();
```

Rahul has:

```text id="p9q4v1"
employee:read
employee:update
```

but not:

```text id="s7k2c8"
employee:delete
```

Therefore:

```java id="w5m9r3"
.hasAuthority("employee:read")
```

works.

But:

```java id="g8p2x6"
.hasAuthority("employee:delete")
```

fails.

---

# 11. Real Employee API

Let's design our authorization rules.

### USER

```text id="k4v8q1"
GET /api/employees
GET /api/employees/101
```

### ADMIN

```text id="r9m3c7"
GET
POST
PUT
PATCH
DELETE
```

One way is role-based authorization:

```java id="a6p2n8"
http.authorizeHttpRequests(auth -> auth

    .requestMatchers(
        HttpMethod.GET,
        "/api/employees/**"
    ).hasAnyRole("USER", "ADMIN")

    .requestMatchers(
        HttpMethod.POST,
        "/api/employees/**"
    ).hasRole("ADMIN")

    .requestMatchers(
        HttpMethod.PUT,
        "/api/employees/**"
    ).hasRole("ADMIN")

    .requestMatchers(
        HttpMethod.PATCH,
        "/api/employees/**"
    ).hasRole("ADMIN")

    .requestMatchers(
        HttpMethod.DELETE,
        "/api/employees/**"
    ).hasRole("ADMIN")

    .anyRequest().denyAll()
);
```

---

# 12. `hasAnyRole()`

Sometimes multiple roles should be allowed:

```java id="m7c4x9"
.hasAnyRole("USER", "ADMIN")
```

This means:

```text id="q3p8n5"
ROLE_USER OR ROLE_ADMIN
```

Either is sufficient.

---

# 13. `hasAnyAuthority()`

Similarly:

```java id="v5r2m8"
.hasAnyAuthority(
    "employee:read",
    "employee:admin"
)
```

requires at least one matching authority.

---

# 14. Role-Based vs Permission-Based Design

There are two common approaches.

## Approach 1: Roles

```text id="j8q3v6"
USER
ADMIN
MANAGER
```

Rules:

```text id="c4p7n1"
ADMIN → Everything
USER  → Read
```

Simple.

Good for smaller applications.

---

## Approach 2: Authorities

```text id="b9m2x5"
employee:read
employee:create
employee:update
employee:delete
```

Rules become more granular.

For example:

```text id="n6q4r8"
USER
 ├── employee:read

HR
 ├── employee:read
 └── employee:update

ADMIN
 ├── employee:read
 ├── employee:create
 ├── employee:update
 └── employee:delete
```

This provides finer control.

---

# 15. Which Should You Use?

For your **1.5-year experience level**:

```text id="d3p7m9"
Simple application
    ↓
Roles

More granular enterprise permissions
    ↓
Authorities
```

Many real applications use **both**.

For example:

```text id="j8m4q2"
Role
 ↓
ADMIN

Authorities
 ↓
employee:read
employee:create
employee:update
employee:delete
```

The role is coarse-grained while authorities provide finer control.

---

# 16. Where do Authorities Come From?

They usually originate from `UserDetails`.

For example:

```java id="z5q9m3"
return User.withUsername("rahul")
        .password(encodedPassword)
        .roles("USER")
        .build();
```

or:

```java id="t7c2p8"
return User.withUsername("rahul")
        .password(encodedPassword)
        .authorities(
            "employee:read",
            "employee:update"
        )
        .build();
```

Then they are carried inside:

```text id="v4m8x1"
Authentication
      ↓
GrantedAuthorities
```

---

# 17. Complete Authorization Flow

Suppose Rahul is:

```text id="h2r6n9"
ROLE_USER
```

and requests:

```http id="x7p3m5"
DELETE /api/employees/101
```

Flow:

```text id="q9c4v7"
Request
  ↓
SecurityFilterChain
  ↓
Authentication
  ↓
User = Rahul
  ↓
Authorities = ROLE_USER
  ↓
Authorization Rule
  ↓
Required = ROLE_ADMIN
  ↓
Mismatch ❌
  ↓
403 Forbidden
```

---

# 18. What about an Admin?

Admin:

```text id="m8p2x4"
ROLE_ADMIN
```

Request:

```http id="n6c9r3"
DELETE /api/employees/101
```

Flow:

```text id="b5v7k2"
Authorities
   ↓
ROLE_ADMIN

Required
   ↓
ROLE_ADMIN

Match ✅
   ↓
Controller
```

---

# 19. `hasRole()` vs `hasAuthority()`

Memorize this table:

| `hasRole()`         | `hasAuthority()`             |
| ------------------- | ---------------------------- |
| Role-oriented       | Exact authority              |
| Adds `ROLE_` prefix | No prefix added              |
| `hasRole("ADMIN")`  | `hasAuthority("ROLE_ADMIN")` |
| Broad access        | Specific permission          |

The last row is a design guideline rather than a technical rule, but it is useful when deciding how to model authorization.

---

# 20. Important Interview Trap

Suppose:

```java id="w3p8q6"
.roles("ADMIN")
```

and then:

```java id="v6m2k9"
.hasAuthority("ADMIN")
```

Will it match?

**No.**

The role creates:

```text id="r7c4n1"
ROLE_ADMIN
```

not:

```text id="a5x8m2"
ADMIN
```

Therefore:

```java id="z9q3v6"
.hasRole("ADMIN")
```

matches the role.

Or:

```java id="p4m7c1"
.hasAuthority("ROLE_ADMIN")
```

matches the actual authority value.

---

# 21. Another Interview Trap

What about:

```java id="q8n2r4"
.hasRole("ROLE_ADMIN")
```

Don't do that.

Because Spring's role-based matcher adds the prefix.

Use:

```java id="m5x9c2"
.hasRole("ADMIN")
```

---

# 22. Method Security Preview

So far we've secured URLs:

```java id="r7p3m6"
.requestMatchers("/admin/**")
    .hasRole("ADMIN")
```

But sometimes you want security at the **method level**:

```java id="k4m8x2"
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
}
```

That's called **Method Security**.

We'll learn it in the next chapter.

The official Spring Security documentation supports method-level authorization through annotations such as `@PreAuthorize`, enabled using `@EnableMethodSecurity`. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html?utm_source=chatgpt.com))

---

# 23. Why Method Security?

Suppose five different controllers call:

```java id="p6c2v8"
employeeService.deleteEmployee(id);
```

Instead of protecting every URL individually, you can protect the actual business method:

```java id="q9m5x3"
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
}
```

Now the rule follows the business operation.

This is particularly useful as applications become larger.

---

# 24. Where should authorization live?

A strong application often uses layers of protection:

```text id="a7v3n9"
HTTP Layer
    ↓
SecurityFilterChain
    ↓
URL-level authorization

Business Layer
    ↓
Method Security
    ↓
Fine-grained authorization
```

We don't need both for every method, but understanding the distinction is important.

---

# 25. Interview Questions

### What is a `GrantedAuthority`?

> It represents a permission or privilege associated with an authenticated user.

### What is a role?

> A role is a conventional, coarse-grained authority such as `ROLE_USER` or `ROLE_ADMIN`.

### Difference between `hasRole` and `hasAuthority`?

> `hasRole("ADMIN")` checks for the prefixed authority `ROLE_ADMIN`, while `hasAuthority("employee:delete")` checks for that exact authority. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com))

### What does `.roles("ADMIN")` create?

```text id="d4m7q2"
ROLE_ADMIN
```

### What does `.authorities("employee:delete")` create?

```text id="r8n3v5"
employee:delete
```

### Can one user have multiple authorities?

Yes.

```java id="w6p2c9"
.authorities(
    "employee:read",
    "employee:update",
    "employee:delete"
)
```

---

# 26. Best Practices

```text id="j2m8v4"
✅ Use roles for broad categories
✅ Use authorities for granular permissions
✅ Don't put ROLE_ inside hasRole()
✅ Keep permission names consistent
✅ Use method security for sensitive business operations
✅ Don't rely only on frontend authorization
```

A good naming convention for authorities is something like:

```text id="q7v3n1"
employee:read
employee:create
employee:update
employee:delete
```

This is easier to understand than arbitrary values such as:

```text id="m5c9x2"
PERM_1
ACCESS_7
ADMIN_DELETE_X
```

---

# 📍 Where We Are

```text id="v9m3q5"
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
│   Roles
│   Authorities
│   GrantedAuthority
│   hasRole()
│   hasAuthority()
│
└── ⏭️ Chapter 8
      URL Authorization
      ↓
      requestMatchers()
      ↓
      permitAll()
      ↓
      authenticated()
      ↓
      hasRole()
      ↓
      hasAuthority()
      ↓
      Complete Employee API Security
```

Next we'll turn everything we've learned into a **fully configured authorization policy for the Employee API**, including public endpoints, USER endpoints, ADMIN endpoints, HTTP-method-specific rules, rule ordering, and common mistakes.
