# Spring Security — Chapter 20: Complete Secure Employee API

This is the **Spring Security capstone**.

We're going to assemble everything we learned into one architecture:

```text
Client
  ↓
POST /api/auth/login
  ↓
AuthenticationManager
  ↓
UserDetailsService
  ↓
PasswordEncoder
  ↓
JWT
  ↓
Client
  ↓
Authorization: Bearer <JWT>
  ↓
BearerTokenAuthenticationFilter
  ↓
JwtDecoder
  ↓
JwtAuthenticationConverter
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
  ↓
Service + @PreAuthorize
  ↓
Repository
  ↓
Database
```

This uses Spring Security's built-in Resource Server support for bearer JWT authentication rather than maintaining a custom JWT filter. Spring Security's Resource Server configuration wires `BearerTokenAuthenticationFilter`, and `JwtAuthenticationProvider` uses `JwtDecoder` and `JwtAuthenticationConverter` for JWT authentication. ([Home][1])

We'll keep the **Employee database implementation from our previous REST capstone** and add the complete security layer.

---

# 1. Final Project Structure

```text
employee-rest-api
│
├── pom.xml
│
└── src/main
    │
    ├── java/com/practice/employeeapi
    │   │
    │   ├── EmployeeApiApplication.java
    │   │
    │   ├── config
    │   │   ├── SecurityConfig.java
    │   │   ├── JwtConfig.java
    │   │   └── CorsConfig.java
    │   │
    │   ├── controller
    │   │   ├── AuthController.java
    │   │   └── EmployeeController.java
    │   │
    │   ├── dto
    │   │   ├── LoginRequest.java
    │   │   ├── AuthResponse.java
    │   │   └── ErrorResponse.java
    │   │
    │   ├── security
    │   │   ├── JwtService.java
    │   │   ├── RestAuthenticationEntryPoint.java
    │   │   └── RestAccessDeniedHandler.java
    │   │
    │   ├── service
    │   │   └── EmployeeService.java
    │   │
    │   ├── repository
    │   │   └── EmployeeRepository.java
    │   │
    │   └── exception
    │       └── ...
    │
    └── resources
        ├── application.properties
        ├── schema.sql
        └── data.sql
```

---

# 2. Dependencies

For current Spring Boot 4.1, the official starters include `spring-boot-starter-security` and `spring-boot-starter-security-oauth2-resource-server`; Boot's starter list also includes a dedicated security-test starter. ([Home][2])

Add these to the existing project:

```xml
<dependencies>

    <!-- Existing REST API -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- JDBC -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT Resource Server -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security-oauth2-resource-server</artifactId>
    </dependency>

    <!-- H2 -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Tests -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

---

# 3. Configuration

We'll use an environment variable for the signing secret.

### `application.properties`

```properties
spring.application.name=employee-rest-api

server.port=8080

spring.datasource.url=jdbc:h2:mem:employee_db
spring.datasource.username=sa
spring.datasource.password=

spring.sql.init.mode=always

# JWT
app.jwt.secret-base64=${JWT_SECRET_BASE64}
app.jwt.expiration-seconds=900

# CORS
app.cors.allowed-origin=http://localhost:3000
```

Generate a strong secret, for example:

```bash
openssl rand -base64 32
```

Then set:

```bash
export JWT_SECRET_BASE64='your-generated-value'
```

Do **not** put the real production secret into Git.

---

# 4. Login Request

```java
package com.practice.employeeapi.dto;

import jakarta.validation.constraints.NotBlank;

public record LoginRequest(

        @NotBlank
        String username,

        @NotBlank
        String password

) {
}
```

---

# 5. Login Response

```java
package com.practice.employeeapi.dto;

