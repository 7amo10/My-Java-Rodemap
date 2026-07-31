---
id: topic-4-profiling-concurrency-index
aliases: []
tags:
- java
- performance
- jvm
- collections
- concurrency
- method-handles
- cas
- fork-join
- optimizing-java
- phase-2
---

# :material-chart-gantt: Topic 4: Language Performance & Concurrency

> **Book:** Optimizing Java — Practical Techniques for Improving JVM Application Performance
>
> **Authors:** Benjamin J. Evans, James Gough, Chris Newland (O'Reilly Media)
>
> **Part Covered:** Part IV — Performance Tooling (Chapters 11–12)

---

## :material-notebook-outline: Topic Structure

| Document | Chapter | Coverage | Status |
|----------|---------|----------|--------|
| [:material-book-open-page-variant: Chapter 11 — Java Language Performance Techniques](book-reading-ch11.md) | Ch 11 | Collection internals (ArrayList vs LinkedList, HashMap bucket layout, treeifying, TreeMap), domain object memory leak diagnosis, finalization lifecycle & flaws, `try-with-resources`, Method Handles vs reflection | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 12 — Concurrent Performance Techniques](book-reading-ch12.md) | Ch 12 | Amdahl's Law, JMM (happens-before, release-before-acquire), `sun.misc.Unsafe`, CAS & lock-free atomics, spinlocks, `ReentrantLock`/`ReadWriteLock`/`Semaphore`, `ConcurrentHashMap`, `ForkJoinPool` work-stealing, parallel streams, LMAX Disruptor, Actor model | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Chapter 11: Java Language Performance Techniques — Collections, Memory & Method Handles

A practical deep-dive into the **performance characteristics of core Java language constructs** — the data structures and APIs you use every day, but rarely understand at the memory and allocation level. Opens with the fundamental tension: Java stores reference-type fields as heap pointers (unlike C++ which can embed objects inline), which means every collection element is an indirection — and Object Layout / Project Valhalla are the long-term answer. Covers the two collection categories: **sequential containers** (index-based: `ArrayList`, `LinkedList`) and **associative containers** (hash/comparison-based: `HashMap`, `TreeMap`). `ArrayList` is backed by a contiguous array (pointer-bump allocation, O(1) indexed access, O(n) insert-at-head) — key insight: pre-size it with `new ArrayList<>(expectedSize)` or `ensureCapacity()` to avoid costly resizing (copy entire backing array). `LinkedList` has O(1) append/head-insert but O(n) indexed access — JMH benchmarks show `beginLinkedList` at 559 ops/ms vs `beginArrayList` at 3.4 ops/ms for head insertion, but `accessArrayList` at 269,568 ops/ms vs `accessLinkedList` at 0.86 ops/ms for indexed access. `HashMap` internals: bucket array + linked `Node` chains; spread function applies Shannon's Strict Avalanche Criteria; **default capacity 16, load factor 0.75** — set `initialCapacity = maxElements / loadFactor` to avoid rehashing. When a bucket reaches `TREEIFY_THRESHOLD`, chains convert to red-black `TreeNode`s (O(log n) worst case but double memory per node). `TreeMap` for sorted/range queries. Diagnoses domain object memory leaks using `jmap -histo`: healthy heaps show only JDK types (`char[]`, `String`, collections) in the top entries; domain objects in top 30 = memory leak or premature tenuring. **Finalization deep-dive**: unreachable objects with `finalize()` go to `java.lang.ref.Finalizer` queue, get run by the dedicated finalizer thread, then survive one extra GC cycle — nondeterministic, exception-swallowing, resurrection-prone — deprecated Java 9. **`try-with-resources`**: purely compile-time sugar via `javac`, deterministic block-scoped resource release, zero runtime overhead. **Method Handles** (`java.lang.invoke`): strongly-typed, signature-polymorphic direct method references via `MethodHandles.lookup().findVirtual()` — JIT-compilable with no boxing, vs reflection's `Method.invoke(Object[])` which boxes all args.

### Chapter 12: Concurrent Performance Techniques — JMM, CAS, Locks & Fork/Join

