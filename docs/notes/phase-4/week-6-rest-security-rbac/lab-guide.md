---
id: phase-4-week-6-lab-guide
tags: [spring, spring-security, rbac, basic-auth, bcrypt, jdbc-auth, lab, phase-4, week-6]
---

# :material-flask: Week 06 Lab Guide — Tiered Access REST Security Shield

> **Module:** Week 6 — Spring Security: Securing REST APIs & Role-Based Access Control  
> **Lab Repo:** [:material-github: 7amo10/Spring-Labs — Week-06-Stateless-REST-Security-Shield](https://github.com/7amo10/Spring-Labs/tree/main/Week-06-Stateless-REST-Security-Shield)

---

## :material-target: Lab Objective

Erect an enterprise security perimeter around the Employee Directory REST API utilizing **Spring Security 6/7**. Implement a custom `SecurityFilterChain` utilizing Lambda DSL syntax, configure stateless HTTP Basic authentication, disable CSRF protection appropriately for REST clients, provision users across role tiers (`EMPLOYEE`, `MANAGER`, `ADMIN`), migrate from in-memory authentication to database-backed **JDBC authentication** utilizing salted **BCrypt cryptographic password hashing**, and verify defensive boundaries against HTTP 401 Unauthorized and HTTP 403 Forbidden scenarios.

---

## :material-list-box: Key Concepts Covered

- **Spring Security Filter Chain:** Declaring a custom `SecurityFilterChain` bean via `HttpSecurity`.
- **Stateless HTTP Basic & CSRF:** Configuring `http.httpBasic()` and explaining why `http.csrf(csrf -> csrf.disable())` is safe and mandatory for non-browser REST APIs.
- **Role-Based Access Control (RBAC):** Defining path and HTTP method authorization rules (`hasRole("EMPLOYEE")`, `hasRole("MANAGER")`, `hasRole("ADMIN")`).
- **Database Authentication with JDBC:** Configuring `JdbcUserDetailsManager` against standard `users` and `authorities` database tables.
- **Cryptographic Password Hashing:** Generating and storing salted BCrypt hashes with `{bcrypt}` algorithm identifiers.
- **Custom Schema Adaptation:** Mapping non-standard database schemas (`members` and `roles`) using custom SQL queries via `setUsersByUsernameQuery()` and `setAuthoritiesByUsernameQuery()`.

---

## :material-code-braces: Component Architecture

Package: `com.luv2code.springboot.cruddemo`

| Component | Scope / Annotations | Responsibility |
| :--- | :--- | :--- |
| `DemoSecurityConfig` | `@Configuration` | Declares `SecurityFilterChain` and `UserDetailsManager` beans |
| `EmployeeRestController` | `@RestController` | Protected REST API exposing `/api/employees` endpoints |
| `EmployeeService` | `@Service` | Business logic tier coordinating persistence operations |
| `EmployeeRepository` | Interface (`JpaRepository`) | Spring Data JPA persistence tier accessing MySQL database |

---

## :material-database: Database Setup and Schema

Execute the SQL script against a local MySQL instance:

```sql
USE `employee_directory`;

DROP TABLE IF EXISTS `authorities`;
DROP TABLE IF EXISTS `users`;

-- 1. Table structure for table `users`
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` char(68) NOT NULL,
  `enabled` tinyint NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- 2. Insert BCrypt-hashed passwords (all users have password: fun123)
INSERT INTO `users` VALUES 
  ('john', '{bcrypt}$2a$10$qeS0HEh7urweMojsnwNAR.vcXJeXR1UcMRZ2WcGQl9YeuspUdgF.q', 1),
  ('mary', '{bcrypt}$2a$10$qeS0HEh7urweMojsnwNAR.vcXJeXR1UcMRZ2WcGQl9YeuspUdgF.q', 1),
  ('susan', '{bcrypt}$2a$10$qeS0HEh7urweMojsnwNAR.vcXJeXR1UcMRZ2WcGQl9YeuspUdgF.q', 1);

-- 3. Table structure for table `authorities`
CREATE TABLE `authorities` (
  `username` varchar(50) NOT NULL,
  `authority` varchar(50) NOT NULL,
  UNIQUE KEY `authorities_idx_1` (`username`,`authority`),
  CONSTRAINT `authorities_ibfk_1` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- 4. Assign role hierarchies
INSERT INTO `authorities` VALUES 
  ('john', 'ROLE_EMPLOYEE'),
  ('mary', 'ROLE_EMPLOYEE'),
  ('mary', 'ROLE_MANAGER'),
  ('susan', 'ROLE_EMPLOYEE'),
  ('susan', 'ROLE_MANAGER'),
  ('susan', 'ROLE_ADMIN');
```

---

## :material-hammer-wrench: Step-by-Step Implementation

### Step 1: Configure Security Dependencies

Ensure `pom.xml` contains Spring Security:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Step 2: Implement `DemoSecurityConfig` with JDBC Authentication

```java
package com.luv2code.springboot.cruddemo.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.provisioning.JdbcUserDetailsManager;
import org.springframework.security.provisioning.UserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

import javax.sql.DataSource;

@Configuration
public class DemoSecurityConfig {

    // 1. Configure JDBC authentication using the application DataSource
    @Bean
    public UserDetailsManager userDetailsManager(DataSource dataSource) {
        return new JdbcUserDetailsManager(dataSource);
    }

    // 2. Configure HTTP request authorization boundaries
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http.authorizeHttpRequests(configurer ->
                configurer
                        .requestMatchers(HttpMethod.GET, "/api/employees").hasRole("EMPLOYEE")
                        .requestMatchers(HttpMethod.GET, "/api/employees/**").hasRole("EMPLOYEE")
                        .requestMatchers(HttpMethod.POST, "/api/employees").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.PUT, "/api/employees").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.PATCH, "/api/employees/**").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.DELETE, "/api/employees/**").hasRole("ADMIN")
        );

        // Use HTTP Basic authentication
        http.httpBasic(Customizer.withDefaults());

        // Disable CSRF for stateless REST APIs
        http.csrf(csrf -> csrf.disable());

        return http.build();
    }
}
```

### Step 3: (Optional Extension) Adapting Custom Table Names

If your database utilizes non-standard table names (e.g., `members` and `roles`), configure custom SQL queries:

```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
    JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);

    jdbcUserDetailsManager.setUsersByUsernameQuery(
            "SELECT user_id, pw, active FROM members WHERE user_id = ?"
    );

    jdbcUserDetailsManager.setAuthoritiesByUsernameQuery(
            "SELECT user_id, role FROM roles WHERE user_id = ?"
    );

    return jdbcUserDetailsManager;
}
```

---

## :material-routes: Security Authorization Matrix

Base Path: `/api/employees`

| HTTP Method | Path Pattern | Required Authority / Role | Permitted Users | Blocked Users (403) |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/employees` | `ROLE_EMPLOYEE` | `john`, `mary`, `susan` | Unauthenticated (401) |
| `GET` | `/api/employees/{id}` | `ROLE_EMPLOYEE` | `john`, `mary`, `susan` | Unauthenticated (401) |
| `POST` | `/api/employees` | `ROLE_MANAGER` | `mary`, `susan` | `john` (403 Forbidden) |
| `PUT` | `/api/employees` | `ROLE_MANAGER` | `mary`, `susan` | `john` (403 Forbidden) |
| `PATCH` | `/api/employees/{id}` | `ROLE_MANAGER` | `mary`, `susan` | `john` (403 Forbidden) |
| `DELETE` | `/api/employees/{id}` | `ROLE_ADMIN` | `susan` | `john` (403), `mary` (403) |

