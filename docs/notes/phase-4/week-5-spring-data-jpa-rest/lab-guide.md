---
id: phase-4-week-5-lab-guide
tags: [spring, spring-data-jpa, spring-data-rest, swagger, openapi, pagination, lab, phase-4, week-5]
---

# :material-flask: Week 05 Lab Guide — Automated Paginated Fleet Management & Employee REST Engine

> **Module:** Week 5 — Spring Data JPA, Automated Repositories & Spring Data REST  
> **Lab Repo:** [:material-github: 7amo10/Spring-Labs — Week-05-Spring-Data-JPA-and-REST](https://github.com/7amo10/Spring-Labs/tree/main/Week-05-Spring-Data-JPA-and-REST)

---

## :material-target: Lab Objective

Refactor imperative DAO implementations into declarative **Spring Data JPA** interfaces (`JpaRepository<T, ID>`) to achieve zero-boilerplate persistence. Auto-generate hypermedia-compliant HATEOAS RESTful web services via **Spring Data REST**, configure global base paths (`spring.data.rest.base-path`), customize collection relation paths using `@RepositoryRestResource`, test client-driven pagination and sorting, and integrate interactive API documentation via **OpenAPI 3 / Swagger UI**.

---

## :material-list-box: Key Concepts Covered

- **Spring Data JPA Dynamic Repositories:** Replacing custom DAO implementation classes with single interface declarations extending `JpaRepository`.
- **Derived Query Methods:** Authoring custom read methods parsed automatically by Spring Data (`findByLastName`).
- **Spring Data REST:** Zero-code REST API publishing for repository interfaces using `spring-boot-starter-data-rest`.
- **HATEOAS HAL Wire Representation:** Navigating resource representations enriched with hypermedia links (`_links`, `_embedded`).
- **Paging & Sorting:** Query parameter controls (`?page=0&size=2`, `?sort=lastName,desc`).
- **Interactive API Documentation:** Integrating `springdoc-openapi-starter-webmvc-ui` for automated interactive sandboxes at `/swagger-ui.html`.

---

## :material-code-braces: Component Architecture

Package: `com.luv2code.springboot.cruddemo`

| Component | Scope / Annotations | Responsibility |
| :--- | :--- | :--- |
| `Employee` | `@Entity`, `@Table(name="employee")` | Domain entity mapped to relational table `employee` |
| `EmployeeRepository` | `@RepositoryRestResource(path="members")` | Declares `JpaRepository<Employee, Integer>` interface; zero boilerplate code |
| `CruddemoApplication` | `@SpringBootApplication` | Bootstrap entry point configuring container scanning and auto-configuration |

---

## :material-database: Database Setup and Schema

Execute the SQL script against a local MySQL instance:

```sql
CREATE DATABASE IF NOT EXISTS `employee_directory`;
USE `employee_directory`;

DROP TABLE IF EXISTS `employee`;

CREATE TABLE `employee` (
  `id` int NOT NULL AUTO_INCREMENT,
  `first_name` varchar(45) DEFAULT NULL,
  `last_name` varchar(45) DEFAULT NULL,
  `email` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;

INSERT INTO `employee` VALUES 
  (1,'Leslie','Andrews','leslie@luv2code.com'),
  (2,'Emma','Baumgarten','emma@luv2code.com'),
  (3,'Avani','Gupta','avani@luv2code.com'),
  (4,'Yuri','Petrov','yuri@luv2code.com'),
  (5,'Juan','Vega','juan@luv2code.com');
```

Configure `src/main/resources/application.properties`:

```properties
# Database Connectivity
spring.datasource.url=jdbc:mysql://localhost:3306/employee_directory
spring.datasource.username=springstudent
spring.datasource.password=springstudent

# Hibernate logging
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.orm.jdbc.bind=trace

# Spring Data REST Configuration
spring.data.rest.base-path=/magic-api
spring.data.rest.default-page-size=20
```

---

## :material-hammer-wrench: Step-by-Step Implementation

### Step 1: Configure Maven Dependencies

Update `pom.xml` to include Spring Data JPA, Spring Data REST, and OpenAPI Swagger UI:

```xml
<dependencies>
    <!-- Spring Data REST -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webmvc</artifactId>
    </dependency>

    <!-- MySQL JDBC Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- OpenAPI 3 / Swagger UI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>3.0.1</version>
    </dependency>
</dependencies>
```

### Step 2: Declare the `EmployeeRepository` Interface

Delete all legacy `EmployeeDAO` and `EmployeeDAOJpaImpl` classes. Create a clean repository interface:

```java
package com.luv2code.springboot.cruddemo.dao;

import com.luv2code.springboot.cruddemo.entity.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(path = "members")
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    // Custom query methods can be declared here if needed
}
```

### Step 3: Run the Application & Verify Auto-Generated Endpoints

Remove or comment out custom `@RestController` classes to let Spring Data REST govern the endpoints. Run the application and inspect the published paths under `/magic-api/members`.

---

## :material-routes: REST Endpoints Specification

Base Path: `/magic-api/members`

| HTTP Method | Endpoint | Query Parameters | Expected Response | Description |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/magic-api/members` | None | `200 OK` (HAL JSON) | Retrieves first page of member records with hypermedia links |
| `GET` | `/magic-api/members?page=0&size=2` | `page=0`, `size=2` | `200 OK` (HAL JSON) | Retrieves first page containing exactly 2 items with pagination controls |
| `GET` | `/magic-api/members?sort=lastName,desc` | `sort=lastName,desc` | `200 OK` (HAL JSON) | Returns records sorted by last name descending |
| `GET` | `/magic-api/members/1` | None | `200 OK` | Retrieves employee representation with ID 1 |
| `POST` | `/magic-api/members` | JSON `Employee` | `201 Created` | Inserts new employee record and returns assigned self link |
| `PUT` | `/magic-api/members/1` | JSON `Employee` | `200 OK` / `204 No Content` | Replaces existing employee record #1 |
| `PATCH` | `/magic-api/members/1` | JSON partial delta | `200 OK` / `204 No Content` | Modifies specified properties of record #1 |
| `DELETE` | `/magic-api/members/1` | None | `204 No Content` | Deletes record #1 |
| `GET` | `/swagger-ui.html` | None | `302 Redirect` | Redirects to interactive Swagger UI documentation dashboard |
| `GET` | `/v3/api-docs` | None | `200 OK` (JSON) | Produces machine-readable OpenAPI 3 specification |

---

## :material-check-decagram: Verification Checklist

| Test Phase | Request | Verification Target | Expected Result |
| :--- | :--- | :--- | :--- |
| **Phase 1: HAL Collection** | `GET /magic-api/members` | Wire format | `200 OK` HAL document containing `_embedded.members` and `_links.self` |
| **Phase 2: Pagination** | `GET /magic-api/members?page=0&size=2` | Page slice | Exactly 2 records returned; `page.totalElements` is 5; `_links.next` is present |
| **Phase 3: Page Navigation**| `GET /magic-api/members?page=1&size=2` | Next slice | Records 3 and 4 returned; `_links.prev` points back to page 0 |
| **Phase 4: Sorting** | `GET /magic-api/members?sort=lastName,desc` | Element order | Vega, Petrov, Gupta, Baumgarten, Andrews |
| **Phase 5: Single Item** | `GET /magic-api/members/2` | ID lookup | `200 OK` with Emma Baumgarten record and self link |
| **Phase 6: Resource Creation**| `POST /magic-api/members`<br/>`{"firstName":"David","lastName":"Adams","email":"david@luv2code.com"}` | Auto-persist | `201 Created` with `Location: http://localhost:8080/magic-api/members/6` |
| **Phase 7: Swagger UI** | `GET http://localhost:8080/swagger-ui.html` | Browser UI | Renders full interactive OpenAPI 3 dashboard displaying all models and operations |
| **Phase 8: OpenAPI JSON**| `GET http://localhost:8080/v3/api-docs` | API contract | Valid JSON document containing `"openapi": "3.0.1"` |

---

## :material-github: Lab Solution Repository

The full runnable source code with Maven build scripts and SQL seed files is accessible in the GitHub lab repository:

[:material-github: 7amo10/Spring-Labs — Week-05-Spring-Data-JPA-and-REST](https://github.com/7amo10/Spring-Labs/tree/main/Week-05-Spring-Data-JPA-and-REST)
