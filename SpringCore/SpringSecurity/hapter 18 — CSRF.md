# Spring Security — Chapter 18: CSRF

Now we cover **CSRF**, one of the most misunderstood Spring Security topics.

The most important thing to understand is:

> **CSRF is primarily about automatically attached credentials, especially cookies.**

It is not simply "an attack against POST requests", and it is not automatically irrelevant just because you're using JWT.

Spring Security enables CSRF protection by default for servlet applications, and by default protects HTTP methods other than `GET`, `HEAD`, `TRACE`, and `OPTIONS`. ([Home][1])

We'll use:

> **Why → What → How → Where → Internal Flow → Code → JWT Case → Interview Questions → Best Practices**

---

# 1. Why do we need CSRF protection?

Imagine a user logs into:

```text id="n7h3q2"
https://bank.example.com
```

The browser receives a session cookie:

```http id="3tqk8m"
Cookie: JSESSIONID=abc123
```

Later, the user visits a malicious site:

```text id="x8m5p1"
https://evil.example
```

That site tries to submit:

```http id="g4r2w9"
POST https://bank.example.com/transfer
```

The browser may automatically attach the bank's cookie.

So the bank sees:

```text id="z1c6v3"
Authenticated session ✅
```

even though the user never intentionally submitted that transfer.

That's the core CSRF problem.

---

# 2. What is CSRF?

CSRF means:

> **Cross-Site Request Forgery**

Conceptually:

```text id="q9m2x4"
Victim logs into bank
        ↓
Browser stores authentication cookie
        ↓
Victim visits malicious website
        ↓
Malicious site sends request to bank
        ↓
Browser automatically sends cookie
        ↓
Bank thinks request is from victim
```

The attacker is trying to **forge an authenticated request** from another site.

Spring Security's CSRF support specifically protects against this class of attack. ([Home][1])

---

# 3. Authentication Cookie Is the Key

Why does CSRF work?

Because browsers automatically attach certain credentials such as cookies.

For example:

```text id="4u6n7p"
Bank
 ↓
JSESSIONID cookie

Malicious site
 ↓
POST /transfer
 ↓
Browser automatically sends JSESSIONID
```

The malicious site doesn't need to know the cookie value.

That's why CSRF is particularly relevant to **cookie-based authentication**.

---

# 4. How does CSRF protection work?

The standard approach is:

```text id="k2p8m1"
Server generates CSRF token
        ↓
Client receives token
        ↓
Client sends token with unsafe request
        ↓
Server compares token
        ↓
Valid?
```

So the attacker might be able to cause:

```http id="p4z9k6"
POST /transfer
Cookie: JSESSIONID=abc123
```

but doesn't know the unpredictable CSRF token required by the application.

The request gets rejected.

Spring Security's `CsrfFilter` loads the expected token, resolves the token provided by the client, and compares them; an invalid or missing token results in an access-denied path. ([Home][1])

---

# 5. CSRF Token Example

Imagine the server generates:

```text id="h5q3x8"
CSRF Token:
8f7a9c...
```

The browser submits:

```http id="m4w6c2"
POST /api/employees
X-CSRF-TOKEN: 8f7a9c...
Cookie: JSESSIONID=abc123
```

Server checks:

```text id="r9p2v7"
Expected token
      =
Submitted token
```

If they match:

```text id="u1k8d4"
CSRF ✅
```

If not:

```text id="c5m3q9"
CSRF ❌
```

---

# 6. Which HTTP Methods Are Protected?

By default, Spring Security's CSRF protection covers methods other than:

```text id="y7n2m5"
GET
HEAD
TRACE
OPTIONS
```

So it normally focuses on state-changing methods such as:

```text id="g4p8q1"
POST
PUT
PATCH
DELETE
```

The exact default matcher is documented by Spring Security. ([Home][1])

The reasoning is:

```text id="s8c2m6"
GET
 ↓
Should normally be read-only

POST/PUT/PATCH/DELETE
 ↓
Can change server state
```

---

# 7. Why Don't We Need CSRF for Normal GET?

A properly designed REST API should make GET safe/read-only.

For example:

```http id="e3r8x5"
GET /employees/101
```

