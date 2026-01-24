# :material-school: Summary: Arrays, Lists & Generics

> **Combined Knowledge from:** Tim Buchalka's Course + Effective Java  
> **Mastery Level:** :material-star::material-star::material-star::material-star::material-star:

---

## :material-star-shooting: Topic Overview

Mastering Java's data structures from basic arrays to type-safe generic collections, understanding the theoretical foundations and practical applications.

---

## :material-key: Core Concepts

### 1. Arrays

#### Fundamentals

- Fixed size at creation
- Zero-indexed
- Stored contiguously in memory
- Support both primitives and objects
- Covariant (String[] is subtype of Object[])

#### Multi-dimensional Arrays

```java
// Rectangular
int[][] matrix = new int[3][4];

// Jagged
int[][] jagged = new int[3][];
jagged[0] = new int[2];
jagged[1] = new int[4];
```

#### Key Operations

| Operation | Method | Complexity |
|-----------|--------|------------|
| Sort | `Arrays.sort()` | O(n log n) |
| Search | `Arrays.binarySearch()` | O(log n) |
| Copy | `Arrays.copyOf()` | O(n) |
| Fill | `Arrays.fill()` | O(n) |
| Compare | `Arrays.equals()` | O(n) |

---

### 2. ArrayList vs LinkedList

#### Performance Comparison

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| get(index) | O(1) | O(n) |
| add(end) | O(1) amortized | O(1) |
| add(middle) | O(n) | O(n)* |
| remove(index) | O(n) | O(n) |
| Iterator.remove() | O(n) | O(1) |

*Finding position is O(n), but insertion itself is O(1)

#### When to Use Which

<!-- Your synthesized guidance -->



---

### 3. Generics Mastery

#### Why Generics?

1. **Type Safety** - Compile-time error detection
2. **Elimination of Casts** - Cleaner code
3. **Generic Algorithms** - Write once, use for many types

#### Generic Class Template

```java
public class Container<T> {
    private T item;
    
    public Container(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
}
```

#### Type Parameter Conventions

| Letter | Convention |
|--------|------------|
| E | Element (collections) |
| K | Key |
| V | Value |
| T | Type |
| S, U | 2nd, 3rd type |
| N | Number |

---

### 4. Wildcards & PECS

#### The PECS Principle

> **P**roducer **E**xtends, **C**onsumer **S**uper

```java
// Producer - reading items of type E
void copyFrom(Collection<? extends E> source) {
    for (E item : source) {
        this.add(item);  // Reading from source
    }
}

// Consumer - adding items of type E
void copyTo(Collection<? super E> destination) {
    for (E item : this) {
        destination.add(item);  // Writing to destination
    }
}
```

#### Decision Matrix

| If you want to... | Use |
|-------------------|-----|
| Read items of type T | `<? extends T>` |
| Write items of type T | `<? super T>` |
| Read AND write | `<T>` (no wildcard) |
| Use any type | `<?>` |

---

## :material-head-cog: Key Internals

### Type Erasure

- Generic type information removed at runtime
- Replaced with bounds or Object
- Bridge methods generated for polymorphism
- Implications:
  - Can't use `instanceof` with generics
  - Can't create generic arrays
  - Can't use primitives as type parameters

### Arrays vs Generics: Covariance

```java
// Arrays are covariant - this compiles but may fail at runtime
Object[] array = new String[1];
array[0] = 1;  // ArrayStoreException at runtime!

// Generics are invariant - this doesn't compile
List<Object> list = new ArrayList<String>();  // Compile error!
```

---

## :material-alert: Common Pitfalls

1. **Using raw types**
   - Always use parameterized types

2. **Ignoring unchecked warnings**
   - Investigate and fix or suppress with justification

3. **Creating arrays of generic types**
   - Use `List<E>` instead of `E[]`

4. **Wrong wildcard direction**
   - Remember PECS!

---

## :material-lightbulb-on: Best Practices Checklist

- [ ] Never use raw types in new code
- [ ] Prefer lists to arrays
- [ ] Apply PECS for wildcard decisions
- [ ] Suppress warnings only with good reason and narrow scope
- [ ] Document the reason for @SuppressWarnings
- [ ] Use bounded type parameters when needed

---

## :material-link-variant: Related Topics

- [Syntax & Control Flow](My-Java-Rodemap/docs/notes/phase-1/topic-1-syntax-variables-control-flow/summary.md)
- [Lambdas & Streams](../topic-4-lambdas-streams/summary.md)
- [Collections Framework](../topic-5-collections-framework/summary.md)

---

## :material-bookshelf: References

- **Course:** Tim Buchalka - Java Programming Masterclass
- **Book:** Effective Java - Joshua Bloch (Items 26-33)
- **Book:** Core Java Volume I - Cay S. Horstmann (Collections Chapter)

---

*Completed: <!-- Add date --> | Confidence: X/10*
