# :material-pencil: Topic Note: Syntax, Variables & Control Flow

> **Course:** Java Programming Masterclass - Tim Buchalka (Udemy)  
> **Section:** 05. Mastering Java Expressions, Statements, Code Blocks, And Method Overloading  
> **Status:** :material-check-circle: Completed

---

## :material-target: Learning Objectives

- [x] Understand Java keywords and their role in the language
- [x] Master the difference between expressions and statements
- [x] Apply whitespace and indentation for readable code
- [x] Harness code blocks with if-then-else control flow
- [x] Design reusable methods with parameters and return values
- [x] Implement method overloading for flexible APIs

---

## :material-head-cog: Key Concepts from the Course

### Java Keywords

Java has **51 reserved keywords** that form the vocabulary of the language. These cannot be used as identifiers (variable names, class names, method names).

#### Key Points
- Keywords appear in **blue** in IntelliJ - this visual cue confirms they're reserved
- As of Java 9, the underscore (`_`) by itself is a keyword
- JDK 17 introduced **16 contextual keywords** (only keywords in specific contexts)
- `true`, `false`, and `null` are not technically keywords but **literal values** - still cannot be used as identifiers

```java
// ❌ Invalid - "int" is a reserved keyword
int int = 5;  // Error: identifier expected

// ✅ Valid - partial keyword is fine
int int2 = 5;  // Works because "int2" is not a reserved word
```

!!! tip "Pro Tip"
    When you get weird "identifier expected" errors, check if you're accidentally using a reserved keyword as a name.

---

### The Building Blocks of Java Code

Java code follows a **hierarchical structure** from smallest to largest unit:

```
Expression → Statement → Code Block → Method → Class
```

#### 1. Expressions

An **expression** is a construct that computes to a **single value**. It's built using values, variables, and operators.

```java
double kilometers = 100 * 1.609344;
//                  ↑↑↑↑↑↑↑↑↑↑↑↑↑↑
//                  This is the expression
```

**What's NOT part of an expression:**
- The data type declaration (`double`)
- The semicolon (`;`)

**Multiple expressions in one statement:**

```java
int highScore = 50;
if (highScore > 25) {           // Expression: highScore > 25
    highScore = 1000 + highScore;  // Two expressions:
                                    // 1. 1000 + highScore
                                    // 2. highScore = 1000 + highScore
}
```

#### 2. Statements

A **statement** is a complete unit of execution, typically ending with a semicolon.

```java
int myVariable = 50;      // Declaration statement
myVariable++;              // Expression statement
System.out.println("Hi"); // Method call statement
```

**Key Insight:** The semicolon transforms an expression into a statement.

**Statements can span multiple lines:**

```java
System.out.println("This is a test " +
                   "and another part " +
                   "still more");  // All one statement!
```

**Multiple statements on one line (avoid this):**

```java
int x = 5; x++; System.out.println(x);  // Technically valid but hard to read
```

!!! warning "Best Practice"
    Put one statement per line. Break long statements across lines. This makes code readable and maintainable.

#### 3. Code Blocks

A **code block** is a set of statements enclosed in curly braces `{}`, grouped to achieve a single goal.

```java
if (condition) {
    // This is a code block
    statement1;
    statement2;
}
```

---

### Whitespace & Indentation

**Whitespace** is any extra horizontal or vertical spacing in your code. Java completely **ignores whitespace** - it's purely for human readability.

```java
// These are identical to Java:
int x=50;
int x = 50;
int    x    =    50;
```

#### Google Java Style Guide Recommendations

| Rule | Example |
|------|---------|
| Spaces around operators | `x = 5` not `x=5` |
| One statement per line | Always |
| Blank lines for logical grouping | Between variable groups |
| Consistent indentation | Use IDE auto-format |

**IntelliJ Shortcut:** `Ctrl+Alt+L` (Windows/Linux) or `Cmd+Option+L` (Mac) to auto-format code.

---

### Control Flow: if-then-else

The `if` statement is the fundamental decision-making construct in Java.

#### Basic Syntax

```java
if (condition) {
    // Code executes when condition is true
}
```

#### if-else (Either-Or)

