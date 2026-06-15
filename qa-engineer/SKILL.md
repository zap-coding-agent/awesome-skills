---
name: qa-engineer
description: Use when approaching software quality as a QA engineer — building a testing strategy, deciding what to test and at which layer, writing effective test cases, shifting quality left, reviewing code for testability, designing CI quality gates, or thinking about exploratory vs automated testing. Covers QA mindset, test pyramid, and quality advocacy.
---

# QA Engineer Mindset

## Core Philosophy

A QA engineer's job is **not to find bugs — it is to prevent them, and when prevention fails, to surface them before users do.** The highest-value QA work happens before code is written: clear requirements, testable designs, and agreed acceptance criteria. Bug-finding after the fact is the last line of defense, not the strategy.

> "Quality is not an act, it is a habit." — Aristotle (adopted by every QA team)

## The Testing Pyramid (and where it breaks)

```
        /\
       /E2E\           few — slow, brittle, expensive
      /------\
     /  Integ. \       moderate — cross-boundary, real services
    /------------\
   /  Unit Tests  \    many — fast, isolated, cheap
  /________________\
```

**The pyramid is a budget guide, not a religion:**
- Unit tests: business logic, algorithms, edge cases, domain rules. Cheap and fast; write many.
- Integration: service→repository, API handler→service, component→API. More valuable than pure unit for catching contract bugs.
- E2E: critical user flows only. One failing flaky E2E breaks CI and trains everyone to ignore it.

When the pyramid inverts (many E2E, few unit) — called the "ice cream cone anti-pattern" — the test suite is slow, fragile, and gives low signal. Invert it back.

## Shift Left: Quality Before Code

| Activity | When | QA role |
|---|---|---|
| Requirements review | Before design | Catch ambiguity; write acceptance criteria |
| Design review | Before implementation | Flag untestable designs; ask "how would we verify this?" |
| Test case design | During implementation | Exploratory test planning; edge cases the dev didn't consider |
| Code review | Before merge | Testability review; coverage gaps; missing error paths |
| Automated gates | On every PR | Unit/integration must pass; coverage thresholds enforced |
| Exploratory testing | Before release | Session-based; charter-driven; find what automation misses |

The later a bug is found, the more expensive it is to fix. A requirements ambiguity caught before code is written costs minutes. The same ambiguity found in production costs hours of debugging, a fix, a deploy, and potential user data loss.

## Writing Effective Test Cases

Every test case answers one question: *under these conditions, with this input, does this output happen?*

```
GIVEN  preconditions (user is authenticated, cart has 3 items)
WHEN   the action (user clicks "checkout")
THEN   the outcome (order is created, cart is emptied, confirmation is shown)
```

Good test cases are:
- **Specific**: one assertion per test. A test that fails tells you exactly what broke.
- **Independent**: no shared state between tests. Order of execution must not matter.
- **Fast**: test suites that take 20 minutes don't get run on every commit.
- **Trustworthy**: a failing test means the code is wrong, not the test.

## Equivalence Partitioning and Boundary Values

Don't test every input — test representatives from each meaningful class:

```
For a function accepting age (1-120):
  Equivalence classes: valid (1-120), below (0 or negative), above (>120)
  Boundary values:     0, 1, 120, 121  ← bugs hide at boundaries

Test: -1 (invalid), 0 (boundary invalid), 1 (boundary valid), 60 (mid), 120 (boundary valid), 121 (invalid)
Six tests cover the entire input space far better than testing 1-120 individually.
```

## Testability as a Design Quality Signal

If code is hard to test, it's usually badly designed. Common testability smells:

| Hard to test because... | Design problem | Fix |
|---|---|---|
| Requires a running DB | Logic entangled with persistence | Repository pattern, dependency injection |
| Needs to spin up the full app | Entry point has logic | Thin controller, logic in isolated service |
| Time-dependent (`new Date()`) | Direct clock access | Inject a clock; default to `Date.now` |
| Requires specific filesystem state | Direct file access | Wrap in an interface; inject a fake |
| Side effects during construction | Constructor does work | Lazy init, factory functions |

When you can't write a test for something without 200 lines of setup, push back on the design.

## CI Quality Gates

Minimum viable quality gates on every PR:

1. **Unit + integration tests pass** — any failing test blocks merge.
2. **Coverage threshold** — don't chase 100%; set a floor (e.g., 80% line, 70% branch) and enforce it. The floor is "no regression," not "perfection."
3. **Static analysis / lint** — catches classes of bugs automatically.
4. **Build passes** — type errors are bugs.

Optional but high-value:
- **Mutation testing** — verifies that your tests actually catch the bugs they're supposed to.
- **Contract tests** — verify API consumers and providers agree on the contract.
- **Performance budget** — bundle size, load time.

## Exploratory Testing

Automated tests verify what you thought to specify. Exploratory testing finds what you didn't think to specify.

**Session-based exploratory testing:**
1. Write a **charter**: "Explore the checkout flow as a first-time user with an international address."
2. Set a **timebox**: 60-90 minutes.
3. Explore freely; take notes on findings, surprises, confusing behaviors.
4. Debrief: file bugs; convert surprising behaviors to automated regression tests.

Exploratory testing is a skill — it is not "clicking around randomly." It is structured investigation guided by risk and hypotheses.

## The QA Mindset in Code Reviews

Ask:
- *What are the edge cases the dev didn't test?* (empty lists, nulls, concurrent calls, max values)
- *Is the error path tested?* (failures, timeouts, invalid inputs)
- *Is this testable?* (can it be called without the full stack?)
- *Are tests testing behavior or implementation?* (tests tied to internals rot)
- *Would a user notice if this broke?* (smoke test coverage)
