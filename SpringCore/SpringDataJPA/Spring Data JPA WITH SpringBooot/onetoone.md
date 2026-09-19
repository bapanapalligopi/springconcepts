Great. Let's continue with **10. `@OneToOne`**.

This one is simpler than `@OneToMany` / `@ManyToOne`.

---

# 10. `@OneToOne`

## 1. What does One-to-One mean?

It means:

> **One record is associated with exactly one record of another entity.**

For example:

```text
User 1 ───── 1 UserProfile
```

A user has one profile, and that profile belongs to one user.

Other examples:

```text
Employee 1 ───── 1 EmployeeDetails
Person   1 ───── 1 Passport
User     1 ───── 1 Address
```

In JPA, `@OneToOne` represents a single-valued association between entities. ([Jakarta EE][1])

---

# 2. Database example

Suppose we have:

```text
users
----------------
id | name
----------------
1  | Rahul
2  | Amit
```

and:

```text
user_profiles
-------------------------
id | phone | user_id
-------------------------
101| 9999  | 1
102| 8888  | 2
```

The relationship is:

```text
users.id
   ↑
   |
user_profiles.user_id
```

Because `user_id` should be unique, one user cannot have multiple profiles.

So you might have:

```text
user_id
---------
1
2
```

but not:

```text
user_id
---------
1
1   ← another profile for same user
```

A one-to-one foreign-key mapping normally uses a unique foreign key (or another one-to-one strategy such as a shared primary key). ([Jakarta EE][2])

---

# 3. Java representation

### User

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

### UserProfile

```java
@Entity
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String phone;
}
```

Conceptually:

```text
User
  |
  | @OneToOne
  ↓
UserProfile
```

---

# 4. What does `@JoinColumn` mean here?

Suppose we have:

```java
@OneToOne
@JoinColumn(name = "profile_id")
private UserProfile profile;
```

This means the `users` table could contain:

```text
users
----------------------
id | name | profile_id
----------------------
1  | Rahul| 101
2  | Amit | 102
```

So:

```text
users.profile_id → user_profiles.id
```

The important difference from our `@ManyToOne` example is that this relationship must enforce **one-to-one**, commonly through a unique foreign key. ([Jakarta EE][2])

---

# 5. Why `unique`?

You may see:

```java
@OneToOne
@JoinColumn(name = "profile_id", unique = true)
private UserProfile profile;
```

`unique = true` means:

```text
profile_id
----------
101
102
```

A value cannot occur twice.

So:

```text
User 1 → Profile 101
User 2 → Profile 102
```

but you can't have:

```text
User 1 → Profile 101
User 2 → Profile 101
```

That would make the same profile belong to two users.

The JPA specification describes one-to-one foreign-key mappings as typically using a unique foreign key. ([Jakarta EE][2])

---

# 6. Bidirectional One-to-One

We can also navigate in both directions.

### User

```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

### UserProfile

```java
@Entity
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String phone;

    @OneToOne(mappedBy = "profile")
    private User user;
}
```

Now:

```text
User
  ↕
UserProfile
```

You can do:

```java
user.getProfile();
```

and:

```java
profile.getUser();
```

---

# 7. What does `mappedBy = "profile"` mean?

Same concept as before.

We have:

```java
User
```

containing:

```java
private UserProfile profile;
```

Therefore the other side says:

```java
@OneToOne(mappedBy = "profile")
private User user;
```

`profile` refers to the field in `User`:

```java
private UserProfile profile;
```

So:

```text
User.profile
      ↑
      |
mappedBy = "profile"
      |
UserProfile.user
```

The `User` side owns the relationship because it contains the `@JoinColumn`.

The inverse side uses `mappedBy`. ([Jakarta EE][2])

---

# 8. Compare with `@ManyToOne`

This is a useful way to understand it.

### Many-to-One

```text
Customer 1
    ↑
    |
    ├── Order 101
    ├── Order 102
    └── Order 103
```

Many orders can point to one customer.

```java
@ManyToOne
private Customer customer;
```

### One-to-One

```text
User 1
   |
   ↓
