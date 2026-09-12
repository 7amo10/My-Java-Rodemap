---
tags: [jakarta-ee, jpa, n-plus-one, entity-graph, batch-size, performance, phase-3, week-3]
---

# Day 18 — JPA Performance Tuning & Batch Fetching

> **Daily Time Investment:** 2.5 hours | **Week:** 3 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | N+1 diagnosis, JPQL `JOIN FETCH`, `@BatchSize` batch subselects, JPA 3.1 `EntityGraph` (fetchgraph vs loadgraph), scalar DTO projections |
| Book Reading | 30 min | Pro Persistence Ch11 (Native SQL, Stored Procedures, Entity Graphs) + Ch12 (L2 Caching) |
| Hands-On Lab | 75 min | Profile 20-node N+1 baseline (21 queries) → fix with `JOIN FETCH` (1 query) → `@BatchSize` (3 queries) → `EntityGraph` (1 query) → DTO projection (0 entity overhead) |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch11 — Advanced Queries**

    ---

    Native SQL queries (`createNativeQuery()`), `@NamedNativeQuery`, `@SqlResultSetMapping` (`@EntityResult`, `@FieldResult`, `@ColumnResult`, `@ConstructorResult`), Stored Procedures (`StoredProcedureQuery`, `@NamedStoredProcedureQuery`), Entity Graphs (`@NamedEntityGraph`, `@NamedAttributeNode`, Fetch Graph vs Load Graph hints).

    [:octicons-arrow-right-24: Read Pro Persistence Ch11](book-pro-persistence-ch11.md)

-   :material-database-check:{ .lg .middle } **Pro Persistence Ch12 — Advanced Topics (Caching)**

    ---

    Second-Level Cache (L2): `shared-cache-mode` in `persistence.xml` (ALL/NONE/ENABLE_SELECTIVE/DISABLE_SELECTIVE), `@Cacheable`, `CacheRetrieveMode` / `CacheStoreMode` runtime hints, `Cache` API (`evict`, `evictAll`, `contains`).

    [:octicons-arrow-right-24: Read Pro Persistence Ch12 Caching](book-pro-persistence-ch12-caching.md)

-   :material-flask:{ .lg .middle } **Lab Guide — N+1 Diagnosis & Fetching Strategies**

    ---

    Hibernate `StatisticsService` counts queries, `JOIN FETCH` eliminates 95% of queries, `@BatchSize(10)` halves remaining, `EntityGraph` (dynamic fetch plan without JPQL change), scalar `record` projections with zero entity lifecycle overhead.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **N+1 Select Problem** — loading N parent entities with `FetchType.LAZY` and then accessing a child collection fires N additional SELECT queries; for N=20 nodes, that's 21 SQL statements instead of 1
    - **Hibernate `StatisticsService`** — JPA provider API that counts executed SQL statements per transaction; used to prove N+1 was eliminated; enabled via `hibernate.generate_statistics=true`
    - **`@BatchSize(size=10)`** — Hibernate-specific annotation on a collection; instead of N individual SELECTs, uses `WHERE id IN (?, ?, ..., ?)` batch queries; for N=20, becomes `ceil(20/10) = 2` batch queries instead of 20
    - **`EntityGraph`** — JPA 3.1 standard; declares which associations to fetch eagerly at **query time** without modifying entity `FetchType` annotations; two hint types: `fetchgraph` (ONLY listed attributes are EAGER) vs `loadgraph` (listed are EAGER, unlisted inherit their static mapping)
    - **`@NamedNativeQuery`** — pre-defined native SQL query stored on an entity; executed with `em.createNamedQuery()` same as JPQL but runs raw SQL against the database
    - **`@SqlResultSetMapping`** — maps raw SQL result columns to entity classes or constructor projections when column names don't match JPA defaults
    - **`StoredProcedureQuery`** — JPA API for calling database stored procedures; supports `IN`, `OUT`, `INOUT`, and `REF_CURSOR` parameter modes

---

[:octicons-arrow-left-24: Back to Week 3](../index.md)
