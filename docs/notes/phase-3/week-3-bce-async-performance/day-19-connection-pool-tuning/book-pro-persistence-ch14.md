---
tags: [jakarta-ee, jpa, persistence-xml, deployment, datasource, hikaricp, phase-3]
---

# :material-book-open-page-variant: Pro Jakarta Persistence — Chapter 14: Packaging and Deployment

> **Book:** Pro Jakarta Persistence in Jakarta EE 10 (Apress, 2023)  
> **Chapter:** 14 — Packaging and Deployment

---

## :material-file-document: 1. `persistence.xml` Configuration

The primary runtime configuration for a JPA persistence unit. Location: `META-INF/persistence.xml` within the deployment archive.

### Basic Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             version="3.1">

    <persistence-unit name="clusterPU" transaction-type="JTA">
        <jta-data-source>java:app/jdbc/ClusterDS</jta-data-source>

        <!-- Optional: override the container's default JPA provider: -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!-- Classes discovered automatically from the same JAR.
             Alternatively list explicitly: -->
        <!-- <class>com.pulse.node.entity.ClusterNode</class> -->

        <!-- Exclude auto-discovery (for explicit control): -->
        <!-- <exclude-unlisted-classes>true</exclude-unlisted-classes> -->

        <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>

        <properties>
            <!-- Schema generation: -->
            <property name="jakarta.persistence.schema-generation.database.action"
                      value="drop-and-create"/>

            <!-- Hibernate tuning: -->
            <property name="hibernate.show_sql"                  value="true"/>
            <property name="hibernate.format_sql"                value="true"/>
            <property name="hibernate.generate_statistics"       value="true"/>
            <property name="hibernate.default_batch_fetch_size"  value="10"/>
        </properties>
    </persistence-unit>
</persistence>
```

### Transaction Type: `JTA` vs `RESOURCE_LOCAL`

| Setting | `JTA` | `RESOURCE_LOCAL` |
|---------|-------|----------------|
| Environment | Jakarta EE managed server | Java SE / standalone |
| Transaction control | Container manages `begin/commit/rollback` | Application calls `EntityTransaction.begin()` |
| DataSource config | `<jta-data-source>` (JNDI) | JDBC properties (`jakarta.persistence.jdbc.*`) |
| `@Transactional` works? | ✅ Yes | ❌ No — must use `em.getTransaction()` |

---

## :material-database: 2. DataSource Configuration

### JTA DataSource (Server Environment)

```xml
<persistence-unit name="clusterPU" transaction-type="JTA">
    <!-- JNDI name bound to a container-managed connection pool: -->
    <jta-data-source>java:app/jdbc/ClusterDS</jta-data-source>

    <!-- Optional: non-JTA secondary source for optimized reads: -->
    <non-jta-data-source>java:app/jdbc/ClusterReadDS</non-jta-data-source>
</persistence-unit>
```

### JDBC Properties (Java SE / Resource Local)

```xml
<persistence-unit name="clusterPU" transaction-type="RESOURCE_LOCAL">
    <properties>
        <property name="jakarta.persistence.jdbc.driver"   value="org.h2.Driver"/>
        <property name="jakarta.persistence.jdbc.url"      value="jdbc:h2:mem:clusterdb;DB_CLOSE_DELAY=-1"/>
        <property name="jakarta.persistence.jdbc.user"     value="sa"/>
        <property name="jakarta.persistence.jdbc.password" value=""/>
    </properties>
</persistence-unit>
```

---

## :material-folder-open: 3. Packaging Structures

### WAR Packaging (Most Common for Jakarta EE Web Apps)

```
my-cluster-app.war
├── WEB-INF/
│   ├── classes/
│   │   ├── META-INF/
│   │   │   └── persistence.xml      ← PU root is WEB-INF/classes
│   │   └── com/pulse/node/
│   │       ├── entity/ClusterNode.class
│   │       └── boundary/NodeResource.class
│   └── lib/
│       └── hikari-pool.jar
└── index.html
```

### Persistence Archive (Reusable JAR)

```
cluster-persistence.jar   ← placed in WEB-INF/lib/ or EAR lib/
├── META-INF/
│   └── persistence.xml
└── com/pulse/node/entity/
    ├── ClusterNode.class
    └── TelemetryRecord.class
```

### EAR Packaging

```
cluster-platform.ear
├── META-INF/application.xml
├── cluster-web.war
├── cluster-ejb.jar
│   └── META-INF/persistence.xml   ← PU scoped to EJB module
└── lib/
    └── cluster-domain.jar
