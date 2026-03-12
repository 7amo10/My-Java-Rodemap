# :material-pencil: Topic Note: Java Streams API

> **Course:** Java Programming Masterclass — Tim Buchalka (Udemy)  
> **Status:** :material-pencil-outline: To Do

---

## :material-target: Learning Objectives

- [ ] Understand stream pipeline architecture (source → intermediate → terminal)
- [ ] Master intermediate operations (filter, map, flatMap, sorted, distinct, peek)
- [ ] Master terminal operations (collect, reduce, forEach, count, findFirst)
- [ ] Use Collectors effectively (toList, groupingBy, partitioningBy)
- [ ] Use Optional to handle nulls safely
- [ ] Know when to use parallel streams

---

## :material-head-cog: Key Concepts from the Course

### Creating Streams

```java
// From collection
collection.stream()

// From array
Arrays.stream(array)

// From values
Stream.of(1, 2, 3)

// Generate/iterate
Stream.generate(() -> "element")
Stream.iterate(0, n -> n + 1)
```

### Stream Pipeline Pattern

```java
collection.stream()
    .filter(/* predicate */)
    .map(/* function */)
    .sorted(/* comparator */)
    .collect(Collectors.toList());
```

---

<!-- Content will be added when the Streams section is covered -->

---

*Last Updated: <!-- Add date -->*
