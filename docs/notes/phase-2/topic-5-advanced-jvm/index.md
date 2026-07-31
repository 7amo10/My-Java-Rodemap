---
id: topic-5-advanced-jvm-index
aliases: []
tags:
- java
- performance
- jvm
- profiling
- logging
- aeron
- valhalla
- graal
- project-loom
- optimizing-java
- phase-2
---

# :material-wrench: Topic 5: Advanced JVM Optimization Techniques

> **Book:** Optimizing Java — Practical Techniques for Improving JVM Application Performance
>
> **Authors:** Benjamin J. Evans, James Gough, Chris Newland (O'Reilly Media)
>
> **Part Covered:** Part V — Advanced Topics (Chapters 13–15)

---

## :material-notebook-outline: Topic Structure

| Document | Chapter | Coverage | Status |
|----------|---------|----------|--------|
| [:material-book-open-page-variant: Chapter 13 — Profiling](book-reading-ch13.md) | Ch 13 | Execution vs allocation profiling, safepoint bias & `GetCallTrace` vs `AsyncGetCallTrace`, VisualVM/JProfiler/YourKit/JMC-JFR comparisons, Honest Profiler, async-profiler, Linux `perf` + `perf-map-agent`, flame graphs (CPU/allocation/off-CPU), TLAB sampling, heap dump analysis, `hprof` deprecation | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 14 — High-Performance Logging and Messaging](book-reading-ch14.md) | Ch 14 | Logging performance benchmarks (java.util.logging vs Logback vs Log4j), Log4j 2.6 zero-allocation logger (ThreadLocal reuse), Agrona buffers hierarchy & queue cache-line padding & ring buffer, SBE (copy-free, steady-state allocation, word-aligned streaming), Aeron messaging transport (8 latency principles, memory-mapped persistent log, lock-free appender, NAKs, message coordinates) | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 15 — Java 9 and the Future](book-reading-ch15.md) | Ch 15 | Segmented Code Cache, Compact Strings, `invokedynamic` String concat, C2 SIMD improvements, G1 as default, Java 10 JEPs (JEP 307 Parallel GC for G1, JEP 310 AppCDS, JEP 312 Thread-Local Handshakes), VarHandles (JEP 193), Project Valhalla & Value Types, Graal & Truffle (Futamura projection), `jaotc` AOT, SubstrateVM, Project Loom (fibers/continuations) | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Chapter 13: Profiling — Finding What's Actually Hot

The diagnostic chapter that bridges theory and practice. Establishes the **two pre-profiling prerequisites** that are violated most often: (1) confirm CPU is spending time in user mode (not kernel, not GC STW) via GC logs; (2) confirm the problem is in application code, not GC tuning. Explains the fundamental **safepoint bias** problem: all traditional profilers (`VisualVM`, `JProfiler`, `YourKit`) use HotSpot's `GetCallTrace()` internal API which requires application threads to be at a JVM safepoint — but the JIT removes safepoint polls from tight unrolled counted loops, causing the profiler to completely miss code inside those loops. The fix is **`AsyncGetCallTrace()`** — a private HotSpot API that captures stack traces asynchronously via OS signals (`SIGPROF`) without waiting for safepoints. **async-profiler** uses this alongside Linux `perf_events` to produce accurate CPU profiles with < 2% overhead. **JFR (Java Flight Recorder)** records 50+ JVM event types (GC, JIT compilations, class loading, locks, I/O, exceptions, TLAB allocations) into thread-local buffers with < 2% overhead; **JMC (Java Mission Control)** is the desktop analysis tool. **Flame graphs** visualize profiling data as stacked boxes where width = CPU time fraction — the widest boxes at the *top* of tall stacks are the hot spots. **Allocation profiling** via TLAB exhaustion sampling (JDK 7u40+) is production-safe; bytecode instrumentation (ASM-injected `NEW`/`NEWARRAY` hooks) is development-only. **Heap dump analysis** captures the entire object graph (300–400% of heap size on disk!) via tools like YourKit or Eclipse MAT for leak investigation. `hprof` was removed in Java 9 (JEP 240).

