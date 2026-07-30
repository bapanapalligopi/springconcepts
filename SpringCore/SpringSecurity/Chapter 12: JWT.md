# Spring Security — Chapter 12: JWT

Now we reach the **most important Spring Security topic for REST APIs**.

You already understand:

```text
Authentication
       ↓
Authorization
       ↓
SecurityContext
       ↓
SecurityFilterChain
```

Now we replace the Basic Authentication credential flow with a **Bearer Token**, typically a JWT.

Spring Security's OAuth 2.0 Resource Server support can protect APIs with bearer tokens, including JWTs. By default, bearer tokens are resolved from the `Authorization` header. ([Home][1])

We'll follow:

> **Why → What → How → Where → Internal Flow → JWT Structure → Code → Interview Questions → Best Practices**

---

# 1. Why do we need JWT?

With Basic Authentication, the client sends username/password credentials with requests.

Conceptually:

```text
Every request
     ↓
Username + Password
     ↓
Authentication
```

For a REST API, we'd rather avoid repeatedly sending the user's password.

JWT changes the model:

```text
Login
  ↓
Username + Password
  ↓
Authenticate once
  ↓
Issue JWT
  ↓
Client sends JWT on later requests
```

So:

```text
Password
   ↓
Used during login

JWT
   ↓
Used for subsequent API requests
```

---

# 2. What is JWT?

JWT means:

> **JSON Web Token**

A JWT represents a set of claims encoded into a compact token representation. Spring Security's `Jwt` type represents a JWT and its claims. ([Home][2])

A JWT commonly looks like:

```text
xxxxx.yyyyy.zzzzz
```

There are three sections:

```text
Header.Payload.Signature
```

---

# 3. JWT Structure

```text
JWT
│
├── Header
├── Payload
└── Signature
```

For learning, this is the most important JWT diagram.

---

# 4. Header

The header contains metadata about the token, such as the signing algorithm.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Conceptually:

```text
Header
  ↓
"How was this token signed?"
```

The exact algorithms available depend on the JWT/JWS setup and keys you configure.

---

# 5. Payload

The payload contains **claims**.

Example:

```json
{
  "sub": "rahul",
  "roles": ["USER"],
  "iat": 1753862400,
  "exp": 1753863300
}
```

Claims are simply pieces of information about the token subject or its context. Spring's JWT model represents these as a JSON claims set. ([Home][2])

Common claims include:

```text
sub → subject
iat → issued at
exp → expiration
iss → issuer
aud → audience
```

You can also have application-specific claims, such as:

```text
roles
department
tenant
```

---

# 6. Signature

The signature is what allows the resource server to verify that the signed JWT hasn't been altered.

Conceptually:

```text
Header
   +
Payload
   ↓
Signing Algorithm + Key
   ↓
Signature
```

Later, when the token arrives:

```text
Header + Payload
       ↓
Verify Signature
       ↓
Valid?
```

If someone changes:

```json
"roles": ["USER"]
```

to:

```json
"roles": ["ADMIN"]
```

the signature should no longer validate.

---

# 7. Very Important: JWT Is Not Encryption

This is one of the most important interview questions.

A normal signed JWT is generally **signed**, not encrypted.

That means the payload should not be treated as secret.

So don't put:

```text
password
credit-card number
database credentials
secret business data
```

inside a normal JWT.

A client that possesses the token can generally inspect its encoded claims even though it cannot validly modify the signed token without the signing key.

So:

```text
JWT
  ↓
Integrity / authenticity
```

not:

```text
JWT
  ↓
Confidentiality
```

---

# 8. What does "Bearer Token" mean?

The client sends:

```http
Authorization: Bearer <token>
```

The word `Bearer` means, conceptually:

> Whoever possesses this token may use it as the bearer credential.

Spring Security's resource-server support looks for bearer tokens in the `Authorization` header by default. ([Home][3])

So:

```text
Client
   ↓
Authorization: Bearer eyJ...
```

---

# 9. JWT Authentication Flow

This is the most important diagram in this chapter.

## Login

```text
Client
   ↓
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
Authentication Success
   ↓
Generate JWT
   ↓
Return JWT
```

## Subsequent request

