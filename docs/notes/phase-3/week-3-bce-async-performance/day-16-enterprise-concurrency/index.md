---
tags: [jakarta-ee, concurrency, managed-executor, completable-future, phase-3, week-3]
---

# Day 16 — Enterprise Concurrency & Managed Threads

> **Daily Time Investment:** 2.0 hours | **Week:** 3 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | Why raw threads are forbidden, `ManagedExecutorService`, `ManagedThreadFactory`, context propagation, `CompletableFuture` parallel orchestration |
| Book Reading | 30 min | EE Patterns Ch3 (Async Facade section) + EE8 AppDev Ch13 (Servlet Async Processing) |
| Hands-On Lab | 45 min | Context-propagating managed executor; verify parallel fan-out is ~175ms vs ~340ms sequential |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch13 — Servlet Async Processing**

    ---

    `@WebServlet(asyncSupported=true)`, `AsyncContext`, `request.startAsync()`, `asyncContext.complete()`, async vs managed threads comparison. HTTP/2 Server Push. Servlet lifecycle scopes (request/session/application).

    [:octicons-arrow-right-24: Read Book Summary](book-ee8appdev-ch13-async.md)

-   :material-flask:{ .lg .middle } **Lab Guide — Enterprise Concurrency & Managed Threads**

    ---

    `ManagedExecutorService` with context propagation (Security Principal + Correlation ID), `CompletableFuture.allOf()` parallel fan-out for 4 nodes (~175ms vs ~340ms sequential), non-blocking timeout fallback, `.exceptionally()` error recovery, clean lifecycle shutdown.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **Why `new Thread()` is BANNED in enterprise containers** — raw threads bypass the container's classloader boundaries, lack security/transaction/CDI context, cannot be gracefully shut down during undeployment, and can crash the JVM with unbounded creation. The Jakarta EE specification explicitly forbids them
    - **`ManagedExecutorService`** — Jakarta Concurrency 3.0's container-managed thread pool; obtained via `@Resource ManagedExecutorService executor`; captures caller context (security principal, correlation ID) at submission time and restores it on the worker thread
    - **`ManagedThreadFactory`** — creates container-aware threads with structured naming; the factory enforces container lifecycle governance
    - **Context Propagation** — when you submit a `Runnable` or `Supplier` to `ManagedExecutorService`, the container snapshots the calling thread's `SecurityContext`, CDI context, and transaction state, then restores them on the worker thread before the task runs
    - **`CompletableFuture.supplyAsync(supplier, managedExecutor)`** — bridges Java's async API with the enterprise container's managed thread pool; NEVER use `CompletableFuture.supplyAsync(supplier)` without the executor argument in enterprise code — it uses the JVM's common `ForkJoinPool`, which has no container context
    - **`@Asynchronous`** — EJB annotation that makes a business method execute asynchronously; returns `Future<T>` or `AsyncResult<T>`; simpler than JMS for lightweight async, but tied to EJB

---

[:octicons-arrow-left-24: Back to Week 3](../index.md)