public record AuthResponse(

        String accessToken,

        String tokenType,

        long expiresIn

) {
}
```

Response:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

# 6. Password Encoder

We'll use Spring's delegating encoder rather than manually hard-coding one algorithm into every part of the application.

```java
@Bean
PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories
            .createDelegatingPasswordEncoder();
}
```

Spring Security provides `PasswordEncoder` specifically for secure password storage and matching, and its delegating encoder supports encoding identifiers that make future algorithm migration easier. ([docs.spring.io](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html?utm_source=chatgpt.com))

---

# 7. Users for the Capstone

For the capstone, we'll use two users:

```text
rahul
ROLE_USER

admin
ROLE_ADMIN
```

```java
@Bean
UserDetailsService userDetailsService(
        PasswordEncoder passwordEncoder) {

    UserDetails user =
            User.withUsername("rahul")
                    .password(
                        passwordEncoder.encode(
                            "password123"
                        )
                    )
                    .roles("USER")
                    .build();

    UserDetails admin =
            User.withUsername("admin")
                    .password(
                        passwordEncoder.encode(
                            "admin123"
                        )
                    )
                    .roles("ADMIN")
                    .build();

    return new InMemoryUserDetailsManager(
            user,
            admin
    );
}
```

This is intentionally **in-memory for the learning capstone**.

In a real application, replace it with:

```text
UserDetailsService
      ↓
UserRepository
      ↓
Database
```

The rest of the security architecture stays essentially the same.

---

# 8. AuthenticationManager

Now connect username/password authentication.

```java
@Bean
AuthenticationManager authenticationManager(
        UserDetailsService userDetailsService,
        PasswordEncoder passwordEncoder) {

    DaoAuthenticationProvider provider =
            new DaoAuthenticationProvider(
                    userDetailsService
            );

    provider.setPasswordEncoder(
            passwordEncoder
    );

    return new ProviderManager(provider);
}
```

Remember:

```text
AuthenticationManager
       ↓
ProviderManager
       ↓
DaoAuthenticationProvider
       ↓
UserDetailsService
       ↓
PasswordEncoder
```

`DaoAuthenticationProvider` is specifically designed to authenticate username/password credentials using `UserDetailsService` and a `PasswordEncoder`. ([Home][1])

---

# 9. JWT Secret Key

Create:

### `JwtConfig.java`

```java
package com.practice.employeeapi.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

@Configuration
public class JwtConfig {

    @Bean
    SecretKey jwtSecretKey(
            @Value("${app.jwt.secret-base64}")
            String secret) {

        byte[] decoded =
                Base64.getDecoder().decode(secret);

        return new SecretKeySpec(
                decoded,
                "HmacSHA256"
        );
    }
}
```

We're using symmetric HMAC signing for this learning project:

```text
Same key
 ↓
Sign JWT

Same key
 ↓
Verify JWT
```

A production architecture with a dedicated authorization server will often use asymmetric signing so resource servers can verify tokens with a public key without receiving the private signing key.

---

# 10. JWT Encoder

```java
@Bean
JwtEncoder jwtEncoder(
        SecretKey jwtSecretKey) {

    return NimbusJwtEncoder
            .withSecretKey(jwtSecretKey)
            .build();
}
```

`JwtEncoder` is Spring Security's abstraction for creating JWTs; `NimbusJwtEncoder` is a standard implementation. ([Home][1])

---

# 11. JWT Decoder

```java
@Bean
JwtDecoder jwtDecoder(
        SecretKey jwtSecretKey) {

    return NimbusJwtDecoder
            .withSecretKey(jwtSecretKey)
            .build();
}
```

So now:

```text
JwtEncoder
    ↓
Generate JWT

JwtDecoder
    ↓
Validate JWT
```

---

# 12. JWT Service

### `JwtService.java`

```java
package com.practice.employeeapi.security;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.security.oauth2.jose.jws.MacAlgorithm;
import org.springframework.security.oauth2.jwt.JwtClaimsSet;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.JwtEncoderParameters;
import org.springframework.stereotype.Service;

import java.time.Instant;
import java.time.temporal.ChronoUnit;
import java.util.List;

@Service
public class JwtService {

