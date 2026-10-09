---
id: phase-4-week-3-lab-guide
tags: [spring, hibernate, jpa, crud, dao, entitymanager, lab, phase-4, week-3]
---

# :material-flask: Week 03 Lab Guide — Student Information System DAO Tier

> **Module:** Week 3 — Hibernate / JPA CRUD and Data Access Object (DAO) Layer  
> **Lab Repo:** [:material-github: 7amo10/Spring-Labs — Week-03-Hibernate-JPA-CRUD](https://github.com/7amo10/Spring-Labs/tree/main/Week-03-Hibernate-JPA-CRUD)

---

## :material-target: Lab Objective

Construct an enterprise-grade Data Access Object (DAO) persistence layer utilizing **Jakarta Persistence (JPA)** with **Hibernate** under Spring Boot 3. Model a domain entity with primary key identity generation, configure Spring Boot database connection properties for MySQL, encapsulate persistence operations within a typed DAO interface, execute CRUD mutations with `EntityManager` and declarative `@Transactional` boundaries, author dynamic queries using the **Java Persistence Query Language (JPQL)** with named parameters, and observe schema generation behaviors via Hibernate DDL auto-configuration.

---

## :material-list-box: Key Concepts Covered

- **Entity Modeling (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`):** Mapping Java POJOs to relational tables with identity-based surrogate keys.
- **DAO Pattern with `EntityManager`:** Encapsulating persistence operations behind a typed interface to decouple business logic from persistence technology.
- **CRUD Operations via EntityManager:**
  - Create: `entityManager.persist()`
  - Read: `entityManager.find()`
  - Update: `entityManager.merge()` and dirty checking
  - Delete: `entityManager.remove()`
- **JPQL Querying (`TypedQuery`):** Querying against entity classes and fields with sorting, filtering, and named parameters (`:theData`).
- **Bulk DML Operations:** Executing bulk updates and deletes with `TypedQuery.executeUpdate()`.
- **Transaction Boundaries (`@Transactional`):** Managing database transactions declaratively at the DAO tier.
- **DDL Auto Generation:** Configuring and analyzing `spring.jpa.hibernate.ddl-auto`.

---

## :material-code-braces: Component Architecture

Package: `com.luv2code.cruddemo`

| Component | Scope / Annotations | Responsibility |
| :--- | :--- | :--- |
| `Student` | `@Entity`, `@Table(name="student")` | Domain entity mapped to relational table `student` with auto-increment ID |
| `StudentDAO` | Interface | Contract declaring 7 data access methods for full CRUD and custom queries |
| `StudentDAOImpl` | `@Repository` | JPA implementation injecting `EntityManager` via constructor injection |
| `CruddemoApplication` | `@SpringBootApplication`, `CommandLineRunner` | Application entry point executing test suites across all DAO methods |

---

## :material-database: Database Setup and Schema

Execute the initialization script against a local MySQL instance:

```sql
CREATE DATABASE IF NOT EXISTS `student_tracker`;
USE `student_tracker`;

CREATE TABLE IF NOT EXISTS `student` (
  `id` int NOT NULL AUTO_INCREMENT,
  `first_name` varchar(45) DEFAULT NULL,
  `last_name` varchar(45) DEFAULT NULL,
  `email` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;
```

Configure `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/student_tracker
spring.datasource.username=springstudent
spring.datasource.password=springstudent

# Hibernate logging
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.orm.jdbc.bind=trace

# DDL configuration (none for existing schema, update for auto-creation)
spring.jpa.hibernate.ddl-auto=none
```

---

## :material-hammer-wrench: Step-by-Step Implementation

### Step 1: Model the `Student` Entity

```java
package com.luv2code.cruddemo.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "email")
    private String email;

    public Student() {}

    public Student(String firstName, String lastName, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }

    // Getters, setters, and toString()
}
```

### Step 2: Define the `StudentDAO` Interface

```java
package com.luv2code.cruddemo.dao;

import com.luv2code.cruddemo.entity.Student;
import java.util.List;

public interface StudentDAO {
    void save(Student theStudent);
    Student findById(Integer id);
    List<Student> findAll();
    List<Student> findByLastName(String theLastName);
    void update(Student theStudent);
    void delete(Integer id);
    int deleteAll();
}
```

### Step 3: Implement `StudentDAOImpl` with `EntityManager`

```java
package com.luv2code.cruddemo.dao;

import com.luv2code.cruddemo.entity.Student;
import jakarta.persistence.EntityManager;
import jakarta.persistence.TypedQuery;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Repository
public class StudentDAOImpl implements StudentDAO {

    private final EntityManager entityManager;

    @Autowired
    public StudentDAOImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    @Transactional
    public void save(Student theStudent) {
        entityManager.persist(theStudent);
    }

    @Override
    public Student findById(Integer id) {
        return entityManager.find(Student.class, id);
    }

    @Override
    public List<Student> findAll() {
        TypedQuery<Student> theQuery = entityManager.createQuery("FROM Student", Student.class);
        return theQuery.getResultList();
    }

    @Override
    public List<Student> findByLastName(String theLastName) {
        TypedQuery<Student> theQuery = entityManager.createQuery(
                "FROM Student WHERE lastName = :theData", Student.class);
        theQuery.setParameter("theData", theLastName);
        return theQuery.getResultList();
    }

    @Override
    @Transactional
    public void update(Student theStudent) {
        entityManager.merge(theStudent);
    }

    @Override
    @Transactional
    public void delete(Integer id) {
        Student theStudent = entityManager.find(Student.class, id);
        if (theStudent != null) {
            entityManager.remove(theStudent);
        }
    }

    @Override
    @Transactional
    public int deleteAll() {
        return entityManager.createQuery("DELETE FROM Student").executeUpdate();
    }
}
```

### Step 4: Orchestrate Verification in `CommandLineRunner`

```java
package com.luv2code.cruddemo;

import com.luv2code.cruddemo.dao.StudentDAO;
import com.luv2code.cruddemo.entity.Student;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import java.util.List;

@SpringBootApplication
public class CruddemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(CruddemoApplication.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(StudentDAO studentDAO) {
        return runner -> {
            createStudent(studentDAO);
            // queryForStudents(studentDAO);
            // queryForStudentsByLastName(studentDAO);
            // updateStudent(studentDAO);
            // deleteStudent(studentDAO);
            // deleteAllStudents(studentDAO);
        };
    }

    private void createStudent(StudentDAO studentDAO) {
        System.out.println("Creating new student object ...");
        Student tempStudent = new Student("Paul", "Doe", "paul@luv2code.com");

        System.out.println("Saving the student ...");
        studentDAO.save(tempStudent);

        System.out.println("Saved student. Generated id: " + tempStudent.getId());
    }
}
```

---

## :material-check-decagram: Verification Checklist

| Test Phase | Operation | Verification Method | Expected Console / SQL Output |
| :--- | :--- | :--- | :--- |
| **Phase 1** | Save Single Entity | Call `studentDAO.save(student)` | `Hibernate: insert into student (email,first_name,last_name) values (?,?,?)` with generated id assigned |
| **Phase 2** | Save Multiple Entities | Insert batch in a loop | Each instance receives sequential primary keys |
| **Phase 3** | Read by Primary Key | `studentDAO.findById(id)` | `Hibernate: select ... from student where id=?` returns hydrated `Student` object |
| **Phase 4** | Query All Entities | `studentDAO.findAll()` | `Hibernate: select ... from student` returns full `List<Student>` |
| **Phase 5** | Filter with Named Parameter | `studentDAO.findByLastName("Doe")` | `Hibernate: select ... where last_name=?` matches records |
| **Phase 6** | Update via `merge()` | Mutate field and call `studentDAO.update(student)` | `Hibernate: update student set email=?, first_name=?, last_name=? where id=?` |
| **Phase 7** | Delete Single Entity | `studentDAO.delete(id)` | Select issued to verify managed state, followed by `delete from student where id=?` |
| **Phase 8** | Bulk Delete | `studentDAO.deleteAll()` | Single DML statement: `delete from student` with affected row count returned |

---

## :material-github: Lab Solution Repository

The full runnable source code with Maven build scripts and SQL seed files is accessible in the GitHub lab repository:

[:material-github: 7amo10/Spring-Labs — Week-03-Hibernate-JPA-CRUD](https://github.com/7amo10/Spring-Labs/tree/main/Week-03-Hibernate-JPA-CRUD)
