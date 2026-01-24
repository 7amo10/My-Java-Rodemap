# :material-book-open-page-variant: Book Reading: Arrays, Lists & Generics

> **Book:** Effective Java by Joshua Bloch  
> **Relevant Items:** 26-33 (Generics)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Reading Goals

- [ ] Understand the theory behind Java's generics
- [ ] Master wildcard usage with PECS principle
- [ ] Learn when to prefer lists over arrays
- [ ] Avoid common generics pitfalls

---

## :material-book-open-variant: Chapter Notes: Generics

### Item 26: Don't Use Raw Types

#### Key Takeaways

<!-- Why raw types are dangerous -->



#### Examples

```java
// Bad - raw type
List list = new ArrayList();

// Good - parameterized type
List<String> list = new ArrayList<>();
```

#### Exceptions

<!-- When raw types are acceptable -->

- Class literals: `List.class`
- instanceof operator: `o instanceof Set`

---

### Item 27: Eliminate Unchecked Warnings

#### Key Takeaways

<!-- How to handle unchecked warnings -->



#### @SuppressWarnings("unchecked")

<!-- When and how to use it properly -->



---

### Item 28: Prefer Lists to Arrays

#### Key Takeaways

<!-- Why lists are often better than arrays -->



#### Arrays vs Generics Comparison

| Aspect | Arrays | Generics |
|--------|--------|----------|
| Variance | Covariant | Invariant |
| Type checking | Runtime | Compile time |
| Reification | Reified | Erased |

#### Why This Matters

<!-- Covariant arrays lead to runtime errors -->

```java
// This compiles but fails at runtime!
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit"; // ArrayStoreException

// This won't compile - caught at compile time
List<Object> ol = new ArrayList<Long>(); // Compile error
```

---

### Item 29: Favor Generic Types

#### Key Takeaways

<!-- Designing your own generic classes -->



#### Stack Example (from the book)

<!-- Implementation notes -->



---

### Item 30: Favor Generic Methods

#### Key Takeaways

<!-- Designing generic methods -->



#### Type Inference

<!-- How Java infers type parameters -->



---

### Item 31: Use Bounded Wildcards to Increase API Flexibility

#### The PECS Principle

**P**roducer **E**xtends, **C**onsumer **S**uper

```java
// Producer - we read from it
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) push(e);
}

// Consumer - we write to it  
public void popAll(Collection<? super E> dst) {
    while (!isEmpty()) dst.add(pop());
}
```

#### When to Use Which

| Scenario | Wildcard | Example |
|----------|----------|---------|
| Only read T | `? extends T` | `Collection<? extends Number>` |
| Only write T | `? super T` | `Collection<? super Integer>` |
| Read and write | No wildcard | `Collection<T>` |

---

### Item 32: Combine Generics and Varargs Judiciously

#### Key Takeaways

<!-- Heap pollution and @SafeVarargs -->



---

### Item 33: Consider Typesafe Heterogeneous Containers

#### Key Takeaways

<!-- Advanced generic patterns -->



---

## :material-head-cog: Theoretical Framework

### Mental Model for Generics

<!-- Your mental model for understanding generics -->



### Type Erasure Deep Dive

<!-- What happens during compilation -->



---

## :material-thought-bubble: Reflections & Connections

### Connections to Course Material

<!-- How does the book complement Tim's course? -->



### New Perspectives Gained

<!-- What new insights did the book provide? -->



---

## :material-format-list-checks: Summary Points

1. 
2. 
3. 

---

## :material-pin: Bookmarks & Page References

| Topic | Item | Note |
|-------|------|------|
| Raw Types | Item 26 | |
| Lists > Arrays | Item 28 | |
| PECS | Item 31 | |
| @SafeVarargs | Item 32 | |

---

*Last Updated: <!-- Add date -->*
