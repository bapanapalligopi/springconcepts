# Spring Security — Chapter 3: Authentication Internals

Now we go one level deeper.

We already know:

```text id="9m5q7a"
HTTP Request
    ↓
SecurityFilterChain
    ↓
Authentication
    ↓
Authorization
    ↓
Controller
```

The big question is:

> **What exactly does Spring Security mean by "authenticated user"?**

The answer involves these four concepts:

```text id="h9q8kp"
Authentication
      ↓
SecurityContext
      ↓
SecurityContextHolder
      ↓
Current User
```

Spring Security's authentication architecture defines `Authentication` as both an authentication request/credential representation and, after success, the representation of the currently authenticated user. `SecurityContext` contains the current `Authentication`, and `SecurityContextHolder` stores that context. ([Home][1])

---

# 1. Why do we need `Authentication`?

Suppose Rahul sends:

```http
GET /api/employees
```

Spring has to know:

```text id="fc5r7e"
Who is Rahul?
```

It needs information such as:

```text id="3ny16h"
Username
Authorities
Roles
Principal
Authentication Status
```

Spring represents this information using:

```java id="8kdzp2"
Authentication
```

---

# 2. What is `Authentication`?

`Authentication` is an interface that represents either:

1. credentials supplied as part of an authentication attempt, or
2. the currently authenticated user after authentication succeeds. ([Home][1])

Think of it as the **security identity object**.

A typical authenticated object contains:

```text id="y3t1c2"
Authentication
├── principal
├── credentials
├── authorities
└── authenticated
```

---

# 3. `principal`

## What is Principal?

The principal represents:

> **Who is the user?**

For username/password authentication, the principal is often a `UserDetails` object after authentication succeeds. ([Home][1])

For example:

```text id="k4v79d"
username = rahul
```

or a custom object:

```text id="c2om27"
CustomUser
├── id = 101
├── username = rahul
├── email = rahul@example.com
└── roles = USER
```

---

# 4. `credentials`

This usually represents the proof supplied during authentication.

For username/password authentication:

```text id="9x1e8b"
credentials
    ↓
password
```

An important security behavior is that credentials may be cleared after successful authentication so sensitive password data is not retained unnecessarily. ([Home][1])

So don't think of `Authentication` as a permanent place to store a password.

---

# 5. `authorities`

Authorities answer:

> **What permissions does this user have?**

Example:

```text id="y7kqf0"
ROLE_USER
ROLE_ADMIN
employee:read
employee:delete
```

`GrantedAuthority` represents these permissions. Roles and scopes are common examples. ([Home][1])

For example:

```text id="b4h2q9"
Authentication
   ├── principal → Rahul
   └── authorities
          ├── ROLE_USER
          └── employee:read
```

---

# 6. `isAuthenticated()`

`Authentication` also has:

```java id="2n6l41"
authentication.isAuthenticated()
```

Conceptually:

```text id="zg3t54"
false
  ↓
Authentication attempt

true
  ↓
Authenticated identity
```

Spring Security itself uses the distinction between an authentication request and an authenticated identity during the authentication process. ([Home][1])

---

# 7. What is `SecurityContext`?

Now suppose authentication succeeds.

Where does Spring put the authenticated user?

Inside:

```java id="nqg1j5"
SecurityContext
```

`SecurityContext` contains the current `Authentication`. ([Home][1])

Think:

```text id="g8f8h7"
SecurityContext
      │
      └── Authentication
            ├── principal
            ├── authorities
            └── authenticated
```

---

# 8. What is `SecurityContextHolder`?

Now the next question:

> Where is the `SecurityContext` stored?

Spring Security uses:

```java id="6w6v4d"
SecurityContextHolder
```

The official documentation describes `SecurityContextHolder` as the place where Spring Security stores information about who is authenticated. By default, it uses a `ThreadLocal`-based strategy, making the current security context available throughout the current thread. Spring Security's filter infrastructure clears it after the request. ([Home][1])

So the relationship is:

```text id="a7h9z4"
SecurityContextHolder
        ↓
SecurityContext
        ↓
Authentication
        ↓
Current User
```

---

# 9. Why use `SecurityContextHolder`?

