# :material-pencil: Topic Note: Arrays, Lists & Generics

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Learning Objectives

- [ ] Master array declaration and manipulation
- [ ] Understand ArrayList vs LinkedList trade-offs
- [ ] Apply generics for type-safe code
- [ ] Use wildcards effectively

---

## :material-head-cog: Key Concepts from the Course

### Arrays

#### Array Declaration & Initialization

<!-- Your notes on array syntax -->

```java
// Declaration styles
int[] numbers = new int[5];
int[] numbers = {1, 2, 3, 4, 5};
int[] numbers = new int[]{1, 2, 3, 4, 5};
```

#### Multi-dimensional Arrays

<!-- Notes on 2D arrays, jagged arrays -->



#### Array Utilities

<!-- Arrays.sort(), Arrays.binarySearch(), etc. -->



---

### ArrayList

#### Declaration & Basic Operations

<!-- Add, remove, get, set, size -->



#### ArrayList vs Array Comparison

| Feature | Array | ArrayList |
|---------|-------|-----------|
| Size | Fixed | Dynamic |
| Type | Primitives & Objects | Objects only |
| Performance | Faster | Slightly slower |
| Generics | No | Yes |
| Memory | Contiguous | Contiguous + overhead |

---

### LinkedList

#### Structure & Operations

<!-- How LinkedList works internally -->



#### When to Use LinkedList

<!-- Frequent insertions/deletions at beginning/middle -->



---

### Generics Fundamentals

#### Generic Class Definition

```java
public class Box<T> {
    private T content;
    
    public void set(T content) {
        this.content = content;
    }
    
    public T get() {
        return content;
    }
}
```

#### Generic Methods

<!-- Defining type parameters on methods -->



#### Bounded Type Parameters

```java
// Upper bound - T must extend Number
public <T extends Number> double sum(List<T> list) { }

// Multiple bounds
public <T extends Comparable<T> & Serializable> void process(T item) { }
```

---

### Wildcards

#### Unbounded Wildcard (`?`)

<!-- When to use unbounded wildcards -->



#### Upper Bounded Wildcard (`? extends T`)

<!-- Reading from collections - Producer -->



#### Lower Bounded Wildcard (`? super T`)

<!-- Writing to collections - Consumer -->



#### PECS Principle

<!-- Producer Extends, Consumer Super -->



---

## :material-lightbulb-on: Observed Ideas & Insights

### Type Erasure Revelation

<!-- What happens to generics at runtime -->



### Covariance vs Invariance

<!-- Why arrays are different from generics -->



---

## :material-link-variant: Code Snippets & Examples

```java
// Add your practice code snippets here

```

---

## :material-help-circle: Questions to Explore

- [ ] Why can't we create arrays of generic types?
- [ ] What's the difference between `List<Object>` and `List<?>`?
- [ ] 

---

## :material-pin: Quick Reference

| Concept | Key Point |
|---------|-----------|
| Arrays | Fixed size, covariant, primitives allowed |
| ArrayList | Dynamic, backed by array, fast random access |
| LinkedList | Dynamic, fast insert/delete at ends |
| Generics | Type safety at compile time, erased at runtime |
| PECS | Producer Extends, Consumer Super |

---

*Last Updated: <!-- Add date -->*
