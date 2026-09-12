---
tags: [jakarta-ee, jpa, native-sql, stored-procedures, entity-graph, phase-3]
---

# :material-book-open-page-variant: Pro Jakarta Persistence — Chapter 11: Advanced Queries

> **Book:** Pro Jakarta Persistence in Jakarta EE 10 (Apress, 2023)  
> **Chapter:** 11 — Advanced Queries: Native SQL, Stored Procedures, Entity Graphs

---

## :material-database: 1. Native SQL Queries

Native queries bypass JPQL and execute raw SQL directly — necessary for database-specific features (e.g., Oracle `START WITH ... CONNECT BY`, PostgreSQL `LATERAL JOIN`) that JPQL cannot express.

### Dynamic Native Query

```java
@Stateless
public class OrgStructureBean {

    private static final String HIERARCHY_SQL =
        "SELECT emp_id, name, salary, manager_id, dept_id, address_id " +
        "FROM emp " +
        "START WITH manager_id = ? " +
        "CONNECT BY PRIOR emp_id = manager_id";   // Oracle-specific

    @PersistenceContext EntityManager em;

    @SuppressWarnings("unchecked")
    public List<Employee> findReportingTo(int managerId) {
        return em.createNativeQuery(HIERARCHY_SQL, Employee.class)
                 .setParameter(1, managerId)     // positional parameter
                 .getResultList();
    }
}
```

!!! important "Entities from native queries are MANAGED"
    Results returned by `createNativeQuery(sql, EntityClass.class)` are **managed** by the persistence context — the same as JPQL results. You **must** select all columns that JPA needs for the entity; missing columns default to `null` and may overwrite good data on commit.

### Named Native Query — `@NamedNativeQuery`

```java
@Entity
@NamedNativeQuery(
    name    = "Employee.hierarchyUnder",
    query   = "SELECT emp_id, name, salary, manager_id, dept_id, address_id " +
              "FROM emp START WITH manager_id = ? CONNECT BY PRIOR emp_id = manager_id",
    resultClass = Employee.class
)
public class Employee { ... }

// Execution — identical to JPQL named query:
List<Employee> result = em.createNamedQuery("Employee.hierarchyUnder", Employee.class)
                          .setParameter(1, managerId)
                          .getResultList();
```

### DML via Native SQL (Data Manipulation)

```java
// Direct SQL DELETE — bypasses persistence context; use with caution:
em.createNativeQuery("DELETE FROM message_log WHERE created_at < ?")
  .setParameter(1, cutoffDate)
  .executeUpdate();
```

!!! warning "Cache consistency"
    DML executed via native queries bypasses JPA's dirty-checking and first-level cache. After bulk DML, call `em.clear()` to evict stale entities from the L1 cache.

---

## :material-table: 2. SQL Result Set Mapping — `@SqlResultSetMapping`

Used when a native query's column names don't match entity field names, or when mapping to multiple entities or scalar types.

### Mapping Multiple Entities

```java
@SqlResultSetMapping(
    name = "EmployeeWithAddress",
    entities = {
        @EntityResult(entityClass = Employee.class),
        @EntityResult(entityClass = Address.class)
    }
)
```

### Column Aliases with `@FieldResult`

```java
@SqlResultSetMapping(
    name = "EmployeeAliased",
    entities = {
        @EntityResult(
            entityClass = Employee.class,
            fields = {
                @FieldResult(name = "id",     column = "EMP_ID"),
                @FieldResult(name = "name",   column = "FULL_NAME"),
                @FieldResult(name = "salary", column = "ANNUAL_SALARY")
            }
        )
    }
)
```

### Scalar Columns — `@ColumnResult`

```java
@SqlResultSetMapping(
    name = "NameAndManagerName",
    columns = {
        @ColumnResult(name = "EMP_NAME"),
        @ColumnResult(name = "MANAGER_NAME")
    }
)
```

### Constructor Mapping — `@ConstructorResult`

Maps results directly into a non-entity class via constructor — ideal for read-only projections:

```java
// Projection class:
public class EmployeeDetails {
    public EmployeeDetails(String name, Long salary, String deptName) { ... }
}

// Mapping:
@SqlResultSetMapping(
    name = "EmployeeDetailMapping",
    classes = {
        @ConstructorResult(
            targetClass = EmployeeDetails.class,
            columns = {
                @ColumnResult(name = "name"),
                @ColumnResult(name = "salary",   type = Long.class),
                @ColumnResult(name = "dept_name")
            }
        )
    }
)
```

### Compound/Embedded Key Mapping

For entities with `@IdClass` or `@EmbeddedId`, specify each part of the key:

```java
// For @IdClass with country + id:
@FieldResult(name = "country", column = "COUNTRY"),
@FieldResult(name = "id",      column = "EMP_ID"),

// For foreign key relationship manager:
@FieldResult(name = "manager.country", column = "MGR_COUNTRY"),
@FieldResult(name = "manager.id",      column = "MGR_ID"),
```

---

## :material-database-cog: 3. Stored Procedures

JPA provides the `StoredProcedureQuery` interface to call database stored procedures portably.

### Dynamic Stored Procedure Call

