---
tags: [jakarta-ee, jpa, jpql, criteria-api, lab, phase-3, week-2]
---

# :material-flask: Day 10 Lab Guide — Dynamic JPQL Queries, Projections & Criteria API

> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-2-JPA-JTA-Security](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-2-JPA-JTA-Security)  
> **Tech Stack:** Jakarta Persistence 3.1 | Hibernate ORM 6.4 | H2 In-Memory Engine

---

## :material-target: Laboratory Objective

Master enterprise query construction: `@NamedQuery` static pre-compiled definitions, DTO projections via `SELECT NEW`, N+1 elimination with `JOIN FETCH`, aggregation with `GROUP BY`/`HAVING`, and type-safe Criteria API dynamic queries with pagination.

---

## :material-cube-outline: Key Technical Concepts

### 1. Static Named Queries — `@NamedQuery`

Declared on entity class, compiled at deployment time (safer than runtime strings):

```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "ClusterNode.findByStatus",
        query = "SELECT n FROM ClusterNode n WHERE n.status = :status ORDER BY n.hostname ASC"
    ),
    @NamedQuery(
        name = "ClusterNode.findOverloadedNodes",
        query = "SELECT n FROM ClusterNode n " +
                "JOIN n.telemetryRecords t " +
                "GROUP BY n " +
                "HAVING COUNT(t) > :minRecords AND AVG(t.cpuUsagePercent) > :minCpu"
    )
})
public class ClusterNode { ... }
```

Execution:

```java
List<ClusterNode> activeNodes = em.createNamedQuery("ClusterNode.findByStatus", ClusterNode.class)
    .setParameter("status", NodeStatus.ACTIVE)
    .getResultList();
```

### 2. DTO Projections — `SELECT NEW`

Bypass entity hydration and First-Level Cache — ideal for read-only reporting:

```java
// Immutable DTO record:
public record NodeMetricSummaryDto(String hostname, Long recordCount, Double avgCpu) {}

// JPQL constructor expression:
List<NodeMetricSummaryDto> summaries = em.createQuery(
    "SELECT NEW com.pulse.dto.NodeMetricSummaryDto(n.hostname, COUNT(t), AVG(t.cpuUsagePercent)) " +
    "FROM ClusterNode n LEFT JOIN n.telemetryRecords t " +
    "GROUP BY n.hostname",
    NodeMetricSummaryDto.class
).getResultList();
```

!!! tip "Why SELECT NEW beats loading full entities"
    - No entity tracked in L1 cache (lower memory)
    - No `LazyInitializationException` risk
    - Transfers only requested columns from DB — faster for large datasets

### 3. Eliminating N+1 Selects — `JOIN FETCH`

**Without `JOIN FETCH`** (N+1 problem):
```sql
-- 1 query to get nodes:
SELECT * FROM cluster_node WHERE status = 'ACTIVE'

-- Then N separate queries for each node's telemetry:
SELECT * FROM telemetry_record WHERE node_id = 1
SELECT * FROM telemetry_record WHERE node_id = 2
...
SELECT * FROM telemetry_record WHERE node_id = N
```

**With `JOIN FETCH`** (1 query total):
```java
List<ClusterNode> nodes = em.createQuery(
    "SELECT DISTINCT n FROM ClusterNode n " +
    "JOIN FETCH n.telemetryRecords t " +
    "JOIN FETCH n.rack r " +
    "WHERE n.status = :status",
    ClusterNode.class
)
.setParameter("status", NodeStatus.ACTIVE)
.getResultList();

em.close();   // After close, still safe to traverse — already initialized!
nodes.get(0).getTelemetryRecords().size();   // Works! No LazyInitializationException
```

!!! warning "DISTINCT required with JOIN FETCH on collections"
    Without `DISTINCT`, JPA returns duplicate parent records (one per child row). Always add `DISTINCT` when join-fetching `@OneToMany` or `@ManyToMany` collections.

### 4. Aggregation — `GROUP BY` + `HAVING`

```java
// Find overloaded nodes with >10 telemetry records and avg CPU > 80%:
List<Object[]> overloaded = em.createQuery(
    "SELECT n.hostname, COUNT(t), AVG(t.cpuUsagePercent) " +
    "FROM ClusterNode n JOIN n.telemetryRecords t " +
    "GROUP BY n.hostname " +
    "HAVING COUNT(t) > :minRecords AND AVG(t.cpuUsagePercent) > :minCpu",
    Object[].class
)
.setParameter("minRecords", 10L)
.setParameter("minCpu", 80.0)
.getResultList();
```

### 5. Criteria API — Dynamic Multi-Field Search

```java
public List<ClusterNode> searchNodes(NodeSearchCriteria criteria, int page, int pageSize) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<ClusterNode> cq = cb.createQuery(ClusterNode.class);
    Root<ClusterNode> node = cq.from(ClusterNode.class);

    List<Predicate> predicates = new ArrayList<>();

    // Only add predicates for non-null criteria:
    if (criteria.getStatus() != null) {
        predicates.add(cb.equal(node.get("status"), criteria.getStatus()));
    }
    if (criteria.getMinRamGb() != null) {
        predicates.add(cb.ge(node.get("hardware").get("ramGb"), criteria.getMinRamGb()));
    }
    if (criteria.getHostnameContains() != null) {
        predicates.add(cb.like(node.get("hostname"), "%" + criteria.getHostnameContains() + "%"));
    }
    if (criteria.getRackId() != null) {
        Join<ClusterNode, ServerRack> rack = node.join("rack", JoinType.INNER);
        predicates.add(cb.equal(rack.get("id"), criteria.getRackId()));
    }

    cq.select(node)
      .where(cb.and(predicates.toArray(new Predicate[0])))
      .orderBy(cb.asc(node.get("hostname")));

    return em.createQuery(cq)
             .setFirstResult(page * pageSize)     // pagination offset
             .setMaxResults(pageSize)              // page size limit
             .getResultList();
}
```

---

## :material-check-all: Lab Verification Checklist

| # | Test | Expected Result |
|---|------|----------------|
| 1 | **Named Queries** | `ClusterNode.findByStatus` returns filtered nodes with parameter binding |
| 2 | **DTO Projections** | `NodeMetricSummaryDto` populated with aggregated averages without loading entities into L1 cache |
| 3 | **JOIN FETCH eliminates N+1** | Fetched nodes remain accessible after `em.close()` — no `LazyInitializationException` |
| 4 | **Aggregates with HAVING** | `ClusterNode.findOverloadedNodes` isolates nodes matching count and CPU threshold |
| 5 | **Criteria API Pagination** | Multi-predicate search returns correct count and properly sliced paginated results |

---

[:octicons-arrow-left-24: Back to Day 10 Index](index.md) | [:octicons-arrow-right-24: Day 11 — JTA Transactions & Concurrency](../day-11-jta-transactions-concurrency/index.md)
