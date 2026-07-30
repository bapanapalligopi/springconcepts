# Spring Security — Chapter 6: `AuthenticationManager`, `AuthenticationProvider`, and `DaoAuthenticationProvider`

Now the pieces finally come together.

We have learned:

```text id="e2qk91"
SecurityFilterChain
        ↓
Authentication
        ↓
UserDetails
        ↓
UserDetailsService
        ↓
PasswordEncoder
```

The missing question is:

> **Who coordinates the actual authentication process?**

That's where these components come in:

```text id="p8j3x4"
AuthenticationManager
        ↓
ProviderManager
        ↓
AuthenticationProvider
        ↓
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
PasswordEncoder
```

Spring's official architecture describes `AuthenticationManager` as the API used by security filters to perform authentication, `ProviderManager` as its most common implementation, and `AuthenticationProvider` as the component that performs a specific type of authentication. ([Home][1])

---

# 1. Why do we need `AuthenticationManager`?

Imagine Spring Security supports different authentication mechanisms:

```text id="49m1w3"
Username + Password
LDAP
JWT
Remember Me
Custom Authentication
```

We don't want every security filter to know how each authentication mechanism works.

We need a common entry point:

```java id="c8n5v7"
AuthenticationManager
```

Its job is essentially:

> **"I received an authentication request. Find a provider that can authenticate it."**

---

# 2. What is `AuthenticationManager`?

It's an interface with the central authentication operation:

```java id="q7c3m4"
Authentication authenticate(
        Authentication authentication)
        throws AuthenticationException;
```

Think:

```text id="x6d4v2"
Authentication Request
        ↓
AuthenticationManager
        ↓
Authentication Result
```

On successful authentication, it returns a fully authenticated `Authentication`; on failure, an authentication exception is raised. The official architecture identifies `AuthenticationManager` as the API used by Spring Security's filters. ([Home][1])

---

# 3. What is `ProviderManager`?

`AuthenticationManager` is an interface.

The most common implementation is:

```java id="t1j5x6"
ProviderManager
```

Its job is to delegate authentication to one or more `AuthenticationProvider`s. ([Home][1])

Think:

```text id="w2k7p9"
AuthenticationManager
        │
        ▼
ProviderManager
        │
        ├── Provider 1
        ├── Provider 2
        └── Provider 3
```

The provider that supports the given authentication type gets the opportunity to authenticate it.

---

# 4. What is `AuthenticationProvider`?

An `AuthenticationProvider` is responsible for **one particular authentication mechanism**.

For example:

```text id="r8m2q5"
AuthenticationProvider
       │
       ├── DaoAuthenticationProvider
       │       ↓
       │   username/password
       │
       ├── JWT-related authentication
       │
       └── Custom provider
```

The important methods are conceptually:

```java id="a9c4t2"
boolean supports(Class<?> authenticationType);

Authentication authenticate(
        Authentication authentication);
```

So a provider answers two questions:

```text id="n4v8q3"
Can I handle this authentication type?

        ↓

How do I authenticate it?
```

---

# 5. What is `DaoAuthenticationProvider`?

Now we reach the provider you'll use in a traditional username/password application.

`DaoAuthenticationProvider` is an `AuthenticationProvider` implementation that uses:

```text id="f3m7k9"
UserDetailsService
+
PasswordEncoder
```

to authenticate username/password credentials. ([Home][2])

The name "DAO" comes from its use of a data-access abstraction (`UserDetailsService`) to retrieve the user.

So:

```text id="m8q2v5"
Username + Password
        ↓
DaoAuthenticationProvider
        ↓
UserDetailsService
        ↓
UserDetails
        ↓
PasswordEncoder
        ↓
Authentication Result
```

---

# 6. Complete Username/Password Authentication

Now let's connect **all six chapters**.

Suppose Rahul logs in:

```text id="z4c7n2"
username = rahul
password = password123
```

The conceptual flow is:

```text id="c2m7r9"
HTTP Request
     ↓
Authentication Filter
     ↓
Authentication object
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
Password valid?
```

If yes:

```text id="j8q3v5"
Authenticated Authentication
        ↓
SecurityContext
        ↓
SecurityContextHolder
```

If no:

```text id="s6k2p8"
AuthenticationException
        ↓
Authentication failure handling
```

This is the complete mental model.

---

# 7. What exactly is the `Authentication` sent into the manager?

For username/password authentication, Spring commonly uses an authentication object carrying the credentials supplied by the user, such as `UsernamePasswordAuthenticationToken`.

Conceptually:

```text id="x4n8c1"
UsernamePasswordAuthenticationToken
├── principal   = rahul
├── credentials = password123
└── authenticated = false
```

The manager receives this authentication request.

After successful authentication, the provider returns an authenticated `Authentication` containing the principal and authorities.

