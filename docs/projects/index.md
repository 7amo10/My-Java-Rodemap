# :material-folder-open: Projects

Applied engineering projects that translate roadmap learning into production-quality systems. Each project demonstrates real JVM internals, architectural patterns, and performance engineering in action.

---

## :material-view-grid: Projects Overview

<div class="grid cards" markdown>

-   :material-engine:{ .lg .middle } **Helix — JVM Scripting Engine & Profiler**

    ---

    A production-grade dynamic rule compilation engine that generates JVM bytecode at runtime from JSON-defined business rules. Simultaneously an advanced JVM observability platform with ClassLoader isolation, tiered reference caching, JIT compilation monitoring, JFR event streaming, JOL memory layout analysis, and a reactive Lanterna TUI dashboard.

    **Phase:** Phase 2 — Java Internals & Performance

    **Key Tech:** ByteBuddy · ASM · JFR · async-profiler · JOL · Caffeine · Maven Multi-Module

    **Performance:** `> 120,000 ops/sec` · `~8 ns/op` at C2 Tier 4 · `~1.7ms` ASM compile

    [:material-github: GitHub](https://github.com/7amo10/helix-jvm-engine){ .md-button } [:octicons-arrow-right-24: Full Write-Up](phase-2-helix-jvm-engine.md){ .md-button .md-button--primary }

-   :material-brain:{ .lg .middle } **helix-cortex — Enterprise Jakarta EE Platform**

    ---

    A cloud-native supervisory API, asynchronous bytecode analyzer, and live JVM observability platform wrapping the helix engine core. Exposes rule compilation, execution, and live JVM telemetry through a fully-secured JAX-RS REST API with JWT authentication, two-role RBAC, JPA 3.1 persistence, and real-time Server-Sent Events.

    **Phase:** Phase 3 — Enterprise Java (Jakarta EE 10)

    **Key Tech:** Jakarta EE 10 · WildFly 31 · PostgreSQL 16 · HikariCP · helix-jvm-engine · OW2 ASM · Docker Compose

    **API:** `13 endpoints` · `4 BCE layers` · `6 JPA entities` · `0 N+1 queries`

    [:material-github: GitHub](https://github.com/7amo10/helix-cortex){ .md-button } [:octicons-arrow-right-24: Full Write-Up](phase-3-helix-cortex.md){ .md-button .md-button--primary }

</div>

---

## :material-table: All Projects

| Project | Phase | Status | Core Concepts | GitHub |
|---------|-------|--------|---------------|--------|
| [**Helix JVM Engine & Profiler**](phase-2-helix-jvm-engine.md) | Phase 2 | :material-rocket-launch: In Progress | Bytecode generation, ClassLoader isolation, JIT profiling, GC tuning, tiered caching | [:material-github:](https://github.com/7amo10/helix-jvm-engine) |
| [**helix-cortex Enterprise Platform**](phase-3-helix-cortex.md) | Phase 3 | :material-rocket-launch: In Progress | CDI 4.0, JAX-RS 3.1, JPA 3.1, JWT/RBAC, SSE telemetry, ManagedExecutorService, HikariCP | [:material-github:](https://github.com/7amo10/helix-cortex) |

---

## :material-map: Projects by Phase

### Phase 2 — Java Internals & Performance

Projects in this phase demonstrate mastery of JVM internals: classloading, bytecode generation, JIT compilation tiers, garbage collection algorithms, and performance engineering.

| Project | JVM Concepts Demonstrated |
|---------|--------------------------|
| [Helix JVM Scripting Engine](phase-2-helix-jvm-engine.md) | Runtime bytecode generation (ByteBuddy + ASM), multi-tenant ClassLoader isolation, Metaspace management, tiered reference caching (Strong/Soft/Weak), JIT warm-up strategies, GC tuning (G1, Metaspace), JFR custom events, async-profiler integration |

### Phase 3 — Enterprise Java (Jakarta EE 10)

Projects in this phase demonstrate mastery of enterprise Java: the Jakarta EE container model, CDI dependency injection, REST API design, JPA object-relational mapping, transactional boundaries, and cloud-native deployment.

| Project | Jakarta EE Concepts Demonstrated |
|---------|----------------------------------|
| [helix-cortex Enterprise Platform](phase-3-helix-cortex.md) | CDI 4.0 (Producers, @ObservesAsync, interceptors), JAX-RS 3.1 (@NameBinding filters, SSE), JPA 3.1 (Criteria API, EntityGraph, optimistic locking, L2 cache), JTA transactions, Jakarta Security 3.0 (PBKDF2, HMAC-SHA256 JWT), ManagedExecutorService, HikariCP tuning, Docker Compose |

---

## :material-plus-box: Adding a New Project

Use the [Project Template](project-template.md) to document your projects consistently. Every project page should cover:

| Section | Content |
|---------|---------|
| **Problem** | What gap does this project fill? What fails without it? |
| **Solution** | High-level approach and key design decisions |
| **Architecture** | Internal subsystems, data flows, Mermaid diagrams |
| **JVM Concepts Applied** | Map each subsystem to a Phase 1/2/3 concept learned |
| **Benchmarks** | JMH numbers, latency histograms, throughput measurements |
| **Links** | GitHub repo, design specs, related learning notes |
