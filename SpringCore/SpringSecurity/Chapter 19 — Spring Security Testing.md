# Spring Security — Chapter 19: Testing Spring Security

Now we test the security we built.

We have:

```text id="tm4q9c"
JWT Authentication ✅
Roles / Authorities ✅
URL Authorization ✅
Method Security ✅
CORS ✅
CSRF ✅
401 / 403 Handling ✅
```

But how do we prove the security actually works?

That's where **Spring Security Test** comes in.

Spring Security provides dedicated testing support for MockMvc and method security, including mock users, request post-processors, CSRF support, HTTP Basic, and OAuth2/JWT-related testing utilities. ([Home][1])

We'll use:

> **Why → What → How → Where → Complete Code → Test Scenarios → Interview Questions → Best Practices**

---

# 1. Why do we need Security Tests?

Suppose our API says:

```text id="a3g8kq"
USER
 → GET /api/employees

ADMIN
 → POST /api/employees
 → DELETE /api/employees/{id}
```

We need automated tests proving:

```text id="j7c4m2"
USER GET        → 200 ✅
USER DELETE     → 403 ✅
ADMIN DELETE    → 204 ✅
No user GET     → 401 ✅
Invalid JWT     → 401 ✅
```

Otherwise, someone could accidentally change:

```java id="3fv8mr"
.hasRole("ADMIN")
```

to:

```java id="v1k5dz"
.authenticated()
```

and suddenly every logged-in user could delete employees.

Tests protect against that regression.

---

# 2. What is Spring Security Test?

Add the test dependency:

```xml id="u5d1w9"
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

Spring Security's testing support is provided by the `spring-security-test` artifact. ([Home][1])

---

# 3. Why MockMvc?

You already learned Spring MVC.

`MockMvc` lets us perform HTTP requests against Spring MVC without starting a real HTTP server. It performs the MVC request-processing pipeline using mock request/response objects. ([Home][2])

So:

```text id="g2f8r1"
MockMvc
   ↓
Spring MVC
   ↓
Spring Security
   ↓
Controller
```

This is excellent for endpoint security testing.

---

# 4. Configure MockMvc with Spring Security

Example:

```java id="8h4p2r"
@SpringBootTest
@AutoConfigureMockMvc
class EmployeeControllerSecurityTest {

    @Autowired
    private MockMvc mvc;
}
```

With Spring Boot's `@AutoConfigureMockMvc`, Spring Boot configures the MVC test infrastructure.

Spring Security's MockMvc integration also requires its security filter infrastructure to be associated with MockMvc; when configuring it manually, `springSecurity()` adds the required `FilterChainProxy` and test security context support. ([Home][3])

For most Spring Boot integration tests:

```java id="q4v7d2"
@SpringBootTest
@AutoConfigureMockMvc
```

is the convenient starting point.

---

# 5. Testing as a User

This is one of the most useful features.

Spring Security provides:

```java id="b9m3x6"
@WithMockUser
```

Example:

```java id="s7q1n8"
@Test
@WithMockUser
void authenticatedUserCanAccessEmployees()
        throws Exception {

    mvc.perform(
        get("/api/employees")
    )
    .andExpect(
        status().isOk()
    );
}
```

`@WithMockUser` creates a mock authenticated user for the test; the user does not have to exist in your actual database. ([Home][4])

---

# 6. Testing a Specific Role

Suppose:

```text id="k6x2r4"
GET /api/employees
```

requires:

```text id="b1p8m7"
USER or ADMIN
```

Test:

```java id="t5q9c3"
@Test
@WithMockUser(roles = "USER")
void userCanReadEmployees()
        throws Exception {

    mvc.perform(
        get("/api/employees")
    )
    .andExpect(
        status().isOk()
    );
}
```

For ADMIN:

```java id="m7r2x5"
@Test
@WithMockUser(roles = "ADMIN")
void adminCanReadEmployees()
        throws Exception {

    mvc.perform(
        get("/api/employees")
    )
    .andExpect(
        status().isOk()
    );
}
```

---

# 7. Testing Authorization Failure

Now the important test.

USER should not delete:

```java id="z3c8n1"
@Test
@WithMockUser(roles = "USER")
void userCannotDeleteEmployee()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
    )
    .andExpect(
        status().isForbidden()
    );
}
```

Why 403?

```text id="r8x2m6"
User authenticated ✅
      ↓
ROLE_USER
      ↓
ROLE_ADMIN required
      ↓