Imagine:

```text id="xj1q2b"
Controller
   ↓
Service
   ↓
Repository
```

Do we want to pass the username manually through every method?

```java id="tqv1t6"
service.updateEmployee(employee, username);
```

Then:

```java id="v8h1t4"
repository.save(employee, username);
```

That becomes messy.

Instead, Spring Security makes the current authentication available through the security context.

You can access it:

```java id="5ux7ue"
Authentication authentication =
        SecurityContextHolder
                .getContext()
                .getAuthentication();
```

Then:

```java id="2m8tt7"
String username =
        authentication.getName();
```

The official documentation shows this exact pattern for accessing the current username, principal, and authorities. ([Home][1])

---

# 10. Real Example

Suppose Rahul is authenticated.

Security context conceptually contains:

```text id="j73hpy"
SecurityContextHolder
      │
      ▼
SecurityContext
      │
      ▼
Authentication
      ├── principal = rahul
      ├── authorities = ROLE_USER
      └── authenticated = true
```

Then your service can do:

```java id="2l8j26"
Authentication authentication =
        SecurityContextHolder
                .getContext()
                .getAuthentication();

String username =
        authentication.getName();
```

---

# 11. Complete Request Flow

Suppose Rahul logs in successfully.

```text id="e1qjy5"
Username + Password
       ↓
Authentication
       ↓
AuthenticationManager
       ↓
AuthenticationProvider
       ↓
Successful Authentication
       ↓
SecurityContext
       ↓
SecurityContextHolder
       ↓
Controller
       ↓
Service
```

The security context then makes the current authentication available while processing the request. Spring Security's filter infrastructure manages the context lifecycle, including clearing it after request processing. ([Home][1])

---

# 12. What is `UserDetails` then?

You may now ask:

> "What's the difference between `Authentication` and `UserDetails`?"

Good question.

### `UserDetails`

Represents information about the **user account**.

Typical data:

```text id="7v4dk0"
username
password
enabled
accountNonExpired
accountNonLocked
credentialsNonExpired
authorities
```

### `Authentication`

Represents the **current authentication state**.

```text id="o6w3j4"
Authentication
    ↓
principal → UserDetails
```

So:

```text id="z7t0pd"
UserDetails
   ↓
User information

Authentication
   ↓
Current security identity/authentication
```

We'll learn `UserDetails` in the next chapter.

---

# 13. `Authentication` vs `SecurityContext`

Don't confuse these.

### Authentication

```text id="wp4s5g"
Who is the user?
What authorities do they have?
```

### SecurityContext

```text id="h5a1q2"
Container holding the current Authentication
```

So:

```text id="k3b6fa"
SecurityContext
    └── Authentication
```

---

# 14. `SecurityContext` vs `SecurityContextHolder`

Again:

### SecurityContext

```text id="jdsx9n"
Stores Authentication
```

### SecurityContextHolder

```text id="r9c3s7"
Provides access to SecurityContext
```

Think of it like:

```text id="5p45u1"
Holder
  ↓
Box
  ↓
Authentication
```

---

# 15. Getting the Current User in a Controller

Instead of directly using `SecurityContextHolder`, Spring MVC can resolve the current principal using:

```java id="d6xy17"
@AuthenticationPrincipal
```

For example:

```java id="rzn1kf"
@GetMapping("/me")
public String currentUser(
        @AuthenticationPrincipal UserDetails user) {

    return user.getUsername();
}
```

Spring Security provides this MVC integration for resolving the current principal. ([Home][1])

This is often cleaner than directly coupling controller code to `SecurityContextHolder`.

---

# 16. Example with Current User

Suppose:

```java id="g7x5z9"
Rahul
ROLE_USER
```

calls:

```http
GET /api/profile
```

Controller:

```java id="8p2jyz"
@GetMapping("/profile")
public String profile(
        @AuthenticationPrincipal
        UserDetails user) {

    return "Hello " +
            user.getUsername();
}
```

Response:

```text
Hello rahul
```

---

# 17. Why is `ThreadLocal` important?

This is an interview-level internal detail.

