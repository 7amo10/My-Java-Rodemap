---
id: phase-4-week-8-summary
tags: [spring, spring-mvc, spring-security, fullstack-crud, prg-pattern, rbac, summary, reference, phase-4]
---

# Summary & Architecture Reference — Week 8: Spring MVC CRUD & Form-Based Security

---

## 1. Annotation & Class Reference Matrix

| Class / Annotation | Module / Package | Architectural Role | Enterprise Purpose | Low-Level Mechanism / Lifecycle Impact | Common Senior Pitfall |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`@Controller`** | `org.springframework.stereotype` | Presentation Bean | Declares web controller serving server-side rendered HTML views. | Handler return strings resolve to logical view names via `ViewResolver`. | Using `@RestController`, which serializes return strings directly to plain text response bodies. |
| **`SecurityFilterChain`** | `org.springframework.security.web` | Configuration Bean | Defines the ordered filter pipeline governing web access. | Evaluated by `FilterChainProxy` on every incoming HTTP request. | Disabling CSRF in session-based form applications, exposing the system to forgery exploits. |
| **`formLogin()`** | `org.springframework.security.config` | DSL Method | Replaces HTTP Basic with stateful form-based authentication. | Injects `UsernamePasswordAuthenticationFilter` into the filter pipeline. | Creating an unnecessary controller endpoint for `loginProcessingUrl`. |
| **`UsernamePasswordAuthenticationFilter`** | `org.springframework.security.web.authentication` | Security Filter | Intercepts `POST` requests to `loginProcessingUrl`. | Extracts `username` and `password`, delegates to `AuthenticationManager`, creates `SecurityContext`. | Using GET method for login submission, leaking credentials into server access logs. |
| **`CsrfFilter`** | `org.springframework.security.web.csrf` | Security Filter | Guards state-mutating requests (`POST`, `PUT`, `DELETE`) with CSRF tokens. | Validates submitted `_csrf` token against session-stored token. | Using plain HTML `<form action="...">` without `th:action`, omitting the token and causing 403 errors. |
| **`LogoutFilter`** | `org.springframework.security.web.authentication.logout` | Security Filter | Intercepts `POST /logout` to invalidate the session. | Destroys `HttpSession`, wipes `SecurityContextHolder`, deletes `JSESSIONID` cookie. | Allowing `GET` logout, leaving users vulnerable to third-party image tag logout triggers. |
| **`JdbcUserDetailsManager`** | `org.springframework.security.provisioning` | Credential Manager | Queries database credentials over JDBC `DataSource`. | Executes configured user and authority queries with prepared statements. | Mapping queries with missing or misordered column projections. |
| **`JpaRepository`** | `org.springframework.data.jpa.repository` | Persistence Interface | Provides full CRUD and derived sorting queries. | Generates dynamic runtime proxies backed by `SimpleJpaRepository`. | Omitting the hidden ID in forms, causing updates to trigger SQL `INSERT` instead of `UPDATE`. |
| **`sec:authorize`** | `org.thymeleaf.extras.springsecurity6` | Dialect Attribute | Conditionally evaluates SpEL expressions on the active principal. | Renders DOM element only if SpEL expression evaluates to `true`. | Assuming `sec:authorize` provides security without configuring backend route rules. |
| **`sec:authentication`** | `org.thymeleaf.extras.springsecurity6` | Dialect Attribute | Extracts properties from the current `Authentication` principal. | Injects principal username or authorities directly into tag body. | Attempting to access non-existent properties on custom `UserDetails` objects. |

---

