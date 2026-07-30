# Spring Security — Chapter 5: `PasswordEncoder` and BCrypt

Now we reach one of the **most important security topics**.

We know:

```text
UserDetails
      ↓
UserDetailsService
      ↓
User information
```

But there is one critical question:

> **How should the user's password be stored and compared securely?**

That's the job of `PasswordEncoder`.

Spring Security's `PasswordEncoder` is specifically designed for **one-way password transformations** so passwords can be stored securely and later compared with a submitted password. It is not intended for reversible encryption. ([Home][1])

---

# 1. Why do we need `PasswordEncoder`?

Imagine this database:

```text
username    password
----------------------
rahul       password123
admin       admin123
```

This is **plain-text password storage**.

If the database is compromised:

```text
Attacker
   ↓
Database
   ↓
password123
admin123
```

The actual passwords are immediately exposed.

We should never do this.

---

# 2. What should we store?

Instead of:

```text id="v41r2q"
password123
```

we store an encoded password such as:

```text
$2a$10$...
```

So:

```text id="u7kv4x"
Raw Password
    ↓
PasswordEncoder
    ↓
Encoded Password
    ↓
Database
```

Spring Security recommends adaptive one-way password hashing approaches such as BCrypt, PBKDF2, and SCrypt; its current documentation also recommends `DelegatingPasswordEncoder` for password upgrades. ([Home][2])

---

# 3. Is PasswordEncoder encryption?

**No.**

This is a very common interview question.

### Encryption

Encryption is generally:

```text
Plain Text
   ↓
Encryption + Key
   ↓
Cipher Text
   ↓
Decryption + Key
   ↓
Plain Text
```

It's designed to be reversible when you have the key.

### Password hashing/encoding

Conceptually:

```text
Password
   ↓
One-way transformation
   ↓
Encoded Password
```

The application does **not** recover the original password from the stored hash.

Spring's `PasswordEncoder` documentation explicitly describes the transformation as one-way. ([Home][1])

So:

> **Passwords should be hashed/encoded, not reversibly encrypted for normal password storage.**

---

# 4. What is `PasswordEncoder`?

It's an interface:

```java id="5dbm6s"
PasswordEncoder
```

The important methods are:

```java id="4cxfj2"
String encode(CharSequence rawPassword);

boolean matches(
        CharSequence rawPassword,
        String encodedPassword);
```

Spring documents `matches` as checking the submitted raw password against the encoded password from storage; the stored password itself is not decoded. ([Home][3])

---

# 5. `encode()`

During user registration:

```java id="f3r7u8"
String encoded =
        passwordEncoder.encode(
                "password123"
        );
```

You store:

```text id="h9s4ko"
encoded
```

not:

```text id="b8x1t3"
password123
```

---

# 6. `matches()`

During login:

User enters:

```text id="f49r3u"
password123
```

Database contains:

```text id="e8v5s2"
$2a$10$...
```

Spring does conceptually:

```java id="wp2b7x"
passwordEncoder.matches(
    "password123",
    storedEncodedPassword
);
```

Result:

```text id="q8t0v5"
true
```

or:

```text id="y3s1c7"
false
```

---

# 7. Why don't we encode the login password and compare strings?

This is an important detail.

Suppose:

```java id="1r6v4d"
passwordEncoder.encode("password123");
```

produces:

```text id="hh7q2n"
$2a$10$ABC...
```

You might think the next call will produce exactly the same string.

But password hashing algorithms such as BCrypt use a random salt, so encoding the same password again can produce a **different encoded string**.

That's why you use:

```java id="pj3j1m"
matches(rawPassword, storedEncodedPassword)
```

instead of:

```java id="d2y7xq"
encode(rawPassword).equals(storedEncodedPassword)
```

BCrypt uses a random salt and is deliberately slow to make password cracking more expensive. ([Home][4])

---

# 8. What is BCrypt?

BCrypt is an adaptive password-hashing algorithm.

Spring Security provides:

```java id="e7s2a9"
BCryptPasswordEncoder
```

The implementation uses BCrypt and supports a configurable work factor/strength. The current API documentation states that the default strength is 10, and increasing the strength increases the work required exponentially. ([Home][5])

---

# 9. Why is BCrypt intentionally slow?

You might think:

> "Why would we want password hashing to be slow?"

Because attackers may try:

```text id="5qj4sn"
password
123456
qwerty
password1
...
```

Millions or billions of guesses.

A deliberately expensive password hash makes each guess more costly.

So:

```text id="6m9t03"
Faster hashing
→ better for general checksums
→ bad for password storage
```

while:

```text id="7z2n1v"
Adaptive, deliberately expensive hashing
→ better suited for passwords
```

Spring's documentation explicitly notes that BCrypt is deliberately slow and its work factor can be tuned. ([Home][4])

---

# 10. Creating a BCryptPasswordEncoder

```java id="s5r6m2"
@Bean
PasswordEncoder passwordEncoder() {

    return new BCryptPasswordEncoder();
}
```

Now Spring can inject:

