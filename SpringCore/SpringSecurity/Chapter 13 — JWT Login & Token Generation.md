# Spring Security — Chapter 13: JWT Login & Token Generation

Now we will **actually implement JWT login**.

Until now we learned:

```text
SecurityFilterChain
UserDetailsService
PasswordEncoder
AuthenticationManager
AuthenticationProvider
Roles / Authorities
Method Security
Stateful vs Stateless
JWT structure
```

Now we connect them:

```text
POST /api/auth/login
        ↓
Username + Password
        ↓
AuthenticationManager
        ↓
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
PasswordEncoder
        ↓
Authentication SUCCESS
        ↓
JWT Encoder
        ↓
JWT
        ↓
Response
```

Spring Security explicitly supports publishing an `AuthenticationManager` for custom REST login endpoints, which is exactly the pattern we'll use here. ([Home][1])

For token creation, Spring Security provides the `JwtEncoder` abstraction, with `NimbusJwtEncoder` as an implementation. Current Spring Security 7 APIs support configuring it with a secret key, RSA key pair, or EC key pair. ([Home][2])

---

# 1. Why do we need a Login Endpoint?

We want a client to send:

```http
POST /api/auth/login
Content-Type: application/json
```

```json
{
  "username": "rahul",
  "password": "password123"
}
```

The server should:

```text
authenticate Rahul
      ↓
generate JWT
      ↓
return JWT
```

Then the client can call:

```http
GET /api/employees
Authorization: Bearer <JWT>
```

---

# 2. Important Architecture

Our application is temporarily doing two jobs:

```text
                    Employee Application
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       Authentication              Resource Server
       /api/auth/login             /api/employees/**
              │                         │
              ▼                         ▼
       Username/Password                JWT
              │                         │
              └────────────┬────────────┘
                           ▼
                    Protected API
```

In a larger OAuth2 system, token issuance is often handled by a dedicated Authorization Server and the API acts as a Resource Server. Spring Security has separate support for those roles. ([Home][3])

For learning, keeping them in one application makes the flow much easier to understand.

---

# 3. Login DTO

Create:

```java
package com.practice.employeeapi.dto;

import jakarta.validation.constraints.NotBlank;

public record LoginRequest(
        @NotBlank String username,
        @NotBlank String password
) {
}
```

---

# 4. JWT Response DTO

```java
package com.practice.employeeapi.dto;

public record AuthResponse(
        String accessToken,
        String tokenType,
        long expiresIn
) {
}
```

Example response:

```json
{
  "accessToken": "eyJhbGciOi...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

# 5. Publish `AuthenticationManager`

We already learned why `AuthenticationManager` exists.

For a REST login endpoint, we need to be able to call it ourselves.

```java
@Bean
AuthenticationManager authenticationManager(
        UserDetailsService userDetailsService,
        PasswordEncoder passwordEncoder) {

    DaoAuthenticationProvider provider =
            new DaoAuthenticationProvider(userDetailsService);

    provider.setPasswordEncoder(passwordEncoder);

    return new ProviderManager(provider);
}
```

Spring's current documentation shows this pattern for publishing an `AuthenticationManager` for custom REST authentication. ([Home][1])

The flow is:

```text
Controller
    ↓
AuthenticationManager
    ↓
DaoAuthenticationProvider
    ↓
UserDetailsService
    ↓
PasswordEncoder
```

---

# 6. Configure the JWT Secret

For this learning implementation we'll use an HMAC secret.

Put the secret in configuration, **not source code**.

### `application.properties`

```properties
app.jwt.secret=${JWT_SECRET}
app.jwt.expiration=900
```

Then set an environment variable.

For example:

```bash
export JWT_SECRET='replace-this-with-a-long-random-secret'
```

Do not commit the real secret to Git.

For production, secret management should be handled through your deployment/secret-management system.

---

# 7. Create the Secret Key

```java
package com.practice.employeeapi.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

@Configuration
public class JwtConfig {

