# :material-school: Summary: Lambdas & Streams

> **Combined Knowledge from:** Tim Buchalka's Course + Effective Java  
> **Mastery Level:** :material-star::material-star::material-star::material-star::material-star:

---

## :material-star-shooting: Topic Overview

Embracing functional programming in Java through lambda expressions, the Stream API, and understanding when and how to apply these powerful features effectively.

---

## :material-key: Core Concepts

### 1. Lambda Expressions

#### Syntax Variations

```java
// Full syntax
(Type param1, Type param2) -> { return expression; }

// Inferred types
(param1, param2) -> expression

// Single parameter
param -> expression

// No parameters
() -> expression
```

#### Key Characteristics

- **Target typing** - Lambda type is inferred from context
- **Effectively final** - Captured variables must be effectively final
- **No `this` reference** - `this` refers to enclosing class

#### Under the Hood

<!-- How invokedynamic compiles lambdas -->



---

### 2. Functional Interfaces

#### The Standard Six (+ Variants)

```java
// Predicate - Test a condition
Predicate<String> isEmpty = String::isEmpty;

// Function - Transform input
Function<String, Integer> length = String::length;

// Consumer - Accept input, no return
Consumer<String> printer = System.out::println;

// Supplier - Supply a value
Supplier<Double> random = Math::random;

// UnaryOperator - Same input/output type
UnaryOperator<String> upper = String::toUpperCase;

// BinaryOperator - Combine two values
BinaryOperator<Integer> sum = Integer::sum;
```

#### Primitive Specializations

- `IntPredicate`, `LongFunction<R>`, `DoubleConsumer`
- Avoid autoboxing overhead

---

### 3. Method References

#### Types and When to Use

| Type | Example | Use When |
|------|---------|----------|
| Static | `Integer::parseInt` | Calling static method |
| Bound instance | `myStr::length` | Calling on specific object |
| Unbound instance | `String::length` | First param is receiver |
| Constructor | `ArrayList::new` | Creating new instances |

#### Decision: Lambda vs Method Reference

```java
// Method reference is clearer
list.forEach(System.out::println);

// Lambda might be clearer for complex logic
list.stream()
    .map(x -> x.getFirstName() + " " + x.getLastName())
```

---

### 4. Stream API Mastery

#### Stream Pipeline Structure

```
Source → Intermediate Operations (0+) → Terminal Operation (1)
```

#### Intermediate Operations (Lazy)

| Operation | Description | Example |
|-----------|-------------|---------|
| `filter` | Keep matching elements | `.filter(x -> x > 0)` |
| `map` | Transform elements | `.map(String::toUpperCase)` |
| `flatMap` | One-to-many transform | `.flatMap(Collection::stream)` |
| `distinct` | Remove duplicates | `.distinct()` |
| `sorted` | Sort elements | `.sorted(Comparator.reverseOrder())` |
| `limit` | Take first n | `.limit(10)` |
| `skip` | Skip first n | `.skip(5)` |
| `peek` | Debug/side-effects | `.peek(System.out::println)` |

#### Terminal Operations

| Operation | Returns | Description |
|-----------|---------|-------------|
| `collect` | Collection/other | Gather results |
| `forEach` | void | Execute action |
| `reduce` | Optional/value | Combine all elements |
| `count` | long | Count elements |
| `anyMatch` | boolean | Any element matches? |
| `allMatch` | boolean | All elements match? |
| `findFirst` | Optional | First element |
| `findAny` | Optional | Any element (parallel) |

#### Common Collectors

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap(keyMapper, valueMapper)
Collectors.groupingBy(classifier)
Collectors.partitioningBy(predicate)
Collectors.joining(delimiter)
Collectors.counting()
Collectors.summarizingInt(mapper)
```

---

### 5. Optional

#### Best Practices

```java
// Creating
Optional<String> opt = Optional.ofNullable(nullable);

// Using safely
opt.orElse("default")
opt.orElseGet(() -> computeDefault())
opt.orElseThrow(() -> new IllegalStateException())

// Transforming
opt.map(String::toUpperCase)
   .filter(s -> s.length() > 5)
   .ifPresent(System.out::println);
```

#### Anti-Patterns to Avoid

- `opt.get()` without checking
- Using Optional for fields
- Wrapping collections in Optional

---

## :material-head-cog: Key Internals

### Lazy Evaluation

- Intermediate operations don't execute until terminal operation
- Enables short-circuiting (`findFirst`, `anyMatch`)
- Processing happens element-by-element, not operation-by-operation

### Parallel Stream Considerations

| Good For | Bad For |
|----------|---------|
| Large datasets | Small datasets |
| CPU-intensive ops | I/O-bound ops |
| ArrayList, arrays | LinkedList, iterate() |
| Stateless operations | Stateful operations |

---

## :material-alert: Common Pitfalls

1. **Overusing streams**
   - Not every loop should become a stream
   
2. **Side effects in stream operations**
   - Keep operations pure
   
3. **Parallel stream misuse**
   - Often slower for small data
   
4. **Reusing streams**
   - Streams can only be consumed once
   
5. **Ignoring Optional**
   - Using null handling instead

---

## :material-lightbulb-on: Best Practices Checklist

- [ ] Prefer method references when clearer
- [ ] Use standard functional interfaces
- [ ] Keep stream operations side-effect-free
- [ ] Limit stream pipeline length for readability
- [ ] Consider Collection return types for APIs
- [ ] Benchmark before using parallel streams

---

## :material-link-variant: Related Topics

- [OOP & Class Design](../topic-2-oop-class-design/summary.md)
- [Arrays, Lists & Generics](../topic-3-arrays-lists-generics/summary.md)
- [Collections Framework](../topic-5-collections-framework/summary.md)

---

## :material-bookshelf: References

- **Course:** Tim Buchalka - Java Programming Masterclass
- **Book:** Effective Java - Joshua Bloch (Items 42-48)
- **Book:** Modern Java in Action - Urma, Fusco, Mycroft

---

*Completed: <!-- Add date --> | Confidence: X/10*
