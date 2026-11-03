---
id: phase-4-week-9-summary
tags: [spring, jpa, hibernate, advanced-mappings, summary, reference, join-fetch, cascade, phase-4]
---

# Summary & Architecture Reference — Week 9: Advanced JPA Relational Topologies

---

## 1. JPA Mapping Annotation Matrix

| Annotation | Target Placement | Multiplicity | Default Fetch | Owning Side Declaration | Critical Attributes | Common Senior Pitfall |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`@OneToOne`** | Field / Getter | 1:1 | `FetchType.EAGER` | Placed on entity declaring `@JoinColumn` | `cascade`, `fetch`, `mappedBy`, `orphanRemoval` | Forgetting `mappedBy` on inverse side, generating duplicate foreign key columns. |
| **`@OneToMany`** | Collection Field | 1:N | `FetchType.LAZY` | Sits on inverse side (if bidirectional) | `mappedBy`, `cascade`, `fetch`, `orphanRemoval` | Changing to `FetchType.EAGER` to fix lazy exceptions, causing memory bloat and N+1. |
| **`@ManyToOne`** | Entity Field | N:1 | `FetchType.EAGER` | Always sits on owning side | `cascade`, `fetch`, `optional` | Applying `CascadeType.REMOVE`, accidentally deleting parent when child is removed. |
| **`@ManyToMany`** | Collection Field | M:N | `FetchType.LAZY` | Sits on entity declaring `@JoinTable` | `mappedBy`, `cascade`, `fetch` | Using `CascadeType.ALL` or `REMOVE`, deleting peer entities across the network. |
| **`@JoinColumn`** | Entity / Collection | Any Owning | N/A | Designates physical FK column | `name`, `referencedColumnName`, `nullable` | Omission on 1:N unidirectional mapping causes JPA to generate unwanted join table. |
| **`@JoinTable`** | Collection Field | M:N Owning | N/A | Maps dedicated intermediate table | `name`, `joinColumns`, `inverseJoinColumns` | Inverting `joinColumns` vs `inverseJoinColumns`, reversing relational foreign key links. |

---

## 2. Cascade Propagation & Relational Decision Matrix

```mermaid
flowchart TD
    START["Define Association between Entity A and Entity B"] --> CARD{"What is the Cardinality?"}

    CARD -->|"One-to-One (1:1)"| OTO{"Are entities tightly coupled composites?"}
    OTO -->|"Yes (e.g. Instructor -> Detail)"| OTO_ALL["CascadeType.ALL on Owning Side<br/>Selective Cascades on Inverse Side"]
    OTO -->|"No (Independent Lifecycles)"| OTO_SEL["Selective: PERSIST, MERGE, REFRESH, DETACH<br/>(Exclude REMOVE)"]

    CARD -->|"One-to-Many (1:N)"| OTM{"Does Child have meaning without Parent?"}
    OTM -->|"No (e.g. Course -> Reviews)"| OTM_ALL["1:N Unidirectional<br/>CascadeType.ALL + @JoinColumn"]
    OTM -->|"Yes (e.g. Instructor <-> Course)"| OTM_SEL["1:N Bidirectional with mappedBy<br/>Selective Cascades (Exclude REMOVE)<br/>Programmatic Dissociation on Delete"]

    CARD -->|"Many-to-Many (M:N)"| MTM["@ManyToMany via @JoinTable<br/>NEVER USE CascadeType.REMOVE<br/>Peers must have independent lifecycles"]
```

---

## 3. Fetch Strategy Comparison: Default Rules vs JOIN FETCH

