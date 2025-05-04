create sign up form to create a user
====================================
```java
 @GetMapping("/signupme")
    public String signup(@ModelAttribute("user") SecurityUser user) {
    	System.out.println("hai signuou");
    	System.out.println("the user is: "+user.toString());
    	return "signup";
    }
    
    @PostMapping("/signupmeonce")
    public String processSignup(@ModelAttribute("user") SecurityUser user) {
        System.out.println("hai processSignup");
        System.out.println("the user sto is: " + user.toString());
        secureUserService.createUser(user);
        return "redirect:/home";
    }
```
```html
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html>
<html>
<head>
    <title>Signup</title>
</head>
<body>

<h2>Signup Page</h2>
<form:form method="post" action="signupmeonce" modelAttribute="user">
    <label>Username:</label>
  	<form:input path="username"/>
    <label>Password:</label>
    <form:input path="password"/>
    <br/>
    <input type="submit" value="signup"/>
    
   </form:form>


</body>
</html>
```
```java
package springsecurityexample.config;

import org.springframework.stereotype.Component;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Component
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class SecurityUser {
	private String username;
	private String password;
	private int enabledOrNot;
}
```
```java
package springsecurityexample.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Component;

@Component
public class SecureUserService {
	@Autowired
	PasswordEncoder passwordEncoder;
	@Autowired
	NamedParameterJdbcTemplate namedParameterJdbcTemplate;
	public void createUser(SecurityUser securityUser) {
		String sqlString="INSERT INTO security_db.users(username, password,enabled)values(:username,:password,:enabled);";
		MapSqlParameterSource parameterSource = new MapSqlParameterSource();
		parameterSource.addValue("username", securityUser.getUsername());
		parameterSource.addValue("password", passwordEncoder.encode(securityUser.getPassword()));
		parameterSource.addValue("enabled", 1);
	
		int update = namedParameterJdbcTemplate.update(sqlString, parameterSource);
		System.out.println("user created successfully");
		
		//assign authority
		String sqString2="INSERT INTO security_db.authorities(username,authority)values(:username,:authority);";
		MapSqlParameterSource parameterSource2 = new MapSqlParameterSource();
		parameterSource2.addValue("username", securityUser.getUsername());
		parameterSource2.addValue("authority", "ROLE_ADMIN");
		namedParameterJdbcTemplate.update(sqString2, parameterSource2);
		
		System.out.println("user created with roles successfully");
	}
}
```
