# Spring Security — Chapter 4: `UserDetails` and `UserDetailsService`

Now we connect the concepts from the previous chapter to actual user data.

We currently know:

```text id="r8e7m2"
Authentication
      ↓
SecurityContext
      ↓
SecurityContextHolder
```

The next question is:

> **Where does Spring Security get the user's username, password, account status, and authorities from?**

That's where `UserDetails` and `UserDetailsService` come in.

We'll follow:

> **Why → What → How → Where → Internal Flow → Code → Interview Questions → Best Practices**

---

# 1. Why do we need `UserDetails`?

Suppose Rahul logs in:

```text
Username: rahul
Password: ********
```

Spring Security needs information about Rahul:

```text
username
password
roles
enabled?
account locked?
account expired?
credentials expired?
```

Spring Security uses a standard abstraction for this user information:

```java
UserDetails
```

---

# 2. What is `UserDetails`?

`UserDetails` is an interface representing the user's account information used by Spring Security.

Conceptually:

```text id="af9s3d"
UserDetails
├── username
├── password
├── authorities
├── enabled
├── accountNonExpired
├── accountNonLocked
└── credentialsNonExpired
```

Spring's `UserDetailsService` uses `UserDetails` as the standard user representation for username/password authentication.

---

# 3. Why not just use our `Employee` class?

We already have:

```java id="q7f0d3"
public class Employee {

    private Long id;
    private String name;
    private String email;
    private BigDecimal salary;
}
```

But authentication needs security-specific information:

```text id="4v4mj8"
username
password
authorities
account status
```

Your business entity and your security user aren't necessarily the same thing.

For example:

```text id="f5h8j2"
Employee
├── id
├── name
├── email
├── salary
└── department
```

Security user:

```text id="k6z1q3"
UserDetails
├── username
├── password
├── authorities
└── account status
```

They solve different problems.

---

# 4. What is `UserDetailsService`?

Now the next question:

> **Who loads `UserDetails`?**

Spring Security provides:

```java id="q4r1m8"
UserDetailsService
```

Its main method is:

```java id="v7s2k9"
UserDetails loadUserByUsername(String username)
        throws UsernameNotFoundException;
```

Its job is simple:

> **Given a username, find the user and return the user's security information.**

---

# 5. Real-Life Analogy

Think of a company database.

Security gets:

```text id="9q3m1c"
Username = rahul
```

and asks:

> "Give me Rahul's account details."

`UserDetailsService` does the lookup.

```text id="v2b7k6"
username
   ↓
UserDetailsService
   ↓
User store
   ↓
UserDetails
```

The user store could be:

```text id="e4p8t9"
Database
LDAP
In-memory
Custom service
```

---

# 6. In-Memory User

For learning, Spring Security provides:

```java id="h5n3s7"
InMemoryUserDetailsManager
```

Example:

```java id="z2q8w1"
@Bean
UserDetailsService userDetailsService(
        PasswordEncoder passwordEncoder) {

    UserDetails user =
            User.withUsername("rahul")
                    .password(
                        passwordEncoder.encode("password123")
                    )
                    .roles("USER")
                    .build();

    UserDetails admin =
            User.withUsername("admin")
                    .password(
                        passwordEncoder.encode("admin123")
                    )
                    .roles("ADMIN")
                    .build();

    return new InMemoryUserDetailsManager(
            user,
            admin
    );
}
```

Now Spring knows about two users:

```text id="v9h6k4"
rahul
  ↓
ROLE_USER

admin
  ↓
ROLE_ADMIN
```

---

# 7. Why do we need `PasswordEncoder`?

Notice this:

```java id="e6m2x7"
.password(
    passwordEncoder.encode("password123")
)
```

We never store:

```text id="t3k9p2"
password123
```

as plaintext.

Instead:

```text id="a6j1w4"
password123
      ↓
PasswordEncoder
      ↓
Encoded password
```

We'll cover `PasswordEncoder` deeply in the next chapter.

For now, remember:

> **Always use a password encoder; never store plaintext passwords.**

---

# 8. Roles

We used:

```java id="p5w8c2"
.roles("USER")
```

Spring treats this as:

```text id="z3m7q1"
ROLE_USER
```

and:

```java id="g4x9r5"
.roles("ADMIN")
```

becomes:

```text id="s7k2d8"
ROLE_ADMIN
```

Then:

```java id="w1q5n9"
.hasRole("ADMIN")
```

can match:

```text id="u3c8v6"
ROLE_ADMIN
```

