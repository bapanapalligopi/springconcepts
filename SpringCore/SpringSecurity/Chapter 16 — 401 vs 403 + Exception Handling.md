# Spring Security — Chapter 16: 401 vs 403 & Security Exception Handling

We now know how authentication and authorization work.

The next practical question is:

> **What exactly happens when authentication or authorization fails, and how do we return clean JSON errors from a REST API?**

This chapter is important because **Spring Security errors are handled before your controller's `@RestControllerAdvice` normally gets a chance to handle them**. Spring Security also deliberately avoids putting detailed rejection reasons into the response body by default, so logging at DEBUG/TRACE is useful when diagnosing 401/403 responses. ([Home][1])

---

# 1. First: 401 vs 403

Keep this distinction absolutely clear.

### 401 — Authentication problem

The server cannot establish a valid authenticated identity.

Examples:

```text
No token
Malformed token
Expired token
Invalid signature
Invalid authentication credentials
```

Flow:

```text
Request
  ↓
Authentication
  ↓
FAIL ❌
  ↓
401 Unauthorized
```

For bearer-token resource servers, invalid/expired bearer tokens are expected to result in `401 Unauthorized`. ([Home][2])

---

### 403 — Authorization problem

The user **is authenticated**, but doesn't have enough authority.

Example:

```text
Rahul
ROLE_USER
    ↓
DELETE /api/employees/101
    ↓
Requires ROLE_ADMIN
```

Result:

```text
Authenticated ✅
Authorization ❌
        ↓
403 Forbidden
```

The key rule:

```text
401 → Who are you?
403 → I know who you are, but you're not allowed.
```

---

# 2. Why isn't `@RestControllerAdvice` enough?

Suppose this request arrives:

```http
GET /api/employees
Authorization: Bearer invalid-token
```

The request may fail inside:

```text
Spring Security Filter Chain
```

**before it reaches `DispatcherServlet` and your controller.**

So this:

```java
@RestControllerAdvice
```

is excellent for exceptions from your application/controller/service flow, but it isn't the primary mechanism for security failures occurring in the filter chain.

That's why Spring Security has dedicated components.

---

# 3. The Two Important Components

Remember these:

```text
AuthenticationEntryPoint
        ↓
Authentication failure

AccessDeniedHandler
        ↓
Authorization failure
```

Or:

```text
401
 ↓
AuthenticationEntryPoint

403
 ↓
AccessDeniedHandler
```

---

# 4. What is `AuthenticationEntryPoint`?

It handles the case where an unauthenticated client tries to access a protected resource.

Conceptually:

```text
Request
 ↓
Authentication required
 ↓
Not authenticated
 ↓
AuthenticationEntryPoint
 ↓
401
```

For REST APIs, we can customize it to return JSON.

---

# 5. Custom `AuthenticationEntryPoint`

```java
@Component
public class RestAuthenticationEntryPoint
        implements AuthenticationEntryPoint {

    private final ObjectMapper objectMapper;

    public RestAuthenticationEntryPoint(
            ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException authException)
            throws IOException {

        response.setStatus(
                HttpServletResponse.SC_UNAUTHORIZED);

        response.setContentType(
                MediaType.APPLICATION_JSON_VALUE);

        ErrorResponse error = new ErrorResponse(
                401,
                "Authentication required",
                request.getRequestURI()
        );

        response.getWriter().write(
                objectMapper.writeValueAsString(error)
        );
    }
}
```

Now instead of a generic response, your API can return:

```json
{
  "status": 401,
  "message": "Authentication required",
  "path": "/api/employees"
}
```

---

# 6. What is `AccessDeniedHandler`?

Now suppose Rahul is already authenticated:

```text
ROLE_USER
```

but tries:

```http
DELETE /api/employees/101
```

and the endpoint requires:

```text
ROLE_ADMIN
```

The request reaches the authorization decision and fails.

That goes to:

```text
AccessDeniedHandler
```

Flow:

```text
Request
  ↓
Authentication ✅
  ↓
Authorization ❌
  ↓
AccessDeniedHandler
  ↓
403
```

---

# 7. Custom `AccessDeniedHandler`

```java
@Component
public class RestAccessDeniedHandler
        implements AccessDeniedHandler {

    private final ObjectMapper objectMapper;

    public RestAccessDeniedHandler(
            ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException accessDeniedException)
            throws IOException {

        response.setStatus(
                HttpServletResponse.SC_FORBIDDEN);

        response.setContentType(
                MediaType.APPLICATION_JSON_VALUE);

        ErrorResponse error = new ErrorResponse(
                403,
                "Access denied",
                request.getRequestURI()
        );

        response.getWriter().write(
                objectMapper.writeValueAsString(error)
        );
    }
}
```

