---
id: phase-4-week-11-section-3-in-memory-identity-management
aliases: []
tags:
- spring-security
- userdetailsservice
- userdetailsmanager
- userdetails
- inmemoryuserdetailsmanager
- delegatingpasswordencoder
- compromisedpasswordchecker
- phase-4
---

# Section 3: In-Memory Identity Management & Contracts

## 1. Identity Abstractions: `UserDetailsService` vs `UserDetailsManager`

In Spring Security, the retrieval and lifecycle management of user credentials are decoupled through two core interfaces:

```mermaid
classDiagram
    class UserDetailsService {
        <<interface>>
        +loadUserByUsername(String username) UserDetails
    }

    class UserDetailsManager {
        <<interface>>
        +createUser(UserDetails user) void
        +updateUser(UserDetails user) void
        +deleteUser(String username) void
        +changePassword(String oldPassword, String newPassword) void
        +userExists(String username) boolean
    }

    class InMemoryUserDetailsManager {
        -Map~String, MutableUserDetails~ users
        +loadUserByUsername(String username) UserDetails
        +createUser(UserDetails user) void
        +updateUser(UserDetails user) void
        +deleteUser(String username) void
        +changePassword(String oldPassword, String newPassword) void
        +userExists(String username) boolean
    }

    class JdbcUserDetailsManager {
        -DataSource dataSource
        +loadUserByUsername(String username) UserDetails
        +createUser(UserDetails user) void
        +updateUser(UserDetails user) void
        +deleteUser(String username) void
    }

    UserDetailsService <|-- UserDetailsManager : extends
    UserDetailsManager <|.. InMemoryUserDetailsManager : implements
    UserDetailsManager <|.. JdbcUserDetailsManager : implements
```

### The Architectural Distinction

| Interface | Method Signatures | Architectural Responsibility |
| :--- | :--- | :--- |
| **`UserDetailsService`** | `loadUserByUsername(String username)` | **Read-Only Query Contract.** Responsible solely for locating a user identity by username and constructing a `UserDetails` representation for authentication. |
| **`UserDetailsManager`** | `createUser()`, `updateUser()`, `deleteUser()`, `changePassword()`, `userExists()` | **Full Identity Lifecycle (CRUD).** Extends `UserDetailsService` with write capabilities for managing user records in an underlying store. |

!!! tip "Design Principle: Interface Segregation"
    When building custom security architectures where user registration or profile updates are handled by domain services (e.g., standard Spring Data JPA services), your custom security bean only needs to implement **`UserDetailsService`**. Avoid implementing `UserDetailsManager` unless you specifically intend to delegate administrative user mutation tasks to Spring Security's internal workflows.

---

## 2. The `UserDetails` Contract & Account State Flags

The `UserDetails` interface is the canonical representation of an application user in Spring Security. It encapsulates credentials, authorities, and critical security flags:

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

```mermaid
flowchart TD
    UD["UserDetails Object"]
    
    subgraph Identity["Identity & Credentials"]
        U["getUsername() -> 'admin'"]
        P["getPassword() -> '{bcrypt}$2a$12$...'"]
        A["getAuthorities() -> [ROLE_ADMIN, VIEW_BALANCE]"]
    end

    subgraph Flags["Account State Invariants"]
        F1["isEnabled() -> true (Account is active)"]
        F2["isAccountNonLocked() -> true (Not locked by fraud detection)"]
        F3["isAccountNonExpired() -> true (Subscription is valid)"]
        F4["isCredentialsNonExpired() -> true (Password not older than 90 days)"]
    end

    UD --- Identity
    UD --- Flags
```

### Account Invariant Evaluation
During the authentication dispatch in `DaoAuthenticationProvider`:

1. If `isEnabled()` returns `false`, a `DisabledException` is thrown.
2. If `isAccountNonLocked()` returns `false`, a `LockedException` is thrown.
3. If `isAccountNonExpired()` returns `false`, an `AccountExpiredException` is thrown.
4. If `isCredentialsNonExpired()` returns `false`, a `CredentialsExpiredException` is thrown.

All four exceptions inherit from `AccountStatusException`, preventing access before password verification even takes place.

---

## 3. Provisioning Users via `InMemoryUserDetailsManager`

In local development, prototyping, and automated unit testing, managing users in memory avoids database dependencies. Spring provides `InMemoryUserDetailsManager`:

```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails user = User.withUsername("user")
            .password("{noop}EazyBytes@12345")
            .authorities("read")
            .build();

    UserDetails admin = User.withUsername("admin")
            .password("{bcrypt}$2a$12$88.f6upbBvy0okEa7OfHFuorV29qeK.sVbB9VQ6J6dWM1bW6Qef8m")
            .authorities("admin")
            .build();

    return new InMemoryUserDetailsManager(user, admin);
}
```

