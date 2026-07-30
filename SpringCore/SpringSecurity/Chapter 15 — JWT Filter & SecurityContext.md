# Spring Security — Chapter 15: `OncePerRequestFilter` and the JWT Filter Pattern

Now we reach a topic that causes a lot of confusion because you'll see **two different approaches** in Spring projects:

```text
Approach 1
Spring Security Resource Server
    ↓
Built-in BearerTokenAuthenticationFilter ✅

Approach 2
Custom JWT Filter
    ↓
OncePerRequestFilter
```

For a **modern Spring Security 7 application**, you should prefer the built-in Resource Server support when it fits your requirements. You should understand the custom filter pattern because you'll encounter it in existing codebases and interviews. Spring Security's own bearer-token support already provides `BearerTokenAuthenticationFilter`, which itself extends `OncePerRequestFilter`. ([Home][1])

---

# 1. Why do we need a filter?

Suppose every protected request contains:

```http
Authorization: Bearer eyJ...
```

Something must intercept the request **before the controller** and extract that token.

Conceptually:

```text
HTTP Request
      ↓
JWT Filter
      ↓
Extract Token
      ↓
Validate Token
      ↓
Create Authentication
      ↓
SecurityContext
      ↓
Controller
```

That's why filters are involved in JWT authentication.

Servlet filters can run before and after the rest of the filter chain and target servlet processing. ([Home][2])

---

# 2. What is `OncePerRequestFilter`?

Spring Framework provides:

```java
OncePerRequestFilter
```

It is a base class for filters that aims to ensure the filter logic executes once per request dispatch, and it gives you:

```java
doFilterInternal(
    HttpServletRequest request,
    HttpServletResponse response,
    FilterChain filterChain
)
```

for your filtering logic. ([Home][3])

Think:

```text
Filter
   ↓
OncePerRequestFilter
   ↓
Your custom filter
```

---

# 3. Why is it called "Once Per Request"?

A servlet request can have different dispatcher phases, such as:

```text
REQUEST
FORWARD
ASYNC
ERROR
```

`OncePerRequestFilter` provides mechanisms to control whether the filter participates in async and error dispatches and tracks whether it has already filtered the request. ([Home][3])

For your level, remember:

> **It provides a convenient Spring base class when you want custom request filtering without accidentally executing your filter logic multiple times for the same request dispatch.**

---

# 4. Why would we create a custom JWT filter?

You might see code like:

```java
public class JwtAuthenticationFilter
        extends OncePerRequestFilter {
}
```

The filter may:

```text
1. Read Authorization header
2. Extract Bearer token
3. Validate JWT
4. Extract username/claims
5. Create Authentication
6. Put it into SecurityContext
7. Continue the request
```

Conceptually:

```text
Bearer JWT
   ↓
Custom JWT Filter
   ↓
Authentication
   ↓
SecurityContext
```

---

# 5. But Wait — Doesn't Spring Security Already Do This?

**Yes.**

This is extremely important.

When you configure:

```java
http.oauth2ResourceServer(
    oauth2 -> oauth2.jwt(
        Customizer.withDefaults()
    )
);
```

Spring Security already provides the bearer-token infrastructure.

Its `BearerTokenAuthenticationFilter` extends `OncePerRequestFilter`. ([Home][1])

So you generally **do not need to write your own JWT filter** just to parse and validate a standard bearer JWT.

This is the recommended modern direction for a normal JWT resource server.

---

# 6. Then Why Do Tutorials Use `OncePerRequestFilter` Everywhere?

Because many tutorials teach a custom architecture like:

```text
JWT Filter
   ↓
extract token
   ↓
parse JWT
   ↓
find user
   ↓
set SecurityContext
```

This pattern became extremely common in custom JWT examples.

It's useful to understand, but you shouldn't blindly copy it into every new Spring Boot application.

A modern Spring Security application can often use:

```text
oauth2ResourceServer().jwt()
```

instead.

Spring Security's Resource Server documentation describes the built-in bearer-token processing and JWT authentication architecture. ([Home][4])

---