Response:

```json
{
  "status": 403,
  "message": "Access denied",
  "path": "/api/employees/101"
}
```

---

# 8. Configure Both

Now connect them to `HttpSecurity`.

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http,
        RestAuthenticationEntryPoint entryPoint,
        RestAccessDeniedHandler accessDeniedHandler)
        throws Exception {

    http
        .exceptionHandling(exceptions ->
            exceptions
                .authenticationEntryPoint(entryPoint)
                .accessDeniedHandler(
                        accessDeniedHandler
                )
        )

        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**")
                .permitAll()

            .requestMatchers(
                HttpMethod.GET,
                "/api/employees/**"
            ).hasAnyRole("USER", "ADMIN")

            .requestMatchers(
                HttpMethod.DELETE,
                "/api/employees/**"
            ).hasRole("ADMIN")

            .anyRequest()
                .denyAll()
        );

    return http.build();
}
```

Now your security failures have explicit REST responses.

---

# 9. JWT Invalid Token Flow

Suppose:

```http
GET /api/employees
Authorization: Bearer abc.invalid.token
```

Flow:

```text
Client
  ↓
BearerTokenAuthenticationFilter
  ↓
JwtDecoder
  ↓
Validation fails
  ↓
Authentication failure
  ↓
AuthenticationEntryPoint
  ↓
401
```

Spring Security's Resource Server support handles bearer-token authentication failures through the authentication entry-point mechanism. ([Home][2])

---

# 10. JWT Valid but Wrong Role

Suppose:

```text
JWT
 ↓
sub = rahul
ROLE_USER
```

Request:

```http
DELETE /api/employees/101
```

Required:

```text
ROLE_ADMIN
```

Flow:

```text
BearerTokenAuthenticationFilter
        ↓
JWT valid ✅
        ↓
Authentication created ✅
        ↓
AuthorizationFilter
        ↓
ROLE_ADMIN required
        ↓
ROLE_USER found
        ↓
Denied
        ↓
AccessDeniedHandler
        ↓
403
```

The request is authenticated but forbidden. Spring Security's request authorization infrastructure performs the authorization decision using the authentication in the `SecurityContext`. ([Home][3])

---

# 11. What Does `AuthorizationFilter` Do?

We have already seen:

```text
SecurityFilterChain
```

Now another useful internal concept is:

```text
AuthorizationFilter
```

It makes the authorization decision for the current request.

Conceptually:

```text
SecurityContext
      ↓
Authentication
      ↓
AuthorizationFilter
      ↓
AuthorizationManager
      ↓
Allow / Deny
```

Spring Security's request-authorization architecture describes `AuthorizationFilter` obtaining the current `Authentication` from `SecurityContextHolder` and delegating authorization decisions to the configured authorization machinery. ([Home][3])

---

# 12. AuthenticationEntryPoint vs AccessDeniedHandler

This should be automatic in interviews:

| Situation                                 | Component                  | Typical HTTP status |
| ----------------------------------------- | -------------------------- | ------------------: |
| No/invalid authentication                 | `AuthenticationEntryPoint` |                 401 |
| Authenticated but insufficient permission | `AccessDeniedHandler`      |                 403 |

---

# 13. Why do Resource Server Errors Sometimes Look Different?

Spring Security Resource Server has built-in bearer-token error handling.

For example:

```text
invalid_token
```

is associated with an HTTP `401 Unauthorized` response in the bearer-token specification flow. ([Home][2])

A production REST API can customize the failure response, but the underlying security semantics remain:

```text
Invalid authentication
→ 401

Insufficient authority
→ 403
```

---

# 14. What About `@ExceptionHandler`?

Suppose your service throws:

```java
throw new EmployeeNotFoundException(id);
```

That happens **after authentication and authorization succeeded**.

Then:

```text
Controller
  ↓
Service
  ↓
Exception
  ↓
@RestControllerAdvice
```

So there are two different exception-handling zones:

```text
Spring Security layer
    ↓
AuthenticationEntryPoint
AccessDeniedHandler

Application layer
    ↓
@RestControllerAdvice
@ExceptionHandler
```

This distinction is very important.

---

# 15. Complete Request Failure Map

```text
                    HTTP Request
                         │
                         ▼
                Spring Security
                  Filter Chain
                         │
             ┌───────────┴───────────┐
             │                       │
       Authentication           Authentication
          failure?                 success
             │                       │
             ▼                       ▼
      EntryPoint (401)          Authorization
                                     │
                              ┌──────┴──────┐
                              │             │
                            allow         deny
                              │             │
                              ▼             ▼
                        DispatcherServlet  AccessDeniedHandler
                              │             │
                              ▼             ▼
                         Controller         403
                              │
                              ▼
                         Service
                              │
                              ▼
                    @RestControllerAdvice
