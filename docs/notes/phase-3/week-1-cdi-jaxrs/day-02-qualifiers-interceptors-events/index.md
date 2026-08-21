---
tags: [jakarta-ee, cdi, qualifiers, interceptors, events, phase-3, week-1]
---

# Day 02 — Qualifiers, Producers, Interceptors & Decoupled Events

> **Daily Time Investment:** 2.5 hours | **Week:** 1 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | Custom `@Qualifier`, `@Produces`/`@Disposes`, `@AroundInvoke`, `@Observes`/`@ObservesAsync` |
| Book Reading | 30 min | EE8 AppDev Ch5 (Qualifiers & Events) + EE Patterns Ch5 (Interceptors) |
| Hands-On Lab | 75 min | `@Monitored` interceptor with `ThreadMXBean`; decoupled audit event bus |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch5 — Qualifiers & CDI Events**

    ---

    Qualifiers, `@Produces`, `@Disposes`, synchronous vs async events, event ordering with `@Priority`.

    > Chapter 5 scopes & injection are in [Day 01](../day-01-cdi-scopes-lifecycle/book-ee8appdev-ch5.md) — this page focuses on the advanced wiring section.

    [:octicons-arrow-right-24: Read Ch5 Advanced Section](book-ee8appdev-ch5-qualifiers-events.md)

-   :material-book-open-page-variant:{ .lg .middle } **EE Patterns Ch5 — Infrastructural Patterns**

    ---

    Adam Bien's patterns: Service Starter (`@Startup @Singleton`), Singleton caching, Thread Tracker (`@AroundInvoke`), Context Holder (`TransactionSynchronizationRegistry`), Bean Locator, Payload Extractor, Dependency Injection Extender.

    [:octicons-arrow-right-24: Read EE Patterns Ch5](book-ee-patterns-ch5.md)

-   :material-flask:{ .lg .middle } **Lab Guide — CDI Advanced Wiring**

    ---

    Build enum-backed qualifiers, `@Produces`/`@Disposes` resource factories, `@AroundInvoke` performance interceptors, and sync+async CDI event bus using Weld SE.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts in Phase 3 not seen in Phase 1 or Phase 2"
    - **`InvocationContext`** — CDI passes this to `@AroundInvoke` methods; gives access to `getMethod()`, `getParameters()`, `getTarget()`, and critically `proceed()` (calls the actual method)
    - **`ManagementFactory.getThreadMXBean()`** — returns `ThreadMXBean` for per-thread CPU time; bridges Phase 2 JVM internals into Phase 3 interceptors
    - **Record types as event payloads** — Java 16+ records (`record AuditEvent(String id, Instant time) {}`) are ideal CDI event payloads — immutable, value-based, no boilerplate
    - **`CompletionStage`** — returned by `event.fireAsync()` — the same `CompletionStage`/`CompletableFuture` from Phase 1 concurrency
    - **`@InterceptorBinding` vs `@Qualifier`** — both are meta-annotations but serve different purposes: qualifiers disambiguate injection, interceptor bindings attach cross-cutting behavior

---

[:octicons-arrow-left-24: Back to Week 1](../index.md)