# 7. Custom JWT Filter — Basic Structure

For learning, let's see what one looks like.

```java
@Component
public class JwtAuthenticationFilter
        extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String authorization =
                request.getHeader("Authorization");

        if (authorization == null ||
            !authorization.startsWith("Bearer ")) {

            filterChain.doFilter(
                    request,
                    response
            );

            return;
        }

        String token =
                authorization.substring(7);

        // Validate token
        // Extract subject
        // Create Authentication
        // Put Authentication into SecurityContext

        filterChain.doFilter(
                request,
                response
        );
    }
}
```

That's the skeleton.

---

# 8. The Most Important Line

This:

```java
filterChain.doFilter(
    request,
    response
);
```

means:

> **Continue processing the request.**

If you forget it in a branch where processing should continue, the request can stop at your filter.

Think:

```text
Filter
 ↓
Do my work
 ↓
filterChain.doFilter(...)
 ↓
Next filter
```

---

# 9. If the Token Isn't Present?

Suppose:

```http
GET /api/public
```

has no Authorization header.

A custom filter might do:

```java
if (authorization == null) {
    filterChain.doFilter(request, response);
    return;
}
```

Meaning:

> "I don't have a token to process, so let the rest of Spring Security decide what happens."

This is important.

The JWT filter itself doesn't necessarily decide that every tokenless request is allowed.

The **authorization configuration** decides whether authentication is actually required.

---

# 10. What if the Token Exists?

Suppose:

```http
Authorization: Bearer eyJ...
```

Then the filter:

```text
Extract token
     ↓
Validate token
     ↓
Extract subject/claims
     ↓
Create Authentication
     ↓
SecurityContextHolder
```

Conceptually:

```java
Authentication authentication =
        new UsernamePasswordAuthenticationToken(
                username,
                null,
                authorities
        );

SecurityContextHolder
        .getContext()
        .setAuthentication(authentication);
```

This is the key idea behind many custom JWT filters.

---

# 11. Why Is `SecurityContext` Important Here?

Remember our previous chapter:

```text
SecurityContextHolder
      ↓
SecurityContext
      ↓
Authentication
```

The custom filter creates the authenticated identity for the current request.

Then later:

```text
Authorization
     ↓
"Does this Authentication have ROLE_ADMIN?"
```

So:

```text
JWT Filter
    ↓
Authentication
    ↓
SecurityContext
    ↓
AuthorizationFilter
    ↓
Controller
```

Spring Security's request architecture uses the security context to carry the current authentication through request processing. ([Home][5])

---

# 12. What If You Don't Put Authentication in the Context?

Suppose your filter successfully validates:

```text
JWT ✅
```

but doesn't create/set an `Authentication`.

Then later authorization sees:

```text
No authenticated user
```

So your API may still reject the request.

That's why:

```text
JWT validation
+
Authentication creation
```

are separate steps.

---

# 13. Custom Filter Flow

A simplified custom implementation looks like:

```text
HTTP Request
    ↓
JwtAuthenticationFilter
    ↓
Authorization Header
    ↓
Extract Bearer Token
    ↓
Validate JWT
    ↓
Extract subject/roles
    ↓
Create Authentication
    ↓
SecurityContextHolder
    ↓
Continue Filter Chain
    ↓
AuthorizationFilter
    ↓
DispatcherServlet
    ↓
Controller
```

---

# 14. `addFilterBefore`

In custom-filter tutorials, you'll often see:

```java
http.addFilterBefore(
    jwtAuthenticationFilter,
    UsernamePasswordAuthenticationFilter.class
);
```

This tells Spring Security where to place your custom filter relative to another filter.

The filter ordering is significant in Spring Security, although applications generally shouldn't need to memorize the entire default order. ([Home][6])

Conceptually:

```text
Filter A
   ↓
Your JWT Filter
   ↓
Filter B
   ↓
Authorization
```

---

# 15. Why `addFilterBefore` Is Used

A JWT filter usually needs to establish authentication **before authorization decisions are made**.

So the request needs to become:

```text
JWT
 ↓
Authentication
 ↓
Authorization
```

