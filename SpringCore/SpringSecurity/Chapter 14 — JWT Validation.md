# Spring Security — Chapter 14: JWT Validation

Now we complete the **second half of JWT**.

We already built:

```text id="a91f02"
LOGIN
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
JwtEncoder
 ↓
JWT
 ↓
Client
```

Now the client sends:

```http
Authorization: Bearer eyJ...
```

and our API must validate it.

Spring Security's Resource Server support handles bearer-token authentication through `BearerTokenAuthenticationFilter`; for JWTs, `JwtAuthenticationProvider` uses a `JwtDecoder` to decode, verify, and validate the token, then creates an authenticated `Authentication`. ([Home][1])

---

# 1. Why do we need JWT validation?

A client can send any string:

```http id="n53mza"
Authorization: Bearer hello
```

We cannot trust it.

We need to answer:

```text id="u5z8f7"
Is this a real JWT?
Was it signed by our trusted issuer?
Has it been modified?
Has it expired?
Was it issued for this API?
```

That's JWT validation.

---

# 2. Complete Request Flow

For:

```http id="k7r4w3"
GET /api/employees
Authorization: Bearer <JWT>
```

the flow is:

```text id="f2x8q9"
Client
  ↓
BearerTokenAuthenticationFilter
  ↓
BearerTokenAuthenticationToken
  ↓
AuthenticationManager
  ↓
JwtAuthenticationProvider
  ↓
JwtDecoder
  ↓
Verify + Validate JWT
  ↓
JwtAuthenticationToken
  ↓
SecurityContextHolder
  ↓
Authorization
  ↓
DispatcherServlet
  ↓
Controller
```

This is the key JWT architecture to remember. ([Home][1])

---

# 3. What is `BearerTokenAuthenticationFilter`?

This filter looks for a bearer token in the request.

By default:

```http id="ufw1k1"
Authorization: Bearer <token>
```

Spring Security documents `BearerTokenAuthenticationFilter` as the filter that authenticates requests containing OAuth 2.0 bearer tokens. ([Home][2])

Conceptually:

```text id="g2ynqk"
HTTP Request
    ↓
BearerTokenAuthenticationFilter
    ↓
Find Authorization header
    ↓
Extract token
```

---

# 4. What happens when the token is found?

Spring creates an authentication object representing the bearer token:

```text id="r3d9b4"
BearerTokenAuthenticationToken
```

Then:

```text id="h8x2m1"
BearerTokenAuthenticationToken
        ↓
AuthenticationManager
```

Spring Security's resource-server flow documents this exact transition. ([Home][1])

---

# 5. What is `JwtAuthenticationProvider`?

We previously learned:

```text id="d4m7x2"
Username + Password
        ↓
DaoAuthenticationProvider
```

For JWT:

```text id="f1k8n6"
Bearer JWT
        ↓
JwtAuthenticationProvider
```

`JwtAuthenticationProvider` is the `AuthenticationProvider` responsible for JWT bearer-token authentication. ([Home][3])

It uses:

```text id="n3c7p9"
JwtDecoder
+
JwtAuthenticationConverter
```

to process the token.

---

# 6. What is `JwtDecoder`?

`JwtDecoder` is responsible for decoding and validating the JWT. Its main operation is:

```java
Jwt decode(String token)
```

and it throws a `JwtException` when decoding/validation fails. ([Home][4])

Think:

```text id="v7q2m4"
JWT string
   ↓
JwtDecoder
   ↓
Validated Jwt object
```

---

# 7. What does validation mean?

At a practical level, the resource server checks things such as:

```text id="y8m4q6"
Signature
Expiration
Issuer
Other configured claims
```

When using issuer-based configuration, Spring Security can discover the authorization server metadata/JWK set and validate the JWT's signature and standard claims such as `iss`, `exp`, and `nbf`. ([Home][3])

---

# 8. Signature Validation

Suppose our JWT is:

```text id="p4g8x2"
header.payload.signature
```

The server verifies:

```text id="c9v3m6"
header + payload
       ↓
Trusted key
       ↓
Signature
```

If the token has been modified:

```text id="m6z1q8"
roles = USER
```

to:

```text id="b2r5k9"
roles = ADMIN
```

the signature should no longer validate.

So:

```text id="x1q7f4"
Modified JWT
   ↓
Signature invalid
   ↓
Authentication fails
```

---

# 9. Expiration Validation

Suppose the token contains:

```json id="x3n6j8"
{
  "sub": "rahul",
  "exp": 1753863300
}
```

If the current time is beyond `exp`:

```text id="m5q9c2"
JWT
 ↓
Expired
 ↓
Authentication failure
 ↓
401
```

Bearer-token failures are returned as authentication failures, and Spring Security can produce a `WWW-Authenticate` bearer error response. ([Home][5])

---

# 10. What if JWT is Invalid?

Examples:

```text id="k2m8x5"
Malformed token
Expired token
Wrong signature
Wrong issuer
Unsupported algorithm
```

Then:

```text id="r4c7v9"
JwtDecoder
   ↓
JwtException
   ↓
Authentication failure
   ↓
SecurityContext cleared
   ↓
401 Unauthorized
```

Spring Security documents that failed bearer authentication clears the security context and invokes the authentication entry point. ([Home][1])

---

# 11. Configure `JwtDecoder`

Because our learning project creates the JWT using a shared HMAC secret, we can validate it with the same secret.

```java
@Bean
JwtDecoder jwtDecoder(SecretKey jwtSecretKey) {

    return NimbusJwtDecoder
            .withSecretKey(jwtSecretKey)
            .build();
}
```

Spring Security's `NimbusJwtDecoder` supports creating a decoder with a shared secret key for HMAC validation. ([Home][6])

So:

```text id="p9f3d7"
JwtEncoder
   ↓
Secret Key
   ↓
Create JWT
```

and:

```text id="w6h2k8"
JwtDecoder
   ↓
Same Secret Key
   ↓
Validate JWT
```

This is **symmetric signing**.

---

# 12. Configure Resource Server

Now tell Spring Security:

> "Process Bearer JWTs."

```java
http
    .oauth2ResourceServer(oauth2 ->
        oauth2.jwt(
            Customizer.withDefaults()
        )
    );
```

Spring's Resource Server support configures the bearer-token infrastructure, including the JWT authentication provider. ([Home][6])

---

# 13. Complete Security Configuration

For our custom HMAC JWT learning project:

```java
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

                .requestMatchers(
                    "/api/auth/login"
                ).permitAll()

                .requestMatchers(
                    HttpMethod.GET,
                    "/api/employees/**"
                ).hasAnyRole(
                    "USER",
                    "ADMIN"
                )

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

                .anyRequest()
                    .denyAll()
            )

            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(
                    Customizer.withDefaults()
                )
            );

        return http.build();
    }

    @Bean
    JwtDecoder jwtDecoder(
            SecretKey jwtSecretKey) {

        return NimbusJwtDecoder
                .withSecretKey(jwtSecretKey)
                .build();
    }
}
```

Now the application has:

```text id="k3m7v1"
Login
 ↓
Generate JWT

Protected Request
 ↓
Validate JWT
 ↓
Authorize
```

---

# 14. But We Have a Problem: Roles

Our JWT contains:

```json id="s7h1q4"
{
  "sub": "rahul",
  "roles": [
    "ROLE_USER"
  ]
}
```

JWT validation succeeds.

But Spring Security still needs to turn those claims into:

```text id="q5m8x2"
GrantedAuthority
```

This is where:

```text id="g4n9c1"
JwtAuthenticationConverter
```

comes in.

Spring's JWT authentication architecture describes `JwtAuthenticationProvider` as using a `JwtAuthenticationConverter` to convert JWT claims into granted authorities. ([Home][3])

---

# 15. What is `JwtAuthenticationConverter`?

Its responsibility is essentially:

```text id="h6x2q8"
Validated JWT
    ↓
Extract authorities
    ↓
Authentication
```

By default, Spring Security has standard conventions for scopes, but our custom claim is:

```text id="g3r9m2"
roles
```

So we need custom mapping if we want:

```text id="q4k7v1"
ROLE_USER
ROLE_ADMIN
```

from our `roles` claim.

---

# 16. Custom Role Converter

Create:

```java
@Bean
JwtAuthenticationConverter jwtAuthenticationConverter() {

    JwtGrantedAuthoritiesConverter authoritiesConverter =
            new JwtGrantedAuthoritiesConverter();

    authoritiesConverter.setAuthoritiesClaimName(
            "roles"
    );

    authoritiesConverter.setAuthorityPrefix("");

    JwtAuthenticationConverter converter =
            new JwtAuthenticationConverter();

    converter.setJwtGrantedAuthoritiesConverter(
            authoritiesConverter
    );

    return converter;
}
```

Important:

We have:

```json
"roles": [
    "ROLE_USER"
]
```

and:

```java
setAuthorityPrefix("")
```

so the authority remains:

```text
ROLE_USER
```

