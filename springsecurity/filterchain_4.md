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

