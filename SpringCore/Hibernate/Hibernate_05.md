# Hibernate Entity States

In Hibernate, an **entity object can move through different states depending on whether it is associated with a `Session`**.

The 4 important states are:

```text
Transient → Persistent → Detached
                  ↓
               Removed
```

Let's understand each one with a simple `Song` example.

---

# 1. Transient State

### Definition

An object is in the **Transient state** when:

> You have created a Java object, but it is **not associated with a Hibernate Session**.

Example:

```java
Song song = new Song();

song.setName("Hai");
song.setType("Melody");
```

At this point:

```text
Java Application
      |
      ↓
   Song Object
      |
      ↓
   TRANSIENT

Session → Does NOT know about it
Database → No corresponding record
```

### Important points

* Object is created using `new`.
* Hibernate is not managing it.
* It doesn't have to represent a database row yet.
* Changes to the object are not automatically tracked by Hibernate.

### Example

```java
Session session = sessionFactory.openSession();

Song song = new Song();   // Transient

song.setName("Hai");
song.setType("Melody");
```

`Song` is still **Transient** because we haven't associated it with the Session.

---

# 2. Persistent / Managed State

### Definition

An object is in the **Persistent (Managed) state** when it is associated with an active Hibernate `Session`.

For example:

```java
Song song = new Song();

song.setName("Hai");
song.setType("Melody");

session.persist(song);
```

After `persist()`:

```text
Java Application
      |
      ↓
   Song Object
      ↑
      |
    Session
      |
      ↓
   Hibernate
```

Hibernate is now **managing/tracking** the entity.

---

## How does Hibernate track changes?

This is where **Dirty Checking** comes in.

Suppose:

```java
Song song = session.get(Song.class, 1);
```

Now `song` is managed.

Then:

```java
song.setName("Hello");
```

We don't necessarily need to explicitly call:

```java
session.update(song);
```

When the transaction is committed:

```java
transaction.commit();
```

Hibernate detects the change and can generate an `UPDATE`.

```text
Managed Entity
      ↓
Change property
      ↓
Dirty Checking
      ↓
UPDATE SQL
      ↓
Database
```

### Important points

* Session is managing the object.
* Hibernate tracks changes.
* Dirty checking can detect modifications.
* Changes can be synchronized with the database during flush/commit.

---

# 3. Detached State

### Definition

An entity becomes **Detached** when it was previously managed by a Session but is **no longer associated with that Session**.

For example:

```java
Session session = sessionFactory.openSession();

Song song = session.get(Song.class, 1);
```

At this point:

```text
Song → Persistent/Managed
Session → Managing Song
```

Now:

```java
session.close();
```

The Session is closed.

The `Song` Java object can still exist:

```text
Song Object
    ↓
DETACHED

Session → Closed
Hibernate → No longer managing Song
```

### Example

```java
Session session = sessionFactory.openSession();

Song song = session.get(Song.class, 1);

// song is Persistent

session.close();

// song is now Detached
```

The important idea is:

> **Closing the Session does not necessarily destroy the Java object. It removes the object from the Session's persistence context.**

---

## What happens if we modify a detached object?

```java
session.close();

song.setName("New Name");
```

Hibernate is no longer tracking this object through the closed Session.

Therefore, the change isn't automatically synchronized with the database.

If you need to continue working with the entity, you can associate its state with a new persistence context, commonly using `merge()`:

```java
Session newSession = sessionFactory.openSession();

Transaction tx = newSession.beginTransaction();

Song managedSong = newSession.merge(song);

tx.commit();
newSession.close();
```

Conceptually:

```text
Detached
   ↓
merge()
   ↓
Managed
   ↓
Hibernate tracks changes
```

---

# 4. Removed State

There is one more important state: **Removed**.

An entity becomes removed when it is marked for deletion from the database.

Example:

```java
Song song = session.get(Song.class, 1);
```

Currently:

```text
Song → Managed
```

Then:

```java
session.remove(song);
```

Now the entity is in the **Removed** state within that persistence context.

```text
Managed
   |
   | remove()
   ↓
Removed
   |
   | commit()
   ↓
Database record deleted
```

