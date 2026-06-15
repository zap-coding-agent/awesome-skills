---
name: kent-beck-tdd
description: Use when implementing any feature, fixing any bug, or when deciding whether tests are "good enough" — applying Kent Beck's Test-Driven Development as its originator practiced it. Covers Red/Green/Refactor, the simple design rules, YAGNI, baby steps, and the key forcing questions Beck uses to keep code honest.
---

# Kent Beck — Test-Driven Development

## Core Philosophy

Beck's TDD is not about testing. It is about **design feedback at the speed of thought.** A failing test is a precise specification of the next smallest step. The cycle — Red, Green, Refactor — is a rhythm that keeps design honest and courage high.

> "Make it work. Make it right. Make it fast." — in that order, never conflated.

## The Cycle (non-negotiable sequence)

```
RED    Write exactly one failing test for the next behavior.
       Run it — watch it fail for the RIGHT reason.

GREEN  Write the MINIMUM code to make it pass.
       Fake it if you have to: hardcode the return, add the obvious branch.
       Do not write "correct" code; write PASSING code.

REFACTOR  With tests green, remove duplication and clarify intent.
          No new behavior. Tests must stay green throughout.
          Stop when the code says what it means, once.
```

**Red must fail for the right reason.** If your new test passes immediately without any code change, you've tested nothing — you haven't specified behavior.

**Fake it till you make it is real.** Returning a hardcoded value to get green is correct TDD. The test that forces you to generalize it comes next. Faking reveals what the tests are actually asserting.

## The Four Simple Design Rules (Beck, XP Explained)

In priority order — a design is simple if it:

1. **Passes all the tests** — correctness is non-negotiable.
2. **Reveals intention** — the next reader understands what and why.
3. **Has no duplication** — every piece of knowledge exists once.
4. **Has the fewest elements** — no extra classes, methods, or abstractions.

If rules 3 and 4 conflict: eliminate duplication first. Duplication is the root of most design problems.

## YAGNI — You Aren't Gonna Need It

Write code only to pass the current failing test. Not the one you imagine coming next. Not "we'll probably need this later." Only what the *current test demands.*

**Why:** Every abstraction has a cost — it must be understood, maintained, and sometimes deleted. An abstraction you don't yet need pays that cost now for a benefit that may never arrive. TDD's discipline is that the *tests drive the need*; needs that haven't been tested don't exist yet.

When you find yourself writing code "for flexibility," stop and ask: *which test forces this?* If there isn't one, delete it.

## Baby Steps

The size of each step matters. When stuck or scared, make the step smaller:

- Test fails in an incomprehensible way? → Undo to green; add a smaller assertion.
- Refactoring broke tests? → Undo; refactor in a smaller increment.
- Can't see the path? → Write the simplest test that moves in the right direction.

**There is no step too small.** A test that moves one character is a valid step. The point is never to be stuck; always to be moving.

## Beck's Forcing Questions

Apply these during and after TDD to keep the discipline honest:

| Question | What it reveals |
|---|---|
| *Does the test fail before I write code?* | Specification is real |
| *Am I faking? Which test forces generalization?* | Where duplication lives |
| *Is this the minimum code for green?* | Over-engineering caught early |
| *Where is the duplication in red/green?* | Refactoring target |
| *Would deleting this code break a test?* | Dead code (YAGNI violation) |
| *What is the simplest thing that could possibly work?* | Core Beck heuristic |
| *Am I testing behavior or implementation?* | Tests coupled to internals will rot |

## Triangulation

When you're not sure how to generalize, write a second example (test) that forces the right abstraction. Two data points force a general rule; one data point permits faking.

```
Test 1: add(1, 1) == 2  → fake with: return 2
Test 2: add(2, 3) == 5  → forced to generalize: return a + b
```

Triangulation is Beck's answer to "when do I stop faking?"

## What TDD is NOT

- Not a testing technique — it's a design technique that produces tests as a byproduct.
- Not a way to avoid thinking — you must know where you're going; TDD tells you how to get there.
- Not a guarantee of good design — the four rules still require judgment.
- Not slower — the perceived slowness of writing tests is paid back in debugging time eliminated.

## Common Violations (and what they signal)

| Violation | Signal |
|---|---|
| Writing multiple failing tests before going green | Fear of commitment / no clear next step → baby steps |
| Skipping red (writing code then test) | No specification; testing past behavior, not future |
| Faking green but never triangulating | No second test; the abstraction is still unforced |
| Refactoring to add features | Green/refactor conflated; new behavior needs a new red |
| Tests pass but code never changes shape | Skipping refactor → technical debt accumulates in the green phase |
| Tests know too much about internals | Testing implementation → tests become a change-drag, not a safety net |
