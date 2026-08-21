---
tags: [jakarta-ee, jax-rs, rest, http, phase-3, week-1]
---

# Day 03 — JAX-RS 3.1 Fundamentals & REST Resource Architecture

> **Daily Time Investment:** 2.0 hours | **Week:** 1 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `@Path`, HTTP verbs, `@PathParam`/`@QueryParam`/`@HeaderParam`, content negotiation, `Response` builder |
| Book Reading | 30 min | EE8 AppDev Ch10 + (Web APIs / JEETT Ch4) |
| Hands-On Lab | 45 min | CRUD `TelemetryResource` with embedded Grizzly + Jersey |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch10 — RESTful Web Services with JAX-RS**

    ---

    `@Path`, HTTP verbs, parameter annotations, content negotiation, `Response` builder, JAX-RS client API, Server-Sent Events intro.

    [:octicons-arrow-right-24: Read Ch10 Summary](book-ee8appdev-ch10.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JAX-RS REST Architecture**

    ---

    Full CRUD `TelemetryResource` using embedded Jersey + Grizzly. Verify all 7 endpoint behaviors with Java `HttpClient`.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts in Phase 3 not seen in Phase 1 or Phase 2"
    - **Jersey** — the JAX-RS 3.1 reference implementation; Grizzly is a lightweight NIO HTTP server used for embedded testing (no app server needed)
    - **`ResourceConfig`** — Jersey-specific subclass of `Application` for programmatic registration of resources and providers
    - **`UriBuilder`** — fluent builder for constructing absolute URI strings — used for `Location` headers in 201 Created responses
    - **`@BeanParam`** — bundles multiple parameter annotations (`@PathParam`, `@QueryParam`, `@HeaderParam`) into a single POJO, reducing method signature clutter
    - **Richardson Maturity Model** — REST quality levels 0–3; Level 3 (HATEOAS) includes hypermedia links in responses

---

[:octicons-arrow-left-24: Back to Week 1](../index.md)
