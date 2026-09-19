Here are your **Hibernate CRUD + Thread-Safe SessionFactory notes**, cleaned up and corrected for modern Hibernate.

# Hibernate CRUD Operations

CRUD stands for:

```text
C → Create
R → Read
U → Update
D → Delete
```

---

# 1. CREATE — Save Data

To insert a new entity into the database:

```java
Song song = new Song();

song.setName("Hai");
song.setType("Melody");

Transaction transaction = session.beginTransaction();

session.persist(song);

transaction.commit();
```

### Flow

```text
Create Object
     ↓
beginTransaction()
     ↓
persist(entity)
     ↓
commit()
     ↓
Database
```

### Important

Older Hibernate code commonly uses:

```java
session.save(song);
```

In modern Hibernate/JPA-style code, prefer:

```java
session.persist(song);
```

---

# 2. READ — Read Data

To retrieve an entity using its primary key:

```java
Song song = session.get(Song.class, 2);
```

Meaning:

```text
Song.class → Entity type
2          → Primary key
```

Example:

```java
Song s2 = session.get(Song.class, 2);
```

Hibernate searches for the `Song` whose ID is `2`.

Conceptually:

```text
session.get(Song.class, 2)
              ↓
       SELECT ... WHERE id = 2
              ↓
          Song object
```

### What if the record doesn't exist?

`session.get()` returns:

```java
null
```

if no entity with that identifier is found.

---

# 3. `get()` vs `load()`

Older Hibernate versions commonly taught:

```java
session.get(Song.class, 2);
session.load(Song.class, 2);
```

### `get()`

```java
Song song = session.get(Song.class, 2);
```

* Retrieves the entity.
* If it doesn't exist, returns `null`.
* Generally performs the database lookup when needed.

### `load()`

Historically, `load()` was used to obtain a proxy/reference and could defer the database lookup.

Modern Hibernate code should generally prefer:

```java
session.get(Song.class, 2);
```

or the modern `find()` API where appropriate.

> **Interview note:** `load()` behavior has changed/evolved across Hibernate versions, so avoid memorizing old explanations such as "get hits the DB immediately and load never hits the DB."

---

# 4. UPDATE — Update Data

Suppose we want to update an existing Song.

First, retrieve the entity:

```java
Song song = session.get(Song.class, 2);
```

Then modify it:

```java
song.setName("New Song");
song.setType("Rock");
```

Then commit the transaction:

```java
Transaction transaction =
        session.beginTransaction();

Song song = session.get(Song.class, 2);

song.setName("New Song");
song.setType("Rock");

transaction.commit();
```

Hibernate detects the changes and generates the required `UPDATE` SQL.

### Important Concept — Dirty Checking

Hibernate can automatically detect changes made to a **managed entity**.

```text
get()
 ↓
Entity becomes managed
 ↓
Change object
 ↓
commit()
 ↓
Hibernate detects changes
 ↓
UPDATE
```

This is called **dirty checking**.

---

# 5. What About `session.update()`?

Older Hibernate applications commonly use:

```java
session.update(song);
```

This is particularly associated with updating a **detached entity**.

Example:

```java
Song song = ...; // detached entity

Transaction transaction =
        session.beginTransaction();

session.update(song);

transaction.commit();
```

However, for modern Hibernate/JPA code, you will often work with managed entities and dirty checking, or use:

```java
session.merge(song);
```

when you need to copy the state of a detached entity into a managed entity.

### Remember

For a newly loaded entity:

```java
Song song = session.get(Song.class, 2);

song.setName("New Name");

transaction.commit();
```

you generally **don't need `session.update(song)`**.

---

# 6. DELETE — Delete Data

First, retrieve the entity:

```java
Song song = session.get(Song.class, 2);
```

Then delete it inside a transaction:

```java
Transaction transaction =
        session.beginTransaction();

Song song = session.get(Song.class, 2);

session.remove(song);

transaction.commit();
```

Older Hibernate code commonly uses:

```java
session.delete(song);
```

Modern JPA-style Hibernate code prefers:

```java
session.remove(song);
```

### Flow

```text
get()
 ↓
Entity
 ↓
remove()
 ↓
commit()
 ↓
DELETE from database
```

---

# 7. CRUD Summary

| Operation             | Modern Hibernate/JPA-style API  |
| --------------------- | ------------------------------- |
| Create                | `session.persist(entity)`       |
| Read                  | `session.get(Entity.class, id)` |
| Update managed entity | Modify entity + `commit()`      |
| Merge detached entity | `session.merge(entity)`         |
| Delete                | `session.remove(entity)`        |

Older Hibernate tutorials may show:

```text
save()
get()
update()
delete()
```

These are important to recognize when reading legacy Hibernate code, but don't confuse them with the preferred modern JPA-style APIs.