403
```

Spring's authorization testing examples use `@WithMockUser` to verify allowed and forbidden endpoint access. ([Home][5])

---

# 8. Testing ADMIN

```java id="k1m7p4"
@Test
@WithMockUser(roles = "ADMIN")
void adminCanDeleteEmployee()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
    )
    .andExpect(
        status().isNoContent()
    );
}
```

This test verifies the authorization rule is correctly configured.

---

# 9. Testing Unauthenticated Requests

Don't use `@WithMockUser`.

```java id="f9q2x6"
@Test
void unauthenticatedUserCannotReadEmployees()
        throws Exception {

    mvc.perform(
        get("/api/employees")
    )
    .andExpect(
        status().isUnauthorized()
    );
}
```

Now we test the difference:

```text id="d8c4n1"
No Authentication
      ↓
401

USER authentication
      ↓
DELETE requiring ADMIN
      ↓
403
```

That is exactly the distinction we've been learning.

---

# 10. Testing Authorities

Suppose we use:

```java id="f3r8m2"
.hasAuthority("employee:delete")
```

Test:

```java id="x7v2k5"
@Test
@WithMockUser(
    authorities = "employee:delete"
)
void userWithDeleteAuthorityCanDelete()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
    )
    .andExpect(
        status().isNoContent()
    );
}
```

Without the authority:

```java id="p5m9c2"
@Test
@WithMockUser(
    authorities = "employee:read"
)
void readOnlyUserCannotDelete()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
    )
    .andExpect(
        status().isForbidden()
    );
}
```

---

# 11. Role vs Authority in Tests

Role:

```java id="q6n3x8"
@WithMockUser(roles = "ADMIN")
```

produces:

```text id="y7m2p4"
ROLE_ADMIN
```

Authority:

```java id="w8c1k5"
@WithMockUser(
    authorities = "employee:delete"
)
```

produces:

```text id="m3r7q2"
employee:delete
```

So the test should match what the production authorization rule expects.

---

# 12. Testing Method Security

Suppose our service contains:

```java id="j2k8m4"
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
    ...
}
```

You can test it directly:

```java id="s7p4x9"
@Test
@WithMockUser(roles = "ADMIN")
void adminCanDeleteEmployee() {

    employeeService.deleteEmployee(101L);
}
```

And:

```java id="c5n1m8"
@Test
@WithMockUser(roles = "USER")
void userCannotDeleteEmployee() {

    assertThatExceptionOfType(
        AccessDeniedException.class
    )
    .isThrownBy(() ->
        employeeService.deleteEmployee(101L)
    );
}
```

Spring Security's method-security testing documentation demonstrates this approach with `@WithMockUser` and `AccessDeniedException`. ([Home][6])

---

# 13. Why Test Method Security Separately?

Because:

```text id="x9p5k3"
URL Security
```

and:

```text id="m7c2r8"
Method Security
```

are different authorization layers.

You want tests proving both.

Example:

```text id="v4n1q6"
GET endpoint
  ↓
URL authorization test

Service method
  ↓