should not:

```text
Delete employee
Transfer money
Change password
```

If a GET changes state, that's a bad API design.

---

# 8. CSRF vs CORS

We just learned CORS.

Do not confuse them.

### CORS

```text id="a1v6p8"
Which browser origins may call my API?
```

### CSRF

```text id="d9c4m2"
Can another site trick the browser
into sending an authenticated request?
```

So:

```text id="x7m3q1"
CORS
  ↓
Cross-origin browser policy

CSRF
  ↓
Forged authenticated request protection
```

Different problems. Different protections.

---

# 9. Does JWT Automatically Remove CSRF?

**No.**

This is the important correction.

It depends on **how the JWT is transported**.

### Case 1: JWT in `Authorization` header

Client explicitly sends:

```http id="w7n4c2"
Authorization: Bearer eyJ...
```

A malicious site generally cannot make the browser automatically attach an arbitrary Authorization header with your victim's token.

So a typical stateless API that authenticates using a bearer token in the `Authorization` header often doesn't need CSRF protection for that authentication mechanism.

### Case 2: JWT stored in a cookie

Suppose:

```http id="f3q8v1"
Cookie: access_token=eyJ...
```

The browser may automatically attach that cookie.

Now the authentication mechanism is again using browser-managed credentials, so CSRF becomes relevant.

This is why:

> **"We use JWT, therefore disable CSRF" is an incomplete statement.**

The storage and transport mechanism matter.

---

# 10. JWT in Authorization Header

Our current design is:

```http id="c7m2x9"
Authorization: Bearer <JWT>
```

Flow:

```text id="r4n8p3"
React
  ↓
Authorization header
  ↓
Spring Security
  ↓
JWT validation
```

The browser doesn't automatically add that bearer token to arbitrary cross-origin requests the way it automatically sends cookies.

So a typical bearer-header REST API can often disable CSRF protection because its authentication is not based on browser-automatically-attached credentials.

This is an architectural decision, not a universal "JWT rule."

---

# 11. Why Did We Disable CSRF Earlier?

We used:

```java id="h7k3m9"
http.csrf(csrf -> csrf.disable());
```

for our stateless bearer-token REST example.

The reasoning was:

```text id="u8c4v2"
Authentication
     ↓
Authorization: Bearer JWT
     ↓
Explicit client header
```

rather than:

```text id="n5q9r1"
Authentication
     ↓
Browser automatically sends cookie
```

So for our specific REST architecture, disabling CSRF can be appropriate.

But remember:

> **Do not copy that line into a cookie-based browser application without thinking.**

---

# 12. Stateful Session Application

Imagine a traditional MVC application:

```text id="v2m6p8"
Browser
   ↓
Login Form
   ↓
JSESSIONID
   ↓
Server Session
```

Now CSRF is important.

Spring Security's default servlet configuration stores the expected CSRF token in the HTTP session using `HttpSessionCsrfTokenRepository`. ([Home][1])

The application might render:

```html id="p9f1q2"
<input
    type="hidden"
    name="_csrf"
    value="..."
/>
```

Spring MVC form integrations can automatically include the CSRF token in forms. ([Home][1])

---

# 13. JavaScript Application with CSRF

Spring Security also supports storing the expected CSRF token in a cookie through:

```java id="c6w8x2"
CookieCsrfTokenRepository
```

The documentation describes the default cookie/header naming convention:

```text id="r3n7m1"
Cookie:
XSRF-TOKEN

Header:
X-XSRF-TOKEN
```

This allows JavaScript applications to read the token and send it back in a request header. ([Home][1])

Conceptually:

```text id="g4p9v2"
Server
  ↓
XSRF-TOKEN cookie

Frontend reads token
  ↓
X-XSRF-TOKEN header
  ↓
POST/PUT/PATCH/DELETE
```

---

# 14. CSRF Internal Flow

Let's trace a protected state-changing request.

```text id="u5q8c1"
POST /api/employees
        ↓
CsrfFilter
        ↓
Does this request require CSRF?
        ↓
YES
        ↓
Load expected token
        ↓
Read submitted token
        ↓
Compare
   ┌────┴────┐
  VALID     INVALID
    ↓          ↓
Continue    AccessDenied
    ↓
Security
```