---

# 8. Why Do We Need a Transaction?

Database-changing operations should be performed inside a transaction.

For example:

```java
Transaction transaction =
        session.beginTransaction();

session.persist(song);

transaction.commit();
```

For update:

```java
Transaction transaction =
        session.beginTransaction();

song.setName("New Name");

transaction.commit();
```

For delete:

```java
Transaction transaction =
        session.beginTransaction();

session.remove(song);

transaction.commit();
```

### Transaction Flow

```text
beginTransaction()
       ↓
   DB operation
       ↓
    commit()
```

If an error occurs:

```java
transaction.rollback();
```

---

# 9. SessionFactory — Thread Safety

This is an **important interview topic**.

### Is SessionFactory thread-safe?

**Yes. `SessionFactory` is designed to be thread-safe and shared.**

It is also a **heavyweight object**.

Therefore, normally:

> **Create one SessionFactory for a particular database/persistence unit and share it across the application.**

---

# 10. Creating a SessionFactory

Traditional Hibernate:

```java
Configuration configuration =
        new Configuration();

configuration.configure("hibernate.cfg.xml");

configuration.addAnnotatedClass(Song.class);

SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

The important part is:

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

---

# 11. Why Only One SessionFactory?

Creating a SessionFactory is expensive because Hibernate performs initialization and builds its internal infrastructure.

Therefore:

❌ Don't do this for every request:

```java
void saveSong(Song song) {

    Configuration configuration =
            new Configuration();

    SessionFactory sf =
            configuration.buildSessionFactory();

    // ...
}
```

Instead:

```text
Application
     |
     ↓
 ONE SessionFactory
     |
     ├── Session 1
     ├── Session 2
     ├── Session 3
     └── Session 4
```

---

# 12. SessionFactory vs Session

This distinction is **very important**.

| SessionFactory                      | Session                                         |
| ----------------------------------- | ----------------------------------------------- |
| Heavyweight                         | Lightweight compared with SessionFactory        |
| Thread-safe                         | Not thread-safe                                 |
| Shared                              | Should not be shared between concurrent threads |
| Long-lived                          | Short-lived                                     |
| Usually one per DB/persistence unit | Many Sessions                                   |
| Creates Sessions                    | Performs persistence operations                 |

### Easy Rule

> **SessionFactory = shared**

> **Session = not shared**

```java
Session session =
        sessionFactory.openSession();
```

Each thread/request should generally work with its own Session.

---

# 13. Thread-Safe SessionFactory Design

A simple traditional approach is to initialize the SessionFactory once.

```java
public class HibernateUtil {

    private static final SessionFactory sessionFactory;

    static {
        Configuration configuration =
                new Configuration();

        configuration.configure("hibernate.cfg.xml");

        configuration.addAnnotatedClass(Song.class);

        sessionFactory =
                configuration.buildSessionFactory();
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}
```

Then:

```java
SessionFactory sf =
        HibernateUtil.getSessionFactory();
```

And for each unit of work:

```java
Session session =
        sf.openSession();
```

### Architecture

```text
             HibernateUtil
                  |
                  ↓
          SessionFactory
          (shared/thread-safe)
                  |
        ┌─────────┼─────────┐
        ↓         ↓         ↓
    Session 1  Session 2  Session 3
      Thread 1   Thread 2   Thread 3
```

---

# 14. Important: Don't Make Session Static

Avoid:

```java
private static Session session;
```

and sharing that Session between threads.

Instead:

```java
private static SessionFactory sessionFactory;
```

and create Sessions as needed:

```java
Session session =
        sessionFactory.openSession();
```

### Remember

```text
SessionFactory → ONE → Shared
Session        → MANY → Not shared
```

---

# ⭐ Final Revision Notes

```text
CRUD
----------------------------

CREATE
session.persist(entity)
        ↓
transaction.commit()


READ
session.get(Entity.class, id)
        ↓
returns entity / null


UPDATE
session.get(Entity.class, id)
        ↓
modify entity
        ↓
transaction.commit()
        ↓
Hibernate dirty checking
        ↓
UPDATE


DELETE
session.get(Entity.class, id)
        ↓
session.remove(entity)
        ↓
transaction.commit()
```

## SessionFactory

```text
SessionFactory
→ Heavyweight
→ Thread-safe
→ Usually one per database/persistence unit
→ Shared across application
→ Creates Sessions
```

## Session

```text
Session
→ Short-lived
→ Not thread-safe
→ Don't share between threads
→ Used for CRUD/database operations
```

## Most Important Interview Line

> **SessionFactory is a heavyweight, thread-safe object that is normally created once and shared, while Session is a short-lived, non-thread-safe object created from the SessionFactory for database operations.**
