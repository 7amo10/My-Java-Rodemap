---
tags: [jakarta-ee, jpa, persistence-context, entity-lifecycle, phase-3, week-2]
---

# Day 09 — Persistence Context, EntityManager & Lifecycle Events

> **Daily Time Investment:** 2.5 hours | **Week:** 2 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | 4 entity states, First-Level Cache, dirty checking, `flush()`/`refresh()`/`merge()`, lifecycle callbacks |
| Book Reading | 30 min | Pro Persistence Ch6 (Entity Manager) |
| Hands-On Lab | 75 min | `AuditListener` via `@EntityListeners`; prove L1 identity map and auto dirty checking |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch6 — EntityManager & Persistence Context**

    ---

    Transaction-scoped vs extended persistence context, `persist()`, `merge()`, `remove()`, `find()` vs `getReference()`, `flush()`, `refresh()`, `clear()`, `detach()`, First-Level Cache identity map, cascading operations, web architectural patterns.

    [:octicons-arrow-right-24: Read Book Summary](book-pro-persistence-ch6.md)

-   :material-flask:{ .lg .middle } **Lab Guide — Persistence Context & Lifecycle**

    ---

    Prove entity state transitions, L1 identity map (same Java reference for repeated `find()`), auto dirty checking (UPDATE without explicit `merge()`), `AuditListener` logging all `@PrePersist`/`@PostUpdate` events, `@PostLoad` computing transient fields.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **Transaction-Scoped vs Extended persistence context** — transaction-scoped (default) lives only within a JTA transaction; extended lives across multiple transactions (tied to `@Stateful` EJBs)
    - **First-Level Cache (Identity Map)** — JPA guarantees that within one persistence context, `em.find(T.class, id)` for the same `id` always returns the exact same Java object instance (`ref1 == ref2`) — no second SQL query
    - **Automatic Dirty Checking** — Hibernate snapshots entity state at load time; at `flush()` / commit, it compares current state to snapshot and issues `UPDATE` automatically — no explicit save call needed
    - **`em.merge(detached)`** — does NOT make the passed argument managed; it copies state into an existing (or new) managed entity and returns the **managed copy** — always use the return value
    - **`em.getReference()`** — returns a **Hibernate proxy** without hitting the database; use only for foreign-key associations; throws `EntityNotFoundException` lazily if entity doesn't exist
    - **`@EntityListeners`** — an external auditing class; method signature must be `void onEvent(EntityType entity)` — different from in-entity callbacks which take no parameters

---

[:octicons-arrow-left-24: Back to Week 2](../index.md)
