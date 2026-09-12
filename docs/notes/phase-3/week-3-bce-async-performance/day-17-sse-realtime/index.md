---
tags: [jakarta-ee, sse, server-sent-events, jax-rs, real-time, phase-3, week-3]
---

# Day 17 — Real-Time Communication via Server-Sent Events (SSE)

> **Daily Time Investment:** 2.5 hours | **Week:** 3 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | JAX-RS SSE API: `SseEventSink`, `Sse`, `OutboundSseEvent`, `SseBroadcaster`, lifecycle hooks, disconnect detection |
| Book Reading | 30 min | EE8 AppDev Ch10 — JAX-RS Server-Sent Events section |
| Hands-On Lab | 75 min | Unicast progress streaming (5 ETL stages) + Multicast broadcaster (2 subscribers); verify `onClose` eviction |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **EE8 AppDev Ch10 — JAX-RS SSE API**

    ---

    `SseEventSink`, `Sse` factory, `OutboundSseEvent` builder (`.name()` / `.id()` / `.data()` / `.reconnectDelay()` / `.comment()`), `SseBroadcaster` pub/sub fan-out, `onClose` + `onError` lifecycle hooks, SSE wire format (`text/event-stream`), JavaScript `EventSource` client.

    [:octicons-arrow-right-24: Read Book Summary](book-ee8appdev-ch10-sse.md)

-   :material-flask:{ .lg .middle } **Lab Guide — SSE Unicast & Multicast**

    ---

    Unicast: stream 5 progressive ETL stages over single persistent connection; Multicast: 2 subscribers receive broadcaster alerts simultaneously; `onClose` listener detects disconnected sinks; `sink.isClosed()` prevents orphaned threads.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **Server-Sent Events (SSE)** — W3C standard HTTP/1.1 protocol for unidirectional server-to-client streaming; the connection stays open and the server pushes events; client auto-reconnects on disconnect (built-in retry mechanism)
    - **SSE Wire Format** — plain text over HTTP with `Content-Type: text/event-stream`; each event has optional fields: `event: name\n`, `id: eventId\n`, `data: payload\n`, `retry: milliseconds\n`, `: comment\n`; events are separated by a blank line `\n\n`
    - **`SseEventSink`** — represents ONE open client connection; injected via `@Context SseEventSink sink`; the JAX-RS method must return `void` and be annotated with `@Produces(MediaType.SERVER_SENT_EVENTS)`
    - **`Sse`** — the container-provided factory; injected via `@Context Sse sse`; used to create `OutboundSseEvent.Builder` instances and `SseBroadcaster` instances
    - **`SseBroadcaster`** — enterprise pub/sub channel; multiple `SseEventSink` instances subscribe; a single `broadcast(event)` call fans out to ALL registered subscribers simultaneously
    - **`sink.isClosed()`** — check before writing to detect client disconnect; prevents throwing exceptions and resource leaks from writing to a dead connection
    - **SSE vs WebSockets** — SSE is unidirectional (server → client only) over plain HTTP; WebSockets are bidirectional (full-duplex) over WS/WSS protocol; SSE works transparently through firewalls/proxies; WebSockets require proxy WS upgrade support

---

[:octicons-arrow-left-24: Back to Week 3](../index.md)
