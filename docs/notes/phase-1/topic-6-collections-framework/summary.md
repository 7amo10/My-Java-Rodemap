# :material-school: Summary: Collections Framework

> **Combined Knowledge from:** Tim Buchalka's Course + Effective Java + Core Java  
> **Mastery Level:** :material-star::material-star::material-star::material-star::material-star:

---

## :material-star-shooting: Topic Overview

The Java Collections Framework provides a unified architecture for representing and manipulating collections. Understanding its design, implementation details, and performance characteristics is essential for effective Java programming.

---

## :material-key: Core Interfaces

### Interface Hierarchy

```
                     Iterable<E>
                         │
                    Collection<E>
                         │
        ┌────────────────┼────────────────┐
        │                │                │
     List<E>          Set<E>          Queue<E>
        │                │                │
        │         ┌──────┴──────┐         │
        │     SortedSet<E>    │      Deque<E>
        │         │           │
        │   NavigableSet<E>   │
        │                     │
        └─────────────────────┘
        
                    Map<K,V>
                         │
                  SortedMap<K,V>
                         │
                NavigableMap<K,V>
```

---

## :material-format-list-bulleted: List Implementations

### ArrayList

**Use When:** Random access is frequent, mostly appending

```java
ArrayList<String> list = new ArrayList<>();
// Initial capacity: 10
// Growth: 50% increase (oldCapacity + oldCapacity >> 1)
```

### LinkedList

**Use When:** Frequent add/remove at both ends, implementing Queue/Deque

### Vector (Legacy)

**Use When:** Never in new code - use `Collections.synchronizedList()` instead

### Performance Matrix

| Operation | ArrayList | LinkedList |
|-----------|:---------:|:----------:|
| get(i) | `O(1)` | `O(n)` |
| add(E) | `O(1)*` | `O(1)` |
| add(i, E) | `O(n)` | `O(n)` |
| remove(i) | `O(n)` | `O(n)` |
| contains | `O(n)` | `O(n)` |
| iterator.remove | `O(n)` | `O(1)` |

*amortized

---

## :material-circle-multiple: Set Implementations

### HashSet

**Use When:** Fast lookup, no ordering needed

**Internal:** Backed by HashMap (elements are keys, dummy value)

### LinkedHashSet

**Use When:** Insertion order matters

**Internal:** Backed by LinkedHashMap

### TreeSet

**Use When:** Sorted order needed, range operations

**Internal:** Backed by TreeMap (Red-Black tree)

### EnumSet

**Use When:** Set of enum values

**Internal:** Bit vector - extremely efficient

### Performance Matrix

| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|:-------:|:-------------:|:-------:|
| add | `O(1)` | `O(1)` | `O(log n)` |
| remove | `O(1)` | `O(1)` | `O(log n)` |
| contains | `O(1)` | `O(1)` | `O(log n)` |
| iteration | unordered | insertion order | sorted |

---

## :material-map: Map Implementations

### HashMap

**Use When:** General purpose key-value storage

**Internals:**
- Array of buckets
- Each bucket: linked list → Red-Black tree (8+ elements)
- Load factor: 0.75 (rehash when 75% full)
- Treeification: bucket > 8 AND table > 64

### LinkedHashMap

**Use When:** Maintain insertion/access order

**Modes:**
- Insertion order (default)
- Access order (useful for LRU cache)

### TreeMap

**Use When:** Sorted keys, range queries

**Internal:** Red-Black tree ensuring O(log n) operations

### ConcurrentHashMap

**Use When:** Thread-safe map access

**Internal:** 
- Java 7: Segment-based locking
- Java 8+: Bucket-level CAS + synchronized

### Performance Matrix

| Operation | HashMap | LinkedHashMap | TreeMap | ConcurrentHashMap |
|-----------|:-------:|:-------------:|:-------:|:-----------------:|
| get | `O(1)` | `O(1)` | `O(log n)` | `O(1)` |
| put | `O(1)` | `O(1)` | `O(log n)` | `O(1)` |
| containsKey | `O(1)` | `O(1)` | `O(log n)` | `O(1)` |
| Thread-safe | :material-close: | :material-close: | :material-close: | :material-check: |

---

## :material-target: Queue & Deque

### PriorityQueue

**Use When:** Priority-based processing

**Internal:** Binary heap

### ArrayDeque

**Use When:** Stack or queue operations (faster than Stack and LinkedList)

**Internal:** Resizable circular array

---

## :material-head-cog: Key Internals

### HashMap Collision Resolution

```
Bucket Structure:
┌─────────────────────────────────┐
│ table[0] → null                 │
│ table[1] → Node → Node → Node   │  (linked list for < 8 entries)
│ table[2] → TreeNode (root)      │  (tree for >= 8 entries)
│    ...                          │
└─────────────────────────────────┘
```

### Red-Black Tree Properties

1. Every node is red or black
2. Root is black
3. Leaves (null) are black
4. Red node's children are black
5. Same black-height on all paths

**Guarantees:** O(log n) for search, insert, delete

---

## :material-lightbulb-on: Choosing the Right Collection

### Decision Tree

```
What do you need?
├── Key-Value pairs?
│   └── Map
│       ├── Thread-safe? → ConcurrentHashMap
│       ├── Sorted? → TreeMap
│       ├── Insertion order? → LinkedHashMap
│       └── None of above → HashMap
├── Unique elements?
│   └── Set
│       ├── Sorted? → TreeSet
│       ├── Insertion order? → LinkedHashSet
│       └── None of above → HashSet
├── Ordered sequence?
│   └── List
│       ├── Frequent random access? → ArrayList
│       └── Frequent add/remove at ends? → LinkedList
└── FIFO/Priority processing?
    └── Queue/Deque
        ├── Priority? → PriorityQueue
        └── FIFO/LIFO? → ArrayDeque
```

---

## :material-alert: Common Pitfalls

1. **Modifying collection while iterating**
   - Use Iterator.remove() or ConcurrentHashMap
   
2. **Wrong equals/hashCode for keys**
   - Always override both consistently

3. **Using mutable objects as keys**
   - Prefer immutable keys

4. **Not sizing collections upfront**
   - Use initial capacity when size is known

5. **Returning null instead of empty collection**
   - Return Collections.emptyList() etc.

---

## :material-lightbulb-on: Best Practices Checklist

- [ ] Choose collection based on access patterns
- [ ] Size collections appropriately when known
- [ ] Use interface types for declarations
- [ ] Prefer immutable collections when possible
- [ ] Return empty collections, not null
- [ ] Override equals/hashCode for custom keys
- [ ] Use ConcurrentHashMap for thread safety

---

## :material-link-variant: Related Topics

- [Arrays, Lists & Generics](../topic-3-arrays-lists-generics/summary.md)
- [Lambdas & Streams](../topic-4-lambdas-streams/summary.md)
- [OOP & Class Design](../topic-2-oop-class-design/summary.md)

---

## :material-bookshelf: References

- **Course:** Tim Buchalka - Java Programming Masterclass
- **Book:** Effective Java - Joshua Bloch (Items 54, 55, 58)
- **Book:** Core Java Volume I - Cay S. Horstmann (Chapter 9)
- **Documentation:** [Oracle Collections Framework](https://docs.oracle.com/javase/tutorial/collections/)

---

*Completed: <!-- Add date --> | Confidence: X/10*
