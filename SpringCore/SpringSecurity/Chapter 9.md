# Spring Security — Chapter 9: Method Security

Now we move from **URL-level authorization** to **method-level authorization**.

You already know:

```text
URL Security
    ↓
SecurityFilterChain
    ↓
"Can this HTTP request access this endpoint?"
```

Now we want:

```text
Method Security
    ↓
"Can this user execute this business method?"
```

This is especially important because it connects **Spring Security + Spring AOP**, which you already learned.

---

# 1. Why do we need Method Security?

Suppose we protect:

```http
DELETE /api/employees/101
```

using:

```java
.requestMatchers(HttpMethod.DELETE, "/api/employees/**")
    .hasRole("ADMIN")
```

That's good.

But imagine another part of the application calls:

```java
employeeService.deleteEmployee(101);
```

For example:

```text
Controller
   ↓
EmployeeService

Scheduler
   ↓
EmployeeService

Another Service
   ↓
EmployeeService
```

URL security only protects the HTTP endpoint.

It doesn't directly express:

> "Only ADMIN may execute this business operation."

Method Security solves that.

---

# 2. What is Method Security?

Method Security allows Spring Security authorization annotations to protect Java methods.

The most important annotation is:

```java
@PreAuthorize
```

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
    ...
}
```

This means:

> Before executing this method, the current user must have the `ADMIN` role.

Spring's method-security support includes `@PreAuthorize`, `@PostAuthorize`, `@PreFilter`, and `@PostFilter`; method security is enabled with `@EnableMethodSecurity`.

---

# 3. How do we enable Method Security?

Add:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
}
```

Then Spring Security can process method-security annotations.

Example:

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

---

# 4. The Most Important Annotation: `@PreAuthorize`

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
    repository.delete(id);
}
```

Flow:

```text
Caller
   ↓
Security Proxy
   ↓
@PreAuthorize
   ↓
Authorization Check
   ↓
Allowed?
 ┌──────┴──────┐
YES            NO
 ↓              ↓
Method        Access Denied
Executes         ↓
               403
```

So the security check happens **before the method executes**.

---

# 5. Why is this related to AOP?

You learned Spring AOP earlier.

Remember:

```text
Client
   ↓
Proxy
   ↓
Target Object
```

Method security uses a similar proxy/interceptor mechanism.

Conceptually:

```text
Client
   ↓
Spring Security Method Proxy
   ↓
Authorization Check
   ↓
EmployeeService.deleteEmployee()
```

So:

```text
Spring AOP
    +
Spring Security
    ↓
Method-level authorization
```

This is a very useful connection to remember.

---

# 6. Real Employee Example

Suppose:

```java
@Service
public class EmployeeService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteEmployee(Long id) {

        repository.delete(id);
    }
}
```

Then:

```text
ADMIN
   ↓
deleteEmployee()
   ↓
Allowed ✅
```

But:

```text
USER
   ↓
deleteEmployee()
   ↓
Denied ❌
```

The repository isn't called for the unauthorized user.

---

# 7. `@PreAuthorize` with Authority

You aren't limited to roles.

```java
@PreAuthorize("hasAuthority('employee:delete')")
public void deleteEmployee(Long id) {
    ...
}
```

Now the user needs the exact authority:

```text
employee:delete
```

This is useful for fine-grained permissions.

---

# 8. `hasAnyRole`

```java
@PreAuthorize(
    "hasAnyRole('ADMIN', 'HR_MANAGER')"
)
public EmployeeResponse updateEmployee(...) {
    ...
}
```

Meaning:

```text
ROLE_ADMIN
OR
ROLE_HR_MANAGER
```

---

# 9. Multiple Conditions

You can combine conditions.

```java
@PreAuthorize(
    "hasRole('ADMIN') or hasAuthority('employee:update')"
)
public void updateEmployee(Long id) {
}
```

Meaning:

```text
ADMIN
     OR
employee:update
```

---

# 10. AND Condition

```java
@PreAuthorize(
    "hasRole('ADMIN') and hasAuthority('employee:delete')"
)
public void deleteEmployee(Long id) {
}
```

Now both must be true.

```text
ROLE_ADMIN ✅
        +
employee:delete ✅
        =
Allowed ✅
```

---

# 11. Using the Current User

Method Security can also use method arguments and authentication information.

For example:

```java
@PreAuthorize("#userId == authentication.principal.id")
public EmployeeResponse getProfile(Long userId) {
    ...
}
```

Conceptually:

```text
Current authenticated user ID
        ↓
