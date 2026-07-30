# Spring Security — Chapter 17: CORS

Now we cover **CORS**, which is very common when your Spring REST API is called from React, Angular, Vue, or another browser application.

This chapter is especially important because **CORS is a browser security rule**, not an authentication mechanism. Spring Security integrates with Spring MVC's CORS support, and CORS needs to be handled before security authentication in scenarios such as browser preflight requests. ([Home][1])

We'll use:

> **Why → What → How → Where → Internal Flow → Complete Code → Interview Questions → Best Practices**

---

# 1. Why do we need CORS?

Suppose your frontend runs here:

```text
http://localhost:3000
```

and your Spring API runs here:

```text
http://localhost:8080
```

They have different:

```text
scheme
host
port
```

So they are **different origins**.

Your React application makes:

```http
GET http://localhost:8080/api/employees
```

The browser may say:

```text
Blocked by CORS policy
```

even though your Spring endpoint works perfectly when called directly from Postman.

That's because **CORS is enforced by browsers**. Spring's CORS support controls which cross-origin browser requests are allowed. ([Home][1])

---

# 2. What is an Origin?

An origin is made from:

```text
scheme + host + port
```

For example:

```text
http://localhost:3000
```

and:

```text
http://localhost:8080
```

are different origins because the ports differ.

Similarly:

```text
https://example.com
```

and:

```text
http://example.com
```

are different origins because the schemes differ.

---

# 3. What does CORS mean?

CORS stands for:

> **Cross-Origin Resource Sharing**

It is a mechanism that allows a server to specify which cross-origin browser requests are permitted.

Think:

```text
Frontend Origin
      ↓
"Can I call this API?"
      ↓
Spring API
      ↓
CORS Policy
      ↓
Allowed / Rejected
```

---

# 4. CORS is NOT Authentication

This is very important.

CORS answers:

> **Which browser origins may make cross-origin requests?**

Authentication answers:

> **Who is the user?**

Authorization answers:

> **What is the user allowed to do?**

So:

```text
CORS
 ↓
Browser-origin policy

Authentication
 ↓
Identity

Authorization
 ↓
Permission
```

CORS does **not** replace Spring Security.

---

# 5. Postman vs Browser

This explains a common confusion.

You call:

```http
GET /api/employees
```

from Postman:

```text
✅ Works
```

But from React:

```text
❌ CORS error
```

Why?

Because CORS is fundamentally a **browser-enforced** mechanism. A non-browser client such as Postman does not enforce browser CORS rules in the same way. ([Home][1])

So:

> **"Works in Postman but fails in browser" is a classic CORS symptom.**

---

# 6. What is a Preflight Request?

This is one of the most important CORS concepts.

Suppose your React app wants to send:

```http
DELETE /api/employees/101
```

with:

```http
Authorization: Bearer ...
```

The browser may first send an `OPTIONS` request.

That's called a **preflight request**.

Conceptually:

```http
OPTIONS /api/employees/101
Origin: http://localhost:3000
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: Authorization
```

The browser is essentially asking:

> "Is this origin allowed to make this DELETE request with this header?"

---

# 7. Preflight Flow

```text
Browser
   ↓
OPTIONS /api/employees/101
   ↓
CORS processing
   ↓
Allowed?
 ┌───────┴───────┐
 YES             NO
  ↓               ↓
200-ish        Browser blocks
CORS headers   actual request
```

Spring Framework provides CORS support including handling of preflight requests. ([Home][2])

---

# 8. Why Must CORS Run Before Security?

This is a very important Spring Security point.

A preflight request is typically:

```http
OPTIONS /api/employees/101
```

and it may not contain the application's authentication credentials in the way a normal authenticated request does.

Spring's Security documentation explicitly says that CORS must be processed before Spring Security, because a preflight request can otherwise be rejected as unauthenticated before CORS has a chance to handle it. ([Home][1])

So conceptually:

```text
Browser
   ↓
CORS
   ↓
Spring Security
   ↓
DispatcherServlet
```

not:

```text
Browser
   ↓
Spring Security
   ↓
CORS
```

for the relevant preflight handling.

---

# 9. How do we configure CORS?

