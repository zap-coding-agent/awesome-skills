---
name: martin-fowler-refactoring
description: Use when cleaning up existing code without changing behavior — recognizing code smells, choosing the right refactoring move, sequencing safe incremental steps, and knowing when to stop. Applies Fowler's refactoring catalog: extract/inline/move/rename mechanics with tests as a safety net.
---

# Martin Fowler — Refactoring

## Core Philosophy

Refactoring is **behavior-preserving transformation of code structure.** The purpose is to improve the design of existing code — make it easier to understand, cheaper to change — without altering what the software does.

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Fowler

The discipline: **never refactor and add features at the same time.** Put on the refactoring hat or the feature hat; switching mid-step multiplies risk.

## When to Refactor (the Rule of Three)

1. The **first time** you do something, just do it.
2. The **second time** you do something similar, wince but do it anyway.
3. The **third time** you do something similar, **refactor.**

Also refactor: before adding a feature ("make the change easy, then make the easy change"), when you find a bug (understand the code), and during code review.

## The Code Smells (most important to recognize)

| Smell | What you see | Primary fix |
|---|---|---|
| **Long Method** | Function doing many things | Extract Method |
| **Large Class** | Class knowing too much | Extract Class, Extract Superclass |
| **Long Parameter List** | 4+ parameters | Introduce Parameter Object, Preserve Whole Object |
| **Divergent Change** | One class changed for many reasons | Extract Class (per reason) |
| **Shotgun Surgery** | One change requires edits in many classes | Move Method/Field to consolidate |
| **Feature Envy** | Method uses another class's data more than its own | Move Method |
| **Data Clumps** | Same 3 fields appear together repeatedly | Extract Class |
| **Primitive Obsession** | string/int where a concept exists | Replace Primitive with Object |
| **Switch Statements** | Long conditionals on type | Replace Conditional with Polymorphism |
| **Parallel Inheritance Hierarchies** | Adding a subclass requires another somewhere | Move Method/Field |
| **Lazy Class** | A class doing barely anything | Inline Class |
| **Speculative Generality** | "We might need this" abstractions | Collapse Hierarchy, Inline Class |
| **Temporary Field** | Field only set in some paths | Extract Class, Introduce Null Object |
| **Message Chains** | `a.b().c().d()` | Hide Delegate |
| **Middle Man** | Class that mostly delegates | Remove Middle Man |
| **Inappropriate Intimacy** | Classes accessing each other's private fields | Move Method/Field, Extract Class |
| **Duplicate Code** | Same logic in two places | Extract Method, Pull Up Method |
| **Dead Code** | Never called | Delete it |
| **Data Class** | Only getters/setters, no behavior | Move Method in |
| **Comments** | Explaining unclear code | Rename, Extract Method until the comment is the name |

## The Catalog — Core Moves

### Extracting and Inlining

```
Extract Method      — pull lines into a named method; use when you need to explain what code does
Inline Method       — replace a call with its body; use when indirection no longer earns its keep
Extract Variable    — name a sub-expression; use when an expression is hard to read
Inline Variable     — remove a variable that only re-names an obvious expression
Extract Class       — split a class when it has too many responsibilities
Inline Class        — collapse a class that does too little into its caller
Extract Function    — same as Extract Method in function-oriented languages
```

### Moving Code

```
Move Method         — method belongs more to another class (Feature Envy fix)
Move Field          — field used more in another class
Move Statements     — move code into the method that uses it, or before/after a call
Split Phase         — split code that does two sequential things into two functions
```

### Organizing Data

```
Replace Primitive with Object   — when a string/number needs behavior or validation
Encapsulate Variable            — add getter/setter before moving or renaming a field
Replace Temp with Query         — replace a variable with a method call (enabling reuse)
Introduce Parameter Object      — replace a data clump with a class
Preserve Whole Object           — pass the object instead of several of its fields
```

### Simplifying Conditionals

```
Decompose Conditional           — extract condition and branches into named methods
Consolidate Conditional         — merge conditions that lead to the same result
Replace Nested Conditional      — with guard clauses (early return)
Replace Conditional with Polymorphism — type-switching replaced by subclass dispatch
Introduce Special Case          — replace null/edge-case checks with a special-case object
```

### Handling Inheritance

```
Pull Up Method / Field          — common to subclasses → move to superclass
Push Down Method / Field        — only one subclass uses it → move down
Extract Superclass              — create a superclass from shared parts
Replace Subclass with Delegate  — inheritance for one thing, use delegation instead
Replace Superclass with Delegate— "is-a" that isn't really an is-a
```

## Safe Mechanics (the how, not just the what)

Every refactoring has mechanics — the exact, reversible steps that keep tests green throughout:

**Extract Method:**
1. Create a new method; give it a name explaining its *purpose*, not its mechanics.
2. Copy the extracted code into the new method.
3. Check for variables local to the extracted fragment — pass as params, or return as value.
4. Replace the original code with a call.
5. Test.

The key: **test after every step**, not at the end. If a step breaks a test, you took too large a step — undo and try smaller.

## Sequencing Refactorings

Complex refactors are chains of atomic moves. Fowler's heuristic: start with the move that makes the next move easier.

> "Make the change easy (warning: this may be hard), then make the easy change."

Before adding a feature, ask: *what shape does the code need to be in for this change to be easy?* Refactor to that shape first, then make the change.

## The Boy Scout Rule

Leave the code cleaner than you found it. Not a complete rewrite — just one small improvement each time you touch an area. Compound interest applied to code quality.

## When NOT to Refactor

- **Broken tests.** Refactoring without a green test suite is reckless surgery. Fix tests first.
- **When it's easier to rewrite.** If the code is so bad that understanding it costs more than rewriting, rewrite — but with a test suite around the current behavior first.
- **Deadline pressure.** Schedule refactoring as part of feature work ("make the change easy first"), not as a separate big-bang cleanup.

**REQUIRED COMPANION:** kent-beck-tdd provides the safety net (tests) that makes refactoring safe. uncle-bob-solid provides the design goals that guide which refactoring to apply.
