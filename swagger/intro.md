Since I can't directly generate a PDF here, I'll provide you with **all the Swagger/OpenAPI URLs and configurations in a structured format** that you can easily save as a PDF yourself. Below is the complete guide:

---

# **Complete Swagger/OpenAPI Reference Guide**  
*(For Spring Boot with SpringDoc OpenAPI)*  

## **1. Core URLs**  
| Purpose                     | URL                                   |
|----------------------------|---------------------------------------|
| **Swagger UI** (Interactive Docs) | `http://localhost:8080/swagger-ui/index.html` |
| **OpenAPI JSON Spec**       | `http://localhost:8080/v3/api-docs`   |
| **OpenAPI YAML Spec**       | `http://localhost:8080/v3/api-docs.yaml` |
| **Grouped APIs**            | `http://localhost:8080/v3/api-docs/group-name` |

---

## **2. Configuration**  
### **`pom.xml` Dependencies**  
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version> <!-- Check for updates -->
</dependency>
```

### **`application.properties` Customizations**  
```properties
# Change Swagger UI path
springdoc.swagger-ui.path=/custom-docs

# Disable Swagger UI
springdoc.swagger-ui.enabled=false

# Change API docs endpoint
springdoc.api-docs.path=/api-docs
```

---

## **3. Security Setup**  
### **Allow Swagger in Spring Security**  
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers(
                "/swagger-ui/**",
                "/v3/api-docs/**",
                "/swagger-resources/**"
            ).permitAll()
        );
    return http.build();
}
```

---

## **4. Advanced Features**  
### **Deep Linking**  
- Access a specific endpoint:  
  `http://localhost:8080/swagger-ui/index.html#/UserController/getUserById`  

### **Filter by Tag**  
`http://localhost:8080/swagger-ui/index.html?tags=user-controller`  

### **Export Documentation**  
1. Copy the JSON from `http://localhost:8080/v3/api-docs`.  
2. Paste into:  
   - [Swagger Editor](https://editor.swagger.io/) → Generate HTML/PDF.  
   - [Redoc](https://redocly.github.io/redoc/) → Interactive docs.  

---

## **5. Troubleshooting**  
| Issue                      | Solution                              |
|----------------------------|---------------------------------------|
| **404 Error**              | Check dependencies and security config. |
| **Blank Swagger UI**       | Ensure `@Bean OpenAPI` is configured. |
| **CORS Errors**            | Add CORS config for `/v3/api-docs`.   |

---

## **How to Save as PDF**  
1. **Copy this guide** into a text editor (e.g., Word, Google Docs).  
2. **Export as PDF** (File → Save As → PDF).  
3. **For code snippets**: Use a Markdown tool like [Dillinger](https://dillinger.io/) → Export PDF.  

---

### **Need a pre-formatted PDF?**  
Let me know—I can help you format this into a template! 🚀