Example:

```java
Transaction tx = session.beginTransaction();

Song song = session.get(Song.class, 1);

session.remove(song);

tx.commit();
```

Hibernate will issue a `DELETE` during synchronization with the database.

---

# 5. Complete Entity State Diagram

This is the diagram you should remember for interviews:

```text
                         new
                          |
                          ↓
                  ┌─────────────┐
                  │  TRANSIENT  │
                  └──────┬──────┘
                         |
                      persist()
                         |
                         ↓
                  ┌─────────────┐
                  │   MANAGED   │
                  │ PERSISTENT  │
                  └───┬─────┬───┘
                      |     |
             close()  |     | remove()
             detach() |     |
             clear()  |     ↓
                      |  ┌─────────┐
                      |  │ REMOVED │
                      |  └─────────┘
                      |
                      ↓
                ┌─────────────┐
                │  DETACHED   │
                └──────┬──────┘
                       |
                     merge()
                       |
                       ↓
                  ┌─────────────┐
                  │   MANAGED   │
                  └─────────────┘
```

---

# 6. Simple Real-Life Example

Imagine a `Song` object.

### Step 1 — Create the object

```java
Song song = new Song();
```

```text
TRANSIENT
```

Hibernate doesn't know about it.

---

### Step 2 — Give it to Hibernate

```java
session.persist(song);
```

```text
MANAGED
```

Hibernate now manages it.

---

### Step 3 — Change it

```java
song.setName("Hello");
```

Hibernate can detect this change through **dirty checking**.

```text
MANAGED
   ↓
Change
   ↓
Dirty Checking
```

---

### Step 4 — Close Session

```java
session.close();
```

```text
DETACHED
```

The Java object can still exist, but Hibernate is no longer managing it through that Session.

---

### Step 5 — Merge it into another Session

```java
newSession.merge(song);
```

```text
DETACHED
   ↓
 merge()
   ↓
MANAGED
```

---

### Step 6 — Delete it

```java
newSession.remove(song);
```

If it's managed:

```text
MANAGED
   ↓
remove()
   ↓
REMOVED
   ↓
commit()
   ↓
Database row deleted
```

---

# 7. State Comparison

| State                  | Session manages object?              | Database row?          | Hibernate tracks changes?       |
| ---------------------- | ------------------------------------ | ---------------------- | ------------------------------- |
| **Transient**          | ❌ No                                 | Usually no             | ❌ No                            |
| **Managed/Persistent** | ✅ Yes                                | Usually associated     | ✅ Yes                           |
| **Detached**           | ❌ No                                 | Usually exists         | ❌ No                            |
| **Removed**            | ✅ Yes, until removal is synchronized | Scheduled for deletion | Entity is scheduled for removal |

---

# ⭐ Most Important Interview Answer

If interviewer asks:

### **"What are the states of an object in Hibernate?"**

Say:

> **Hibernate entities mainly have four states: Transient, Persistent/Managed, Detached, and Removed. A newly created object is Transient. When it becomes associated with a Session, it becomes Persistent or Managed. When it is no longer associated with the Session, for example after `session.close()`, it becomes Detached. When a managed entity is marked for deletion using `remove()`, it enters the Removed state.**

### Easy memory trick:

```text
NEW OBJECT
    ↓
TRANSIENT
    ↓ persist()
MANAGED
    ↓ close()/detach()
DETACHED

MANAGED
    ↓ remove()
REMOVED
```

**One key correction to your notes:** say **"the entity becomes detached when the Session is closed"**, rather than "the object is destroyed." The Java object itself can continue to exist; it simply isn't managed by that Session anymore.

<img width="323" height="254" alt="image" src="https://github.com/user-attachments/assets/b87fc2ef-0dce-48ed-aa8c-3ea70b37d9c9" />
<img width="314" height="258" alt="image" src="https://github.com/user-attachments/assets/e1e373ca-8947-4a3c-8f77-2ded413ee516" />
<img width="330" height="269" alt="image" src="https://github.com/user-attachments/assets/43ac7785-4e4b-4cd5-bd6d-7729952a3941" />
