# :material-pencil: Topic Note: OOP & Class Design Internals

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Learning Objectives

- [ ] Understand class and object fundamentals
- [ ] Master the four pillars of OOP
- [ ] Learn how polymorphism works at runtime
- [ ] Apply SOLID principles in class design

---

## :material-head-cog: Key Concepts from the Course

### Classes & Objects

#### Class Anatomy

<!-- Your notes on class structure: fields, methods, constructors -->



#### Object Creation Process

<!-- How Java creates objects in memory -->



---

### Encapsulation

#### Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | :material-check: | :material-check: | :material-check: | :material-check: |
| `protected` | :material-check: | :material-check: | :material-check: | :material-close: |
| *default* | :material-check: | :material-check: | :material-close: | :material-close: |
| `private` | :material-check: | :material-close: | :material-close: | :material-close: |

#### Getters & Setters Design

<!-- When and how to use them effectively -->



---

### Inheritance

#### The `extends` Keyword

<!-- Notes on class extension -->



#### Constructor Chaining

<!-- super() calls and initialization order -->



#### The `super` Keyword

<!-- Accessing parent class members -->



---

### Polymorphism

#### Method Overriding

<!-- Rules and best practices -->



#### Dynamic Binding (Runtime Polymorphism)

<!-- How JVM decides which method to call -->



#### The VTable Mechanism

<!-- Internal implementation of method dispatch -->



---

### Abstraction

#### Abstract Classes

<!-- When to use abstract classes -->



#### Interfaces

<!-- Interface design and default methods -->



#### Abstract vs Interface Comparison

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Multiple Inheritance | No | Yes |
| Constructor | Yes | No |
| Instance Variables | Yes | Only constants |
| Access Modifiers | Any | Public (default since Java 9) |
| State | Can have state | Stateless |

---

### Composition Over Inheritance

<!-- Why and when to prefer composition -->



---

## :material-lightbulb-on: Observed Ideas & Insights

### Design Patterns Glimpse

<!-- Any patterns you noticed (Factory, Builder, etc.) -->



### Code Smells to Avoid

<!-- Anti-patterns in OOP -->



---

## :material-link-variant: Code Snippets & Examples

```java
// Add your practice code snippets here

```

---

## :material-help-circle: Questions to Explore

- [ ] How does the JVM implement vtables?
- [ ] When is static binding used vs dynamic binding?
- [ ] 

---

## :material-pin: Quick Reference

| Concept | Key Point |
|---------|-----------|
| Encapsulation | Hide implementation, expose interface |
| Inheritance | IS-A relationship |
| Composition | HAS-A relationship |
| Polymorphism | One interface, multiple implementations |

---

*Last Updated: <!-- Add date -->*