---

# 9. Authorities

You can also define explicit authorities:

```java id="q9j4m3"
UserDetails user =
        User.withUsername("rahul")
                .password(
                    passwordEncoder.encode("password123")
                )
                .authorities(
                    "employee:read",
                    "employee:update"
                )
                .build();
```

Now the user has:

```text id="x6k2f8"
employee:read
employee:update
```

This is more granular than a role.

---

# 10. Role vs Authority

You've already seen the concept.

### Role

Broad category:

```text id="c8m1x6"
ROLE_USER
ROLE_ADMIN
```

### Authority

Specific permission:

```text id="n5q7b2"
employee:read
employee:create
employee:update
employee:delete
```

For example:

```java id="a1p6y8"
.hasRole("ADMIN")
```

versus:

```java id="s7q3k2"
.hasAuthority("employee:delete")
```

We'll cover role/authority design separately.

---

# 11. How does `UserDetailsService` fit into Authentication?

Now connect the pieces.

Suppose Rahul sends:

```text id="h4n9z2"
username = rahul
password = password123
```

Conceptually:

```text id="p2v7c4"
Login Request
     ↓
Authentication Filter
     ↓
AuthenticationManager
     ↓
AuthenticationProvider
     ↓
UserDetailsService
     ↓
loadUserByUsername("rahul")
     ↓
UserDetails
```

Then the authentication provider compares the supplied password with the stored encoded password.

If successful:

```text id="m8r2d1"
Authentication
     ↓
SecurityContext
     ↓
SecurityContextHolder
```

Spring's standard username/password architecture uses `UserDetailsService` as the source from which an `AuthenticationProvider` obtains user information.

---

# 12. Database-backed `UserDetailsService`

In a real application, users usually come from a database.

Example table:

```sql id="q7y3m9"
CREATE TABLE app_user (
    id BIGINT PRIMARY KEY,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    enabled BOOLEAN NOT NULL
);
```

Example data:

```text id="r4n8k2"
username | password_hash | role
-----------------------------------
rahul    | $2a$...       | USER
admin    | $2a$...       | ADMIN
```

Then we create:

```java id="c5w1p7"
@Service
public class CustomUserDetailsService
        implements UserDetailsService {
```

and implement:

```java id="j3m8q4"
@Override
public UserDetails loadUserByUsername(
        String username)
        throws UsernameNotFoundException {

    // Find user in database

    // Convert to UserDetails

}
```

---

# 13. Complete Database-backed Example

Suppose we have:

```java id="u6k2x9"
public class AppUser {

    private Long id;
    private String username;
    private String password;
    private String role;
    private boolean enabled;

    // getters/setters
}
```

Service:

```java id="p9v4m7"
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    private final UserRepository repository;

    public CustomUserDetailsService(
            UserRepository repository) {

        this.repository = repository;
    }

    @Override
    public UserDetails loadUserByUsername(
            String username)
            throws UsernameNotFoundException {

        AppUser user =
                repository.findByUsername(username);

        if (user == null) {

            throw new UsernameNotFoundException(
                    "User not found: " + username
            );
        }

        return User.withUsername(
                    user.getUsername())
                .password(user.getPassword())
                .roles(user.getRole())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

The important conversion is:

```text id="t7c5n1"
Database User
      ↓
