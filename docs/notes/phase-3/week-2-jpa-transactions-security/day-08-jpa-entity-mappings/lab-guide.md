---
tags: [jakarta-ee, jpa, entity-mapping, relationships, lab, phase-3, week-2]
---

# :material-flask: Day 08 Lab Guide — JPA 3.1 Entity Mappings & Relationship Boundaries

> **Lab Repo:** [:material-github: 7amo10/JavaEE-Labs — Week-2-JPA-JTA-Security](https://github.com/7amo10/JavaEE-Labs/tree/main/Week-2-JPA-JTA-Security)  
> **Tech Stack:** Jakarta Persistence 3.1 | Hibernate ORM 6.4 | H2 In-Memory Engine

---

## :material-target: Laboratory Objective

Master Jakarta Persistence 3.1 (JPA 3.1) entity modeling, embedded value objects, element collections, all four relational cardinalities (1:1, 1:N, N:1, N:M), cascade lifecycle propagation, orphan removal semantics, and persistence context boundaries (`LazyInitializationException`).

---

## :material-cube-outline: Key Technical Concepts Covered

### Entity Foundations

```java
@Entity
@Table(name = "cluster_node", schema = "telemetry")
public class ClusterNode {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "node_hostname", nullable = false, length = 255)
    private String hostname;

    @Enumerated(EnumType.STRING)   // store "ACTIVE" not "0"
    private NodeStatus status;
}
```

### Enumerations & Embedded Value Types

```java
@Embeddable
public class HardwareSpec {
    @Column(name = "cpu_cores")     private int cpuCores;
    @Column(name = "ram_gb")        private int ramGb;
    @Column(name = "storage_tb")    private double storageTb;
}

// Used in entity:
@Embedded
private HardwareSpec hardware;

// @AttributeOverride allows column renaming per entity:
@AttributeOverride(name = "cpuCores", column = @Column(name = "hw_cpu_count"))
```

### Element Collections — `Set<String>` without an Entity

```java
@ElementCollection(fetch = FetchType.LAZY)
@CollectionTable(
    name = "node_tags",
    joinColumns = @JoinColumn(name = "node_id")
)
@Column(name = "tag_value")
private Set<String> tags = new HashSet<>();
```

### Relationship Mapping Matrix

| Cardinality | Owning Side | `@JoinColumn` Location | `mappedBy` |
|-------------|-------------|----------------------|-----------|
| `@OneToOne` | `ClusterNode` | `ClusterNode.diagnostics` | `NodeDiagnostics.node` |
| `@ManyToOne` | `ClusterNode` | `ClusterNode.rack` FK | — |
| `@OneToMany` | `ServerRack` (via `mappedBy`) | `ClusterNode.rack_id` | `rack.nodes` |
| `@ManyToMany` | `ClusterNode` | `@JoinTable("node_security_group")` | `SecurityGroup.nodes` |

```java
// One-to-One — owning side
@OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "diagnostics_id")
private NodeDiagnostics diagnostics;

// One-to-Many — parent side
@OneToMany(mappedBy = "rack", cascade = CascadeType.ALL, orphanRemoval = true)
private List<ClusterNode> nodes = new ArrayList<>();

// Many-to-One — child side
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "rack_id", nullable = false)
private ServerRack rack;

// Many-to-Many — owning side with join table
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
@JoinTable(
    name = "node_security_group",
    joinColumns = @JoinColumn(name = "node_id"),
    inverseJoinColumns = @JoinColumn(name = "group_id")
)
private Set<SecurityGroup> securityGroups = new HashSet<>();
```

### Cascade & orphanRemoval

```java
// CascadeType.ALL: persist/merge/remove/refresh/detach all cascade to children
// orphanRemoval = true: removing from collection auto-deletes child row
ServerRack rack = new ServerRack("RACK-01");
ClusterNode node = new ClusterNode("node-1.cluster.local");
rack.addNode(node);        // adds to collection
em.persist(rack);          // persists rack + node via CascadeType.ALL

rack.removeNode(node);     // orphanRemoval fires a DELETE on commit
```

---

## :material-alert: Persistence Boundary Trap — `LazyInitializationException`

!!! danger "LazyInitializationException — the #1 JPA mistake"
    Accessing a `LAZY` collection **after** closing the `EntityManager` throws `LazyInitializationException` (Hibernate) or undefined behavior (other providers). The proxy cannot load data without an open persistence context.

    ```java
    EntityManager em = emf.createEntityManager();
    em.getTransaction().begin();
    ClusterNode node = em.find(ClusterNode.class, 1L);
    em.getTransaction().commit();
    em.close();   // ← context closed here

    // DANGER — telemetryRecords is LAZY and context is closed:
    node.getTelemetryRecords().size();  // throws LazyInitializationException!
    ```

    **Fix options:**
    1. Access inside transaction: `node.getTelemetryRecords().size()` before `em.close()`
    2. Use `JOIN FETCH` in JPQL (Day 10)
    3. Change to `FetchType.EAGER` (only if ALWAYS needed)

---

## :material-check-all: Lab Verification Checklist

Run `mvn clean compile exec:java` and confirm all of the following:

| # | Test | Expected Result |
|---|------|----------------|
| 1 | **Cascade Persist** | Persisting `ServerRack` auto-inserts `ClusterNode`, `NodeDiagnostics`, `TelemetryRecord`, `SecurityGroup` |
| 2 | **Graph Traversal** | All properties (embedded, element collections, 1:1, 1:N, N:1, N:M) fetched inside active context |
| 3 | **Orphan Removal** | Removing `TelemetryRecord` from node's list executes a database `DELETE` on commit |
| 4 | **Detached Boundary Failure** | Accessing lazy collections after `em.close()` raises `LazyInitializationException` |
| 5 | **Cascade Delete** | Removing `ServerRack` deletes all dependent nodes while preserving independent `SecurityGroup` rows |

---

[:octicons-arrow-left-24: Back to Day 08 Index](index.md) | [:octicons-arrow-right-24: Day 09 — Persistence Context & Lifecycle](../day-09-persistence-context-lifecycle/index.md)
