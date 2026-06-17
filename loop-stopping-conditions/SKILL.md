---
name: loop-stopping-conditions
description: Use when a loop doesn't terminate, runs forever on a bad goal, burns budget without making progress, or needs a reliable halt strategy. Covers primary stopping conditions (goal met), fallback budgets (max turns/cost/no-progress), and escalation when neither fires.
---

# Loop Stopping Conditions

> "Your actual engineering work is making sure they halt." — Boris Cherny

The most common loop failure mode isn't a wrong answer — it's a loop that runs indefinitely on a goal it can never satisfy. Every loop needs two layers of stopping conditions: a primary (goal met) and a hard fallback budget.

## The Four-Part Goal Contract

A stopping condition that actually terminates must specify all four:

```
END STATE    — what does "done" look like?
EVIDENCE     — how do we prove the end state was reached?
CONSTRAINTS  — what must remain true (tests passing, no regressions)?
BUDGET       — max turns / max cost / time limit regardless of the above
```

### Writing the End State

The end state must be evaluable by the checker without asking the maker:

```
❌ "improve test coverage"         — how much? which files? what counts?
✅ "coverage for src/billing/ ≥ 90% per istanbul"

❌ "fix the bug"                   — which bug? how do we know it's fixed?
✅ "the test in test/auth.spec.ts:L42 passes; no other tests regress"

❌ "triage the issues"             — what does a triaged issue look like?
✅ "every open P1 issue has an assignee and a comment starting 'Action plan:'"
```

### Writing the Evidence

Prefer objective evidence. In order of preference:

1. **Exit code** — `npm test` exits 0; `tsc --noEmit` exits 0
2. **CI pipeline** — all checks green on the branch
3. **Script output** — a small script you write that checks the condition and returns 0/1
4. **Checker agent** — a fresh model evaluating against an explicit rubric
5. **Human review** — for irreversible or high-risk operations

### Writing the Constraints

What must stay true even as the loop makes changes:

```
- No existing tests may be deleted or skipped
- Public API signatures must be preserved (no breaking changes)
- No new dependencies may be added without a comment explaining why
- The build must pass before each commit
```

Constraints catch a loop that satisfies the goal condition by cheating (e.g., deletes tests to make coverage appear to pass, disables type checks to clear errors).

## The Fallback Budget (Required)

Even with a perfect goal condition, add a hard budget:

```python
STOPPING_CONDITIONS = {
    "primary": checker_returns_pass,
    "max_turns": 20,           # absolute ceiling
    "max_cost_usd": 2.00,      # financial circuit breaker
    "no_progress_turns": 5,    # give up if checker keeps returning same findings
    "time_limit_minutes": 30,  # wall-clock ceiling for real-time loops
}
```

**No-progress detection** is the most important fallback. If the checker returns the same specific failure 3 turns in a row, the maker is stuck — it needs human input, not more retries.

```python
def no_progress(findings_history, window=3):
    if len(findings_history) < window:
        return False
    recent = findings_history[-window:]
    return all(f == recent[0] for f in recent)   # identical findings = stuck
```

## Escalation, Not Silent Failure

When a fallback budget fires, the loop must not silently exit. It must:

1. Write the current state and all findings to memory
2. Open a draft PR or issue with the findings attached
3. Label it for human attention ("needs-review", "loop-blocked")
4. Record the specific blocker — what the checker kept finding

```markdown
# loop-memory.md — escalation record
Date: 2026-06-17
Loop: billing-coverage
Status: ESCALATED (no-progress after 5 turns)
Blocker: Checker keeps flagging getTotal() edge case — zero line items path not reachable from current test fixtures
Draft PR: #891
Human needed: Yes — fixture design decision required
```

Tomorrow's run reads this memory and skips the task until a human clears the blocker.

## Common Failure Patterns and Fixes

| Pattern | Symptom | Fix |
|---|---|---|
| **Vague goal** | Loop runs until budget, checker always FAIL | Rewrite goal with measurable end state and evidence |
| **Missing constraints** | Loop satisfies goal by breaking something else | Add constraints to the goal contract |
| **No fallback budget** | Loop runs forever on an impossible task | Add max_turns + max_cost |
| **No no-progress detection** | Loop burns budget retrying the same failing approach | Add 3-turn identical-findings circuit breaker |
| **Silent escalation** | Loop exits without a trace | Require memory write + draft PR/issue on any fallback trigger |
| **Self-certification** | Maker says done, actually isn't | Add independent checker — see loop-maker-checker |

**REQUIRED COMPANION:** loop-design — where stopping conditions fit in the full loop architecture. loop-maker-checker — the checker that produces the PASS/FAIL verdict. cherny-loop — Boris Cherny's principle that halting is the real engineering work.
