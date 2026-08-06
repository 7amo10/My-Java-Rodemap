---
id: phase-2-helix-jvm-engine
aliases: []
tags:
- java
- jvm-internals
- bytecode
- classloader
- jit
- gc
- profiling
- phase-2
- project
---

# :material-engine: Helix JVM Engine & Profiler

> **Phase:** Phase 2 — Java Internals & Performance
>
> **Repo:** [:material-github: 7amo10/helix-jvm-engine](https://github.com/7amo10/helix-jvm-engine)
>
> **Stack:** Java 17 · ByteBuddy · ASM · JFR · async-profiler · JOL · Caffeine · Maven Multi-Module
>
> **Build:** ![passing](https://img.shields.io/badge/build-passing-brightgreen.svg) ![JDK17+](https://img.shields.io/badge/JDK-17%2B-blue.svg) ![Coverage](https://img.shields.io/badge/Coverage-80%25-green.svg)

---

## :material-lightbulb: The Problem

Modern enterprise systems frequently need to evaluate **dynamic business logic** — fraud detection rules, pricing strategies, compliance checks — that must be deployable *without* restarting the application or recompiling source code. The naive approaches all have fundamental flaws:

| Approach | Problem |
|----------|---------|
| **Hardcoded `if/else`** | Rules require full recompile + redeploy cycle |
| **Groovy/script evaluation** | Interpreter overhead — no JIT, no static type checking, no inlining |
| **Reflection-based dispatch** | `Method.invoke(Object[])` — boxing overhead, no JIT optimization, not inlineable |
| **Rule engines (Drools)** | Massive dependency tree, opaque internals, black-box execution, difficult to profile |

Beyond the execution problem, there is a second, deeper problem: **JVM internals are invisible by default**. A developer might see "slow" performance but have no actionable visibility into whether the bottleneck is:

- GC pressure from dynamic class allocation
- JIT compilation failing to optimize rule evaluation methods
- ClassLoader Metaspace leaks from improperly closed loaders
- Cache miss cascades through L1→L2→L3 reference tiers
- Thread contention in async execution pipelines

---

## :material-check-circle: The Solution — Helix

Helix is a **production-grade JVM scripting engine** that compiles JSON-defined business rules directly into JVM bytecode at runtime, achieving near-native execution speeds. It is simultaneously an **advanced JVM observability platform** — exposing every internal JVM mechanism (classloading, JIT tiers, GC events, object memory layout) through a reactive terminal dashboard and JFR event streams.

```mermaid
flowchart LR
    subgraph INPUT["Rule Definition"]
        JSON["fraud-detection.json<br/>(JSON rule schema)"]
    end

    subgraph HELIX["Helix Engine Pipeline"]
        PARSE["JSON AST Parser<br/>(ExpressionNode tree)"]
        OPT["AST Optimizer<br/>(constant folding<br/>dead code elimination)"]
        GEN["Bytecode Generator<br/>ByteBuddy (high-level)<br/>or ASM (low-level)"]
        CL["ClassLoader Manager<br/>ISOLATED / SHARED<br/>/ HIERARCHICAL mode"]
        CACHE["TieredRuleCache<br/>L1 Strong -> L2 Soft<br/>-> L3 Weak"]
        EXEC["Executor<br/>Sync (0B alloc)<br/>Async (184B/call)<br/>Batch (450K ops/sec)"]
    end

    subgraph PROFILER["Observability Layer"]
        JIT["JIT Monitor<br/>C1 to C2 tier tracking"]
        JFR["JFR Event Stream<br/>custom events + TLAB sampling"]
        TUI["Lanterna TUI Dashboard<br/>live metrics + flame graphs"]
    end

    JSON --> PARSE
    PARSE --> OPT
    OPT --> GEN
    GEN --> CL
    CL --> CACHE
    CACHE --> EXEC
    EXEC --> JIT
    EXEC --> JFR
    JFR --> TUI

    style HELIX fill:#1e3a5f,color:#fff
    style PROFILER fill:#2d5a27,color:#fff
    style INPUT fill:#4a3728,color:#fff
```

**Key achievement**: Rules compile in ~1.7ms (ASM) or ~5.1ms (ByteBuddy) and execute at **> 120,000 ops/sec** with the `SyncExecutor` — comparable to hand-written Java — because the JIT can inline the generated `CompiledRule::eval()` method directly into the call site.

---

## :material-sitemap: Internal Architecture

### Module Topology

```mermaid
flowchart TD
    API["engine-api<br/>(Pure interfaces - zero deps)<br/>Rule, RuleEngine,<br/>ExecutionContext, Profiler"]
    CORE["engine-core<br/>(Main implementation)<br/>Parser - ByteBuddy/ASM generators<br/>ClassLoaderManager - Executors<br/>TieredRuleCache - EventBus"]
    PROFILER["engine-profiler<br/>(JIT/GC monitoring)<br/>JIT tier tracker - GC analyzer<br/>JFR custom events - async-profiler<br/>Lanterna TUI dashboard"]
    AGENT["engine-agent<br/>(Fat-JAR Java Agent)<br/>ASM Transformer - JOL Inspector<br/>JMX MBeans - AllocationTracker"]
    EXPERIMENTS["engine-experiments<br/>(Benchmarks and JVM scenarios)<br/>JMH suites - MetaspaceLeak<br/>JitCompilation - GcStress<br/>Safepoint - ObjectLayout"]

    API --> CORE
    CORE --> PROFILER
    API --> AGENT
    CORE --> EXPERIMENTS
    PROFILER --> EXPERIMENTS
    AGENT --> EXPERIMENTS

    style API fill:#3d59a1,color:#fff
    style CORE fill:#4a6fa5,color:#fff
    style PROFILER fill:#4caf7c,color:#fff
    style AGENT fill:#e8933a,color:#fff
    style EXPERIMENTS fill:#7b68ae,color:#fff
```

### End-to-End Data Flow (Rule Execution)

```mermaid
flowchart TD
    %% Define the nodes (participants)
    CLI[Helix CLI]
    PARSER[JSON AST Parser]
    OPT[AST Optimizer]
    GEN[ByteBuddy or ASM Generator]
    CLM[ClassLoader Manager]
    CACHE[TieredRuleCache]
    EXEC[Executor]
    JIT[JIT Monitor via JFR]

    %% Define the flow (messages)
    CLI -->|Load fraud detection JSON| PARSER
    PARSER -->|Build ExpressionNode AST| OPT
    
    %% Self-loops
    OPT -->|Constant folding & dead code elimination| OPT
    
    OPT -->|Send optimized AST| GEN
    GEN -->|Emit generated bytecode array| CLM
    
    CLM -->|Register class in isolated RuleClassLoader| CLM
    
    CLM -->|Promote entry to L1 Strong cache| CACHE
    CACHE -.->|Return CompiledRule handle| CLI
    
    CLI -->|Execute rule with context| EXEC
    EXEC -->|Perform L1 lookup| CACHE
    CACHE -.->|Return CompiledRule reference| EXEC
    
    EXEC -->|Evaluate rule at C2 Tier 4| EXEC
    
    EXEC -->|Emit JFR CompilationEvent| JIT
    JIT -.->|Return ExecutionResult| CLI
```

---

## :material-layers: Core Subsystems Deep Dive

### 1. Dual Bytecode Generation Engines

Helix implements two distinct bytecode generators, each optimized for different scenarios:

```mermaid
flowchart LR
    subgraph BB["ByteBuddy Generator<br/>(High-Level API)"]
        BB1["Fluent API over AST<br/>subclass(CompiledRule)<br/>method delegation<br/>Annotation processing"]
        BB2["Output: ~1,180 bytes/rule<br/>Compile time: ~5.1ms<br/>Type-safe. Ideal for<br/>dynamic runtime generation"]
    end

    subgraph ASM["ASM Generator<br/>(Low-Level Opcodes)"]
        ASM1["Direct ClassWriter<br/>Manual stack manipulation<br/>Direct opcode emission<br/>Constant pool control"]
        ASM2["Output: ~640 bytes/rule<br/>Compile time: ~1.7ms<br/>3x smaller. Ideal for<br/>bulk over 1,000 rules/sec"]
    end

    AST["Optimized AST"] --> BB1
    AST --> ASM1

    style BB fill:#3d59a1,color:#fff
    style ASM fill:#4caf7c,color:#fff
```

**AST Optimization pass** (runs before both generators):

| Optimization | What it Does |
|---|---|
| **Constant Folding** | `1 + 2 > amount` → `3 > amount` at compile time |
| **Dead Code Elimination** | Removes unreachable branches from the AST |
| **Inline Constants** | Replaces known-constant variable reads with literal opcodes |
| **Method Inlining Hints** | Keeps generated methods `< 35 bytes` to stay within `MaxInlineSize` |

### 2. Multi-Tenant ClassLoader Isolation

One of the most JVM-specific features of Helix — each rule can run in complete Metaspace isolation:

```mermaid
flowchart TD
    BSL["Bootstrap ClassLoader<br/>(JVM core - java.lang, java.util)"]
    PSL["Platform/Extension ClassLoader<br/>(JDK modules)"]
    APPCL["Application ClassLoader<br/>(Helix engine JARs)"]
    SUCL["SharedUtilityClassLoader<br/>(shared rule utilities, deps)<br/>Singleton - loaded once"]

    R1["RuleClassLoader_fraud_v1<br/>(isolated)"]
    R2["RuleClassLoader_pricing_v2<br/>(isolated)"]
    RN["RuleClassLoader_..._vN<br/>(isolated)"]

    BSL --> PSL
    PSL --> APPCL
    APPCL --> SUCL
    SUCL --> R1
    SUCL --> R2
    SUCL --> RN

    style SUCL fill:#3d59a1,color:#fff
    style R1 fill:#4a6fa5,color:#fff
    style R2 fill:#4a6fa5,color:#fff
    style RN fill:#4a6fa5,color:#fff
```

**Three isolation modes** controlled by `ClassLoaderManager`:

| Mode | Memory | Use Case | Metaspace Impact |
|------|--------|---------|-----------------|
| `ISOLATED` | Highest | One `RuleClassLoader` per rule — complete isolation | Proportional to rule count; must close loaders |
| `SHARED` | Medium | Multiple rules share one loader | Lower; rules cannot be unloaded independently |
| `HIERARCHICAL` | Balanced | Rule families share parent loaders (e.g., all `fraud/*` rules) | Optimized per family |

!!! important "The Metaspace Leak Experiment"
    Helix includes an intentional `MetaspaceLeakExperiment` that loads 5,000 rules without closing their loaders — demonstrating `OutOfMemoryError: Metaspace`. The `demonstrateCleanup()` counterpart uses `try-with-resources` on `AutoCloseable` loaders, keeping Metaspace stable by triggering `loader.close()` → GC eligibility for the loader and all its defined classes.

### 3. Tiered Reference Cache (L1/L2/L3)

Helix maps Java's three reference strengths directly to a 3-tier caching hierarchy:

```mermaid
flowchart TD
    REQ["Incoming rule.execute() call"]

    L1["L1 - Strong Reference Cache<br/>(Caffeine in-memory)<br/>Latency: &lt; 12 ns<br/>Capacity: peakRules * 1.25<br/>Footprint: 1,248 bytes/entry<br/>GC: NEVER collected"]

    L2["L2 - Soft Reference Cache<br/>Latency: &lt; 45 ns<br/>Footprint: 1,280 bytes/entry<br/>GC: Freed when heap &gt; 95%<br/>(-XX:SoftRefLRUPolicyMSPerMB=1000)"]

    L3["L3 - Weak Reference Cache<br/>Latency: &lt; 80 ns<br/>Footprint: 1,272 bytes/entry<br/>GC: Freed on ANY minor GC"]

    COMPILE["Full recompile pipeline<br/>(JSON -> AST -> bytecode -> defineClass)<br/>Latency: 1.7 - 5.1 ms"]

    RETURN["Return CompiledRule<br/>to executor"]

    REQ --> L1
    L1 -->|"Hit"| RETURN
    L1 -->|"Miss"| L2
    L2 -->|"Hit"| RETURN
    L2 -->|"Miss"| L3
    L3 -->|"Hit"| RETURN
    L3 -->|"Miss"| COMPILE
    COMPILE --> L1

    style L1 fill:#4caf7c,color:#fff
    style L2 fill:#e8933a,color:#fff
    style L3 fill:#dc5c59,color:#fff
    style COMPILE fill:#7b68ae,color:#fff
```

### 4. Executor Subsystem — Abstraction Cost Analysis

Helix exposes three executor strategies, each with measured allocation and latency costs (via JOL + JMH):

| Executor | Allocation / Call | Latency | Throughput | When to Use |
|----------|------------------|---------|-----------|-------------|
| `SyncExecutor` | **0 bytes** | ~8 ns | **125,000 ops/sec** | High-frequency single-thread hot loops |
| `AsyncExecutor` | **184 bytes** (CompletableFuture 48B + Task 64B + queue entry 72B) | ~1.2 µs | 85,000 ops/sec | Non-blocking async web pipelines |
| `BatchExecutor` | **64 bytes** (spliterator chunk pointers) | ~2.4 µs | **450,000 ops/sec** (batch ≥ 100) | Multi-core parallel batch processing |

### 5. Profiler Module — JVM Observability

```mermaid
flowchart LR
    subgraph PROFILER["engine-profiler Subsystems"]
        JIT["JIT Compilation Monitor<br/>Tracks C1 to C2 tier transitions<br/>Inlining decisions<br/>Deoptimizations"]
        GC["GC Analyzer<br/>GC event parsing<br/>Pause time histograms<br/>Allocation rate tracking"]
        ASYNC["async-profiler Integration<br/>CPU flame graphs<br/>Allocation flame graphs<br/>Lock contention profiles"]
        JFR["JFR Recording Manager<br/>Custom JFR events<br/>TLAB allocation sampling<br/>Rule execution events"]
        TUI["Lanterna TUI Dashboard<br/>Live metrics (ncurses-style)<br/>Real-time flame graph view<br/>JIT + GC + cache dials"]
    end

    style JIT fill:#3d59a1,color:#fff
    style GC fill:#4a6fa5,color:#fff
    style ASYNC fill:#4caf7c,color:#fff
    style JFR fill:#e8933a,color:#fff
    style TUI fill:#7b68ae,color:#fff
```

### 6. Java Agent — Instrumentation & Memory Layout

The `engine-agent` module builds as a **fat JAR** (shades ASM + JOL to avoid classpath conflicts) and attaches to the JVM as a `-javaagent`:

| Component | What It Does |
|-----------|-------------|
| `RuleClassTransformer` (ASM) | Intercepts `defineClass()` calls — logs bytecode size, method count, instruction histogram |
| `AllocationTracker` | JVMTI-based allocation sampling — tracks bytes allocated per rule execution path |
| `ObjectLayoutInspector` (JOL) | Prints exact object header layout with field offsets: `ExecutionResult` = 32B shallow, 64B deep |
| `EngineControlMBean` (JMX) | Runtime controls: flush cache, trigger GC, adjust log level without restart |

**JOL measurements (64-bit JVM, `-XX:+UseCompressedOops`):**

```
64-Bit Object Header: [MarkWord 8B][KlassPointer 4B][Padding 4B] = 16B total

ExecutionResult:  32B shallow, 64B deep    (header + success + result + error + nanos)
ExecutionContext:  24B shallow, 256B deep   (header + variables Map ref)
CacheKey:         32B shallow, 184B deep    (header + ruleName + version + schema refs)
RuleClassLoader:  112B shallow, 12.4KB deep (header + ClassLoader native vectors + class defs)
CompiledRuleClass: 640B (ASM) / 1,180B (ByteBuddy) — lives in Metaspace
```

---

## :material-book-open-variant: JVM Concepts Applied

This project is a hands-on implementation lab for every major topic covered in the Phase 2 notes:

| Phase 2 Topic | Applied in Helix |
|---|---|
| **GC Algorithms & Tuning** | G1GC production config (`-XX:MaxGCPauseMillis=200`), L2/L3 reference strength cache tiers, `SoftRefLRUPolicyMSPerMB` tuning, GcStressExperiment |
| **Bytecode & JIT** | ByteBuddy + ASM bytecode generation, `< 35 byte` method size targeting for `MaxInlineSize`, tiered compilation tracking (Tier 0→4), `FreqInlineSize=325` for hot `eval()` path |
| **ClassLoader Internals** | Custom `URLClassLoader` subclass, Metaspace management, isolated/shared/hierarchical modes, class unloading verification with `WeakReference` |
| **Profiling (async-profiler, JFR)** | `AsyncGetCallTrace` via async-profiler, custom JFR events, TLAB allocation sampling, flame graph generation, 2-prerequisite profiling discipline |
| **Language Performance** | JOL object layout analysis, `ConcurrentHashMap` for `activeLoaders`, `Caffeine` cache for L1 (strong refs), `try-with-resources` on `AutoCloseable` classloaders |
| **Concurrency** | `BatchExecutor` on `ForkJoinPool`, `AsyncExecutor` on `CompletableFuture`, `ConcurrentHashMap<String, RuleClassLoader>` for thread-safe loader registry |
| **Logging & Observability** | SLF4J + Logback, Micrometer metrics, JMX MBeans for runtime control, AppCDS for sub-100ms startup |

---

## :material-chart-line: Performance Benchmarks (JMH)

| Benchmark | Score | Unit | Notes |
|-----------|-------|------|-------|
| `CompilationBenchmark.asmCompile` | **~1.7 ms** | ms/rule | ASM direct opcode generation |
| `CompilationBenchmark.byteBuddyCompile` | **~5.1 ms** | ms/rule | ByteBuddy high-level API |
| `ExecutionBenchmark.syncExecution` (C2 hot) | **~8 ns** | ns/op | After 15,000 invocations → C2 Tier 4 |
| `ExecutionBenchmark.syncExecution` (cold) | **~250 ns** | ns/op | Interpreter Tier 0 |
| `CacheBenchmark.l1Hit` | **< 12 ns** | ns/op | Strong ref, Caffeine |
| `CacheBenchmark.l2Hit` | **< 45 ns** | ns/op | Soft ref |
| `CacheBenchmark.l3Hit` | **< 80 ns** | ns/op | Weak ref |
| `BatchExecutor` throughput (N≥100) | **> 450,000** | ops/sec | ForkJoinPool parallel |

---

## :material-cogs: JVM Tuning Matrix

Production-recommended JVM flags for running Helix at peak efficiency:

```bash
java -server \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:InitiatingHeapOccupancyPercent=45 \
     -XX:MaxMetaspaceSize=256m \
     -XX:SoftRefLRUPolicyMSPerMB=1000 \
     -XX:CompileThreshold=10000 \
     -XX:MaxInlineSize=35 \
     -XX:FreqInlineSize=325 \
     -XX:+TieredCompilation \
     -XX:+UseCompressedOops \
     -jar engine-core-*.jar execute --rule fraud-detection.json
```

| Flag | Value | Reason |
|------|-------|--------|
| `-XX:+UseG1GC` | G1 | Predictable pauses for mixed short-lived + long-lived class data |
| `-XX:MaxGCPauseMillis=200` | 200ms | Caps STW under peak throughput |
| `-XX:MaxMetaspaceSize=256m` | 256m | Prevents ClassLoader leak from causing host OOM |
| `-XX:SoftRefLRUPolicyMSPerMB=1000` | 1000 | L2 soft cache survives 1s/MB of free heap |
| `-XX:CompileThreshold=10000` | 10000 | Triggers C2 aggressive compilation + loop unrolling |
| `-XX:FreqInlineSize=325` | 325 bytes | Allows hot `eval()` method to be inlined at call sites |

---

## :material-flag-checkered: Sprint Delivery Plan

Helix was designed and built in a **7-sprint, 7-day** schedule:

| Sprint | Day | Milestone | Deliverable |
|--------|-----|-----------|-------------|
| Sprint 1 | Day 1 | **M1: Foundation** | Maven multi-module setup, all API interfaces |
| Sprint 2 | Day 2 | **M2: Compilation Pipeline** | JSON parser, AST builder, ByteBuddy generator |
| Sprint 3 | Day 3 | **M3: ClassLoaders & Execution** | ClassLoaderManager, 3 executors, TieredRuleCache |
| Sprint 4 | Day 4 | **M4: Agent & Instrumentation** | ASM transformer, JOL inspector, JMX MBeans |
| Sprint 5 | Day 5 | **M5: Profiler & Observability** | JIT monitor, GC analyzer, JFR events, Lanterna TUI |
| Sprint 6 | Day 6 | **M6: Experiments & Benchmarks** | JMH suites, MetaspaceLeak, JIT, GC, SafepointExperiment |
| Sprint 7 | Day 7 | **M7: Production Readiness** | E2E tests, CI/CD, AppCDS, full documentation |

---

## :material-link: References & Further Reading

### :material-engine: Bytecode Generation

| Resource | Why It Matters |
|----------|---------------|
| [ByteBuddy — Official Tutorial](https://bytebuddy.net/#/tutorial) | The canonical guide to ByteBuddy's fluent API: subclassing, method delegation, class naming strategies — exactly what Helix uses for high-level rule generation |
| [ASM User Guide (PDF)](https://asm.ow2.io/asm4-guide.pdf) | Low-level reference for `ClassWriter`, `MethodVisitor`, opcode emission, and `COMPUTE_FRAMES` — the foundation of Helix's ASM generator |
| [JVM Specification — Chapter 4 & 6](https://docs.oracle.com/javase/specs/jvms/se17/html/jvms-4.html) | Official bytecode format, constant pool structure, and instruction set — essential for understanding what ASM emits |
| [Javassist Documentation](https://www.javassist.org/tutorial/tutorial.html) | Alternative bytecode manipulation library; useful for comparing API approaches against ByteBuddy |

### :material-chip: ClassLoaders & Metaspace

| Resource | Why It Matters |
|----------|---------------|
| [Understanding Java Class Loaders — Baeldung](https://www.baeldung.com/java-classloaders) | Clear walkthrough of the Bootstrap → Platform → App loader delegation chain that Helix's hierarchy sits on top of |
| [Java ClassLoader API — JDK 17 Javadoc](https://docs.oracle.com/en/java/docs/api/java.base/java/lang/ClassLoader.html) | Precise semantics of `defineClass()`, `loadClass()`, and `close()` — methods Helix's `RuleClassLoader` directly overrides |
| [Metaspace In OpenJDK — Jon Masamitsu (Oracle Blog)](https://stuefe.de/posts/metaspace/what-is-metaspace/) | Deep dive into Metaspace allocators, class metadata lifecycle, and the conditions under which classes are unloaded — directly explains Helix's leak experiments |
| [JEP 122: Remove Permanent Generation](https://openjdk.org/jeps/122) | Why PermGen was replaced by Metaspace and what that means for dynamic class generation tools like Helix |

### :material-fire: JIT Compilation & Performance

| Resource | Why It Matters |
|----------|---------------|
| [HotSpot JIT Compilation — OpenJDK Wiki](https://wiki.openjdk.org/display/HotSpot/PerformanceTacticIndex) | Tiered compilation (C1/C2), inlining heuristics, and deoptimization — the exact mechanisms Helix's JIT monitor tracks |
| [JVM Anatomy Quarks — Aleksey Shipilёv](https://shipilev.net/jvm/anatomy-quarks/) | 30+ micro-articles on JVM internals: inlining, safepoints, object headers, escape analysis — essential companion reading for Helix's profiler module |
| [JMH GitHub & Samples](https://github.com/openjdk/jmh) | The benchmarking harness Helix uses for `CompilationBenchmark`, `ExecutionBenchmark`, and `CacheBenchmark` — includes pitfall avoidance (dead code, constant folding) |
| [Escape Analysis in HotSpot](https://www.ibm.com/docs/en/sdk-java-technology/8?topic=debugging-escape-analysis) | How HotSpot's escape analysis enables stack allocation — relevant to `SyncExecutor`'s zero-allocation claim |

### :material-memory: GC & Reference Strengths

| Resource | Why It Matters |
|----------|---------------|
| [G1 GC Tuning Guide — Oracle](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html) | Official flags reference for `-XX:MaxGCPauseMillis`, IHOP, and region sizing — the basis for Helix's production JVM tuning matrix |
| [Java Reference Objects — Baeldung](https://www.baeldung.com/java-soft-references) | `SoftReference`, `WeakReference`, and `PhantomReference` semantics — the exact mechanism behind Helix's L2 and L3 cache tiers |
| [GC Log Analysis with GCEasy](https://gceasy.io/) | Online GC log analyzer — useful for visualising Helix's `GcStressExperiment` outputs |
| [ZGC & Shenandoah — OpenJDK](https://openjdk.org/projects/zgc/) | Low-latency GC alternatives; understanding their trade-offs contextualizes why Helix defaults to G1 |

### :material-chart-areaspline: Profiling & Observability

| Resource | Why It Matters |
|----------|---------------|
| [async-profiler GitHub](https://github.com/async-profiler/async-profiler) | The profiler Helix integrates — `AsyncGetCallTrace` API, CPU/allocation/wall-clock modes, flame graph generation |
| [Java Flight Recorder — JDK 17 Guide](https://docs.oracle.com/en/java/javase/17/jfapi/why-use-jfr-api.html) | How to write custom `@Name` / `@Label` JFR event classes — exactly what Helix's `CustomJfrEvents` does |
| [JOL — Java Object Layout](https://github.com/openjdk/jol) | The library behind Helix's `ObjectLayoutInspector` — prints exact field offsets, header sizes, and padding bytes |
| [Brendan Gregg — Flame Graphs](https://www.brendangregg.com/flamegraphs.html) | The canonical reference for reading CPU and allocation flame graphs generated by async-profiler + Helix's `FlameGraphGenerator` |
| [JDK Mission Control Downloads](https://adoptium.net/jmc/) | GUI for analysing `.jfr` recordings produced by Helix's `JfrRecordingManager` |

### :material-github: Project Repository

| Resource | Link |
|----------|------|
| **Helix Source Code** | [github.com/7amo10/helix-jvm-engine](https://github.com/7amo10/helix-jvm-engine) |
| **Phase 2 Study Notes — Bytecode & JIT** | [Topic 3](../notes/phase-2/topic-3-bytecode-jit/index.md) |
| **Phase 2 Study Notes — GC Deep Dive** | [Topic 2](../notes/phase-2/topic-2-garbage-collection/index.md) |
| **Phase 2 Study Notes — Profiling** | [Topic 5 — Ch 13](../notes/phase-2/topic-5-advanced-jvm/book-reading-ch13.md) |
| **Phase 2 Study Notes — Logging & Mechanical Sympathy** | [Topic 5 — Ch 14](../notes/phase-2/topic-5-advanced-jvm/book-reading-ch14.md) |

---

*Project Start: 2026-08-01 | Target Completion: 2026-08-07 | Status: :material-rocket-launch: In Progress*