The comprehensive chapter on **making concurrent Java code fast and correct**. Starts from Herb Sutter's "The Free Lunch Is Over" (2005): CPU clocks plateaued at ~3 GHz; scaling now requires parallelism bounded by **Amdahl's Law** (`T(N) = S + (T−S)/N` — 5% serial code caps max speedup at 20×). The **Java Memory Model (JMM)** is a *weak* memory model (not all cores see changes simultaneously) defined via happens-before chains: `synchronized` creates Release-Before-Acquire, `volatile` creates synchronizes-with. `volatile` guarantees *visibility* but NOT atomicity — `i++` is three bytecodes (`getfield`, `iadd`, `putfield`) and races even with `volatile`. **`sun.misc.Unsafe`**: the foundation of all lock-free JDK internals — `allocateInstance()`, `compareAndSwapInt()`, field offset arithmetic. **CAS (Compare-And-Swap)**: single atomic CPU instruction; `AtomicInteger` wraps it in a retry loop; lock-free and deadlock-immune but degrades under high contention. **Spinlocks**: busy-wait via x86 `xchg`/`test`/`jnz` — eliminates context-switch overhead for ultra-short lock hold times, used by LMAX Disruptor. **`ReentrantLock`** (via AQS + `LockSupport.park/unpark`): CAS-based uncontended acquisition, interruptible, timed, fair/unfair modes. **`ReentrantReadWriteLock`**: multiple concurrent readers; exclusive writer — ideal for read-heavy caches. **`Semaphore`**: manages fixed permits; binary semaphore acts as mutex but any thread can release. **`ConcurrentHashMap`**: snapshot iterators, segment-level write locking, non-blocking reads. **`ForkJoinPool`**: work-stealing deque algorithm — idle threads steal from busy threads' tails; powers parallel streams via `commonPool()` (size = `availableProcessors() - 1`; beware NUMA/hyperthread anomalies). **LMAX Disruptor**: lock-free ring buffer using spinlocks and `volatile` variables; 4.87–8.70× faster than `ArrayBlockingQueue` across single/multi producer-consumer patterns. **Actor model (Akka)**: isolated state + async immutable messages = no shared state, no deadlocks.

---

## :material-book-open-variant: What You'll Master

- **Collection Performance Internals** — ArrayList resizing, LinkedList node cost, HashMap bucket/treeify mechanics
- **HashMap Tuning** — `initialCapacity = maxElements / loadFactor` prevents rehashing; treeifying = double memory
- **TreeMap vs HashMap** — When sorted iteration, range queries, or subset operations justify O(log n)
- **Memory Leak Diagnosis** — `jmap -histo` signatures, domain object tenuring, GC mark time inflation
- **Finalization Lifecycle** — Why `finalize()` is deprecated; the one-extra-GC-cycle survivor problem
- **`try-with-resources`** — Compile-time only; deterministic; zero GC cost; suppressed exceptions
- **Method Handles** — Signature polymorphism, no boxing, JIT-friendly vs reflection's `Object[]` overhead
- **Amdahl's Law (Applied)** — GC STW pauses and lock contention contribute to the serial fraction
- **JMM Guarantees** — Happens-before, synchronizes-with, release-before-acquire; why `volatile` ≠ atomic
- **CAS Mechanics** — AtomicInteger internals; lock-free but contention-sensitive
- **`ReentrantLock` Hierarchy** — AQS, `LockSupport`, fair vs unfair, `tryLock()`, timed
- **ForkJoinPool Work-Stealing** — Task subdivision, `commonPool()` sizing gotchas
- **LMAX Disruptor** — Why spinlocks + ring buffer beats `ArrayBlockingQueue` by 5–9×

---

## :material-map-marker-path: Optimizing Java — Book Context

| Reading Group | Chapters | Topics |
|:---:|---|---|
| Week 1 (Topic 1) | 1–5 | Foundations |
| Week 2 (Topic 2) | 6–8 | Garbage Collection |
| Week 3 (Topic 3) | 9–10 | Bytecode & JIT |
| **Week 4 (This part)** | 11–12 | **Language Performance & Concurrency** |
| Week 5 (Topic 5) | 13–15 | Advanced JVM Optimization Techniques |

---

## :material-cogs: Key Internals to Understand

- **ArrayList Growth Factor** — Each resize doubles capacity; `Arrays.copyOf()` internally; total copy cost is amortized O(1) but individual resize is O(n) — avoid with `ensureCapacity()` before bulk insertion
- **HashMap Treeify Threshold** — `TREEIFY_THRESHOLD = 8`; treeifying is a warning sign of bad hash distribution; `TreeNode` memory cost = 2× regular `Node`; re-evaluate hash function or increase initial capacity
- **`HashMap.indexFor()` Spread Function** — XOR with right-shifted self `h ^ (h >>> 16)` ensures high bits influence bucket selection when array length is small; satisfies Shannon's Strict Avalanche Criteria
- **Finalizer Thread vs GC Pauses** — Finalizer thread is a single-threaded serial queue drainer; a backlogged finalization queue causes objects to accumulate in old gen and inflates GC pause times
- **AQS (AbstractQueuedSynchronizer)** — The internal base class for `ReentrantLock`, `Semaphore`, `CountDownLatch`, `CyclicBarrier` — uses a single `volatile int state` field + CLH (Craig-Landin-Hagersten) wait queue of parked threads
- **`LongAdder` vs `AtomicLong`** — `LongAdder` uses a `Cell[]` array where each cell is cache-line-padded (`@Contended`); threads write to different cells (by thread ID hash), then `sum()` adds all cells — O(cells) on read, zero CAS contention on writes under high parallelism

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Read Chapter 11 — Java Language Performance Techniques
- [x] Read Chapter 12 — Concurrent Performance Techniques
- [x] Write Chapter 11 notes
- [x] Write Chapter 12 notes
- [ ] Week 5: Read Chapters 13–15 (Advanced JVM)

---

*Start Date: 2026-07-31 | Week 4 Completed: 2026-07-31*