UserDetails
```

---

# 14. Why throw `UsernameNotFoundException`?

If the requested username doesn't exist:

```java id="w2k8j4"
throw new UsernameNotFoundException(...)
```

This tells Spring Security:

> The user could not be found.

Don't return `null`.

The expected contract of `loadUserByUsername` is to return a user or throw `UsernameNotFoundException` when the user cannot be found.

---

# 15. Custom `UserDetails`

You don't always have to use Spring's built-in `User`.

You can create:

```java id="n3q7c8"
public class CustomUserDetails
        implements UserDetails {

    private final AppUser user;

    public CustomUserDetails(AppUser user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority>
    getAuthorities() {

        return List.of(
                new SimpleGrantedAuthority(
                        "ROLE_" + user.getRole()
                )
        );
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }
}
```

Then:

```java id="j1r6w2"
return new CustomUserDetails(user);
```

This is useful when you need additional application-specific user information.

---

# 16. `User` vs `UserDetails`

Spring provides a convenient implementation:

```java id="k4m8s1"
org.springframework.security.core.userdetails.User
```

So for simple projects:

```java id="y5v2p9"
User.withUsername(...)
```

is enough.

For more complex systems:

```text id="x8q3n6"
CustomUserDetails
       ↓
App-specific information
```

may be useful.

---

# 17. Where is `UserDetailsService` used?

Common sources:

```text id="p6r2t8"
Database
LDAP
In-memory
External identity system
Custom API
```

For a normal Spring Boot application with users stored in MySQL/PostgreSQL:

```text id="u7m4k1"
Database
   ↓
Repository
   ↓
UserDetailsService
   ↓
UserDetails
```

---

# 18. Internal Authentication Flow

Let's put everything together.

```text id="b5q8m2"
User enters:

rahul
password123

        ↓

Authentication Filter

        ↓

AuthenticationManager

        ↓

AuthenticationProvider

        ↓

UserDetailsService

        ↓

loadUserByUsername("rahul")

        ↓

Database

        ↓

UserDetails

        ↓

PasswordEncoder

        ↓

Password Match?

     ┌───────┴────────┐
     │                │
    YES              NO
     │                │
     ▼                ▼
Authentication     Failure
     │
     ▼
SecurityContext
     │
     ▼
SecurityContextHolder
```

Don't worry about `AuthenticationManager` and `AuthenticationProvider` deeply yet—we'll dedicate the next chapter to them.

---

# 19. `UserDetailsService` Does NOT Authenticate by Itself

This is an important interview trap.

Many beginners say:

> "`UserDetailsService` authenticates the user."

Not exactly.

Its job is:

> **Load user information.**

The authentication process is coordinated by the authentication infrastructure, such as `AuthenticationManager` and `AuthenticationProvider`.

So:

```text id="f2q9c5"
UserDetailsService
      ↓
Find user

AuthenticationProvider
      ↓
Perform authentication logic
```

---

# 20. `UserDetailsService` vs Repository

They are also different.

### Repository

```text id="g1v6p8"
Database Access
```

### UserDetailsService

```text id="m3r7k2"
Security-specific user lookup
```

Architecture:

```text id="q8x5n1"
AuthenticationProvider
       ↓
UserDetailsService
       ↓
UserRepository
       ↓
Database
```

The service adapts application user data into Spring Security's expected `UserDetails` format.

---

# 21. Interview Questions

### What is `UserDetails`?

> It is Spring Security's abstraction representing the user account information needed for authentication and authorization, including username, password, authorities, and account-status flags.

### What is `UserDetailsService`?

> It is a service interface used by Spring Security to load user-specific data by username.

### Does `UserDetailsService` authenticate the user?

> No. It loads user information. Authentication is performed by the authentication infrastructure, typically through `AuthenticationManager` and an `AuthenticationProvider`.

### What happens if the user isn't found?

> `loadUserByUsername` should throw `UsernameNotFoundException`.

### Where can `UserDetailsService` get users from?

> Database, in-memory storage, LDAP, or another user store.

### Why not return the Employee entity directly as `UserDetails`?

> Because `UserDetails` is a security abstraction, while the Employee entity is a business/persistence model. A separate security representation keeps responsibilities cleaner.

---

# 22. Best Practices

For your level:

```text id="e3m7q8"
Controller
    ↓
Authentication infrastructure
    ↓
UserDetailsService
    ↓
Repository
    ↓
Database
```

Keep security lookup logic separate from controllers.

Never return plaintext passwords.

Never log passwords.

Store encoded passwords using a strong adaptive password hashing algorithm.

We'll learn the password-storage details in the next chapter.

---

# 📍 Where We Are

```text id="c7v4n1"
Spring Security
│
├── ✅ Chapter 1
│   Why Security?
│   Authentication vs Authorization
│
├── ✅ Chapter 2
│   SecurityFilterChain
│   FilterChainProxy
│   HttpSecurity
│   Authorization Rules
│
├── ✅ Chapter 3
│   Authentication
│   Principal
│   SecurityContext
│   SecurityContextHolder
│
├── ✅ Chapter 4
│   UserDetails
│   UserDetailsService
│   InMemoryUserDetailsManager
│   Database-backed UserDetailsService
│
└── ⏭️ Chapter 5
      PasswordEncoder ⭐⭐⭐⭐⭐
      ↓
      BCrypt
      ↓
      Password hashing
      ↓
      Password matching
      ↓
      Password storage best practices
```

Next is **`PasswordEncoder` and BCrypt**. This is where we'll understand exactly how passwords should be stored, why encryption is the wrong concept for password storage, how `encode()` differs from `matches()`, and how to configure it correctly in Spring Security.