```java id="m4q7c1"
PasswordEncoder
```

wherever you need it.

---

# 11. Registration Flow

Suppose user registers:

```json id="u8k2p4"
{
  "username": "rahul",
  "password": "password123"
}
```

Service:

```java id="x2n7m4"
@Service
public class UserService {

    private final PasswordEncoder passwordEncoder;

    public UserService(
            PasswordEncoder passwordEncoder) {

        this.passwordEncoder = passwordEncoder;
    }

    public void register(UserRequest request) {

        String encodedPassword =
                passwordEncoder.encode(
                        request.getPassword()
                );

        // Save encodedPassword in database
    }
}
```

Database:

```text id="p3v8y9"
username = rahul
password = $2a$10$...
```

---

# 12. Login Flow

Suppose Rahul submits:

```text id="g9k3w1"
username = rahul
password = password123
```

Spring Security eventually performs a password comparison using:

```java id="n2v4r8"
passwordEncoder.matches(
        rawPassword,
        storedEncodedPassword
);
```

Conceptually:

```text id="j8m6w2"
Login Password
     ↓
matches()
     ↓
Stored Encoded Password
     ↓
true / false
```

---

# 13. Complete Authentication Flow

Now connect the previous chapters:

```text id="m6p9q2"
Username + Password
        ↓
Authentication Filter
        ↓
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService
        ↓
UserDetails
        ↓
PasswordEncoder.matches()
        ↓
Password Correct?
     ┌──────┴──────┐
    YES           NO
     │             │
     ▼             ▼
Authenticated    Failure
     │
     ▼
SecurityContext
     │
     ▼
SecurityContextHolder
```

We're getting very close to the complete username/password authentication architecture.

---

# 14. `BCryptPasswordEncoder` Example

```java id="g2j8k4"
@Configuration
public class SecurityConfig {

    @Bean
    PasswordEncoder passwordEncoder() {

        return new BCryptPasswordEncoder();
    }
}
```

Then:

```java id="t7m1c5"
@Service
public class UserService {

    private final PasswordEncoder passwordEncoder;

    public UserService(
            PasswordEncoder passwordEncoder) {

        this.passwordEncoder = passwordEncoder;
    }

    public String encodePassword(
            String rawPassword) {

        return passwordEncoder.encode(
                rawPassword);
    }
}
```

---

# 15. Important: BCrypt Produces Different Encodings

For example:

```java id="w8r3y1"
passwordEncoder.encode("password123");
```

might produce one value.

Calling it again:

```java id="q1t6m7"
passwordEncoder.encode("password123");
```

may produce a different value.

That's expected because BCrypt uses a random salt. ([Home][4])

But:

```java id="h4v9s2"
passwordEncoder.matches(
    "password123",
    storedHash
);
```

will still return `true`.

---

# 16. What is the salt?

A salt is random data incorporated into password hashing.

Conceptually:

```text id="n5z3r7"
Password
   +
Random Salt
   ↓
BCrypt
   ↓
Encoded Password
```

Why?

Without salt, identical passwords would tend to have identical hashes.

With salt:

```text id="s4c9k2"
Rahul → hash A
Amit  → hash B
```

even if both users happen to use the same password.

BCrypt itself incorporates a salt into its encoded representation. ([Home][4])

---

# 17. Why not SHA-256 for Password Storage?

This is an important conceptual question.

A general-purpose hash like SHA-256 is designed to be very fast.

For passwords, being extremely fast is not necessarily desirable because attackers can test huge numbers of candidate passwords quickly.

Spring Security's documentation specifically recommends adaptive one-way functions such as BCrypt, PBKDF2, and SCrypt rather than insecure legacy message-digest approaches. ([Home][2])

At your level, remember:

```text id="9j5k1r"
General-purpose hash
→ fast

Password hash
→ deliberately expensive/adaptive
```

---

# 18. What is `DelegatingPasswordEncoder`?

Spring Security also provides:

```java id="h2f7m4"
DelegatingPasswordEncoder
```

It can choose the appropriate encoder based on an identifier/prefix stored with the encoded password. This supports multiple password encodings and makes password-algorithm upgrades easier. ([Home][6])

Example stored value:

```text
{bcrypt}$2a$10$...
```

The prefix:

```text
{bcrypt}
```

identifies which encoder should be used.

This is particularly useful when migrating an existing application from an older password-hashing scheme to a stronger one.

---

# 19. Should you use `NoOpPasswordEncoder`?

For production:

**No.**

It stores passwords without secure hashing.

Spring explicitly describes `NoOpPasswordEncoder` as insecure. ([Home][7])

You may see:

```java id="v8k3m1"
{noop}password
```

in simple demos, but don't copy that into production.

---

# 20. `User.withDefaultPasswordEncoder()`

You may encounter examples like:

```java id="y9x4s2"
User.withDefaultPasswordEncoder()
```

Spring's documentation warns that this is intended only for getting-started examples and should not be used for production; it does not solve the problem of keeping credentials out of source code. ([Home][8])

