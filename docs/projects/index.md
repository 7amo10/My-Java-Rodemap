# :material-folder-open: Projects

Applied engineering projects that translate Phase 1 & 2 learning into production-quality systems. Each project demonstrates real JVM internals, architectural patterns, and performance engineering in action.

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

</div>

---

## :material-table: All Projects

| Project | Phase | Status | Core JVM Concepts | GitHub |
|---------|-------|--------|--------------------|--------|
| [**Helix JVM Engine & Profiler**](phase-2-helix-jvm-engine.md) | Phase 2 | :material-rocket-launch: In Progress | Bytecode generation, ClassLoader isolation, JIT profiling, GC tuning, tiered caching | [:material-github:](https://github.com/7amo10/helix-jvm-engine) |

---

## :material-map: Projects by Phase

### Phase 2 — Java Internals & Performance

Projects in this phase demonstrate mastery of JVM internals: classloading, bytecode generation, JIT compilation tiers, garbage collection algorithms, and performance engineering.

| Project | JVM Concepts Demonstrated |
|---------|--------------------------|
| [Helix JVM Scripting Engine](phase-2-helix-jvm-engine.md) | Runtime bytecode generation (ByteBuddy + ASM), multi-tenant ClassLoader isolation, Metaspace management, tiered reference caching (Strong/Soft/Weak), JIT warm-up strategies, GC tuning (G1, Metaspace), JFR custom events, async-profiler integration |

---

## :material-plus-box: Adding a New Project

Use the [Project Template](project-template.md) to document your projects consistently. Every project page should cover:

| Section | Content |
|---------|---------|
| **Problem** | What gap does this project fill? What fails without it? |
| **Solution** | High-level approach and key design decisions |
| **Architecture** | Internal subsystems, data flows, Mermaid diagrams |
| **JVM Concepts Applied** | Map each subsystem to a Phase 1/2 concept learned |
| **Benchmarks** | JMH numbers, latency histograms, throughput measurements |
| **Links** | GitHub repo, design specs, related learning notes |