Spring's authentication architecture describes `Authentication` as serving both roles: an input carrying credentials and, after authentication succeeds, the currently authenticated identity. ([Home][1])

---

# 8. Before vs After Authentication

This distinction is important.

### Before

```text id="u5k2p7"
Authentication

principal = rahul
credentials = password123
authenticated = false
```

This is essentially:

> "Please authenticate this user."

---

### After

```text id="y8m3c6"
Authentication

principal = UserDetails
authorities = ROLE_USER
authenticated = true
```

This means:

> "This user has been successfully authenticated."

---

# 9. What does `DaoAuthenticationProvider` actually do?

Conceptually:

```text id="p6w9r2"
Username + Password
        ↓
loadUserByUsername(username)
        ↓
UserDetails
        ↓
Get stored encoded password
        ↓
PasswordEncoder.matches(
    submittedPassword,
    storedPassword
)
        ↓
Success / Failure
```

The official documentation states that `DaoAuthenticationProvider` retrieves user information from `UserDetailsService` and uses `PasswordEncoder` to validate the username/password combination. ([Home][2])

---

# 10. Complete Example Configuration

Now let's configure the pieces we've learned.

```java id="s5m8q2"
@Configuration
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
    DaoAuthenticationProvider
    authenticationProvider(
            UserDetailsService userDetailsService,
            PasswordEncoder passwordEncoder) {

        DaoAuthenticationProvider provider =
                new DaoAuthenticationProvider(
                        userDetailsService
                );

        provider.setPasswordEncoder(
                passwordEncoder
        );

        return provider;
    }

    @Bean
    AuthenticationManager authenticationManager(
            DaoAuthenticationProvider provider) {

        return new ProviderManager(provider);
    }
}
```

The exact configuration details can vary by Spring Security version and auto-configuration, but the architecture remains the same: a provider uses `UserDetailsService` and `PasswordEncoder` to authenticate username/password credentials. ([Home][3])

---

# 11. What is happening here?

Let's break it down.

### PasswordEncoder

```java id="w7p5k2"
@Bean
PasswordEncoder passwordEncoder()
```

Responsible for secure password encoding/matching.

---

### UserDetailsService

```java id="r4c8m6"
@Bean
UserDetailsService userDetailsService(...)
```

Responsible for finding users.

---

### DaoAuthenticationProvider

```java id="n8x2v5"
DaoAuthenticationProvider
```

Combines:

```text id="y2k6q9"
UserDetailsService
+
PasswordEncoder
```

for username/password authentication. ([Home][2])

---

### ProviderManager

```java id="h6m3q8"
new ProviderManager(provider)
```

Acts as the `AuthenticationManager` implementation and delegates authentication to providers. ([Home][1])

---

# 12. What happens if we have multiple providers?

Suppose an enterprise application supports:

```text id="b3q7m1"
Database username/password
LDAP
Custom authentication
```

We could conceptually have:

```text id="v8n2c5"
ProviderManager
     │
     ├── DaoAuthenticationProvider
     │
     ├── LdapAuthenticationProvider
     │
     └── CustomAuthenticationProvider
```

The manager delegates to providers based on whether they support the authentication type. ([Home][1])

You don't need to implement multiple providers at your experience level yet.

But understand **why the abstraction exists**.

---

# 13. `AuthenticationManager` vs `AuthenticationProvider`

This is a classic interview question.

### AuthenticationManager

Think:

> **Coordinator**

```text id="y7p3m1"
"Who should authenticate this?"
```

### AuthenticationProvider

Think:

> **Worker**

```text id="c5q8r2"
"I know how to authenticate this type."
```

So:

```text id="m9v2k6"
AuthenticationManager
        ↓
Provider
        ↓
Actual Authentication Logic
```

---

# 14. `ProviderManager` vs `AuthenticationProvider`

Another common question.

### ProviderManager

```text id="z5r8m3"
Manages/delegates
to providers
```

### AuthenticationProvider

```text id="j2c7n9"
Actually performs
a particular authentication mechanism
```

So:

```text id="x4v6p1"
ProviderManager
      ↓
DaoAuthenticationProvider
      ↓
UserDetailsService
      ↓
PasswordEncoder
```

---

# 15. `UserDetailsService` vs `AuthenticationProvider`

Don't mix these.

### UserDetailsService

```text id="w6m3q9"
Find user
```

### AuthenticationProvider

```text id="a8k2v4"
Authenticate user
```

So:

```text id="z1p7n6"
AuthenticationProvider
        ↓
UserDetailsService
        ↓
Get user
        ↓
Provider validates credentials
```

---

# 16. What happens after successful authentication?

The provider returns an authenticated `Authentication`.

Then Spring Security stores it in:

