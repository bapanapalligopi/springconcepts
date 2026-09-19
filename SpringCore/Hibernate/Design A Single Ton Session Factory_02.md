Here are your notes cleaned up, corrected, and organized for revision. I’ve also fixed a few terminology/API issues so you don’t memorize something incorrectly.

# Hibernate Notes — Configuration, SessionFactory, Session

## 1. Required Dependencies

For a basic Hibernate + MySQL application, you typically need:

### Hibernate Core

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>7.1.5.Final</version>
</dependency>
```

### MySQL Connector

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>
```

> **Note:** Dependency versions change over time. The versions above are examples of current stable releases; for a real project, verify compatibility between the Hibernate version, JDK, and MySQL driver.

---

# 2. What is DataSource?

**DataSource is an object that provides database connections to a Java application.**

It is part of JDBC:

```text
Java Application
       ↓
   DataSource
       ↓
 Database Connection
       ↓
    Database
```

### Simple definition

> **DataSource is an abstraction used by Java applications to obtain connections to a database.**

It can also support features such as **connection pooling**.

---

# 3. What is Configuration?

In traditional Hibernate bootstrapping, `Configuration` is a Hibernate class used to **configure Hibernate and build the SessionFactory**.

Example:

```java
Configuration configuration = new Configuration();

configuration.configure("hibernate.cfg.xml");
```

It reads configuration information from:

```text
hibernate.cfg.xml
```

You can also register annotated entity classes:

```java
configuration.addAnnotatedClass(Song.class);
```

Then:

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

### Important

The overall flow is:

```text
Configuration
      ↓
hibernate.cfg.xml
      ↓
addAnnotatedClass()
      ↓
buildSessionFactory()
      ↓
SessionFactory
```

> **Modern Hibernate applications, especially Spring Boot applications, often use different bootstrapping mechanisms, so `Configuration` is especially important to understand for traditional/native Hibernate applications.**

---

# 4. `hibernate.cfg.xml`

A traditional Hibernate configuration file looks like:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>

    <session-factory>

        <property name="hibernate.connection.username">
            root
        </property>

        <property name="hibernate.connection.password">
            root
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/db
        </property>

        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

    </session-factory>

</hibernate-configuration>
```

### Important corrections

The correct XML is:

```xml
<hibernate-configuration>
    <session-factory>
    </session-factory>
</hibernate-configuration>
```

Not:

```xml
<hibernate-configuration>
    <Session-Factory>
```

XML element names are case-sensitive.

---

# 5. Hibernate Configuration Properties

| Property                            | Purpose             |
| ----------------------------------- | ------------------- |
| `hibernate.connection.username`     | Database username   |
| `hibernate.connection.password`     | Database password   |
| `hibernate.connection.url`          | Database URL        |
| `hibernate.connection.driver_class` | JDBC driver         |
| `hibernate.show_sql`                | Shows generated SQL |

Example:

```xml
<property name="hibernate.show_sql">true</property>
```

When enabled, Hibernate prints SQL statements in the console.

---

# 6. What is Environment?

The Hibernate `Environment` class provides **Hibernate configuration property constants and related configuration information**.

For example, Hibernate provides constants corresponding to properties such as:

```java
Environment.SHOW_SQL
Environment.DIALECT
```

Conceptually:

```text
Environment
     ↓
Hibernate configuration properties/constants
```

### Important distinction

Don't say:

> "Environment contains all Hibernate properties."

A better interview answer is:

> **Environment is a Hibernate utility class that provides constants for Hibernate configuration properties and related settings.**

---

# 7. What is SessionFactory?

**SessionFactory is an object used to create Hibernate `Session` objects.**

```text
SessionFactory
      ↓
   Session
      ↓
 Transaction / CRUD operations
      ↓
   Database
```

### Example

```java
SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

---

# 8. Why is SessionFactory Important?

`SessionFactory` is a **heavyweight object**.

Creating it involves significant initialization/configuration work, such as setting up Hibernate's persistence infrastructure.

Therefore:

> **Usually, an application creates one SessionFactory for a particular database/persistence unit and reuses it.**

It is designed to be **thread-safe** and shared.

### Important

Do **not** create a new SessionFactory for every database operation.

❌ Bad:

```java
void saveSong(Song song) {
    Configuration c = new Configuration();
    SessionFactory sf = c.buildSessionFactory();
}
```

✅ Better:

```text
Application
     ↓
One shared SessionFactory
     ↓
Multiple Sessions
```

---

# 9. What is Session?

A **Session** is a Hibernate object that represents a unit of interaction between the Java application and the database.

A Session is used for operations such as:

* Save
* Get
* Update
* Delete
* Query

Example:

```java
Session session = sessionFactory.openSession();
```

### Architecture

```text
SessionFactory
      ↓
Session
      ↓
Database operations
```

---

# 10. How to Initialize a Session?

```java
Session session = sessionFactory.openSession();
```

Usually, after using the Session, it should be closed:

```java
session.close();
```

A common pattern is:

```java
Session session = sessionFactory.openSession();

try {
    // database operations
} finally {
    session.close();
}
```

---

# 11. What is an Entity?

An **Entity** is a Java class whose objects are intended to be persisted in a database.

We use:

```java
@Entity
public class Song {

    @Id
    private int id;

    private String name;
    private String type;
}
```

### What does `@Entity` mean?

```java
@Entity
```

tells Hibernate/JPA that:

> **This class is a persistent entity that should be mapped to a database table.**

Conceptually:

```text
Java Class
    ↓
@Entity
    ↓
Database Table
```

For example:

```java
@Entity
public class Song {

    @Id
    private int id;

    private String name;
    private String type;
}
```

can represent:

```text
SONG
----------------------
id | name | type
----------------------
1  | Hai  | Hai
```

---

# 12. Why Do We Need `@Id`?

Every entity needs an **identifier**.

We normally use:

```java
@Id
private int id;
```

`@Id` identifies the **primary key property** of the entity.

Example:

```java
@Entity
public class Song {

    @Id
    private int id;

    private String name;
    private String type;
}
```

Conceptually:

```text
Song Object
    ↓
id → Primary Key
name → Column
type → Column
```

---

# 13. How Does Hibernate Know About the Entity?

When using traditional Hibernate configuration, you can register the annotated class:

```java
configuration.addAnnotatedClass(Song.class);
```

For example:

```java
Configuration configuration = new Configuration();

configuration.configure("hibernate.cfg.xml");

configuration.addAnnotatedClass(Song.class);

SessionFactory sessionFactory =
        configuration.buildSessionFactory();
```

So:

```text
Song.class
   ↓
addAnnotatedClass()
   ↓
Hibernate knows about Song
```

---

# 14. Mapping Java Properties to Database Columns

Hibernate can map Java fields/properties to database columns.

For example:

```java
private String songName;
```

If you want to explicitly specify the database column:

```java
@Column(name = "song_name")
private String songName;
```

So:

```text
Java property       Database column
------------------------------------
songName       →    song_name
```

### Without `@Column`

Hibernate uses its configured naming/mapping strategy to determine the column name.

Therefore, don't memorize:

> "Database column names and object properties must always be the same."

Instead:

> **Hibernate can automatically map fields to columns, and `@Column` can be used when we need to explicitly specify the column name.**

---

# 15. Saving an Entity

Your notes mention:

```java
session.save(song);
```

This is a **legacy Hibernate API**.

With modern Hibernate/JPA-style code, prefer:

```java
session.persist(song);
```

Example:

```java
Song song = new Song();

song.setName("Hai");
song.setType("Hai");

session.persist(song);
```

---

# 16. Transaction

Database modifications should be performed within a transaction.

Example:

```java
Transaction transaction = session.beginTransaction();

session.persist(song);

transaction.commit();
```

Conceptually:

```text
beginTransaction()
       ↓
   Database operation
       ↓
     commit()
```

If something goes wrong, you can roll back:

```java
transaction.rollback();
```

---

# 17. Complete Basic Example

```java
Configuration configuration = new Configuration();

configuration.configure("hibernate.cfg.xml");

configuration.addAnnotatedClass(Song.class);

SessionFactory sessionFactory =
        configuration.buildSessionFactory();

Session session = sessionFactory.openSession();

Transaction transaction =
        session.beginTransaction();

Song song = new Song();

song.setName("Hai");
song.setType("Hai");

session.persist(song);

transaction.commit();

session.close();
```

---

# 18. Complete Flow to Remember

This is the **most important flow** from your notes:

```text
hibernate.cfg.xml
        ↓
 Configuration
        ↓
addAnnotatedClass(Song.class)
        ↓
buildSessionFactory()
        ↓
 SessionFactory
        ↓
openSession()
        ↓
    Session
        ↓
beginTransaction()
        ↓
persist()
        ↓
   commit()
        ↓
   Database
```

---

# 19. SessionFactory vs Session

This is a very common interview question.

| SessionFactory                             | Session                                              |
| ------------------------------------------ | ---------------------------------------------------- |
| Heavyweight                                | Lightweight compared with SessionFactory             |
| Usually created once                       | Created as needed                                    |
| Thread-safe                                | Not intended to be shared between concurrent threads |
| Creates Sessions                           | Performs database operations                         |
| Long-lived                                 | Short-lived                                          |
| One per persistence/database configuration | Many Sessions can be created                         |

### Easy way to remember

> **SessionFactory creates Sessions.**

```java
Session session = sessionFactory.openSession();
```

---

# 20. Important Points for Interview

### ORM

> **ORM is a technique for mapping Java objects to relational database tables.**

### JPA

> **JPA is a specification that defines standard APIs and rules for persistence and ORM in Java.**

### Hibernate

> **Hibernate is an ORM framework and a JPA provider that implements the JPA specification.**

### Configuration

> **Configuration is a Hibernate class traditionally used to load configuration and bootstrap Hibernate.**

### SessionFactory

> **SessionFactory is a heavyweight, thread-safe object that creates and manages Hibernate Sessions and is normally shared for the lifetime of the application.**

### Session

> **Session represents a unit of interaction with the database and is used to perform persistence operations.**

### Entity

> **An entity is a Java class mapped to a database table and identified by a primary key.**

### `@Entity`

> Marks a class as a persistent entity.

### `@Id`

> Specifies the entity's identifier/primary-key property.

### `@Column`

> Specifies the database column to which a field/property is mapped.

### Transaction

> A transaction groups database operations into a unit of work that can be committed or rolled back.

---

## ⭐ Super Short Revision

```text
ORM
→ Java Objects ↔ Database Tables

JPA
→ Specification

Hibernate
→ ORM Framework + JPA Provider

Configuration
→ Loads/configures Hibernate

SessionFactory
→ Heavyweight, shared
→ Creates Sessions

Session
→ Performs DB operations

@Entity
→ Class becomes persistent entity

@Id
→ Primary key / identifier

@Column
→ Explicit column mapping

Transaction
→ begin → operation → commit/rollback

Basic Flow
→ Configuration
→ SessionFactory
→ Session
→ Transaction
→ persist()
→ commit()
```

**One important correction to your original notes:** `MyBatis` is not a JPA provider. Hibernate and EclipseLink are JPA providers; MyBatis is a separate SQL-mapping/data-access framework.
