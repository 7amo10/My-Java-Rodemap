# :material-pencil: Topic Note Part 3: Strings & StringBuilder

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Section:** 07. Mastering Java OOP Classes & Inheritance  
> **Status:** :material-check-circle: Complete

---

## :material-target: Learning Objectives

- [x] Master Text Blocks for multi-line strings (JDK 15+)
- [x] Understand escape sequences and format specifiers
- [x] Use `printf()` and `String.format()` for formatted output
- [x] Apply String inspection methods (length, charAt, indexOf)
- [x] Use String comparison methods (equals, contains, startsWith)
- [x] Perform String manipulation (substring, replace, join)
- [x] Understand StringBuilder for mutable string operations
- [x] Know when to use String vs StringBuilder

---

## :material-text-box-multiple: Text Blocks (JDK 15+)

### What is a Text Block?

A **Text Block** is a multi-line string literal introduced in Java 15. It eliminates the need for escape sequences and concatenation when working with multi-line text.

### Traditional vs Text Block

```java
// Traditional approach - messy with escape sequences
String bulletIt = "Print a Bulleted List:\n" +
        "\t\u2022 First Point\n" +
        "\t\t\u2022 Sub Point";

// Text Block approach - clean and readable
String textBlock = """
        Print a Bulleted List:
              • First Point
                  • Sub Point
        """;
```

### Text Block Rules

| Rule | Description |
|------|-------------|
| Opening delimiter | Must be `"""` followed by a newline |
| Closing delimiter | `"""` on its own line or at end of content |
| Whitespace | Leading whitespace relative to closing `"""` is preserved |
| Line breaks | Natural line breaks are included automatically |

!!! tip "Incidental Whitespace"
    The position of the closing `"""` determines the base indentation. All whitespace to the left of it is removed from each line.

---

## :material-format-text: Escape Sequences

### Common Escape Sequences

Use these special character combinations within strings:

| Escape Sequence | Description | Example Output |
|-----------------|-------------|----------------|
| `\n` | Newline | Next line |
| `\t` | Tab | Horizontal spacing |
| `\"` | Double quote | `"` |
| `\\` | Backslash | `\` |
| `\uXXXX` | Unicode character | `\u2022` → • (bullet) |

```java
String formatted = "Line 1\n\tIndented line 2\n\"Quoted\"";
// Output:
// Line 1
//     Indented line 2
// "Quoted"
```

---

## :material-format-float-center: printf() and Format Specifiers

### The `printf()` Method

`System.out.printf()` provides C-style formatted output:

```java
int age = 35;
int yearOfBirth = 2023 - age;

System.out.printf("Your age is %d%n", age);
System.out.printf("Age = %d, Birth year = %d%n", age, yearOfBirth);
```

### Common Format Specifiers

| Specifier | Type | Example |
|-----------|------|---------|
| `%d` | Decimal integer | `35` |
| `%f` | Floating point | `35.000000` |
| `%.2f` | Float with 2 decimals | `35.00` |
| `%s` | String | `"Hello"` |
| `%c` | Character | `'H'` |
| `%n` | Platform-specific newline | (newline) |
| `%6d` | Integer with width 6 | `    35` |

### Format Specifier Structure

```
%[flags][width][.precision]conversion
```

```java
// Width: right-align numbers in 6-character field
for (int i = 1; i <= 100000; i *= 10) {
    System.out.printf("Number: %6d%n", i);
}
// Output:
// Number:      1
// Number:     10
// Number:    100
// Number:   1000
// Number:  10000
// Number: 100000

// Precision: limit decimal places
System.out.printf("Price: %.2f%n", 19.99);  // Price: 19.99
```

### String Formatting Methods

Three ways to format strings:

```java
int age = 35;

// 1. printf - prints directly to console
System.out.printf("Your age is %d%n", age);

// 2. String.format() - static method, returns formatted string
String formatted = String.format("Your age is %d", age);

