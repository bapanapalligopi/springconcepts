Loading user from db (database authentication)
=============================================
spring security default schema
---------------------------------
 ![image](https://github.com/user-attachments/assets/0f19f5e1-c54e-4897-bd1e-4ff66570620c) — Spring Security provides **built-in support for default database schema** to handle authentication **without writing any custom user-loading logic** — *as long as you follow its expected schema*.

---

## ✅ Default Schema Used by Spring Security (JdbcUserDetailsManager)

Spring Security expects **two tables** with specific columns:

### 🔹 1. `users` table

```sql
CREATE TABLE users (
    username VARCHAR(50) NOT NULL PRIMARY KEY,
    password VARCHAR(100) NOT NULL,
    enabled BOOLEAN NOT NULL
);
```

* `username`: Unique ID of the user
* `password`: Encoded (hashed) password
* `enabled`: `true` or `false` to allow/disable login (used for soft deletes)

---

### 🔹 2. `authorities` table

```sql
CREATE TABLE authorities (
    username VARCHAR(50) NOT NULL,
    authority VARCHAR(50) NOT NULL,
    CONSTRAINT fk_username FOREIGN KEY(username) REFERENCES users(username)
);
```

* `username`: Must match one from the `users` table
* `authority`: Role name like `'ROLE_USER'`, `'ROLE_ADMIN'`

> 🔐 Spring uses this to assign roles to users during authentication.

---

## ✅ How Spring Loads This

Spring provides `JdbcUserDetailsManager`, which:

* Automatically queries these tables
* Loads user and roles into `UserDetails`
* Authenticates the user with no extra code required

---

### 🔧 Sample Spring Security Config Using It

```java
@Bean
public UserDetailsService userDetailsService(DataSource dataSource) {
    JdbcUserDetailsManager manager = new JdbcUserDetailsManager(dataSource);
    return manager;
}
```

Spring will now:

* Use JDBC to query `users` and `authorities` table
* Authenticate the user automatically

---

## 🛠️ When Is This Useful?

* ✅ You want simple, quick integration without custom login code
* ✅ You want to control user access through the DB only
* ❌ You don't need a custom user model (e.g., with full name, email, etc.)

---

## 💡 Custom Schema?

If you have custom table names or column names:

* You must implement your own `UserDetailsService` and map your DB schema manually

---

## ✅ Summary

| Feature      | Default Tables        | Code Needed? |
| ------------ | --------------------- | ------------ |
| User login   | `users`               | ❌ No         |
| Role mapping | `authorities`         | ❌ No         |
| Soft delete  | Set `enabled = false` | ✅            |
| Hard delete  | Delete from `users`   | ✅            |

---

Would you like a SQL insert example or a working Spring Boot + JDBC project setup for this?

 
 ```mysql
USE security_db;
create table users(
	username varchar(50) not null primary key,
	password varchar(50) not null,
	enabled boolean not null
);
INSERT  INTO users values('gopi','gopi1234',1);
INSERT  INTO users values('ravi','ravi1234',1);
SELECT  * FROM users u ;

create table authorities (
	username varchar(50) not null,
	authority varchar(50) not null,
	constraint fk_authorities_users foreign key(username) references users(username)
);
create unique index ix_auth_username on authorities (username,authority);
DELETE FROM authorities;
INSERT INTO  authorities values('gopi','ROLE_ADMIN');
INSERT INTO  authorities values('ravi','ROLE_USER');
SELECT  * FROM  authorities;
```
![image](https://github.com/user-attachments/assets/0b0a381e-6e94-4524-88e9-ac34715d43ca)

