---
name: caveman-review
description: Use when reviewing a PR or diff — writes one-line-per-finding code review comments in format "location: problem. fix." Triggers on "review this PR", "code review", "review the diff", or /caveman-review.
---

# Caveman Review

Write code review comments terse and actionable. One line per finding. Location, problem, fix. No throat-clearing.

## Format

`L<line>: <problem>. <fix>.`

Multi-file: `<file>:L<line>: <problem>. <fix>.`

**Severity prefix (when mixed):**
- `🔴 bug:` — broken behavior, will cause incident
- `🟡 risk:` — works but fragile (race condition, missing null check, swallowed error)
- `🔵 nit:` — style, naming, micro-optimisation — author can ignore
- `❓ q:` — genuine question, not a suggestion

## Drop

- "I noticed that...", "It seems like...", "You might want to consider..."
- Restating what the line does — the reviewer can read the diff
- Hedging ("perhaps", "maybe") — if unsure use `❓ q:`
- "Great work!" per comment — say it once at the top if warranted
- Trailing "let me know if you have questions"

## Example Output

```
L42: 🔴 bug: `user.role` read before null check. Move null check above L38.
L67: 🟡 risk: `parseInt` without radix. Use `parseInt(s, 10)`.
L89: 🔵 nit: `getData` → `fetchOrders` — name the domain concept.
L103: ❓ q: Why `setTimeout(fn, 0)` here — deferring for a reason?
```

**REQUIRED COMPANION:** caveman — base compressed-response mode. caveman-commit — same philosophy for commit messages.