Method authorization test
```

---

# 14. Testing CSRF

Suppose your application has CSRF protection enabled.

A POST request may require a CSRF token.

Spring Security provides:

```java id="q5z9m3"
csrf()
```

as a MockMvc request post-processor. The security test support includes dedicated CSRF testing support. ([Home][7])

Example:

```java id="k8r2p7"
@Test
@WithMockUser
void postWithCsrfTokenWorks()
        throws Exception {

    mvc.perform(
        post("/api/employees")
            .with(csrf())
            .contentType(
                MediaType.APPLICATION_JSON
            )
            .content("""
                {
                    "name": "Rahul"
                }
                """)
    )
    .andExpect(
        status().isCreated()
    );
}
```

---

# 15. What Happens Without CSRF?

If CSRF protection is enabled:

```java id="f6x2n9"
@Test
@WithMockUser
void postWithoutCsrfFails()
        throws Exception {

    mvc.perform(
        post("/api/employees")
            .contentType(
                MediaType.APPLICATION_JSON
            )
            .content("""
                {
                    "name": "Rahul"
                }
                """)
    )
    .andExpect(
        status().isForbidden()
    );
}
```

That's a useful security regression test.

Spring Security's default CSRF support protects unsafe HTTP methods such as POST. ([Home][7])

For our stateless bearer-token API where CSRF is deliberately disabled, this particular test would not apply.

---

# 16. Testing HTTP Basic

Spring Security also provides:

```java id="s0q3m6"
httpBasic("user", "password")
```

as a MockMvc request post-processor.

The official testing support provides `httpBasic()` for populating the Authorization header with Basic credentials. ([Home][8])

Example:

```java id="h4n8p2"
@Test
void basicAuthenticationWorks()
        throws Exception {

    mvc.perform(
        get("/api/employees")
            .with(
                httpBasic(
                    "rahul",
                    "password123"
                )
            )
    )
    .andExpect(
        status().isOk()
    );
}
```

This is useful if you're testing the Chapter 10 Basic Authentication configuration.

---

# 17. Testing JWT

For JWT-secured APIs, Spring Security test support also provides JWT request-post-processing support.

The important idea is:

> You don't need to construct a real signed JWT for every controller authorization test.

You can create a mock JWT authentication and specify its authorities.

Conceptually:

```java id="g6q3r8"
mvc.perform(
    get("/api/employees")
        .with(
            jwt()
                .authorities(
                    new SimpleGrantedAuthority(
                        "ROLE_USER"
                    )
                )
        )
)
.andExpect(
    status().isOk()
);
```

Spring Security provides MockMvc OAuth2/JWT testing utilities specifically to simulate authenticated OAuth2/JWT requests. ([Home][9])

---

# 18. JWT ADMIN Test

```java id="n3v8c1"
@Test
void jwtAdminCanDeleteEmployee()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
            .with(
                jwt()
                    .authorities(
                        new SimpleGrantedAuthority(
                            "ROLE_ADMIN"
                        )
                    )
            )
    )
    .andExpect(
        status().isNoContent()
    );
}
```

USER:

```java id="q7m2x4"
@Test
void jwtUserCannotDeleteEmployee()
        throws Exception {

    mvc.perform(
        delete("/api/employees/101")
            .with(
                jwt()
                    .authorities(
                        new SimpleGrantedAuthority(
                            "ROLE_USER"
                        )
                    )
            )
    )
    .andExpect(
        status().isForbidden()
    );
}
```

This directly tests your authorization rules without requiring a real login flow.

---

# 19. Why Mock JWT Instead of Real JWT?

Because these tests are primarily testing:

```text id="b8q3m6"
Authorization
```

not:

```text id="p2n7x1"
JWT cryptography
```

You should test them separately.

### Authorization test

```text id="r5k8c2"
Mock JWT
 ↓
ROLE_USER
 ↓
403
```

### JWT validation integration test

```text id="m4x9p7"
Real JWT
 ↓
Signature
 ↓
Expiration
 ↓
Decoder
 ↓
Authentication
```

This separation makes tests faster and more focused.

---

# 20. Complete Employee Security Test Class

Here's a useful starting point for our project:

```java id="c9m4q7"
@SpringBootTest
@AutoConfigureMockMvc
class EmployeeSecurityTest {

    @Autowired
    private MockMvc mvc;

    @Test
    @WithMockUser(roles = "USER")
    void userCanReadEmployees()
            throws Exception {

        mvc.perform(
            get("/api/employees")
        )
        .andExpect(
            status().isOk()
        );
    }