    private final JwtEncoder jwtEncoder;
    private final long expirationSeconds;

    public JwtService(
            JwtEncoder jwtEncoder,
            @Value("${app.jwt.expiration-seconds}")
            long expirationSeconds) {

        this.jwtEncoder = jwtEncoder;
        this.expirationSeconds = expirationSeconds;
    }

    public String generateToken(
            Authentication authentication) {

        Instant now = Instant.now();

        List<String> authorities =
                authentication
                        .getAuthorities()
                        .stream()
                        .map(authority ->
                                authority.getAuthority())
                        .toList();

        JwtClaimsSet claims =
                JwtClaimsSet.builder()

                        .subject(
                                authentication.getName()
                        )

                        .issuer("employee-api")

                        .issuedAt(now)

                        .expiresAt(
                                now.plus(
                                    expirationSeconds,
                                    ChronoUnit.SECONDS
                                )
                        )

                        .claim(
                                "roles",
                                authorities
                        )

                        .build();

        return jwtEncoder
                .encode(
                    JwtEncoderParameters.from(
                        org.springframework.security.oauth2.jose.jws.JwsHeader
                            .with(MacAlgorithm.HS256)
                            .build(),
                        claims
                    )
                )
                .getTokenValue();
    }
}
```

We're deliberately putting:

```json
"roles": [
    "ROLE_USER"
]
```

into the token.

---

# 13. Login Controller

### `AuthController.java`

```java
package com.practice.employeeapi.controller;

import com.practice.employeeapi.dto.AuthResponse;
import com.practice.employeeapi.dto.LoginRequest;
import com.practice.employeeapi.security.JwtService;

import jakarta.validation.Valid;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final long expirationSeconds;

    public AuthController(
            AuthenticationManager authenticationManager,
            JwtService jwtService,
            @Value("${app.jwt.expiration-seconds}")
            long expirationSeconds) {

        this.authenticationManager =
                authenticationManager;

        this.jwtService = jwtService;

        this.expirationSeconds =
                expirationSeconds;
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(
            @Valid
            @RequestBody
            LoginRequest request) {

        Authentication authentication =
                authenticationManager.authenticate(
                    UsernamePasswordAuthenticationToken
                        .unauthenticated(
                            request.username(),
                            request.password()
                        )
                );

        String token =
                jwtService.generateToken(
                        authentication
                );

        return ResponseEntity.ok(
                new AuthResponse(
                        token,
                        "Bearer",
                        expirationSeconds
                )
        );
    }
}
```

Spring Security's documented custom REST login pattern uses an `AuthenticationManager` to authenticate the supplied username/password credentials before issuing a response. ([docs.spring.io](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/index.html?utm_source=chatgpt.com))

---

# 14. JWT Role Converter

Our JWT has:

```json
"roles": [
    "ROLE_USER"
]
```

We need to convert this into Spring Security authorities.

### `JwtConfig.java`

Add:

```java
@Bean
JwtAuthenticationConverter jwtAuthenticationConverter() {

    JwtGrantedAuthoritiesConverter authorities =
            new JwtGrantedAuthoritiesConverter();

    authorities.setAuthoritiesClaimName("roles");

    authorities.setAuthorityPrefix("");

    JwtAuthenticationConverter converter =
            new JwtAuthenticationConverter();

    converter.setJwtGrantedAuthoritiesConverter(
            authorities
    );

    return converter;
}
```

Spring Security's `JwtAuthenticationConverter` allows you to customize how JWT claims become `GrantedAuthority` values. ([Home][3])

Why:

```text
JWT

"roles": ["ROLE_USER"]

       ↓

ROLE_USER

       ↓

GrantedAuthority
```

---

# 15. Security Configuration

Now the most important file.

### `SecurityConfig.java`

```java
package com.practice.employeeapi.config;

import com.practice.employeeapi.security.RestAccessDeniedHandler;
import com.practice.employeeapi.security.RestAuthenticationEntryPoint;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.http.HttpMethod;