```java
if (score < 5000) {
    System.out.println("Score is less than 5000");
} else {
    System.out.println("Score is 5000 or more");
}
```

**Guarantee:** One of these blocks ALWAYS executes.

#### if-else-if-else (Multiple Conditions)

```java
if (score < 5000 && score > 1000) {
    System.out.println("Between 1000 and 5000");
} else if (score < 1000) {
    System.out.println("Less than 1000");
} else {
    System.out.println("5000 or more");
}
```

**Execution Flow:**
1. Check first `if` condition
2. If false, check `else if` condition(s) in order
3. If all false, execute `else` block
4. **Once ANY condition is true, skip all remaining checks**

!!! success "Best Practice"
    Always use code blocks `{}` even for single-line conditionals. This prevents bugs when adding more lines later.

---

### Methods: Reusable Code

A **method** declares executable code that can be invoked, passing a fixed number of values as arguments.

#### Why Methods?

| Problem | Solution |
|---------|----------|
| Code duplication | Write once, call many times |
| Maintenance nightmare | Change in one place |
| Unreadable main method | Logical organization |
| Inflexible code | Parameters allow variation |

#### Method Declaration Anatomy

```java
public static void calculateScore() {
//│      │      │    └─ Method name (lowerCamelCase)
//│      │      └─ Return type (void = returns nothing)
//│      └─ Can be called without creating an object
//└─ Accessible from anywhere
    
    // Method body (code block)
}
```

#### Calling a Method

```java
calculateScore();  // Executes the method's code
```

**What happens:**
1. Execution jumps to the method
2. All statements in the method body execute
3. Execution returns to where it left off

---

### Method Parameters & Arguments

**Parameters** make methods flexible and reusable by accepting input data.

```java
public static void calculateScore(boolean gameOver, int score, int levelCompleted, int bonus) {
//                               └─────────────── Parameters (definition) ───────────────┘
    // Can use gameOver, score, levelCompleted, bonus as local variables
}
```

**Calling with arguments:**

```java
calculateScore(true, 800, 5, 100);
//             └── Arguments (actual values passed) ──┘
```

#### Parameters vs Arguments

| Term | Where | What |
|------|-------|------|
| **Parameter** | Method declaration | Definition (type + name) |
| **Argument** | Method call | Actual value passed |

#### Rules for Arguments

1. Must match **number** of parameters
2. Must match **types** of parameters  
3. Must match **order** of parameters

```java
// Method signature
void process(String name, int age, boolean active)

// ✅ Valid calls
process("Tim", 30, true);

// ❌ Invalid calls
process("Tim", 30);           // Wrong number
process(30, "Tim", true);     // Wrong order/types
process("Tim", "30", true);   // Wrong type for age
```

---

### Return Values

Methods can send data back to the calling code using the `return` keyword.

```java
public static int calculateScore(int score, int bonus) {
//             └─ Return type (int)
    int finalScore = score + bonus;
    return finalScore;  // Sends value back to caller
//  └─ Required when return type is not void
}
```

**Using the returned value:**

```java
// Option 1: Store in variable
int result = calculateScore(500, 100);

// Option 2: Use directly in expression
System.out.println("Score: " + calculateScore(500, 100));

// Option 3: Ignore the return (valid but usually pointless)
calculateScore(500, 100);  // Value is discarded
```

#### void vs Return Type

| Declaration | Return Statement |
|-------------|-----------------|
| `void` | Optional (implicit return at end) |
| Any data type | **Required** |

---

### Method Overloading

**Method overloading** allows multiple methods with the **same name** but **different parameters**.

```java
public static int calculateScore(String playerName, int score) {
    System.out.println("Player " + playerName + " scored " + score);
    return score * 1000;
}

public static int calculateScore(int score) {  // Overloaded!
    return calculateScore("Anonymous", score);  // Calls the other version
}

public static int calculateScore() {  // Another overload!
    return 0;
}
```

#### Method Signature

The **signature** determines uniqueness:

| Part of Signature | Example |
|-------------------|---------|
| Method name | `calculateScore` |
| Parameter types | `(String, int)` |
| Parameter order | `(int, String)` ≠ `(String, int)` |
| Number of parameters | `()` ≠ `(int)` ≠ `(int, int)` |