// 3. formatted() - instance method on String (JDK 15+)
String formatted = "Your age is %d".formatted(age);
```

!!! info "Prefer `%n` over `\\n`"
    `%n` produces the platform-specific line separator (works on Windows, Mac, Linux), while `\n` always produces the Unix newline which may not display correctly on all platforms.

---

## :material-magnify: String Inspection Methods

### Strings as Character Sequences

Strings are **indexed sequences of characters** starting at index 0:

```
Index:      0   1   2   3   4   5   6   7   8   9   10
Character:  H   e   l   l   o       W   o   r   l   d
```

### Key Inspection Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `length()` | `int` | Number of characters |
| `charAt(int)` | `char` | Character at specified index |
| `indexOf(char/String)` | `int` | First occurrence position (-1 if not found) |
| `lastIndexOf(char/String)` | `int` | Last occurrence position |
| `isEmpty()` | `boolean` | True if length is 0 |
| `isBlank()` | `boolean` | True if empty or only whitespace |

### Code Examples

```java
String str = "Hello World";

// Length and character access
System.out.println(str.length());      // 11
System.out.println(str.charAt(0));     // H
System.out.println(str.charAt(10));    // d (last character)

// Finding positions
System.out.println(str.indexOf('W'));        // 6
System.out.println(str.indexOf("World"));    // 6
System.out.println(str.indexOf('l'));        // 2 (first 'l')
System.out.println(str.lastIndexOf('l'));    // 9 (last 'l')

// Search from specific position
System.out.println(str.indexOf('l', 3));     // 3 (find 'l' starting at index 3)
System.out.println(str.lastIndexOf('l', 8)); // 3 (find 'l' backwards from index 8)

// Empty vs Blank
String empty = "";
String blank = "   \t\n";
System.out.println(empty.isEmpty());   // true
System.out.println(empty.isBlank());   // true
System.out.println(blank.isEmpty());   // false (has characters)
System.out.println(blank.isBlank());   // true (only whitespace)
```

### Print Information Helper

```java
public static void printInformation(String string) {
    int length = string.length();
    System.out.printf("Length = %d%n", length);
    
    if (string.isEmpty()) {
        System.out.println("String is Empty");
        return;
    }
    
    if (string.isBlank()) {
        System.out.println("String is Blank");
    }
    
    System.out.printf("First char = %c%n", string.charAt(0));
    System.out.printf("Last char = %c%n", string.charAt(length - 1));
}
```

---

## :material-compare: String Comparison Methods

### Comparison Methods Overview

| Method | Description |
|--------|-------------|
| `equals(Object)` | Exact match (case-sensitive) |
| `equalsIgnoreCase(String)` | Match ignoring case |
| `contentEquals(CharSequence)` | Content match (works with StringBuilder) |
| `contains(CharSequence)` | True if string contains substring |
| `startsWith(String)` | True if string starts with prefix |
| `endsWith(String)` | True if string ends with suffix |

### Code Examples

```java
String helloWorld = "Hello World";
String helloWorldLower = helloWorld.toLowerCase();

// Equality checks
if (helloWorld.equals(helloWorldLower)) {
    System.out.println("Values match exactly");  // Won't print
}

if (helloWorld.equalsIgnoreCase(helloWorldLower)) {
    System.out.println("Values match ignoring case");  // Prints
}

// Substring checks
if (helloWorld.startsWith("Hello")) {
    System.out.println("String starts with Hello");
}

if (helloWorld.endsWith("World")) {
    System.out.println("String ends with World");
}

if (helloWorld.contains("llo Wo")) {
    System.out.println("String contains 'llo Wo'");
}
```

!!! warning "equals() vs =="
    Always use `.equals()` for String comparison, not `==`:
    ```java
    String a = new String("Hello");
    String b = new String("Hello");
    
    System.out.println(a == b);       // false (different objects)
    System.out.println(a.equals(b));  // true (same content)
    ```

---

## :material-content-cut: String Manipulation Methods

### Two Types of Manipulation

1. **Cleanup methods** - Don't change meaning (trim, case changes)
2. **Transformation methods** - Create different text values

### Cleanup Methods

| Method | Description |
|--------|-------------|
| `trim()` | Removes leading/trailing whitespace |
| `strip()` | Unicode-aware trim (JDK 11+) |
| `toLowerCase()` | Converts to lowercase |
| `toUpperCase()` | Converts to uppercase |
| `indent(int)` | Adjusts indentation of each line (JDK 15+) |

### Transformation Methods

#### substring() - Extract Parts

```java
String birthDate = "25/11/1982";

// Get year (from index 6 to end)
String year = birthDate.substring(6);  // "1982"