Spring Security's default `SecurityContextHolder` strategy stores the security context using a `ThreadLocal`. That means code executing on the same request thread can access the current security context without passing it explicitly as a method parameter. The security filter infrastructure clears the context after processing. ([Home][1])

Conceptually:

```text id="5z6j9d"
Request Thread
     │
     ├── SecurityContext
     │
     ├── Controller
     │
     ├── Service
     │
     └── Repository
```

All can access:

```java id="hr8s4x"
SecurityContextHolder.getContext()
```

---

# 18. Important Caution

Because the context is associated with execution threads, asynchronous work requires special consideration.

Don't assume:

```text id="h4l2m5"
Request Thread
    ↓
New Thread
```

automatically has identical security-context behavior.

Spring Security provides dedicated support for propagating security context into certain asynchronous/concurrent execution models, but that is an advanced topic.

For your level:

> **Understand that `SecurityContext` is request/execution-context information, not ordinary global application state.**

---

# 19. Anonymous Authentication

One subtle point:

Spring Security can also place an anonymous authentication into the security context when a user isn't authenticated. This helps application code reliably work with the security context instead of always having to assume `Authentication` is null. ([Home][2])

Conceptually:

```text id="g9q1cx"
No Login
   ↓
AnonymousAuthenticationToken
   ↓
SecurityContext
```

Don't confuse:

```text id="r4qv6a"
Anonymous
```

with:

```text id="k1f1q5"
Authenticated User
```

---

# 20. Interview Questions

### What is `Authentication`?

> It represents authentication information, including the principal, credentials, authorities, and authentication state.

### What is `SecurityContext`?

> It contains the current `Authentication`.

### What is `SecurityContextHolder`?

> It is Spring Security's mechanism for accessing the current `SecurityContext`, with the default servlet strategy using a `ThreadLocal`. ([Home][1])

### What is the difference between `Authentication` and `UserDetails`?

> `UserDetails` represents user account information, while `Authentication` represents the current authentication identity and state; after username/password authentication, the principal is commonly a `UserDetails` instance.

### How do you get the current authenticated user?

Using:

```java id="yqj30u"
SecurityContextHolder
    .getContext()
    .getAuthentication();
```

or, in Spring MVC, often more cleanly:

```java id="5xaz5g"
@AuthenticationPrincipal
```

Spring Security officially supports `@AuthenticationPrincipal` for resolving the principal in MVC. ([Home][1])

---

# 21. Best Practices

Prefer:

```java id="gr21o7"
@AuthenticationPrincipal
```

in controllers when you only need the current principal.

Use `SecurityContextHolder` when you genuinely need lower-level access to the current authentication.

Don't store the current user in a static variable.

Don't put passwords or sensitive credentials into logs.

Remember that security context lifecycle is managed by Spring Security; don't manually clear or replace it casually.

---

# The Core Mental Model

Memorize this:

```text id="d2q79k"
User Successfully Authenticated
             ↓
       Authentication
             ↓
       SecurityContext
             ↓
    SecurityContextHolder
             ↓
    Current Request Code
             ↓
       Controller
             ↓
         Service
```

And inside `Authentication`:

```text id="0snrjt"
Authentication
├── principal
├── credentials
├── authorities
└── authenticated
```

This model will make the next chapters much easier.

---

# 📍 Where We Are

```text id="m88s6a"
Spring Security
│
├── ✅ Chapter 1
│   Why Security?
│   Authentication vs Authorization
│
├── ✅ Chapter 2
│   SecurityFilterChain
│   FilterChainProxy
│   HttpSecurity
│   Authorization Rules
│
├── ✅ Chapter 3
│   Authentication
│   Principal
│   Credentials
│   Authorities
│   SecurityContext
│   SecurityContextHolder
│
└── ⏭️ Chapter 4
      UserDetails ⭐⭐⭐⭐⭐
      UserDetailsService
      InMemoryUserDetailsManager
      Custom UserDetails
```

Next we'll build actual users and learn **how Spring gets user information during authentication**, starting with `UserDetails` and `UserDetailsService`.

[1]: https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html?utm_source=chatgpt.com "Servlet Authentication Architecture :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/7.0/servlet/authentication/anonymous.html?utm_source=chatgpt.com "Anonymous Authentication :: Spring Security"