---

# 17. Configure the Converter

Now:

```java
.oauth2ResourceServer(oauth2 ->
    oauth2.jwt(jwt ->
        jwt.jwtAuthenticationConverter(
            jwtAuthenticationConverter()
        )
    )
)
```

Complete:

```java
http
    .oauth2ResourceServer(oauth2 ->
        oauth2.jwt(jwt ->
            jwt.jwtAuthenticationConverter(
                jwtAuthenticationConverter()
            )
        )
    );
```

---

# 18. Now the Complete JWT Flow

```text id="x5q9m2"
Authorization: Bearer <JWT>
              ↓
BearerTokenAuthenticationFilter
              ↓
BearerTokenAuthenticationToken
              ↓
AuthenticationManager
              ↓
JwtAuthenticationProvider
              ↓
JwtDecoder
              ↓
Verify Signature
              ↓
Validate Claims
              ↓
Jwt
              ↓
JwtAuthenticationConverter
              ↓
GrantedAuthorities
              ↓
JwtAuthenticationToken
              ↓
SecurityContextHolder
              ↓
Authorization
              ↓
Controller
```

This is the **complete JWT authentication architecture** you should know for a 1.5–2 year Spring developer role. ([Home][1])

---

# 19. What is `JwtAuthenticationToken`?

After successful JWT authentication, Spring Security returns an authenticated object commonly represented as:

```text id="m7c2x5"
JwtAuthenticationToken
```

It contains:

```text id="r3v8q1"
Principal → Jwt
Authorities → ROLE_USER / ROLE_ADMIN / ...
```

Spring's JWT documentation describes the resulting authentication as `JwtAuthenticationToken`, with the `Jwt` as its principal and the converted authorities attached. ([Home][3])

---

# 20. Getting JWT Claims in Controller

Because the principal is a `Jwt`, you can access it.

For example:

```java
@GetMapping("/me")
public String me(
        @AuthenticationPrincipal Jwt jwt) {

    return jwt.getSubject();
}
```

If:

```json
"sub": "rahul"
```

then:

```text
rahul
```

is returned.

---

# 21. Getting Current User

You can also use:

```java
@GetMapping("/me")
public String me(
        Authentication authentication) {

    return authentication.getName();
}
```

With JWT authentication, `getName()` is commonly derived from the JWT subject (`sub`) unless configured otherwise. Spring's JWT resource-server documentation describes this default behavior. ([Home][3])

---

# 22. User Request Example

Suppose Rahul has:

```json
{
  "sub": "rahul",
  "roles": [
    "ROLE_USER"
  ]
}
```

Request:

```http
GET /api/employees
Authorization: Bearer eyJ...
```

Flow:

```text id="9c4k2p"
JWT valid ✅
       ↓
ROLE_USER
       ↓
GET requires USER or ADMIN
       ↓
Authorization ✅
       ↓
Controller
```

But:

```http
DELETE /api/employees/101
```

requires:

```text
ROLE_ADMIN
```

So:

```text id="2w8n5m"
ROLE_USER
   ↓
Required ROLE_ADMIN
   ↓
403 Forbidden
```

---

# 23. What if JWT Doesn't Contain Roles?

Suppose:

```json
{
  "sub": "rahul"
}
```

The token may authenticate successfully.

But there may be no `ROLE_USER` authority.

Then:

```text id="w4q8c6"
Authentication ✅
Authorization ❌
```

So:

> **Authentication and authorization are still separate even with JWT.**

This is a very important concept.

---

# 24. What if the JWT is Valid but Expired?

```text id="p1m7x4"
Valid signature ✅
        ↓
exp expired ❌
        ↓
Authentication failure
        ↓
401
```

A valid signature does not mean the token is currently valid.

---

# 25. What if Someone Changes the Role?

Original:

```json
"roles": ["ROLE_USER"]
```

Attacker changes:

```json
"roles": ["ROLE_ADMIN"]
```

but cannot generate a valid signature.

Then:

```text id="h6r2p8"
Signature mismatch
      ↓
JWT invalid
      ↓
Authentication failure
```

This is why the signature is essential.

---

# 26. Symmetric vs Asymmetric JWT Validation

Our tutorial currently uses:

```text id="r9x3m1"
HS256
```

with:

```text
Same Secret
```

for signing and verification.

Production OAuth2 environments often use:

```text id="g8m2q4"
RS256
```

or another asymmetric algorithm:

```text
Private Key
   ↓
Authorization Server signs

Public Key
   ↓
Resource Server verifies
```