// Get month (from index 3 to 5, exclusive)
String month = birthDate.substring(3, 5);  // "11"
```

#### join() - Combine Strings

```java
// Static method - joins with delimiter
String date = String.join("/", "25", "11", "1982");
// Result: "25/11/1982"

String csv = String.join(", ", "Apple", "Banana", "Orange");
// Result: "Apple, Banana, Orange"
```

#### concat() - Append Strings

```java
String date = "25";
date = date.concat("/");
date = date.concat("11");
// Result: "25/11"

// Method chaining (but inefficient!)
date = "25".concat("/").concat("11").concat("/").concat("1982");
```

!!! warning "Avoid Multiple concat() Calls"
    Each `concat()` creates a new String object. Use `+` operator with literals or StringBuilder instead:
    ```java
    // Better - compiler optimizes this
    String date = "25" + "/" + "11" + "/" + "1982";
    ```

#### replace() Methods

| Method | Description |
|--------|-------------|
| `replace(char, char)` | Replace all occurrences of character |
| `replace(String, String)` | Replace all occurrences of string |
| `replaceFirst(String, String)` | Replace first occurrence only |
| `replaceAll(String, String)` | Replace using regex pattern |

```java
String date = "25/11/1982";

System.out.println(date.replace('/', '-'));     // "25-11-1982"
System.out.println(date.replace("2", "00"));    // "005/11/1900"
System.out.println(date.replaceFirst("/", "--")); // "25--11/1982"
System.out.println(date.replaceAll("/", "---")); // "25---11---1982"
```

#### repeat() and indent() (JDK 11+/15+)

```java
// Repeat a string
String line = "-".repeat(20);  // "--------------------"

String abc = "ABC\n".repeat(3);
// "ABC
//  ABC
//  ABC"

// Adjust indentation
String indented = "ABC\n".repeat(3).indent(4);  // Add 4 spaces to each line
String outdented = "    ABC\n".repeat(3).indent(-2);  // Remove 2 spaces from each line
```

---

## :material-hammer-wrench: StringBuilder - Mutable Strings

### Why StringBuilder?

**String is immutable** - every modification creates a new object:

```java
String s = "Hello";
s = s.concat(" World");  // Creates NEW string, old one becomes garbage
```

**StringBuilder is mutable** - modifications happen in-place:

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies SAME object
```

### Creating StringBuilder

```java
// Four constructors
StringBuilder sb1 = new StringBuilder();              // Empty, capacity 16
StringBuilder sb2 = new StringBuilder(32);            // Empty, capacity 32
StringBuilder sb3 = new StringBuilder("Hello");       // With initial value
StringBuilder sb4 = new StringBuilder(anotherStringBuilder);
```

!!! info "Capacity vs Length"
    - **Length** = actual number of characters
    - **Capacity** = allocated space (can grow automatically)
    
    ```java
    StringBuilder sb = new StringBuilder();
    System.out.println(sb.length());    // 0
    System.out.println(sb.capacity());  // 16 (default)
    ```

### String vs StringBuilder Comparison

```java
// String - immutable
String helloWorld = "Hello" + " World";
helloWorld.concat(" and Goodbye");  // MISTAKE: result not assigned!
System.out.println(helloWorld);     // Still "Hello World"

// StringBuilder - mutable  
StringBuilder builder = new StringBuilder("Hello World");
builder.append(" and Goodbye");     // Modifies builder directly
System.out.println(builder);        // "Hello World and Goodbye"
```

### StringBuilder Methods

| Method | Description |
|--------|-------------|
| `append(...)` | Add to end (many overloaded versions) |
| `insert(int, ...)` | Insert at specified position |
| `delete(int, int)` | Delete from start to end index |
| `deleteCharAt(int)` | Delete character at index |
| `replace(int, int, String)` | Replace range with string |
| `reverse()` | Reverse all characters |
| `setLength(int)` | Truncate or extend (with nulls) |
| `toString()` | Convert to String |

### Code Examples