For Spring MVC/Spring Security, a common approach is to provide a:

```java
UrlBasedCorsConfigurationSource
```

Spring Security can automatically integrate CORS when such a configuration source is present. ([Home][1])

---

# 10. Complete CORS Configuration

For our employee API:

```java
@Bean
UrlBasedCorsConfigurationSource corsConfigurationSource() {

    CorsConfiguration configuration =
            new CorsConfiguration();

    configuration.setAllowedOrigins(
            List.of("http://localhost:3000")
    );

    configuration.setAllowedMethods(
            List.of(
                    "GET",
                    "POST",
                    "PUT",
                    "PATCH",
                    "DELETE",
                    "OPTIONS"
            )
    );

    configuration.setAllowedHeaders(
            List.of(
                    "Authorization",
                    "Content-Type"
            )
    );

    configuration.setExposedHeaders(
            List.of("Location")
    );

    configuration.setAllowCredentials(false);

    UrlBasedCorsConfigurationSource source =
            new UrlBasedCorsConfigurationSource();

    source.registerCorsConfiguration(
            "/**",
            configuration
    );

    return source;
}
```

Then enable CORS in Spring Security:

```java
http.cors(Customizer.withDefaults());
```

Spring Security's `CorsConfigurer` integrates the CORS configuration into the security filter chain. ([Home][3])

---

# 11. Complete Security Configuration

Putting the pieces together:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .cors(Customizer.withDefaults())

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
                        HttpMethod.DELETE,
                        "/api/employees/**"
                ).hasRole("ADMIN")

                .anyRequest().denyAll()
            )

            .oauth2ResourceServer(oauth2 ->
                oauth2.jwt(
                        Customizer.withDefaults()
                )
            );

        return http.build();
    }

    @Bean
    UrlBasedCorsConfigurationSource corsConfigurationSource() {

        CorsConfiguration configuration =
                new CorsConfiguration();

        configuration.setAllowedOrigins(
                List.of("http://localhost:3000")
        );

        configuration.setAllowedMethods(
                List.of(
                        "GET",
                        "POST",
                        "PUT",
                        "PATCH",
                        "DELETE",
                        "OPTIONS"
                )
        );

        configuration.setAllowedHeaders(
                List.of(
                        "Authorization",
                        "Content-Type"
                )
        );

        configuration.setAllowCredentials(false);

        UrlBasedCorsConfigurationSource source =
                new UrlBasedCorsConfigurationSource();

        source.registerCorsConfiguration(
                "/**",
                configuration
        );

        return source;
    }
}
```

---

# 12. What does `allowedOrigins` mean?

```java
configuration.setAllowedOrigins(
    List.of("http://localhost:3000")
);
```

It means:

> Allow browser requests originating from `http://localhost:3000`.

If your frontend is:

```text
http://localhost:5173
```

you would need to configure that origin instead.

Production example:

```java
List.of("https://app.example.com")
```

---

# 13. Don't Use `*` Carelessly

You may see:

```java
configuration.setAllowedOrigins(
    List.of("*")
);
```

That means:

> Allow any origin, subject to the rest of the CORS rules.

This is usually too broad for a production application.

Use an explicit allowlist whenever possible:

```text
https://app.example.com
https://admin.example.com
```

rather than:

```text
*
```

---

# 14. Credentials and `*`

This is especially important.

If you allow credentials such as cookies:

```java
configuration.setAllowCredentials(true);
```

you should not use a wildcard origin in the usual credentialed CORS configuration.

Instead specify concrete allowed origins.

For our JWT-in-`Authorization`-header design:

```java
configuration.setAllowCredentials(false);
```

is often appropriate because the browser isn't using a cross-origin session cookie for authentication.

---

# 15. `allowedMethods`

Example:

```java
configuration.setAllowedMethods(
    List.of(
        "GET",
        "POST",
        "PUT",
        "PATCH",
        "DELETE",
        "OPTIONS"
    )
);
```

This controls which cross-origin HTTP methods may be used.

---

# 16. `allowedHeaders`

Our React frontend might send:

```http
Authorization: Bearer eyJ...
Content-Type: application/json
```

