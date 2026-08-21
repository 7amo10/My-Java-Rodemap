---
tags: [jakarta-ee, cdi, lab, phase-3, week-1]
---

# :material-flask: Day 01 Lab Guide — CDI Scopes & Proxying

> **Module:** Week 1 — CDI 4.0, JAX-RS 3.1 & REST Services  
> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-1-CDI-JAXRS-REST](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)

---

## :material-target: Lab Objective

In CDI 4.0, understanding **normal scopes** (`@ApplicationScoped`, `@RequestScoped`) versus **pseudo-scopes** (`@Dependent`) is essential for building efficient enterprise applications. This lab guides you through constructing a multi-scoped bean hierarchy to observe **CDI Client Proxying** and **lazy instantiation mechanics**.

---

## :material-list-box: Key Concepts Covered

- **Normal Scopes vs. Pseudo-Scopes:** How CDI manages proxying for normal-scoped beans
- **Client Proxies:** Why injected fields hold a generated proxy reference rather than the actual contextual instance
- **Lifecycle Callbacks:** Observing `@PostConstruct` and `@PreDestroy` execution timing

---

## :material-code-braces: Component Architecture

Package: `com.ee.lab.cdi`

| Component | Scope | Responsibility |
|-----------|-------|---------------|
| `ApplicationCounter` | `@ApplicationScoped` | Tracks cumulative application calls. Contains an integer counter, `increment()` method, and `@PostConstruct` / `@PreDestroy` logging |
| `RequestContextService` | `@RequestScoped` | Simulates request-scoped business operations. Injects `ApplicationCounter` and `DependentHelper` |
| `DependentHelper` | `@Dependent` | Pseudo-scoped helper bean — directly bound to the lifetime of the owning injection point |
| `AppRunner` | Main Class | Uses Weld SE (`SeContainerInitializer`) to bootstrap the CDI container, simulate multiple request contexts, and invoke methods on injected beans |

---

## :material-stairs: Step-by-Step Implementation

### Step 1: Verify Dependencies (`pom.xml`)

```xml
<!-- Jakarta CDI 4.0 API -->
<dependency>
    <groupId>jakarta.enterprise</groupId>
    <artifactId>jakarta.enterprise.cdi-api</artifactId>
    <version>4.0.1</version>
</dependency>

<!-- Weld SE — CDI Reference Implementation -->
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-core</artifactId>
    <version>5.1.0.Final</version>
</dependency>
```

### Step 2: Implement Beans

In every constructor and `@PostConstruct` method, print:

```java
System.out.println(getClass().getName() + " constructor — hashCode: " + System.identityHashCode(this));
```

```java
@PostConstruct
void init() {
    System.out.println(getClass().getName() + " @PostConstruct — hashCode: " + System.identityHashCode(this));
}

@PreDestroy
void cleanup() {
    System.out.println(getClass().getName() + " @PreDestroy");
}
```

### Step 3: Bootstrap Container & Run Experiments

```java
SeContainerInitializer initializer = SeContainerInitializer.newInstance();
try (SeContainer container = initializer.initialize()) {
    // 1. Retrieve RequestContextService proxy
    RequestContextService svc = container.select(RequestContextService.class).get();

    // 2. Print proxy class name — will be a WeldSubclass
    System.out.println("Proxy class: " + svc.getClass().getName());

    // 3. Activate request context programmatically and execute business methods
    WeldContainer weld = (WeldContainer) container;
    weld.select(BoundRequestContext.class).get().associate(new HashMap<>());
    weld.select(BoundRequestContext.class).get().activate();

    svc.performBusinessOperation();   // triggers lazy instantiation
    svc.performBusinessOperation();   // same instance reused within request

    weld.select(BoundRequestContext.class).get().deactivate();
}
```

---

## :material-check-all: Verification Checklist

When running `mvn clean compile exec:java`, confirm:

1. **Proxy class name** — Printing the class of an injected `@RequestScoped` or `@ApplicationScoped` bean shows a generated subclass:
   ```
   Proxy class: com.ee.lab.cdi.RequestContextService$Proxy$_$$_WeldSubclass
   ```

2. **Lazy instantiation** — The proxy constructor fires once at injection time, but the actual target bean's constructor and `@PostConstruct` execute **lazily** upon the first actual method call within an active context.

3. **Scope boundary behavior:**
   - `ApplicationCounter` maintains its counter state **across all request boundaries**
   - `RequestContextService` creates a **fresh instance per request activation**
   - `DependentHelper` is created as a **direct instance** (no proxy) tied to `RequestContextService`'s lifetime

---

[:octicons-arrow-left-24: Back to Day 01 Index](index.md) | [:material-github: View Lab Solution](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-1-CDI-JAXRS-REST)
