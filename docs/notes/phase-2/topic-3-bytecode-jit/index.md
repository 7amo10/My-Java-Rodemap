---
id: topic-3-bytecode-jit-index
aliases: []
tags:
- java
- performance
- jvm
- bytecode
- jit
- compilation
- optimizing-java
- phase-2
---

# :material-lightning-bolt: Topic 3: Bytecode & JIT Internals

> **Book:** Optimizing Java — Practical Techniques for Improving JVM Application Performance
>
> **Authors:** Benjamin J. Evans, James Gough, Chris Newland (O'Reilly Media)
>
> **Part Covered:** Part III — JIT Compilation (Chapters 9–10)

---

## :material-notebook-outline: Topic Structure

| Document | Chapter | Coverage | Status |
|----------|---------|----------|--------|
| [:material-book-open-page-variant: Chapter 9 — Code Execution on the JVM](book-reading-ch9.md) | Ch 9 | JVM as stack machine, bytecode opcode families (load/store, arithmetic, flow control, method invocation, platform), HotSpot template interpreter, JIT fundamentals (C1/C2/tiered), Code Cache, AOT vs JIT | :material-check-circle: Complete |
| [:material-book-open-page-variant: Chapter 10 — Understanding JIT Compilation](book-reading-ch10.md) | Ch 10 | JITWatch tool, Method Data Objects (MDOs), inlining (gateway optimization), loop unrolling, escape analysis & scalar replacement, lock elision/coarsening, monomorphic dispatch, intrinsics, OSR, 8000-byte compilation wall | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Chapter 9: Code Execution on the JVM — Stack Machine, Bytecode & JIT Basics

Establishes the JVM's fundamental execution model: a **stack-based virtual machine** where all computations happen on a per-method evaluation stack, not in CPU registers. Every JVM opcode pops operands from the stack, performs an operation, and pushes results back. Introduces **bytecode** as the intermediate representation produced by `javac`: architecture-neutral, platform-agnostic, and strictly big-endian. Each opcode is 1 byte (200 of 256 values used in Java 10). Covers the five **opcode families**: Load/Store (`load`, `store`, `ldc`, `getfield`), Arithmetic (`iadd`, `dmul`, `fcmp`), Flow Control (`goto`, `if_icmpge`, `tableswitch`), Method Invocation (`invokevirtual`, `invokespecial`, `invokeinterface`, `invokestatic`, `invokedynamic`), and Platform (`new`, `monitorenter`, `monitorexit`). Deep-dives into the five **method invocation opcodes**: `invokevirtual` (standard virtual dispatch through vtable), `invokespecial` (direct dispatch for private/constructors/super), `invokeinterface` (interface vtable lookup — slower), `invokestatic` (no receiver needed), and `invokedynamic` (runtime strategy pattern — powers lambdas in Java 8+). Explains why `final` methods use `invokevirtual` in bytecode for binary compatibility (JLS 13.4.17) but HotSpot uses an internal private opcode for fast direct dispatch. Introduces HotSpot's **Template Interpreter** — dynamically generated at JVM startup using native CPU assembly — as opposed to a slow switch-loop interpreter. Covers **JIT compilation basics**: Profile-Guided Optimization (PGO), why profile data is discarded at shutdown, On-Stack Replacement (OSR) for hot loops in otherwise-cold methods, and the 5-tier tiered compilation model (Tier 0 = Interpreter → Tier 4 = C2 aggressive). Explains the **Code Cache** as a fixed-size native memory region for compiled nmethods, and why exhausting it silently halts all JIT compilation.

### Chapter 10: Understanding JIT Compilation — Optimizations Deep Dive

The definitive chapter on what the JIT actually does inside. Centers on **JITWatch** — an open-source JavaFX tool for inspecting JIT decisions, showing Java source, bytecode, and native assembly side-by-side. Covers **inlining** as the gateway optimization that enables everything else: by copying callee code into the caller, the JIT can then see escape analysis, dead code elimination, loop unrolling, and lock elision opportunities that were previously invisible. Inlining limits: `MaxInlineSize` (35 bytes, non-hot), `FreqInlineSize` (325 bytes, hot), `MaxInlineLevel` (9 call levels deep). **Loop unrolling** duplicates loop bodies to reduce back-branch frequency — works for `int`/`short`/`char` counters but NOT `long` counters, causing ~64% performance penalty for long-typed loops. Also enables **bounds check elimination** (split loop into pre/main/post sections; main section has no bounds checks). **Escape Analysis**: determines if an object escapes its defining method (`NoEscape` → scalar replace into registers, avoiding heap allocation entirely; `ArgEscape` → passes to inlined callee; `GlobalEscape` → must heap-allocate). Critical limit: arrays > 64 elements are NOT scalar replaced (configurable via `-XX:EliminateAllocationArraySizeLimit`). **Lock optimizations**: lock elision (remove locks on non-escaping objects), lock coarsening (merge adjacent `synchronized` blocks), nested lock elimination. **Monomorphic dispatch**: at monomorphic call sites (one receiver type observed), replaces vtable lookup with klass word check + direct branch — degraded at megamorphic sites (3+ types); recoverable via explicit `instanceof` type peeling. **Intrinsics**: pre-written architecture-specific assembly for core methods (`System.arraycopy`, AES, `Math.log10`, `System.currentTimeMillis`). **The 8,000-byte compilation wall**: methods > 8,000 bytes of bytecode are NEVER JIT-compiled — they run at interpreter speed forever (~50% performance).

---

## :material-book-open-variant: What You'll Master

- **Stack Machine Model** — How the JVM evaluation stack works vs CPU registers
- **Bytecode Opcode Families** — Load/store, arithmetic, flow control, method invocation, platform
- **Method Invocation Semantics** — `invokevirtual` vs `invokespecial` vs `invokedynamic` dispatch
- **Template Interpreter** — HotSpot's dynamically-generated native-code interpreter
- **Tiered Compilation** — 5 execution levels (Tier 0-4), C1 vs C2, compilation pathways
- **Code Cache** — Fixed-size native code region; exhaustion silently kills JIT
- **JITWatch** — TriView: Java source + bytecode + native assembly side-by-side
- **Inlining** — The gateway optimization; size limits (`MaxInlineSize`, `FreqInlineSize`)
- **Loop Unrolling** — Why `int` loops are 64% faster than `long` loops
- **Escape Analysis & Scalar Replacement** — Stack allocation via object decomposition into scalars
- **Monomorphic vs Megamorphic Dispatch** — Why 3+ receiver types kill call site performance
- **Intrinsics** — Pre-built assembly implementations of core library methods
- **8,000-Byte Wall** — Never write methods > 8,000 bytes of bytecode

---

## :material-map-marker-path: Optimizing Java — Book Context

| Reading Group | Chapters | Topics |
|:---:|---|---|
| Week 1 (Topic 1) | 1–5 | Foundations: observables, JVM anatomy, hardware, testing, statistics |
| Week 2 (Topic 2) | 6–8 | Garbage Collection: algorithms, heap analysis, GC tuning |
| **Week 3 (This part)** | 9–10 | **Bytecode & JIT: stack machine, opcodes, inlining, escape analysis** |
| Week 4 | 11–12 | Profiling & Concurrency: profilers, thread analysis, lock contention |
| Week 5 | 13–15 | Advanced JVM Optimization Techniques |

---

## :material-cogs: Key Internals to Understand

- **Evaluation Stack vs CPU Registers** — Why the JVM's stack architecture adds an extra abstraction layer that JIT must eliminate to produce efficient native code
- **`invokedynamic` Bootstrap Method** — How lambda desugaring works: a `CallSite` object with a `MethodHandle` target is created once at first invocation; subsequent calls bypass the bootstrap entirely
- **Method Data Objects (MDOs)** — Per-method profiling data structures (invocation counts, branch probabilities, observed receiver types at call sites) that C1 populates and C2 consumes for speculative optimization
- **Deoptimization** — When a C2 guard fails (unexpected class type, branch probability changes), HotSpot must "deopt" the nmethod: mark it `made not entrant`, and re-interpret the method from the point of failure
- **Scalar Replacement Limit** — Why the `EliminateAllocationArraySizeLimit=64` default means a 65-element array allocates on the heap while a 64-element array gets register-allocated
- **Type Peeling for Megamorphic Sites** — Adding an explicit `instanceof` check before a polymorphic call site to peel off the dominant type into a monomorphic fast path, leaving the remaining site bimorphic

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Read Chapter 9 — Code Execution on the JVM
- [x] Read Chapter 10 — Understanding JIT Compilation
- [x] Write Chapter 9 notes
- [x] Write Chapter 10 notes
- [ ] Week 4: Read Chapters 11–12 (Profiling & Concurrency)

---

*Start Date: 2026-07-23 | Week 3 Completed: 2026-07-23*
