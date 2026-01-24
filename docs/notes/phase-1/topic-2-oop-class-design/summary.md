# :material-school: Summary: OOP & Class Design Internals

> **Combined Knowledge from:** Tim Buchalka's Course + Effective Java  
> **Mastery Level:** :material-star::material-star::material-star::material-star::material-star:

---

## :material-star-shooting: Topic Overview

A deep understanding of object-oriented programming principles, their implementation in Java, and best practices for designing robust, maintainable classes.

---

## :material-key: The Four Pillars of OOP

### 1. Encapsulation

**Definition:** Bundling data and methods that operate on that data within a single unit (class), restricting direct access to internal state.

#### Key Principles

<!-- Your synthesized understanding -->



#### Best Practices

- Use the most restrictive access level possible
- Make fields private by default
- Provide controlled access through methods

---

### 2. Inheritance

**Definition:** Mechanism where a new class inherits properties and behaviors from an existing class.

#### Key Principles

<!-- Your synthesized understanding -->



#### When to Use Inheritance

- Clear IS-A relationship exists
- Subclass is truly a subtype of superclass
- Behavior needs to be shared across hierarchy

#### When to Avoid

- Just for code reuse (use composition instead)
- Violates Liskov Substitution Principle

---

### 3. Polymorphism

**Definition:** Ability of objects to take many forms, allowing same interface to be used for different underlying types.

#### Compile-Time (Static) Polymorphism

```java
// Method Overloading
public void print(int x) { }
public void print(String s) { }
```

#### Runtime (Dynamic) Polymorphism

```java
// Method Overriding
Animal animal = new Dog();
animal.speak(); // Calls Dog's speak() at runtime
```

#### VTable Mechanism (Internal)

<!-- How JVM implements dynamic dispatch -->



---

### 4. Abstraction

**Definition:** Hiding complex implementation details and showing only essential features.

#### Abstract Class vs Interface Decision Matrix

| Criteria | Choose Abstract Class | Choose Interface |
|----------|----------------------|------------------|
| Shared code | :material-check: | :material-alert: (default methods) |
| State needed | :material-check: | :material-close: |
| Multiple inheritance | :material-close: | :material-check: |
| API evolution | :material-alert: | :material-check: (default methods) |
| Constructor needed | :material-check: | :material-close: |

---

## :material-head-cog: Object Creation Deep Dive

### Object Creation Process

1. Memory allocation on heap
2. Instance variable initialization to defaults
3. Instance initializer blocks execute
4. Constructor body executes
5. Reference returned

### Best Practices (from Effective Java)

1. **Static Factory Methods** over constructors when:
   - Need descriptive names
   - Want to control instance creation
   - Need to return subtypes

2. **Builder Pattern** when:
   - Many constructor parameters
   - Many optional parameters
   - Immutable objects needed

---

## :material-lightning-bolt: Class Design Principles

### SOLID Principles Applied

| Principle | Description | Application |
|-----------|-------------|-------------|
| **S**ingle Responsibility | One reason to change | Small, focused classes |
| **O**pen/Closed | Open for extension, closed for modification | Use abstraction |
| **L**iskov Substitution | Subtypes replaceable | Honor contracts |
| **I**nterface Segregation | Small, specific interfaces | Don't force implementations |
| **D**ependency Inversion | Depend on abstractions | Use interfaces |

### Composition Over Inheritance

<!-- Your understanding of when and why -->



---

## :material-alert: Common Pitfalls

1. **Breaking encapsulation**
   - Exposing internal state unnecessarily
   
2. **Inheritance abuse**
   - Using inheritance for code reuse only
   
3. **equals/hashCode contract violations**
   - Override one without the other
   
4. **Mutable objects in wrong places**
   - Using mutable objects as keys

---

## :material-lightbulb-on: Best Practices Checklist

- [ ] Make classes immutable when possible
- [ ] Favor composition over inheritance
- [ ] Program to interfaces, not implementations
- [ ] Override equals and hashCode together
- [ ] Consider static factory methods
- [ ] Minimize accessibility of everything

---

## :material-link-variant: Related Topics

- [Syntax, Variables & Control Flow](My-Java-Rodemap/docs/notes/phase-1/topic-1-syntax-variables-control-flow/summary.md)
- [Arrays, Lists & Generics](../topic-3-arrays-lists-generics/summary.md)
- [Collections Framework](../topic-5-collections-framework/summary.md)

---

## :material-bookshelf: References

- **Course:** Tim Buchalka - Java Programming Masterclass
- **Book:** Effective Java - Joshua Bloch (Chapters 2-4)
- **Book:** Head First Design Patterns (OOP principles applied)

---

*Completed: <!-- Add date --> | Confidence: X/10*
