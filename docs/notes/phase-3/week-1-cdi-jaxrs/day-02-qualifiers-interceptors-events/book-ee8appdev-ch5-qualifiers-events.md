---
tags: [jakarta-ee, cdi, qualifiers, producers, events, phase-3]
---

# :material-book-open-page-variant: EE8 AppDev — Chapter 5: Qualifiers, Producers & CDI Events

> **Book:** Java EE 8 Application Development — David R. Heffelfinger (Packt)  
> **Note:** Chapter 5 scope & lifecycle content is covered in [Day 01](../day-01-cdi-scopes-lifecycle/book-ee8appdev-ch5.md). This page focuses on the **advanced wiring** section of Ch5: Qualifiers, Producers, and Events.

---

## :material-label: Qualifiers — Compile-Time Disambiguation

### The Problem with Interface Injection Without Qualifiers

When multiple classes implement the same interface, CDI cannot decide which one to inject — it throws an `AmbiguousResolutionException` at deployment time:

```
AmbiguousResolutionException: Two beans match injection point:
  - StandardTelemetryProcessor
  - HighThroughputTelemetryProcessor
```

### Solution: Custom `@Qualifier` Annotation

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD,
         ElementType.FIELD, ElementType.PARAMETER})
public @interface Premium {}
```

Apply to the specific implementation and the injection point:

```java
@Named @RequestScoped @Premium
public class PremiumCustomer extends Customer {
    private Integer discountCode;
}

// At injection point:
@Inject @Premium
private Customer customer;   // resolves to PremiumCustomer
```

### Enum-Backed Qualifiers (Enterprise Pattern)

For more than two variants, use an enum member inside the qualifier annotation:

```java
@Qualifier @Retention(RUNTIME) @Target({TYPE, METHOD, FIELD, PARAMETER})
public @interface MetricEngine {
    EngineType value();

    @Nonbinding String description() default "";   // @Nonbinding = excluded from resolution

    enum EngineType {
        HIGH_THROUGHPUT,   // for bulk, low-latency metrics
        STANDARD           // for standard precision
    }
}
```

Usage:

```java
@MetricEngine(EngineType.HIGH_THROUGHPUT)
public class HighThroughputTelemetryProcessor implements TelemetryProcessor { ... }

@MetricEngine(EngineType.STANDARD)
public class StandardTelemetryProcessor implements TelemetryProcessor { ... }

// Inject specific implementation:
@Inject @MetricEngine(EngineType.HIGH_THROUGHPUT)
private TelemetryProcessor processor;
```

---

## :material-factory: Producers & Disposers

### `@Produces` — Factory Methods for Unmanageable Resources

Use `@Produces` when CDI cannot construct the bean itself:
- External library classes without CDI annotations (e.g., `ThreadMXBean`, JDBC `Connection`)
- Resources requiring custom initialization logic
- Beans that depend on runtime configuration

```java
@ApplicationScoped
public class SystemInfrastructureProducer {

    // Produce a shared ThreadMXBean as @ApplicationScoped
    @Produces
    @ApplicationScoped
    public ThreadMXBean produceThreadMXBean() {
        ThreadMXBean bean = ManagementFactory.getThreadMXBean();
        bean.setThreadCpuTimeEnabled(true);   // enable CPU time tracking
        return bean;
    }

    // Produce a @RequestScoped TelemetryChannel
    @Produces
    @RequestScoped
    public TelemetryChannel produceTelemetryChannel(
            @MetricEngine(EngineType.HIGH_THROUGHPUT) TelemetryProcessor processor) {
        return new TelemetryChannel("tcp://localhost:5000", processor);
    }

    // @Disposes — called automatically when produced instance goes out of scope
    public void disposeTelemetryChannel(@Disposes TelemetryChannel channel) {
        channel.close();
        System.out.println("TelemetryChannel closed by @Disposes");
    }
}
```

### Producer Fields

A field annotated with `@Produces` also works:

```java
@Produces
@ApplicationScoped
ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
```

### `@Disposes` Rules

- Must be in the **same class** as the corresponding `@Produces` method
- The `@Disposes` parameter type must **exactly match** the produced type
- Called by CDI when the produced bean is destroyed (scope ends)
- If producer is `@ApplicationScoped`, disposer fires at application shutdown
- If producer is `@RequestScoped`, disposer fires at request end

---

## :material-bell: CDI Events — Full Detail

### Synchronous Events

```java
@ApplicationScoped
public class OrderProcessingService {

    @Inject
    private Event<AuditEvent> auditEventBus;

    public void processOrder(String orderId) {
        // Business logic...

        // Fire synchronous event — blocks until ALL @Observes handlers complete
        auditEventBus.fire(new AuditEvent(orderId, "PROCESSED", Instant.now()));

        System.out.println("All synchronous observers finished — continuing");
    }
}
```

Observer method:

```java
@ApplicationScoped
public class AuditLoggingObserver {