```java
StoredProcedureQuery q = em.createStoredProcedureQuery("update_employee_salary");

// Register parameters:
q.registerStoredProcedureParameter("emp_id",    Long.class,   ParameterMode.IN);
q.registerStoredProcedureParameter("new_salary", Double.class, ParameterMode.IN);
q.registerStoredProcedureParameter("updated",    Boolean.class, ParameterMode.OUT);

// Set values and execute:
q.setParameter("emp_id",     employeeId);
q.setParameter("new_salary", newSalary);
q.execute();

// Retrieve OUT parameter:
boolean wasUpdated = (Boolean) q.getOutputParameterValue("updated");
```

### `REF_CURSOR` — Returning Result Sets

```java
StoredProcedureQuery q = em.createStoredProcedureQuery("fetch_active_employees", Employee.class);
q.registerStoredProcedureParameter("result_set", void.class, ParameterMode.REF_CURSOR);
List<Employee> employees = (List<Employee>) q.getResultList();   // implicitly executes
```

### Named Stored Procedure — `@NamedStoredProcedureQuery`

```java
@NamedStoredProcedureQuery(
    name          = "Employee.fetchByDept",
    procedureName = "fetch_dept_employees",
    parameters    = {
        @StoredProcedureParameter(name = "dept_id",    type = Long.class,  mode = ParameterMode.IN),
        @StoredProcedureParameter(name = "employees",  type = void.class,  mode = ParameterMode.REF_CURSOR)
    },
    resultClasses = Employee.class
)

// Usage:
StoredProcedureQuery q = em.createNamedStoredProcedureQuery("Employee.fetchByDept");
q.setParameter("dept_id", deptId);
List<Employee> result = q.getResultList();
```

---

## :material-graph: 4. Entity Graphs — Dynamic Fetch Plans

Entity Graphs are **templates** that override the static `FetchType` configuration of an entity **at query time** — without modifying entity annotations. This allows different call sites to request different levels of eagerness.

### Annotation-Based Graph — `@NamedEntityGraph`

```java
@Entity
@NamedEntityGraph(
    name           = "Employee.withPhonesAndDept",
    attributeNodes = {
        @NamedAttributeNode("name"),
        @NamedAttributeNode(value = "phones",     subgraph = "phone-subgraph"),
        @NamedAttributeNode(value = "department", subgraph = "dept-subgraph")
    },
    subgraphs = {
        @NamedSubgraph(name = "phone-subgraph", attributeNodes = {
            @NamedAttributeNode("number"),
            @NamedAttributeNode("type")
        }),
        @NamedSubgraph(name = "dept-subgraph", attributeNodes = {
            @NamedAttributeNode("name")
        })
    }
)
public class Employee { ... }
```

### Programmatic Entity Graph API

```java
EntityGraph<Employee> graph = em.createEntityGraph(Employee.class);
graph.addAttributeNodes("name", "salary");

Subgraph<Address> addressSubgraph = graph.addSubgraph("address");
addressSubgraph.addAttributeNodes("street", "city");

// Inheritance — targeting a specific subclass:
graph.addSubclassSubgraph(ContractEmployee.class).addAttributeNodes("hourlyRate");
```

### Fetch Graph vs Load Graph

```java
// FETCH GRAPH — ONLY explicitly listed attributes are EAGER:
// Everything NOT listed is treated as LAZY (overrides static EAGER mappings!)
query.setHint("jakarta.persistence.fetchgraph", em.getEntityGraph("Employee.withPhonesAndDept"));

// LOAD GRAPH — listed attributes are EAGER; unlisted inherit their static @FetchType:
query.setHint("jakarta.persistence.loadgraph", em.getEntityGraph("Employee.withPhonesAndDept"));
```

| Hint | Listed Attributes | Unlisted Attributes |
|------|-------------------|---------------------|
| `fetchgraph` | **EAGER** | **LAZY** (overrides any static EAGER) |
| `loadgraph` | **EAGER** | Inherits static `FetchType` mapping |

### Using EntityGraph with `find()`

```java
Map<String, Object> hints = new HashMap<>();
hints.put("jakarta.persistence.fetchgraph", em.getEntityGraph("Employee.withPhonesAndDept"));

Employee emp = em.find(Employee.class, empId, hints);
// Now emp.getPhones() and emp.getDepartment() are fully loaded — no lazy init exception
```

---

## :material-key: Key Takeaways — Chapter 11

1. **Native queries** are for vendor-specific SQL only — JPQL is preferred for portability
2. **`@NamedNativeQuery`** stores raw SQL on the entity class; `@SqlResultSetMapping` maps non-standard column names
3. **`@ConstructorResult`** is the cleanest way to map native SQL results into lightweight projection classes
4. **Only positional parameters** (`?`) are JPA-portable for native queries — named parameters are provider-specific
5. **`StoredProcedureQuery`** maps all parameter modes; `REF_CURSOR` returns result sets
6. **`fetchgraph`** overrides ALL unlisted attributes to LAZY — use carefully when some static EAGER mappings are intentional
7. **`loadgraph`** is safer — listed are EAGER, unlisted keep their static mapping

---

[:octicons-arrow-left-24: Back to Day 18 Index](index.md)
