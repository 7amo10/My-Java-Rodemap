---
tags: [jakarta-ee, hikaricp, connection-pool, performance-tuning, gc, phase-3, week-3]
---

# Day 19 — Database Connection Pool & HTTP Throughput Tuning

> **Daily Time Investment:** 2.0 hours | **Week:** 3 | **Phase:** 3

---

## :material-calendar-today: Daily Schedule

| Segment | Duration | Activity |
|---------|----------|----------|
| Core Theory | 45 min | HikariCP sizing formula, HTTP worker alignment, `connectionTimeout` tuning, GC pauses vs P99 latency |
| Book Reading | 30 min | Pro Persistence Ch14 — Packaging and Deployment (DataSource & Connection Pool configuration) |
| Hands-On Lab | 45 min | Benchmark 4 pool profiles (starved, balanced, oversized, GC stress); record throughput and tail latency under 25 concurrent requests |

---

## :material-file-document: Files in This Day

<div class="grid cards" markdown>

-   :material-book-open-page-variant:{ .lg .middle } **Pro Persistence Ch14 — Packaging, Deployment & Connection Pools**

    ---

    `persistence.xml` configuration, `<jta-data-source>` / `<non-jta-data-source>` JNDI lookup, packaging in WAR/EAR, `transaction-type=JTA` vs `RESOURCE_LOCAL`, schema generation properties, JDBC properties for Java SE, `@Table`/`@Column` DDL hints, HikariCP datasource properties.

    [:octicons-arrow-right-24: Read Book Summary](book-pro-persistence-ch14.md)

-   :material-flask:{ .lg .middle } **Lab Guide — Connection Pool & Throughput Profiling**

    ---

    Four experiment profiles: starved pool (2 conns / 32 workers), balanced production (16 / 16), oversized pool (80 / 64), heap memory stress. Measure mean response time, P99 latency, and GC pause accumulation.

    [:octicons-arrow-right-24: Start Lab](lab-guide.md)

</div>

---

## :material-note-alert: Prerequisites to Continue

!!! note "New concepts not seen in Phase 1 or Phase 2"
    - **HikariCP** — the fastest JDBC connection pool; default in WildFly, Quarkus, and Spring Boot; uses a lock-free ConcurrentBag for connection management; much faster than DBCP2 or c3p0
    - **Connection Pool Sizing Law** — counterintuitively, MORE connections ≠ MORE performance; beyond a certain point, additional connections cause OS context-switch overhead and disk I/O seek contention; HikariCP's formula: `Pool Size = (CPU_cores × 2) + effective_spindle_count`
    - **Thread Starvation** — when HTTP worker threads outnumber available DB connections, workers queue waiting for connections; queue depth causes tail latency spikes; `connectionTimeout` limits how long a thread waits before throwing `SQLTimeoutException`
    - **P95 / P99 Latency** — percentile latency metrics; P99 = the 99th percentile response time (i.e., 99% of requests complete within this time); it reveals outliers that averages hide; crucial for SLA compliance
    - **GC Stop-the-World pause** — during a garbage collection cycle (Young Generation minor GC), ALL application threads pause; during a full GC, the pause is longer; high object allocation rates from endpoint handlers cause frequent GC, inflating P99 latency
    - **`connectionTimeout`** — HikariCP property: max milliseconds a thread waits to acquire a connection from the pool; if exceeded, throws `SQLTimeoutException`; prevents thread starvation from cascading across the service
    - **`maxLifetime`** — HikariCP property: max age of a connection in the pool; prevents DB-side stale connection closure; should be slightly less than the database's `wait_timeout`
    - **Grizzly** — the embedded NIO HTTP server used in standalone Payara/GlassFish; configures its HTTP worker thread pool separately from the JTA container thread pool

---

[:octicons-arrow-left-24: Back to Week 3](../index.md)
