# Topic 2: OOP & Class Design Internals

> Mastering object-oriented programming principles and understanding how Java implements them internally.

---

## :material-folder-open: Topic Structure

| Document | Purpose | Status |
|----------|---------|--------|
| [:material-pencil: Topic Note Part 1](topic-note.md) | Classes, Objects & Encapsulation | :material-check-circle: Complete |
| [:material-pencil: Topic Note Part 2](topic-note-part2.md) | Inheritance & Method Overriding | :material-check-circle: Complete |
| [:material-pencil: Topic Note Part 3](topic-note-part3.md) | Strings & StringBuilder | :material-check-circle: Complete |
| [:material-pencil: Topic Note Part 4](topic-note-part4.md) | Composition | :material-check-circle: Complete |
| [:material-pencil: Topic Note Part 5](topic-note-part5.md) | Encapsulation (Advanced) | :material-check-circle: Complete |
| [:material-pencil: Topic Note Part 6](topic-note-part6.md) | Polymorphism | :material-check-circle: Complete |
| [:material-sword-cross: OOP Challenges](topic-challenges.md) | Inheritance & OOP Master Challenges | :material-check-circle: Complete |
| [:material-book-open-page-variant: Book Reading](book-reading.md) | Effective Java insights | :material-check-circle: Complete |
| [:material-school: Summary](summary.md) | Combined final understanding | :material-check-circle: Complete |

---

## :material-notebook-outline: Topic Notes Overview

### Part 1: Classes, Objects & Encapsulation
Covers foundational OOP concepts including class anatomy, access modifiers, getters/setters, constructors (default, parameterized, chaining), references vs objects, static vs instance members, POJOs, and Java Records.

### Part 2: Inheritance & Method Overriding
Covers the `extends` keyword, superclass/subclass relationships, the `super` keyword, constructor chaining across classes, method overriding with `@Override`, `java.lang.Object` as the root class, and the differences between overloading vs overriding.

### Part 3: Strings & StringBuilder
Covers Text Blocks (JDK 15+), escape sequences, printf formatting with specifiers, String inspection methods (length, charAt, indexOf), comparison methods (equals, contains), manipulation methods (substring, replace, join), and StringBuilder for mutable string operations.

### Part 4: Composition
Covers IS-A vs HAS-A relationships, building composite objects from simpler parts, delegation patterns, real-world composition examples (PersonalComputer, SmartKitchen), and when to prefer composition over inheritance.

### Part 5: Encapsulation (Advanced)
Covers deep dive into encapsulation principles, the three problems of poor encapsulation, data validation in constructors and setters, proper access modifier usage, and the Printer challenge.

### Part 6: Polymorphism
Covers compile-time vs runtime types, factory methods, type casting, `instanceof` operator, pattern matching (JDK 16+), Local Variable Type Inference (`var`), and the Car challenge demonstrating polymorphic behavior.

### OOP Challenges
Two comprehensive challenges: (1) **Inheritance Challenge** - Worker/Employee hierarchy with SalariedEmployee and HourlyEmployee; (2) **OOP Master Challenge** - Burger Restaurant application using all OOP principles (composition, inheritance, encapsulation, polymorphism).

---

## :material-target: What You'll Master

- **Classes & Objects** - Instance creation, memory allocation
- **Encapsulation** - Access modifiers, information hiding, data validation
- **Inheritance** - Class hierarchy, constructor chaining
- **Polymorphism** - Runtime dispatch, factory methods, type casting
- **Abstraction** - Abstract classes vs interfaces
- **Composition** - HAS-A relationships, delegation over inheritance

---

## :material-bookshelf: Resources

### Primary Course
- [Tim Buchalka's Java Masterclass](https://www.udemy.com/course/java-the-complete-java-developer-course/) (Udemy)

### Book Reference  
- **Effective Java** by Joshua Bloch - Chapters 2-4 (Creating & Destroying Objects, Common Methods, Classes & Interfaces)

### Key Internals to Understand
- Method dispatch mechanism (vtable)
- How polymorphism works at runtime
- Object creation process (memory allocation, initialization blocks, constructor execution)
- Interface vs Abstract class performance considerations

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Complete Tim's course sections on OOP (Section 07-08)
- [x] Read Effective Java Chapters 2-4
- [x] Write topic notes (Part 1-6)
- [x] Complete challenges documentation
- [x] Complete book reading notes
- [x] Synthesize final summary
- [ ] Implement OOP-focused mini-project

---

*Start Date: 2026-01-25 | Completed: 2026-01-26*