```text
Client
   ↓
Authorization: Bearer <JWT>
   ↓
BearerTokenAuthenticationFilter
   ↓
JWT Authentication
   ↓
JwtDecoder / Validation
   ↓
Authentication
   ↓
SecurityContext
   ↓
Authorization
   ↓
Controller
```

Spring Security's resource-server configuration installs a `BearerTokenAuthenticationFilter` that parses bearer tokens and attempts authentication. ([Home][4])

---

# 10. Notice Something Important

With Basic:

```text
Request
 ↓
Basic credentials
 ↓
Username/password authentication
```

With JWT:

```text
Login
 ↓
Username/password authentication
 ↓
JWT issued

Later:
Request
 ↓
JWT
 ↓
Validate JWT
 ↓
Authentication
```

So the password is involved in the **login step**, but generally not in every API request.

---

# 11. Who Creates the JWT?

This is where many beginners get confused.

There are two different responsibilities:

### Authorization Server

Issues access tokens.

### Resource Server

Accepts and validates access tokens to protect APIs.

Spring Security has dedicated Resource Server support for validating bearer tokens. ([Home][5])

In a simple learning project, you may put login/token generation and API protection in the **same application**.

In a larger OAuth2 architecture:

```text
Client
   ↓
Authorization Server
   ↓
JWT Access Token
   ↓
Resource Server / API
```

For your experience level, we'll first understand the single-application implementation.

---

# 12. JWT Validation

Suppose the API receives:

```http
Authorization: Bearer eyJ...
```

The resource server needs to determine:

```text
Is this token valid?
```

Validation commonly includes:

```text
Signature
Expiration
Issuer
Audience
Other configured claims
```

Spring Security's JWT resource-server support uses a `JwtDecoder` for decoding and validating JWTs. When configured through an issuer, Spring Boot can auto-configure the resource-server validation infrastructure. ([Home][6])

---

# 13. What is `JwtDecoder`?

`JwtDecoder` is the component responsible for decoding and validating JWTs.

Conceptually:

```text
JWT
 ↓
JwtDecoder
 ↓
Validate
 ↓
Jwt
```

The resulting `Jwt` object contains the claims.

Spring Security provides implementations and configuration mechanisms for constructing a `JwtDecoder`. ([Home][7])

---

# 14. What is `JwtAuthenticationProvider`?

This is another important internal component.

For JWT bearer authentication, Spring Security provides:

```text
JwtAuthenticationProvider
```

It is an `AuthenticationProvider` for JWT-encoded bearer tokens. ([Home][8])

Conceptually:

```text
Bearer Token
     ↓
BearerTokenAuthenticationFilter
     ↓
AuthenticationManager
     ↓
JwtAuthenticationProvider
     ↓
JwtDecoder
     ↓
Validated Jwt
     ↓
Authentication
```

Notice that this is different from:

```text
DaoAuthenticationProvider
```

which we studied for username/password authentication.

---

# 15. DaoAuthenticationProvider vs JwtAuthenticationProvider

This is a great interview comparison.

### Username/password

```text
DaoAuthenticationProvider
       ↓
UserDetailsService
       ↓
PasswordEncoder
```

### JWT bearer token

```text
JwtAuthenticationProvider
       ↓
JwtDecoder
       ↓
Validate JWT
```

So:

```text
Username + Password
    ↓
DaoAuthenticationProvider

Bearer JWT
    ↓
JwtAuthenticationProvider
```

---

# 16. Do we need `UserDetailsService` for every JWT request?

**Not necessarily.**

That's an important point.

With a self-contained JWT:

```text
JWT
 ↓
Claims
 ↓
Validate signature/claims
 ↓
Create Authentication
```

The resource server doesn't necessarily need to query the user database on every request.

That is one of the attractive properties of self-contained tokens.

However, if your application needs current user data, permissions, revocation checks, or other dynamic information, you may still consult a database or another service.

---

# 17. Where Does the User's Role Come From?

Suppose JWT contains:

```json
{
  "sub": "rahul",
  "roles": ["USER"]
}
```

After validation, your application needs to map that claim into Spring Security authorities.

Conceptually:

```text
JWT
 ↓
roles = USER
 ↓
GrantedAuthority
 ↓
ROLE_USER
```

