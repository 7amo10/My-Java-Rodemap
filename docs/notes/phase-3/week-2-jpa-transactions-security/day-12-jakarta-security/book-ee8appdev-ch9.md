---
tags: [jakarta-ee, security, identity-store, pbkdf2, rbac, authentication, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 9: Securing Java EE Applications

> **Book:** Java EE 8 Application Development — David R. Heffelfinger (Packt)  
> **Chapter:** 9 — Securing Java EE Applications  
> **Namespace note:** Use `jakarta.*` packages — not `javax.*`

---

## :material-information: Jakarta Security 3.0 Overview

Jakarta Security provides a **unified, portable security API** across all Jakarta EE containers. Before Jakarta Security, each container (WildFly, Payara, GlassFish) had its own proprietary mechanism. Jakarta Security standardizes:

1. **`IdentityStore`** — WHERE and HOW credentials are validated (DB, LDAP, custom)
2. **`HttpAuthenticationMechanism`** — HOW credentials are collected from HTTP requests
3. **`SecurityContext`** — Programmatic access to the authenticated caller

---

## :material-store: Identity Stores

### Database Identity Store — `@DatabaseIdentityStoreDefinition`

Declarative configuration — no code required:

```java
@DatabaseIdentityStoreDefinition(
    dataSourceLookup = "jdbc/userAuth",
    callerQuery = "SELECT password FROM users WHERE name = ?",
    groupsQuery = "SELECT g.GROUP_NAME FROM USER_GROUPS ug " +
                  "JOIN USERS u ON ug.USER_ID = u.user_id " +
                  "JOIN GROUPS g ON g.GROUP_ID = ug.GROUP_ID " +
                  "WHERE u.USERNAME = ?",
    hashAlgorithm = Pbkdf2PasswordHash.class,
    hashAlgorithmParameters = {
        "Pbkdf2PasswordHash.Iterations=3072",
        "Pbkdf2PasswordHash.Algorithm=PBKDF2WithHmacSHA512",
        "Pbkdf2PasswordHash.SaltSizeBytes=64"
    }
)
@ApplicationScoped
public class ApplicationConfig {}
```

### LDAP Identity Store — `@LdapIdentityStoreDefinition`

```java
@LdapIdentityStoreDefinition(
    url = "ldap://ldap.enterprise.com:389/",
    callerBaseDn = "ou=users,dc=enterprise,dc=com",
    groupSearchBase = "ou=groups,dc=enterprise,dc=com",
    groupSearchFilter = "(&(objectClass=groupOfNames)(member={0}))"
)
@ApplicationScoped
public class LdapConfig {}
```

### Custom `IdentityStore` Implementation

Full control over credential validation — implement the `IdentityStore` interface:

```java
@ApplicationScoped
public class CustomIdentityStore implements IdentityStore {

    @Inject Pbkdf2PasswordHash pbkdf2;
    @Inject UserRepository userRepo;

    @PostConstruct
    public void init() {
        pbkdf2.initialize(Map.of(
            "Pbkdf2PasswordHash.Iterations",   "2048",
            "Pbkdf2PasswordHash.Algorithm",    "PBKDF2WithHmacSHA256",
            "Pbkdf2PasswordHash.SaltSizeBytes", "32"
        ));
    }

    @Override
    public CredentialValidationResult validate(Credential credential) {
        if (!(credential instanceof UsernamePasswordCredential upc)) {
            return CredentialValidationResult.NOT_VALIDATED_RESULT;
        }

        return userRepo.findByUsername(upc.getCaller())
            .filter(UserAccount::isEnabled)
            .filter(u -> pbkdf2.verify(upc.getPasswordAsString().toCharArray(), u.getPasswordHash()))
            .map(u -> new CredentialValidationResult(new CallerPrincipal(u.getUsername()), u.getRoles()))
            .orElse(CredentialValidationResult.INVALID_RESULT);
    }
}
```

**`CredentialValidationResult` States:**

| State | Factory | Meaning |
|-------|---------|---------|
| `VALID` | `new CredentialValidationResult(principal, roles)` | Authentication success |
| `INVALID` | `CredentialValidationResult.INVALID_RESULT` | Bad credentials or locked |
| `NOT_VALIDATED` | `CredentialValidationResult.NOT_VALIDATED_RESULT` | Skip — try next store |

---

## :material-web: HTTP Authentication Mechanisms

### Basic Authentication

Browser shows native credentials dialog:

```java
@BasicAuthenticationMechanismDefinition(realmName = "ClusterAPI Realm")
@ApplicationScoped
public class SecurityConfig {}
```

### Form Authentication (Container-Managed HTML Form)

```java
@FormAuthenticationMechanismDefinition(
    loginToContinue = @LoginToContinue(
        loginPage  = "/login.html",
        errorPage  = "/login-error.html"
    )
)
@ApplicationScoped
public class FormSecurityConfig {}
```

Required HTML form structure:

```html
<form method="POST" action="j_security_check">
    <input type="text"     name="j_username" placeholder="Username">
    <input type="password" name="j_password" placeholder="Password">
    <button type="submit">Login</button>
</form>
```

### Custom Form Authentication (JSF / Programmatic)

```java
@CustomFormAuthenticationMechanismDefinition(
    loginToContinue = @LoginToContinue(
        loginPage  = "/faces/login.xhtml",
        errorPage  = ""
    )
)
@ApplicationScoped
public class CustomSecurityConfig {}
```

Programmatic login in a CDI backing bean:

```java
@Named @RequestScoped
public class LoginBean {

    @Inject private SecurityContext securityContext;

    private String username;
    private String password;

    public String login() {
        UsernamePasswordCredential credential =
            new UsernamePasswordCredential(username, new Password(password));

        AuthenticationParameters params =
            AuthenticationParameters.withParams().credential(credential);

        AuthenticationStatus status = securityContext.authenticate(
            FacesContext.getCurrentInstance().getExternalContext().getRequest(),
            FacesContext.getCurrentInstance().getExternalContext().getResponse(),
            params
        );

        if (status == AuthenticationStatus.SEND_CONTINUE) {
            return "/dashboard?faces-redirect=true";   // success
        } else {
            return "/login?error=true";   // failure
        }
    }
}
```

---

## :material-shield-account: Role-Based Access Control (RBAC)

### Servlet-Level RBAC

```java
@WebServlet("/admin/*")
@DeclareRoles({"ADMIN", "OPERATOR"})
@ServletSecurity(
    @HttpConstraint(rolesAllowed = {"ADMIN"})   // default: all methods require ADMIN
)
public class AdminServlet extends HttpServlet {

    // Override for specific HTTP methods:
    @HttpMethodConstraint(value = "GET", rolesAllowed = {"ADMIN", "OPERATOR"})
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) { ... }

    @HttpMethodConstraint(value = "DELETE", rolesAllowed = {"ADMIN"})
    protected void doDelete(HttpServletRequest req, HttpServletResponse resp) { ... }
}
```

### CDI Bean Method RBAC

```java
@ApplicationScoped
@DeclareRoles({"ADMIN", "OPERATOR", "VIEWER"})
public class NodeManagementService {

    @Inject SecurityContext securityContext;

    @RolesAllowed("ADMIN")
    public void deleteNode(Long id) { ... }

    @RolesAllowed({"ADMIN", "OPERATOR"})
    public void updateNode(ClusterNode node) { ... }

    @PermitAll
    public List<ClusterNode> listNodes() { ... }

    @DenyAll
    public void deprecatedBulkOp() { ... }
}
```

### Programmatic `SecurityContext` API

```java
@Inject SecurityContext securityContext;

// Get authenticated caller:
Principal caller = securityContext.getCallerPrincipal();
String username = caller.getName();

// Check role:
boolean isAdmin = securityContext.isCallerInRole("ADMIN");

// Get all principals associated with the caller:
Set<Principal> principals = securityContext.getPrincipalsByType(CallerPrincipal.class);
```

---

## :material-key-chain: `Pbkdf2PasswordHash` — Detailed API

### Algorithm Options

| Algorithm | Security Level |
|---------|--------------|
| `PBKDF2WithHmacSHA256` | Good — 256-bit digest |
| `PBKDF2WithHmacSHA512` | Better — 512-bit digest (preferred for new systems) |

### Configuration Parameters

| Parameter | Default | Recommended |
|-----------|---------|-------------|
| `Pbkdf2PasswordHash.Iterations` | `2048` | `310000`+ (OWASP 2023) |
| `Pbkdf2PasswordHash.Algorithm` | `PBKDF2WithHmacSHA256` | `PBKDF2WithHmacSHA512` |
| `Pbkdf2PasswordHash.SaltSizeBytes` | `16` | `32` |
| `Pbkdf2PasswordHash.KeySizeBytes` | `32` | `32` |

### Hash Storage Format

The `generate()` output is a self-describing string that includes all parameters needed for verification:

```
PBKDF2WithHmacSHA256:2048:base64EncodedSalt:base64EncodedHash
```

This means the stored hash is fully portable — you can change the algorithm/iterations for new passwords while existing hashes still verify correctly.

---

## :material-key: Key Takeaways — Chapter 9

1. **Jakarta Security 3.0** provides portable, container-agnostic security — no proprietary XML configuration
2. **`@DatabaseIdentityStoreDefinition`** = zero-code DB authentication; use for simple cases
3. **Custom `IdentityStore`** = full control; implement `validate(Credential)` returning three possible states
4. **`NOT_VALIDATED_RESULT`** (not `INVALID`) for unknown users/credentials — allows multi-store delegation to continue
5. **`Pbkdf2PasswordHash`** handles salt generation, key stretching, and constant-time comparison — never roll your own
6. **`@RolesAllowed`** operates at method level — annotate both service beans AND JAX-RS resources
7. **`SecurityContext`** is CDI-injectable — use for programmatic role checks in complex authorization logic

---

[:octicons-arrow-left-24: Back to Day 12 Index](index.md)