So we allow:

```java
configuration.setAllowedHeaders(
    List.of(
        "Authorization",
        "Content-Type"
    )
);
```

If the browser's preflight asks for a header your CORS policy does not allow, the browser can reject the actual cross-origin request.

---

# 17. Exposed Headers

Suppose your API returns:

```http
Location: /api/employees/101
```

Browsers don't necessarily expose every response header to JavaScript automatically.

You can configure:

```java
configuration.setExposedHeaders(
    List.of("Location")
);
```

Then browser JavaScript can read that header.

For our Employee API, this can be useful for a `201 Created` response.

---

# 18. `@CrossOrigin`

You can also configure CORS at the controller or method level.

Example:

```java
@CrossOrigin(
    origins = "http://localhost:3000"
)
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {
}
```

or:

```java
@CrossOrigin(
    origins = "http://localhost:3000"
)
@GetMapping
public List<EmployeeResponse> getEmployees() {
    ...
}
```

Spring MVC supports `@CrossOrigin` at both type and method level. ([Home][4])

---

# 19. Global vs `@CrossOrigin`

For a single endpoint:

```java
@CrossOrigin
```

can be convenient.

For an entire application:

```text
Central CorsConfiguration
```

is usually easier to manage.

Think:

```text
One endpoint
    ↓
@CrossOrigin

Many endpoints
    ↓
Central CORS configuration
```

---

# 20. CORS Internal Flow

Let's trace a React request.

Frontend:

```text
http://localhost:3000
```

API:

```text
http://localhost:8080
```

React calls:

```http
GET /api/employees
Authorization: Bearer eyJ...
```

Browser may first do:

```text
OPTIONS /api/employees
```

Flow:

```text
Browser
   ↓
CORS Preflight
   ↓
CorsFilter / CORS processing
   ↓
Origin allowed?
   ↓
Method allowed?
   ↓
Headers allowed?
   ↓
YES
   ↓
Actual GET request
   ↓
Spring Security
   ↓
JWT validation
   ↓
Authorization
   ↓
Controller
```

Spring's `CorsFilter` handles preflight, simple, and actual CORS requests and updates the response with appropriate CORS headers based on the matching configuration. ([Home][5])

---

# 21. CORS + JWT

This combination is extremely common.

Browser sends:

```http
Authorization: Bearer eyJ...
Origin: https://app.example.com
```

The CORS configuration must allow:

```text
Origin
Authorization header
HTTP method
```

Then Spring Security handles:

```text
JWT authentication
```

So remember:

```text
CORS
 ↓
"Can this browser origin call me?"

JWT
 ↓
"Who is making this authenticated request?"

Authorization
 ↓
"Can they perform this operation?"
```

These are three different concerns.

---

# 22. Common Mistake: "I disabled CORS"

Some developers do:

```java
http.cors(cors -> cors.disable());
```

and think:

> "CORS is disabled, so browser requests should work."

That's backwards.

Spring Security's documentation explicitly warns that disabling its CORS support does not remove the browser's CORS enforcement. Without a proper CORS configuration, browser cross-origin requests can still fail. ([Home][1])

So:

```text
Disable Spring Security CORS support
≠
Disable browser CORS protection
```

---

# 23. Common Mistake: CORS vs CSRF

These are **not the same thing**.

### CORS

Controls:

```text
Which browser origins may make cross-origin requests?
```

### CSRF

Protects against a different attack involving unwanted requests made with automatically attached credentials, especially cookies.

So:

```text
CORS ≠ CSRF
```

We'll study CSRF in the next chapter.

---

# 24. Common Mistake: CORS Error Means Backend Is Broken

Sometimes the API actually responds correctly, but the browser refuses to expose the response to JavaScript because the required CORS headers are missing.

So:

```text
Postman ✅
Browser ❌
```

doesn't necessarily mean your controller or database code is broken.

Check:

```text
Origin
Allowed Origin
Allowed Methods
Allowed Headers
Preflight
CORS response headers
```

---

# 25. Common Mistake: Allowing Everything

Avoid production configurations like:

```java
allowedOrigins("*")
allowedMethods("*")
allowedHeaders("*")
```

