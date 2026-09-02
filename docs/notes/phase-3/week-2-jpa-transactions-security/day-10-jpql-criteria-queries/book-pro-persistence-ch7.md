---
tags: [jakarta-ee, jpa, queries, named-query, pagination, phase-3]
---

# :material-book-open-page-variant: Pro Persistence — Chapter 7: Using Queries

> **Book:** Pro Jakarta Persistence in Jakarta EE 10 (Apress, 2023)  
> **Chapter:** 7 — Using Queries

---

## :material-information: Overview

Chapter 7 covers the **query execution API** — how to create, parameterize, paginate, and execute JPQL and native SQL queries. It focuses on the **mechanics of running** queries (the "how"), not the query language syntax (see Ch8) or Criteria API (see Ch9).

---

## :material-code-braces: Dynamic Query Creation — `createQuery()`

Two variants — always prefer `TypedQuery<T>` to avoid unchecked casts:

```java
// Legacy untyped — requires explicit cast:
Query q = em.createQuery("SELECT n FROM ClusterNode n");
List<ClusterNode> nodes = (List<ClusterNode>) q.getResultList();   // ← unchecked cast

// TypedQuery (strongly preferred):
TypedQuery<ClusterNode> q = em.createQuery(
    "SELECT n FROM ClusterNode n WHERE n.status = :status",
    ClusterNode.class
);
List<ClusterNode> nodes = q.getResultList();   // no cast needed
```

---

## :material-variable: Query Parameters — Preventing SQL Injection

Never concatenate strings into queries. Always use named or positional parameters:

```java
// Named parameters (preferred — readable, order-independent):
TypedQuery<ClusterNode> q = em.createQuery(
    "SELECT n FROM ClusterNode n " +
    "WHERE n.status = :status AND n.hostname LIKE :pattern",
    ClusterNode.class
);
q.setParameter("status", NodeStatus.ACTIVE);
q.setParameter("pattern", "node-%.cluster.local");

// Positional parameters (legacy — order matters):
TypedQuery<ClusterNode> q = em.createQuery(
    "SELECT n FROM ClusterNode n WHERE n.status = ?1 AND n.rack.id = ?2",
    ClusterNode.class
);
q.setParameter(1, NodeStatus.ACTIVE);
q.setParameter(2, rackId);
```

### Setting Temporal Parameters

```java
// java.util.Date / java.util.Calendar require TemporalType hint:
q.setParameter("since", new Date(), TemporalType.TIMESTAMP);
q.setParameter("cutoffDate", new Date(), TemporalType.DATE);

// Java 8+ Instant / LocalDate work directly (no TemporalType needed):
q.setParameter("since", Instant.now().minus(Duration.ofDays(7)));
```

---

## :material-bookmark: Named Queries — `@NamedQuery`

Pre-compiled at **deployment time** — JPQL errors are caught before the first request arrives, not during production runtime:

```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "ClusterNode.findByStatus",
        query = "SELECT n FROM ClusterNode n WHERE n.status = :status ORDER BY n.hostname"
    ),
    @NamedQuery(
        name = "ClusterNode.countByRack",
        query = "SELECT COUNT(n) FROM ClusterNode n WHERE n.rack.id = :rackId"
    ),
    @NamedQuery(
        name = "ClusterNode.findAll",
        query = "SELECT n FROM ClusterNode n ORDER BY n.hostname ASC"
    )
})
public class ClusterNode {
    // Entity fields...
}
```

Execution:

```java
// Standard usage:
List<ClusterNode> activeNodes = em
    .createNamedQuery("ClusterNode.findByStatus", ClusterNode.class)
    .setParameter("status", NodeStatus.ACTIVE)
    .getResultList();

// Count query:
Long count = em
    .createNamedQuery("ClusterNode.countByRack", Long.class)
    .setParameter("rackId", 5L)
    .getSingleResult();
```

!!! tip "Naming convention"
    Use `EntityName.descriptiveName` — e.g., `ClusterNode.findByStatus`. This groups all queries for an entity together and makes them easy to discover.

---

## :material-format-list-text: Query Result Retrieval Methods

| Method | Returns | When Empty | Exception Cases |
|--------|---------|-----------|----------------|
| `getResultList()` | `List<T>` | Empty `List` (never null) | None |
| `getSingleResult()` | Single `T` | `NoResultException` | `NonUniqueResultException` if multiple found |
| `getResultStream()` | `Stream<T>` | Empty `Stream` | None |
| `executeUpdate()` | `int` rows affected | `0` | `TransactionRequiredException` |

```java
// Safe single result without exception risk:
List<ClusterNode> results = em.createNamedQuery("ClusterNode.findByStatus", ClusterNode.class)
    .setParameter("status", NodeStatus.ACTIVE)
    .setMaxResults(1)
    .getResultList();
ClusterNode node = results.isEmpty() ? null : results.get(0);

// Stream processing for large datasets:
em.createQuery("SELECT n FROM ClusterNode n", ClusterNode.class)
  .getResultStream()
  .filter(n -> n.getHardware().getRamGb() >= 64)
  .map(ClusterNode::getHostname)
  .forEach(System.out::println);
```

