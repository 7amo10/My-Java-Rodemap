# Topic 3: Arrays, Lists & Autoboxing

> Understanding Java's core data structures — from fixed-size arrays to dynamic lists, wrapper classes, and enum types.

---

## :material-folder-open: Topic Structure

| Document | Purpose | Status |
|----------|---------|--------|
| [:material-pencil: Part 1 — Arrays](topic-note.md) | Array fundamentals, `java.util.Arrays`, multi-dimensional arrays, varargs | :material-check-circle: Complete |
| [:material-pencil: Part 2 — ArrayList](topic-note-part2.md) | Collections intro, `ArrayList` CRUD, searching, sorting, Array ↔ ArrayList conversion | :material-check-circle: Complete |
| [:material-pencil: Part 3 — LinkedList & Iterators](topic-note-part3.md) | LinkedList as List/Queue/Deque/Stack, `Iterator`, `ListIterator`, Travel Itinerary challenge | :material-check-circle: Complete |
| [:material-pencil: Part 4 — Autoboxing, Unboxing & Enums](topic-note-part4.md) | Wrapper classes, autoboxing/unboxing, Banking challenge, enum types & custom methods | :material-check-circle: Complete |
| [:material-book-open-page-variant: Book Reading](book-reading.md) | Effective Java insights (Items 28, 34–39) | :material-check-circle: Complete |
| [:material-school: Summary](summary.md) | Combined final understanding | :material-check-circle: Complete |

---

## :material-target: What You'll Master

- **Arrays** — Declaration, initialization, `java.util.Arrays` utilities, multi-dimensional arrays, varargs
- **ArrayList** — Resizable arrays, CRUD operations, searching, sorting, `List.of()` vs `Arrays.asList()`
- **LinkedList** — Doubly linked list, Queue/Deque/Stack interfaces, Big O performance comparison
- **Iterators** — `Iterator` and `ListIterator`, safe modification during iteration, bidirectional traversal
- **Autoboxing & Unboxing** — Primitives vs wrapper classes, automatic conversions, common pitfalls
- **Enums** — Predefined constants, `name()`/`ordinal()`/`values()`, switch statements, custom methods

---

## :material-bookshelf: Resources

### Primary Course
- [Tim Buchalka's Java Masterclass](https://www.udemy.com/course/java-the-complete-java-developer-course/) — Sections 9 & 10

### Book Reference
- **Effective Java** by Joshua Bloch — Items related to Lists and Autoboxing

### Course Sections Covered
| Section | Lectures | Part |
|---------|----------|------|
| Section 9: Arrays | 13 lectures | Part 1 |
| Section 10: Lists & Autoboxing | Lectures 1–6 | Part 2 |
| Section 10: LinkedList & Iterators | Lectures 7, 9–13 | Part 3 |
| Section 10: Autoboxing & Enums | Lectures 15–18, 20–21 | Part 4 |

### Key Internals to Understand
- How arrays store elements contiguously in memory (cache-friendly access)
- ArrayList's dynamic resizing strategy (grow by 50%, amortized O(1) add)
- LinkedList's node-based memory layout (non-contiguous, pointer overhead)
- Why `Iterator.remove()` is safe during iteration but `List.remove()` throws `ConcurrentModificationException`
- Autoboxing: when the JVM inserts `Integer.valueOf()` and `intValue()` calls
- Integer cache: `Integer.valueOf()` caches values [-128, 127] (identity vs equality trap)
- Enum's implementation as a class with `public static final` instances

---

## :material-checkbox-marked-outline: Progress Tracker

- [x] Complete Tim's course Section 9 (Arrays)
- [x] Complete Tim's course Section 10 (Lists, Iterators, Autoboxing, Enums)
- [x] Write Part 1 topic notes (Arrays)
- [x] Write Part 2 topic notes (ArrayList)
- [x] Write Part 3 topic notes (LinkedList & Iterators)
- [x] Write Part 4 topic notes (Autoboxing, Unboxing & Enums)
- [x] Read Effective Java related items
- [x] Complete book reading notes
- [x] Synthesize final summary

---

*Last Updated: 2026-02-11*