    @Bean
    SecretKey jwtSecretKey(
            @Value("${app.jwt.secret}") String secret) {

        return new SecretKeySpec(
                secret.getBytes(StandardCharsets.UTF_8),
                "HmacSHA256"
        );
    }
}
```

For real production systems, use a cryptographically strong secret with appropriate length and manage it securely rather than using an easy-to-guess string.

---

# 8. Create `JwtEncoder`

Spring Security's `NimbusJwtEncoder` supports constructing an encoder from a `SecretKey` in current Spring Security 7 APIs. ([Home][2])

```java
package com.practice.employeeapi.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;

import javax.crypto.SecretKey;

@Configuration
public class JwtEncoderConfig {

    @Bean
    JwtEncoder jwtEncoder(SecretKey jwtSecretKey) {

        return NimbusJwtEncoder
                .withSecretKey(jwtSecretKey)
                .build();
    }
}
```

Now Spring has:

```text
JwtEncoder
    ↓
Create signed JWT
```

The `JwtEncoder` API encodes JWT claims into the compact JWT representation. ([Home][4])

---

# 9. JWT Service

Now create a dedicated service.

```java
package com.practice.employeeapi.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.jose.jws.MacAlgorithm;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.List;

@Service
public class JwtService {

    private final JwtEncoder jwtEncoder;
    private final long expirationSeconds;

    public JwtService(
            JwtEncoder jwtEncoder,
            @Value("${app.jwt.expiration}") long expirationSeconds) {

        this.jwtEncoder = jwtEncoder;
        this.expirationSeconds = expirationSeconds;
    }

    public String generateToken(
            Authentication authentication) {

        Instant now = Instant.now();

        List<String> roles =
                authentication.getAuthorities()
                        .stream()
                        .map(authority ->
                                authority.getAuthority())
                        .toList();

        JwtClaimsSet claims =
                JwtClaimsSet.builder()

                        .subject(
                                authentication.getName())

                        .issuedAt(now)

                        .expiresAt(
                                now.plus(
                                    expirationSeconds,
                                    ChronoUnit.SECONDS
                                ))

                        .claim("roles", roles)

                        .issuer("employee-api")

                        .build();

        return jwtEncoder
                .encode(
                    JwtEncoderParameters
                        .from(
                            org.springframework.security.oauth2.jose.jws.JwsHeader
                                .with(MacAlgorithm.HS256)
                                .build(),
                            claims
                        )
                )
                .getTokenValue();
    }
}
```

The important part is:

```text
Authentication
    ↓
username
roles
    ↓
JwtClaimsSet
    ↓
JwtEncoder
    ↓
JWT String
```

---

# 10. What claims are we putting into the token?

Our token contains:

```json
{
  "sub": "rahul",
  "roles": [
    "ROLE_USER"
  ],
  "iss": "employee-api",
  "iat": 1753862400,
  "exp": 1753863300
}
```

Meaning:

```text
sub
 ↓
Who?

roles
 ↓
What authorities?

iss
 ↓
Who issued it?

iat
 ↓
When issued?

exp
 ↓
When does it expire?
```

Notice:

**No password.**

---

# 11. Login Controller

Now create:

```java
package com.practice.employeeapi.controller;

import com.practice.employeeapi.dto.AuthResponse;
import com.practice.employeeapi.dto.LoginRequest;
import com.practice.employeeapi.service.JwtService;
import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;

    public AuthController(
            AuthenticationManager authenticationManager,
            JwtService jwtService) {

        this.authenticationManager =
                authenticationManager;

        this.jwtService = jwtService;
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(
            @Valid @RequestBody LoginRequest request) {

        Authentication authenticationRequest =
                UsernamePasswordAuthenticationToken
                        .unauthenticated(
                                request.username(),
                                request.password()
                        );

        Authentication authentication =
                authenticationManager.authenticate(
                        authenticationRequest
                );

        String token =
                jwtService.generateToken(
                        authentication
                );

        return ResponseEntity.ok(
                new AuthResponse(
                        token,
                        "Bearer",
                        900
                )
        );
    }
}
```

This follows the same REST authentication pattern shown in Spring Security's current documentation: create an unauthenticated `UsernamePasswordAuthenticationToken`, pass it to the `AuthenticationManager`, and continue with the successful authentication result. ([Home][1])

---

# 12. Complete Login Flow

Now you should understand every step.

Request:

```http
POST /api/auth/login
```

```json
{
  "username": "rahul",
  "password": "password123"
}
```

Flow:

```text
Client
   ↓
