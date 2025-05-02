Creating a user and password encoder
==============================
Configure Spring Security in-memory authentication
================================================
To sign up, we want a username and a login
Before this, we must register the user and password in memory or a database.
Hard-code the user info inside your server(in-memory)
![image](https://github.com/user-attachments/assets/e52c83b1-d906-4971-91fc-76e64e58f7ae)
to authenticate
@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
	}
Inside the class WebSecurityConfigurerAdapter 
```java @Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		//create user and store in your memory(tomcat)
		//inmemory auth
		auth.inMemoryAuthentication()
		.withUser("gopi")
		.password("gopi1234")
		.roles("admin");
	}
```
![image](https://github.com/user-attachments/assets/03292482-b21c-4fc0-91e5-9c24c7304f27)
![image](https://github.com/user-attachments/assets/f8cf8e20-34d0-4a86-8d6b-ba26e25cb013)
We need a password encoder
==================
can not store plain password , need a algotihm to encode
for this we have an interface PasswordEncoder
```java
public interface PasswordEncoder {

	/**
	 * Encode the raw password. Generally, a good encoding algorithm applies a SHA-1 or
	 * greater hash combined with an 8-byte or greater randomly generated salt.
	 */
	String encode(CharSequence rawPassword);

	/**
	 * Verify the encoded password obtained from storage matches the submitted raw
	 * password after it too is encoded. Returns true if the passwords match, false if
	 * they do not. The stored password itself is never decoded.
	 *
	 * @param rawPassword the raw password to encode and match
	 * @param encodedPassword the encoded password from storage to compare with
	 * @return true if the raw password, after encoding, matches the encoded password from
	 * storage
	 */
	boolean matches(CharSequence rawPassword, String encodedPassword);

}
```
![image](https://github.com/user-attachments/assets/ebf8bada-4517-4dfd-9956-801d3c240bd5)

BcryptPasswordEncoder is mostly used password encoder

NoOpPasswordEncoder BEan for only plain text password while creating a user
Not recommended for productions

```java
@Bean
	@SuppressWarnings("deprecation")
	PasswordEncoder getPasswordEncoder() {
		return NoOpPasswordEncoder.getInstance();
	}
```
 ![image](https://github.com/user-attachments/assets/d592da1d-fdf1-46e1-ba65-116ced48d35c)
![image](https://github.com/user-attachments/assets/e8d09d3e-f0a2-4522-9deb-ca8b3667f777)
![image](https://github.com/user-attachments/assets/53055703-dc78-4348-b0b0-a1717ec70ba9)
================================================
```java
@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		//create user and store in your memory(tomcat)
		//inmemory auth
		auth.inMemoryAuthentication()
		.withUser("gopi")
		.password("gopi1234")
		.roles("admin");
	}
	@Bean
	PasswordEncoder getPasswordEncoder() {
		return new BCryptPasswordEncoder();
	}
```

![image](https://github.com/user-attachments/assets/a74190ad-9767-4428-a2af-d65f8a8b057b)

becasue password is not encrypted and bcrypt password will match the ecode string
and compare hashcode of password
```java
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		//create user and store in your memory(tomcat)
		//inmemory auth
		auth.inMemoryAuthentication()
		.withUser("gopi")
		.password("$2a$12$BNYXdF2nDqPNrIVlBhfQA.1ZgYq8jkx9viSdMYCtl6.Uy1OEAQtXy")
		.roles("admin");
	}
	@Bean
	PasswordEncoder getPasswordEncoder() {
		return new BCryptPasswordEncoder();
	}
```

gopi1234=>hash=>$2a$12$BNYXdF2nDqPNrIVlBhfQA.1ZgYq8jkx9viSdMYCtl6.Uy1OEAQtXy

![image](https://github.com/user-attachments/assets/b6e2f5f4-9c37-44af-adb8-653771291fa1)

to generate our hashcode
```java
@Autowired
	private PasswordEncoder bcryptPasswordEncoder;
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		//create user and store in your memory(tomcat)
		//in-memory auth
		auth.inMemoryAuthentication()
		.withUser("gopi")
		.password(bcryptPasswordEncoder.encode("gopi1234"))
		.roles("admin");
	}
```

![image](https://github.com/user-attachments/assets/5451ba3e-4278-49da-8fb8-de604b3ea96b)
![image](https://github.com/user-attachments/assets/8d8c6cc8-c817-4bc0-a131-02ccf0573e2c)
![image](https://github.com/user-attachments/assets/a06dd717-7fd4-4041-9d0c-4b3b22e85500)
![image](https://github.com/user-attachments/assets/b78f3348-18bd-428f-b33b-bd5c2242f963)