For your learning project, prefer an explicit:

```java id="j5m2c8"
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

---

# 21. Password Storage Architecture

A production-style system looks like:

```text id="d7q3m1"
User Registration
      ↓
Raw Password
      ↓
PasswordEncoder
      ↓
Encoded Password
      ↓
Database
```

Login:

```text id="s8k4p2"
Raw Password
      ↓
AuthenticationProvider
      ↓
PasswordEncoder.matches()
      ↓
Stored Encoded Password
      ↓
Success / Failure
```

---

# 22. What should NEVER happen?

### Don't do this:

```java id="f2j5q8"
user.setPassword(
        request.getPassword()
);
```

and save it directly.

### Don't do this:

```java id="m8r4k1"
System.out.println(
        request.getPassword());
```

### Don't do this:

```java id="q6n3v9"
passwordEncoder.encode(...)
        .equals(storedPassword);
```

Use:

```java id="e4w7m2"
passwordEncoder.matches(
        rawPassword,
        storedPassword
);
```

---

# 23. Interview Questions

### What is `PasswordEncoder`?

> It is Spring Security's abstraction for securely transforming passwords into one-way encoded forms and verifying submitted passwords against stored encoded values. ([Home][1])

### Is BCrypt encryption?

> No. BCrypt is a password-hashing algorithm with a deliberately expensive work factor.

### Why doesn't `encode()` return the same value every time?

> Because BCrypt incorporates a random salt, so the same raw password can produce different encoded values. ([Home][4])

### How does Spring verify a password?

```java id="h7v2m4"
passwordEncoder.matches(
    rawPassword,
    encodedPassword
);
```

### Why shouldn't we use SHA-256 directly for password storage?

> General-purpose digest algorithms are designed for speed, whereas password hashing should use adaptive, deliberately expensive algorithms such as BCrypt, PBKDF2, or SCrypt. ([Home][2])

### What is `DelegatingPasswordEncoder`?

> It chooses a password encoder based on an identifier stored with the encoded password and supports gradual migration/upgrades between encoding schemes. ([Home][6])

---

# 24. Best Practices

```text id="v6m2q9"
✅ Use PasswordEncoder
✅ Use BCrypt or another modern adaptive password hash
✅ Store only encoded passwords
✅ Use matches() for verification
✅ Never log passwords
✅ Never return passwords in API responses
✅ Use DelegatingPasswordEncoder where algorithm migration matters
❌ Never use plaintext passwords
❌ Never use NoOpPasswordEncoder in production
```

Spring's current documentation also highlights adaptive hashing and `DelegatingPasswordEncoder` as the preferred direction for secure password storage and future upgrades. ([Home][2])

---

# 📍 Where We Are

```text id="v8x2m1"
Spring Security
│
├── ✅ Chapter 1
│   Why Security?
│   Authentication vs Authorization
│
├── ✅ Chapter 2
│   SecurityFilterChain
│   FilterChainProxy
│   Authorization Rules
│
├── ✅ Chapter 3
│   Authentication
│   SecurityContext
│   SecurityContextHolder
│
├── ✅ Chapter 4
│   UserDetails
│   UserDetailsService
│
├── ✅ Chapter 5
│   PasswordEncoder
│   BCrypt
│   encode()
│   matches()
│   DelegatingPasswordEncoder
│
└── ⏭️ Chapter 6
      AuthenticationManager ⭐⭐⭐⭐⭐
      ↓
      ProviderManager
      ↓
      AuthenticationProvider
      ↓
      DaoAuthenticationProvider
      ↓
      Complete Username/Password Authentication Flow
```

Next we'll connect **`AuthenticationManager` + `AuthenticationProvider` + `UserDetailsService` + `PasswordEncoder`** into one complete authentication pipeline. This is the chapter where the individual pieces finally come together.

[1]: https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html?utm_source=chatgpt.com "Password Storage :: Spring Security"
[2]: https://docs.spring.io/spring-security/reference/api/java/deprecated-list.html?utm_source=chatgpt.com "Deprecated List (spring-security-docs 7.1.0 API)"
[3]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/crypto/password/PasswordEncoder.html?utm_source=chatgpt.com "PasswordEncoder (spring-security-docs 7.1.0 API)"
[4]: https://docs.spring.io/spring-security/reference/features/integrations/cryptography.html?utm_source=chatgpt.com "Spring Security Crypto Module"
[5]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/crypto/bcrypt/BCryptPasswordEncoder.html?utm_source=chatgpt.com "BCryptPasswordEncoder (spring-security-docs 7.1.0 API)"
[6]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/crypto/password/DelegatingPasswordEncoder.html?utm_source=chatgpt.com "DelegatingPasswordEncoder (spring-security-docs 7.1.0 API)"
[7]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/crypto/password/NoOpPasswordEncoder.html?utm_source=chatgpt.com "NoOpPasswordEncoder (spring-security-docs 7.1.0 API)"
[8]: https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html?utm_source=chatgpt.com "In-Memory Authentication :: Spring Security"
