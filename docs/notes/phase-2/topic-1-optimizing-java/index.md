---
id: topic-1-optimizing-java-index
aliases: []
tags:
- java
- performance
- jvm
- optimizing-java
- phase-2
---

# :material-speedometer: Topic 1: Optimizing Java — Part I: Foundations

> **Book:** Optimizing Java — Practical Techniques for Improving JVM Application Performance
>
> **Authors:** Benjamin J. Evans, James Gough, Chris Newland (O'Reilly Media)
>
> **Part Covered:** Part I — Foundations (Chapters 1–5)

---

## :material-notebook-outline: Topic Structure

| Document | Chapter | Coverage | Status |
|----------|---------|----------|--------|
| [:material-book-open-page-variant: Chapter 1 — Intro & Performance Vocabulary](book-reading-ch1.md) | Ch 1 | Performance as experimental science, 7 observables (Throughput/Latency/Capacity/Utilization/Efficiency/Scalability/Degradation), performance graphs, non-normal statistics, HdrHistogram | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 2 — Overview of the JVM](book-reading-ch2.md) | Ch 2 | Classloading chain (Bootstrap/Extension/Application), bytecode & `javap`, class file anatomy, HotSpot JIT (C1/C2/Tiered), Code Cache, JMX/Java agents/JVMTI, VisualVM | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 3 — Hardware & OS](book-reading-ch3.md) | Ch 3 | CPU memory hierarchy, cache lines, false sharing, NUMA, TLBs & large pages, branch prediction, hardware memory models, OS scheduler, context switches, `vmstat` | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 4 — Performance Testing Types & Antipatterns](book-reading-ch4.md) | Ch 4 | Latency/Throughput/Load/Stress/Endurance/Capacity/Degradation tests, top-down performance, performance antipatterns catalogue (Distracted by Shiny, Distracted by Simple, DataLite, UAT Is My Desktop) | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 5 — Microbenchmarking & Statistics](book-reading-ch5.md) | Ch 5 | Seven benchmark flaws, JMH framework (annotations, Blackhole, forks, warmup), non-normal distributions, logarithmic percentiles, HdrHistogram, spurious correlation | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Chapter 1: Optimizing Java — Introduction & Performance Vocabulary

Establishes the **philosophical and methodological foundation** for all Java performance work. The central thesis: performance is an **experimental science** — it requires defining quantitative objectives, measuring existing behavior, forming hypotheses, making targeted changes, and retesting. There are no magic JVM flags or secret algorithms. The book introduces a **taxonomy of seven performance observables**: Throughput (work rate), Latency (response time), Capacity (concurrent load), Utilization (resource consumption %), Efficiency (throughput/utilization), Scalability (performance growth with resources), and Degradation (behavior under increasing load). Critically explains why JVM performance data is **never normally distributed** — the hard lower bound from JIT steady state and the long right tail from GC pauses make standard deviation meaningless. Introduces logarithmic percentile sampling (p50→p90→p99→p99.9→p99.99) and HdrHistogram as the correct tools. Covers the five disciplines required: methodology, testing theory, measurement & statistics, analysis, and underlying technology.

### Chapter 2: Overview of the JVM — Bytecode, Classloading & HotSpot

Provides the JVM internals foundation needed for all subsequent performance work. Covers the **three-tier classloading delegation chain** (Bootstrap loads rt.jar/java.base; Extension loads ext/ and Nashorn; Application loads user classpath), with the crucial rule that class identity = ClassLoader + FQN. Explains **bytecode** as the intermediate, architecture-neutral representation produced by `javac`, and how to read `javap -c` disassembly (opcode naming conventions: type prefix + operation). Introduces **HotSpot JIT's 5-tier compilation model**: Tier 0 (interpreter, profiling), Tiers 1-3 (C1, progressively more profiling), Tier 4 (C2, aggressive optimization using profiling data — inlining, escape analysis, loop unrolling, devirtualization). Explains the **warmup problem** — any measurement before C2 compilation at ~10K-15K invocations is measuring a mix of interpreted and C1 code. Covers the four JVM monitoring mechanisms (JMX, Java agents, JVMTI, Serviceability Agent) and VisualVM's five monitoring tabs.

### Chapter 3: Hardware & OS — CPU Caches, NUMA, the Scheduler & Mechanical Sympathy

Establishes the hardware reality that Java code must work within — the concept of **mechanical sympathy** (understanding hardware to write sympathetic code). The **CPU memory hierarchy** is the central concept: L1 (~2ns), L2 (~5ns), L3 (~20ns), RAM (~100ns) — a 50x difference between L1 and RAM means cache-friendly algorithms can be 50x faster than cache-unfriendly ones. **Cache lines** (64 bytes) mean spatial locality matters enormously — sequential array traversal is fast; pointer chasing through object graphs is slow. **False sharing** occurs when two threads write to different fields in the same cache line, causing invisible cache invalidation storms that can 10x degrade throughput. **NUMA** (multi-socket servers) means remote memory access is ~2x slower — `-XX:+UseNUMA` enables NUMA-aware Eden allocation. **TLB misses** from virtual memory translation add 100-1000 cycle penalties — large pages (`-XX:+UseLargePages`) reduce this. **Branch misprediction** costs ~15-20 pipeline cycles — unpredictable conditionals kill performance. **Hardware memory models** vary by architecture (ARMv7/POWER allow more reorderings than x86) — the JMM abstracts this but requires explicit synchronization. The **OS scheduler** manages thread access to CPU via run queues and time quanta; context switches cost ~1-50µs with cache warming overhead. `vmstat 1` provides the key system-level view.

### Chapter 4: Performance Testing Types, Antipatterns & Best Practices

