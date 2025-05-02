your security class must extends WebSecurityConfigurerAdapter class
@EnableWebSecurity for security 
regidter your security filter chain class with your application
basic setup
```java
package springsecurityexample.config;

import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity(debug = true)
public class SpringSecurityFilterCustom extends WebSecurityConfigurerAdapter{

}
```
```java
package springsecurityexample.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

@Configuration
//initialize and register the filter chain
public class SpringSecurityFilterChainInitializer extends  AbstractSecurityWebApplicationInitializer{

}
```
```java
package springsecurityexample.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@SuppressWarnings("unused")
@Configuration
@ComponentScan(basePackages = {"springsecurityexample"})
@EnableWebMvc
public class GopiServeletConfiguationFile implements WebMvcConfigurer{
	
	@Bean
    public InternalResourceViewResolver getInternalResourceViewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/views");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```
```java
package springsecurityexample.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

@Configuration
public class GopiDispatcherServelet extends AbstractAnnotationConfigDispatcherServletInitializer{

	@Override
	protected Class<?>[] getRootConfigClasses() {
		// TODO Auto-generated method stub
		return null;
	}

	@Override
	protected Class<?>[] getServletConfigClasses() {
		// TODO Auto-generated method stub
		Class[] servelts= {GopiServeletConfiguationFile.class};
		return servelts;
	}

	@Override
	protected String[] getServletMappings() {
		// TODO Auto-generated method stub
		 String[] mappingStrings ={"/"};
		return mappingStrings;
	}

}
```
```java
package springsecurityexample.config;


import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GopiController {
	@GetMapping("/hello")
	public String sayHello() {
		return "hello";
	}
	@GetMapping("/hai")
	public String sayHai() {
		return "hai";
	}
	@GetMapping("/bye")
	public String sayBye() {
		return "bye";
	}
}
```
AbstractSecurityWebApplicationInitializer=>onStartUp()