That mapping is important.

We'll implement it in a later chapter.

---

# 18. JWT vs Session

Now the difference should be clear.

### Session

```text
Login
 ↓
Server Session
 ↓
JSESSIONID
```

### JWT

```text
Login
 ↓
JWT
 ↓
Client stores token
 ↓
Bearer token on each request
```

And:

```text
Session
 → Server-side authentication state

JWT
 → Token carries claims that can be validated
```

---

# 19. JWT Expiration

A token should normally have an expiration claim:

```json
{
  "sub": "rahul",
  "exp": 1753863300
}
```

After expiration:

```text
Request
  ↓
JWT
  ↓
exp exceeded
  ↓
Validation fails
  ↓
401
```

Token expiration is a central part of JWT validation.

---

# 20. JWT Signing Algorithms

Common signing approaches include:

```text
HS256
RS256
ES256
```

The important conceptual distinction for your level is:

### Symmetric signing

```text
Same secret
   ↓
Sign
   ↓
Verify
```

Example:

```text
HS256
```

### Asymmetric signing

```text
Private key
   ↓
Sign

Public key
   ↓
Verify
```

Example:

```text
RS256
```

Asymmetric signing is especially useful when a centralized authorization server signs tokens and multiple resource servers need to verify them without possessing the private signing key.

---

# 21. Why Is Asymmetric Signing Useful?

Imagine:

```text
Authorization Server
     ↓
Private Key
     ↓
Signs JWT
```

and:

```text
Employee API
Order API
Payment API
```

Each resource server gets:

```text
Public Key
```

They can verify:

```text
Is this token genuinely signed?
```

without being able to create new valid tokens themselves.

This is a very useful microservices architecture pattern.

---

# 22. Minimal Spring Resource Server Configuration

For an API that validates JWTs issued by an external authorization server, Spring Security can be configured as a Resource Server.

For example:

```java
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
            .anyRequest()
                .authenticated()
        )
        .oauth2ResourceServer(
            oauth2 -> oauth2.jwt(
                Customizer.withDefaults()
            )
        );

    return http.build();
}
```

With Spring Boot, a typical production setup can use `spring.security.oauth2.resourceserver.jwt.issuer-uri` so that the resource server discovers the authorization server metadata and keys and configures JWT validation automatically. ([Home][6])

For example:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://issuer.example.com
```

This is an important distinction from the simple custom JWT tutorial flow we'll build later: in a production OAuth2 architecture, **your API often validates tokens issued by a dedicated authorization server**. ([Home][6])

---

# 23. What Happens Internally With Resource Server JWT?

```text
HTTP Request
      ↓
Authorization: Bearer eyJ...
      ↓
BearerTokenAuthenticationFilter
      ↓
AuthenticationManager
      ↓
JwtAuthenticationProvider
      ↓
JwtDecoder
      ↓
Signature / Claims Validation
      ↓
Authenticated Authentication
      ↓
SecurityContext
      ↓
Authorization
      ↓
Controller
```

This is the most important JWT request flow.

Spring Security's Resource Server infrastructure wires a bearer-token filter and JWT authentication provider around the JWT decoder. ([Home][4])

---

# 24. What Exactly Is Inside the JWT?

Let's use:

```json
{
  "sub": "rahul",
  "roles": ["USER"],
  "iss": "https://auth.example.com",
  "aud": ["employee-api"],
  "iat": 1753862400,
  "exp": 1753863300
}
```

Conceptually:

```text
sub
 ↓
Who?

roles
 ↓
Authorities/roles

iss
 ↓
Who issued the token?

aud
 ↓
Who is this token intended for?

iat
 ↓
When was it issued?

exp
 ↓
