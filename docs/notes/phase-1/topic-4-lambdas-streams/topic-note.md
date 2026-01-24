# :material-pencil: Topic Note: Lambdas & Streams

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Learning Objectives

- [ ] Master lambda expression syntax
- [ ] Understand functional interfaces
- [ ] Apply Stream API for data processing
- [ ] Use Optional to handle null safely

---

## :material-head-cog: Key Concepts from the Course

### Lambda Expressions

#### Lambda Syntax

```java
// Full syntax
(parameters) -> { statements; }

// Single expression (no braces needed)
(parameters) -> expression

// Single parameter (no parentheses needed)
parameter -> expression

// Examples
(a, b) -> a + b
x -> x * x
() -> System.out.println("Hello")
(String s) -> s.length()
```

#### Capturing Variables

<!-- Effectively final variables, closures -->



#### Comparison: Anonymous Class vs Lambda

<!-- When to use which -->



---

### Functional Interfaces

#### Core Functional Interfaces

| Interface | Abstract Method | Description |
|-----------|-----------------|-------------|
| `Predicate<T>` | `test(T t)` → boolean | Test a condition |
| `Function<T, R>` | `apply(T t)` → R | Transform input to output |
| `Consumer<T>` | `accept(T t)` → void | Consume input, no return |
| `Supplier<T>` | `get()` → T | Supply a value |
| `UnaryOperator<T>` | `apply(T t)` → T | Transform same type |
| `BinaryOperator<T>` | `apply(T t1, T t2)` → T | Combine two values |
| `BiFunction<T, U, R>` | `apply(T t, U u)` → R | Two inputs, one output |

#### @FunctionalInterface Annotation

<!-- Purpose and usage -->



---

### Method References

#### Types of Method References

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static | `ClassName::staticMethod` | `x -> ClassName.staticMethod(x)` |
| Instance (bound) | `object::instanceMethod` | `x -> object.instanceMethod(x)` |
| Instance (unbound) | `ClassName::instanceMethod` | `(obj, x) -> obj.instanceMethod(x)` |
| Constructor | `ClassName::new` | `x -> new ClassName(x)` |

#### Examples

```java
// Static method reference
list.forEach(System.out::println);

// Instance method reference
String::toLowerCase  // unbound
myString::toUpperCase  // bound

// Constructor reference
ArrayList::new
```

---

### Stream API

#### Creating Streams

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

#### Intermediate Operations (Lazy)

<!-- filter, map, flatMap, distinct, sorted, peek, limit, skip -->



#### Terminal Operations (Execute Pipeline)

<!-- forEach, collect, reduce, count, anyMatch, allMatch, findFirst -->



#### Stream Pipeline Pattern

```java
collection.stream()
    .filter(/* predicate */)
    .map(/* function */)
    .sorted(/* comparator */)
    .collect(Collectors.toList());
```

---

### Optional Class

#### Creating Optionals

```java
Optional.of(value)          // Non-null value
Optional.ofNullable(value)  // Nullable value
Optional.empty()            // Empty optional
```

#### Using Optionals

<!-- orElse, orElseGet, orElseThrow, map, flatMap, ifPresent -->



---

### Parallel Streams

#### When to Use

<!-- CPU-intensive operations, large data sets -->



#### Caution Areas

<!-- Stateful operations, ordering, overhead -->



---

## :material-lightbulb-on: Observed Ideas & Insights

### Declarative vs Imperative

<!-- How functional style changes your thinking -->



### Pipeline Thinking

<!-- Composing operations mentally -->



---

## :material-link-variant: Code Snippets & Examples

```java
// Example: Transform and filter a list
List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
```

---

## :material-help-circle: Questions to Explore

- [ ] How does lazy evaluation work internally?
- [ ] When does parallel stream hurt performance?
- [ ] 

---

## :material-pin: Quick Reference

| Concept | Key Point |
|---------|-----------|
| Lambda | Anonymous function, single abstract method |
| Stream | Pipeline of operations on data |
| Intermediate | Lazy, returns Stream |
| Terminal | Executes pipeline, returns result |
| Optional | Container for nullable values |

---

*Last Updated: <!-- Add date -->*