Compare
        ↓
Requested user ID
```

This is extremely useful for ownership-based access.

For example:

```text
Rahul → Can view Rahul's profile
Rahul → Cannot view Amit's private profile
```

---

# 12. `authentication`

Inside the expression:

```java
authentication
```

represents the current authentication.

Example:

```java
@PreAuthorize(
    "authentication.name == #username"
)
public UserProfile getProfile(String username) {
    ...
}
```

Spring evaluates the expression using the current security context.

---

# 13. `principal`

You can also access the principal:

```java
@PreAuthorize(
    "principal.username == #username"
)
```

The exact available properties depend on the principal object.

This is especially useful with a custom `UserDetails`.

---

# 14. Ownership Example

Suppose:

```java
@PreAuthorize(
    "authentication.name == #username"
)
public void updateProfile(String username) {
}
```

Request:

```text
Rahul
   ↓
updateProfile("Rahul")
```

Allowed.

But:

```text
Rahul
   ↓
updateProfile("Amit")
```

Denied.

This is more powerful than simply checking:

```text
ROLE_USER
```

because it checks **which specific user owns the resource**.

---

# 15. `@PostAuthorize`

Now let's look at another annotation.

```java
@PostAuthorize
```

Unlike:

```java
@PreAuthorize
```

which runs **before** the method,

`@PostAuthorize` evaluates **after** the method returns.

Example:

```java
@PostAuthorize(
    "returnObject.owner == authentication.name"
)
public EmployeeResponse getEmployee(Long id) {
    return repository.find(id);
}
```

Conceptually:

```text
Call Method
    ↓
Method Executes
    ↓
Employee returned
    ↓
Authorization Check
    ↓
Allowed?
```

Spring supports `@PostAuthorize` for post-invocation authorization.

---

# 16. Why use `@PostAuthorize`?

Imagine the authorization decision depends on the returned object.

For example:

```text
Employee
owner = Rahul
```

Current user:

```text
Rahul
```

Then:

```text
Allowed ✅
```

But if:

```text
Employee
owner = Amit
```

then Rahul shouldn't receive it.

This is useful for **object-level authorization**.

---

# 17. Important Difference

### `@PreAuthorize`

```text
Check
 ↓
Method
```

### `@PostAuthorize`

```text
Method
 ↓
Check returned result
```

Memorize:

```text
PRE  = Before
POST = After
```

---

# 18. `@PreFilter`

This is more advanced.

It can filter collection arguments **before method execution**.

Conceptually:

```java
@PreFilter(
    "filterObject.owner == authentication.name"
)
public void process(List<EmployeeRequest> employees) {
}
```

Suppose input contains:

```text
Rahul's Employee
Amit's Employee
Rahul's Employee
```

Spring can filter the collection according to the security expression before the method executes.

For your level:

> **Know what it is. Don't spend much time implementing it yet.**

---

# 19. `@PostFilter`

Similarly:

```java
@PostFilter(
    "filterObject.owner == authentication.name"
)
public List<Employee> getEmployees() {
    ...
}
```

The method executes, returns a collection, and Spring filters elements according to the expression.

Again:

> Useful to know conceptually; less commonly needed in modern REST APIs than explicit service-level filtering/querying.

---

# 20. URL Security vs Method Security

This is important.

## URL Security

```java
.requestMatchers(
    HttpMethod.DELETE,
    "/api/employees/**"
).hasRole("ADMIN")
```

Protects:

```text
HTTP Request
```

---

## Method Security

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteEmployee(Long id) {
}
```

Protects:

```text
Java Method
```

So:

```text
HTTP Boundary
    ↓
URL Authorization

Business Boundary
    ↓
Method Authorization
```

---

# 21. Should we use both?

Often, yes.

For example:

```text
SecurityFilterChain
    ↓
Only authenticated users can enter /api/**
```

Then:

```text
@PreAuthorize
    ↓
Only ADMIN can perform sensitive business operation
```

The two mechanisms protect different layers.

But don't blindly duplicate every rule everywhere. Design the authorization boundaries intentionally.

---

# 22. A Strong Employee Service

Our Employee Service could look like:

```java
@Service
public class EmployeeService {

    @PreAuthorize(
        "hasAnyRole('USER', 'ADMIN')"
    )
    public EmployeeResponse getById(Long id) {
        ...
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    public EmployeeResponse create(
            EmployeeCreateRequest request) {
        ...
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    public EmployeeResponse update(
            Long id,
            EmployeeUpdateRequest request) {
        ...
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    public void delete(Long id) {
        ...
    }
}
```

Now the business service itself expresses its security requirements.

---

# 23. Method Security and Self-Invocation

Remember the transaction self-invocation problem?

```text
methodA()
   ↓
methodB()
```

and the proxy can be bypassed.

The same general proxy limitation can affect method-security annotations.

For example:

```java
public void methodA() {
    methodB();
}

@PreAuthorize("hasRole('ADMIN')")
public void methodB() {
}
```

A direct internal call may bypass the Spring proxy, so the method-security interceptor may not run.

This is another reason your earlier **Spring AOP / self-invocation** knowledge is important.

---

# 24. Important Interview Question

> "Why does `@PreAuthorize` sometimes not work?"

Strong answer:

> Method security is implemented through Spring's method interception infrastructure. A call must go through the Spring-managed proxy for the authorization interceptor to execute. A direct self-invocation within the same object can bypass that proxy.

That's the same family of problem we saw with:

```text
@Transactional
```

---

# 25. What is SpEL doing here?

You've already learned SpEL in Spring Core.

Now you see it again:

```java
@PreAuthorize(
    "hasRole('ADMIN')"
)
```

This is a **Spring Expression Language expression** used by method security.

More examples:

```java
@PreAuthorize("hasRole('ADMIN')")
```

```java
@PreAuthorize(
    "hasAuthority('employee:delete')"
)
```

```java
@PreAuthorize(
    "#userId == authentication.principal.id"
)
```

So this is another practical use of the SpEL knowledge you learned earlier.

---

# 26. Interview Questions

### What is Method Security?

> Method Security provides authorization at the Java method level using annotations such as `@PreAuthorize` and `@PostAuthorize`.

### How do you enable it?

```java
@EnableMethodSecurity
```

### What does `@PreAuthorize` do?

> Evaluates an authorization expression before the method executes.

### What does `@PostAuthorize` do?

> Evaluates an authorization expression after the method returns.

### Difference between URL authorization and method security?

> URL authorization protects HTTP requests/endpoints; method security protects Java methods/business operations.

### Can `@PreAuthorize` use method arguments?

Yes:

```java
@PreAuthorize("#id == authentication.principal.id")
```

### Can method security suffer from self-invocation?

Yes. Because proxy-based interception can be bypassed by direct calls inside the same object.

---

# 27. Best Practices

```text
✅ Use @PreAuthorize for sensitive business operations
✅ Keep authorization rules close to business operations when appropriate
✅ Use roles for coarse-grained access
✅ Use authorities for fine-grained permissions
✅ Use ownership checks where required
✅ Don't rely on frontend authorization
✅ Be aware of proxy/self-invocation behavior
```

Avoid putting enormous SpEL expressions directly into annotations.

If authorization logic becomes extremely complicated:

```text
@PreAuthorize("...")
```

with a giant expression can become difficult to maintain.

In larger systems, move complex authorization decisions into dedicated components/services.

---

# 📍 Where We Are

```text
Spring Security
│
├── ✅ Chapter 1
│   Why Security?
│
├── ✅ Chapter 2
│   SecurityFilterChain
│
├── ✅ Chapter 3
│   Authentication / SecurityContext
│
├── ✅ Chapter 4
│   UserDetails / UserDetailsService
│
├── ✅ Chapter 5
│   PasswordEncoder / BCrypt
│
├── ✅ Chapter 6
│   AuthenticationManager
│   AuthenticationProvider
│   DaoAuthenticationProvider
│
├── ✅ Chapter 7
│   Roles / Authorities
│
├── ✅ Chapter 8
│   URL Authorization
│
├── ✅ Chapter 9
│   Method Security
│   @PreAuthorize
│   @PostAuthorize
│   @PreFilter / @PostFilter
│
└── ⏭️ Chapter 10
      Basic Authentication
      ↓
      HTTP Basic
      ↓
      Browser/Postman authentication
      ↓
      Complete username/password request flow
```

Next we'll implement **HTTP Basic Authentication** against our Employee API. That gives you a concrete working authentication mechanism before we move to **stateless REST security and JWT**, which is the most important part for modern Spring Boot REST interviews.