---

## :material-check-decagram: Verification Checklist

Execute the following test cases via cURL or Postman:

| Test Phase | Request | Credentials | Expected Status | Verification Target |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 1: Unauthenticated** | `GET /api/employees` | None | `401 Unauthorized` | Response contains `WWW-Authenticate: Basic` header |
| **Phase 2: Bad Password** | `GET /api/employees` | `john` : `wrongPass` | `401 Unauthorized` | Invalid credentials rejected |
| **Phase 3: Employee Read** | `GET /api/employees` | `john` : `fun123` | `200 OK` | Successfully receives JSON array of employees |
| **Phase 4: Employee Write Attempt**| `POST /api/employees`<br/>`{"firstName":"Test","email":"t@t.com"}` | `john` : `fun123` | `403 Forbidden` | Access denied: `john` lacks `ROLE_MANAGER` |
| **Phase 5: Manager Write** | `POST /api/employees`<br/>`{"firstName":"David","lastName":"Adams","email":"d@luv2code.com"}` | `mary` : `fun123` | `200 OK` / `201 Created` | Creation succeeds for manager role |
| **Phase 6: Manager Delete Attempt**| `DELETE /api/employees/1` | `mary` : `fun123` | `403 Forbidden` | Access denied: `mary` lacks `ROLE_ADMIN` |
| **Phase 7: Admin Delete** | `DELETE /api/employees/1` | `susan` : `fun123` | `200 OK` | Deletion succeeds exclusively for admin role |

---

## :material-github: Lab Solution Repository

The complete implementation with all modules (default security, basic auth, role restrictions, JDBC plain-text, JDBC BCrypt, and custom queries) is accessible in the GitHub lab repository:

[:material-github: 7amo10/Spring-Labs — Week-06-Stateless-REST-Security-Shield](https://github.com/7amo10/Spring-Labs/tree/main/Week-06-Stateless-REST-Security-Shield)