AuthController
   ↓
UsernamePasswordAuthenticationToken
   ↓
AuthenticationManager
   ↓
ProviderManager
   ↓
DaoAuthenticationProvider
   ↓
UserDetailsService
   ↓
UserDetails
   ↓
PasswordEncoder.matches()
   ↓
Authentication SUCCESS
   ↓
JwtService
   ↓
JwtClaimsSet
   ↓
JwtEncoder
   ↓
Signed JWT
   ↓
AuthResponse
   ↓
Client
```

That is the complete login side.

---

# 13. What happens if password is wrong?

Suppose:

```json
{
  "username": "rahul",
  "password": "wrong"
}
```

Then:

```text
AuthenticationManager
       ↓
DaoAuthenticationProvider
       ↓
PasswordEncoder
       ↓
matches() = false
       ↓
AuthenticationException
```

The JWT is **not generated**.

That's important:

> **We generate the token only after authentication succeeds.**

---

# 14. Security Configuration

Now our login endpoint must be public.

```java
package com.practice.employeeapi.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

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
            );

        return http.build();
    }

    @Bean
    AuthenticationManager authenticationManager(
            UserDetailsService userDetailsService,
            PasswordEncoder passwordEncoder) {

        DaoAuthenticationProvider provider =
                new DaoAuthenticationProvider(
                        userDetailsService
                );

        provider.setPasswordEncoder(
                passwordEncoder
        );

        return new ProviderManager(provider);
    }

    @Bean
    PasswordEncoder passwordEncoder() {

        return PasswordEncoderFactories
                .createDelegatingPasswordEncoder();
    }
}
```

One correction from our earlier simplified examples: for current Spring Security, the built-in password factory is a good production-oriented default when you don't have a reason to choose a single encoder explicitly, because it supports multiple encodings and migration through identifiers. Spring's current documentation uses `PasswordEncoderFactories.createDelegatingPasswordEncoder()` in its examples for a published `AuthenticationManager`. ([Home][1])

---

# 15. We Still Need JWT Validation

This is extremely important.

We've created:

```text
JWT Generation ✅
```

But our API doesn't yet know how to process:

```http
Authorization: Bearer <JWT>
```

We need:

```text
JWT Decoder
+
Resource Server
```

That's the next chapter.

Spring Security's Resource Server support is designed to validate JWT bearer tokens, with a `JwtDecoder` responsible for validation. ([Home][5])

For our HMAC learning implementation, we can configure:

```java
@Bean
JwtDecoder jwtDecoder(SecretKey jwtSecretKey) {

    return NimbusJwtDecoder
            .withSecretKey(jwtSecretKey)
            .build();
}
```

Current Spring Security 7 APIs provide `NimbusJwtDecoder.withSecretKey(...)` specifically for validating a JWS with a shared secret key. ([Home][6])

Then:

```java
http.oauth2ResourceServer(
    oauth2 -> oauth2.jwt(
        Customizer.withDefaults()
    )
);
```

will make Spring Security process bearer JWTs.

---

# 16. But Why Can't We Add That Yet?

Because we need to understand what happens here:

```text
Authorization: Bearer <JWT>
```

Spring needs to:

```text
Extract Token
    ↓
Decode Token
    ↓
Validate Signature
    ↓
Validate exp
    ↓
Create Authentication
    ↓
Put it into SecurityContext
    ↓
Authorize Request
```

That's **Chapter 14: JWT Validation and the Bearer Token Filter**.

That's where the second half of JWT security becomes clear.

---

# 17. Login vs Protected Request

This distinction is very important.

## Login

```text
POST /api/auth/login
      ↓
Username + Password
      ↓
AuthenticationManager
      ↓
JWT generated
```

## Protected API

```text
GET /api/employees
Authorization: Bearer JWT
      ↓
BearerTokenAuthenticationFilter
      ↓
JwtDecoder
      ↓
JWT Validation
      ↓
Authentication
      ↓
Authorization
      ↓