    @Test
    @WithMockUser(roles = "USER")
    void userCannotDeleteEmployee()
            throws Exception {

        mvc.perform(
            delete("/api/employees/101")
        )
        .andExpect(
            status().isForbidden()
        );
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void adminCanDeleteEmployee()
            throws Exception {

        mvc.perform(
            delete("/api/employees/101")
        )
        .andExpect(
            status().isNoContent()
        );
    }

    @Test
    void unauthenticatedUserCannotReadEmployees()
            throws Exception {

        mvc.perform(
            get("/api/employees")
        )
        .andExpect(
            status().isUnauthorized()
        );
    }
}
```

There is one practical issue with the exact expected response of the DELETE test: the database must actually contain employee `101` and the endpoint must perform the delete. In a real suite, you'd either seed controlled test data or mock the service/repository behavior.

---

# 21. Unit Test vs Integration Test

This distinction is important.

### Unit test

Tests one component in isolation.

Example:

```text id="h1v6r9"
EmployeeService
    ↓
Mock Repository
```

### Integration test

Tests multiple Spring components together:

```text id="t5n8q2"
HTTP
 ↓
Security
 ↓
Controller
 ↓
Service
 ↓
Repository
```

`MockMvc` is especially useful for testing the Spring MVC request path without starting an actual HTTP server. ([Home][2])

---

# 22. What Should We Test?

For our Employee API:

```text id="y3p8c4"
Authentication
├── No credentials → 401
├── Invalid JWT → 401
└── Valid JWT → authenticated

Authorization
├── USER GET → 200
├── USER POST → 403
├── USER DELETE → 403
├── ADMIN POST → 201
└── ADMIN DELETE → 204

Method Security
├── USER → AccessDeniedException
└── ADMIN → allowed

CSRF
├── Missing token → 403
└── Valid token → allowed

CORS
└── Browser integration tests where appropriate
```

That's a strong security test matrix.

---

# 23. The Most Important Testing Principle

Don't only test:

```text id="4p2q9x"
ADMIN can delete
```

Also test:

```text id="9c7m1k"
USER cannot delete
Anonymous cannot delete
```

Security tests are often about proving that **access is denied**, not just that access works.

This is particularly important because Spring Security authorization rules are intended to be explicit and can be tested with its MockMvc support. ([Home][5])

---

# 24. Interview Questions

### What is `spring-security-test`?

> It is Spring Security's dedicated testing support for testing authentication, authorization, CSRF, MockMvc integration, method security, and related security behavior. ([Home][1])

### What does `@WithMockUser` do?

> It runs a test with a mocked authenticated user; the user does not need to exist in the actual user store. ([Home][4])

### Why use MockMvc?

> It tests Spring MVC request handling and can integrate the Spring Security filter chain without starting a real HTTP server. ([Home][2])

### How do you test USER vs ADMIN?

```java id="z4q8p1"
@WithMockUser(roles = "USER")
```

versus:

```java id="m8c2r6"
@WithMockUser(roles = "ADMIN")
```

### How do you test CSRF?

```java id="d6p4q9"
.with(csrf())
```

Spring Security provides dedicated CSRF request post-processors. ([Home][10])

### How do you test a JWT-secured endpoint?

> Use Spring Security's JWT/OAuth2 MockMvc support to simulate an authenticated bearer-token request and supply the authorities needed for the authorization decision. ([Home][9])

---

# 25. Best Practices

```text id="n8p3q5"
✅ Test both allowed and denied access
✅ Test 401 separately from 403
✅ Test roles and exact authorities
✅ Test method security independently
✅ Test CSRF when CSRF is enabled
✅ Use mock JWTs for authorization-focused tests
✅ Use real JWT integration tests for decoder/signature behavior
✅ Keep security tests deterministic
✅ Don't depend on production users in tests
```

---

# 📍 Where We Are

```text id="j4q7m2"
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
├── ✅ Chapter 18 — CSRF
├── ✅ Chapter 19 — Security Testing
│
└── ⏭️ Chapter 20 — Complete Secure Employee API ⭐⭐⭐⭐⭐
      ↓
      Login
      ↓
      JWT generation
      ↓
      JWT validation
      ↓
      USER / ADMIN
      ↓
      Method security
      ↓
      CORS
      ↓
      Exception handling
      ↓
      Tests
```

## Next: Chapter 20 — Complete Secure Employee API

This will be our **Spring Security capstone**. We'll take the Employee REST API we built earlier and assemble the complete production-style flow:

```text id="q7m4x9"
POST /api/auth/login
        ↓
JWT

Authorization: Bearer <JWT>
        ↓
SecurityFilterChain
        ↓
JWT Validation
        ↓
SecurityContext
        ↓
USER / ADMIN
        ↓
Controller
        ↓
Service
        ↓
@PreAuthorize
        ↓
Repository
        ↓
Database
```

After that, **Spring Security will be completed**, and we will start the **Spring Boot module**.

[1]: https://docs.spring.io/spring-security/reference/servlet/test/index.html?utm_source=chatgpt.com "Testing :: Spring Security"
[2]: https://docs.spring.io/spring-framework/reference/testing/mockmvc.html?utm_source=chatgpt.com "MockMvc :: Spring Framework"
[3]: https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/setup.html?utm_source=chatgpt.com "Setting Up MockMvc and Spring Security"
[4]: https://docs.spring.io/spring-security/reference/servlet/test/method.html?utm_source=chatgpt.com "Testing Method Security"
[5]: https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html?utm_source=chatgpt.com "Authorize HttpServletRequests :: Spring Security"
[6]: https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html?utm_source=chatgpt.com "Method Security"
[7]: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html?utm_source=chatgpt.com "Cross Site Request Forgery (CSRF)"
[8]: https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/http-basic.html?utm_source=chatgpt.com "Testing HTTP Basic Authentication"
[9]: https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/oauth2.html?utm_source=chatgpt.com "Testing OAuth 2.0 :: Spring Security"
[10]: https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/request-post-processors.html?utm_source=chatgpt.com "SecurityMockMvcRequestPostPr..."