not:

```text
Authorization
 ↓
JWT validation
```

because authorization needs an authenticated principal and authorities.

---

# 16. But Modern Resource Server Doesn't Need This

This is the most important practical takeaway.

With:

```java
http.oauth2ResourceServer(
    oauth2 -> oauth2.jwt(
        Customizer.withDefaults()
    )
);
```

Spring Security already installs its bearer-token support, including `BearerTokenAuthenticationFilter`. ([Home][4])

So don't add:

```java
JwtAuthenticationFilter
```

just because every YouTube tutorial does.

---

# 17. When Would a Custom Filter Be Useful?

A custom filter can make sense when you have a **non-standard authentication mechanism** or custom request-processing requirement.

Examples might include:

```text
Custom proprietary token
Special authentication protocol
Legacy authentication format
Custom header-based identity mechanism
Additional request-level security processing
```

But for normal:

```http
Authorization: Bearer <JWT>
```

with a standard signed JWT:

```text
Use Resource Server support
```

rather than reinventing JWT authentication.

---

# 18. Custom JWT Filter vs Resource Server

| Custom JWT Filter           | Resource Server                     |
| --------------------------- | ----------------------------------- |
| You write filter            | Spring provides filter              |
| You parse token             | Spring parses bearer token          |
| You validate token          | `JwtDecoder` validates              |
| You create Authentication   | Spring creates authentication       |
| More code                   | Less code                           |
| More responsibility         | Framework-managed                   |
| Useful for custom protocols | Best for standard JWT bearer tokens |

The official Resource Server architecture is specifically designed around bearer-token authentication, JWT decoding, and creation of the authenticated security context. ([Home][4])

---

# 19. A Common Beginner Mistake

Many developers write:

```java
if (token != null) {
    validate(token);
}
```

and then:

```java
filterChain.doFilter(...)
```

but don't properly handle:

```text
Expired token
Invalid signature
Malformed token
Invalid claims
```

A robust security implementation must have clear failure handling.

That's another reason to prefer Spring Security's built-in resource-server implementation for standard JWTs.

---

# 20. Another Common Mistake

Developers sometimes do this inside the JWT filter:

```text
JWT
 ↓
username
 ↓
Database query
 ↓
UserDetails
 ↓
Authentication
```

for **every request**.

That can be valid for some architectures, especially when you need current user state, but a self-contained JWT can often establish authentication from the validated token itself.

Resource Server's JWT flow validates the token and converts claims into an authenticated principal without requiring a user database lookup on every request. ([Home][4])

---

# 21. One Very Important SecurityContext Detail

Spring Security's filter infrastructure ensures the `SecurityContext` is cleared after the request. ([Home][5])

So think:

```text
Request 1
   ↓
SecurityContext
   ↓
Authentication Rahul
   ↓
Request completes
   ↓
Context cleared

Request 2
   ↓
New Authentication established
```

This is why stateless JWT authentication can still use a `SecurityContext` during each request.

---

# 22. Custom Filter Example — Better Structure

If you actually needed a custom JWT filter, the architecture should be closer to:

```java
@Component
public class JwtAuthenticationFilter
        extends OncePerRequestFilter {

    private final JwtService jwtService;

    public JwtAuthenticationFilter(
            JwtService jwtService) {

        this.jwtService = jwtService;
    }

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String header =
                request.getHeader("Authorization");

        if (header == null ||
            !header.startsWith("Bearer ")) {

            filterChain.doFilter(
                    request,
                    response
            );

            return;
        }

        String token =
                header.substring(7);

        try {

            String username =
                    jwtService.extractUsername(token);

            if (username != null &&
                SecurityContextHolder
                    .getContext()
                    .getAuthentication() == null) {

                // Validate token

                Authentication authentication =
                        jwtService.createAuthentication(
                                token
                        );

                SecurityContextHolder
                    .getContext()
                    .setAuthentication(
                            authentication
                    );
            }

        } catch (JwtException ex) {

            // Clear authentication / let
            // security failure handling occur
            SecurityContextHolder.clearContext();
        }

        filterChain.doFilter(
                request,
                response
        );
    }
}
```

