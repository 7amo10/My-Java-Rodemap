# :material-book-open-page-variant: Book Reading: OOP & Class Design Internals

> **Book:** Effective Java by Joshua Bloch  
> **Relevant Chapters:** 2-4 (Creating & Destroying Objects, Common Methods, Classes & Interfaces)  
> **Status:** :material-progress-wrench: In Progress

---

## :material-target: Reading Goals

- [ ] Understand best practices for object creation
- [ ] Learn common method contracts (equals, hashCode, toString)
- [ ] Master class and interface design principles

---

## :material-book-open-variant: Chapter 2: Creating and Destroying Objects

### Item 1: Consider Static Factory Methods Instead of Constructors

#### Key Takeaways

<!-- Your notes on static factory methods -->



#### Advantages Over Constructors

1. They have names
2. Not required to create new object each time
3. Can return subtypes
4. Returned class can vary based on parameters
5. Class of returned object need not exist when writing the method

---

### Item 2: Consider a Builder When Faced with Many Constructor Parameters

#### Key Takeaways

<!-- Builder pattern insights -->



#### When to Use

<!-- Guidelines for applying the pattern -->



---

### Item 3: Enforce the Singleton Property with a Private Constructor or an Enum Type

#### Key Takeaways

<!-- Singleton implementation best practices -->



---

### Item 7: Eliminate Obsolete Object References

#### Key Takeaways

<!-- Memory leak prevention -->



---

## :material-book-open-variant: Chapter 3: Methods Common to All Objects

### Item 10: Obey the General Contract When Overriding equals

#### The Contract

- **Reflexive:** `x.equals(x)` must return true
- **Symmetric:** `x.equals(y)` ↔ `y.equals(x)`
- **Transitive:** `x.equals(y)` && `y.equals(z)` → `x.equals(z)`
- **Consistent:** Multiple calls return same result
- **Non-null:** `x.equals(null)` must return false

#### Your Notes

<!-- Implementation insights -->



---

### Item 11: Always Override hashCode When You Override equals

#### Key Takeaways

<!-- hashCode contract and implementation -->



---

### Item 12: Always Override toString

#### Key Takeaways

<!-- toString best practices -->



---

## :material-book-open-variant: Chapter 4: Classes and Interfaces

### Item 15: Minimize the Accessibility of Classes and Members

#### Key Takeaways

<!-- Information hiding principles -->



---

### Item 17: Minimize Mutability

#### Key Takeaways

<!-- Immutable class design -->



#### Rules for Immutable Classes

1. Don't provide mutators
2. Ensure class can't be extended
3. Make all fields final
4. Make all fields private
5. Ensure exclusive access to mutable components

---

### Item 18: Favor Composition Over Inheritance

#### Key Takeaways

<!-- Why composition is often better -->



#### Wrapper Class Pattern

<!-- Decorator pattern insights -->



---

### Item 20: Prefer Interfaces to Abstract Classes

#### Key Takeaways

<!-- Interface design philosophy -->



---

## :material-head-cog: Theoretical Framework

### Mental Model for Object Design

<!-- Build your mental model here -->



### SOLID Principles Connection

<!-- How Effective Java relates to SOLID -->

- **S**ingle Responsibility: 
- **O**pen/Closed: 
- **L**iskov Substitution: 
- **I**nterface Segregation: 
- **D**ependency Inversion: 

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
| Static Factories | Item 1 | |
| Builder Pattern | Item 2 | |
| Singleton | Item 3 | |
| equals/hashCode | Items 10-11 | |
| Composition | Item 18 | |

---

*Last Updated: <!-- Add date -->*