The practical guide to **how to run performance tests correctly** and **how to avoid wasting time** on the wrong things. Defines seven distinct test types, each answering a specific question: **Latency** (what's the response time distribution?), **Throughput** (how many ops/sec at what load?), **Load** (can it handle this specific volume?), **Stress** (where is the breaking point?), **Endurance/Soak** (does it hold up over 24-72 hours? — memory leaks, cache pollution), **Capacity Planning** (how does adding resources affect capacity?), and **Degradation/Chaos** (how does it behave when a component fails?). Presents the **performance antipatterns catalogue**: Distracted by Shiny (targeting new tech because it's interesting, not because measurement says so), Distracted by Simple (avoiding complex specialist code), Performance Tuning Wizard (lone genius myth), Blame Donkey (blaming libraries without evidence), Missing the Bigger Picture (micro-optimizing the wrong bottleneck), UAT Is My Desktop (wrong hardware), and DataLite (simplified test data that doesn't match production behavior). The **three golden rules**: identify what you care about and measure it, optimize what matters (not what's easy), play the big points first. Advocates **top-down performance**: start at system level, drill into subsystems, then specific methods — never start at the method level.

### Chapter 5: Microbenchmarking, Statistics & JMH

**When and how to use microbenchmarks correctly** — and why they are almost always the wrong tool. Dissects the classic `ClassicSort` hand-rolled benchmark's **seven fatal flaws**: no JVM warmup (JIT compiles during measurement), dead code elimination (JIT removes result-less sort), GC interference (non-deterministic pauses pollute timing), no iteration calibration, no statistical margin of error, observer effect, and wrong JVM mode. Introduces **JMH** (Java Microbenchmark Harness) by OpenJDK engineers — it generates correct harness code via annotation processing (not reflection) and addresses all seven flaws: `@Warmup` forces JIT steady state, `Blackhole.consume()` defeats DCE, `@Fork(N)` creates independent JVM processes for each run, `@State(Scope.Thread)` correctly manages state lifecycle, and `@Measurement` controls timing capture. Covers the **statistical problem** with JVM data: right-skewed, non-Gaussian distributions where standard deviation is meaningless. The correct approach: **logarithmic percentile sampling** (p50→p90→p99→p99.9→...) and **HdrHistogram** for high dynamic range distributions. Warns against **spurious correlation** — observed correlation between metrics is not causation; always control variables explicitly.

---

## :material-book-open-variant: What You'll Master

- **Performance Methodology** — Treating tuning as experimental science with quantitative objectives
- **Performance Observables** — Throughput, Latency, Capacity, Utilization, Efficiency, Scalability, Degradation
- **JVM Execution Model** — Classloading, bytecode, JIT tiers (C1/C2), Code Cache, warmup
- **Hardware Foundations** — CPU caches, cache lines, false sharing, NUMA, TLBs, branch prediction
- **OS Internals** — Scheduler, context switches, `vmstat` interpretation, thread lifecycle
- **Memory Models** — Why different architectures allow different reorderings; JMM as the abstraction
- **Testing Taxonomy** — Six test types and what question each answers
- **Antipattern Recognition** — Seven named antipatterns and how to avoid them
- **Correct Benchmarking** — JMH setup, annotations, Blackhole, forks, warmup strategy
- **Statistical Literacy** — Why JVM data is non-normal, logarithmic percentiles, HdrHistogram

---

## :material-map-marker-path: Optimizing Java — Complete Book Structure

The full book (15 chapters) is split into reading parts that align with your weekly study plan:

| Reading Group | Chapters | Topics |
|:---:|---|---|
| **Week 1 (This part)** | 1–5 | Foundations: observables, JVM anatomy, hardware, testing, statistics |
| **Week 2** | 6–8 | Garbage Collection: GC algorithms, heap analysis, GC tuning |
| **Week 3** | 9–10 | Bytecode & JIT: bytecode in depth, JIT compilation internals |
| **Week 4** | 11–12 | Profiling & Concurrency: profilers, thread analysis, lock contention |
| **Week 5** | 13–15 | Profiling tools, JVM as a platform, advanced optimization techniques |

---

## :material-cogs: Key Internals to Understand

- **JIT Compilation Tiers** — How HotSpot transitions code from Tier 0 (interpreter) to Tier 4 (C2 aggressive) based on invocation counts and profiling data
- **Escape Analysis** — How C2 can stack-allocate objects, eliminate synchronization on non-shared objects, and perform scalar replacement
- **Code Cache Management** — Fixed-size region for JIT-compiled native code; flushing behavior and `-XX:ReservedCodeCacheSize`
- **Cache Line Invalidation Protocol** — MESI protocol: how multicore CPUs maintain cache coherence and what false sharing actually does at the hardware level
- **NUMA Memory Allocation** — How `-XX:+UseNUMA` partitions Eden across NUMA nodes, and why it matters for allocation-heavy workloads
- **HdrHistogram Internals** — Logarithmic bucket allocation, concurrent recording strategy, compressed serialization format

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Read Chapter 1 — Introduction & Performance Vocabulary
- [x] Read Chapter 2 — Overview of the JVM
- [x] Read Chapter 3 — Hardware and Operating Systems
- [x] Read Chapter 4 — Performance Testing Patterns and Antipatterns
- [x] Read Chapter 5 — Microbenchmarking and Statistics
- [x] Write Chapter 1 notes
- [x] Write Chapter 2 notes
- [x] Write Chapter 3 notes
- [x] Write Chapter 4 notes
- [x] Write Chapter 5 notes
- [ ] Week 2: Read Chapters 6–8 (Garbage Collection)

---

*Start Date: 2026-07-16 | Week 1 Completed: 2026-07-16*
