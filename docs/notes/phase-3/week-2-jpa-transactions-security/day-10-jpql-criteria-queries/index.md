---
tags: [jakarta-ee, jpa, jpql, criteria-api, queries, phase-3, week-2]
---

# Day 10 — Dynamic JPQL Queries & Criteria API

> **Daily Time Investment:** 2.5 hours | **Week:** 2 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | JPQL join fetches, `SELECT NEW` DTO projections, `GROUP BY`/`HAVING`, `CriteriaBuilder` type-safe construction |
| Book Reading | 30 min | Pro Persistence Ch7 (Using Queries) + Ch8 (JPQL Language) + Ch9 (Criteria API) |
| Hands-On Lab | 75 min | Dynamic multi-field search service with Criteria API; verify N+1 elimination with `JOIN FETCH` |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch7 — Using Queries**

    ---

    `createQuery()`, `TypedQuery<T>`, named/positional parameters, `@NamedQuery` (pre-compiled), `getSingleResult()` vs `getResultList()` vs `getResultStream()`, `setFirstResult()`/`setMaxResults()` pagination, `executeUpdate()` bulk ops, native SQL, query hints.

    [:octicons-arrow-right-24: Read Ch7 — Using Queries](book-pro-persistence-ch7.md)

-   :material-code-braces:{ .lg .middle } **Pro Persistence Ch8 — Query Language (JPQL)**

    ---

    JPQL vs SQL, `SELECT NEW` constructor expressions, `INNER JOIN` / `LEFT OUTER JOIN` / `JOIN FETCH`, `BETWEEN` / `LIKE` / `IS EMPTY` / `EXISTS`, `GROUP BY` + `HAVING`, correlated subqueries, polymorphism, `TREAT` downcasting, bulk `UPDATE`/`DELETE`.

    [:octicons-arrow-right-24: Read Ch8 — JPQL Language](book-pro-persistence-ch8.md)

-   :material-cog:{ .lg .middle } **Pro Persistence Ch9 — Criteria API**

    ---

    `CriteriaBuilder`, `CriteriaQuery<T>`, `Root<T>`, `Predicate` composition, dynamic multi-field search pattern, `Join` / fetch joins, `Tuple` projections, `cb.construct()` DTO, `ORDER BY`/`GROUP BY`/`HAVING`, correlated subqueries (`correlate()`), `CriteriaUpdate`/`CriteriaDelete`, canonical metamodel.

    [:octicons-arrow-right-24: Read Ch9 — Criteria API](book-pro-persistence-ch9.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JPQL & Criteria API**

    ---

    `@NamedQuery` static queries, `NodeMetricSummaryDto` projection via `SELECT NEW`, N+1 elimination with `JOIN FETCH`, `GROUP BY + HAVING` overload detection, `CriteriaBuilder` multi-predicate paginated search.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **JPQL vs SQL** — JPQL operates on the **object model** (entity class names and field names), not table/column names; `FROM ClusterNode c` not `FROM cluster_node c`
    - **N+1 Select Problem** — loading N parent entities each triggering a separate SELECT for their lazy children = N+1 total queries; `JOIN FETCH` collapses this into 1 query
    - **`TypedQuery<T>`** — type-safe version of `Query`; avoids unchecked casts on `getResultList()`
    - **`SELECT NEW com.pulse.dto.NodeMetricSummaryDto(...)`** — constructor expression; bypasses entity hydration and First-Level Cache entirely; must match a public constructor
    - **`CriteriaBuilder`** — the factory for type-safe programmatic queries; obtained from `em.getCriteriaBuilder()`
    - **`Root<T>`** — represents the entity being queried (`FROM` clause); similar to `FROM Employee e` in JPQL
    - **`Predicate`** — the type-safe equivalent of a JPQL `WHERE` condition; can be composed with `cb.and()` / `cb.or()`

---

[:octicons-arrow-left-24: Back to Week 2](../index.md)
