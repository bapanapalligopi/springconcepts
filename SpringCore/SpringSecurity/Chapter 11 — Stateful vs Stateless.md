# Spring Security — Chapter 11: Stateful vs Stateless Authentication

This is the **bridge between the username/password security we've learned and JWT**.

We already know:

```text
HTTP Request
   ↓
SecurityFilterChain
   ↓
Authentication
   ↓
Authorization
```

Now the important question is:

> **After a user is authenticated, how does the server remember that authentication for the next request?**

There are two broad approaches:

```text id="m3c7w1"
STATEFUL
   ↓
Server-side session

STATELESS
   ↓
Client sends authentication information again
```

Spring Security's session-management documentation explicitly supports both session-based authentication persistence and stateless authentication strategies. ([Home][1])

---

# 1. Why do we need to understand this?

Suppose Rahul successfully logs in.

```text id="q4n8s2"
Request 1

username = rahul
password = password123

        ↓

Authentication SUCCESS
```

Now Rahul makes another request:

```http id="v7b2m9"
GET /api/employees
```

How does the server know:

> "This is Rahul, and he is already authenticated"?

There are different ways to solve this.

---

# 2. Stateful Authentication

In a traditional session-based application:

```text id="k9p3r5"
Login
 ↓
Authenticate
 ↓
Create HTTP Session
 ↓
Store SecurityContext/session information
 ↓
Return session cookie
```

The client receives something like:

```http id="x1w4q8"
Set-Cookie: JSESSIONID=abc123
```

Then the next request contains:

```http id="n6c2v7"
Cookie: JSESSIONID=abc123
```

Server uses the session to recover the user's authentication.

---

# 3. Stateful Flow

```text id="a5d8k3"
LOGIN
  ↓
username/password
  ↓
Authentication
  ↓
SecurityContext
  ↓
HTTP Session
  ↓
JSESSIONID
```

Next request:

```text id="e7r2m6"
GET /api/employees
  ↓
JSESSIONID
  ↓
Server Session
  ↓
SecurityContext
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
```

So the server maintains state about the authenticated user.

---

# 4. What is the HTTP Session?

The session is server-side state associated with a client session.

Conceptually:

```text id="y8c1p4"
JSESSIONID=abc123
        ↓
Server
        ↓
Session
        ↓
SecurityContext
        ↓
Rahul
```

The browser doesn't normally contain the whole authentication state; it contains the session identifier.

---

# 5. Where is Stateful Security Common?

Traditional web applications often use session-based authentication:

```text id="z2m7q5"
Browser
   ↓
Login Form
   ↓
Session
   ↓
JSESSIONID
   ↓
Server
```

Typical examples:

```text id="w4k9c2"
JSP applications
Thymeleaf applications
Traditional enterprise web applications
```

This fits naturally with server-rendered MVC.

---

# 6. Stateless Authentication

Now consider a REST API.

We want:

```text id="p8r3x6"
GET /api/employees
```

The server should not need to keep an authenticated session for every client.

Instead, each request carries authentication information.

Conceptually:

```text id="q7m2v9"
Request
  ↓
Bearer Token
  ↓
Validate Token
  ↓
Create Authentication
  ↓
SecurityContext
  ↓
Authorization
```

No server-side login session is required to remember the authentication between requests.

---

# 7. Stateless Flow

Suppose the client has a JWT:

```text id="h3c8k5"
eyJhbGciOiJIUzI1Ni...
```

The client sends:

```http id="r9n4p2"
Authorization: Bearer eyJ...
```

For each request:

```text id="u6x1m8"
Request
   ↓
Bearer Token
   ↓
JWT Validation
   ↓
Authentication
   ↓
SecurityContext
   ↓
Authorization
   ↓
Controller
```

This is the model we'll use for our REST API.

---

# 8. Stateful vs Stateless

| Stateful                          | Stateless                                                |
| --------------------------------- | -------------------------------------------------------- |
| Server stores session state       | Server doesn't need auth session state                   |
| Usually JSESSIONID cookie         | Commonly Bearer token                                    |
| Session identifies client         | Token carries authentication claims/identity information |
| Common in traditional web apps    | Common in REST APIs                                      |
| Session management required       | Token validation on requests                             |
| Scaling requires session strategy | Easier horizontal scaling in this respect                |

Stateless does **not** mean the server has literally no state anywhere. The key point is that authentication state isn't persisted server-side in an HTTP session between requests.

---

# 9. Why is Stateless Useful for REST APIs?

Imagine:

```text id="w3p7n9"
Load Balancer
      │
 ┌────┼────┐
 ▼    ▼    ▼
App1 App2 App3
```

With a stateless token:

```text id="r5k2c8"
Request
   ↓
App1

Next Request
   ↓
App3
```

Each application instance can validate the token independently.

You don't inherently need to route the same client back to the same server instance simply because of authentication session state.

That's one major reason stateless authentication fits distributed REST architectures well.

---

# 10. What does `SessionCreationPolicy.STATELESS` mean?

Spring Security provides:

```java id="v8m3q1"
SessionCreationPolicy.STATELESS
```

Example:

```java id="n2c7r5"
http.sessionManagement(session ->
    session.sessionCreationPolicy(
        SessionCreationPolicy.STATELESS
    )
);
```

It tells Spring Security:

> **Don't use the HTTP session to obtain/store the `SecurityContext` for this application flow.**

Spring Security documents `STATELESS` as a policy where the application will never create an `HttpSession` and will never use it to obtain the `SecurityContext`. ([Home][1])

---

# 11. Complete REST Security Configuration

For our future JWT API, the basic shape becomes:

```java id="k5r8n3"
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .csrf(csrf -> csrf.disable())

            .sessionManagement(session ->
                session.sessionCreationPolicy(
                    SessionCreationPolicy.STATELESS
                )
            )

            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**")
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

**Important:** We're showing the structural configuration here. We're not yet implementing JWT validation. Also, don't blindly disable CSRF just because an API is present; whether CSRF is needed depends on how credentials are transported and whether the application uses browser-managed credentials such as cookies. Spring Security enables CSRF protection by default for unsafe methods in typical servlet applications. ([Home][2])

We'll cover CSRF properly later.

---

# 12. Why JWT Fits Stateless REST Security

JWT stands for:

> **JSON Web Token**

A JWT lets a server issue a signed token representing claims about an authenticated subject.

Then the client sends:

```http id="q7f3m9"
Authorization: Bearer <JWT>
```

Spring Security can validate bearer tokens through its OAuth 2.0 Resource Server support; JWT is one of the supported bearer-token formats. ([Home][3])

---

# 13. Very Important: JWT Does NOT Automatically Mean "Authentication Server"

This is an important conceptual distinction.

There are usually three roles:

```text id="m9r2c6"
Client
   ↓
Authentication
   ↓
Authorization Server

        ↓ JWT

Resource Server
   ↓
Protected API
```

For a simple project, people often implement login and JWT creation in the same Spring application.

In a larger architecture, an **Authorization Server** may issue tokens while your application acts as a **Resource Server** that validates them.

Spring Security officially provides Resource Server support for JWT and opaque bearer tokens. ([Home][3])

For your 1.5-year level, we'll first learn the practical single-application JWT pattern.

---

# 14. What happens with JWT?

Login:

```text id="v1c8x4"
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
Authentication SUCCESS
       ↓
Generate JWT
       ↓
Return JWT to Client
```

Later:

```text id="b7m3q9"
Client
   ↓
Authorization: Bearer <JWT>
   ↓
JWT Validation
   ↓
Authentication
   ↓
SecurityContext
   ↓
Authorization
   ↓
Controller
```

Notice something important:

> **The login flow and the request-authentication flow are different.**

---

# 15. Session vs JWT

This is a very common interview comparison.

### Session

```text id="y3p8k1"
Login
 ↓
Session created
 ↓
JSESSIONID
 ↓
Server looks up session
```

### JWT

```text id="c6m2r9"
Login
 ↓
JWT generated
 ↓
Client stores token
 ↓
Bearer token sent with requests
 ↓
Server validates token
```

---

# 16. Where is the User Stored?

### Stateful session

The authentication context is persisted through the server-side session mechanism.

```text id="m7q4v2"
Server
 ↓
Session
 ↓
SecurityContext
 ↓
User
```

### Stateless JWT

The client sends the bearer token, and the resource server reconstructs an authenticated security context for the request after validating the token.

```text id="r2c8n5"
Client
 ↓
JWT
 ↓
Validate
 ↓
Authentication
 ↓
SecurityContext
```

The context still exists **during request processing**. Stateless does not mean there is no `SecurityContext`; it means the authentication isn't persisted in the HTTP session between requests. ([Home][1])

---

# 17. Does Stateless Mean "No SecurityContext"?

**No.**

This is an important interview trap.

During a request:

```text id="p6x3m8"
JWT
 ↓
Authentication
 ↓
SecurityContext
 ↓
Controller
```

The `SecurityContext` still exists for that request.

What changes is **where the authentication state comes from** and whether it is persisted in a server-side HTTP session.

---

# 18. Does JWT Automatically Mean Better Security?

No.

JWT solves a particular problem: carrying verifiable claims in a token.

You still need to think about:

```text id="u7m4q2"
HTTPS
Token expiry
Token storage
Key management
Refresh tokens
Revocation strategy
CSRF depending on credential transport
CORS
Least privilege
```

A badly designed JWT system can be insecure.

---

# 19. JWT Access Token

The token typically contains claims such as:

```text id="c9r2m6"
sub = rahul
roles = USER
iat = ...
exp = ...
```

A signature allows the resource server to verify that the token wasn't modified.

We'll dissect the structure next.

---

# 20. Why don't we put the password inside JWT?

Never.

A JWT can contain claims necessary for authorization, such as:

```text id="k8m3q5"
subject
roles
expiration
issuer
```

but the user's password should never be placed inside the token.

The token is not a secure password vault.

---

# 21. JWT Expiration

A token should generally have an expiration time:

```text id="f4n7x2"
exp = expiration
```

Example:

```text id="r8c1m5"
Access Token
Lifetime = 15 minutes
```

After expiration:

```text id="q2v9k6"
JWT validation
     ↓
