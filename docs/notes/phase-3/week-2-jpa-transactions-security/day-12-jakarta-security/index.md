---
tags: [jakarta-ee, security, pbkdf2, identity-store, rbac, phase-3, week-2]
---

# Day 12 — Jakarta Security 3.0 & Password Hashing

> **Daily Time Investment:** 2.5 hours | **Week:** 2 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `Pbkdf2PasswordHash`, custom `IdentityStore`, `CredentialValidationResult`, `@RolesAllowed`, `SecurityContext` |
| Book Reading | 30 min | EE8 AppDev Ch9 — Securing Java EE Applications |
| Hands-On Lab | 75 min | Custom `IdentityStore` with PBKDF2 hashing; prove RBAC matrix across Admin/Operator/Viewer/Anonymous roles |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch9 — Securing Java EE Applications**

    ---

    Jakarta Security 3.0 overview, `HttpAuthenticationMechanism` (`@BasicAuthenticationMechanismDefinition`, `@FormAuthenticationMechanismDefinition`), `IdentityStore` interface, `@DatabaseIdentityStoreDefinition`, `@LdapIdentityStoreDefinition`, custom `IdentityStore`, `Pbkdf2PasswordHash`, `SecurityContext`, `@DeclareRoles`, `@RolesAllowed`, `@PermitAll`, `@DenyAll`, `CallerPrincipal`.

    [:octicons-arrow-right-24: Read Book Summary](book-ee8appdev-ch9.md)

-   :material-flask:{ .lg .middle } **Lab Guide — Jakarta Security & PBKDF2**

    ---

    Implement `Pbkdf2PasswordHash` (2048 iterations, dynamic salt), custom `IdentityStore` validating `UsernamePasswordCredential`, three-state `CredentialValidationResult` (`VALID`/`INVALID`/`NOT_VALIDATED`), RBAC enforcement matrix.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **Jakarta Security 3.0** — standardized security API across Jakarta EE containers; replaces container-specific JAAS and servlet security configurations
    - **`Pbkdf2PasswordHash`** — Password-Based Key Derivation Function 2; applies a pseudorandom function (HMAC-SHA256) iteratively with a unique random salt; the output is different **every time** even for the same password — rainbow table attacks are defeated
    - **Key stretching** — repeating the hash function N times (2048+) makes brute-force attacks exponentially slower without noticeable overhead for legitimate logins
    - **`IdentityStore`** — Jakarta Security's pluggable credential validation contract; can chain multiple stores (DB, LDAP, in-memory) through a priority system
    - **`CredentialValidationResult`** — three possible states: `VALID` (caller + roles returned), `INVALID` (wrong password), `NOT_VALIDATED` (this store doesn't handle this credential type → try next store)
    - **`SecurityContext`** (CDI-injectable) — programmatic access to the authenticated principal and role checks; complements annotation-based `@RolesAllowed`
    - **`@DeclareRoles`** — must be declared at the application/class level; these are the valid role names the container recognizes for authorization decisions

---

[:octicons-arrow-left-24: Back to Week 2](../index.md)
