# Spring Security — Chapter 10: HTTP Basic Authentication

Now we're going to **implement a real authentication mechanism**.

Until now, we've learned the pieces:

```text
SecurityFilterChain
UserDetailsService
UserDetails
PasswordEncoder
AuthenticationManager
AuthenticationProvider
DaoAuthenticationProvider
Roles / Authorities
Method Security
```

Now we'll connect them using:

> **HTTP Basic Authentication**

Spring Security provides built-in support for username/password authentication through both **HTTP Basic** and **Form Login**. ([Home][1])

---

# 1. Why do we need HTTP Basic Authentication?

We have this protected endpoint:

```http
GET /api/employees
```

Our security configuration says:

```java
.authenticated()
```

But how does the client actually prove:

> "I am Rahul."

That's where an authentication mechanism comes in.

For this chapter:

```text
Client
  ↓
username + password
  ↓
HTTP Basic
  ↓
Spring Security
  ↓
Authentication
```

---

# 2. What is HTTP Basic Authentication?

HTTP Basic is an authentication mechanism where the client sends credentials using the HTTP `Authorization` header.

Conceptually:

```http
Authorization: Basic <base64(username:password)>
```

For example, the credentials:

```text
rahul:password123
```

are Base64-encoded and sent after `Basic`.

Spring Security's `BasicAuthenticationFilter` processes the Basic authentication header and, on success, places the resulting `Authentication` in the `SecurityContextHolder`. ([Home][2])

---

# 3. Important: Base64 is NOT Encryption

This is a very common interview question.

Suppose:

```text
rahul:password123
```

is encoded as:

```text
Base64
```

That does **not** make it secret.

Base64 can be decoded easily.

So:

```text
Basic Authentication
     +
HTTP
```

is **not secure by itself**.

You should use:

```text
HTTPS
```

so the credentials are protected while traveling over the network.

Spring's own documentation notes that Basic authentication sends credentials in a form that is undesirable without transport protection. ([Home][2])

---

# 4. How does Basic Authentication work?

Suppose the client requests:

```http
GET /api/employees
Authorization: Basic <credentials>
```

Flow:

```text
Client
   ↓
Authorization: Basic ...
   ↓
Spring Security Filter Chain
   ↓
BasicAuthenticationFilter
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
   ↓
Authorization
   ↓
Controller
```

This is the username/password architecture you've been learning.

---

# 5. Enable HTTP Basic

Our configuration becomes:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**")
                    .permitAll()

                .requestMatchers(
                    HttpMethod.GET,
                    "/api/employees/**"
                )
                    .hasAnyRole("USER", "ADMIN")

                .requestMatchers(
                    HttpMethod.POST,
                    "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .requestMatchers(
                    HttpMethod.PUT,
                    "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .requestMatchers(
                    HttpMethod.PATCH,
                    "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .requestMatchers(
                    HttpMethod.DELETE,
                    "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .anyRequest()
                    .denyAll()
            )

            .httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

The modern configuration uses:

```java
.httpBasic(Customizer.withDefaults())
```

to enable HTTP Basic authentication. Spring's current username/password documentation shows this configuration style. ([Home][1])

---

# 6. Complete Configuration

Let's put our authentication pieces together.

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    PasswordEncoder passwordEncoder() {

        return new BCryptPasswordEncoder();
    }

    @Bean
    UserDetailsService userDetailsService(
            PasswordEncoder passwordEncoder) {

        UserDetails user =
                User.withUsername("rahul")
                    .password(
                        passwordEncoder.encode(
                            "password123"
                        )
                    )
                    .roles("USER")
                    .build();

        UserDetails admin =
                User.withUsername("admin")
                    .password(
                        passwordEncoder.encode(
                            "admin123"
                        )
                    )
                    .roles("ADMIN")
                    .build();

        return new InMemoryUserDetailsManager(
                user,
                admin
        );
    }

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth

                .requestMatchers("/api/public/**")
                    .permitAll()

                .requestMatchers(
                        HttpMethod.GET,
                        "/api/employees/**"
                )
                    .hasAnyRole("USER", "ADMIN")

                .requestMatchers(
                        HttpMethod.POST,
                        "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .requestMatchers(
                        HttpMethod.DELETE,
                        "/api/employees/**"
                )
                    .hasRole("ADMIN")

                .anyRequest()
                    .authenticated()
            )

            .httpBasic(
                Customizer.withDefaults()
            );

        return http.build();
    }
}
```

For learning, this is enough to demonstrate the complete username/password flow.

---

# 7. What happens when Rahul calls the API?

Suppose Rahul sends:

```http
GET /api/employees
Authorization: Basic <rahul-credentials>
```

Spring's Basic authentication filter extracts the credentials and invokes the authentication infrastructure. On successful authentication, the resulting authentication is stored in the security context. ([Home][2])

Conceptually:

```text
Basic Header
     ↓