Spring Resource Server supports configuring JWT validation from issuer metadata/JWK sets or a public key. ([Home][6])

---

# 27. Production Configuration

With a proper authorization server, your resource server often needs very little custom JWT code.

For example:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://auth.example.com
```

and:

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http
        .authorizeHttpRequests(auth ->
            auth.anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 ->
            oauth2.jwt(Customizer.withDefaults())
        );

    return http.build();
}
```

Spring Boot can use the issuer URI to discover the authorization server metadata and configure JWT validation. ([Home][6])

This is the **production-oriented approach** you should recognize in interviews.

---

# 28. Why is our custom JWT implementation still useful?

Because it teaches you:

```text
Login
 ↓
AuthenticationManager
 ↓
JwtEncoder
 ↓
JWT
```

and:

```text
Bearer Token
 ↓
JwtDecoder
 ↓
JwtAuthenticationProvider
 ↓
JwtAuthenticationConverter
 ↓
SecurityContext
```

After you understand those, Boot's auto-configuration isn't magic anymore.

---

# 29. JWT Validation vs JWT Generation

Memorize this:

```text
GENERATION

Authentication
      ↓
JwtEncoder
      ↓
JWT
```

```text
VALIDATION

JWT
      ↓
JwtDecoder
      ↓
JwtAuthenticationProvider
      ↓
Authentication
```

Or:

```text id="f5k8q1"
Generate → JwtEncoder
Validate → JwtDecoder
```

This is one of the simplest interview answers you can give.

---

# 30. Interview Questions

### What filter processes JWT bearer tokens?

> `BearerTokenAuthenticationFilter`. ([Home][2])

### What does `JwtAuthenticationProvider` do?

> It authenticates JWT bearer tokens using a `JwtDecoder` and a `JwtAuthenticationConverter`. ([Home][3])

### What does `JwtDecoder` do?

> It decodes and validates the JWT. ([Home][4])

### How are JWT claims turned into authorities?

> `JwtAuthenticationProvider` uses `JwtAuthenticationConverter` to convert JWT claims into `GrantedAuthority` values. ([Home][3])

### What happens when JWT validation fails?

> Authentication fails, the security context is cleared, and the authentication entry point handles the failure, typically producing a `401 Unauthorized` bearer-token response. ([Home][1])

### What happens when the JWT is valid but the role is insufficient?

> Authentication succeeds, authorization fails, and the client typically receives `403 Forbidden`.

---

# 31. Best Practices

```text id="z4m7q9"
✅ Validate signature
✅ Validate expiration
✅ Validate issuer/audience when applicable
✅ Use HTTPS
✅ Keep JWT claims minimal
✅ Map roles/authorities explicitly
✅ Prefer asymmetric signing in suitable distributed architectures
✅ Keep signing keys outside source code
✅ Use short-lived access tokens
✅ Don't manually parse JWTs in controllers
✅ Let Spring Security Resource Server handle bearer authentication
```

---

# 📍 Where We Are

```text id="f1i9m3"
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
│
└── ⏭️ Chapter 15 — JWT Filter & SecurityContext
      ↓
      OncePerRequestFilter
      ↓
      Why custom JWT filters exist
      ↓
      JWT → Authentication
      ↓
      SecurityContext
      ↓
      Custom authentication flow
      ↓
      When to use custom filter vs Resource Server
```

Next we'll learn **`OncePerRequestFilter` and the JWT filter pattern**—including an important practical distinction: **when you should build a custom JWT filter and when you should NOT build one because Spring Security Resource Server already provides the required filter for you.**

[1]: https://docs.spring.io/spring-security/reference/7.0/servlet/oauth2/resource-server/index.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/7.0/api/java/org/springframework/security/oauth2/server/resource/web/authentication/BearerTokenAuthenticationFilter.html?utm_source=chatgpt.com "BearerTokenAuthenticationFilter (spring-security-docs 7.0.6 API)"
[3]: https://docs.spring.io/spring-security/reference/7.1-SNAPSHOT/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT :: Spring Security"
[4]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/JwtDecoder.html?utm_source=chatgpt.com "JwtDecoder (spring-security-docs 7.1.0 API)"
[5]: https://docs.spring.io/spring-security/reference/7.0/servlet/oauth2/resource-server/bearer-tokens.html?utm_source=chatgpt.com "OAuth 2.0 Bearer Tokens :: Spring Security"
[6]: https://docs.spring.io/spring-security/reference/servlet/oauth2/?utm_source=chatgpt.com "OAuth2 :: Spring Security"
