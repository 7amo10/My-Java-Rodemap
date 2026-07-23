---
id: topic-2-garbage-collection-index
aliases: []
tags:
- java
- performance
- jvm
- garbage-collection
- gc
- optimizing-java
- phase-2
---

# :material-recycle: Topic 2: Garbage Collection Deep Dive

> **Book:** Optimizing Java — Practical Techniques for Improving JVM Application Performance
>
> **Authors:** Benjamin J. Evans, James Gough, Chris Newland (O'Reilly Media)
>
> **Part Covered:** Part II — GC in the JVM (Chapters 6–8)

---

## :material-notebook-outline: Topic Structure

| Document | Chapter | Coverage | Status |
|----------|---------|----------|--------|
| [:material-book-open-page-variant: Chapter 6 — GC Algorithms & Collectors](book-reading-ch6.md) | Ch 6 | Mark-and-sweep theory, GC roots, reference counting flaws, compaction, copying collectors, tri-color marking, stop-the-world vs concurrent, Serial/Parallel/CMS/G1/ZGC/Shenandoah overview | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 7 — Heap Analysis & GC Logging](book-reading-ch7.md) | Ch 7 | GC log format (unified `-Xlog:gc*`), GC log parsing, pause time analysis, allocation profiling, live set measurement, jmap/jcmd, heap histograms, GC event types | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 8 — GC Tuning & Collector Selection](book-reading-ch8.md) | Ch 8 | Collector selection criteria, G1GC tuning (`-XX:MaxGCPauseMillis`, regions, IHOP), ZGC/Shenandoah configuration, heap sizing, GC overhead budget, Epsilon GC | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Chapter 6: GC Fundamentals — Algorithms & Collectors

Covers the **theoretical foundations of garbage collection** that underpin all JVM collector implementations. The central insight: GC must solve two problems simultaneously — **liveness detection** (which objects are reachable?) and **space reclamation** (how to efficiently return memory?). Walks through why reference counting fails for cyclic structures and why tracing collection (following references from GC roots) is the correct approach. Introduces **mark-and-sweep** as the baseline algorithm: mark all reachable objects via graph traversal from roots (thread stacks, static fields, JNI references), then sweep through the heap reclaiming unmarked objects. Explains why naive mark-and-sweep creates **fragmentation** (interleaved live/dead objects), leading to compaction or copying alternatives. The **copying collector** uses two semi-spaces: live objects are copied to the "to" space, automatically compacting, but halves effective heap. The **tri-color marking** scheme (white/gray/black) enables concurrent GC without stopping the world — the write barrier maintains the invariant that no black object directly references a white object. Covers the generational hypothesis and why it's so powerful. Reviews all major HotSpot collectors: Serial (single-threaded, small heaps), Parallel GC (throughput-optimized, stop-the-world), CMS (low-latency, concurrent mark-sweep, fragmentation), G1GC (region-based, predictable pauses, default since JDK 9), ZGC (sub-millisecond pauses, colored pointers), and Shenandoah (Brooks pointers, concurrent compaction).

### Chapter 7: GC Log Analysis & Heap Profiling

The **practical measurement chapter** — how to observe what GC is actually doing in your application. Covers the unified JVM logging framework (`-Xlog:gc*`) introduced in JDK 9, which replaced the old `-verbose:gc`, `-XX:+PrintGCDetails`, and `-XX:+PrintGCDateStamps` flags. A single `-Xlog:gc*:file=gc.log:time,uptime,level,tags` captures everything needed. Explains how to parse GC log output: understanding concurrent vs stop-the-world phases, reading pause time reports, tracking heap occupancy before/after GC, and identifying the cause of each GC event. Introduces **GC log analysis tools**: GCEasy, GCViewer for visual parsing. Covers **allocation profiling** — identifying what is being allocated, in what quantities, and which application threads are responsible. Explains the difference between **allocation rate** (bytes/sec created) vs **live set** (bytes that survive GC). The live set is the critical constraint — if it grows without bound, you have a memory leak. Covers `jcmd`, `jmap -histo`, and heap dump analysis with Eclipse MAT for finding memory leaks. Explains how to measure **GC overhead** (% of CPU time spent in GC vs application) using `-XX:+PrintGCApplicationConcurrentTime` and `-XX:+PrintGCApplicationStoppedTime`.

### Chapter 8: GC Tuning — Collector Selection & Configuration