BasicAuthenticationFilter
     ↓
AuthenticationManager
     ↓
DaoAuthenticationProvider
     ↓
UserDetailsService
     ↓
UserDetails("rahul")
     ↓
PasswordEncoder.matches()
     ↓
SUCCESS
     ↓
Authentication
     ↓
SecurityContext
```

Then authorization checks:

```text
ROLE_USER
     ↓
GET /api/employees
     ↓
USER allowed ✅
```

Then:

```text
DispatcherServlet
     ↓
EmployeeController
     ↓
EmployeeService
     ↓
Repository
```

---

# 8. What if Rahul tries DELETE?

Rahul has:

```text
ROLE_USER
```

Request:

```http
DELETE /api/employees/101
```

Security rule:

```java
.hasRole("ADMIN")
```

Flow:

```text
Authentication
     ↓
Rahul = authenticated ✅
     ↓
Authorities = ROLE_USER
     ↓
Required = ROLE_ADMIN
     ↓
Mismatch ❌
     ↓
403 Forbidden
```

The controller isn't allowed to perform the operation.

---

# 9. What if Rahul sends the wrong password?

Suppose:

```text
Username = rahul
Password = wrongPassword
```

Flow:

```text
BasicAuthenticationFilter
        ↓
AuthenticationManager
        ↓
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
UserDetails
        ↓
PasswordEncoder.matches()
        ↓
FALSE
        ↓
Authentication Failure
```

The authentication doesn't succeed.

---

# 10. What happens if no credentials are sent?

Request:

```http
GET /api/employees
```

No `Authorization` header.

Security sees:

```text
Authentication required
```

For an HTTP Basic-protected resource, the authentication entry point can challenge the client, normally resulting in a `401 Unauthorized` response. Spring Security's Basic authentication support uses an `AuthenticationEntryPoint` for failed authentication. ([Home][2])

---

# 11. `401` vs `403` in Basic Authentication

This is now very easy to understand.

### No valid authentication

```text
No credentials
     ↓
401 Unauthorized
```

### Authentication succeeds but role is insufficient

```text
ROLE_USER
     ↓
ADMIN endpoint
     ↓
403 Forbidden
```

So:

```text
401 → Authentication failure
403 → Authorization failure
```

---

# 12. How does Postman work with Basic Auth?

In Postman, you can choose:

```text
Authorization
   ↓
Type: Basic Auth
```

Username:

```text
rahul
```

Password:

```text
password123
```

Postman generates the appropriate:

```http
Authorization: Basic ...
```

header.

Your Spring Security filter processes it.

---

# 13. How does curl work?

Example:

```bash
curl -u rahul:password123 \
     http://localhost:8080/api/employees
```

The client sends Basic credentials.

The server processes them through Spring Security.

---

# 14. Is Basic Authentication Stateless?

This needs careful understanding.

HTTP Basic itself sends credentials with each authenticated request rather than requiring a login session token in the way form login commonly does.

For a REST API, however, you should explicitly choose your session behavior rather than assuming every authentication mechanism automatically means "stateless."

For example:

```java
.sessionManagement(session ->
    session.sessionCreationPolicy(
        SessionCreationPolicy.STATELESS
    )
)
```

This tells Spring Security not to use the HTTP session to store the security context for authentication persistence.

That configuration becomes particularly important when we move to **JWT**.

For HTTP Basic learning, you can understand:

```text
Basic
 ↓
Credentials accompany requests
```

and later:

```text
JWT
 ↓
Bearer token accompanies requests
```

---

# 15. Basic Authentication vs Form Login

You've seen both in Spring Security.

## HTTP Basic

```text
Authorization: Basic ...
```

Common for:

* APIs
* Internal services
* Simple clients
* Testing

## Form Login

```text
Login HTML page
    ↓
Username + Password
```

Common for:

* Traditional web applications
* Browser-based applications with server-rendered pages

Spring Security supports both as username/password authentication mechanisms. ([Home][1])

For our REST-focused roadmap, Basic is useful as a learning bridge, but **JWT is the important mechanism we'll focus on next for stateless REST APIs**.

---

# 16. Why are we learning Basic before JWT?

Because JWT introduces another concept:

```text
Credential
     ↓