Profile 1
```

Only one profile is associated with that user.

```java
@OneToOne
private UserProfile profile;
```

---

# 9. Very important: `@OneToOne` doesn't automatically mean database design should be one-to-one

This is a practical point.

Suppose:

```text
User → Address
```

You might think:

> "A user has one address, so I'll use `@OneToOne`."

But what if your business requirement changes later?

```text
User
 ├── Home Address
 ├── Work Address
 └── Previous Address
```

Now it's no longer one-to-one.

So the relationship annotation should represent the **actual business/database relationship**, not just what happens to be true at one moment.

---

# 10. `optional`

You can specify:

```java
@OneToOne(optional = false)
```

This means the relationship is mandatory.

Conceptually:

```text
User MUST have Profile
```

Without `optional = false`, the relationship can generally be absent (`null`). The JPA `OneToOne` annotation defines `optional` for exactly this purpose. ([Jakarta EE][1])

For example:

```java
@OneToOne(optional = false)
@JoinColumn(name = "profile_id", nullable = false, unique = true)
private UserProfile profile;
```

Now you're expressing:

```text
User
 ↓
Profile is required
```

---

# 11. `@OneToOne` and `FetchType`

There's another important point.

For `@OneToOne`, the JPA default fetch strategy is **EAGER**. `LAZY` is allowed as a hint to the persistence provider. ([Jakarta EE][1])

You can write:

```java
@OneToOne(fetch = FetchType.LAZY)
private UserProfile profile;
```

Conceptually:

### EAGER

```text
Load User
   ↓
Load Profile immediately
```

### LAZY

```text
Load User
   ↓
Profile may be loaded when accessed
```

We'll cover **LAZY vs EAGER properly later**, because it's an important JPA topic by itself.

---

# 12. One important difference from `@OneToMany`

With:

```java
@OneToMany
private List<Order> orders;
```

you have a **collection**:

```text
List<Order>
```

With:

```java
@OneToOne
private UserProfile profile;
```

you have a **single object**:

```text
UserProfile
```

That's why the mental model is:

```text
@OneToMany
       ↓
Collection

@OneToOne
       ↓
Single entity
```

---

# 13. The database picture

For example:

```text
users
-----------------------
id | name | profile_id
-----------------------
1  | Rahul| 101
2  | Amit | 102
```

```text
user_profiles
----------------
id  | phone
----------------
101 | 9999
102 | 8888
```

Relationship:

```text
users.profile_id
       |
       ↓
user_profiles.id
```

And because `profile_id` is unique:

```text
One User → One Profile
```

---

# 14. The mental model to remember

Think:

```text
@OneToOne
```

means:

> "This entity has one associated instance of another entity."

For example:

```java
@OneToOne
private UserProfile profile;
```

means:

```text
User
 ↓
one UserProfile
```

And if the database relationship is represented by a foreign key:

```java
@JoinColumn(name = "profile_id")
```

means:

```text
users.profile_id
       ↓
user_profiles.id
```

If it's bidirectional:

```java
// Owner
@OneToOne
@JoinColumn(name = "profile_id")
private UserProfile profile;
```

and:

```java
// Inverse side
@OneToOne(mappedBy = "profile")
private User user;
```

---

## The 4 things to remember

For your level, make sure you can explain:

**1. `@OneToOne`**

```text
One User → One Profile
```

**2. `@JoinColumn`**

```text
Defines the foreign-key column used for the relationship.
```

**3. `mappedBy`**

```text
"I'm not the owner; the other entity's field manages the relationship."
```

**4. Owning side**

```text
The side that maps the foreign key controls the relationship.
```

JPA explicitly defines bidirectional relationships as having an owning side and an inverse side, with the inverse side identifying the owner through `mappedBy`. ([Jakarta EE][3])

Next is **`@ManyToMany`**. That's the one where you'll learn the **join table**, and understanding that will make all four relationship types much easier to visualize.

[1]: https://jakarta.ee/specifications/platform/11/apidocs/jakarta/persistence/onetoone?utm_source=chatgpt.com "OneToOne (Jakarta EE Platform API)"
[2]: https://jakarta.ee/specifications/persistence/4.0/apidocs/jakarta.persistence/jakarta/persistence/onetoone?utm_source=chatgpt.com "OneToOne (Jakarta Persistence API documentation)"
[3]: https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2?utm_source=chatgpt.com "Jakarta Persistence"