This is a **learning example**, not something you should copy blindly into production.

---

# 23. Why Check Existing Authentication?

This:

```java
SecurityContextHolder
    .getContext()
    .getAuthentication()
```

may already contain authentication.

A custom filter should avoid unnecessarily replacing an existing valid authentication.

Conceptually:

```text
Already authenticated?
    ↓
YES → don't recreate
NO  → process JWT
```

---

# 24. How Does Resource Server Replace All This?

Instead of:

```text
Custom Filter
 ↓
Extract
 ↓
Validate
 ↓
Authentication
 ↓
Context
```

you configure:

```java
http.oauth2ResourceServer(
    oauth2 -> oauth2.jwt(
        Customizer.withDefaults()
    )
);
```

and Spring Security handles the standard bearer-token flow. ([Home][4])

That means your application code can focus on:

```text
Authorization rules
Business logic
DTOs
Controllers
Services
```

instead of maintaining low-level JWT parsing infrastructure.

---

# 25. Interview Question

### Why extend `OncePerRequestFilter`?

Good answer:

> It is a Spring-provided base class for servlet filters that provides once-per-request execution semantics and a convenient `doFilterInternal` method. It is commonly used when implementing custom request-processing filters. ([Home][3])

### Is a custom JWT filter required for Spring Security JWT?

> No. For standard bearer JWT authentication, Spring Security Resource Server provides `BearerTokenAuthenticationFilter` and the JWT authentication infrastructure. A custom filter is mainly needed for non-standard/custom authentication behavior. ([Home][1])

### What does `filterChain.doFilter()` do?

> It passes the request to the next filter in the chain.

### Where should JWT authentication be established?

> Before authorization decisions are made, so the authenticated principal and authorities are available to the authorization layer.

---

# 26. Best Practices

```text
✅ Prefer Resource Server for standard JWT bearer authentication
✅ Use OncePerRequestFilter for genuine custom filtering needs
✅ Don't manually parse JWTs in controllers
✅ Don't duplicate Spring Security's built-in JWT processing
✅ Establish Authentication before authorization
✅ Handle invalid tokens safely
✅ Don't log bearer tokens
✅ Keep the SecurityContext request-scoped
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
├── ✅ Chapter 15 — OncePerRequestFilter / Custom JWT Filter
│
└── ⏭️ Chapter 16 — 401 vs 403 + Exception Handling
       ↓
       AuthenticationEntryPoint
       ↓
       AccessDeniedHandler
       ↓
       JWT failures
       ↓
       Consistent JSON security errors
```

Next we'll make the security errors **production-style**: exactly why Spring returns **401 vs 403**, what `AuthenticationEntryPoint` does, what `AccessDeniedHandler` does, and how to return a consistent JSON error response from your secured REST API.

[1]: https://docs.spring.io/spring-security/site/docs/7.0.x/api/org/springframework/security/oauth2/server/resource/web/authentication/BearerTokenAuthenticationFilter.html?utm_source=chatgpt.com "BearerTokenAuthenticationFilter (spring-security-docs 7.0.0 API)"
[2]: https://docs.spring.io/spring-framework/reference/web/webmvc/filters.html?utm_source=chatgpt.com "Filters :: Spring Framework"
[3]: https://docs.spring.io/spring-framework/docs/7.0.0/javadoc-api/org/springframework/web/filter/OncePerRequestFilter.html?utm_source=chatgpt.com "OncePerRequestFilter (Spring Framework 7.0.0 API)"
[4]: https://docs.spring.io/spring-security/reference/7.0/servlet/oauth2/resource-server/index.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server :: Spring Security"
[5]: https://docs.spring.io/spring-security/reference/7.0/servlet/authentication/architecture.html?utm_source=chatgpt.com "Servlet Authentication Architecture :: Spring Security"
[6]: https://docs.spring.io/spring-security/reference/7.0/servlet/architecture.html?utm_source=chatgpt.com "Architecture :: Spring Security"
