---
tags: [jakarta-ee, jpa, n-plus-one, entity-graph, batch-size, join-fetch, lab, phase-3, week-3]
---

# :material-flask: Day 18 Lab Guide — JPA Performance Tuning & Batch Fetching

> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-3-BCE-Async-Performance](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-3-BCE-Async-Performance)  
> **Tech Stack:** Jakarta Persistence 3.1 | Hibernate 6.4 | `StatisticsService` | `EntityGraph` | `@BatchSize`

---

## :material-target: Laboratory Objective

Master ORM query profiling and performance optimization. Diagnose the N+1 Select Problem with Hibernate's `StatisticsService`, and eliminate unnecessary database round-trips using **JPQL `JOIN FETCH`**, **Hibernate `@BatchSize`**, **JPA 3.1 `EntityGraph`**, and **Scalar DTO Projections**.

---

## :material-compare: Fetching Strategies Comparison (N=20 nodes)

| Strategy | SQL Executions | Reduction | Best For |
|---------|---------------|-----------|---------|
| **Lazy Traversal (N+1)** | `1 + 20 = 21 queries` | Baseline | Single lookups where children rarely accessed |
| **JPQL `JOIN FETCH`** | `1 query` | -95.2% | Known query paths requiring full child graph |
| **Hibernate `@BatchSize(10)`** | `1 + ⌈20/10⌉ = 3 queries` | -85.7% | Mitigation when JOIN FETCH causes Cartesian products |
| **JPA 3.1 `EntityGraph`** | `1 query` | -95.2% | Dynamic fetch requirements varying per call site |
| **Scalar DTO Projections** | `1 query + 0 entity overhead` | -100% entity lifecycle | Read-only dashboards and reporting |

---

## :material-bug: The N+1 Trap (Baseline to Profile)

```java
// THIS CODE FIRES 21 QUERIES for 20 nodes:

// Query 1: SELECT all nodes
List<ServerNode> nodes = em.createQuery("SELECT n FROM ServerNode n", ServerNode.class)
                           .getResultList();

// Queries 2..21: Each iteration triggers a lazy SELECT for metrics collection:
for (ServerNode node : nodes) {
    List<MetricRecord> metrics = node.getMetrics();   // ← N individual SELECTs!
    processMetrics(metrics);
}
```

**Hibernate statistics confirming the problem:**
```java
// Enable in persistence.xml:
// <property name="hibernate.generate_statistics" value="true"/>

SessionFactory sf = em.unwrap(Session.class).getSessionFactory();
Statistics stats = sf.getStatistics();
stats.clear();

// Run your code...

System.out.printf("Queries executed: %d%n", stats.getPrepareStatementCount());
// Output: Queries executed: 21
```

---

## :material-speedometer: Solution 1 — JPQL `JOIN FETCH` (1 query)

```java
// Single SQL with an INNER JOIN — loads nodes + metrics in one round-trip:
List<ServerNode> nodes = em.createQuery(
    "SELECT DISTINCT n FROM ServerNode n JOIN FETCH n.metrics",
    ServerNode.class
).getResultList();

// Now iterating metrics fires ZERO additional queries:
for (ServerNode node : nodes) {
    List<MetricRecord> metrics = node.getMetrics();   // ← already loaded!
    processMetrics(metrics);
}
// stats.getPrepareStatementCount() == 1  ✅
```

!!! warning "DISTINCT prevents duplicate rows"
    Without `DISTINCT`, SQL JOIN produces duplicate parent rows (one per child). Always use `SELECT DISTINCT` with `JOIN FETCH` on collection relationships.

---

## :material-group: Solution 2 — Hibernate `@BatchSize` (3 queries for N=20)

Entity annotation — tells Hibernate to group lazy loads into `IN(...)` batches:

```java
@Entity
public class ServerNode {
    @Id private Long id;

    @OneToMany(mappedBy = "node", fetch = FetchType.LAZY)
    @BatchSize(size = 10)   // ← load at most 10 collections per SQL roundtrip
    private List<MetricRecord> metrics;
}
```

**Generated SQL for 20 nodes:**
```sql
-- Query 1: Load all nodes:
SELECT * FROM server_node;

-- Query 2: First batch of 10 nodes' metrics:
SELECT * FROM metric_record WHERE node_id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

-- Query 3: Second batch:
SELECT * FROM metric_record WHERE node_id IN (11, 12, 13, 14, 15, 16, 17, 18, 19, 20);
-- Total: 3 queries (vs 21)
```

---

## :material-graph: Solution 3 — JPA 3.1 `EntityGraph` (1 query, dynamic)

Dynamic fetch plan without changing JPQL or entity annotations:

```java
// Define the graph (or use @NamedEntityGraph):
EntityGraph<ServerNode> graph = em.createEntityGraph(ServerNode.class);
graph.addAttributeNodes("metrics");   // fetch this collection eagerly

TypedQuery<ServerNode> query = em.createQuery(
    "SELECT n FROM ServerNode n",
    ServerNode.class
);

// Inject the graph as a hint:
query.setHint("jakarta.persistence.fetchgraph", graph);
// OR for loadgraph (preserves other static EAGER mappings):
// query.setHint("jakarta.persistence.loadgraph", graph);

List<ServerNode> nodes = query.getResultList();
// stats.getPrepareStatementCount() == 1  ✅
```

**Named entity graph (declared on entity):**
```java
@Entity
@NamedEntityGraph(
    name = "ServerNode.withMetrics",
    attributeNodes = @NamedAttributeNode("metrics")
)
public class ServerNode { ... }

// Usage:
query.setHint("jakarta.persistence.fetchgraph", em.getEntityGraph("ServerNode.withMetrics"));
```

---

## :material-table-arrow-down: Solution 4 — Scalar DTO Projections (zero entity overhead)

When you only need specific columns (no full entity lifecycle):

```java
// Java 21 record as projection target:
public record NodeMetricSummary(Long nodeId, String hostname, Double avgCpu, Long metricCount) {}

// JPQL constructor expression — maps directly to record:
List<NodeMetricSummary> summaries = em.createQuery(
    "SELECT new com.pulse.node.entity.NodeMetricSummary(" +
    "  n.id, n.hostname, AVG(m.cpuUsage), COUNT(m)) " +
    "FROM ServerNode n LEFT JOIN n.metrics m " +
    "GROUP BY n.id, n.hostname",
    NodeMetricSummary.class
).getResultList();

// Results are NOT managed entities — zero dirty-checking, zero L1 cache overhead
// stats.getPrepareStatementCount() == 1  ✅
```

---

## :material-check-all: Lab Verification Checklist

| # | Scenario | Expected |
|---|----------|---------|
| 1 | **N+1 Baseline** | Unoptimized traversal triggers `21` prepared statements for 20 nodes |
| 2 | **JPQL `JOIN FETCH`** | `JOIN FETCH` reduces statements from 21 → `1` (-95.2%) |
| 3 | **Hibernate `@BatchSize`** | Batching produces `3` queries using `WHERE node_id IN (...)` (-85.7%) |
| 4 | **JPA 3.1 `EntityGraph`** | Dynamic fetch plan executes in exactly `1` query without changing JPQL |
| 5 | **Scalar DTO Projections** | `SELECT new ...` returns 60 summary records with `0` entity lifecycle overhead |

---

[:octicons-arrow-left-24: Back to Day 18 Index](index.md) | [:octicons-arrow-right-24: Day 19 — Connection Pool Tuning](../day-19-connection-pool-tuning/index.md)