import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;

import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;

import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtEncoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtEncoder;
import org.springframework.security.oauth2.server.resource.authentication.JwtAuthenticationConverter;
import org.springframework.security.oauth2.server.resource.authentication.JwtGrantedAuthoritiesConverter;

import org.springframework.security.web.SecurityFilterChain;

import javax.crypto.SecretKey;

@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    PasswordEncoder passwordEncoder() {

        return PasswordEncoderFactories
                .createDelegatingPasswordEncoder();
    }

    @Bean
    UserDetailsService userDetailsService(
            PasswordEncoder passwordEncoder) {

        UserDetailsService service =
                new InMemoryUserDetailsManager(
                    User.withUsername("rahul")
                        .password(
                            passwordEncoder.encode(
                                "password123"
                            )
                        )
                        .roles("USER")
                        .build(),

                    User.withUsername("admin")
                        .password(
                            passwordEncoder.encode(
                                "admin123"
                            )
                        )
                        .roles("ADMIN")
                        .build()
                );

        return service;
    }

    @Bean
    AuthenticationManager authenticationManager(
            UserDetailsService userDetailsService,
            PasswordEncoder passwordEncoder) {

        DaoAuthenticationProvider provider =
                new DaoAuthenticationProvider(
                        userDetailsService
                );

        provider.setPasswordEncoder(
                passwordEncoder
        );

        return new ProviderManager(provider);
    }

    @Bean
    JwtEncoder jwtEncoder(
            SecretKey jwtSecretKey) {

        return NimbusJwtEncoder
                .withSecretKey(jwtSecretKey)
                .build();
    }

    @Bean
    JwtDecoder jwtDecoder(
            SecretKey jwtSecretKey) {

        return NimbusJwtDecoder
                .withSecretKey(jwtSecretKey)
                .build();
    }

    @Bean
    JwtAuthenticationConverter
    jwtAuthenticationConverter() {

        JwtGrantedAuthoritiesConverter
                authoritiesConverter =
                    new JwtGrantedAuthoritiesConverter();

        authoritiesConverter
                .setAuthoritiesClaimName("roles");

        authoritiesConverter
                .setAuthorityPrefix("");

        JwtAuthenticationConverter converter =
                new JwtAuthenticationConverter();

        converter.setJwtGrantedAuthoritiesConverter(
                authoritiesConverter
        );

        return converter;
    }

    @Bean
    SecurityFilterChain securityFilterChain(
            HttpSecurity http,
            RestAuthenticationEntryPoint
                    authenticationEntryPoint,
            RestAccessDeniedHandler
                    accessDeniedHandler,
            JwtAuthenticationConverter
                    jwtAuthenticationConverter)
            throws Exception {

        http

            .cors(Customizer.withDefaults())

            /*
             * This API uses:
             *
             * Authorization: Bearer <JWT>
             *
             * rather than browser-managed
             * authentication cookies.
             */
            .csrf(csrf ->
                csrf.disable()
            )

            .sessionManagement(session ->
                session.sessionCreationPolicy(
                    SessionCreationPolicy.STATELESS
                )
            )

            .exceptionHandling(exceptions ->
                exceptions

                    .authenticationEntryPoint(
                        authenticationEntryPoint
                    )

                    .accessDeniedHandler(
                        accessDeniedHandler
                    )
            )

            .authorizeHttpRequests(auth -> auth

                /*
                 * Login must be public.
                 */
                .requestMatchers(
                    "/api/auth/login"
                )
                .permitAll()

                /*
                 * USER + ADMIN can read.
                 */
                .requestMatchers(
                    HttpMethod.GET,
                    "/api/employees/**"
                )
                .hasAnyRole(
                    "USER",
                    "ADMIN"
                )

                /*
                 * Only ADMIN can create.
                 */
                .requestMatchers(
                    HttpMethod.POST,
                    "/api/employees/**"
                )
                .hasRole("ADMIN")

                /*
                 * Only ADMIN can replace.
                 */
                .requestMatchers(
                    HttpMethod.PUT,
                    "/api/employees/**"
                )
                .hasRole("ADMIN")

                /*
                 * Only ADMIN can partially update.
                 */
                .requestMatchers(
                    HttpMethod.PATCH,
                    "/api/employees/**"
                )
                .hasRole("ADMIN")

                /*
                 * Only ADMIN can delete.
                 */
                .requestMatchers(
                    HttpMethod.DELETE,
                    "/api/employees/**"
                )
                .hasRole("ADMIN")

                /*
                 * Anything not explicitly allowed
                 * is denied.
                 */
                .anyRequest()
                .denyAll()
            )

            /*
             * Standard Spring Security JWT
             * Resource Server support.
             */
            .oauth2ResourceServer(
                oauth2 ->
                    oauth2.jwt(jwt ->
                        jwt.jwtAuthenticationConverter(
                            jwtAuthenticationConverter
                        )
                    )
            );

        return http.build();
    }
}
```

The key architectural point is that `oauth2ResourceServer().jwt()` uses Spring Security's built-in bearer-token support rather than requiring us to register a custom `OncePerRequestFilter`. Spring Security documents `BearerTokenAuthenticationFilter` as the filter used by Resource Server to parse bearer tokens. ([Home][4])

---

# 16. CORS Configuration

### `CorsConfig.java`

```java
package com.practice.employeeapi.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
public class CorsConfig {