| Cardinality | JPA Default Fetch | Lazy Proxy Type | N+1 Risk? | Optimized Query Strategy |
| :--- | :--- | :--- | :--- | :--- |
| **`@OneToOne`** | `FetchType.EAGER` | ByteBuddy proxy (if optional lazy) | Low | `JOIN FETCH i.instructorDetail` |
| **`@ManyToOne`** | `FetchType.EAGER` | ByteBuddy proxy (if optional lazy) | Low | `JOIN FETCH c.instructor` |
| **`@OneToMany`** | `FetchType.LAZY` | `PersistentBag` / `PersistentSet` | **Critical** | `JOIN FETCH i.courses` |
| **`@ManyToMany`** | `FetchType.LAZY` | `PersistentBag` / `PersistentSet` | **Critical** | `JOIN FETCH c.students` |

---

## 4. The N+1 Query Problem: Performance Benchmark

```mermaid
flowchart LR
    subgraph NaiveQuery["Naive Lazy Loading (N+1 Problem)"]
        N1["1 Query: SELECT * FROM instructor (100 rows)"]
        N2["100 Queries: SELECT * FROM course WHERE instructor_id=?"]
        N1 --> N2
        N2 --> TOTAL1["Total: 101 Database Roundtrips<br/>Latency: ~250ms - 1500ms"]
    end

    subgraph OptimizedFetch["JPQL JOIN FETCH (Single Statement)"]
        OPT["1 Query: SELECT i FROM Instructor i JOIN FETCH i.courses"]
        OPT --> SQL["Single SQL: SELECT i.*, c.* FROM instructor i INNER JOIN course c ON ..."]
        SQL --> TOTAL2["Total: 1 Database Roundtrip<br/>Latency: ~10ms - 25ms"]
    end
```

### JPQL JOIN FETCH Syntax Cheatsheet:
```java
// Single Collection Fetch
"SELECT i FROM Instructor i JOIN FETCH i.courses WHERE i.id = :id"

// Multiple Joins (1:1 + 1:N)
"SELECT i FROM Instructor i JOIN FETCH i.courses JOIN FETCH i.instructorDetail WHERE i.id = :id"

// Left Outer Join Fetch (includes Instructors with ZERO courses)
"SELECT i FROM Instructor i LEFT JOIN FETCH i.courses WHERE i.id = :id"
```

---

## 5. Programmatic Entity Dissociation Protocol Checklist

When deleting entities that participate in associations without cascading removal, follow the **Dissociation Protocol** to avoid foreign key constraint failures:

```mermaid
sequenceDiagram
    autonumber
    participant Service as Business Service
    participant DAO as Data Access Object
    participant EM as EntityManager
    participant DB as Relational Database

    Service->>DAO: deleteInstructor(instructorId)
    DAO->>EM: find(Instructor.class, instructorId)
    EM->>DB: SELECT * FROM instructor WHERE id=?
    DB-->>EM: Instructor (Managed)
    Note over DAO: Step 1: Programmatic Dissociation
    loop For each Course in instructor.getCourses()
        DAO->>DAO: course.setInstructor(null)
    end
    Note over DAO: Step 2: Entity Removal
    DAO->>EM: remove(instructor)
    Note over EM,DB: Flush: Flushes FK updates before DELETE
    EM->>DB: UPDATE course SET instructor_id=NULL WHERE id=?
    EM->>DB: DELETE FROM instructor WHERE id=?
    DB-->>EM: Success
```

### The 4 Dissociation Rules:
1. **1:1 Child Deletion (`InstructorDetail`)**:
   `detail.getInstructor().setInstructorDetail(null);`
   `entityManager.remove(detail);`
2. **1:N Parent Deletion (`Instructor` preserving `Courses`)**:
   `for (Course c : instructor.getCourses()) { c.setInstructor(null); }`
   `entityManager.remove(instructor);`
3. **M:N Inverse Deletion (`Student` preserving `Courses`)**:
   `for (Course c : student.getCourses()) { c.getStudents().remove(student); }`
   `entityManager.remove(student);`
4. **M:N Owner Deletion (`Course` preserving `Students`)**:
   `entityManager.remove(course);` (Hibernate automatically deletes join table rows).