The **decision-making chapter** for production GC configuration. Establishes the fundamental **GC tuning triangle**: you can optimize for pause time (latency), throughput, or heap footprint — but you can't maximize all three simultaneously. Provides a collector selection flowchart: for batch/throughput workloads use Parallel GC; for mixed workloads with some latency requirement use G1GC; for strict low-latency (< 10ms pauses) use ZGC or Shenandoah. Deep-dives into **G1GC tuning**: the `MaxGCPauseMillis` target (G1 will try to meet this, but it's a hint not a guarantee), region sizing (`-XX:G1HeapRegionSize`), Initiating Heap Occupancy Percent (IHOP — when to start concurrent marking), and how the remembered set (RSet) and card table enable region-by-region collection. Explains why the G1 **humongous object** problem (objects > 50% of region size allocated directly in old gen) causes premature Full GCs and how to size regions to avoid it. Covers **ZGC configuration**: colored pointers (load barriers at pointer dereference), concurrent relocation, and the setting of `-XX:SoftMaxHeapSize`. Explains **Epsilon GC** (no-op GC used for allocation-rate profiling and ultra-short-lived processes). Gives rules of thumb: set `-Xms` = `-Xmx` in production to avoid heap resizing pauses; keep live set < 50% of heap; target GC overhead < 5% of CPU time.

---

## :material-book-open-variant: What You'll Master

- **GC Theory** — Mark-and-sweep, compaction, copying collectors, tri-color marking, write barriers
- **Generational Hypothesis** — Why young/old gen split works; eden, survivor spaces, tenuring
- **Collector Landscape** — Serial, Parallel, CMS, G1, ZGC, Shenandoah — when to use which
- **GC Log Analysis** — Parsing `-Xlog:gc*` output, identifying problematic GC patterns
- **Allocation Profiling** — Measuring allocation rate, finding hot allocation paths
- **Live Set Analysis** — Memory leak detection with heap histograms and MAT
- **G1GC Tuning** — Regions, IHOP, humongous objects, RSet overhead
- **Low-Latency GCs** — ZGC colored pointers, Shenandoah Brooks pointers, concurrent compaction
- **GC Overhead Budget** — Targeting < 5% CPU in GC; measuring GC pause impact

---

## :material-map-marker-path: Optimizing Java — Book Context

| Reading Group | Chapters | Topics |
|:---:|---|---|
| Week 1 (Topic 1) | 1–5 | Foundations: observables, JVM anatomy, hardware, testing, statistics |
| **Week 2 (This part)** | 6–8 | **Garbage Collection: algorithms, heap analysis, GC tuning** |
| Week 3 (Topic 3) | 9–10 | Bytecode & JIT: bytecode in depth, JIT compilation internals |
| Week 4 | 11–12 | Profiling & Concurrency: profilers, thread analysis, lock contention |
| Week 5 | 13–15 | Advanced JVM Optimization Techniques |

---

## :material-cogs: Key Internals to Understand

- **Card Table & Remembered Set** — How the JVM tracks cross-generational references without scanning the entire old gen on every young GC
- **Write Barrier Mechanics** — What happens at the bytecode level when you store a reference; how `oop_store` triggers card marking
- **G1 Region Anatomy** — How G1 divides heap into 1–32 MB regions, labels them (Eden/Survivor/Old/Humongous), and evacuates the lowest-liveness regions
- **SATB vs Incremental Update** — Two concurrent marking write barrier strategies; G1 uses SATB (Snapshot At The Beginning) to handle concurrent object mutations
- **ZGC Colored Pointer** — How ZGC embeds metadata bits (marked0/marked1/remapped/finalizable) in the unused upper bits of 64-bit pointers, enabling concurrent relocation with a load barrier
- **Shenandoah Brooks Pointer** — Forwarding pointer at object header for concurrent compaction; every object has an extra word pointing to its current location

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Read Chapter 6 — GC Algorithms & Collectors
- [x] Read Chapter 7 — Heap Analysis & GC Logging
- [x] Read Chapter 8 — GC Tuning & Collector Selection
- [x] Write Chapter 6 notes
- [x] Write Chapter 7 notes
- [x] Write Chapter 8 notes
- [ ] Week 3: Read Chapters 9–10 (Bytecode & JIT)

---

*Start Date: 2026-07-23 | Week 2 Completed: 2026-07-23*