    @Bean
    UrlBasedCorsConfigurationSource
    corsConfigurationSource(
            @Value("${app.cors.allowed-origin}")
            String allowedOrigin) {

        CorsConfiguration configuration =
                new CorsConfiguration();

        configuration.setAllowedOrigins(
                List.of(allowedOrigin)
        );

        configuration.setAllowedMethods(
                List.of(
                    "GET",
                    "POST",
                    "PUT",
                    "PATCH",
                    "DELETE",
                    "OPTIONS"
                )
        );

        configuration.setAllowedHeaders(
                List.of(
                    "Authorization",
                    "Content-Type"
                )
        );

        configuration.setExposedHeaders(
                List.of("Location")
        );

        configuration.setAllowCredentials(false);

        UrlBasedCorsConfigurationSource source =
                new UrlBasedCorsConfigurationSource();

        source.registerCorsConfiguration(
                "/**",
                configuration
        );

        return source;
    }
}
```

Spring Security integrates its CORS handling with Spring MVC CORS configuration; preflight requests need to be handled before authentication because browsers can send preflight requests without application credentials. ([Home][1])

---

# 17. 401 Handler

### `RestAuthenticationEntryPoint.java`

```java
package com.practice.employeeapi.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.practice.employeeapi.dto.ErrorResponse;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;

import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class RestAuthenticationEntryPoint
        implements AuthenticationEntryPoint {

    private final ObjectMapper objectMapper;

    public RestAuthenticationEntryPoint(
            ObjectMapper objectMapper) {

        this.objectMapper = objectMapper;
    }

    @Override
    public void commence(
            HttpServletRequest request,
            HttpServletResponse response,
            AuthenticationException ex)
            throws IOException {

        response.setStatus(
                HttpServletResponse.SC_UNAUTHORIZED
        );

        response.setContentType(
                MediaType.APPLICATION_JSON_VALUE
        );

        ErrorResponse error =
                new ErrorResponse(
                    401,
                    "Authentication required",
                    request.getRequestURI()
                );

        response.getWriter().write(
            objectMapper.writeValueAsString(error)
        );
    }
}
```

---

# 18. 403 Handler

### `RestAccessDeniedHandler.java`

```java
package com.practice.employeeapi.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.practice.employeeapi.dto.ErrorResponse;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.http.MediaType;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;

