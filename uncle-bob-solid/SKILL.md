---
name: uncle-bob-solid
description: Use when reviewing class/module design, deciding whether to extract/split/combine code, evaluating coupling and cohesion, or writing a justification for a design decision. Applies Robert C. Martin's SOLID principles with the original formulations, failure modes each prevents, and the common over-application traps.
---

# Uncle Bob — SOLID Principles

## Core Philosophy

SOLID principles are **not rules to follow; they are heuristics for managing change.** Software that is easy to extend without modification, easy to understand, easy to test — that is the goal. The principles describe the design properties that make that true.

> "The goal of software architecture is to minimize the human resources required to build and maintain the required system." — Robert C. Martin, Clean Architecture

## The Five Principles

### S — Single Responsibility Principle

**A module should have one, and only one, reason to change.**

"Reason to change" = a stakeholder or actor whose requirements govern this module. Not "one thing" in a naive sense — one *responsibility to one actor*.

```
❌ UserService: handles authentication, sends welcome emails, generates reports
    → Three actors: Security team, Marketing, Finance. Each change risks the others.

✅ AuthService (Security), NotificationService (Marketing), ReportingService (Finance)
    → Each module changes for one reason.
```

**Failure it prevents:** A change for one actor accidentally breaking behavior another actor depends on.

**Over-application trap:** Don't split until you have evidence of diverging change. One-line classes are not the goal.

### O — Open/Closed Principle

**A module should be open for extension but closed for modification.**

Add new behavior by adding new code, not changing existing code. Typically achieved through abstraction: define an interface/abstract type; add new behavior by implementing it, not by modifying the consumer.

```
❌ Shape renderer with a switch on shape type:
   if (shape.type === "circle") drawCircle(...)
   else if (shape.type === "rect") drawRect(...)     // adding triangle = modify this

✅ Shape interface with draw(); each shape implements it.
   Adding Triangle = new class, zero changes to renderer.
```

**Failure it prevents:** Regression in working code when adding a new variant.

**Over-application trap:** You can't predict every extension point. Don't abstract preemptively — let the first real duplication or extension drive the abstraction (see martin-fowler-refactoring Rule of Three).

### L — Liskov Substitution Principle

**Subtypes must be substitutable for their base types without breaking the program.**

Barbara Liskov's original: if S is a subtype of T, anywhere T is used, S can be used without changing program correctness. In practice: a subclass must honor the *contract* of its parent — postconditions, preconditions, invariants.

```
❌ Square extends Rectangle; Square.setWidth also sets height.
   Code that expects rectangles (area = w*h) breaks with a Square.
   (The classic LSP violation — "is-a" in English ≠ "is-a" in behavior.)

✅ Square and Rectangle are separate types; neither extends the other.
    Or: model via Shape → area(), without set methods.
```

**Failure it prevents:** Casting, `instanceof` checks, surprising behavior when a subtype flows through code expecting the parent.

**Heuristic:** If your subclass overrides a parent method to throw an exception or do nothing, that's a LSP violation. The subclass is not truly substitutable.

### I — Interface Segregation Principle

**Clients should not be forced to depend on interfaces they do not use.**

Fat interfaces create coupling to methods that don't apply. Narrow, role-specific interfaces let clients depend only on what they use.

```
❌ interface Worker { work(); eat(); sleep(); }
   RobotWorker implements Worker → must implement eat/sleep (meaningless)

✅ interface Workable { work(); }
   interface Feedable { eat(); }
   Human implements Workable, Feedable
   Robot implements Workable
```

**Failure it prevents:** Ripple changes — adding a method to a fat interface forces changes to every implementor, even ones that don't use it.

### D — Dependency Inversion Principle

**High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.**

Business rules (high-level) should not import database drivers, framework classes, or HTTP libraries (low-level). Define an abstraction (interface/port) owned by the high-level module; the low-level module implements it.

```
❌ OrderService (business) imports MySQLOrderRepository (infra)
   → Changing from MySQL to Postgres requires touching business code.

✅ OrderService depends on OrderRepository (interface, in business layer)
   MySQLOrderRepository implements OrderRepository (in infra layer)
   → Business code is untouched when persistence changes.
```

This is the principle behind dependency injection, hexagonal architecture, and ports-and-adapters. (See clean-architecture for the full picture.)

**Failure it prevents:** Business code that can't be tested without the database, can't run without the framework, changes when infrastructure changes.

## Applying SOLID in Review

Ask these during code review:

| Principle | Review question |
|---|---|
| SRP | *How many actors' requirements govern this class?* |
| OCP | *Does adding the next variant require modifying existing code?* |
| LSP | *Can I swap subclass for parent without checking which one it is?* |
| ISP | *Does any implementor of this interface implement a method it doesn't use?* |
| DIP | *Does this module import a concrete infrastructure type?* |

## The Meta-Principle (Fowler's complement)

SOLID tells you *what to avoid*. kent-beck-tdd tells you *when to refactor toward it*: wait for the second or third time the pain appears, then apply the principle. Premature SOLID is speculative generality — a code smell.

> "A good architecture maximizes the number of decisions NOT made." — Uncle Bob, Clean Architecture
