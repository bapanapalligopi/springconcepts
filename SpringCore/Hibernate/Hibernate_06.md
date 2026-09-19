# Hibernate Dirty Checking

**Dirty Checking** is one of the most important Hibernate concepts.

### Simple definition

> **Dirty checking is the process by which Hibernate automatically detects changes made to a managed/persistent entity and generates the required SQL `UPDATE` when the persistence context is flushed.**

In simple words:

> **You change the Java object → Hibernate notices the change → Hibernate updates the database.**

---

# 1. Simple Example

Suppose we have:

```java
@Entity
public class Song {

    @Id
    private int id;

    private String name;
    private String type;
}
```

Now retrieve a Song:

```java
Session session = sessionFactory.openSession();

Transaction tx = session.beginTransaction();

Song song = session.get(Song.class, 1);
```

At this point:

```text
Database
   ↓
session.get()
   ↓
Song Object
   ↓
MANAGED
```

Hibernate is now managing this object.

---

# 2. Change the Object

Now we change its name:

```java
song.setName("New Song");
```

Notice something important:

We **didn't write**:

```java
session.update(song);
```

We also didn't write:

```sql
UPDATE song SET name = 'New Song' WHERE id = 1;
```

We simply changed the Java object.

```text
Before:

song.name = "Old Song"

        ↓

song.setName("New Song")

        ↓

After:

song.name = "New Song"
```

Hibernate detects this change.

That's **Dirty Checking**.

---

# 3. What Happens Internally?

When Hibernate loads an entity, it keeps track of its state.

Conceptually:

```text
Database
   ↓
Song
{
   id = 1
   name = "Old Song"
   type = "Melody"
}
```

Hibernate maintains information that allows it to determine whether the managed entity has changed.

Later:

```java
song.setName("New Song");
```

Now the entity has:

```text
Original state             Current state

name = "Old Song"          name = "New Song"
type = "Melody"            type = "Melody"
```

Hibernate detects that:

```text
"Old Song" != "New Song"
```

So Hibernate considers the entity **dirty**.

---

# 4. What Does "Dirty" Mean?

**Dirty does NOT mean the object is bad or invalid.**

In Hibernate terminology:

> **Dirty means the state of a managed entity has changed compared with the state Hibernate is tracking.**

Example:

```text
Clean:

name = "Java"
type = "Programming"

        ↓

No changes
```

The entity is clean.

But:

```text
name = "Java"
        ↓
name = "Hibernate"
```

The entity is now **dirty**.

---

# 5. When Does Hibernate Generate UPDATE?

Dirty checking happens as part of Hibernate's **flush process**.

For example:

```java
Transaction tx = session.beginTransaction();

Song song = session.get(Song.class, 1);

song.setName("New Song");

tx.commit();
```

Conceptually:

```text
session.get()
     ↓
Managed Entity
     ↓
Change Entity
     ↓
song.setName(...)
     ↓
Hibernate Dirty Checking
     ↓
Flush
     ↓
UPDATE SQL
     ↓
commit
```

The SQL may conceptually look like:

```sql
UPDATE song
SET name = 'New Song'
WHERE id = 1;
```

The exact SQL generated depends on the mapping, Hibernate version, configuration, and database.

---

# 6. Very Important: Dirty Checking Works on Managed Entities

This is one of the most important points.

### Managed entity

```java
Song song = session.get(Song.class, 1);

song.setName("New Name");

transaction.commit();
```

Hibernate can detect the change.

### Detached entity

```java
Song song = session.get(Song.class, 1);

session.close();

song.setName("New Name");
```

Now the entity is **detached**.

Hibernate is no longer tracking it through that Session.

Therefore, changing it does not automatically trigger dirty checking in the closed Session.

---

# 7. Dirty Checking + Session

Remember:

```text
Session
   ↓
Persistence Context
   ↓
Managed Entities
   ↓
Dirty Checking
```

The Session maintains a persistence context containing managed entities.

Example:

```java
Song song = session.get(Song.class, 1);
```

