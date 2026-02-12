# :material-book-open-page-variant: Book Reading: Collections Framework

> **Book:** Effective Java by Joshua Bloch  
> **Relevant Items:** Items on Collections, plus Core Java Volume I  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Reading Goals

- [ ] Understand collection interface design principles
- [ ] Learn best practices for using collections
- [ ] Master proper equals/hashCode for collection usage
- [ ] Understand concurrent collection patterns

---

## :material-book-open-variant: Effective Java: Collection Items

### Item 54: Return Empty Collections or Arrays, Not Nulls

#### Key Takeaways

<!-- Why empty collections are better than null -->



#### Best Practice

```java
// Bad
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}

// Good
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() 
        ? Collections.emptyList() 
        : new ArrayList<>(cheesesInStock);
}
```

---

### Item 55: Return Optionals Judiciously

#### Key Takeaways

<!-- When Optional is appropriate for return types -->



---

### Item 52: Use Overloading Judiciously

#### Key Takeaways (Collection Context)

<!-- Why List.remove(int) vs List.remove(Object) is problematic -->



---

### Item 58: Prefer For-Each Loops to Traditional For Loops

#### Key Takeaways

<!-- Iteration best practices -->



#### When Traditional For is Needed

1. Filtering - removing elements
2. Transforming - replacing elements  
3. Parallel iteration - multiple collections

---

## :material-book-open-variant: Core Java Volume I: Collections Chapter

### Collection Interface Design

#### Key Takeaways

<!-- The interface hierarchy design philosophy -->



---

### List Implementations Deep Dive

#### ArrayList Internals

<!-- Capacity, growth factor, System.arraycopy -->



#### LinkedList Internals

<!-- Node structure, doubly-linked design -->



---

### Set Implementations Deep Dive

#### HashSet Internals

<!-- How it wraps HashMap -->



#### TreeSet Internals

<!-- NavigableSet, Red-Black tree properties -->



---

### Map Implementations Deep Dive

#### HashMap Deep Dive

<!-- Hash function, bucket structure, treeification -->

**Key Internal Details:**
- Initial capacity: 16
- Load factor: 0.75
- Treeification threshold: 8
- Untreeify threshold: 6
- Minimum tree capacity: 64

**Hash Collision Resolution:**
1. Java 7: Linked list only
2. Java 8+: Linked list → Red-Black tree when bucket size > 8

#### TreeMap Internals

<!-- Red-Black tree guarantees -->



---

### Concurrent Collections

#### ConcurrentHashMap Design

<!-- Lock striping, segment-based locking (pre-Java 8), bucket-level locking -->



#### CopyOnWriteArrayList

<!-- When to use, trade-offs -->



---

## :material-head-cog: Theoretical Framework

### Choosing the Right Collection

```
Need ordering?
├── No → Need uniqueness?
│         ├── Yes → HashSet
│         └── No → ArrayList
└── Yes → What kind of order?
          ├── Insertion order → LinkedHashSet/LinkedHashMap
          ├── Sorted order → TreeSet/TreeMap
          └── Access order → LinkedHashMap (accessOrder=true)
```

### Performance Complexity Summary

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|-----------|-----------|------------|---------|---------|---------|---------|
| add | O(1)* | O(1) | O(1) | O(log n) | O(1) | O(log n) |
| remove | O(n) | O(1)** | O(1) | O(log n) | O(1) | O(log n) |
| get/contains | O(1)/O(n) | O(n) | O(1) | O(log n) | O(1) | O(log n) |

*amortized, **at known position

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

| Topic | Reference | Note |
|-------|-----------|------|
| HashMap Internals | Core Java Ch. 9 | |
| Empty Collections | EJ Item 54 | |
| For-Each Loops | EJ Item 58 | |
| Concurrent Collections | Core Java Ch. 12 | |

---

*Last Updated: <!-- Add date -->*
