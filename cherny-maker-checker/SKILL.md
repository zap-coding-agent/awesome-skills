---
name: cherny-maker-checker
description: Use when designing a multi-agent system, an agent is self-certifying its own work as complete, outputs aren't trustworthy without independent review, or a loop needs a quality gate. Applies Boris Cherny's maker-checker pattern — the most important structural move inside any agent loop.
---

# Cherny — Maker-Checker

> "The single most useful structural move in a loop is splitting the agent that writes from the agent that checks." — Boris Cherny

## The Problem with Self-Certification

A single agent that does work and certifies its own completion has a fundamental conflict of interest. It has context about what it tried, it's invested in the task being done, and it will find reasons the output satisfies the goal. This isn't a hallucination problem — it's an evaluation structure problem.

The same agent that wrote the code is the worst possible reviewer of that code. Not because it's deceptive, but because it shares all the same blind spots that led to the mistakes in the first place.

## The Pattern

```
Task
  │
  ▼
┌─────────────┐      work product       ┌──────────────────┐
│    MAKER    │ ─────────────────────►  │    CHECKER       │
│             │                         │                   │
│ Full context│                         │ Fresh context     │
│ Knows what  │                         │ Only sees output  │
│ was tried   │                         │ Asks: does this   │
│             │                         │ actually satisfy  │
│             │                         │ the goal?         │
└─────────────┘                         └──────────────────┘
                                                │
                                    ┌───────────┴───────────┐
                                    │                       │
                                   PASS                   FAIL
                                    │                       │
                               next step             back to maker
                                                     with findings
```

**The maker** has full working context — what it tried, why it made choices, what it considered and rejected. It produces the work.

**The checker** starts fresh — it sees only the output and the goal condition. It asks only: does this output satisfy the goal? It has no attachment to the approach, no knowledge of the effort involved, no reason to stretch the definition of "done."

## Implementation

Separate by at least one of:
- **Different agent invocation** — a new Claude session with no access to the maker's conversation
- **Different instructions** — the checker's system prompt is skeptical, not collaborative; it's looking for failure modes, not ways to validate
- **Different model** — sometimes a cheaper/faster model works well as a checker since it's just pattern-matching against a clear criterion

The checker's prompt should be radically different from the maker's:

```
# Maker prompt (collaborative, constructive)
"Implement the feature described in issue #42. Write tests. Follow the patterns in src/orders/."

# Checker prompt (adversarial, specific)
"Review the diff below. The goal was: 'all existing tests pass, new tests cover the happy path and 
the credit-limit error case, no new TypeScript errors.' Report: PASS or FAIL. If FAIL, list 
exactly what's missing or broken. Be specific — line numbers and filenames."
```

## Escalation on Repeated Failure

A loop where the maker keeps failing the checker needs a circuit breaker:

```
Attempt 1: Maker → Checker → FAIL → Maker with checker findings
Attempt 2: Maker → Checker → FAIL → Maker with both rounds of findings
Attempt 3: Maker → Checker → FAIL → ESCALATE: write findings to memory, open draft PR, 
                                     alert human — do not auto-merge
```

Without a circuit breaker, a loop that can't satisfy its goal runs forever and burns budget.

## Where to Apply the Pattern

| Situation | Maker | Checker |
|---|---|---|
| Code generation | Agent that writes the code | Agent (or CI) that runs tests |
| PR review | Agent that writes review comments | Human (the loop surfaces findings; human decides) |
| Content generation | Agent that writes the doc | Agent with a rubric: accurate? complete? no hallucinations? |
| Data transformation | Agent that runs the pipeline | Agent that samples output and validates against schema |
| Bug fix | Agent that applies the fix | Fresh agent + test suite that confirms the symptom is gone |

**REQUIRED COMPANION:** cherny-loop — the full loop framework this pattern lives inside. loop-maker-checker — practical implementation blueprints. loop-stopping-conditions — how the checker's verdict becomes the stopping condition.
