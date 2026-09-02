---
tags: [jakarta-ee, jpa, entity-manager, persistence-context, lifecycle, phase-3]
---

# :material-book-open-page-variant: Pro Persistence — Chapter 6: Entity Manager

> **Book:** Pro Jakarta Persistence in Jakarta EE 10 (Apress, 2023)  
> **Chapter:** 6 — Entity Manager

---

## :material-information: Core Concepts

A **Persistence Context** is a managed set of entity instances associated with a persistence unit. It acts as a First-Level Cache (Identity Map) — tracking all changes to entities (dirty checking) and synchronizing them to the database at flush/commit time.

An **EntityManager** is the Java interface through which you interact with the persistence context.

---

## :material-server: Types of EntityManagers

### 1. Container-Managed EntityManager (Jakarta EE)

Injected via `@PersistenceContext` — lifecycle managed by the container:

```java
@Stateless
public class ClusterNodeService {
    // Transaction-scoped (default) — context lives only for the JTA transaction
    @PersistenceContext(unitName = "telemetryPU")
    private EntityManager em;

    public ClusterNode findById(Long id) {
        return em.find(ClusterNode.class, id);
    }
}
```

**Transaction-Scoped (Default):** Context created per JTA transaction and destroyed when transaction ends. Entities become detached after the transaction commits.

**Extended (Stateful Only):** Context tied to the `@Stateful` bean lifecycle — spans multiple transactions:

```java
@Stateful
public class NodeEditSession {
    @PersistenceContext(unitName = "telemetryPU",
                        type = PersistenceContextType.EXTENDED)
    private EntityManager em;

    private ClusterNode current;

    public void beginEdit(Long id) {
        current = em.find(ClusterNode.class, id);  // stays managed across calls
    }

    @Remove
    public void done() {}  // @Remove destroys the bean, closing extended context
}
```

!!! warning "Extended Context Collision"
    Calling an Extended stateful bean from a stateless bean that already started a transaction (creating its own transaction-scoped context) causes an exception. Workaround: annotate the stateful bean with `@TransactionAttribute(REQUIRES_NEW)` or `NOT_SUPPORTED`.

### 2. Application-Managed EntityManager (Java SE / EE)

Created manually via `EntityManagerFactory`. Application must call `close()`:

```java
@PersistenceUnit(unitName = "telemetryPU")
EntityManagerFactory emf;

public void doWork() {
    EntityManager em = emf.createEntityManager();
    try {
        em.getTransaction().begin();
        // ... work ...
        em.getTransaction().commit();
    } catch (Exception e) {
        em.getTransaction().rollback();
    } finally {
        em.close();   // MUST always close
    }
}
```

---

## :material-api: EntityManager Operations

### `persist(entity)` — Transient → Managed

```java
ClusterNode node = new ClusterNode("node-1.cluster.local");   // Transient
em.persist(node);   // Managed — scheduled for INSERT
// SQL INSERT fires at flush/commit, not immediately
```

- Throws `TransactionRequiredException` if called on transaction-scoped EM without active transaction
- Throws `EntityExistsException` if duplicate primary key detected

### `find(Class, id)` — Load by Primary Key

```java
ClusterNode node = em.find(ClusterNode.class, 1L);
// Returns null if not found (NOT an exception)
// Checks L1 cache first — no SQL if already loaded in context
```

### `getReference(Class, id)` — Proxy Without DB Hit

```java
// Useful for creating FK relationships WITHOUT loading the target entity:
ServerRack rack = em.getReference(ServerRack.class, 10L);
node.setRack(rack);    // FK assignment — no SELECT for ServerRack executed
em.persist(node);
```

!!! warning "`getReference()` throws lazily"
    If the entity doesn't exist, `EntityNotFoundException` is thrown when you first **access an attribute** on the proxy — NOT when you call `getReference()`. 

### `remove(entity)` — Managed → Removed

```java
ClusterNode node = em.find(ClusterNode.class, 1L);
em.remove(node);    // Schedules DELETE on commit
// Must pass a MANAGED entity — detached entities throw IllegalArgumentException
```

!!! important "Relationship cleanup before remove"
    If other entities hold a FK reference to this entity, you must clear those relationships first (e.g., `node.setRack(null)`) before calling `em.remove()` — otherwise a DB constraint violation fires at commit.