```java
StringBuilder builder = new StringBuilder("Hello World and Goodbye");

// Delete and Insert
builder.deleteCharAt(16);  // Remove 'G' at index 16
builder.insert(16, "g");   // Insert lowercase 'g'

// Can chain methods (returns self-reference!)
builder.deleteCharAt(16).insert(16, "g");

// Replace (different from String.replace!)
builder.replace(16, 17, "G");  // Replace character at index 16-17

// Reverse and truncate
builder.reverse().setLength(7);
System.out.println(builder);  // "eybdooG" (Goodbye reversed)
```

### Method Chaining

StringBuilder methods return `this`, enabling chaining:

```java
StringBuilder sb = new StringBuilder()
    .append("Hello")
    .append(" ")
    .append("World")
    .insert(0, "Say: ")
    .append("!");

System.out.println(sb);  // "Say: Hello World!"
```

---

## :material-scale-balance: When to Use What?

### String vs StringBuilder Decision Guide

| Scenario | Use | Why |
|----------|-----|-----|
| Simple text | `String` | Immutable, safe, optimized |
| Concatenating literals | `String` with `+` | Compiler optimizes |
| Building text in loops | `StringBuilder` | Avoids creating many objects |
| Many modifications | `StringBuilder` | Mutable, efficient |
| Thread-safe needed | `StringBuffer` | Synchronized (slower) |
| API return value | `String` | Standard convention |

### Performance Example

```java
// BAD - creates many String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result = result + i + ", ";  // New String each iteration!
}

// GOOD - modifies single StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i).append(", ");
}
String result = sb.toString();
```

---

## :material-code-braces: Complete Examples

### String Information Method

```java
public static void printInformation(String string) {
    int length = string.length();
    System.out.printf("Length = %d%n", length);
    
    if (string.isEmpty()) {
        System.out.println("String is Empty");
        return;
    }
    
    if (string.isBlank()) {
        System.out.println("String is Blank");
    }
    
    System.out.printf("First char = %c%n", string.charAt(0));
    System.out.printf("Last char = %c%n", string.charAt(length - 1));
}

public static void printInformation(StringBuilder builder) {
    System.out.println("StringBuilder = " + builder);
    System.out.println("Length = " + builder.length());
    System.out.println("Capacity = " + builder.capacity());
}
```

### Date Manipulation Example

```java
String birthDate = "25/11/1982";

// Extract year
int yearIndex = birthDate.indexOf("1982");
String year = birthDate.substring(yearIndex);  // "1982"

// Extract month
String month = birthDate.substring(3, 5);  // "11"

// Rebuild with different format
String newDate = String.join("-", "1982", "11", "25");  // "1982-11-25"

// Replace separators
String dashes = birthDate.replace('/', '-');  // "25-11-1982"
```

---

## :material-pin: Quick Reference

### Format Specifier Cheat Sheet

```
%d  = integer
%f  = floating point
%s  = string
%c  = character
%n  = newline
%6d = width 6, right-aligned
%.2f = 2 decimal places
```

### String vs StringBuilder

```
┌─────────────────┬────────────────────┬────────────────────┐
│     Aspect      │      String        │   StringBuilder    │
├─────────────────┼────────────────────┼────────────────────┤
│ Mutability      │ Immutable          │ Mutable            │
│ Thread Safety   │ Yes (immutable)    │ No                 │
│ Performance     │ New object on mod  │ Same object        │
│ Literal support │ Yes ("hello")      │ No (use new)       │
│ Main methods    │ concat, replace    │ append, insert     │
└─────────────────┴────────────────────┴────────────────────┘
```

---

## :material-help-circle: Questions Explored

- [x] How do I create multi-line strings easily?
- [x] What are format specifiers and how do I use printf?
- [x] How do I inspect and compare strings?
- [x] How do I manipulate string content?
- [x] What is StringBuilder and when should I use it?
- [x] What's the difference between String and StringBuilder?

---

## :material-navigation: Related Notes

| Part | Topic | Link |
|:----:|-------|------|
| 1 | Classes, Objects & Encapsulation | [← Part 1](topic-note.md) |
| 2 | Inheritance & Method Overriding | [← Part 2](topic-note-part2.md) |
| 3 | Strings & StringBuilder | **You are here** |
| 4 | Composition | [Part 4 →](topic-note-part4.md) |
| 5 | Encapsulation (Advanced) | [Part 5 →](topic-note-part5.md) |
| 6 | Polymorphism | [Part 6 →](topic-note-part6.md) |

---

*Last Updated: 2026-01-26*
