# :material-book-open-page-variant: Book Reading: Lambdas & Streams

> **Book:** Effective Java by Joshua Bloch  
> **Relevant Items:** 42-48 (Lambdas & Streams)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Reading Goals

- [ ] Understand best practices for lambda usage
- [ ] Learn when streams are appropriate
- [ ] Master stream API design principles
- [ ] Avoid common functional programming pitfalls

---

## :material-book-open-variant: Chapter Notes: Lambdas and Streams

### Item 42: Prefer Lambdas to Anonymous Classes

#### Key Takeaways

<!-- When and why lambdas are better -->



#### When Anonymous Classes Are Still Needed

- Creating instances of abstract classes
- Creating interfaces with multiple abstract methods
- Need access to `this` (the anonymous class instance)

---

### Item 43: Prefer Method References to Lambdas

#### Key Takeaways

<!-- When method references improve readability -->



#### The Decision

| Method Reference | Lambda |
|-----------------|--------|
| `String::toLowerCase` | `s -> s.toLowerCase()` |
| `System.out::println` | `x -> System.out.println(x)` |
| `Integer::sum` | `(a, b) -> a + b` |

#### When Lambda is Clearer

<!-- Cases where explicit lambda is better -->



---

### Item 44: Favor the Use of Standard Functional Interfaces

#### Key Takeaways

<!-- Why to prefer built-in interfaces -->



#### The Core Six

| Interface | Function Signature | Example |
|-----------|-------------------|---------|
| `UnaryOperator<T>` | `T apply(T t)` | `String::toLowerCase` |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | `BigInteger::add` |
| `Predicate<T>` | `boolean test(T t)` | `Collection::isEmpty` |
| `Function<T,R>` | `R apply(T t)` | `Arrays::asList` |
| `Supplier<T>` | `T get()` | `Instant::now` |
| `Consumer<T>` | `void accept(T t)` | `System.out::println` |

#### When to Create Custom

<!-- Guidelines for custom functional interfaces -->



---

### Item 45: Use Streams Judiciously

#### Key Takeaways

<!-- When streams help, when they hurt -->



#### Overusing Streams

<!-- Warning signs of stream abuse -->



#### Ideal Stream Use Cases

1. Transform elements
2. Filter elements
3. Combine elements
4. Collect into a collection
5. Find an element
6. Any/all match conditions

---

### Item 46: Prefer Side-Effect-Free Functions in Streams

#### Key Takeaways

<!-- Pure functions in stream operations -->



#### Anti-Pattern

```java
// Bad - mutating external state
Map<String, Long> freq = new HashMap<>();
words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
});

// Good - pure function
Map<String, Long> freq = words.stream()
    .collect(groupingBy(String::toLowerCase, counting()));
```

---

### Item 47: Prefer Collection to Stream as a Return Type

#### Key Takeaways

<!-- API design for stream/collection returns -->



#### Guidelines

- If client might need to iterate multiple times → Collection
- If result is very large or infinite → Stream
- If computation is expensive per element → Stream

---

### Item 48: Use Caution When Making Streams Parallel

#### Key Takeaways

<!-- When parallel streams help/hurt -->



#### Good Candidates for Parallelism

- Large data sets
- CPU-intensive operations
- Splittable sources (ArrayList, arrays)
- Stateless, associative operations

#### Bad Candidates

- Small data sets
- I/O-bound operations
- LinkedList, Stream.iterate()
- Stateful operations
- Ordering-dependent operations

---

## :material-head-cog: Theoretical Framework

### Mental Model for Functional Programming

<!-- Your mental model for thinking functionally -->



### Stream Pipeline Optimization

<!-- How JVM optimizes stream operations -->



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
| Lambdas vs Anonymous | Item 42 | |
| Method References | Item 43 | |
| Standard Interfaces | Item 44 | |
| Stream Judiciousness | Item 45 | |
| Side-Effect-Free | Item 46 | |
| Parallel Streams | Item 48 | |

---

*Last Updated: <!-- Add date -->*
