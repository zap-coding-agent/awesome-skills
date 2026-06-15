---
name: rich-hickey-simplicity
description: Use when evaluating whether a design is genuinely simple or just familiar — distinguishing simple from easy, identifying complected concepts that should be separated, deciding whether to add an abstraction, or reviewing architecture for hidden complexity. Applies Rich Hickey's "Simple Made Easy" philosophy.
---

# Rich Hickey — Simplicity

## Core Philosophy

The most important distinction in software: **simple vs easy.** They are not synonyms and conflating them is the source of most accidental complexity.

- **Simple**: one braid, one role, one concept. From *simplex* — one fold. **Objective.** A thing is simple if it has no interleaving with other things. It can be analyzed in isolation.
- **Easy**: near to hand, familiar, close to our capabilities. **Subjective.** What is easy for one person is hard for another.

> "Simplicity is a prerequisite for reliability." — Hickey, "Simple Made Easy" (Strange Loop 2011)

The trap: we reach for what is *easy* (familiar libraries, ORMs, frameworks) and call the result *simple*. Easy things can be profoundly complex. Simple things may take effort to learn. Optimize for simplicity; you'll live with it far longer than the moment of learning.

## Complect vs Compose

**Complect**: to interleave, braid together. From Latin *complectere*. When two concepts are complected, you cannot change one without the other. This is the root of brittleness.

**Compose**: to place together while keeping separate. True composition requires the components to be genuinely independent.

> "If you want everything to be familiar you'll never learn anything new." — Hickey

Examples of complecting:

| What gets complected | Why it's a problem |
|---|---|
| State + identity | Can't reason about past values; tests see mutable objects |
| Logic + time (place-oriented programming) | Code that modifies in place hides when values changed |
| Business rules + framework concerns | Can't run rules without the framework; can't test without the DB |
| Data + behavior (OOP defaults) | Encapsulation creates barriers to generic processing |
| What + how (imperative loops) | Implementation detail obscures intent |

## The Simplicity / Complexity Tool Inventory

Hickey's comparison (from "Simple Made Easy"):

| Construct | Complects | Simple alternative |
|---|---|---|
| State | Everything it touches | Values (immutable data) |
| Objects | State + identity + behavior | Data + functions + polymorphism separated |
| Methods | Function + state + namespace | Functions |
| Syntax | Meaning + order | Data (homoiconicity) |
| Inheritance | Types + parentage | Protocols/interfaces |
| Conditionals | Logic + order | Rules / maps |
| Variables | Value + time | Values |
| Loops | What + how | Higher-order functions / sequences |

This isn't "never use objects" — it's "be conscious of what you're complecting and whether it's necessary."

## Information vs Place

One of Hickey's most practical points: **most OOP mutates in place when it should produce information.**

> "Place-oriented programming": programs that reach into the world, change state, and rely on the next caller finding the changed state. Concurrency makes this catastrophic; debugging makes it miserable.

The alternative: functions that take values, produce values. No shared mutable state. State lives at the edges (databases, message queues); the processing in the middle is pure.

## Simplicity in Practice — the Forcing Questions

Apply when designing or reviewing:

| Question | What it catches |
|---|---|
| *What concepts are interleaved here?* | Complecting — find the braid |
| *Can I understand this in isolation?* | True simplicity — no external state needed |
| *Is this simple or just familiar?* | Easy vs simple confusion |
| *What would this look like as data?* | Behavior that should be data |
| *Who owns the when?* | Time/state hidden in the abstraction |
| *How do I test this without the framework?* | Logic complected with infrastructure |
| *If I delete this, what breaks?* | Hidden coupling |

## Simplicity is Not Minimalism

Simple is not small. Simple means *not interleaved*. A large program built from simple, composable pieces is simple. A short program that tangles state, time, and logic is complex regardless of its line count.

## Applied: Evaluating a Design

```
Proposed: User class with getId(), getName(), save(), sendWelcomeEmail(), generateReport()

Is it simple?
- save() complects User data with persistence → not simple (domain + infra)
- sendWelcomeEmail() complects User identity with notification side effect → not simple
- generateReport() complects User data with presentation → not simple

Simple alternatives:
- User: plain data structure (id, name, email) — immutable value
- UserRepository: persistence concern, separate
- NotificationService: side-effect concern, separate
- ReportRenderer: presentation concern, separate

Each can be understood, tested, and changed in isolation.
```

## The Trade-off Hickey Accepts

Simplicity often requires more upfront design. The "easy path" (ActiveRecord, God classes, mutable singletons) delivers features faster early. The cost appears later, when the complected system resists change, breaks under concurrency, and requires global understanding to modify any part.

> "Complected systems are not systems you can maintain." — Hickey

Build simple. The investment is in the design; the return is in every future change.