Token
```

If you jump directly into JWT, you can memorize:

```text
JWT Filter
JWT Token
Bearer Token
```

without understanding authentication.

Now you understand:

```text
Authentication
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

So JWT will become much easier.

---

# 17. Basic Authentication vs JWT

| Basic                                        | JWT                                     |
| -------------------------------------------- | --------------------------------------- |
| Credentials sent with requests               | Token sent with requests                |
| Simple                                       | More involved                           |
| Common for simple/internal APIs              | Common for stateless API authentication |
| Requires HTTPS                               | Requires HTTPS                          |
| No custom token generation                   | Token must be generated/validated       |
| Server authenticates credentials per request | Server validates token per request      |

The exact operational model can vary, but this is the practical distinction we need at your level.

---

# 18. Important Security Point

Never deploy Basic Authentication over plain HTTP.

Use:

```text
HTTPS
```

because Basic credentials are not encrypted by Base64.

The BasicAuthenticationFilter documentation explicitly warns that Basic authentication has security limitations and should be protected by transport security. ([Home][2])

---

# 19. Interview Questions

### What is HTTP Basic Authentication?

> A mechanism where the client sends a username and password in the HTTP `Authorization` header using the `Basic` scheme.

### Is Base64 encryption?

> No. Base64 is an encoding, not encryption.

### Which Spring Security filter processes Basic authentication?

> `BasicAuthenticationFilter`. ([Home][2])

### What happens after successful Basic authentication?

> Spring obtains an authenticated `Authentication` and stores it in the `SecurityContextHolder`. ([Home][2])

### What happens when credentials are invalid?

> Authentication fails and the request is rejected/challenged through Spring Security's authentication failure handling.

### Difference between Basic and JWT?

> Basic sends credentials using the Basic scheme, while JWT uses a signed token carried by the client and validated on subsequent requests.

### Is Basic Authentication suitable for production?

> It can be used in appropriate scenarios, but it must be protected with HTTPS and is often not the preferred mechanism for modern public REST APIs where stateless bearer-token authentication is more appropriate.

---

# 20. Best Practices

```text
✅ Always use HTTPS
✅ Use BCrypt/PasswordEncoder for stored passwords
✅ Don't log Authorization headers
✅ Don't store passwords in source code
✅ Use strong authentication rules
✅ Return 401 for authentication failures
✅ Return 403 for authorization failures
✅ Prefer stateless token authentication for many modern REST APIs
```

---

# Complete Authentication Picture

At this point you can explain:

```text
Client
  │
  │ Authorization: Basic ...
  ▼
Spring Security Filter Chain
  │
  ▼
BasicAuthenticationFilter
  │
  ▼
AuthenticationManager
  │
  ▼
ProviderManager
  │
  ▼
DaoAuthenticationProvider
  │
  ├───────────────┐
  ▼               ▼
UserDetailsService PasswordEncoder
  │               │
  ▼               ▼
UserDetails    matches()
  │               │
  └───────┬───────┘
          ▼
   Authentication
          ▼
   SecurityContext
          ▼
    Authorization
          ▼
   DispatcherServlet
          ▼
      Controller
```

That's a **very strong Spring Security interview explanation**.

---

# 📍 Where We Are

```text
Spring Security
│
├── ✅ Chapter 1 — Why Security
├── ✅ Chapter 2 — SecurityFilterChain
├── ✅ Chapter 3 — Authentication/SecurityContext
├── ✅ Chapter 4 — UserDetails/UserDetailsService
├── ✅ Chapter 5 — PasswordEncoder/BCrypt
├── ✅ Chapter 6 — AuthenticationManager/Provider
├── ✅ Chapter 7 — Roles/Authorities
├── ✅ Chapter 8 — URL Authorization
├── ✅ Chapter 9 — Method Security
├── ✅ Chapter 10 — HTTP Basic
│
└── ⏭️ Chapter 11 — REST Security & Stateless Authentication
             ↓
         Session vs Stateless
             ↓
         SessionCreationPolicy
             ↓
         Why JWT?
             ↓
         JWT architecture
             ↓
         Login + Token flow
```

**Next is the important transition to JWT:** we'll first understand **stateful vs stateless authentication**, because without that distinction, JWT tends to become a memorized implementation rather than something you actually understand.

[1]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html?utm_source=chatgpt.com "Username/Password Authentication"
[2]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/web/authentication/www/BasicAuthenticationFilter.html?utm_source=chatgpt.com "BasicAuthenticationFilter (spring-security-docs 7.1.0 API)"