Controller
```

So JWT has **two sides**:

```text
1. Token Generation
2. Token Validation
```

We just completed side 1.

---

# 18. What is `JwtEncoder`?

Interview answer:

> `JwtEncoder` is Spring Security's abstraction for encoding JWT claims into a JWT representation; `NimbusJwtEncoder` is the standard implementation supplied by Spring Security. ([Home][4])

---

# 19. What is `JwtDecoder`?

Interview answer:

> `JwtDecoder` is the abstraction responsible for decoding and validating a JWT into a Spring Security `Jwt` object. ([Home][6])

So:

```text
JwtEncoder
    ↓
Create JWT

JwtDecoder
    ↓
Validate JWT
```

Memorize this pair.

---

# 20. Why `AuthenticationManager` at Login?

Because we don't want the controller to manually do:

```text
Find User
Compare Password
Check Account Status
Build Authentication
```

Instead:

```text
Controller
   ↓
AuthenticationManager
   ↓
DaoAuthenticationProvider
   ↓
UserDetailsService
   ↓
PasswordEncoder
```

The official Spring Security REST-login example follows this architecture. ([Home][1])

The controller's job is essentially:

```text
Receive credentials
      ↓
Authenticate
      ↓
Generate token
      ↓
Return token
```

---

# 21. Interview Questions

### Why do we publish an `AuthenticationManager` bean?

> To allow application code, such as a REST login controller, to invoke Spring Security's authentication infrastructure directly. ([Home][1])

### What is the login flow?

```text
Username/password
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
JWT Encoder
    ↓
JWT
```

### What is `JwtEncoder`?

> Component used to create/encode a JWT from claims. ([Home][4])

### What is `JwtDecoder`?

> Component used to decode and validate a JWT. ([Home][6])

### Does JWT replace AuthenticationManager?

**Not exactly.**

For our custom login endpoint:

```text
AuthenticationManager
→ authenticates username/password
```

Then:

```text
JwtEncoder
→ creates token
```

For subsequent bearer-token requests:

```text
JwtAuthenticationProvider
→ authenticates the JWT
```

Those are different authentication steps.

---

# 22. Best Practices

```text
✅ Authenticate before generating token
✅ Never put password in JWT
✅ Use an expiration time
✅ Keep claims minimal
✅ Keep signing secret outside source code
✅ Use HTTPS
✅ Use a dedicated JWT service
✅ Keep login controller thin
✅ Don't manually compare passwords
✅ Don't manually parse JWTs in controllers
```

For production OAuth2 architectures, a dedicated Authorization Server is often preferable to writing your own token-issuing protocol endpoint; the custom login implementation here is primarily a learning bridge for understanding the mechanics. Spring Security provides dedicated Authorization Server capabilities for standardized OAuth2/OIDC token issuance. ([Home][3])

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
├── ✅ Chapter 13 — JWT Login & Generation
│
└── ⏭️ Chapter 14 — JWT Validation ⭐⭐⭐⭐⭐
      ↓
      Authorization: Bearer <JWT>
      ↓
      BearerTokenAuthenticationFilter
      ↓
      JwtDecoder
      ↓
      JwtAuthenticationProvider
      ↓
      JwtAuthenticationToken
      ↓
      SecurityContext
      ↓
      Roles / Authorities
      ↓
      Controller
```

Next we'll complete the JWT flow by implementing **Bearer-token validation**, including `JwtDecoder`, `oauth2ResourceServer().jwt()`, how the filter works, and how the JWT's `roles` claim becomes Spring Security authorities.

[1]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html "Username/Password Authentication :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/NimbusJwtEncoder.html "NimbusJwtEncoder (spring-security-docs 7.1.0 API)"
[3]: https://docs.spring.io/spring-security/reference/servlet/oauth2/authorization-server/core-model-components.html?utm_source=chatgpt.com "Core Model / Components :: Spring Security"
[4]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/JwtEncoder.html "JwtEncoder (spring-security-docs 7.1.0 API)"
[5]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT"
[6]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/jwt/NimbusJwtDecoder.html?utm_source=chatgpt.com "NimbusJwtDecoder (spring-security-docs 7.1.0 API)"
