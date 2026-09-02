---
tags: [jakarta-ee, jpa, entity-mapping, relationships, phase-3, week-2]
---

# Day 08 — JPA 3.1 Entity Mapping & Relationship Boundaries

> **Daily Time Investment:** 2.5 hours | **Week:** 2 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, all 4 cardinalities, `@Embeddable`, `@ElementCollection`, cascade lifecycle, `orphanRemoval`, Lazy vs Eager fetching |
| Book Reading | 30 min | Pro Persistence Ch4 (ORM) + Ch5 (Collection Mapping) + EE8 AppDev Ch3 |
| Hands-On Lab | 75 min | Multi-entity cluster schema (ServerRack, ClusterNode, TelemetryRecord, SecurityGroup); verify `LazyInitializationException` at context boundary |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch4 & Ch5 — Object-Relational Mapping & Collections**

    ---

    `@Entity`, `@Table`, `@Id`, `@GeneratedValue` strategies, `@Column`, `@Embeddable`, `@ElementCollection`, all 4 relationship cardinalities, `@JoinColumn`, `@JoinTable`, `CascadeType`, `FetchType`, `orphanRemoval`, `@OrderBy`.

    [:octicons-arrow-right-24: Read Book Summary](book-pro-persistence-ch4-ch5.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JPA Entity Mappings & Boundaries**

    ---

    Build the `ClusterNode` domain: `ServerRack → ClusterNode → TelemetryRecord` (1:N), `ClusterNode ↔ SecurityGroup` (N:M), `ClusterNode ↔ NodeDiagnostics` (1:1), `HardwareSpec` (`@Embeddable`), `Set<String> tags` (`@ElementCollection`). Verify cascade persist, orphan removal, and `LazyInitializationException`.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **`EntityManager` & `EntityManagerFactory`** — the central JPA API; `EntityManagerFactory` is a thread-safe heavyweight factory created once per persistence unit; `EntityManager` is a lightweight, non-thread-safe unit-of-work instance
    - **`persistence.xml`** — the JPA configuration descriptor placed in `src/main/resources/META-INF/`; declares the persistence unit name, JPA provider class (Hibernate), JDBC connection properties, and entity scanning
    - **Hibernate ORM 6.4** — the JPA 3.1 reference implementation used in Jakarta EE 10; generates SQL DDL automatically when `hibernate.hbm2ddl.auto=create-drop`
    - **H2 In-Memory Database** — a pure-Java embedded database; ideal for lab environments since zero installation is required; data vanishes when JVM exits
    - **`GenerationType.IDENTITY`** — relies on `AUTO_INCREMENT` in MySQL/H2; IDs are only available after the SQL INSERT fires (not before `persist()` returns)
    - **Persistence Context** — an in-memory cache of managed entity instances; not to be confused with Spring's `ApplicationContext` — this is purely a JPA concept scoped to a transaction or stateful bean
    - **`mappedBy`** — marks the **inverse (non-owning) side** of a bidirectional relationship; only the owning side has the physical `@JoinColumn` in the database; failing to set `mappedBy` creates duplicate join tables

---

[:octicons-arrow-left-24: Back to Week 2](../index.md)
