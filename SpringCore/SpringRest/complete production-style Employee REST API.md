Yes. Let's build the **complete Spring REST capstone**, not just fragments.

We'll use **Spring Boot 4.1.0 + Java 25 + Spring Framework 7.0.x**, with **Spring MVC + Spring JDBC + H2**. Spring Boot 4.1.0 requires Spring Framework 7.0.8+ and supports Java 25. The official starters include `spring-boot-starter-webmvc`, `spring-boot-starter-jdbc`, and `spring-boot-starter-validation`. ([Home][1])

I deliberately use **JdbcTemplate/NamedParameterJdbcTemplate instead of JPA** here because you've already learned Spring JDBC. We'll learn JPA separately later.

# 1. What we are building

```text
                   CLIENT
                     │
                     ▼
             Employee REST API
                     │
                     ▼
              EmployeeController
                     │
                     ▼
               EmployeeService
                     │
                     ▼
             EmployeeRepository
                     │
                     ▼
       NamedParameterJdbcTemplate
                     │
                     ▼
                  H2 DB
```

Features:

```text
POST    /api/employees
GET     /api/employees/{id}
GET     /api/employees
PUT     /api/employees/{id}
PATCH   /api/employees/{id}
DELETE  /api/employees/{id}

Filtering
Sorting
Pagination
Validation
Global Exception Handling
DTOs
Transactions
```

The Spring Boot JDBC auto-configuration provides `JdbcTemplate` and `NamedParameterJdbcTemplate`, so you can inject them directly. ([Home][2])

---

# 2. Project Structure

```text
employee-rest-api
│
├── pom.xml
│
└── src
    └── main
        ├── java
        │   └── com.practice.employeeapi
        │       │
        │       ├── EmployeeApiApplication.java
        │       │
        │       ├── config
        │       │   └── TransactionConfig.java
        │       │
        │       ├── controller
        │       │   └── EmployeeController.java
        │       │
        │       ├── dto
        │       │   ├── EmployeeCreateRequest.java
        │       │   ├── EmployeeUpdateRequest.java
        │       │   ├── EmployeeResponse.java
        │       │   ├── ErrorResponse.java
        │       │   └── PageResponse.java
        │       │
        │       ├── exception
        │       │   ├── EmployeeNotFoundException.java
        │       │   └── DuplicateEmployeeException.java
        │       │
        │       ├── model
        │       │   ├── Employee.java
        │       │   └── EmployeeStatus.java
        │       │
        │       ├── repository
        │       │   ├── EmployeeRepository.java
        │       │   └── EmployeeRowMapper.java
        │       │
        │       └── service
        │           └── EmployeeService.java
        │
        └── resources
            ├── application.properties
            ├── schema.sql
            └── data.sql
```

---

# 3. `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0
           https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.0</version>
        <relativePath/>
    </parent>

    <groupId>com.practice</groupId>
    <artifactId>employee-rest-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <name>employee-rest-api</name>
    <description>Spring REST Employee API</description>

    <properties>
        <java.version>25</java.version>
    </properties>

    <dependencies>

        <!-- Spring MVC + Embedded Tomcat -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
        </dependency>

        <!-- Spring JDBC + HikariCP -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <!-- Jakarta Bean Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- H2 Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

        </plugins>
    </build>

</project>
```

Spring Boot manages versions for its curated dependencies, so you generally should not add individual Spring Framework versions to these dependencies. ([Home][3])

---

# 4. Main Application

### `EmployeeApiApplication.java`

```java
package com.practice.employeeapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class EmployeeApiApplication {

    public static void main(String[] args) {

        SpringApplication.run(
                EmployeeApiApplication.class,
                args
        );
    }
}
```

---

# 5. Database Configuration

### `application.properties`

```properties
spring.application.name=employee-rest-api

server.port=8080

# H2 Database
spring.datasource.url=jdbc:h2:mem:employee_db
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver

# H2 Console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# SQL initialization
spring.sql.init.mode=always