Expired
     ↓
401
```

Short-lived access tokens reduce the impact of token theft, although they don't eliminate the need for a broader token lifecycle strategy.

---

# 22. Access Token vs Refresh Token

We'll study these properly later, but understand the basic idea.

### Access Token

Short-lived:

```text id="n5x2c8"
Used for API access
```

### Refresh Token

Longer-lived:

```text id="m7q4r1"
Used to obtain a new access token
```

Conceptually:

```text id="b9k3v6"
Login
 ↓
Access Token + Refresh Token
```

Then:

```text id="d1x8m4"
Access Token expires
 ↓
Refresh Token
 ↓
New Access Token
```

Don't implement refresh tokens yet. We'll first understand plain JWT.

---

# 23. What about Session Fixation?

Session-related attacks matter primarily to stateful applications.

Spring Security provides session-management protections, including session fixation protection. ([Home][1])

For a JWT-based stateless API, HTTP session fixation isn't the central authentication model because we're not relying on an HTTP session to persist authentication.

But security concepts don't disappear just because you use JWT.

---

# 24. When should you use Session Authentication?

Good fit:

```text id="z7m1q5"
Traditional MVC application
Server-rendered UI
JSP / Thymeleaf
Browser-centric login
```

Example:

```text id="7t2c9r"
Browser
 ↓
Login Form
 ↓
Session
 ↓
JSESSIONID
```

---

# 25. When should you use Stateless Authentication?

Good fit:

```text id="p8m4x1"
REST APIs
Mobile clients
SPA + API architectures
Microservices
Distributed services
```

Example:

```text id="y6q2n9"
Mobile App
 ↓
Bearer Token
 ↓
REST API
```

This is a guideline, not an absolute rule.

---

# 26. Interview Questions

### What is stateful authentication?

> Authentication state is persisted server-side, commonly using an HTTP session.

### What is stateless authentication?

> Each request carries the authentication information needed to authenticate the request, without relying on server-side HTTP session state between requests.

### What does `SessionCreationPolicy.STATELESS` do?

> It configures Spring Security not to create or use the HTTP session for obtaining the security context in the stateless flow. ([Home][1])

### Does stateless authentication mean there is no `SecurityContext`?

> No. Spring Security can still create a `SecurityContext` for the current request; it simply isn't persisted in the HTTP session.

### Why is JWT commonly used for REST APIs?

> JWT is a bearer-token format that allows a resource server to validate the token and establish authentication for each request without depending on a server-side login session. Spring Security provides JWT bearer-token validation through OAuth 2.0 Resource Server support. ([Home][4])

### Session vs JWT?

> Session authentication stores authentication state server-side; JWT-based stateless authentication carries a verifiable bearer token with requests and reconstructs authentication per request.

---

# 27. Best Practices

For our upcoming JWT implementation:

```text id="q3m8v6"
✅ Use HTTPS
✅ Use short-lived access tokens
✅ Validate signature
✅ Validate expiration
✅ Validate issuer/audience where appropriate
✅ Use strong signing keys
✅ Keep token claims minimal
✅ Don't put passwords in JWT
✅ Plan token revocation/refresh carefully
✅ Don't assume "JWT = secure"
```

Spring Security's Resource Server support can automatically validate JWT bearer tokens once configured with the required decoder/key or issuer information. ([Home][4])

---

# 📍 Where We Are

```text id="w2p7m4"
Spring Security
│
├── ✅ Chapter 1 — Why Security
├── ✅ Chapter 2 — SecurityFilterChain
├── ✅ Chapter 3 — Authentication / SecurityContext
├── ✅ Chapter 4 — UserDetails / UserDetailsService
├── ✅ Chapter 5 — PasswordEncoder / BCrypt
├── ✅ Chapter 6 — AuthenticationManager / Provider
├── ✅ Chapter 7 — Roles / Authorities
├── ✅ Chapter 8 — URL Authorization
├── ✅ Chapter 9 — Method Security
├── ✅ Chapter 10 — HTTP Basic
├── ✅ Chapter 11 — Stateful vs Stateless
│
└── ⏭️ Chapter 12 — JWT ⭐⭐⭐⭐⭐
      ↓
      What exactly is JWT?
      ↓
      Header
      ↓
      Payload
      ↓
      Signature
      ↓
      Claims
      ↓
      Access Token
      ↓
      JWT Authentication Flow
```

Next we'll dissect a JWT **part by part**, then build the complete login → token generation → `Authorization: Bearer` → token validation flow.

[1]: https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html?utm_source=chatgpt.com "Authentication Persistence and Session Management"
[2]: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html?utm_source=chatgpt.com "Cross Site Request Forgery (CSRF) :: Spring Security"
[3]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server"
[4]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT"
