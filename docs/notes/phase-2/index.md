# Phase 2: Java Internals & Performance

Master the JVM internals, performance engineering, garbage collection, and the full stack of Java optimization — from bytecode to CPU cache lines.

**Duration:** 2-3 Months

---

## :material-bookshelf: Learning Resources

| Resource Type | Name |
|--------------|------|
| **Book** | [Optimizing Java](https://www.oreilly.com/library/view/optimizing-java/9781492039259/) — Benjamin J. Evans, James Gough, Chris Newland (O'Reilly) |
| **Book** | Java Performance: The Definitive Guide — Scott Oaks (O'Reilly) |
| **Tool** | JDK Mission Control (JMC) + Java Flight Recorder (JFR) |
| **Tool** | async-profiler / Honest Profiler (flame graphs) |

---

## :material-folder-open: Topics

<div class="grid cards" markdown>

-   :material-speedometer:{ .lg .middle } **Topic 1: Optimizing Java — Part I: Foundations**

    ---

    Performance vocabulary (throughput, latency, observables), JVM anatomy (classloading, JIT tiers), hardware (CPU caches, NUMA, OS scheduler), performance testing types & antipatterns, microbenchmarking with JMH, non-normal statistics & HdrHistogram.

    **Book:** Optimizing Java, Chapters 1–5

    [:octicons-arrow-right-24: Explore Topic](topic-1-optimizing-java/index.md)

-   :material-recycle:{ .lg .middle } **Topic 2: Garbage Collection Deep Dive**

    ---

    GC algorithms (Serial, Parallel, CMS, G1, ZGC, Shenandoah), heap structure, tri-color marking, SATB, Brooks pointers, colored pointers, Card Tables, safepoints, GC log analysis, allocation profiling, GC tuning parameters.

    **Book:** Optimizing Java, Chapters 6–8

    [:octicons-arrow-right-24: Explore Topic](topic-2-garbage-collection/index.md)

-   :material-lightning-bolt:{ .lg .middle } **Topic 3: Bytecode & JIT Internals**

    ---

    JVM as a stack machine, bytecode opcode families, method invocation opcodes, template interpreter, tiered compilation (Tiers 0–4), Code Cache, inlining (the gateway optimization), loop unrolling, escape analysis & scalar replacement, monomorphic dispatch, intrinsics, 8000-byte compilation wall.

    **Book:** Optimizing Java, Chapters 9–10

    [:octicons-arrow-right-24: Explore Topic](topic-3-bytecode-jit/index.md)

-   :material-chart-gantt:{ .lg .middle } **Topic 4: Profiling & Concurrency Performance**

    ---

    CPU and allocation profilers (async-profiler, JFR), flame graphs, lock contention analysis, thread dump analysis, Amdahl's Law in practice.

    **Book:** Optimizing Java, Chapters 11–12

    *[:octicons-clock-24: Coming — Week 4]*

-   :material-wrench:{ .lg .middle } **Topic 5: Advanced JVM Optimization Techniques**

    ---

    Profiler tools in depth, JVM as a platform, advanced code-level optimizations, data structures for performance, off-heap memory, Project Valhalla preview.

    **Book:** Optimizing Java, Chapters 13–15

    *[:octicons-clock-24: Coming — Week 5]*

</div>

---

## :material-book-open-variant: Topic Structure

Each topic folder contains reading notes keyed to chapters of the source book:

| Document Type | Purpose |
|---------------|---------|
| `book-reading-chN.md` | Deep notes for Chapter N of Optimizing Java — concepts, code examples, diagrams, internals |
| `index.md` | Topic overview, chapter map, progress tracker, key internals to understand |

---

## :material-map-marker-path: Optimizing Java — Book Reading Plan

| Reading Group | Chapters | Topics | Status |
|:---:|---|---|:---:|
| Week 1 | 1–5 | Foundations: observables, JVM, hardware, testing, statistics | :material-check-circle: Complete |
| Week 2 | 6–8 | Garbage Collection: algorithms, heap, GC logs, tuning | :material-check-circle: Complete |
| Week 3 | 9–10 | Bytecode & JIT: internals, inlining, escape analysis | :material-check-circle: Complete |
| Week 4 | 11–12 | Profiling & Concurrency: profilers, thread analysis, locks | :octicons-clock-24: Upcoming |
| Week 5 | 13–15 | Advanced Tools & Optimization | :octicons-clock-24: Upcoming |

---

## :material-target: Project Ideas

1. **JVM Performance Profiling Lab** — Profile a real application with JFR + async-profiler, generate flame graphs, identify GC and CPU bottlenecks
2. **GC Tuning Experiment** — Run the same workload under G1, ZGC, and Shenandoah; compare GC pause distributions using HdrHistogram
3. **Cache-Friendly Data Structures** — Implement and benchmark array-of-structs vs struct-of-arrays; measure false sharing impact
4. **JMH Benchmark Suite** — Benchmark algorithm alternatives for a real business problem (string parsing, JSON deserialization, sorting)
5. **NUMA-Aware Application** — Profile a multi-threaded application on a NUMA machine with and without `-XX:+UseNUMA`

---

## :material-checkbox-marked-outline: Phase Progress

- [x] Topic 1: Optimizing Java — Part I: Foundations (Chapters 1–5)
- [x] Topic 2: Garbage Collection Deep Dive (Chapters 6–8)
- [x] Topic 3: Bytecode & JIT Internals (Chapters 9–10)
- [ ] Topic 4: Profiling & Concurrency Performance (Chapters 11–12)
- [ ] Topic 5: Advanced JVM Optimization Techniques (Chapters 13–15)
- [ ] Phase 2 Capstone Project

---

*Phase Start Date: 2026-07-16 | Target Completion: <!-- Add date -->*