```

That's an excellent mental model.

---

# 16. Security Error Response DTO

We can use the same error model from our REST API:

```java
public record ErrorResponse(
        int status,
        String message,
        String path
) {
}
```

401:

```json
{
  "status": 401,
  "message": "Authentication required",
  "path": "/api/employees"
}
```

403:

```json
{
  "status": 403,
  "message": "Access denied",
  "path": "/api/employees/101"
}
```

This gives frontend clients a predictable response format.

---

# 17. Do Not Return Sensitive Security Details

Avoid:

```json
{
  "message": "JWT signature failed because secret XYZ..."
}
```

or:

```json
{
  "message": "User exists but password was wrong"
}
```

Keep public errors generic enough that they don't unnecessarily reveal security-sensitive details.

Spring Security intentionally avoids adding detailed reasons for rejected security requests to the response body by default, while providing DEBUG/TRACE logging for diagnosis. ([Home][1])

---

# 18. How to Debug 401/403

When you see:

```text
401
```

check:

```text
Token present?
Token format?
Token expired?
Signature valid?
Issuer/audience correct?
Authentication created?
```

When you see:

```text
403
```

check:

```text
Authentication exists?
Which authorities?
Which role is required?
Does ROLE_ prefix match?
Did method security deny it?
```

Spring Security recommends DEBUG/TRACE logging for security troubleshooting; it can reveal why authorization or authentication failed without exposing those details to clients. ([Home][1])

---

# 19. Example

Suppose we have:

```java
.requestMatchers(
    HttpMethod.DELETE,
    "/api/employees/**"
).hasRole("ADMIN")
```

And JWT:

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
DELETE /api/employees/101
Authorization: Bearer eyJ...
```

Result:

```text
JWT validation
    ↓
SUCCESS

Authentication
    ↓
ROLE_USER

Authorization
    ↓
ROLE_ADMIN required

FAIL
    ↓
AccessDeniedHandler

403
```

---

# 20. What if There Is No JWT?

```http
DELETE /api/employees/101
```

Result:

```text
No authentication
    ↓
AuthenticationEntryPoint
    ↓
401
```

So now you should never confuse these two cases.

---

# 21. Interview Questions

### What is `AuthenticationEntryPoint`?

> It handles authentication failures where the client is not successfully authenticated and initiates the appropriate authentication response, commonly a `401 Unauthorized` response for REST APIs.

### What is `AccessDeniedHandler`?

> It handles authorization failures when an authenticated user doesn't have sufficient authority, commonly resulting in `403 Forbidden`.

### Why doesn't `@RestControllerAdvice` always handle 401/403?

> Security failures often occur inside the Spring Security filter chain before the request reaches the MVC controller layer.

### Who handles a JWT validation failure?

> It is handled through Spring Security's authentication failure infrastructure, typically resulting in `AuthenticationEntryPoint` behavior and a `401` response. ([Home][2])

### Who handles insufficient role?

> `AccessDeniedHandler`, producing `403 Forbidden`.

### Can we customize the JSON error response?

Yes. Configure custom `AuthenticationEntryPoint` and `AccessDeniedHandler` implementations.

---

# 22. Best Practices

```text
✅ Keep 401 and 403 semantics correct
✅ Return consistent JSON error structures
✅ Don't expose stack traces or token details
✅ Keep application exceptions in @RestControllerAdvice
✅ Keep security failures in security exception handlers
✅ Use DEBUG/TRACE logging for diagnosis
✅ Don't log passwords or bearer tokens
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
├── ✅ Chapter 16 — Security Exception Handling
│
└── ⏭️ Chapter 17 — CORS
      ↓
      What is CORS?
      ↓
      Browser same-origin policy
      ↓
      Preflight OPTIONS
      ↓
      CorsConfiguration
      ↓
      Spring Security CORS integration
      ↓
      Common REST API mistakes
```

Next we'll cover **CORS**, especially why a React/Angular frontend can get a browser error even though your Spring REST API itself is working correctly.

[1]: https://docs.spring.io/spring-security/reference/servlet/architecture.html?utm_source=chatgpt.com "Architecture :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/oauth2/core/OAuth2ErrorCodes.html?utm_source=chatgpt.com "OAuth2ErrorCodes (spring-security-docs 7.1.0 API)"
[3]: https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com "Authorize HttpServletRequests :: Spring Security"