Now:

```text
Persistence Context
--------------------------
Song(id=1)
Song(id=2)
Song(id=3)
```

Hibernate manages these entities and can detect their changes.

---

# 8. Do We Need `session.update()`?

For a managed entity, usually **NO**.

Example:

```java
Song song = session.get(Song.class, 1);

song.setName("New Name");

transaction.commit();
```

You don't need:

```java
session.update(song);
```

Hibernate's dirty checking handles the update.

### Old-style code

You may see:

```java
session.update(song);
```

This is particularly associated with **detached entities and older Hibernate APIs**.

Modern JPA-style Hibernate code generally relies on:

```text
Managed Entity
      ↓
Modify Entity
      ↓
Dirty Checking
      ↓
Flush
      ↓
UPDATE
```

For detached objects, `merge()` is commonly used:

```java
Song managedSong = session.merge(detachedSong);
```

---

# 9. Dirty Checking Example

Let's look at a complete example.

```java
Session session = sessionFactory.openSession();

Transaction tx = session.beginTransaction();

Song song = session.get(Song.class, 1);

System.out.println(song.getName());

// Change the managed entity
song.setName("Hibernate Song");

// No session.update() required

tx.commit();

session.close();
```

What happens?

```text
1. get()
      ↓
2. Song becomes MANAGED
      ↓
3. setName()
      ↓
4. Hibernate detects change
      ↓
5. Flush
      ↓
6. UPDATE SQL
      ↓
7. commit()
```

---

# 10. What If Nothing Changes?

Suppose:

```java
Song song = session.get(Song.class, 1);

System.out.println(song.getName());

tx.commit();
```

You didn't change anything.

Hibernate has no entity change to synchronize.

Conceptually:

```text
Original:
name = "Java"

Current:
name = "Java"

        ↓

No change
        ↓
No UPDATE needed
```

---

# 11. Dirty Checking and `flush()`

A very important concept is **flush**.

`flush()` synchronizes the current persistence context with the database.

Example:

```java
song.setName("New Name");

session.flush();
```

Hibernate may generate the required `UPDATE` during the flush.

Then:

```java
transaction.commit();
```

commits the transaction.

### Remember

```text
Dirty Checking
     ↓
Detect changes
     ↓
Flush
     ↓
SQL generated/executed
     ↓
Commit
```

**Dirty checking and `commit()` are not exactly the same thing.**

Dirty checking determines what changed; flushing synchronizes those changes with the database. A transaction commit normally causes a flush under the usual configuration.

---

# 12. Why Is Dirty Checking Useful?

Without dirty checking, you might have to manually tell Hibernate every time something changed:

```java
song.setName("New Name");
session.update(song);
```

With managed entities, Hibernate can do:

```java
song.setName("New Name");
```

and detect the change automatically during flush.

This reduces boilerplate code.

---

# ⭐ Interview Answer

If an interviewer asks:

### "What is dirty checking in Hibernate?"

You can answer:

> **Dirty checking is a Hibernate mechanism that automatically detects changes made to managed entities. During flush, Hibernate compares the entity's tracked state with its current state and generates the necessary SQL statements, such as `UPDATE`, to synchronize the database.**

### Simple example:

```java
Song song = session.get(Song.class, 1);

song.setName("New Name");

transaction.commit();
```

You don't explicitly call:

```java
session.update(song);
```

Hibernate detects the modification and synchronizes it with the database.

---

## 🔥 Remember This Diagram

```text
          session.get()
               ↓
        ┌──────────────┐
        │ Managed      │
        │ Entity       │
        └──────┬───────┘
               ↓
      Change Java Object
               ↓
     song.setName(...)
               ↓
       DIRTY CHECKING
               ↓
       Change detected
               ↓
             FLUSH
               ↓
          UPDATE SQL
               ↓
            COMMIT
               ↓
           DATABASE
```

### One-line memory trick:

> **Managed Entity + Change + Flush = Hibernate automatically synchronizes the change with the database.**
