---
tags: [jakarta-ee, cdi, qualifiers, interceptors, events, lab, phase-3]
---

# :material-flask: Day 02 Lab Guide — CDI Advanced Wiring

> **Module:** Week 1 — CDI 4.0, JAX-RS 3.1 & REST Services  
> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-1-CDI-JAXRS-REST](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)

---

## :material-target: Lab Objective

In enterprise CDI 4.0, managing complex architectures requires resolving ambiguous dependencies **without brittle string names**, producing dynamic resources, non-invasively wrapping cross-cutting concerns (logging/profiling), and decoupling business flows via events. This lab guides you through building and verifying these patterns using Weld SE.

---

## :material-list-box: Key Concepts Covered

- **Custom Qualifiers (`@Qualifier`):** Disambiguating multiple bean implementations of the same interface at compile time
- **Producers & Disposers (`@Produces`, `@Disposes`):** Encapsulating factory creation and lifecycle cleanup for external/third-party resources
- **Around-Invoke Interceptors (`@AroundInvoke`):** Profiling latency and thread CPU metrics transparently using `ThreadMXBean`
- **Decoupled Event Bus (`@Observes`, `@ObservesAsync`):** Firing synchronous vs. asynchronous event payloads on separate worker threads

---

## :material-code-braces: Component Architecture

Package: `com.ee.lab.cdi`

| Component | Role |
|-----------|------|
| `@MetricEngine` | Custom enum-backed `@Qualifier` annotation |
| `EngineType` | Enum: `HIGH_THROUGHPUT`, `STANDARD` |
| `HighThroughputTelemetryProcessor` | `@MetricEngine(HIGH_THROUGHPUT)` implementation |
| `StandardTelemetryProcessor` | `@MetricEngine(STANDARD)` implementation |
| `SystemInfrastructureProducer` | `@Produces` `ThreadMXBean` and factory-created `TelemetryChannel`; `@Disposes` cleanup |
| `@Monitored` | Custom `@InterceptorBinding` annotation |
| `PerformanceMonitoringInterceptor` | `@Monitored` interceptor measuring wall-clock and CPU time |
| `AuditEvent` | Immutable record — CDI event payload |
| `AuditLoggingObserver` | Synchronous (`@Observes`) and async (`@ObservesAsync`) handlers |
| `OrderProcessingService` | `@Monitored` service injecting qualified processors, firing events |
| `AppRunner` | Weld SE bootstrap & end-to-end verification |

---

## :material-stairs: Step-by-Step Implementation

### Step 1: Define the Custom Qualifier

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
public @interface MetricEngine {
    EngineType value();

    enum EngineType { HIGH_THROUGHPUT, STANDARD }
}
```

### Step 2: Implement the Producer

```java
@ApplicationScoped
public class SystemInfrastructureProducer {

    @Produces
    @ApplicationScoped
    public ThreadMXBean produceThreadMXBean() {
        return ManagementFactory.getThreadMXBean();
    }

    @Produces
    @RequestScoped
    public TelemetryChannel produceTelemetryChannel() {
        return new TelemetryChannel("tcp://localhost:5000");
    }

    public void disposeTelemetryChannel(@Disposes TelemetryChannel channel) {
        channel.close();
        System.out.println("TelemetryChannel closed via @Disposes");
    }
}
```

### Step 3: Implement the `@AroundInvoke` Interceptor

```java
@Monitored
@Interceptor
@Priority(Interceptor.Priority.APPLICATION + 100)
public class PerformanceMonitoringInterceptor {

    @Inject ThreadMXBean threadMXBean;

    @AroundInvoke
    public Object monitor(InvocationContext ctx) throws Exception {
        long startWall = System.currentTimeMillis();
        long startCPU = threadMXBean.getCurrentThreadCpuTime();

        System.out.println("[ENTER] " + ctx.getMethod().getName());
        try {
            return ctx.proceed();
        } finally {
            long wallMs = System.currentTimeMillis() - startWall;
            long cpuMs = (threadMXBean.getCurrentThreadCpuTime() - startCPU) / 1_000_000;
            System.out.printf("[EXIT] %s — wall: %dms, cpu: %dms%n",
                ctx.getMethod().getName(), wallMs, cpuMs);
        }
    }
}
```

### Step 4: Fire Synchronous & Async Events

```java
@ApplicationScoped
@Monitored
public class OrderProcessingService {

    @Inject @MetricEngine(EngineType.HIGH_THROUGHPUT) TelemetryProcessor processor;
    @Inject Event<AuditEvent> auditEvent;

    public void processOrder(String orderId) {
        processor.process(orderId);
        // Synchronous — runs on calling thread before this method returns
        auditEvent.fire(new AuditEvent(orderId, "PROCESSED", Instant.now()));
        // Asynchronous — dispatched to ForkJoinPool
        auditEvent.fireAsync(new AuditEvent(orderId, "ASYNC_AUDIT", Instant.now()));
    }
}
```

---

## :material-check-all: Verification Checklist

Run `mvn clean compile exec:java` and confirm:

1. **`PerformanceMonitoringInterceptor`** prints entry/exit with wall-clock duration and thread CPU time around `processOrder()`
2. **Producer lifecycle:** `TelemetryChannel` is allocated by producer; upon container shutdown, `@Disposes` method is called automatically — look for the "TelemetryChannel closed" log line
3. **Thread context verification:**
   - Synchronous `@Observes` handler: `Thread.currentThread().getName()` shows **main** thread
   - Async `@ObservesAsync` handler: shows a **ForkJoinPool.commonPool-worker-X** thread name

---

[:octicons-arrow-left-24: Back to Day 02 Index](index.md) | [:material-github: View Lab Solution](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)
