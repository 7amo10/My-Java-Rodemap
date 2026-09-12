---
tags: [jakarta-ee, bce, architecture, boundary-control-entity, phase-3, week-3]
---

# Day 15 — Adam Bien's BCE Architecture Pattern

> **Daily Time Investment:** 2.5 hours | **Week:** 3 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | Boundary (JAX-RS facade), Control (business logic), Entity (JPA as DTO), anti-patterns eliminated |
| Book Reading | 30 min | EE Patterns Ch3 — Rethinking the Business Tier |
| Hands-On Lab | 75 min | Refactor monolithic service into strict BCE package structure; verify zero DTO mappings and zero redundant interfaces |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE Patterns Ch3 — BCE Pattern & Rethinking the Business Tier**

    ---

    Adam Bien's BCE pattern: Service Façade (Boundary), Service (Control), Persistent Domain Object (Entity as DTO). Package-by-feature, eliminating interface-per-class, eliminating DTO explosion, Gateway pattern, Fluid Logic, retired J2EE patterns.

    [:octicons-arrow-right-24: Read Book Summary](book-adam-bien-bce-ch3.md)

-   :material-flask:{ .lg .middle } **Lab Guide — BCE Architecture Refactoring**

    ---

    Build `NodeResource` (Boundary), `HealthCalculator` + `NodeLifecycleController` (Controls), `ClusterNode` + `NodeSummary` record (Entities). Direct entity serialization via JSON-B. JPQL constructor expressions for lightweight projections.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **Package-by-Feature vs Package-by-Layer** — traditional architectures use horizontal layers (`service/`, `dao/`, `dto/`); BCE uses vertical feature packages (`node/boundary/`, `node/control/`, `node/entity/`) — all code for one domain feature lives together
    - **Entity as DTO (Entity as Transfer Object)** — instead of writing a parallel `NodeDTO` for every `ClusterNode` entity, the JPA `@Entity` is serialized directly by JSON-B; `@JsonbTransient` protects internal state (e.g., `passwordHash`) from serialization
    - **No-Interface View (CDI 4.0)** — CDI proxies concrete `@ApplicationScoped` or `@Stateless` beans directly without requiring an interface; the interface-per-class pattern (`INodeService` + `NodeServiceImpl`) is pure overhead
    - **Anemic Domain Model Anti-Pattern** — entities with only getters/setters and no behavior; BCE fixes this by placing domain methods (e.g., `canTransitionTo(NodeStatus)`, `calculateHealthScore()`) directly on the entity class
    - **`@JsonbTransient`** — Jakarta JSON Binding annotation that excludes a field from serialization output; equivalent to `@JsonIgnore` in Jackson
    - **`NodeSummary record`** — Java 21 `record` used as a lightweight read-only DTO created via JPQL `SELECT NEW` constructor expressions; immutable, no entity lifecycle overhead

---

[:octicons-arrow-left-24: Back to Week 3](../index.md)