```text id="h3q9m7"
SecurityContext
       ↓
SecurityContextHolder
```

Now:

```java id="s7k1v4"
SecurityContextHolder
    .getContext()
    .getAuthentication()
```

can tell your application who is authenticated.

Spring's official architecture describes the successful authenticated `Authentication` as the representation of the current user in the `SecurityContext`. ([Home][1])

---

# 17. What happens to the password?

After successful authentication, Spring Security supports erasing sensitive credentials from the resulting authentication object. `ProviderManager` enables credential erasure after authentication by default. ([Home][4])

That's another reason not to write custom authentication carelessly.

---

# 18. Complete Architecture Diagram

This is the diagram you should remember for interviews:

```text id="0qj5x7"
                  LOGIN REQUEST
                  username/password
                         │
                         ▼
                Authentication Filter
                         │
                         ▼
                  Authentication
                         │
                         ▼
               AuthenticationManager
                         │
                         ▼
                   ProviderManager
                         │
                         ▼
              AuthenticationProvider
                         │
                         ▼
             DaoAuthenticationProvider
                    /            \
                   /              \
                  ▼                ▼
       UserDetailsService     PasswordEncoder
                  │                │
                  ▼                │
              UserDetails          │
                  │                │
                  └───────┬────────┘
                          ▼
                 Password Validation
                          │
                    ┌─────┴─────┐
                   YES           NO
                    │             │
                    ▼             ▼
            Authenticated     AuthenticationException
            Authentication
                    │
                    ▼
              SecurityContext
                    │
                    ▼
           SecurityContextHolder
```

This is the architecture you should be able to explain in an interview.

---

# 19. Where does JWT fit later?

JWT will change the **authentication mechanism**, but the concepts you've learned still matter.

With username/password:

```text id="5m7q2n"
Username + Password
    ↓
DaoAuthenticationProvider
    ↓
UserDetailsService
    ↓
PasswordEncoder
```

With JWT:

```text id="8v3k6p"
Bearer Token
    ↓
JWT Validation
    ↓
Authentication
    ↓
SecurityContext
```

So JWT doesn't replace everything you learned.

It changes **how authentication information is established**.

We'll get there after basic authentication and authorization.

---

# 20. Interview Questions

### What is `AuthenticationManager`?

> It is Spring Security's central authentication API used by security filters to authenticate an `Authentication` request. ([Home][1])

### What is `ProviderManager`?

> It is the common implementation of `AuthenticationManager` that delegates authentication to a list of `AuthenticationProvider`s. ([Home][1])

### What is `AuthenticationProvider`?

> It performs a specific type of authentication and indicates whether it supports a given authentication type. ([Home][1])

### What is `DaoAuthenticationProvider`?

> It authenticates username/password credentials by using a `UserDetailsService` to load the user and a `PasswordEncoder` to validate the password. ([Home][2])

### What is the difference between `AuthenticationManager` and `AuthenticationProvider`?

> `AuthenticationManager` coordinates authentication; `AuthenticationProvider` performs the actual mechanism-specific authentication.

### Who loads the user?

```text id="u6w2c4"
UserDetailsService
```

### Who validates the password?

```text id="s1q5n8"
DaoAuthenticationProvider
+
PasswordEncoder
```

---

# 21. Best Practices

For your level:

```text id="p4m8c1"
AuthenticationManager
    ↓
ProviderManager
    ↓
DaoAuthenticationProvider
    ↓
UserDetailsService
    ↓
Repository
```

Keep this separation.

Don't write database queries inside an `AuthenticationProvider`.

Don't manually compare plaintext passwords.

Don't manually put authenticated users into `SecurityContextHolder` unless implementing a custom authentication mechanism and understanding the lifecycle.

Use Spring's authentication infrastructure.

---

# 📍 Where We Are

```text id="v1c7m4"
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
│   ProviderManager
│   AuthenticationProvider
│   DaoAuthenticationProvider
│
└── ⏭️ Chapter 7
      Roles & Authorities ⭐⭐⭐⭐⭐
      ↓
      ROLE_USER / ROLE_ADMIN
      ↓
      GrantedAuthority
      ↓
      hasRole()
      ↓
      hasAuthority()
      ↓
      Real Employee API authorization
```

Next we'll learn **Roles vs Authorities in depth**, then use them to protect our Employee API so that `USER` can read employees while `ADMIN` can create, update, and delete them.

[1]: https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html?utm_source=chatgpt.com "Servlet Authentication Architecture :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html?utm_source=chatgpt.com "DaoAuthenticationProvider :: Spring Security"
[3]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html?utm_source=chatgpt.com "Username/Password Authentication"
[4]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/erasure.html?utm_source=chatgpt.com "Password Erasure - Untitled :: Spring Security"
