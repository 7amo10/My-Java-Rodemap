---
id: phase-4-week-3-index
tags: [spring, hibernate, jpa, crud, dao, entitymanager, jpql, phase-4]
---

# Week 3 — Hibernate / JPA: Data Access Layer & CRUD

## Goal

Understand how Spring Boot integrates the JPA specification and Hibernate ORM to persist Java objects to relational databases. By the end of this week you should be able to wire a complete DAO layer using `EntityManager`, write JPQL queries for read operations, and understand when to reach for Spring Data JPA's `JpaRepository` as a higher-level abstraction.

---

## :material-calendar: Week 3 Schedule

| Topic | Lectures | Key Annotations / Concepts |
|-------|----------|----------------------------|
| Hibernate / JPA overview | Slides 1–10 | ORM, JPA spec, Hibernate as vendor, JDBC beneath |
| Entity mapping & identity | Slides 11–22 | `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column` |
| DAO pattern with EntityManager | Slides 23–40 | `@Repository`, `@Transactional`, `EntityManager.persist()` |
| Reading: findById & findAll | Slides 41–55 | `EntityManager.find()`, `TypedQuery`, JPQL `FROM` |
| JPQL queries & named params | Slides 56–68 | `:named`, `setParameter()`, `LIKE`, `ORDER BY` |
| Update & delete operations | Slides 69–85 | `EntityManager.merge()`, `EntityManager.remove()` |
| Schema generation options | Slides 86–95 | `spring.jpa.hibernate.ddl-auto` |
| EntityManager vs JpaRepository | Slides 96–100 | When to use each; boilerplate elimination |

---

## :material-folder-open: Week 3 Content

<div class="grid cards" markdown>

-   :material-note-text:{ .lg .middle } **Topic Notes — Part 1**

    ---

    JPA / Hibernate overview, ORM concepts, the EntityManager, entity class mapping (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`), DAO pattern, `@Repository`, `@Transactional`, and the full CRUD save/read cycle.

    [:octicons-arrow-right-24: Topic Notes Part 1](topic-notes-part1.md)

-   :material-note-text:{ .lg .middle } **Topic Notes — Part 2**

    ---

    JPQL — query language syntax, `TypedQuery`, named parameters, ordering; update and delete via `EntityManager.merge()` / `EntityManager.remove()`; bulk JPQL DML; DDL auto-generation configuration; EntityManager vs JpaRepository comparison.

    [:octicons-arrow-right-24: Topic Notes Part 2](topic-notes-part2.md)

-   :material-book-open-variant:{ .lg .middle } **Book Reading — Spring in Action Ch 3**

    ---

    Craig Walls on working with data: JdbcTemplate without Hibernate, JDBC repositories, schema / data SQL bootstrapping, Spring Data JDBC, transitioning to Spring Data JPA, `CrudRepository`, and `JpaRepository`.

    [:octicons-arrow-right-24: Book Reading](book-reading.md)

-   :material-flask:{ .lg .middle } **Lab Guide — Student Information System DAO Tier**

    ---

    Full CRUD against a MySQL `student` table using `EntityManager`, `@Repository`, `@Transactional`, JPQL `findAll` / `findByLastName`, update with `merge()`, and delete.

    [:octicons-arrow-right-24: Lab Guide](lab-guide.md)

-   :material-clipboard-check:{ .lg .middle } **Week 3 Summary**

    ---

    Annotation quick-reference, ORM concept map, EntityManager method table, JPQL cheat sheet, `ddl-auto` options table, EntityManager vs JpaRepository comparison, and prerequisite checklist for Week 4.

    [:octicons-arrow-right-24: Summary](summary.md)

</div>

---

## :material-checkbox-marked-outline: Week 3 Progress

- [ ] Explain the difference between JPA (spec) and Hibernate (implementation)
- [ ] Annotate a class with `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`
- [ ] Explain what Spring Boot auto-configures when `spring-boot-starter-data-jpa` is on the classpath
- [ ] Inject `EntityManager` via constructor injection into a `@Repository` class
- [ ] Implement `save()` with `entityManager.persist()` and `@Transactional`
- [ ] Implement `findById()` with `entityManager.find()` — no `@Transactional` needed
- [ ] Write a JPQL `FROM Student` query using `TypedQuery` to retrieve a list
- [ ] Write a parameterized JPQL query using `:theData` named parameter and `setParameter()`
- [ ] Implement `update()` with `entityManager.merge()` and `@Transactional`
- [ ] Implement `delete()` with `entityManager.find()` + `entityManager.remove()` and `@Transactional`
- [ ] Explain the difference between `ddl-auto=create`, `update`, `validate`, and `none`
- [ ] Articulate when to use `EntityManager` directly vs `JpaRepository`
- [ ] Complete Week 3 Lab: Student Information System DAO Tier
