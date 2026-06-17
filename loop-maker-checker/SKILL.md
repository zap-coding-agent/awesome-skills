---
name: loop-maker-checker
description: Use when an agent loop needs a quality gate, an agent is self-certifying its own output, outputs are unreliable without independent review, or you need to split code generation from code verification inside a loop. Practical implementation of the maker-checker pattern.
---

# Loop Maker-Checker

The maker-checker pattern splits a loop into two agents with opposite orientations: one produces work, one audits it. The key structural constraint: **the checker must not have access to the maker's working context** — only to the output and the goal condition.

## Why Self-Certification Fails

An agent asked "did you complete the task?" with access to its own working history will almost always say yes. It shares the same blind spots that caused any mistakes, it's contextually committed to the approach it took, and "done" is the correct terminal state in its reward model. The problem isn't honesty — it's that the evaluator and the worker share a context that makes independent verification structurally impossible.

## Two-Agent Implementation

```python
MAKER_PROMPT = """
You are working on: {task_description}
Project context: {claude_md_content}
Current state: {memory_contents}

Complete the task. Use the available tools. When done, output a summary of what changed.
"""

CHECKER_PROMPT = """
Goal condition: {goal_condition}

Review the output below. Answer only: PASS or FAIL.
If FAIL, list exactly what's missing or incorrect — file names, line numbers, specific conditions.
Do not suggest how to fix it. Do not consider effort. Does the output satisfy the goal? Yes or no.

Output:
{maker_output}
"""
```

The checker prompt is structurally adversarial. It has no knowledge of what was tried, only what was produced.

## The Checker's Output Contract

The checker should return a structured response:

```
VERDICT: PASS

or

VERDICT: FAIL
REASON:
- src/billing/invoice.ts:L47: getTotal() not covered — no test for the zero-line-items edge case
- CI check "type-check" is failing: Property 'id' does not exist on type 'DraftInvoice'
GOAL_CONDITION: coverage for src/billing/ ≥ 90% and CI green
```

The `REASON` becomes the maker's input on the next iteration — concrete, actionable, not a vague "try again."

## Escalation Pattern

```
Round 1: Maker → Checker(FAIL) → Maker + [checker findings]
Round 2: Maker → Checker(FAIL) → Maker + [round 1 findings] + [round 2 findings]
Round 3: Maker → Checker(FAIL) → ESCALATE
          └─ write full findings to memory
          └─ open draft PR with notes
          └─ label "needs-human" 
          └─ halt loop (do NOT retry indefinitely)
```

Hard max on retries: 3 is almost always enough. If a task can't be completed in 3 maker-checker cycles, it either needs human input or the goal condition is underspecified.

## Choosing the Checker

| Task type | Best checker |
|---|---|
| Code correctness | Test suite exit code — zero human judgment needed |
| Code quality / style | Fresh Claude session with a rubric |
| PR review quality | Human (loop surfaces findings; human approves) |
| Data correctness | Schema validation + statistical sampling script |
| Documentation accuracy | Fresh Claude session with the source of truth |

Use the most objective checker available. A test suite is better than a model because it doesn't hallucinate. A model is better than nothing when the criterion is inherently subjective.

## Checker ≠ Reviewer

The checker doesn't improve the work — it only verdicts it. Improvement is the maker's job, guided by the checker's specific findings. A checker that starts suggesting solutions is drifting into maker territory and loses its independence.

**REQUIRED COMPANION:** cherny-maker-checker — Boris Cherny's philosophy behind this pattern. loop-design — where the maker-checker fits in the full loop architecture. loop-stopping-conditions — the checker's PASS/FAIL becomes the primary stopping condition.
