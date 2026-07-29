Excellent. Now we move to **JDBC Transactions**, because before understanding `@Transactional`, you should know how transactions work in plain JDBC.

This is the foundation of Spring Transaction Management.

---

# Chapter 3: JDBC Transactions

As always, we'll learn:

> **Why → What → How → Where → Internal Working → Code → Interview Questions → Best Practices**

---

# 1. Why do we need JDBC Transactions?

Suppose you're transferring **₹10,000** from one account to another.

Two SQL statements:

```sql
UPDATE account
SET balance = balance - 10000
WHERE id = 1;
```

```sql
UPDATE account
SET balance = balance + 10000
WHERE id = 2;
```

If the first query succeeds and the second fails:

| Account | Balance |
| ------- | ------- |
| A       | ₹40,000 |
| B       | ₹20,000 |

The database is inconsistent.

To solve this, JDBC provides transaction support.

---

# 2. What is Auto Commit?

## What?

Every JDBC `Connection` has a property called **Auto Commit**.

By default:

```java
connection.getAutoCommit();
```

returns

```text
true
```

Meaning:

Every SQL statement is treated as its own transaction.

---

## Internal Working

```text
UPDATE Employee

↓

Execute SQL

↓

Automatically Commit
```

Even if you don't call `commit()`, JDBC commits automatically.

---

# Example

```java
Connection con = DriverManager.getConnection(...);

System.out.println(con.getAutoCommit());
```

Output

```text
true
```

---

# Problem with Auto Commit

Suppose:

```java
UPDATE Account A
```

Auto Commit immediately saves it.

Then:

```java
UPDATE Account B
```

throws an exception.

Too late.

The first update is already permanent.

---

# 3. Disabling Auto Commit

To create a transaction manually:

```java
Connection con = DriverManager.getConnection(...);

con.setAutoCommit(false);
```

Now:

```text
Auto Commit Disabled
```

JDBC waits for your decision.

---

# Internal Flow

```text
Connection

↓

setAutoCommit(false)

↓

Execute SQL

↓

Execute SQL

↓

Execute SQL

↓

You decide

Commit

OR

Rollback
```

---

# 4. commit()

## What?

Commit permanently saves all changes.

```java
con.commit();
```

Example:

```java
Connection con = DriverManager.getConnection(...);

con.setAutoCommit(false);

PreparedStatement ps1 = ...

ps1.executeUpdate();

PreparedStatement ps2 = ...

ps2.executeUpdate();

con.commit();
```

Everything becomes permanent.

---

# Internal Flow

```text
SQL 1

↓

SQL 2

↓

SQL 3

↓

commit()

↓

Permanent Database Changes
```

---

# 5. rollback()

Suppose:

```text
SQL 1

✓

SQL 2

❌
```

Instead of:

```java
commit();
```

Call:

```java
rollback();
```

Example

```java
try{

    con.setAutoCommit(false);

    ps1.executeUpdate();

    ps2.executeUpdate();

    con.commit();

}catch(Exception e){

    con.rollback();

}
```

---

# Internal Working

```text
Start Transaction

↓

SQL 1

↓

SQL 2

↓

Exception

↓

Rollback

↓

Undo SQL 1
```

Everything returns to its original state.

---

# 6. Complete Example

```java
Connection con = null;

try{

    con = DriverManager.getConnection(url,user,password);

    con.setAutoCommit(false);

    PreparedStatement debit =
        con.prepareStatement(
            "UPDATE account SET balance=balance-? WHERE id=?"
        );

    debit.setInt(1,10000);
    debit.setInt(2,1);
    debit.executeUpdate();

    PreparedStatement credit =
        con.prepareStatement(
            "UPDATE account SET balance=balance+? WHERE id=?"
        );

    credit.setInt(1,10000);
    credit.setInt(2,2);
    credit.executeUpdate();

    con.commit();

}catch(Exception e){

    if(con!=null){

        con.rollback();

    }

}
```

---

# 7. Internal Flow

```text
Application

↓

Connection

↓

Auto Commit = false

↓

Debit Query

↓

Credit Query

↓

Success?

│

├── Yes

│

▼

Commit

│

└── No

▼

Rollback
```

---

# 8. Why is Manual Transaction Difficult?

Imagine every method:

```java
Connection con;

try{

    con.setAutoCommit(false);

    ...

    con.commit();

}catch(Exception e){

    con.rollback();

}
```

Problems:

* Lots of boilerplate code
* Easy to forget `rollback()`
* Easy to forget `commit()`
* Resource leaks
* Hard to maintain

Spring solves all of this.

---

# 9. How Spring Solves It

Instead of writing:

```java
con.setAutoCommit(false);

...

con.commit();

...

con.rollback();
```

Spring allows:

```java
@Transactional
public void transferMoney(){

}
```

Spring manages:

* Begin Transaction
* Commit
* Rollback
* Resource cleanup

Automatically.

---

# Internal Comparison

## Plain JDBC

```text
Connection

↓

setAutoCommit(false)

↓

SQL

↓

commit()

↓

close()
```

Developer writes everything.

---

## Spring

```text
@Transactional

↓

Spring

↓

Begin Transaction

↓

Commit

↓

Rollback

↓

Close Connection
```

Developer writes only business logic.

---

# Interview Questions

### Q1. What is Auto Commit?

Auto Commit is a JDBC feature where every SQL statement is automatically committed immediately after execution.

Default value:

```text
true
```

---

### Q2. Why disable Auto Commit?

To group multiple SQL statements into a single transaction.

```java
con.setAutoCommit(false);
```

---

### Q3. What does `commit()` do?

Permanently saves all changes made in the current transaction.

---

### Q4. What does `rollback()` do?

Undoes all changes made during the current transaction.

---

### Q5. Why doesn't Spring require us to call `commit()`?

Because Spring's transaction manager automatically commits or rolls back the transaction based on the method outcome.

---

# Best Practices

✅ Disable auto commit only when you need a transaction involving multiple operations.

✅ Always commit after successful execution.

✅ Always rollback in case of exceptions.

✅ Close database resources in a `finally` block or use try-with-resources (when appropriate).

✅ In Spring applications, prefer `@Transactional` instead of manual transaction handling.

---

# Summary

```text
Auto Commit = true

SQL

↓

Automatically Commit

-----------------------------

Auto Commit = false

SQL 1

↓

SQL 2

↓

SQL 3

↓

Success?

│

├── Yes → Commit

└── No → Rollback
```

---

# 📍 Where We Are

```text
Spring Transaction Management

✅ Why Transactions
✅ ACID Properties
✅ JDBC Transactions
    • Auto Commit
    • commit()
    • rollback()

⏭️ Next

PlatformTransactionManager
        ↓
Spring Transaction Architecture
        ↓
@Transactional
        ↓
Propagation
        ↓
Isolation Levels
```

---

# 🎯 Before We Move to Spring

Here's an interview question:

**Question:** If `autoCommit` is `true` and you execute two `UPDATE` statements, where the first succeeds and the second fails, what happens?

**Answer:**

* The first `UPDATE` is **already committed** because auto commit is enabled.
* The second `UPDATE` fails.
* The database is left in a **partially updated (inconsistent)** state.
* This is exactly why transactions and `@Transactional` are needed.

---

## 🚀 Next Topic: Spring Transaction Architecture (Most Important)

Now we'll learn:

* Why Spring introduced `PlatformTransactionManager`
* How `@Transactional` works internally
* How Spring uses **AOP proxies** to manage transactions
* What happens before and after your method is called

This is one of the most important Spring interview topics because it connects **Spring Core + AOP + JDBC + Transactions** into one complete picture.
