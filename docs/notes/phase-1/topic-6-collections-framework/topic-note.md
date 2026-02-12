# :material-pencil: Topic Note: Collections Framework

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Learning Objectives

- [ ] Master the Collections hierarchy
- [ ] Choose the right collection for each use case
- [ ] Understand internal implementations
- [ ] Apply collections efficiently in real-world scenarios

---

## :material-head-cog: Key Concepts from the Course

### Collections Hierarchy Overview

```
                    Iterable
                       │
                   Collection
                       │
          ┌────────────┼────────────┐
         List         Set         Queue
          │            │            │
    ┌─────┼─────┐  ┌───┼───┐   ┌────┼────┐
ArrayList  │  LinkedList │  │   │     │
       LinkedList  HashSet │  TreeSet  │
                         │       PriorityQueue
                   LinkedHashSet      ArrayDeque
                   
                      Map (separate hierarchy)
                       │
          ┌────────────┼────────────┐
       HashMap    LinkedHashMap   TreeMap
          │
    ConcurrentHashMap
```

---

### List Interface

#### ArrayList

<!-- Notes on ArrayList: backed by array, dynamic resizing -->

**Characteristics:**
- Backed by resizable array
- O(1) random access
- O(n) insertion/deletion in middle
- Good default choice for lists

#### LinkedList

<!-- Notes on LinkedList: doubly-linked list, implements Deque -->

**Characteristics:**
- Doubly-linked list
- O(n) random access
- O(1) insertion/deletion at known position
- Implements both List and Deque

#### ArrayList vs LinkedList

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| get(index) | O(1) | O(n) |
| add(end) | O(1)* | O(1) |
| add(index) | O(n) | O(n)** |
| remove(index) | O(n) | O(n)** |
| Iterator.remove() | O(n) | O(1) |

*amortized, **finding position is O(n), operation is O(1)

---

### Set Interface

#### HashSet

<!-- Notes on HashSet: backed by HashMap, no order guarantee -->



#### LinkedHashSet

<!-- Notes on LinkedHashSet: maintains insertion order -->



#### TreeSet

<!-- Notes on TreeSet: sorted, backed by TreeMap (Red-Black tree) -->



#### Choosing the Right Set

| Set Type | Ordering | Null | Performance |
|----------|----------|------|-------------|
| HashSet | None | Yes | O(1) |
| LinkedHashSet | Insertion | Yes | O(1) |
| TreeSet | Sorted | No* | O(log n) |

*TreeSet allows null only if using a Comparator that handles it

---

### Map Interface

#### HashMap

<!-- Notes on HashMap: hash table implementation -->

**Internal Structure:**
- Array of buckets
- Hash function determines bucket
- Collision resolution: linked list → Red-Black tree (Java 8+)

**Key Operations:**
- Load factor (default 0.75)
- Rehashing when threshold exceeded

#### LinkedHashMap

<!-- Notes on LinkedHashMap: maintains insertion/access order -->



#### TreeMap

<!-- Notes on TreeMap: Red-Black tree, sorted by keys -->



#### ConcurrentHashMap

<!-- Notes on ConcurrentHashMap: thread-safe, lock striping -->



---

### Queue & Deque

#### PriorityQueue

<!-- Notes on PriorityQueue: heap implementation -->



#### ArrayDeque

<!-- Notes on ArrayDeque: double-ended queue -->



---

### Special Collections

#### Collections Utility Class

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.binarySearch(sortedList, key);
Collections.unmodifiableList(list);
Collections.synchronizedList(list);
```

#### Immutable Collections (Java 9+)

```java
List.of(1, 2, 3);
Set.of("a", "b", "c");
Map.of("key1", "val1", "key2", "val2");
```

---

## :material-lightbulb-on: Observed Ideas & Insights

### Performance Patterns

<!-- When to choose which collection -->



### Memory Considerations

<!-- Memory overhead of different collections -->



---

## :material-link-variant: Code Snippets & Examples

```java
// Add your practice code snippets here

```

---

## :material-help-circle: Questions to Explore

- [ ] How does HashMap handle hash collisions?
- [ ] When does HashMap convert buckets to trees?
- [ ] What's the difference between fail-fast and fail-safe iterators?

---

## :material-pin: Quick Reference

| Collection | Best For |
|------------|----------|
| ArrayList | Random access, iteration |
| LinkedList | Frequent insert/delete at ends |
| HashSet | Fast uniqueness check |
| TreeSet | Sorted unique elements |
| HashMap | Key-value storage |
| TreeMap | Sorted key-value pairs |
| PriorityQueue | Priority-based processing |

---

*Last Updated: <!-- Add date -->*