---

## :material-page-next: Query Pagination — `setFirstResult()` / `setMaxResults()`

```java
public List<ClusterNode> findPage(NodeStatus status, int page, int pageSize) {
    return em.createNamedQuery("ClusterNode.findByStatus", ClusterNode.class)
             .setParameter("status", status)
             .setFirstResult(page * pageSize)   // 0-indexed starting position
             .setMaxResults(pageSize)            // max records to return
             .getResultList();
}

// Page 0 (first): setFirstResult(0)
// Page 1:         setFirstResult(pageSize)
// Page 2:         setFirstResult(2 * pageSize)
```

!!! danger "Never paginate queries with collection JOINs"
    Joining a collection relationship (`@OneToMany`) produces **duplicate parent rows** in results (one per child). `setFirstResult`/`setMaxResults` operates on raw DB rows — not on logical entity count. This produces **incorrect pagination**.

    **Solution:** Use a separate count query + sub-select, or fetch with Criteria API on the root entity only.

---

## :material-lightning-bolt: Bulk Update & Delete — `executeUpdate()`

Operates **directly on the database** without touching the persistence context — much faster for large batch operations:

```java
// Bulk update — set all nodes in a rack to MAINTENANCE:
int updatedCount = em.createQuery(
    "UPDATE ClusterNode n SET n.status = :newStatus " +
    "WHERE n.rack.id = :rackId AND n.status = :oldStatus"
)
.setParameter("newStatus",  NodeStatus.MAINTENANCE)
.setParameter("rackId",     10L)
.setParameter("oldStatus",  NodeStatus.ACTIVE)
.executeUpdate();

System.out.println("Nodes updated: " + updatedCount);

// Bulk delete — purge old telemetry records:
int deletedCount = em.createQuery(
    "DELETE FROM TelemetryRecord t WHERE t.recordedAt < :cutoff"
)
.setParameter("cutoff", Instant.now().minus(Duration.ofDays(90)))
.executeUpdate();
```

!!! warning "Bulk operations bypass the L1 cache"
    Any `ClusterNode` already loaded in the current persistence context will **NOT** see the bulk update. The in-memory entity still has the old status. Call `em.clear()` immediately after bulk operations if you need fresh entity state.

    ```java
    em.createQuery("UPDATE ClusterNode n SET n.status = :s WHERE n.rack.id = :r")
      .setParameter("s", NodeStatus.MAINTENANCE).setParameter("r", 10L)
      .executeUpdate();
    em.clear();   // ← evict all stale entities from context
    ```

---

## :material-code-json: Native SQL Queries

Escape hatch for DB-specific features (window functions, JSONB operators, etc.) not available in JPQL:

```java
// Native SQL returning entity:
List<ClusterNode> nodes = em.createNativeQuery(
    "SELECT * FROM cluster_node WHERE pg_column_size(hostname) > :minBytes",
    ClusterNode.class
)
.setParameter("minBytes", 20)
.getResultList();

// Native SQL returning scalar:
List<Object[]> rows = em.createNativeQuery(
    "SELECT hostname, cpu_count FROM cluster_node ORDER BY cpu_count DESC LIMIT 10"
).getResultList();
for (Object[] row : rows) {
    String hostname = (String) row[0];
    Integer cpuCount = (Integer) row[1];
}
```

---

## :material-hint: Query Hints

Configure provider-specific behaviour per query:

```java
TypedQuery<ClusterNode> q = em.createQuery("SELECT n FROM ClusterNode n", ClusterNode.class);

// Timeout — abort if query takes longer than 5000ms:
q.setHint("jakarta.persistence.query.timeout", 5000);

// L2 cache control:
q.setHint("jakarta.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
q.setHint("jakarta.persistence.cache.storeMode",    CacheStoreMode.REFRESH);
```

---

## :material-key: Key Takeaways — Chapter 7

1. **Always use `TypedQuery<T>`** — eliminates unchecked casts and improves type safety
2. **Named parameters** (`":name"`) over positional — more readable and injection-safe
3. **`@NamedQuery`** — compiled at deployment time; bugs caught before runtime
4. **`getSingleResult()`** throws on empty/multiple results — prefer `getResultList()` + check size for safety
5. **Pagination** (`setFirstResult`/`setMaxResults`) — never on queries with collection JOINs
6. **Bulk `executeUpdate()`** — bypasses L1 cache; always `em.clear()` after if entities are still in context
7. **Native SQL** — last resort for DB-specific features; prefer JPQL/Criteria API when possible

---

[:octicons-arrow-left-24: Back to Day 10 Index](index.md) | [:octicons-arrow-right-24: Ch8 — JPQL Language](book-pro-persistence-ch8.md)