### `merge(detached)` — Integrates Detached State

```java
// node is detached (was loaded in a previous transaction):
node.setHostname("updated-hostname.cluster.local");

// WRONG — do not use the passed argument after merge:
em.merge(node);    // ← node is still detached!

// CORRECT — always use the return value:
ClusterNode managed = em.merge(node);  // copies state, returns managed copy
managed.setUpdatedAt(Instant.now());    // operate on managed copy
```

### `flush()` — Push SQL to DB Without Committing

```java
em.flush();   // Generates and executes SQL immediately; transaction still open
// Useful to check DB constraints before long batch operations complete
```

### `refresh(entity)` — Overwrite Memory from DB

```java
em.refresh(node);   // Discards in-memory changes; reloads from current DB row
// Useful after external (non-JPA) changes to the database
```

### `clear()` / `detach(entity)` — Evict from Context

```java
em.clear();         // Evicts ALL entities — in-memory changes lost!
em.detach(node);    // Evicts only this entity from context
```

---

## :material-database: First-Level Cache — Identity Map

Within a single persistence context, JPA guarantees **object identity** for the same primary key:

```java
ClusterNode ref1 = em.find(ClusterNode.class, 1L);  // SQL SELECT
ClusterNode ref2 = em.find(ClusterNode.class, 1L);  // Cache hit — NO SQL

ref1 == ref2   // TRUE — same Java instance
```

This means:
- Zero redundant SQL for repeated loads of the same entity in one transaction
- Modifications via any reference are visible through all references
- `contains(entity)` checks if entity is currently in the context

---

## :material-autorenew: Automatic Dirty Checking

Hibernate snapshots entity state when entities enter the managed state. At `flush()` / commit, it compares current state to snapshot and issues `UPDATE` for any changed fields — **without explicit `save()` or `merge()` calls**:

```java
em.getTransaction().begin();
ClusterNode node = em.find(ClusterNode.class, 1L);

// Automatic dirty tracking — just modify:
node.setStatus(NodeStatus.MAINTENANCE);      // dirtied
node.getHardware().setRamGb(256);           // embedded also tracked

em.getTransaction().commit();
// Hibernate: UPDATE cluster_node SET status='MAINTENANCE', ram_gb=256 WHERE id=1
```

---

## :material-web: Web Application Patterns for Detachment

### Pattern 1: Eager Pre-Loading

```java
// Load all needed associations while context is open:
ClusterNode node = em.find(ClusterNode.class, id);
node.getTelemetryRecords().size();   // trigger lazy load inside transaction
return node;   // safe to use detached outside
```

### Pattern 2: Extended Persistence Context (Stateful Bean)

```java
@Stateful
public class NodeEditSession {
    @PersistenceContext(type = PersistenceContextType.EXTENDED)
    EntityManager em;

    public ClusterNode getNode() { return current; }  // always managed!
    // No merge() needed — just modify and call save()
}
```

### Pattern 3: Unsynchronized Persistence Context (CDI Conversation)

```java
@Stateful
public class NodeCheckoutSession {
    @PersistenceContext(type = EXTENDED,
                        synchronization = SynchronizationType.UNSYNCHRONIZED)
    EntityManager em;

    public void addPendingChange(ClusterNode node) {
        node.setStatus(NodeStatus.MAINTENANCE);  // tracked but NOT flushed
    }

    @Remove
    public void commit() {
        em.joinTransaction();   // NOW join JTA transaction to flush all pending changes
    }
}
```

---

## :material-key: Key Takeaways

1. **Transaction-scoped EM** (default) — context dies when transaction ends; entities become detached
2. **Extended EM** — tied to `@Stateful` bean lifecycle; entities stay managed across transactions
3. **`merge()` returns the managed copy** — the passed argument is still detached
4. **`getReference()`** returns a proxy without DB hit — use only for FK-only operations
5. **First-Level Cache guarantees** `ref1 == ref2` for same PK in same context
6. **Automatic dirty checking** fires UPDATE automatically — no explicit save needed

---

[:octicons-arrow-left-24: Back to Day 09 Index](index.md)