**NOT part of signature:**
- Return type
- Parameter names

```java
// ✅ Valid overloads
void doSomething(int a)
void doSomething(float a)           // Different type
void doSomething(int a, float b)    // Different count
void doSomething(float b, int a)    // Different order

// ❌ Invalid - same signature
void doSomething(int x)             // Same as first (name doesn't matter)
int doSomething(int a)              // Same as first (return type doesn't matter)
```

#### Simulating Default Parameters

Java doesn't support default parameter values, but overloading provides the same functionality:

```java
// Full signature
public static int calculateScore(String playerName, int score) {
    return score * 1000;
}

// Simplified - uses default for playerName
public static int calculateScore(int score) {
    return calculateScore("Anonymous", score);  // Delegates to full version
}
```

---

## :material-lightbulb-on: Observed Ideas & Insights

### The Power of Abstraction

Methods fundamentally change how we think about code:
- **Without methods:** Think line by line
- **With methods:** Think in terms of operations and actions

This is the first step toward **object-oriented thinking**.

### Code Duplication is Evil

Duplicated code leads to:

1. **Bugs** - Fix in one place, forget another
2. **Maintenance Hell** - Changes multiply
3. **Inconsistency** - Different behaviors accidentally

**Rule of Thumb:** If you copy-paste code more than once, create a method.

### The Boolean Trap in Control Flow

Notice how easily bugs creep in with complex conditions:

```java
if (score < 5000 && score > 1000)  // What about score == 5000 or score == 1000?
```

Always trace through edge cases mentally.

---

## :material-link-variant: Code Snippets & Examples

### Complete Method Pattern

```java
public class Main {
    public static void main(String[] args) {
        // Variables
        boolean gameOver = true;
        int score = 800;
        int levelCompleted = 5;
        int bonus = 100;
        
        // Method call with variables
        int finalScore = calculateScore(gameOver, score, levelCompleted, bonus);
        System.out.println("Final Score: " + finalScore);
        
        // Method call with literals
        int secondScore = calculateScore(true, 10000, 8, 200);
        System.out.println("Second Score: " + secondScore);
    }
    
    public static int calculateScore(boolean gameOver, int score, 
                                      int levelCompleted, int bonus) {
        if (gameOver) {
            int finalScore = score + (levelCompleted * bonus);
            finalScore += 1000;  // Bonus for completing
            return finalScore;
        }
        return -1;  // Game not over
    }
}
```

### Method Overloading Pattern

```java
public class Calculator {
    // Version 1: Full parameters
    public static int add(int a, int b) {
        return a + b;
    }
    
    // Version 2: Three parameters
    public static int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // Version 3: Different types
    public static double add(double a, double b) {
        return a + b;
    }
}
```

---

## :material-help-circle: Questions to Explore

- [x] What makes a statement different from an expression?
- [x] Why can't we use reserved keywords as variable names?
- [x] How does Java determine which overloaded method to call?
- [ ] What happens if two overloaded methods could both match the arguments?
- [ ] How do varargs interact with method overloading?

---

## :material-pin: Quick Reference

| Concept | Definition |
|---------|------------|
| **Expression** | Code that computes to a single value |
| **Statement** | Complete unit of execution (usually ends with `;`) |
| **Code Block** | Group of statements in `{}` |
| **Method** | Named, reusable block of code |
| **Parameter** | Variable declared in method signature |
| **Argument** | Value passed to method when called |
| **Return Type** | Data type of value method sends back |
| **Method Signature** | Name + parameter types (determines uniqueness) |
| **Overloading** | Multiple methods, same name, different signatures |

---

## :material-check-all: Key Takeaways

1. **Expressions compute values; statements execute actions**
2. **Whitespace is for humans; semicolons are for Java**
3. **Always use code blocks** even for single statements
4. **Methods eliminate duplication** and improve maintainability
5. **Parameters make methods flexible**; arguments are the actual data
6. **Return values enable two-way communication** between caller and method
7. **Overloading creates flexible APIs** based on signature uniqueness
8. **Overloading can simulate default parameters** through delegation

---

*Last Updated: 2026-01-22*