### Chapter 14: High-Performance Logging and Messaging — Mechanical Sympathy in Practice

Opens with Kirk Pepperdine's customer case study: 4.5-second budget, 4.2 seconds was logging — logging was the bottleneck. Benchmarks (iMac & AWS EC2) show `java.util.logging` (JUL) is 2.5–3× slower than Logback and 14× slower than Log4j 2.7 for formatted output. **Log4j 2.6's zero-allocation logger** uses `ThreadLocal` object pools and reusable byte buffers to eliminate all temporary object creation — JFR confirmation shows **141 GC collections** (Log4j 2.5) → **0 GC collections** (Log4j 2.6) over the same 12-second test run. Introduces the **Real Logic stack** — Martin Thompson's suite of open-source low-latency libraries built on Mechanical Sympathy: **Agrona** (off-heap buffers with `UnsafeBuffer`, lockless queues with 15×8=120-byte cache-line padding between producer `tail` and consumer `head` fields, ring buffers for IPC), **SBE** (Simple Binary Encoding — zero-copy binary codec for FIX protocol; copy-free native type mapping, allocation-free flyweight pattern, 4/8-byte word-aligned sequential access, compiled from XML schema via `sbe-tool`), and **Aeron** (OSI Layer 4 message transport over UDP/IPC/InfiniBand; 8 latency principles including garbage-free steady state, lock-free message path, non-blocking I/O, single-writer principle; memory-mapped `tmpfs` persistent log files, atomic lock-free tail appender, message coordinates `(streamId, sessionId, termId, termOffset)`, NAK-based retransmission, Active/Dirty/Clean file rotation).

### Chapter 15: Java 9 and the Future — JVM Evolution

The forward-looking chapter covering Java 9's performance improvements and the JVM's trajectory. **Segmented Code Cache** splits the monolithic code cache into 3 regions: nonmethod (never swept), profiled (C1 code), nonprofiled (C2 code) — shorter sweeper times and better code locality. **Compact Strings** halves ASCII string heap footprint by switching `char[]` (UTF-16) to `byte[] + byte coder` flag — controlled by `-XX:+CompactStrings`. **New String Concatenation** replaces `StringBuilder` bytecode desugaring with `invokedynamic` → `StringConcatFactory.makeConcatWithConstants()` — smaller bytecode, VM-optimizable strategy. **C2 SIMD improvements**: automatic vectorization, SuperWord loop unrolling, `CMovVD` on Intel AVX — the frontier of HotSpot optimization. **G1 made default** (replaces Parallel GC). **Java 10 JEPs**: JEP 307 (Parallel Full GC for G1 — replaces single-threaded full GC fallback), JEP 310 (AppCDS — shared class data archive across JVM instances for faster startup), JEP 312 (Thread-Local Handshakes — per-thread callbacks without global STW safepoint). **VarHandles (JEP 193)**: the official safe replacement for `sun.misc.Unsafe` CAS operations and memory ordering fences. **Project Valhalla & Value Types**: identity-less, header-less, reference-less struct-like memory layout (`Point3D[]` → contiguous doubles with no object headers), new opcodes `vdefault`/`withfield`. **Graal** (Java-written JIT compiler via JVMCI, supports partial escape analysis), **Truffle** (language framework using Futamura projection to auto-generate JITs for JRuby/Jython/R), **`jaotc`** (AOT compilation of `java.base` to ELF `.so`), **SubstrateVM** (single native executable in milliseconds). **Project Loom**: lightweight fibers/continuations scheduled on ForkJoinPool, replacing heavyweight OS threads.

---

## :material-book-open-variant: What You'll Master

