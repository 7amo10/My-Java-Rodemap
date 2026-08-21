---
tags: [jakarta-ee, jax-rs, filters, security, name-binding, phase-3, week-1]
---

# Day 05 — JAX-RS Filters & Request/Response Pipelines

> **Daily Time Investment:** 2.0 hours | **Week:** 1 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | `ContainerRequestFilter`, `ContainerResponseFilter`, `@PreMatching`, `@NameBinding`, `@Priority`, `abortWith()` |
| Book Reading | 30 min | EE8 AppDev Ch10 (Filters) + EE8 AppDev Ch13 (Servlet Filters — conceptual comparison) |
| Hands-On Lab | 45 min | Correlation ID filter + Bearer token auth with `abortWith` + response header enrichment |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch10 — JAX-RS Filters**

    ---

    `@PreMatching`, `ContainerRequestFilter`, `ContainerResponseFilter`, `abortWith()`, correlation ID pattern.

    > Ch10 full content is in [Day 03](../day-03-jaxrs-fundamentals/book-ee8appdev-ch10.md). This note focuses on the **Filters** section specifically.

    [:octicons-arrow-right-24: Read Filter Section](book-ee8appdev-ch10-filters.md)

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch13 — Servlet Development & Filters**

    ---

    Servlet lifecycle (`doGet`/`doPost`), `@WebServlet`, Servlet `Filter` interface, `@WebFilter`, `FilterChain`, Async Servlet (`AsyncContext`), Servlet listeners, HTTP/2 server push.

    [:octicons-arrow-right-24: Read Ch13 Summary](book-ee8appdev-ch13.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JAX-RS Filters & Pipelines**

    ---

    `@PreMatching` correlation ID injection, `@Authenticated` name-bound Bearer token filter with `abortWith(401)`, `ContainerResponseFilter` response enrichment.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts in Phase 3 not seen in Phase 1 or Phase 2"
    - **`javax.ws.rs.Priorities`** — standard priority constants for filter chain ordering: `AUTHENTICATION(1000)`, `AUTHORIZATION(2000)`, `HEADER_DECORATOR(3000)`, `USER(5000)`
    - **`ResourceInfo`** — injectable into post-matching filters via `@Context ResourceInfo`; gives access to the matched `Method` and `Class` (not available in `@PreMatching` filters)
    - **`abortWith(Response)`** — short-circuits the entire pipeline; the resource method is **never invoked** but `ContainerResponseFilter` still fires
    - **Correlation IDs** — distributed tracing primitive; every request gets a unique UUID propagated through request properties, response headers, and logs
    - **Servlet Filter vs JAX-RS Filter** — Servlet `@WebFilter` fires at the Servlet container level (before JAX-RS runtime); `ContainerRequestFilter` fires inside the JAX-RS runtime (after Servlet dispatch)

---

[:octicons-arrow-left-24: Back to Week 1](../index.md)