# JDBC logging
logging.level.org.springframework.jdbc.core=DEBUG
```

H2 is in-memory, so every application restart recreates the database.

For this training project, that's convenient.

---

# 6. Database Schema

### `schema.sql`

```sql
CREATE TABLE employee (
    id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

    name VARCHAR(100) NOT NULL,

    email VARCHAR(150) NOT NULL UNIQUE,

    salary DECIMAL(15,2) NOT NULL,

    department VARCHAR(100) NOT NULL,

    status VARCHAR(20) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

# 7. Sample Data

### `data.sql`

```sql
INSERT INTO employee
(name, email, salary, department, status)
VALUES
('Rahul', 'rahul@example.com', 60000, 'IT', 'ACTIVE');

INSERT INTO employee
(name, email, salary, department, status)
VALUES
('Amit', 'amit@example.com', 75000, 'HR', 'ACTIVE');

INSERT INTO employee
(name, email, salary, department, status)
VALUES
('Priya', 'priya@example.com', 90000, 'IT', 'ACTIVE');

INSERT INTO employee
(name, email, salary, department, status)
VALUES
('John', 'john@example.com', 50000, 'FINANCE', 'INACTIVE');

INSERT INTO employee
(name, email, salary, department, status)
VALUES
('Sneha', 'sneha@example.com', 85000, 'IT', 'ACTIVE');
```

---

# 8. Employee Status

### `EmployeeStatus.java`

```java
package com.practice.employeeapi.model;

public enum EmployeeStatus {

    ACTIVE,
    INACTIVE
}
```

---

# 9. Employee Model

### `Employee.java`

```java
package com.practice.employeeapi.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Employee {

    private Long id;

    private String name;

    private String email;

    private BigDecimal salary;

    private String department;

    private EmployeeStatus status;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    public Employee() {
    }

    public Employee(
            Long id,
            String name,
            String email,
            BigDecimal salary,
            String department,
            EmployeeStatus status,
            LocalDateTime createdAt,
            LocalDateTime updatedAt) {

        this.id = id;
        this.name = name;
        this.email = email;
        this.salary = salary;
        this.department = department;
        this.status = status;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    // getters and setters

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public EmployeeStatus getStatus() {
        return status;
    }

    public void setStatus(EmployeeStatus status) {
        this.status = status;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }

    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }

    public void setUpdatedAt(LocalDateTime updatedAt) {
        this.updatedAt = updatedAt;
    }
}
```

---

# 10. Create Request DTO

### `EmployeeCreateRequest.java`

```java
package com.practice.employeeapi.dto;

import com.practice.employeeapi.model.EmployeeStatus;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.Size;

import java.math.BigDecimal;

public class EmployeeCreateRequest {

    @NotBlank(message = "Name is required")
    @Size(max = 100, message = "Name must not exceed 100 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email")
    @Size(max = 150, message = "Email must not exceed 150 characters")
    private String email;

    @NotNull(message = "Salary is required")
    @Positive(message = "Salary must be greater than zero")
    private BigDecimal salary;

    @NotBlank(message = "Department is required")
    @Size(max = 100)
    private String department;

    @NotNull(message = "Status is required")
    private EmployeeStatus status;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public EmployeeStatus getStatus() {
        return status;
    }

    public void setStatus(EmployeeStatus status) {
        this.status = status;
    }
}
```

---

# 11. Update DTO

For `PUT`, we'll require all fields.

### `EmployeeUpdateRequest.java`

```java
package com.practice.employeeapi.dto;

import com.practice.employeeapi.model.EmployeeStatus;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.Size;

import java.math.BigDecimal;

public class EmployeeUpdateRequest {

    @NotBlank(message = "Name is required")
    @Size(max = 100)
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email")
    @Size(max = 150)
    private String email;

    @NotNull(message = "Salary is required")
    @Positive(message = "Salary must be greater than zero")
    private BigDecimal salary;

    @NotBlank(message = "Department is required")
    @Size(max = 100)
    private String department;

    @NotNull(message = "Status is required")
    private EmployeeStatus status;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public EmployeeStatus getStatus() {
        return status;
    }

    public void setStatus(EmployeeStatus status) {
        this.status = status;
    }
}
```

---

# 12. PATCH DTO

PATCH allows partial changes.

### `EmployeePatchRequest.java`

```java
package com.practice.employeeapi.dto;

import com.practice.employeeapi.model.EmployeeStatus;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.Size;

import java.math.BigDecimal;

public class EmployeePatchRequest {

    @Size(min = 2, max = 100)
    private String name;

    @Email(message = "Invalid email")
    @Size(max = 150)
    private String email;

    @Positive(message = "Salary must be greater than zero")
    private BigDecimal salary;

    @Size(min = 2, max = 100)
    private String department;

    private EmployeeStatus status;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public EmployeeStatus getStatus() {
        return status;
    }

    public void setStatus(EmployeeStatus status) {
        this.status = status;
    }
}
```

---

# 13. Response DTO

### `EmployeeResponse.java`

```java
package com.practice.employeeapi.dto;

import com.practice.employeeapi.model.EmployeeStatus;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public record EmployeeResponse(
        Long id,
        String name,
        String email,
        BigDecimal salary,
        String department,
        EmployeeStatus status,
        LocalDateTime createdAt,
        LocalDateTime updatedAt
) {
}
```

---

# 14. Page Response

Instead of exposing Spring Data's `Page`—which we aren't using yet—we'll create our own API response.

### `PageResponse.java`

```java
package com.practice.employeeapi.dto;

import java.util.List;

public record PageResponse<T>(
        List<T> content,
        int page,
        int size,
        long totalElements,
        int totalPages,
        boolean first,
        boolean last,
        boolean hasNext
) {
}
```

---

# 15. Error Response

### `ErrorResponse.java`

```java
package com.practice.employeeapi.dto;

import java.time.LocalDateTime;
import java.util.Map;

public record ErrorResponse(
        int status,
        String message,
        LocalDateTime timestamp,
        String path,
        Map<String, String> errors
) {

    public ErrorResponse(
            int status,
            String message,
            String path) {

        this(
                status,
                message,
                LocalDateTime.now(),
                path,
                null
        );
    }
}
```

---

# 16. Custom Exceptions

### `EmployeeNotFoundException.java`

```java
package com.practice.employeeapi.exception;

public class EmployeeNotFoundException
        extends RuntimeException {

    public EmployeeNotFoundException(Long id) {

        super("Employee not found with id: " + id);
    }
}
```

### `DuplicateEmployeeException.java`

```java
package com.practice.employeeapi.exception;

public class DuplicateEmployeeException
        extends RuntimeException {

    public DuplicateEmployeeException(String email) {

        super("Employee already exists with email: " + email);
    }
}
```

---

# 17. RowMapper

### `EmployeeRowMapper.java`

```java
package com.practice.employeeapi.repository;

import com.practice.employeeapi.model.Employee;
import com.practice.employeeapi.model.EmployeeStatus;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

public class EmployeeRowMapper
        implements RowMapper<Employee> {

    @Override
    public Employee mapRow(
            ResultSet rs,
            int rowNum)
            throws SQLException {

        Employee employee = new Employee();

        employee.setId(rs.getLong("id"));

        employee.setName(
                rs.getString("name"));

        employee.setEmail(
                rs.getString("email"));

        employee.setSalary(
                rs.getBigDecimal("salary"));

        employee.setDepartment(
                rs.getString("department"));

        employee.setStatus(
                EmployeeStatus.valueOf(
                        rs.getString("status")));

        employee.setCreatedAt(
                rs.getTimestamp("created_at")
                        .toLocalDateTime());

        employee.setUpdatedAt(
                rs.getTimestamp("updated_at")
                        .toLocalDateTime());

        return employee;
    }
}
```

This is a good place to reinforce what you learned earlier:

```text
ResultSet row
     ↓
RowMapper
     ↓
Employee object
```

---

# 18. Repository

This is where the SQL lives.

### `EmployeeRepository.java`

```java
package com.practice.employeeapi.repository;

import com.practice.employeeapi.dto.EmployeeCreateRequest;
import com.practice.employeeapi.dto.EmployeePatchRequest;
import com.practice.employeeapi.dto.EmployeeUpdateRequest;
import com.practice.employeeapi.model.Employee;
import com.practice.employeeapi.model.EmployeeStatus;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.List;
import java.util.Map;

@Repository
public class EmployeeRepository {

    private final NamedParameterJdbcTemplate jdbc;

    private static final EmployeeRowMapper ROW_MAPPER =
            new EmployeeRowMapper();

    public EmployeeRepository(
            NamedParameterJdbcTemplate jdbc) {

        this.jdbc = jdbc;
    }

    public Employee save(
            EmployeeCreateRequest request) {

        String sql = """
                INSERT INTO employee
                (name, email, salary, department, status)
                VALUES
                (:name, :email, :salary, :department, :status)
                """;

        MapSqlParameterSource params =
                new MapSqlParameterSource()
                        .addValue("name", request.getName())
                        .addValue("email", request.getEmail())
                        .addValue("salary", request.getSalary())
                        .addValue(
                                "department",
                                request.getDepartment())
                        .addValue(
                                "status",
                                request.getStatus().name());

        try {

            jdbc.update(sql, params);

        } catch (DuplicateKeyException ex) {

            throw ex;
        }

        return findByEmail(request.getEmail());
    }

    public Employee findById(Long id) {

        String sql = """
                SELECT *
                FROM employee
                WHERE id = :id
                """;

        return jdbc.query(
                        sql,
                        Map.of("id", id),
                        ROW_MAPPER)
                .stream()
                .findFirst()
                .orElse(null);
    }

    public Employee findByEmail(String email) {

        String sql = """
                SELECT *
                FROM employee
                WHERE email = :email
                """;

        return jdbc.query(
                        sql,
                        Map.of("email", email),
                        ROW_MAPPER)
                .stream()
                .findFirst()
                .orElse(null);
    }

    public int update(
            Long id,
            EmployeeUpdateRequest request) {

        String sql = """
                UPDATE employee
                SET
                    name = :name,
                    email = :email,
                    salary = :salary,
                    department = :department,
                    status = :status,
                    updated_at = CURRENT_TIMESTAMP
                WHERE id = :id
                """;

        MapSqlParameterSource params =
                new MapSqlParameterSource()
                        .addValue("id", id)
                        .addValue("name", request.getName())
                        .addValue("email", request.getEmail())
                        .addValue("salary", request.getSalary())
                        .addValue(
                                "department",
                                request.getDepartment())
                        .addValue(
                                "status",
                                request.getStatus().name());

        return jdbc.update(sql, params);
    }

    public int patch(
            Long id,
            EmployeePatchRequest request) {

        StringBuilder sql = new StringBuilder("""
                UPDATE employee
                SET updated_at = CURRENT_TIMESTAMP
                """);

        MapSqlParameterSource params =
                new MapSqlParameterSource()
                        .addValue("id", id);

        if (request.getName() != null) {

            sql.append(", name = :name");

            params.addValue(
                    "name",
                    request.getName());
        }

        if (request.getEmail() != null) {

            sql.append(", email = :email");

            params.addValue(
                    "email",
                    request.getEmail());
        }

        if (request.getSalary() != null) {

            sql.append(", salary = :salary");

            params.addValue(
                    "salary",
                    request.getSalary());
        }

        if (request.getDepartment() != null) {

            sql.append(", department = :department");

            params.addValue(
                    "department",
                    request.getDepartment());
        }

        if (request.getStatus() != null) {

            sql.append(", status = :status");

            params.addValue(
                    "status",
                    request.getStatus().name());
        }

        sql.append(" WHERE id = :id");

        return jdbc.update(
                sql.toString(),
                params);
    }

    public int delete(Long id) {

        String sql = """
                DELETE FROM employee
                WHERE id = :id
                """;

        return jdbc.update(
                sql,
                Map.of("id", id));
    }

    public List<Employee> search(
            String name,
            String department,
            BigDecimal minSalary,
            EmployeeStatus status,
            int limit,
            int offset,
            String sortColumn,
            String sortDirection) {

        StringBuilder sql = new StringBuilder("""
                SELECT *
                FROM employee
                WHERE 1 = 1
                """);

        MapSqlParameterSource params =
                new MapSqlParameterSource();

        if (name != null && !name.isBlank()) {

            sql.append("""
                    AND LOWER(name)
                    LIKE LOWER(:name)
                    """);

            params.addValue(
                    "name",
                    "%" + name.trim() + "%");
        }

        if (department != null &&
                !department.isBlank()) {

            sql.append("""
                    AND LOWER(department)
                    = LOWER(:department)
                    """);

            params.addValue(
                    "department",
                    department.trim());
        }

        if (minSalary != null) {

            sql.append("""
                    AND salary >= :minSalary
                    """);

            params.addValue(
                    "minSalary",
                    minSalary);
        }

        if (status != null) {

            sql.append("""
                    AND status = :status
                    """);

            params.addValue(
                    "status",
                    status.name());
        }

        /*
         * sortColumn and sortDirection are NOT directly
         * supplied by the client. They have already been
         * validated/whitelisted by the service layer.
         */
        sql.append(" ORDER BY ")
                .append(sortColumn)
                .append(" ")
                .append(sortDirection);

        sql.append("""
                LIMIT :limit
                OFFSET :offset
                """);

        params.addValue("limit", limit);
        params.addValue("offset", offset);

        return jdbc.query(
                sql.toString(),
                params,
                ROW_MAPPER);
    }

    public long count(
            String name,
            String department,
            BigDecimal minSalary,
            EmployeeStatus status) {

        StringBuilder sql = new StringBuilder("""
                SELECT COUNT(*)
                FROM employee
                WHERE 1 = 1
                """);

        MapSqlParameterSource params =
                new MapSqlParameterSource();

        if (name != null && !name.isBlank()) {

            sql.append("""
                    AND LOWER(name)
                    LIKE LOWER(:name)
                    """);

            params.addValue(
                    "name",
                    "%" + name.trim() + "%");
        }

        if (department != null &&
                !department.isBlank()) {

            sql.append("""
                    AND LOWER(department)
                    = LOWER(:department)
                    """);

            params.addValue(
                    "department",
                    department.trim());
        }

        if (minSalary != null) {

            sql.append("""
                    AND salary >= :minSalary
                    """);

            params.addValue(
                    "minSalary",
                    minSalary);
        }

        if (status != null) {

            sql.append("""
                    AND status = :status
                    """);

            params.addValue(
                    "status",
                    status.name());
        }

        Long count = jdbc.queryForObject(
                sql.toString(),
                params,
                Long.class);

        return count == null ? 0 : count;
    }
}
```

Notice what we're practicing here:

```text
NamedParameterJdbcTemplate
        ↓
Named parameters
        ↓
RowMapper
        ↓
Filtering
        ↓
Sorting
        ↓
Pagination
```

Spring Boot's JDBC support auto-configures `NamedParameterJdbcTemplate`, and Spring's JDBC support handles the repetitive JDBC resource work. ([Home][2])

---

# 19. Service Layer

This is where business rules belong.

### `EmployeeService.java`

```java
package com.practice.employeeapi.service;

import com.practice.employeeapi.dto.EmployeeCreateRequest;
import com.practice.employeeapi.dto.EmployeePatchRequest;
import com.practice.employeeapi.dto.EmployeeResponse;
import com.practice.employeeapi.dto.EmployeeUpdateRequest;
import com.practice.employeeapi.dto.PageResponse;
import com.practice.employeeapi.exception.DuplicateEmployeeException;
import com.practice.employeeapi.exception.EmployeeNotFoundException;
import com.practice.employeeapi.model.Employee;
import com.practice.employeeapi.model.EmployeeStatus;
import com.practice.employeeapi.repository.EmployeeRepository;
import org.springframework.dao.DuplicateKeyException;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;
import java.util.List;
import java.util.Map;

@Service
public class EmployeeService {

    private static final int MAX_PAGE_SIZE = 100;

    private static final Map<String, String> SORT_FIELDS =
            Map.of(
                    "id", "id",
                    "name", "name",
                    "email", "email",
                    "salary", "salary",
                    "department", "department",
                    "status", "status",
                    "createdAt", "created_at"
            );

    private final EmployeeRepository repository;

    public EmployeeService(
            EmployeeRepository repository) {

        this.repository = repository;
    }

    @Transactional
    public EmployeeResponse create(
            EmployeeCreateRequest request) {

        if (repository.findByEmail(
                request.getEmail()) != null) {

            throw new DuplicateEmployeeException(
                    request.getEmail());
        }

        try {

            Employee employee =
                    repository.save(request);

            return toResponse(employee);

        } catch (DuplicateKeyException ex) {

            /*
             * Protect against a race condition where
             * another request inserts the same email
             * after our initial check.
             */
            throw new DuplicateEmployeeException(
                    request.getEmail());
        }
    }

    @Transactional(readOnly = true)
    public EmployeeResponse getById(Long id) {

        Employee employee =
                repository.findById(id);

        if (employee == null) {
            throw new EmployeeNotFoundException(id);
        }

        return toResponse(employee);
    }

    @Transactional
    public EmployeeResponse update(
            Long id,
            EmployeeUpdateRequest request) {

        getExisting(id);

        Employee existingEmail =
                repository.findByEmail(
                        request.getEmail());

        if (existingEmail != null &&
                !existingEmail.getId().equals(id)) {

            throw new DuplicateEmployeeException(
                    request.getEmail());
        }

        try {

            repository.update(id, request);

        } catch (DuplicateKeyException ex) {

            throw new DuplicateEmployeeException(
                    request.getEmail());
        }

        return getById(id);
    }

    @Transactional
    public EmployeeResponse patch(
            Long id,
            EmployeePatchRequest request) {

        getExisting(id);

        if (request.getEmail() != null) {

            Employee existingEmail =
                    repository.findByEmail(
                            request.getEmail());

            if (existingEmail != null &&
                    !existingEmail.getId().equals(id)) {

                throw new DuplicateEmployeeException(
                        request.getEmail());
            }
        }

        try {

            int updated =
                    repository.patch(id, request);

            if (updated == 0) {
                throw new EmployeeNotFoundException(id);
            }

        } catch (DuplicateKeyException ex) {

            throw new DuplicateEmployeeException(
                    request.getEmail());
        }

        return getById(id);
    }

    @Transactional
    public void delete(Long id) {

        int deleted =
                repository.delete(id);

        if (deleted == 0) {

            throw new EmployeeNotFoundException(id);
        }
    }

    @Transactional(readOnly = true)
    public PageResponse<EmployeeResponse> search(
            String name,
            String department,
            BigDecimal minSalary,
            EmployeeStatus status,
            int page,
            int size,
            String sort) {

        if (page < 0) {

            throw new IllegalArgumentException(
                    "page must be >= 0");
        }

        if (size < 1 || size > MAX_PAGE_SIZE) {

            throw new IllegalArgumentException(
                    "size must be between 1 and "
                    + MAX_PAGE_SIZE);
        }

        SortSpec sortSpec =
                parseSort(sort);

        int offset = page * size;

        List<Employee> employees =
                repository.search(
                        name,
                        department,
                        minSalary,
                        status,
                        size,
                        offset,
                        sortSpec.column(),
                        sortSpec.direction());

        long totalElements =
                repository.count(
                        name,
                        department,
                        minSalary,
                        status);

        int totalPages =
                totalElements == 0
                        ? 0
                        : (int) Math.ceil(
                                (double) totalElements
                                        / size);

        List<EmployeeResponse> responses =
                employees.stream()
                        .map(this::toResponse)
                        .toList();

        boolean first = page == 0;

        boolean last =
                totalPages == 0 ||
                page >= totalPages - 1;

        boolean hasNext =
                totalPages > 0 &&
                page < totalPages - 1;

        return new PageResponse<>(
                responses,
                page,
                size,
                totalElements,
                totalPages,
                first,
                last,
                hasNext
        );
    }

    private Employee getExisting(Long id) {

        Employee employee =
                repository.findById(id);

        if (employee == null) {

            throw new EmployeeNotFoundException(id);
        }

        return employee;
    }

    private EmployeeResponse toResponse(
            Employee employee) {

        return new EmployeeResponse(
                employee.getId(),
                employee.getName(),
                employee.getEmail(),
                employee.getSalary(),
                employee.getDepartment(),
                employee.getStatus(),
                employee.getCreatedAt(),
                employee.getUpdatedAt()
        );
    }

    private SortSpec parseSort(String sort) {

        String value =
                (sort == null || sort.isBlank())
                        ? "createdAt,desc"
                        : sort;

        String[] parts =
                value.split(",");

        if (parts.length != 2) {

            throw new IllegalArgumentException(
                    "sort must be in field,direction format");
        }

        String field = parts[0].trim();

        String direction =
                parts[1].trim().toLowerCase();

        String column =
                SORT_FIELDS.get(field);

        if (column == null) {

            throw new IllegalArgumentException(
                    "Unsupported sort field: " + field);
        }

        if (!direction.equals("asc") &&
                !direction.equals("desc")) {

            throw new IllegalArgumentException(
                    "sort direction must be asc or desc");
        }

        return new SortSpec(
                column,
                direction.toUpperCase());
    }

    private record SortSpec(
            String column,
            String direction) {
    }
}
```

---

# 20. Why is the sort allowlist important?

We never do this:

```java
sql.append(sortFromClient);
```

That would be dangerous.

Instead:

```text
Client
   ↓
sort=name,asc
   ↓
Service validates
   ↓
"name" → "name"
   ↓
SQL
```

Only known database columns are accepted.

That's an important real-world REST/API design practice.

---

# 21. Controller

### `EmployeeController.java`

```java
package com.practice.employeeapi.controller;

import com.practice.employeeapi.dto.EmployeeCreateRequest;
import com.practice.employeeapi.dto.EmployeePatchRequest;
import com.practice.employeeapi.dto.EmployeeResponse;
import com.practice.employeeapi.dto.EmployeeUpdateRequest;
import com.practice.employeeapi.dto.PageResponse;
import com.practice.employeeapi.model.EmployeeStatus;
import com.practice.employeeapi.service.EmployeeService;
import jakarta.validation.Valid;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;
import java.net.URI;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    private final EmployeeService service;

    public EmployeeController(
            EmployeeService service) {

        this.service = service;
    }

    // CREATE
    @PostMapping
    public ResponseEntity<EmployeeResponse> create(
            @Valid
            @RequestBody
            EmployeeCreateRequest request) {

        EmployeeResponse response =
                service.create(request);

        URI location =
                URI.create(
                        "/api/employees/"
                                + response.id());

        return ResponseEntity
                .created(location)
                .body(response);
    }

    // GET BY ID
    @GetMapping("/{id}")
    public ResponseEntity<EmployeeResponse> getById(
            @PathVariable Long id) {

        return ResponseEntity.ok(
                service.getById(id));
    }

    // SEARCH + FILTER + SORT + PAGINATION
    @GetMapping
    public ResponseEntity<
            PageResponse<EmployeeResponse>>
    search(

            @RequestParam(required = false)
            String name,

            @RequestParam(required = false)
            String department,

            @RequestParam(required = false)
            BigDecimal minSalary,

            @RequestParam(required = false)
            EmployeeStatus status,

            @RequestParam(defaultValue = "0")
            int page,

            @RequestParam(defaultValue = "20")
            int size,

            @RequestParam(
                    defaultValue = "createdAt,desc")
            String sort) {

        return ResponseEntity.ok(
                service.search(
                        name,
                        department,
                        minSalary,
                        status,
                        page,
                        size,
                        sort));
    }

    // FULL UPDATE
    @PutMapping("/{id}")
    public ResponseEntity<EmployeeResponse> update(
            @PathVariable Long id,

            @Valid
            @RequestBody
            EmployeeUpdateRequest request) {

        return ResponseEntity.ok(
                service.update(id, request));
    }

    // PARTIAL UPDATE
    @PatchMapping("/{id}")
    public ResponseEntity<EmployeeResponse> patch(
            @PathVariable Long id,

            @Valid
            @RequestBody
            EmployeePatchRequest request) {

        return ResponseEntity.ok(
                service.patch(id, request));
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(
            @PathVariable Long id) {

        service.delete(id);

        return ResponseEntity
                .noContent()
                .build();
    }
}
```

---

# 22. Global Exception Handler

### `GlobalExceptionHandler.java`

```java
package com.practice.employeeapi.exception;

import com.practice.employeeapi.dto.ErrorResponse;
import org.springframework.dao.DataAccessException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.LinkedHashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EmployeeNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            EmployeeNotFoundException ex) {

        ErrorResponse response =
                new ErrorResponse(
                        404,
                        ex.getMessage(),
                        "/api/employees"
                );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(response);
    }

    @ExceptionHandler(DuplicateEmployeeException.class)
    public ResponseEntity<ErrorResponse>
    handleDuplicate(
            DuplicateEmployeeException ex) {

        ErrorResponse response =
                new ErrorResponse(
                        409,
                        ex.getMessage(),
                        "/api/employees"
                );

        return ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(response);
    }

    @ExceptionHandler(
            MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse>
    handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors =
                new LinkedHashMap<>();

        ex.getBindingResult()
                .getFieldErrors()
                .forEach(error ->
                        errors.put(
                                error.getField(),
                                error.getDefaultMessage()
                        )
                );

        ErrorResponse response =
                new ErrorResponse(
                        400,
                        "Validation failed",
                        LocalDateTime.now(),
                        "/api/employees",
                        errors
                );

        return ResponseEntity
                .badRequest()
                .body(response);
    }

    @ExceptionHandler(
            HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse>
    handleMalformedJson(
            HttpMessageNotReadableException ex) {

        ErrorResponse response =
                new ErrorResponse(
                        400,
                        "Malformed request body",
                        "/api/employees"
                );

        return ResponseEntity
                .badRequest()
                .body(response);
    }

    @ExceptionHandler(
            IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse>
    handleBadRequest(
            IllegalArgumentException ex) {

        ErrorResponse response =
                new ErrorResponse(
                        400,
                        ex.getMessage(),
                        "/api/employees"
                );

        return ResponseEntity
                .badRequest()
                .body(response);
    }

    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ErrorResponse>
    handleDatabaseError(
            DataAccessException ex) {

        ErrorResponse response =
                new ErrorResponse(
                        500,
                        "Database operation failed",
                        "/api/employees"
                );

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse>
    handleUnexpected(Exception ex) {

        ErrorResponse response =
                new ErrorResponse(
                        500,
                        "Internal server error",
                        "/api/employees"
                );

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(response);
    }
}
```

For larger applications, Spring also provides `ResponseEntityExceptionHandler` as a convenient base class for centralized MVC exception handling. ([Home][4])

---

# 23. Transaction Configuration

Because we're using JDBC and you've already learned transaction management, let's wire it explicitly.

### `TransactionConfig.java`

```java
package com.practice.employeeapi.config;

import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.support.JdbcTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
public class TransactionConfig {

    @Bean
    public PlatformTransactionManager transactionManager(
            DataSource dataSource) {

        return new JdbcTransactionManager(dataSource);
    }
}
```

`JdbcTransactionManager` is Spring's JDBC-oriented `PlatformTransactionManager` implementation and adds JDBC exception translation around commit/rollback. ([Home][5])

Our `@Transactional` annotations in the service layer therefore fit directly into the architecture we've already studied.

---

# 24. Complete API

Now your API is:

```text
POST   /api/employees
GET    /api/employees/{id}
GET    /api/employees
PUT    /api/employees/{id}
PATCH  /api/employees/{id}
DELETE /api/employees/{id}
```

With:

```text
Filtering
Sorting
Pagination
Validation
DTOs
Exception handling
Transactions
JDBC
```

---

# 25. Test It

Start:

```bash
mvn spring-boot:run
```

Or:

```bash
mvn clean package
java -jar target/employee-rest-api-0.0.1-SNAPSHOT.jar
```

Spring Boot's Maven integration supports executable application packaging, and Boot applications can be run as standalone applications. ([Home][6])

---

# 26. CREATE Employee

### Request

```http
POST http://localhost:8080/api/employees
Content-Type: application/json
```

```json
{
  "name": "Vikram",
  "email": "vikram@example.com",
  "salary": 80000,
  "department": "IT",
  "status": "ACTIVE"
}
```

### Response

```http
201 Created
Location: /api/employees/6
```

```json
{
  "id": 6,
  "name": "Vikram",
  "email": "vikram@example.com",
  "salary": 80000,
  "department": "IT",
  "status": "ACTIVE",
  "createdAt": "...",
  "updatedAt": "..."
}
```

---

# 27. GET Employee

```http
GET http://localhost:8080/api/employees/1
```

Response:

```json
{
  "id": 1,
  "name": "Rahul",
  "email": "rahul@example.com",
  "salary": 60000,
  "department": "IT",
  "status": "ACTIVE",
  "createdAt": "...",
  "updatedAt": "..."
}
```

---

# 28. GET All Employees

```http
GET http://localhost:8080/api/employees
```

Default:

```text
page = 0
size = 20
sort = createdAt,desc
```

---

# 29. Pagination

```http
GET http://localhost:8080/api/employees?page=0&size=2
```

Response shape:

```json
{
  "content": [
    {
      "id": 5,
      "name": "Sneha"
    },
    {
      "id": 3,
      "name": "Priya"
    }
  ],
  "page": 0,
  "size": 2,
  "totalElements": 5,
  "totalPages": 3,
  "first": true,
  "last": false,
  "hasNext": true
}
```

---

# 30. Sorting

```http
GET http://localhost:8080/api/employees?sort=salary,desc
```

Or:

```http
GET http://localhost:8080/api/employees?sort=name,asc
```

Only our allowlisted fields are accepted.

---

# 31. Filtering

By department:

```http
GET http://localhost:8080/api/employees?department=IT
```

By name:

```http
GET http://localhost:8080/api/employees?name=rahul
```

Minimum salary:

```http
GET http://localhost:8080/api/employees?minSalary=70000
```

Status:

```http
GET http://localhost:8080/api/employees?status=ACTIVE
```

---

# 32. Everything Together

This is the kind of API you see in actual enterprise applications:

```http
GET /api/employees
    ?department=IT
    &minSalary=60000
    &status=ACTIVE
    &page=0
    &size=20
    &sort=salary,desc
```

Flow:

```text
HTTP Request
      ↓
DispatcherServlet
      ↓
HandlerMapping
      ↓
HandlerAdapter
      ↓
Controller
      ↓
@RequestParam
      ↓
Service
      ↓
Validate Sort / Pagination
      ↓
Repository
      ↓
NamedParameterJdbcTemplate
      ↓
Database
      ↓
Filter
      ↓
Sort
      ↓
LIMIT / OFFSET
      ↓
RowMapper
      ↓
Employee
      ↓
Response DTO
      ↓
JSON
```

---

# 33. Validation Example

Send:

```json
{
  "name": "",
  "email": "abc",
  "salary": -500,
  "department": "",
  "status": "ACTIVE"
}
```

Response:

```http
400 Bad Request
```

```json
{
  "status": 400,
  "message": "Validation failed",
  "timestamp": "...",
  "path": "/api/employees",
  "errors": {
    "name": "Name is required",
    "email": "Invalid email",
    "salary": "Salary must be greater than zero",
    "department": "Department is required"
  }
}
```

---

# 34. Employee Not Found

```http
GET /api/employees/999
```

Response:

```http
404 Not Found
```

```json
{
  "status": 404,
  "message": "Employee not found with id: 999"
}
```

---

# 35. Duplicate Email

```http
POST /api/employees
```

with an existing email:

```json
{
  "name": "Another Rahul",
  "email": "rahul@example.com",
  "salary": 70000,
  "department": "IT",
  "status": "ACTIVE"
}
```

Response:

```http
409 Conflict
```

```json
{
  "status": 409,
  "message": "Employee already exists with email: rahul@example.com"
}
```

---

# 36. What you have implemented

You now have one project using almost everything we've learned:

```text
Spring Core
    ↓
Dependency Injection

Spring MVC
    ↓
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
@PathVariable
@RequestParam
@RequestBody

Spring REST
    ↓
HTTP Status Codes
ResponseEntity
DTOs
Validation
Error Handling
Pagination
Sorting
Filtering

Spring JDBC
    ↓
NamedParameterJdbcTemplate
RowMapper
SQL
DataAccessException

Spring Transactions
    ↓
@Transactional
JdbcTransactionManager
```

This is a very good **1.5–2 year interview-level project**.

Spring MVC's annotated-controller model supports request mappings, input binding, exception handling, and related controller features, while Spring MVC's response handling uses `HttpMessageConverter` for representations such as JSON. ([Home][7])

---

# 📍 Where We Are Now

```text
Spring Core                  ✅
Spring AOP                   ✅
Spring JDBC                  ✅
Spring Transactions          ✅
Spring MVC                   ✅
Spring REST                  ✅ ← Capstone Complete

        ↓

NEXT: SPRING BOOT ⭐⭐⭐⭐⭐
```

And **this is the right point to start Spring Boot**, because you now know what Boot is actually simplifying rather than treating Boot as magic.

We'll start Spring Boot from **Why → What → How → Where**, beginning with:

```text
Why Spring Boot?
      ↓
Problems with traditional Spring configuration
      ↓
Spring Boot solution
      ↓
@SpringBootApplication
      ↓
Auto-Configuration
      ↓
Starter Dependencies
      ↓
Embedded Tomcat
      ↓
SpringApplication.run()
```

[1]: https://docs.spring.io/spring-boot/system-requirements.html?utm_source=chatgpt.com "System Requirements :: Spring Boot"
[2]: https://docs.spring.io/spring-boot/reference/data/sql.html "SQL Databases :: Spring Boot"
[3]: https://docs.spring.io/spring-boot/reference/using/build-systems.html "Build Systems :: Spring Boot"
[4]: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html?utm_source=chatgpt.com "Error Responses"
[5]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/support/JdbcTransactionManager.html?utm_source=chatgpt.com "JdbcTransactionManager (Spring Framework 7.0.8 API)"
[6]: https://docs.spring.io/spring-boot/index.html?utm_source=chatgpt.com "Spring Boot"
[7]: https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html?utm_source=chatgpt.com "Annotated Controllers"