Spring Security's `CsrfFilter` implements this processing. ([Home][1])

---

# 15. What Happens When the CSRF Token Is Invalid?

Suppose:

```http id="x2r8m5"
POST /api/employees
Cookie: JSESSIONID=abc123
X-CSRF-TOKEN: wrong
```

Then:

```text id="q7c4n9"
CsrfFilter
   ↓
Token mismatch
   ↓
AccessDeniedException
   ↓
AccessDeniedHandler
```

Spring Security documents that invalid or missing CSRF tokens result in an access-denied handling path. ([Home][1])

---

# 16. Why Does CSRF Use `AccessDeniedHandler`?

This is a useful connection to our previous chapter.

We learned:

```text id="m8p3q7"
AuthenticationEntryPoint
   ↓
401

AccessDeniedHandler
   ↓
403
```

A CSRF violation generally goes through an access-denied path.

So:

```text id="j5n9c2"
Invalid CSRF token
      ↓
AccessDeniedException
      ↓
AccessDeniedHandler
```

---

# 17. CSRF with HTML Forms

Traditional Spring MVC:

```html id="t5r8w2"
<form method="post" action="/employees">
```

The CSRF token needs to be included.

Spring-integrated form technologies such as Spring's form tags and Thymeleaf can automatically include the CSRF token in unsafe forms. ([Home][1])

So conceptually:

```text id="q9m4p7"
HTML Form
   +
CSRF Token
   ↓
Spring Security
```

---

# 18. CSRF with REST + Cookie Authentication

Suppose your REST API uses:

```text id="f8c2x5"
Cookie: access_token=...
```

Now your browser automatically attaches the cookie.

If you don't use a CSRF defense:

```text id="y6p4m8"
evil.example
   ↓
POST api.example.com/transfer
   ↓
Browser automatically sends cookie
   ↓
API sees authenticated user
```

This is still a CSRF concern.

So:

> **Statelessness alone does not solve CSRF.**

Spring Security's documentation also discusses CSRF considerations for stateless browser applications. ([Home][2])

---

# 19. Stateless vs CSRF

This is a subtle interview question.

Suppose:

```java id="r5m7c3"
SessionCreationPolicy.STATELESS
```

Does that automatically mean CSRF is unnecessary?

**No.**

You have to ask:

> **Where does the browser get the credential from?**

If:

```text id="e8q2n6"
Authorization header
```

CSRF risk is generally much lower for that authentication mechanism.

If:

```text id="a4v9p1"
Cookie
```

the browser can automatically attach the credential, so CSRF can still matter.

---

# 20. When Should We Keep CSRF Enabled?

Keep CSRF protection when your application uses browser-managed credentials in ways that make CSRF possible, especially:

```text id="u7p3m4"
HTTP Session
Cookie-based authentication
Traditional server-rendered forms
Cookie-based SPA authentication
```

Spring Security provides CSRF support for these scenarios. ([Home][1])

---

# 21. When Is Disabling CSRF Reasonable?

For a REST API where:

```text id="x2n8q5"
Authentication:
Authorization: Bearer <JWT>
```

and authentication isn't stored in browser cookies, disabling CSRF is commonly reasonable because the credential isn't automatically attached by the browser.

But you should make that decision based on the actual credential transport, not simply because:

> "It's an API."

---

# 22. Example: Our Employee API

We have:

```http id="mzj7q1"
Authorization: Bearer eyJ...
```

and:

```java id="e4q8n2"
SessionCreationPolicy.STATELESS
```

So our architecture is:

```text id="j9m3p6"
React
  ↓
Authorization Header
  ↓
Spring Security
  ↓
JWT Validation
  ↓
Authorization
```

For this architecture, disabling CSRF is reasonable.

So:

```java id="k7v4x8"
.csrf(csrf -> csrf.disable())
```

fits our specific design.

---

# 23. But Don't Do This in a Traditional MVC App

Suppose:

```text id="h5q2r9"
@Controller
   ↓
JSP
   ↓
JSESSIONID
```

and you simply write:

```java id="k9m3v1"
.csrf(csrf -> csrf.disable())
```

