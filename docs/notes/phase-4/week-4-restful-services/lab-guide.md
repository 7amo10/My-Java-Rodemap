---
id: phase-4-week-4-lab-guide
tags: [spring, rest, crud, employee, service-layer, transactional, patch, lab, phase-4, week-4]
---

# :material-flask: Week 04 Lab Guide — Corporate Enterprise Employee REST Gateway

> **Module:** Week 4 — RESTful Web Services Architecture & Error Handling Pipelines  
> **Lab Repo:** [:material-github: 7amo10/Spring-Labs — Week-04-RESTful-Services-Error-Handling](https://github.com/7amo10/Spring-Labs/tree/main/Week-04-RESTful-Services-Error-Handling)

---

## :material-target: Lab Objective

Construct a production-grade, enterprise RESTful HR directory service adhering strictly to **3-Tier Architecture** (Presentation Tier → Business Service Tier → Data Access Tier). Implement complete CRUD endpoints utilizing proper HTTP verb semantics (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`), enforce service-tier transaction demarcation with `@Transactional`, implement partial resource modifications (`PATCH`) using Jackson's `JsonMapper.updateValue()`, and centralize exception interception using `@ControllerAdvice` returning standardized RFC-aligned error JSON payloads.

---

## :material-list-box: Key Concepts Covered

- **3-Tier Architecture:** Complete physical and logical decoupling of presentation (`@RestController`), business service (`@Service`), and data access (`@Repository`).
- **Service Transaction Boundaries:** Enforcing ACID transaction demarcation on the service layer rather than the DAO tier, ensuring multi-DAO operations execute within atomic boundaries.
- **RESTful Endpoints & HTTP Verbs:**
  - `GET /api/employees`: List all employee representations
  - `GET /api/employees/{id}`: Single resource retrieval with custom exception triggering on absent records
  - `POST /api/employees`: Resource creation with defensive primary key reset (`theEmployee.setId(0)`)
  - `PUT /api/employees`: Idempotent full entity replacement
  - `PATCH /api/employees/{id}`: Selective delta modification of entity fields via Jackson
  - `DELETE /api/employees/{id}`: Resource removal by primary key
- **Centralized Error Handling:** Global exception interception via `@ControllerAdvice` converting domain exceptions into structured JSON responses.

---

## :material-code-braces: Component Architecture

Package: `com.luv2code.springboot.cruddemo`

| Component | Scope / Annotations | Responsibility |
| :--- | :--- | :--- |
| `Employee` | `@Entity`, `@Table(name="employee")` | Domain entity mapped to relational table `employee` with auto-increment ID |
| `EmployeeDAO` | Interface | Persistence contract declaring low-level CRUD operations |
| `EmployeeDAOJpaImpl` | `@Repository` | JPA implementation utilizing `EntityManager` and `merge()` |
| `EmployeeService` | Interface | Business workflow contract decoupling controllers from DAOs |
| `EmployeeServiceImpl` | `@Service` | Orchestrates operations and manages `@Transactional` boundaries |
| `EmployeeRestController` | `@RestController`, `@RequestMapping("/api")` | Exposes REST HTTP endpoints, extracts request parameters, and delegates to service |
| `EmployeeRestExceptionHandler` | `@ControllerAdvice` | Intercepts exceptions application-wide and returns structured error JSON |
| `EmployeeNotFoundException` | Runtime Exception | Domain exception thrown when an employee record cannot be resolved |
| `EmployeeErrorResponse` | POJO | Standardized error payload model (`status`, `message`, `timeStamp`) |

---

## :material-database: Database Setup and Schema

Execute the initialization script against a local MySQL instance:

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
spring.datasource.url=jdbc:mysql://localhost:3306/employee_directory
spring.datasource.username=springstudent
spring.datasource.password=springstudent

# Hibernate logging
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.orm.jdbc.bind=trace
```

---

## :material-routes: REST Endpoints Specification

Base Path: `/api`

| HTTP Method | Endpoint | Request Body | Success Status | Error Status | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `GET` | `/api/employees` | None | `200 OK` | `500 Internal Error` | Retrieves array of all employees |
| `GET` | `/api/employees/{id}` | None | `200 OK` | `404 Not Found` | Retrieves single employee by ID |
| `POST` | `/api/employees` | JSON `Employee` | `200 OK` / `201 Created` | `400 Bad Request` | Creates a new employee (forces ID = 0) |
| `PUT` | `/api/employees` | JSON `Employee` with ID | `200 OK` | `400 Bad Request` | Full replacement update of existing record |
| `PATCH` | `/api/employees/{id}` | JSON Map of updated fields | `200 OK` | `400 Bad Request` / `404 Not Found` | Partially modifies specified fields |
| `DELETE` | `/api/employees/{id}` | None | `200 OK` | `404 Not Found` | Deletes employee record by ID |

---

## :material-hammer-wrench: Step-by-Step Implementation

### Step 1: Model Entity & Standard DAO

Map the `Employee` entity with `@Entity`, `@Id`, and `@GeneratedValue(strategy = GenerationType.IDENTITY)`.

Implement `EmployeeDAOJpaImpl` using `entityManager.merge(theEmployee)` inside `save()`. Remember that `merge()` acts as an `INSERT` if `id == 0`, and an `UPDATE` if `id > 0`.

### Step 2: Implement Service Layer with `@Transactional`

Declare `@Transactional` directly on mutating methods in `EmployeeServiceImpl`. Leave read operations unannotated or marked as `readOnly = true`:

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {
    private final EmployeeDAO employeeDAO;

    @Autowired
    public EmployeeServiceImpl(EmployeeDAO theEmployeeDAO) {
        this.employeeDAO = theEmployeeDAO;
    }

    @Override public List<Employee> findAll() { return employeeDAO.findAll(); }
    @Override public Employee findById(int theId) { return employeeDAO.findById(theId); }

    @Override
    @Transactional
    public Employee save(Employee theEmployee) { return employeeDAO.save(theEmployee); }

    @Override
    @Transactional
    public void deleteById(int theId) { employeeDAO.deleteById(theId); }
}
```

### Step 3: Centralize Error Handling via `@ControllerAdvice`

Implement the global exception handler to intercept domain errors:

```java
@ControllerAdvice
public class EmployeeRestExceptionHandler {

    @ExceptionHandler
    public ResponseEntity<EmployeeErrorResponse> handleException(EmployeeNotFoundException exc) {
        EmployeeErrorResponse error = new EmployeeErrorResponse();
        error.setStatus(HttpStatus.NOT_FOUND.value());
        error.setMessage(exc.getMessage());
        error.setTimeStamp(System.currentTimeMillis());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler
    public ResponseEntity<EmployeeErrorResponse> handleException(Exception exc) {
        EmployeeErrorResponse error = new EmployeeErrorResponse();
        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage(exc.getMessage());
        error.setTimeStamp(System.currentTimeMillis());
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

### Step 4: Implement `EmployeeRestController` with Full CRUD & `PATCH`

Implement the handler methods in `EmployeeRestController`:
- In `addEmployee`: Guarantee security by forcing `theEmployee.setId(0)` before saving.
- In `patchEmployee`: Intercept `Map<String, Object> patchPayload`, verify that `patchPayload.containsKey("id")` is false, and invoke `jsonMapper.updateValue(tempEmployee, patchPayload)`.

---

## :material-check-decagram: Verification Checklist

| Test Phase | Request | Verification Target | Expected Result |
| :--- | :--- | :--- | :--- |
| **Phase 1: Read All** | `GET /api/employees` | Response status and body | `200 OK` with JSON array containing 5 employees |
| **Phase 2: Read Single** | `GET /api/employees/1` | ID lookup | `200 OK` with Leslie Andrews JSON representation |
| **Phase 3: Not Found** | `GET /api/employees/999` | Global error interceptor | Global handler returns `404 NOT FOUND` with error JSON |
| **Phase 4: Defensive Create** | `POST /api/employees`<br/>`{"id": 1, "firstName":"David","lastName":"Adams","email":"david@luv2code.com"}` | ID reset check | `200 OK`, assigned generated ID `6` (did NOT overwrite employee #1!) |
| **Phase 5: Full Update**| `PUT /api/employees`<br/>`{"id":6,"firstName":"David","lastName":"Adams","email":"d.adams@luv2code.com"}` | Idempotency | `200 OK`, email updated in database |
| **Phase 6: Partial Update**| `PATCH /api/employees/6`<br/>`{"email":"david.adams@corporate.com"}` | In-memory delta merge | `200 OK`, only email updated; first and last names retained |
| **Phase 7: Security Guard**| `PATCH /api/employees/6`<br/>`{"id": 99, "email":"test@test.com"}` | Immutable ID verification | `400 BAD REQUEST`, error message states ID cannot be modified |
| **Phase 8: Delete** | `DELETE /api/employees/6` | Removal check | `200 OK`, subsequent `GET /api/employees/6` returns 404 |

---

## :material-github: Lab Solution Repository

The complete implementation with all packages (controller, service, dao, entity) and SQL seed scripts is available in the GitHub lab repository:

[:material-github: 7amo10/Spring-Labs — Week-04-RESTful-Services-Error-Handling](https://github.com/7amo10/Spring-Labs/tree/main/Week-04-RESTful-Services-Error-Handling)
