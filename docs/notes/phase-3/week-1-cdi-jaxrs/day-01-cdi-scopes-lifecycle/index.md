---
tags: [jakarta-ee, cdi, scopes, lifecycle, phase-3, week-1]
---

# Day 01 — CDI 4.0 Scopes, Contexts & Lifecycle Management

> **Daily Time Investment:** 2.5 hours | **Week:** 1 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `@RequestScoped`, `@ApplicationScoped`, `@SessionScoped`, `@Dependent` — scope proxying mechanics via bytecode generation |
| Book Reading | 30 min | EE8 AppDev — Chapter 5: Contexts and Dependency Injection |
| Hands-On Lab | 75 min | Multi-scoped bean hierarchy; proxy analysis via `System.identityHashCode()` |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Book Summary — EE8 AppDev, Chapter 5**

    ---

    Deep summary of CDI: Named beans, `@Inject`, qualifiers, scope lifecycle table, CDI events (`@Observes`, `@ObservesAsync`), event ordering.

    [:octicons-arrow-right-24: Read Chapter Summary](book-ee8appdev-ch5.md)

-   :material-flask:{ .lg .middle } **Lab Guide — CDI Scopes & Proxying**

    ---

    Build a multi-scoped bean hierarchy (`@ApplicationScoped`, `@RequestScoped`, `@Dependent`) using Weld SE. Observe proxy class names, lazy instantiation, and scope boundaries.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts in Phase 3 not seen in Phase 1 or Phase 2"
    - **Jakarta EE Container vs raw JVM** — Phase 2 tuned the JVM directly; Phase 3 runs *inside* a managed container (WildFly, Payara, GlassFish) that wraps the JVM
    - **CDI BeanManager** — the central registry managing contextual instances, injection point resolution, and event dispatch
    - **Weld SE** — the JBoss Weld CDI reference implementation in standalone Java SE mode (no app server required for labs)
    - **`SeContainerInitializer`** — Jakarta CDI 4.0 API for bootstrapping Weld without an application server
    - **`beans.xml`** — CDI bean archive marker file placed in `src/main/resources/META-INF/`; can be empty or specify `bean-discovery-mode="annotated"`
    - **Proxy bytecode generation** — CDI containers use ByteBuddy / Javassist to generate subclasses at deployment time; conceptually the same as Phase 2's ByteBuddy work but done transparently

---

[:octicons-arrow-left-24: Back to Week 1](../index.md)