without another protection strategy.

That could leave state-changing browser requests vulnerable to CSRF.

So:

```text id="b8x4p6"
REST + Bearer Header
    → CSRF often disabled

Session/Cookie Browser App
    → CSRF protection normally retained
```

---

# 24. CORS vs CSRF vs JWT

Now put the three concepts together.

```text id="p7m2x9"
CORS
 ↓
Can this browser origin call the API?

JWT
 ↓
Who is making the authenticated request?

CSRF
 ↓
Can another site forge a request using
automatically attached credentials?
```

These solve different problems.

---

# 25. Interview Questions

### What is CSRF?

> Cross-Site Request Forgery is an attack where a malicious site tricks a user's browser into sending a state-changing authenticated request to another application, typically by exploiting automatically attached credentials such as cookies.

### Why is CSRF commonly associated with sessions/cookies?

> Browsers automatically attach cookies to requests, which means a malicious site can potentially cause an authenticated request without knowing the cookie value.

### Is JWT immune to CSRF?

> Not inherently. If the JWT is sent through a browser-managed cookie, CSRF can still be relevant. If it is sent explicitly in an `Authorization: Bearer` header, the authentication mechanism is generally not vulnerable to this same cookie-based CSRF pattern.

### Why do we often disable CSRF for JWT REST APIs?

> Because a stateless bearer-token API using an explicitly supplied Authorization header does not rely on browser-automatically-attached authentication cookies.

### Does `STATELESS` automatically mean CSRF can be disabled?

> No. The important question is how authentication credentials are transported.

### What does `CsrfFilter` do?

> It determines whether a request needs CSRF protection and, when required, validates the submitted CSRF token against the expected token. ([Home][1])

---

# 26. Best Practices

```text id="d3x8m1"
✅ Understand how authentication credentials are transported
✅ Keep CSRF enabled for cookie/session-based browser apps
✅ For bearer-token APIs, evaluate whether CSRF applies to the actual design
✅ Don't disable CSRF blindly
✅ Don't confuse CSRF with CORS
✅ Don't use GET for state-changing operations
✅ Use HTTPS
```

---

# 📍 Where We Are

```text id="b6m2q9"
Spring Security
│
├── ✅ Chapter 1 — Why Security
├── ✅ Chapter 2 — SecurityFilterChain
├── ✅ Chapter 3 — Authentication / SecurityContext
├── ✅ Chapter 4 — UserDetails / UserDetailsService
├── ✅ Chapter 5 — PasswordEncoder / BCrypt
├── ✅ Chapter 6 — AuthenticationManager / Providers
├── ✅ Chapter 7 — Roles / Authorities
├── ✅ Chapter 8 — URL Authorization
├── ✅ Chapter 9 — Method Security
├── ✅ Chapter 10 — HTTP Basic
├── ✅ Chapter 11 — Stateful vs Stateless
├── ✅ Chapter 12 — JWT Fundamentals
├── ✅ Chapter 13 — JWT Login & Generation
├── ✅ Chapter 14 — JWT Validation
├── ✅ Chapter 15 — OncePerRequestFilter
├── ✅ Chapter 16 — Security Exception Handling
├── ✅ Chapter 17 — CORS
├── ✅ Chapter 18 — CSRF
│
└── ⏭️ Chapter 19 — Spring Security Testing
      ↓
      SecurityMockMvcRequestPostProcessors
      ↓
      @WithMockUser
      ↓
      Testing authenticated requests
      ↓
      Testing roles
      ↓
      Testing JWT-secured endpoints
      ↓
      Testing 401 / 403
```

Next we'll cover **Spring Security Testing**, including how to test `ROLE_USER` vs `ROLE_ADMIN`, authenticated/unauthenticated requests, CSRF-protected requests, and JWT-secured endpoints without manually constructing the entire security stack for every test.

[1]: https://docs.spring.io/spring-security/reference/7.0/servlet/exploits/csrf.html?utm_source=chatgpt.com "Cross Site Request Forgery (CSRF) :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/features/exploits/csrf.html?utm_source=chatgpt.com "Cross Site Request Forgery (CSRF) :: Spring Security"