### The `User.withUsername()` Builder
Spring Security provides the `User.UserBuilder` fluent API to construct immutable `UserDetails` instances. Notice the password prefixes:

- `{noop}EazyBytes@12345`: Informs the delegating encoder that this password is plain text (no operation).
- `{bcrypt}$2a$12$...`: Informs the delegating encoder that this hash is computed using BCrypt with work factor 12.

---

## 4. The Delegating Password Encoder

Why do passwords require prefix identifiers like `{noop}` or `{bcrypt}`?

If you register a plain text password without a prefix and without declaring a `PasswordEncoder` bean:
```java
UserDetails user = User.withUsername("user").password("12345").build();
```
Upon login, Spring Security crashes with:
```text
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

This fail-safe behavior was introduced in Spring Security 5 to eradicate plain text passwords in enterprise applications.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

`PasswordEncoderFactories.createDelegatingPasswordEncoder()` constructs a `DelegatingPasswordEncoder` whose default encoding algorithm is **BCrypt**:

```mermaid
flowchart LR
    RAW["Raw Password from Login<br/>'EazyBytes@12345'"]
    DELEGATE["DelegatingPasswordEncoder"]
    MATCH{"matches()?"}

    subgraph Registry["Algorithm Map"]
        direction TB
        BCRYPT["{bcrypt} -> BCryptPasswordEncoder (Default)"]
        ARGON2["{argon2} -> Argon2PasswordEncoder"]
        PBKDF2["{pbkdf2} -> Pbkdf2PasswordEncoder"]
        NOOP["{noop} -> NoOpPasswordEncoder (Insecure)"]
    end

    RAW --> DELEGATE

    DELEGATE -->|"{bcrypt}"| BCRYPT
    DELEGATE -->|"{argon2}"| ARGON2
    DELEGATE -->|"{pbkdf2}"| PBKDF2
    DELEGATE -->|"{noop}"| NOOP

    BCRYPT --> MATCH
    ARGON2 --> MATCH
    PBKDF2 --> MATCH
    NOOP --> MATCH

```

### Zero-Downtime Cryptographic Migration
The delegating encoder pattern allows enterprise systems to migrate historical passwords seamlessly. If an application historically used `{pbkdf2}` and transitions to `{bcrypt}`:

1. Existing `{pbkdf2}` hashes continue to authenticate successfully.
2. When an existing user logs in, the application can detect that the algorithm is outdated via `passwordEncoder.upgradeEncoding(storedHash)` and re-encode the raw password using `{bcrypt}` without forcing an outage or mass password reset.

---

## 5. Compromised Password Screening (Spring Security 6.3+)

A major security enhancement introduced in **Spring Security 6.3** is the **`CompromisedPasswordChecker`**:

```java
@Bean
public CompromisedPasswordChecker compromisedPasswordChecker() {
    return new HaveIBeenPwnedRestApiPasswordChecker();
}
```

### How `HaveIBeenPwnedRestApiPasswordChecker` Works
Instead of sending user passwords over the internet, this component implements the **$k$-Anonymity Mathematical Model**:

```mermaid
sequenceDiagram
    autonumber
    actor User as Registering Customer
    participant App as Spring Boot Service
    participant Checker as HaveIBeenPwnedRestApiPasswordChecker
    participant HIBP as HaveIBeenPwned Cloud API

    User->>App: Submits password "Password123!"
    App->>Checker: check("Password123!")
    Checker->>Checker: Compute SHA-1("Password123!")<br/>-> 7C4A8D09CA3762AF61E59520943DC26494F8941B
    Checker->>Checker: Extract prefix: 7C4A8 (First 5 hex characters)
    Checker->>HIBP: GET https://api.pwnedpasswords.com/range/7C4A8
    HIBP-->>Checker: Returns list of matching hash suffixes with breach counts
    Checker->>Checker: Look for remaining suffix D09CA37... in returned list
    
    alt Suffix Found in Breach Database
        Checker-->>App: CompromisedPasswordDecision(isCompromised=true)
        App-->>User: 400 Bad Request ("Password appears in data breaches! Choose a secure password.")
    else Suffix Not Found
        Checker-->>App: CompromisedPasswordDecision(isCompromised=false)
        App->>App: Hash password with BCrypt and save to DB
    end
```

!!! important "Privacy Guarantee of k-Anonymity"
    The full password or full hash is **never transmitted** to the external API. Only the first 5 hexadecimal characters of the SHA-1 hash are sent. The external service returns hundreds of possible suffix hashes, and the comparison is performed entirely inside your application memory.