```

### Additional JARs in Persistence Unit

```xml
<!-- Reference other JARs containing entity classes: -->
<jar-file>../lib/cluster-domain.jar</jar-file>
<jar-file>../lib/audit-entities.jar</jar-file>
```

---

## :material-schema: 4. Schema Generation

### DDL Generation Actions

```xml
<properties>
    <!-- Database action: none | create | drop | drop-and-create -->
    <property name="jakarta.persistence.schema-generation.database.action"
              value="drop-and-create"/>

    <!-- Script generation (optional): -->
    <property name="jakarta.persistence.schema-generation.scripts.action"
              value="drop-and-create"/>
    <property name="jakarta.persistence.schema-generation.scripts.create-target"
              value="/tmp/create-schema.sql"/>
    <property name="jakarta.persistence.schema-generation.scripts.drop-target"
              value="/tmp/drop-schema.sql"/>

    <!-- Seed data on startup: -->
    <property name="jakarta.persistence.sql-load-script-source"
              value="META-INF/initial-data.sql"/>
</properties>
```

### Schema Generation Source Order

```xml
<!-- Use metadata (annotations) first, then SQL script: -->
<property name="jakarta.persistence.schema-generation.create-source"
          value="metadata-then-script"/>
<property name="jakarta.persistence.schema-generation.scripts.create-script-source"
          value="META-INF/supplemental-schema.sql"/>
```

### DDL Annotations Affecting Schema

```java
@Entity
@Table(
    name = "cluster_node",
    uniqueConstraints = @UniqueConstraint(columnNames = {"hostname"}),
    indexes = @Index(name = "idx_node_status", columnList = "status")
)
public class ClusterNode {

    @Column(
        name     = "hostname",
        length   = 255,
        nullable = false,
        unique   = true
    )
    private String hostname;

    @Column(
        name              = "cpu_cores",
        columnDefinition  = "SMALLINT"   // override generated DDL type
    )
    private int cpuCores;

    @OneToMany(
        mappedBy = "node",
        cascade  = CascadeType.ALL
    )
    @JoinColumn(foreignKey = @ForeignKey(
        value                = ConstraintMode.CONSTRAINT,
        foreignKeyDefinition = "FOREIGN KEY (node_id) REFERENCES cluster_node(id) ON DELETE CASCADE"
    ))
    private List<TelemetryRecord> records;
}
```

---

## :material-server-network: 5. HikariCP Connection Pool Properties

HikariCP is the default JDBC connection pool in most modern Jakarta EE / Quarkus / Spring Boot applications. Configure via `persistence.xml` properties or a `DataSource` resource definition.

```xml
<persistence-unit name="clusterPU">
    <properties>
        <!-- HikariCP-specific tuning: -->

        <!-- Max connections in pool (see sizing formula below): -->
        <property name="hibernate.hikari.maximumPoolSize"   value="16"/>

        <!-- Min idle connections kept alive: -->
        <property name="hibernate.hikari.minimumIdle"       value="4"/>

        <!-- Max time to wait for a connection (ms); throws SQLTimeoutException: -->
        <property name="hibernate.hikari.connectionTimeout" value="3000"/>

        <!-- Max time a connection can sit idle before removal (ms): -->
        <property name="hibernate.hikari.idleTimeout"       value="600000"/>   <!-- 10 min -->

        <!-- Max lifetime of a connection in pool (ms); must be less than DB wait_timeout: -->
        <property name="hibernate.hikari.maxLifetime"       value="1800000"/>  <!-- 30 min -->

        <!-- Pool name for JMX/logging: -->
        <property name="hibernate.hikari.poolName"          value="ClusterNodePool"/>

        <!-- Connection validation query: -->
        <property name="hibernate.hikari.connectionTestQuery" value="SELECT 1"/>
    </properties>
</persistence-unit>
```

### HikariCP Sizing Formula

```
Optimal Pool Size = (CPU_cores × 2) + effective_spindle_count
```

| CPU Cores | Spindles | Recommended Pool Size |
|-----------|---------|----------------------|
| 4 | 1 (SSD) | (4 × 2) + 1 = **9** |
| 8 | 1 (SSD) | (8 × 2) + 1 = **17** |
| 16 | 1 (SSD) | (16 × 2) + 1 = **33** |

!!! warning "Bigger pool ≠ better throughput"
    Beyond the optimal size, additional connections compete for the same disk I/O bandwidth and cause OS scheduler overhead (context switching). HikariCP's own benchmarks show performance actually *decreasing* with oversized pools.

---

## :material-key: Key Takeaways — Chapter 14

1. **`persistence.xml`** lives in `META-INF/` — mandatory for every persistence unit
2. **`transaction-type="JTA"`** for server environments; **`RESOURCE_LOCAL`** for Java SE
3. **`<jta-data-source>`** = JNDI name bound by the container; never hardcode JDBC URLs in JTA mode
4. **Schema generation `drop-and-create`** is convenient for development — NEVER use in production
5. **`sql-load-script-source`** seeds initial data after schema creation — great for dev/test environments
6. **HikariCP sizing formula**: `(CPU_cores × 2) + spindle_count` — more connections can reduce performance
7. **`connectionTimeout`** = fail fast with `SQLTimeoutException` rather than queuing indefinitely

---

[:octicons-arrow-left-24: Back to Day 19 Index](index.md)
