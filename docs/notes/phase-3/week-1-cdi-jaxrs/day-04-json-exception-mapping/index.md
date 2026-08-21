---
tags: [jakarta-ee, json-b, json-p, exception-mapping, rfc-7807, phase-3, week-1]
---

# Day 04 — JSON-B / JSON-P Processing & Unified Exception Handling

> **Daily Time Investment:** 2.5 hours | **Week:** 1 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | JSON-B 3.0 annotations, JSON-P DOM/Streaming API, `ExceptionMapper<T>`, RFC-7807 |
| Book Reading | 30 min | EE8 AppDev Ch6 (JSON-P + JSON-B) + EE8 AppDev Ch10 (ExceptionMapper) |
| Hands-On Lab | 75 min | RFC-7807 `ExceptionMapper` pipeline + JSON-P AST endpoint |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch6 — JSON-P & JSON-B**

    ---

    JSON-P Model API, Streaming API, JSON Pointer, JSON Patch. JSON-B `toJson()`/`fromJson()` basics.

    [:octicons-arrow-right-24: Read Ch6 Summary](book-ee8appdev-ch6.md)

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch10 — ExceptionMapper (cross-ref)**

    ---

    JAX-RS `ExceptionMapper<T>`, `@Provider`, RFC-7807 problem details.

    > Ch10 full content is in [Day 03](../day-03-jaxrs-fundamentals/book-ee8appdev-ch10.md). This page covers the **ExceptionMapper** section specifically.

    [:octicons-arrow-right-24: Read ExceptionMapper Section](book-ee8appdev-ch10-exception-mapper.md)

-   :material-flask:{ .lg .middle } **Lab Guide — JSON Processing & Exception Architecture**

    ---

    `@JsonbProperty`, `@JsonbTransient`, `JsonbAdapter`, dynamic JSON-P DOM, RFC-7807 ExceptionMapper chain.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts in Phase 3 not seen in Phase 1 or Phase 2"
    - **RFC-7807** — "Problem Details for HTTP APIs" IETF standard; defines `application/problem+json` media type for standardized error responses
    - **`JsonbAdapter<OriginalType, AdaptedType>`** — two-method interface: `adaptToJson(O)` converts to wire type; `adaptFromJson(A)` converts back
    - **JSON-P vs JSON-B** — JSON-P is the low-level DOM/streaming API; JSON-B builds on top for transparent POJO mapping
    - **`@Provider` registration** — in embedded Jersey use `ResourceConfig.register(MyMapper.class)`; on a real app server it's auto-discovered via classpath scanning
    - **`Content-Type: application/problem+json`** — a distinct MIME type from `application/json`; clients switch error handling behavior on it

---

[:octicons-arrow-left-24: Back to Week 1](../index.md)