## 2. Post-Redirect-Get (PRG) Complete Lifecycle Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant Browser as Web Browser Client
    participant Tomcat as Servlet Container (Tomcat)
    participant SecChain as Spring Security Filter Chain
    participant DS as DispatcherServlet
    participant Controller as StaffAdministrationController
    participant Service as EmployeeServiceImpl
    participant Repo as EmployeeRepository
    participant DB as MySQL Database
    participant View as Thymeleaf Engine

    Note over Browser,DB: Phase 1: Mutating Submission (POST)
    Browser->>Tomcat: POST /staff/save (Employee Form Data + _csrf)
    Tomcat->>SecChain: Execute Filter Chain
    SecChain->>SecChain: CsrfFilter validates _csrf token (OK)
    SecChain->>SecChain: AuthorizationFilter verifies ROLE_MANAGER (OK)
    SecChain->>DS: Delegate to DispatcherServlet
    DS->>Controller: saveEmployee(theEmployee)
    Controller->>Service: save(theEmployee)
    Service->>Repo: save(theEmployee)
    Repo->>DB: Executes SQL INSERT or UPDATE
    DB-->>Repo: Saved Entity with ID
    Repo-->>Service: Employee Entity
    Service-->>Controller: Return Employee
    Controller-->>DS: Return redirect to /staff/list
    DS-->>Tomcat: Issue HTTP 302 Redirect (Location: /staff/list)
    Tomcat-->>Browser: HTTP 302 Found

    Note over Browser,DB: Phase 2: Idempotent Fetch (GET)
    Browser->>Tomcat: GET /staff/list (Follows Redirect)
    Tomcat->>SecChain: Execute Filter Chain
    SecChain->>SecChain: AuthorizationFilter verifies ROLE_EMPLOYEE (OK)
    SecChain->>DS: Delegate to DispatcherServlet
    DS->>Controller: listEmployees(theModel)
    Controller->>Service: findAll()
    Service->>Repo: findAllByOrderByLastNameAsc()
    Repo->>DB: Executes SELECT * FROM employee ORDER BY lastName ASC
    DB-->>Repo: List of Employees
    Repo-->>Service: List of Employees
    Service-->>Controller: Return List
    Controller->>Controller: theModel.addAttribute("employees", list)
    Controller-->>DS: Return logical view name staff/staff-list
    DS->>View: Render "staff/staff-list" with Model
    View->>View: Evaluate sec:authorize and inject data
    View-->>Tomcat: Rendered HTML Response Stream
    Tomcat-->>Browser: HTTP 200 OK (Content-Type: text/html)
```

---

## 3. Form Login Authentication Filter Execution Flow

```mermaid
sequenceDiagram
    autonumber
    participant Browser as Web Browser Client
    participant Tomcat as Tomcat Container
    participant UPAF as UsernamePasswordAuthenticationFilter
    participant AM as ProviderManager (AuthenticationManager)
    participant DaoProvider as DaoAuthenticationProvider
    participant UserStore as JdbcUserDetailsManager
    participant Encoder as BCryptPasswordEncoder
    participant SessionStrategy as SessionAuthenticationStrategy

    Browser->>Tomcat: POST /authenticate (username='mary', password='fun123', _csrf)
    Tomcat->>UPAF: Intercepts POST to loginProcessingUrl
    UPAF->>UPAF: Extracts username & password parameters
    UPAF->>AM: authenticate(UsernamePasswordAuthenticationToken)
    AM->>DaoProvider: authenticate(authentication)
    DaoProvider->>UserStore: loadUserByUsername("mary")
    UserStore-->>DaoProvider: UserDetails (username='mary', passwordHash='{bcrypt}$2a$...', roles)
    DaoProvider->>Encoder: matches("fun123", passwordHash)
    Encoder->>Encoder: Computes BCrypt hash with stored salt
    Encoder-->>DaoProvider: Matches (true)
    DaoProvider-->>AM: Fully Populated Authentication Token
    AM-->>UPAF: Authentication Success
    UPAF->>SessionStrategy: onAuthentication (Prevent Session Fixation)
    UPAF->>UPAF: Store SecurityContext in HttpSession
    UPAF-->>Tomcat: HTTP 302 Redirect to defaultSuccessUrl (/staff/list)
    Tomcat-->>Browser: HTTP 302 Found (Set-Cookie: JSESSIONID=...)
