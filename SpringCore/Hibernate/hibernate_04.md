

# 1. Generate ID Automatically

Usually, we don't want to manually generate IDs:

```java
Song song = new Song();
song.setId(101);
```

Instead, we can tell JPA/Hibernate to generate the ID automatically.

Use:

```java
@Id
@GeneratedValue(...)
private int id;
```

Example:

```java
@Entity
public class Song {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;
    private String type;
}
```

---

# 8. `GenerationType.IDENTITY`

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

With `IDENTITY`:

> **The database generates the ID, typically using an identity/auto-increment column.**

For MySQL:

```sql
CREATE TABLE song (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    type VARCHAR(100)
);
```

Conceptually:

```text
Hibernate
   ↓
INSERT Song
   ↓
MySQL
   ↓
Generates ID
```

Example:

```text
Song 1 → ID = 1
Song 2 → ID = 2
Song 3 → ID = 3
```

### Remember

```text
IDENTITY
→ Database generates ID
→ Common with MySQL AUTO_INCREMENT
```

---

# 9. `GenerationType.AUTO`

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

`AUTO` tells the JPA provider:

> **Choose an appropriate ID-generation strategy for the database/dialect.**

So don't memorize it as simply:

> "Hibernate is responsible for ID generation."

More accurately:

```text
AUTO
 ↓
JPA provider chooses strategy
 ↓
Based on database/provider capabilities
```

The exact mechanism can vary depending on the Hibernate version and database dialect.

---

# 10. `GenerationType.SEQUENCE`

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

With `SEQUENCE`, a **database sequence** is used to generate IDs.

Conceptually:

```text
Hibernate
   ↓
Database Sequence
   ↓
Next ID
```

For example:

```text
Sequence:
100
101
102
103
```

Databases such as PostgreSQL and Oracle support sequences.

Example:

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE)
private Long id;
```

You can also specify a sequence:

```java
@Id
@GeneratedValue(
    strategy = GenerationType.SEQUENCE,
    generator = "song_seq"
)
@SequenceGenerator(
    name = "song_seq",
    sequenceName = "song_sequence",
    allocationSize = 1
)
private Long id;
```

---

# 11. `GenerationType.TABLE`

```java
@GeneratedValue(strategy = GenerationType.TABLE)
```

With `TABLE`, a **database table is used to keep track of generated ID values**.

Conceptually:

```text
ID Generator Table
-------------------
next_id = 101
```

Hibernate uses this table to obtain/generate identifier values.

### Remember

```text
TABLE
→ Uses a database table
→ Table maintains ID generation information
```

It is less commonly used in modern applications compared with identity or sequence-based strategies.

---

# 12. GenerationType Summary

| Strategy   | Who/What generates ID?           | Common idea                 |
| ---------- | -------------------------------- | --------------------------- |
| `IDENTITY` | Database identity/auto-increment | MySQL `AUTO_INCREMENT`      |
| `AUTO`     | JPA provider chooses strategy    | Provider/database dependent |
| `SEQUENCE` | Database sequence                | PostgreSQL, Oracle, etc.    |
| `TABLE`    | ID generator table               | Uses a database table       |

---

# 13. Complete Example

```java
@Entity
@Table(name = "song")
public class Song {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String type;

    // getters and setters
}
```

Then:

```java
Song song = new Song();

song.setName("Hai");
song.setType("Melody");

session.persist(song);
```

We don't need to do:

```java
song.setId(1);
```

The database generates the ID.

---

# ⭐ Interview Revision

### What are the states of a Hibernate entity?

> **Transient, Managed/Persistent, Detached, and Removed.**

### What is Transient?

> An object created using `new` that is not currently associated with a persistence context.

### What is Managed?

> An entity currently associated with a persistence context/Session and tracked by Hibernate.

### What is Detached?

> An entity that was previously managed but is no longer associated with the current persistence context.

### What is Removed?

> A managed entity that has been marked for deletion.

---

### What is `@GeneratedValue`?

> `@GeneratedValue` specifies that the entity's identifier should be generated automatically according to the selected generation strategy.

### `IDENTITY`

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

> Database identity/auto-increment generates the ID.

### `AUTO`

```java
@GeneratedValue(strategy = GenerationType.AUTO)
```

> The JPA provider chooses an appropriate generation strategy.

### `SEQUENCE`

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

> Uses a database sequence to generate IDs.

### `TABLE`

```java
@GeneratedValue(strategy = GenerationType.TABLE)
```

> Uses a database table to maintain ID-generation information.

---

## 🔥 Easy Memory Trick

```text
IDENTITY → Database Identity / Auto Increment
AUTO     → Provider Automatically chooses
SEQUENCE → Database Sequence
TABLE    → Database Table
```