import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class RestAccessDeniedHandler
        implements AccessDeniedHandler {

    private final ObjectMapper objectMapper;

    public RestAccessDeniedHandler(
            ObjectMapper objectMapper) {

        this.objectMapper = objectMapper;
    }

    @Override
    public void handle(
            HttpServletRequest request,
            HttpServletResponse response,
            AccessDeniedException ex)
            throws IOException {

        response.setStatus(
                HttpServletResponse.SC_FORBIDDEN
        );

        response.setContentType(
                MediaType.APPLICATION_JSON_VALUE
        );

        ErrorResponse error =
                new ErrorResponse(
                    403,
                    "Access denied",
                    request.getRequestURI()
                );

        response.getWriter().write(
            objectMapper.writeValueAsString(error)
        );
    }
}
```

---

# 19. Error DTO

```java
package com.practice.employeeapi.dto;

import java.time.LocalDateTime;

public record ErrorResponse(

        int status,

        String message,

        String path,

        LocalDateTime timestamp

) {

    public ErrorResponse(
            int status,
            String message,
            String path) {

        this(
            status,
            message,
            path,
            LocalDateTime.now()
        );
    }
}
```

---

# 20. Protect the Service Too

URL security says:

```text
DELETE /api/employees/**
       ↓
ADMIN
```

We can also protect the business operation.

### `EmployeeService.java`

```java
@Service
public class EmployeeService {

    @PreAuthorize(
        "hasAnyRole('USER', 'ADMIN')"
    )
    @Transactional(readOnly = true)
    public EmployeeResponse getById(Long id) {

        // existing implementation
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    @Transactional
    public EmployeeResponse create(
            EmployeeCreateRequest request) {

        // existing implementation
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    @Transactional
    public EmployeeResponse update(
            Long id,
            EmployeeUpdateRequest request) {

        // existing implementation
    }

    @PreAuthorize(
        "hasRole('ADMIN')"
    )
    @Transactional
    public void delete(Long id) {

        // existing implementation
    }
}
```

Now we have two layers:

```text
HTTP Boundary
     ↓
SecurityFilterChain

Business Boundary
     ↓
@PreAuthorize
```

Method security is enabled through `@EnableMethodSecurity`. Spring Security's method-security support uses method interceptors to enforce annotations such as `@PreAuthorize`. ([Home][3])

---

# 21. Login Test

Start the application.

Then:

```bash
curl \
  -X POST \
  http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "rahul",
    "password": "password123"
  }'
```

You should receive:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

---

# 22. Decode the Token

You can inspect the JWT payload during development.

Conceptually it should contain:

```json
{
  "sub": "rahul",
  "iss": "employee-api",
  "roles": [
    "ROLE_USER"
  ],
  "iat":  ...,
  "exp":  ...
}
```

Remember:

> Decoding a JWT is not the same as validating it.

The server validates its signature and claims before trusting it.

---

# 23. Call GET as Rahul

```bash
curl \
  http://localhost:8080/api/employees \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Flow:

```text
JWT
 ↓
Validate
 ↓
ROLE_USER
 ↓
GET allowed
 ↓
EmployeeController
```

Expected:

```text
200 OK
```

---

# 24. Try DELETE as Rahul

```bash
curl \
  -X DELETE \
  http://localhost:8080/api/employees/1 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Rahul has:

```text
ROLE_USER
```

but DELETE requires:

```text
ROLE_ADMIN
```

Therefore:

```text
403 Forbidden
```

Response:

```json
{
  "status": 403,
  "message": "Access denied",
  "path": "/api/employees/1",
  "timestamp": "..."
}
```

---

# 25. Login as Admin

```bash
curl \
  -X POST \
  http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "admin123"
  }'
```

The JWT contains:

```json
{
  "sub": "admin",
  "roles": [
    "ROLE_ADMIN"
  ]
}
```

Now:

```bash
curl \
  -X DELETE \
  http://localhost:8080/api/employees/1 \
  -H "Authorization: Bearer ADMIN_TOKEN"
```

Authorization:

```text
ROLE_ADMIN
   ↓
DELETE requires ROLE_ADMIN
   ↓
Allowed
```

Expected:

```text
204 No Content
```

---

# 26. No Token

Try:

```bash
curl \
  http://localhost:8080/api/employees
```

No `Authorization` header.

Result:

```text
401 Unauthorized
```

Not:

```text
403
```

because the user hasn't authenticated.

---

# 27. Invalid Token

```bash
curl \
  http://localhost:8080/api/employees \
  -H "Authorization: Bearer abc.def.ghi"
```

JWT validation fails:

```text
BearerTokenAuthenticationFilter
        ↓
JwtDecoder
        ↓
Invalid JWT
        ↓
Authentication failure
        ↓
401
```

Spring Security's Resource Server bearer-token infrastructure handles these authentication failures before the request reaches your controller. ([Home][4])

---

# 28. The Complete Architecture

This is the diagram I want you to be able to draw in an interview:

```text
                         CLIENT
                            │
                            │
              POST /api/auth/login
                            │
                            ▼
                    AuthController
                            │
                            ▼
                 AuthenticationManager
                            │
                            ▼
               DaoAuthenticationProvider
                       /          \
                      /            \
                     ▼              ▼
          UserDetailsService   PasswordEncoder
                     │              │
                     └──────┬───────┘
                            ▼
                    Authentication
                            │
                            ▼
                       JwtEncoder
                            │
                            ▼
                           JWT
                            │
                            │
                            │ Authorization:
                            │ Bearer JWT
                            ▼
               BearerTokenAuthenticationFilter
                            │
                            ▼
                 JwtAuthenticationProvider
                       /            \
                      /              \
                     ▼                ▼
               JwtDecoder      JwtAuthenticationConverter
                     │                │
                     ▼                ▼
                  Valid JWT      ROLE_USER / ROLE_ADMIN
                       \              /
                        \            /
                         ▼          ▼
                       Authentication
                            │
                            ▼
                     SecurityContext
                            │
                            ▼
                      Authorization
                            │
               ┌────────────┴────────────┐
               │                         │
              USER                      ADMIN
               │                         │
             GET                    GET/POST/PUT/
                                    PATCH/DELETE
               │                         │
               └────────────┬────────────┘
                            ▼
                       Controller
                            │
                            ▼
                         Service
                            │
                       @PreAuthorize
                            │
                            ▼
                       Repository
                            │
                            ▼
                         Database
```

---

# 29. What We Have Covered

At this point Spring Security is genuinely complete for your target level:

```text
Spring Security
│
├── Authentication
├── Authorization
├── SecurityFilterChain
├── FilterChainProxy
├── AuthenticationManager
├── ProviderManager
├── AuthenticationProvider
├── DaoAuthenticationProvider
├── UserDetails
├── UserDetailsService
├── PasswordEncoder
├── BCrypt
├── Roles
├── Authorities
├── URL Authorization
├── Method Security
├── HTTP Basic
├── Stateful vs Stateless
├── JWT
├── JwtEncoder
├── JwtDecoder
├── JwtAuthenticationProvider
├── JwtAuthenticationConverter
├── BearerTokenAuthenticationFilter
├── SecurityContext
├── SecurityContextHolder
├── 401
├── 403
├── AuthenticationEntryPoint
├── AccessDeniedHandler
├── CORS
├── CSRF
├── Security Testing
└── Complete JWT REST API
```

---

# 30. What You Should Be Able to Explain in an Interview

If the interviewer asks:

> **"Explain how JWT authentication works in Spring Security."**

Your answer should be:

```text
1. User sends username/password to login endpoint.

2. The controller passes the credentials to
   AuthenticationManager.

3. AuthenticationManager delegates to
   DaoAuthenticationProvider.

4. DaoAuthenticationProvider loads the user
   through UserDetailsService and verifies the
   password through PasswordEncoder.

5. After successful authentication, the application
   generates a signed JWT containing claims such
   as subject, roles, issue time and expiration.

6. The client sends the JWT on subsequent requests
   using Authorization: Bearer <token>.

7. Spring Security's BearerTokenAuthenticationFilter
   extracts the token.

8. JwtAuthenticationProvider uses JwtDecoder to
   validate the JWT.

9. JwtAuthenticationConverter converts JWT claims
   into GrantedAuthority values.

10. Spring Security creates Authentication and stores
    it in the SecurityContext.

11. Authorization rules check the user's authorities.

12. If authorized, the request reaches the controller.
    Otherwise the client gets 401 or 403 depending
    on whether authentication or authorization failed.
```

That is a **strong 1.5–2 year Spring Boot interview answer**.

---

# 31. One Important Production Note

Our capstone generates JWTs inside the same application that protects the API.

That's excellent for learning the mechanics.

In production OAuth2/OIDC architectures, you will commonly encounter:

```text
Client
   ↓
Authorization Server
   ↓
Access Token
   ↓
Resource Server
```

The Resource Server validates the access token rather than inventing its own authentication protocol. Spring Security has dedicated Resource Server support for JWT and opaque bearer tokens, while Spring Authorization Server provides standardized token-issuance infrastructure. ([Home][1])

You should recognize that architecture when you start working with microservices.

---

# 32. Spring Security Status

```text
╔══════════════════════════════════════╗
║       SPRING SECURITY — DONE ✅      ║
╚══════════════════════════════════════╝

Core Concepts                 ✅
Authentication               ✅
Authorization                ✅
SecurityFilterChain          ✅
UserDetails                  ✅
Password Security             ✅
Roles / Authorities          ✅
Method Security              ✅
HTTP Basic                   ✅
Stateless Security           ✅
JWT Generation               ✅
JWT Validation               ✅
JWT Authorities              ✅
CORS                         ✅
CSRF                         ✅
401 / 403                    ✅
Security Testing             ✅
Complete REST API            ✅
```

## Next Module: **Spring Boot**

Now we move to the last major module you requested:

```text
Spring Boot
│
├── Chapter 1 — Why Spring Boot?
├── Chapter 2 — Spring vs Spring Boot
├── Chapter 3 — Spring Boot Starters
├── Chapter 4 — @SpringBootApplication
├── Chapter 5 — Auto-Configuration ⭐⭐⭐⭐⭐
├── Chapter 6 — Component Scanning
├── Chapter 7 — Conditional Configuration
├── Chapter 8 — Embedded Server
├── Chapter 9 — application.properties / YAML
├── Chapter 10 — Externalized Configuration
├── Chapter 11 — Profiles
├── Chapter 12 — @ConfigurationProperties
├── Chapter 13 — Actuator
├── Chapter 14 — Logging
├── Chapter 15 — Testing
└── Chapter 16 — Production / Deployment
```

The current Spring Boot documentation describes Boot as providing conventions and auto-configuration that simplify application setup, with starters providing convenient dependency groupings. ([Home][2])

**Next: Spring Boot Chapter 1 — Why Spring Boot?** We'll start from the problem Spring Boot was created to solve, then compare **traditional Spring configuration vs Spring Boot**, so Boot's auto-configuration doesn't feel like magic.

[1]: https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html?utm_source=chatgpt.com "OAuth 2.0 Resource Server JWT"
[2]: https://docs.spring.io/spring-boot/reference/using/build-systems.html?utm_source=chatgpt.com "Build Systems :: Spring Boot"
[3]: https://docs.spring.io/spring-security/reference/7.1-SNAPSHOT/api/java/org/springframework/security/oauth2/server/resource/authentication/JwtAuthenticationConverter.html?utm_source=chatgpt.com "Class JwtAuthenticationConverter"
[4]: https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/config/annotation/web/configurers/oauth2/server/resource/OAuth2ResourceServerConfigurer.html?utm_source=chatgpt.com "Class OAuth2ResourceServerConfigurer<H extends ..."
