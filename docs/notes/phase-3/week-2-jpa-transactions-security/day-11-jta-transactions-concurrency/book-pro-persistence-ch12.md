---
tags: [jakarta-ee, jpa, locking, concurrency, caching, lifecycle-callbacks, phase-3]
---

# :material-book-open-page-variant: Pro Persistence — Chapter 12: Advanced Topics (Locking, Caching & Lifecycle)

> **Book:** Pro Jakarta Persistence in Jakarta EE 10 (Apress, 2023)  
> **Chapter:** 12 — Other Advanced Topics

---

## :material-lock: 1. Locking and Concurrency Control

### Optimistic Locking — `@Version`

Assume conflicts are rare — no DB lock acquired upfront. A version column tracks changes:

```java
@Entity
public class ClusterNode {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Version    // Supported types: int, Integer, short, Short, long, Long, java.sql.Timestamp
    private Long version;   // JPA auto-increments on every UPDATE
}
```

**Generated SQL behavior:**
```sql
-- Normal update:
UPDATE cluster_node SET status='MAINTENANCE', version=2 WHERE id=1 AND version=1

-- Stale data (version mismatch):
UPDATE cluster_node SET status='OFFLINE', version=2 WHERE id=1 AND version=1
-- → 0 rows updated → JPA throws OptimisticLockException
```

**Handling `OptimisticLockException`:**

```java
try {
    node.setStatus(NodeStatus.OFFLINE);
    em.flush();   // or on transaction commit
} catch (OptimisticLockException | RollbackException e) {
    // Retry logic or inform user of conflict
    throw new ConflictException("Concurrent modification detected", e);
}
```

### Advanced Optimistic Lock Modes — `LockModeType`

```java
// Default — version checked at flush:
LockModeType.OPTIMISTIC   // aka READ — ensures repeatable read

// Force version increment even if entity was only READ (not modified):
LockModeType.OPTIMISTIC_FORCE_INCREMENT  // aka WRITE
```

`OPTIMISTIC_FORCE_INCREMENT` is useful when you read an entity and want to ensure nobody else modifies it — even if you don't change it yourself:

```java
// Reserve the node — bump version to block concurrent modification:
ClusterNode node = em.find(ClusterNode.class, id, LockModeType.OPTIMISTIC_FORCE_INCREMENT);
// Even if node fields unchanged, version will increment on commit
```

### Pessimistic Locking

Acquires actual database lock immediately (`SELECT ... FOR UPDATE`):

```java
// Exclusive write lock — blocks all concurrent readers AND writers:
ClusterNode node = em.find(ClusterNode.class, 1L, LockModeType.PESSIMISTIC_WRITE);

// Shared read lock — allows concurrent reads, blocks writers:
ClusterNode node = em.find(ClusterNode.class, 1L, LockModeType.PESSIMISTIC_READ);

// Force version increment + write lock:
ClusterNode node = em.find(ClusterNode.class, 1L, LockModeType.PESSIMISTIC_FORCE_INCREMENT);
```

**Lock on an already-managed entity:**

```java
ClusterNode node = em.find(ClusterNode.class, 1L);   // loaded without lock
em.lock(node, LockModeType.PESSIMISTIC_WRITE);       // acquire lock now
```

**Lock timeout (prevents indefinite blocking):**

```java
Map<String, Object> hints = new HashMap<>();
hints.put("jakarta.persistence.lock.timeout", 3000);   // 3 seconds; 0 = no-wait

ClusterNode node = em.find(ClusterNode.class, 1L, LockModeType.PESSIMISTIC_WRITE, hints);
// Throws LockTimeoutException or PessimisticLockException if lock not acquired in time
```

### `LockModeType` Quick Reference

| `LockModeType` | Lock Type | Version Increment | Use Case |
|--------------|-----------|-----------------|---------|
| `NONE` | None | No | Default — no explicit lock |
| `OPTIMISTIC` | None | On conflict only | Repeatable read guarantee |
| `OPTIMISTIC_FORCE_INCREMENT` | None | Always | Read-only but block modifications |
| `PESSIMISTIC_READ` | Shared (S) | No | Allow reads, block writes |
| `PESSIMISTIC_WRITE` | Exclusive (X) | No | Block all concurrent access |
| `PESSIMISTIC_FORCE_INCREMENT` | Exclusive (X) | Always | Most restrictive |

---

## :material-database-check: 2. Second-Level Cache (L2 Cache)

The **First-Level Cache** (L1) is per-persistence-context (transaction-scoped). The **Second-Level Cache** (L2) is shared across all transactions and EntityManager instances within the same application.

### Static Configuration — `persistence.xml`

```xml
<persistence-unit name="telemetryPU">
    <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
    <!-- Other settings -->
</persistence-unit>
```

| `shared-cache-mode` Value | Behavior |
|--------------------------|---------|
| `ALL` | Cache ALL entities |
| `NONE` | Disable L2 cache entirely |
| `ENABLE_SELECTIVE` | Only cache entities with `@Cacheable(true)` |
| `DISABLE_SELECTIVE` | Cache everything EXCEPT entities with `@Cacheable(false)` |
| `NOT_SPECIFIED` | Provider default behavior |

### `@Cacheable` Annotation

```java
// With ENABLE_SELECTIVE mode — opt entities into cache:
@Entity
@Cacheable(true)
public class ServerRack { ... }  // ← will be cached

@Entity
@Cacheable(false)
public class TelemetryRecord { ... }  // ← never cached (high write frequency)

@Entity
// No @Cacheable — not cached with ENABLE_SELECTIVE
public class ClusterNode { ... }
```