```

---

## 4. Architectural Comparison: HTTP Basic vs Form-Based Authentication

| Dimension | HTTP Basic Authentication | Form-Based Authentication |
| :--- | :--- | :--- |
| **Primary Architecture** | Stateless REST APIs / Microservices | Stateful Server-Side Rendered Web Portals |
| **User Interface** | Unstyled, browser-native pop-up modal | Custom branded HTML / CSS (Bootstrap 5) |
| **Session Model** | Stateless (`STATELESS` session creation) | Stateful (`IF_REQUIRED` or `ALWAYS` session creation) |
| **Credential Lifetime** | Transmitted on every single HTTP request | Transmitted once during login; session cookie thereafter |
| **Logout Capabilities** | Difficult / non-standard in browsers | Clean `POST /logout` invalidating session & cookies |
| **CSRF Requirement** | Disabled (`csrf.disable()`) | **Strictly Enabled** (Mandatory for session cookies) |
| **Unauthorized Outcome** | Sends `WWW-Authenticate: Basic` $\rightarrow$ Browser popup | Issues HTTP 302 redirect to custom `/login` page |
| **Spring DSL Hook** | `http.httpBasic(Customizer.withDefaults())` | `http.formLogin(form -> form.loginPage(...))` |

---

## 5. Thymeleaf Spring Security 6 Dialect Reference

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

| Tag Expression | Type | Purpose | Example |
| :--- | :--- | :--- | :--- |
| **`sec:authentication="name"`** | Attribute | Outputs authenticated username | `<span sec:authentication="name"></span>` |
| **`sec:authentication="principal.username"`** | Attribute | Outputs username from `UserDetails` | `<span sec:authentication="principal.username"></span>` |
| **`sec:authentication="principal.authorities"`** | Attribute | Outputs list of granted authorities | `<span sec:authentication="principal.authorities"></span>` |
| **`sec:authorize="hasRole('ADMIN')"`** | Conditional | Renders element if principal has `ROLE_ADMIN` | `<div sec:authorize="hasRole('ADMIN')">...</div>` |
| **`sec:authorize="hasAnyRole('MGR','ADM')"`** | Conditional | Renders if principal holds any listed role | `<div sec:authorize="hasAnyRole('MANAGER','ADMIN')">` |
| **`sec:authorize="isAuthenticated()"`** | Conditional | Renders element if user is logged in | `<div sec:authorize="isAuthenticated()">...</div>` |
| **`sec:authorize="isAnonymous()"`** | Conditional | Renders element if user is a guest | `<div sec:authorize="isAnonymous()">...</div>` |
| **`sec:authorize="hasAuthority('DELETE')"`** | Conditional | Renders element matching exact authority | `<button sec:authorize="hasAuthority('DELETE')">` |

---

## 6. Custom JDBC Authentication Query Contracts

When adapting non-standard relational databases, `JdbcUserDetailsManager` requires strict compliance with projection contracts:

```mermaid
flowchart TD
    subgraph UsersQuery["1. setUsersByUsernameQuery() Contract"]
        UQ["SELECT username_col, password_col, enabled_col FROM members_table WHERE username_col=?"]
        UQ1["Column 1: Username (String)"]
        UQ2["Column 2: Password Hash (String with {bcrypt})"]
        UQ3["Column 3: Enabled / Active (boolean or 1/0)"]
        UQ --> UQ1
        UQ --> UQ2
        UQ --> UQ3
    end

    subgraph AuthoritiesQuery["2. setAuthoritiesByUsernameQuery() Contract"]
        AQ["SELECT username_col, authority_col FROM roles_table WHERE username_col=?"]
        AQ1["Column 1: Username (String)"]
        AQ2["Column 2: Authority / Role String (e.g. 'ROLE_EMPLOYEE')"]
        AQ --> AQ1
        AQ --> AQ2
    end
```

### Reference Configuration:
```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
    JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);
    
    // Exact 3-column projection contract
    jdbcUserDetailsManager.setUsersByUsernameQuery(
        "select user_id, pw, active from members where user_id=?");
        
    // Exact 2-column projection contract
    jdbcUserDetailsManager.setAuthoritiesByUsernameQuery(
        "select user_id, role from roles where user_id=?");
        
    return jdbcUserDetailsManager;
}
```