unless you have a specific reason and understand the security implications.

Prefer a deliberate policy:

```text
Origins
   ↓
Known frontend applications

Methods
   ↓
Methods your API actually needs

Headers
   ↓
Headers your clients actually use
```

---

# 26. CORS with Credentials

Suppose instead of Authorization headers, you're using a cookie-based authentication mechanism.

Then:

```java
configuration.setAllowCredentials(true);
```

may be necessary.

The browser's credentialed CORS behavior is different and requires careful origin configuration.

For our JWT bearer-header architecture, we generally don't need cross-origin cookies for authentication.

---

# 27. Interview Questions

### What is CORS?

> CORS is a browser-enforced mechanism that allows a server to specify which cross-origin browser requests are permitted.

### Why does Postman work but React fail?

> Postman is not enforcing browser CORS rules; the browser is.

### What is a preflight request?

> An `OPTIONS` request a browser can send before certain cross-origin requests to determine whether the actual request is permitted.

### Why should CORS be processed before Spring Security?

> Preflight requests may not contain the credentials needed by the security layer, so handling CORS first prevents legitimate preflight requests from being rejected as unauthenticated. ([Home][1])

### Where can Spring configure CORS?

> Through Spring MVC configuration, `CorsConfigurationSource`, `CorsFilter`, or `@CrossOrigin`, depending on the application's needs. ([Home][1])

### Does CORS provide authentication?

> No. CORS controls browser cross-origin access; authentication identifies the caller.

### CORS vs CSRF?

> CORS controls cross-origin browser access, while CSRF protects against unwanted requests that exploit automatically supplied credentials such as cookies.

---

# 28. Best Practices

```text
✅ Allow only trusted frontend origins
✅ Allow only necessary HTTP methods
✅ Allow only required headers
✅ Handle OPTIONS/preflight correctly
✅ Configure CORS before security processing
✅ Keep CORS and authentication concepts separate
✅ Don't use '*' casually
✅ Don't expose unnecessary response headers
```

For our JWT REST API:

```text
Frontend
https://app.example.com
        ↓
CORS allows origin
        ↓
Authorization: Bearer JWT
        ↓
Spring Security validates JWT
        ↓
Authorization checks role
        ↓
Controller
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
├── ✅ Chapter 13 — JWT Login & Generation
├── ✅ Chapter 14 — JWT Validation
├── ✅ Chapter 15 — OncePerRequestFilter
├── ✅ Chapter 16 — 401 / 403 / Security Errors
├── ✅ Chapter 17 — CORS
│
└── ⏭️ Chapter 18 — CSRF ⭐⭐⭐⭐
      ↓
      What is CSRF?
      ↓
      Why cookies are vulnerable
      ↓
      CSRF token
      ↓
      Why stateless JWT APIs are different
      ↓
      When disabling CSRF is appropriate
      ↓
      Common interview questions
```

Next is **CSRF**, and this is where we will clear up one of the most common Spring Security misconceptions:

> **"We use JWT, so CSRF is always unnecessary."**

That's not universally true; it depends heavily on **where and how the authentication credential is stored and sent**. ([Home][6])

[1]: https://docs.spring.io/spring-security/reference/7.0/servlet/integrations/cors.html?utm_source=chatgpt.com "CORS :: Spring Security"
[2]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/cors/package-summary.html?utm_source=chatgpt.com "org.springframework.web.cors (Spring Framework 7.0.8 API)"
[3]: https://docs.spring.io/spring-security/site/docs/7.0.x/api/org/springframework/security/config/annotation/web/configurers/CorsConfigurer.html?utm_source=chatgpt.com "CorsConfigurer (spring-security-docs 7.0.0 API)"
[4]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html?utm_source=chatgpt.com "CrossOrigin (Spring Framework 7.0.8 API)"
[5]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/filter/CorsFilter.html?utm_source=chatgpt.com "CorsFilter (Spring Framework 7.0.8 API)"
[6]: https://docs.spring.io/spring-security/reference/7.0/servlet/exploits/csrf.html?utm_source=chatgpt.com "Cross Site Request Forgery (CSRF) :: Spring Security"
