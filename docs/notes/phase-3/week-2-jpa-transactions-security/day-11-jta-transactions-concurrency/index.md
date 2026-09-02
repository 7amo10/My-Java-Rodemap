---
tags: [jakarta-ee, jta, transactions, optimistic-locking, phase-3, week-2]
---

# Day 11 — JTA Declarative Transactions & Concurrency Control

> **Daily Time Investment:** 2.0 hours | **Week:** 2 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `@Transactional` propagation attributes, rollback rules, `@Version` optimistic locking, pessimistic locking |
| Book Reading | 30 min | Pro Persistence Ch12 (Locking & Concurrency) + EE Patterns Ch6 (Transactions) |
| Hands-On Lab | 45 min | Simulate concurrent updates; verify `OptimisticLockException`; prove `REQUIRES_NEW` audit isolation |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch12 — Advanced Topics: Locking & Concurrency**

    ---

    `@Version` optimistic locking, `OptimisticLockException`, `LockModeType` enum (`OPTIMISTIC`, `OPTIMISTIC_FORCE_INCREMENT`, `PESSIMISTIC_WRITE`, `PESSIMISTIC_READ`), lock timeouts, Second-Level Cache (`@Cacheable`, `shared-cache-mode`), lifecycle callbacks, Bean Validation integration.

    [:octicons-arrow-right-24: Read Book Summary](book-pro-persistence-ch12.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JTA Transactions & Concurrency**

    ---

    `@Transactional(TxType.REQUIRED)` ACID commit, unchecked exception auto-rollback, `REQUIRES_NEW` autonomous audit log, `rollbackOn` for checked exceptions, `@Version` collision detection (`OptimisticLockException`).

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **JTA (Jakarta Transactions)** — a distributed transaction protocol; in Jakarta EE, `@Transactional` is CDI-intercepted; the container manages `begin()`/`commit()`/`rollback()` automatically — you never call these manually
    - **ACID properties** — Atomicity (all-or-nothing), Consistency (invariants maintained), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes)
    - **Transaction propagation** — `REQUIRED` joins existing transaction or starts new; `REQUIRES_NEW` always suspends the outer transaction and starts a completely independent one
    - **`@Version`** — a numeric column (`Long version`) that JPA increments automatically on every `UPDATE`; concurrent updates use `WHERE id=? AND version=?` — the second writer sees the version was already incremented and throws `OptimisticLockException`
    - **Optimistic vs Pessimistic Locking** — Optimistic: no DB lock acquired, conflicts detected only at commit (fast, suitable when conflicts are rare); Pessimistic: acquires `SELECT FOR UPDATE` immediately (slower, prevents all concurrent modifications)
    - **`TxType`** — the CDI equivalent of EJB's `TransactionAttributeType`; use `jakarta.transaction.Transactional.TxType` enum values

---

[:octicons-arrow-left-24: Back to Week 2](../index.md)