### Runtime Cache Control Hints

```java
TypedQuery<ClusterNode> q = em.createQuery("SELECT n FROM ClusterNode n", ClusterNode.class);

// Bypass L2 cache — always go to DB:
q.setHint("jakarta.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);

// Don't store result in L2 cache:
q.setHint("jakarta.persistence.cache.storeMode", CacheStoreMode.BYPASS);

// Store AND refresh existing cache entries from DB:
q.setHint("jakarta.persistence.cache.storeMode", CacheStoreMode.REFRESH);
```

### Cache API — Programmatic Eviction

```java
Cache cache = emf.getCache();

// Check if entity is cached:
boolean isCached = cache.contains(ClusterNode.class, 1L);

// Evict one entity:
cache.evict(ClusterNode.class, 1L);

// Evict all entities of a type:
cache.evict(ClusterNode.class);

// Evict entire L2 cache:
cache.evictAll();
```

---

## :material-callback: 3. Entity Lifecycle Callbacks

All 7 lifecycle callback annotations — can be declared inside the entity or in an `@EntityListeners` external class:

### In-Entity Callbacks (no parameter)

```java
@Entity
@EntityListeners(AuditListener.class)
public class ClusterNode {

    @PrePersist
    private void onPrePersist() {
        this.createdAt = Instant.now();
        this.version = 0L;
    }

    @PostPersist
    private void onPostPersist() {
        System.out.println("Persisted ClusterNode: " + id);  // id now available
    }

    @PreUpdate
    private void onPreUpdate() {
        this.updatedAt = Instant.now();
    }

    @PostUpdate
    private void onPostUpdate() {
        // Invalidate caches, notify observers
    }

    @PreRemove
    private void onPreRemove() {
        // Disconnect from rack before deletion
        this.rack = null;
    }

    @PostRemove
    private void onPostRemove() {
        System.out.println("Node deleted from DB");
    }

    @PostLoad
    private void onPostLoad() {
        // Compute transient field — not stored in DB:
        this.durationSinceCreatedMs = Duration.between(createdAt, Instant.now()).toMillis();
    }
}
```

### External `@EntityListeners`

```java
public class AuditListener {
    @PrePersist
    public void beforePersist(Object entity) {    // entity passed as parameter
        System.out.println("[AUDIT] Persisting: " + entity.getClass().getSimpleName());
    }

    @PostPersist
    public void afterPersist(Object entity) { ... }

    @PreUpdate
    public void beforeUpdate(Object entity) { ... }
}
```

### Listener Inheritance Control

```java
@Entity
@ExcludeSuperclassListeners    // don't inherit listeners from superclass
@ExcludeDefaultListeners       // don't apply default listeners (from orm.xml)
public class ClusterNode extends BaseEntity { ... }
```

---

## :material-shield-check: 4. Bean Validation Integration

JPA automatically validates entity fields **before** `@PrePersist` and `@PreUpdate`:

```java
@Entity
public class ClusterNode {
    @NotNull
    @Size(min = 3, max = 255)
    private String hostname;

    @Min(1) @Max(2048)
    private int cpuCores;

    @Valid   // validate embedded object's constraints too
    @Embedded
    private HardwareSpec hardware;
}

@Embeddable
public class HardwareSpec {
    @Min(1)
    private int cpuCores;

    @Min(1)
    private int ramGb;
}
```

**Configure which lifecycle events trigger validation:**

```xml
<!-- persistence.xml -->
<properties>
    <property name="jakarta.persistence.validation.group.pre-persist"
              value="jakarta.validation.groups.Default"/>
    <property name="jakarta.persistence.validation.group.pre-update"
              value="jakarta.validation.groups.Default"/>
    <!-- Disable pre-remove validation (default: no validation): -->
    <property name="jakarta.persistence.validation.group.pre-remove" value=""/>
</properties>
```

### Custom Validation Constraint

```java
// Step 1: Define annotation:
@Constraint(validatedBy = ValidHostnameValidator.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidHostname {
    String message() default "Invalid hostname format";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Step 2: Implement validator:
public class ValidHostnameValidator implements ConstraintValidator<ValidHostname, String> {
    private static final Pattern HOSTNAME_PATTERN = 
        Pattern.compile("^[a-zA-Z0-9]([a-zA-Z0-9\\-]{0,61}[a-zA-Z0-9])?(\\.[a-zA-Z]{2,})+$");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        return value != null && HOSTNAME_PATTERN.matcher(value).matches();
    }
}

// Step 3: Apply:
@ValidHostname
private String hostname;
```

---

## :material-key: Key Takeaways — Chapter 12

1. **`@Version`** — the standard optimistic locking solution; let JPA manage it; never update `version` manually
2. **`OptimisticLockException`** is thrown at flush/commit — wrap in retry logic for high-contention scenarios
3. **`PESSIMISTIC_WRITE`** blocks ALL concurrent access — use only for genuinely critical operations
4. **L2 Cache** is OFF by default; `ENABLE_SELECTIVE` + `@Cacheable(true)` is the safest strategy
5. **`CacheRetrieveMode.BYPASS`** skips L2 cache — useful when you need guaranteed fresh data
6. **`@EntityListeners`** decouples auditing from business entities — the cleanest architectural pattern
7. **Bean Validation** integrates with JPA lifecycle — `@NotNull`/`@Size` annotations are validated before every INSERT/UPDATE

---

[:octicons-arrow-left-24: Back to Day 11 Index](index.md)