When does it expire?
```

These claims are useful for authentication and authorization decisions.

---

# 25. Don't Put Too Much Data in JWT

Bad:

```json
{
  "password": "...",
  "address": "...",
  "entireEmployeeObject": "...",
  "databaseData": "..."
}
```

Better:

```json
{
  "sub": "rahul",
  "roles": ["USER"],
  "exp": 1753863300
}
```

A JWT is sent with requests, so keeping it compact and limiting claims is generally a good design.

---

# 26. JWT Is Not Automatically Revocable

This is another important interview topic.

Suppose the user receives:

```text
JWT
expires in 15 minutes
```

If you want to immediately invalidate it:

```text
User logs out
```

a self-contained JWT may remain cryptographically valid until it expires unless you introduce additional revocation mechanisms.

That's one reason:

* short access-token lifetimes,
* refresh-token strategies,
* token revocation,
* introspection,
* or server-side deny lists

may be considered depending on the architecture.

Spring Security also supports opaque token introspection, where the authorization server can be consulted to determine whether a token is currently valid. ([Home][9])

For your level, remember:

> **JWT validation and token revocation are different problems.**

---

# 27. JWT vs Opaque Token

Spring Security Resource Server supports both:

```text
JWT
Opaque Token
```

JWT:

```text
Token itself contains claims
 ↓
Validate locally
```

Opaque token:

```text
Token
 ↓
Authorization Server introspection endpoint
 ↓
Is it valid?
```

Spring Security explicitly supports both bearer-token approaches. ([Home][5])

For our roadmap, we'll focus on **JWT**.

---

# 28. Interview Questions

### What is JWT?

> JSON Web Token is a compact token format containing claims that can be digitally signed and validated.

### What are the three parts of a typical signed JWT?

```text
Header.Payload.Signature
```

### Is JWT encrypted?

> A normal signed JWT is not encrypted. Its payload can generally be decoded, while the signature provides integrity/authenticity.

### What is a Bearer token?

> A token presented by the client as proof of authorization, commonly through `Authorization: Bearer <token>`. Spring Security's Resource Server looks for bearer tokens in that header by default. ([Home][3])

### What is `JwtDecoder`?

> It is the component used to decode and validate JWTs.

### What is `JwtAuthenticationProvider`?

> It is an `AuthenticationProvider` used to authenticate JWT-encoded bearer tokens. ([Home][8])

### DaoAuthenticationProvider vs JwtAuthenticationProvider?

> `DaoAuthenticationProvider` authenticates username/password using `UserDetailsService` and `PasswordEncoder`; `JwtAuthenticationProvider` authenticates JWT bearer tokens using JWT validation. ([Home][8])

### Why use JWT for REST APIs?

> It enables bearer-token authentication without requiring the API to persist authentication in an HTTP session between requests.

---

# 29. Best Practices

```text
✅ Always use HTTPS
✅ Validate JWT signature
✅ Validate exp
✅ Validate issuer/audience where applicable
✅ Keep claims minimal
✅ Never put passwords in JWT
✅ Use short-lived access tokens
✅ Protect signing keys carefully
✅ Use asymmetric signing where architecture benefits from it
✅ Plan token revocation/refresh
✅ Don't assume JWT itself provides confidentiality
```

---

# 📍 Where We Are

```text
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
│
└── ⏭️ Chapter 13 — JWT Login & Token Generation ⭐⭐⭐⭐⭐
      ↓
      Login endpoint
      ↓
      AuthenticationManager
      ↓
      Authenticate username/password
      ↓
      Generate JWT
      ↓
      Return access token
      ↓
      Bearer token on API requests
```

Next we'll build the **actual login flow** rather than just discussing JWT: a `/api/auth/login` endpoint, `AuthenticationManager`, successful authentication, JWT generation, and the response containing the access token.

[1]: https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html?utm_source=chatgpt.com "OAuth2 :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/Jwt.html?utm_source=chatgpt.com "Jwt (spring-security-docs 7.1.0 API)"
[3]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/bearer-tokens.html?utm_source=chatgpt.com "OAuth 2.0 Bearer Tokens"
[4]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/config/annotation/web/configurers/oauth2/server/resource/OAuth2ResourceServerConfigurer.html?utm_source=chatgpt.com "Class OAuth2ResourceServerConfigurer<H extends ..."
[5]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server"
[6]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT"
[7]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/package-summary.html?utm_source=chatgpt.com "Package org.springframework.security.oauth2.jwt"
[8]: https://docs.spring.io/spring-security/reference/api/java/allclasses-index.html?utm_source=chatgpt.com "All Classes and Interfaces (spring-security-docs 7.1.0 API)"
[9]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/opaque-token.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server Opaque Token :: Spring Security"