    public void onAuditSync(@Observes AuditEvent event) {
        // Runs on SAME thread as fire() call, before fire() returns
        System.out.println("[SYNC] on thread: " + Thread.currentThread().getName());
        System.out.println("Order " + event.orderId() + " audited");
    }
}
```

### Asynchronous Events (CDI 2.0+)

```java
// Firing async:
CompletionStage<AuditEvent> future = auditEventBus.fireAsync(
    new AuditEvent(orderId, "ASYNC_AUDIT", Instant.now())
);
// fire() returns IMMEDIATELY — observers run on ForkJoinPool workers

// Optional: react when all async observers complete
future.thenAccept(e -> System.out.println("All async observers done for: " + e.orderId()));
```

```java
public void onAuditAsync(@ObservesAsync AuditEvent event) {
    // Runs on ForkJoinPool.commonPool-worker-X (NOT the main thread)
    System.out.println("[ASYNC] on thread: " + Thread.currentThread().getName());
}
```

### Event Payload Best Practice

Use a **Java record** as the event payload — immutable, thread-safe, minimal boilerplate:

```java
public record AuditEvent(String orderId, String action, Instant timestamp) {}
```

### Qualified Events

Filter which observer receives an event using qualifiers:

```java
// Only fires to observers that have both @Observes AND @MetricEngine(HIGH_THROUGHPUT)
auditEventBus.select(new MetricEngineQualifier(EngineType.HIGH_THROUGHPUT))
             .fire(new AuditEvent(orderId, "HIGH_PRIORITY", Instant.now()));

// Observer only receives high-throughput engine events:
public void onHighPriorityAudit(
        @Observes @MetricEngine(EngineType.HIGH_THROUGHPUT) AuditEvent event) { ... }
```

### Event Ordering with `@Priority`

```java
// Priority 1000 (APPLICATION) — fires FIRST
public void handleFirst(
        @Observes @Priority(Interceptor.Priority.APPLICATION) AuditEvent e) {
    System.out.println("First handler");
}

// Priority 1100 (APPLICATION + 100) — fires SECOND
public void handleSecond(
        @Observes @Priority(Interceptor.Priority.APPLICATION + 100) AuditEvent e) {
    System.out.println("Second handler");
}
```

Default priority (no `@Priority`) = `Interceptor.Priority.APPLICATION + 500`.

---

## :material-chart-timeline: Interceptors (EE8 Ch5 Bonus Coverage)

### Defining an Interceptor Binding

```java
@InterceptorBinding
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Monitored {}
```

### Implementing the Interceptor

```java
@Monitored          // binds to @Monitored annotation
@Interceptor
@Priority(Interceptor.Priority.APPLICATION + 100)
public class PerformanceMonitoringInterceptor {

    @Inject ThreadMXBean threadMXBean;

    @AroundInvoke
    public Object monitor(InvocationContext ctx) throws Exception {
        long wallStart = System.currentTimeMillis();
        long cpuStart  = threadMXBean.getCurrentThreadCpuTime();

        System.out.printf("[ENTER] %s::%s%n",
            ctx.getTarget().getClass().getSimpleName(),
            ctx.getMethod().getName());

        try {
            return ctx.proceed();   // calls the actual method (or next interceptor)
        } finally {
            long wallMs = System.currentTimeMillis() - wallStart;
            long cpuMs  = (threadMXBean.getCurrentThreadCpuTime() - cpuStart) / 1_000_000;
            System.out.printf("[EXIT]  %s::%s — wall: %dms, cpu: %dms%n",
                ctx.getTarget().getClass().getSimpleName(),
                ctx.getMethod().getName(), wallMs, cpuMs);
        }
    }
}
```

### Applying the Interceptor

```java
@ApplicationScoped
@Monitored          // any method in this class is intercepted
public class OrderProcessingService {
    public void processOrder(String orderId) { ... }
}
```

!!! important "Enable interceptor in beans.xml"
    Interceptors must be enabled in `beans.xml` (or via `@Priority` in CDI 1.2+):
    ```xml
    <beans>
      <interceptors>
        <class>com.ee.lab.cdi.PerformanceMonitoringInterceptor</class>
      </interceptors>
    </beans>
    ```
    With `@Priority` annotation on the interceptor class, explicit `beans.xml` registration is optional in CDI 1.2+.

---

## :material-key: Key Takeaways

1. **`@Qualifier`** → compile-time injection disambiguation; always prefer over `@Named` for DI
2. **Enum-backed qualifiers** → cleanest pattern for multiple variants of the same type
3. **`@Produces`** → factory for external/unmanageable resources; pairs with `@Disposes` for cleanup
4. **`fire()` is synchronous** — all `@Observes` handlers run before `fire()` returns
5. **`fireAsync()` is non-blocking** — returns `CompletionStage`; observers run on ForkJoinPool workers
6. **`@AroundInvoke` interceptors** wrap method calls non-invasively; `ctx.proceed()` calls the real method

---

[:octicons-arrow-left-24: Back to Day 02 Index](index.md)
