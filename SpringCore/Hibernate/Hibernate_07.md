# Hibernate `update()` vs `merge()`

The easiest way to understand this is through **managed vs detached objects**.

```text
Managed Entity
    ↓
Modify it
    ↓
Dirty Checking
    ↓
No update()/merge() needed
```

But if you have a **detached entity**, then `update()` and `merge()` become relevant.

---

# 1. `update()`

`update()` is a **legacy Hibernate API** used to reassociate a detached object with a Session.

Example:

```java
Song song = session1.get(Song.class, 1);

session1.close();   // song becomes DETACHED

song.setName("New Song");

Session session2 = sessionFactory.openSession();

Transaction tx = session2.beginTransaction();

session2.update(song);

tx.commit();
session2.close();
```

Conceptually:

```text
Session 1
   ↓
get()
   ↓
Managed Song
   ↓
session.close()
   ↓
Detached Song
   ↓
update(song)
   ↓
Managed by Session 2
```

### Important limitation

`update()` associates that **same object instance** with the Session.

If the Session already contains another managed object with the same identifier, you can run into:

```text
NonUniqueObjectException
```

For example:

```java
Song managedSong = session.get(Song.class, 1);

Song detachedSong = ...; // also has id = 1

session.update(detachedSong);
```

Now the Session already has another `Song` with ID `1`.

---

# 2. `merge()`

`merge()` is the preferred JPA-style way to handle the state of a detached entity.

Example:

```java
Song song = session1.get(Song.class, 1);

session1.close();

song.setName("New Song");

Session session2 = sessionFactory.openSession();

Transaction tx = session2.beginTransaction();

Song managedSong = session2.merge(song);

tx.commit();
session2.close();
```

The important thing about `merge()` is:

> **It copies the state of the detached object into a managed instance.**

It does **not necessarily make the original object managed**.

Conceptually:

```text
Detached Song
     |
     | merge()
     ↓
Managed Song
     |
     ↓
Session
```

So:

```java
Song managedSong = session.merge(detachedSong);
```

The returned object is the one you should use as the managed entity.

---

# 3. Biggest Difference

This is the most important point:

### `update()`

```java
session.update(song);
```

The **same object** is associated with the Session.

```text
detachedSong
     ↓ update()
managedSong

Same object
```

### `merge()`

```java
Song managedSong = session.merge(song);
```

Hibernate copies the state into a **managed object**.

```text
detachedSong
     ↓ merge()
managedSong

Different object can be returned
```

Therefore:

> **`update()` re-associates the given instance, while `merge()` copies its state to a managed instance and returns that managed instance.**

---

# 4. Example to Understand `merge()`

```java
Song detachedSong = new Song();
detachedSong.setId(1);
detachedSong.setName("New Song");

Song managedSong = session.merge(detachedSong);
```

After this:

```text
detachedSong
     |
     | state copied
     ↓
managedSong
     |
     ↓
Persistence Context
```

So don't assume:

```java
detachedSong == managedSong
```

They can be different Java objects.

---

# 5. `update()` vs `merge()`

| `update()`                                                      | `merge()`                                         |
| --------------------------------------------------------------- | ------------------------------------------------- |
| Hibernate-specific/legacy API                                   | JPA standard API                                  |
| Used with detached entity                                       | Commonly used with detached entity                |
| Reassociates the given object                                   | Copies state to a managed object                  |
| Same object becomes associated                                  | Returned object is managed                        |
| Can cause `NonUniqueObjectException` if same ID already managed | Handles existing managed instance more gracefully |
| Does not return the managed entity                              | Returns managed entity                            |
| Less preferred in modern JPA-style code                         | Generally preferred                               |

---

# 6. What Should You Use?

For modern JPA/Hibernate applications:

```java
session.merge(detachedSong);
```

is generally preferred over:

```java
session.update(detachedSong);
```

But remember an even more important point:

### If the entity is already managed, use neither.

```java
Song song = session.get(Song.class, 1);

song.setName("New Name");

transaction.commit();
```

Hibernate's **dirty checking** handles the update.

```text
Managed Entity
      ↓
Modify
      ↓
Dirty Checking
      ↓
Flush
      ↓
UPDATE
```

---

# 7. Complete Comparison Example

### Scenario A — Managed Entity

```java
Song song = session.get(Song.class, 1);

song.setName("New Name");

transaction.commit();
```

No `update()` or `merge()` needed.

---

### Scenario B — Detached Entity + `update()`

```java
Song song = session1.get(Song.class, 1);

session1.close();

song.setName("New Name");

session2.update(song);
```

The same `song` object is reassociated.

---

### Scenario C — Detached Entity + `merge()`

```java
Song song = session1.get(Song.class, 1);

session1.close();

song.setName("New Name");

Song managedSong = session2.merge(song);
```

The state of `song` is copied into `managedSong`.

---

# ⭐ Interview Answer

If asked:

### "What is the difference between update and merge in Hibernate?"

Say:

> **`update()` is a Hibernate-specific/legacy operation that reassociates a detached entity instance with the current Session. `merge()` is a JPA-standard operation that copies the state of a detached entity into a managed instance and returns that managed instance. `merge()` is generally preferred in modern JPA-style applications. If the entity is already managed, neither is normally required because Hibernate's dirty checking automatically detects changes.**

### Easy memory trick

```text
update()
→ Same object becomes managed

merge()
→ State is copied
→ Returned object is managed

managed object
→ Neither needed
→ Dirty checking
```