- **Profiling Prerequisites** — Rule out GC and kernel causes before invoking an execution profiler
- **Safepoint Bias** — Why profilers miss tight loops; the `GetCallTrace` vs `AsyncGetCallTrace` fix
- **async-profiler** — CPU, allocation, wall-clock, and off-CPU profiles with < 2% overhead
- **JFR/JMC** — Production-safe event recording; TLAB allocation view; lock contention events
- **Flame Graphs** — CPU and allocation flame graphs; reading width = inclusive time
- **Logging Benchmarks** — JUL is 14× slower than Log4j 2.7; Log4j 2.6 achieves zero GC collections
- **Agrona Buffers** — `UnsafeBuffer`, cache-line-padded queues, ring buffers for IPC
- **SBE** — Zero-copy FIX binary codec; copy-free, allocation-free, word-aligned
- **Aeron Transport** — 8 latency principles; memory-mapped log; lock-free appender; NAKs
- **Compact Strings** — `byte[] + coder` halves ASCII string memory; Java 9 default
- **Segmented Code Cache** — 3-region split; shorter sweep; no cross-region evacuation
- **VarHandles** — JEP 193; replaces `Unsafe` for CAS and memory fences
- **Project Valhalla** — Value types: no headers, no references, flat array layout
- **Graal & Truffle** — Java-written JIT; Futamura projection for polyglot JVMs
- **Project Loom** — Fibers: lightweight cooperative threads on ForkJoinPool

---

## :material-map-marker-path: Optimizing Java — Book Context

| Reading Group | Chapters | Topics |
|:---:|---|---|
| Week 1 (Topic 1) | 1–5 | Foundations |
| Week 2 (Topic 2) | 6–8 | Garbage Collection |
| Week 3 (Topic 3) | 9–10 | Bytecode & JIT |
| Week 4 (Topic 4) | 11–12 | Language Performance & Concurrency |
| **Week 5 (This part)** | 13–15 | **Advanced JVM — Profiling, Messaging & Future** |

---

## :material-cogs: Key Internals to Understand

- **`AsyncGetCallTrace()` Signal Handler** — Uses `SIGPROF` to interrupt individual threads asynchronously; stack trace captured in OS signal handler without requiring a JVM safepoint; enables truly representative sampling of tight unrolled loops
- **JFR Thread-Local Buffers** — JFR events are written to per-thread memory buffers (no lock contention); periodically flushed to a shared repository; read by JMC without stopping any application thread
- **Agrona 15-Long Padding Structure** — Producer fields are separated from consumer fields by exactly 15 × 8 = 120 bytes of unused `long` fields spanning two 64-byte cache lines; prevents the CPU cache coherency protocol from invalidating both sides on every write
- **SBE Flyweight Pattern** — Encoder/decoder objects hold only a reference to the underlying `DirectBuffer` and an offset — they never allocate data; the "object" is really just a cursor into pre-allocated buffer memory
- **Aeron Single Writer Principle** — Each buffer region has exactly one writing thread; eliminates CAS contention for the common case; reads are unsynchronized volatile reads from the reader side
- **Thread-Local Handshakes (JEP 312)** — Enables the JVM to execute a callback on a *single* thread (e.g., for biased lock revocation or stack sampling) without stopping all threads; dramatically reduces safepoint pause frequency
- **Futamura Projection** — Truffle's theoretical foundation: a partial evaluator (Graal) applied to a language interpreter creates a compiler for that language; applying it again to the compiler creates a compiler compiler — the mechanism that makes Truffle polyglot JIT compilation work

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Read Chapter 13 — Profiling (Execution, Allocation, Heap Dumps)
- [x] Read Chapter 14 — High-Performance Logging and Messaging
- [x] Read Chapter 15 — Java 9 and the Future
- [x] Write Chapter 13 notes
- [x] Write Chapter 14 notes
- [x] Write Chapter 15 notes
- [ ] Phase 2 Capstone: Apply profiling + GC tuning + concurrency optimization to a real project

---

*Start Date: 2026-07-31 | Week 5 Completed: 2026-07-31 | Phase 2 Book Reading: Complete*
