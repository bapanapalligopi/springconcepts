Loading user from db (database authentication)
=============================================
spring security default schema
---------------------------------
 ![image](https://github.com/user-attachments/assets/0f19f5e1-c54e-4897-bd1e-4ff66570620c)

 spring provided default schema for users

 table1: users
 username 
 password
 enabled (whether the user is enabled or not, soft delete, hard delete (ifn complete delete go for hard else soft))
 table2: authorities
 username 
 authority

 if you create the userds table exactly same as this, there is no need of writew any persisstence code to load users data from db.
 
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

INSERT INTO  authorities values('gopi','admin');
INSERT INTO  authorities values('ravi','user');
SELECT  * FROM  authorities;
```

