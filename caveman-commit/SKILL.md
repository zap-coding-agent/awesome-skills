---
name: caveman-commit
description: Use when writing a git commit message — generates terse Conventional Commits format with subject ≤50 chars, body only when the "why" isn't obvious. Triggers on "write a commit", "commit message", "generate commit", or /caveman-commit.
---

# Caveman Commit

Write commit messages terse and exact. Conventional Commits format. No fluff. Why over what.

## Format

**Subject line:**
`<type>(<scope>): <imperative summary>` — `<scope>` optional

Types: `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `build`, `ci`, `style`, `revert`

- Imperative mood: "add", "fix", "remove" — not "added", "adds", "adding"
- ≤50 chars when possible, hard cap 72
- No trailing period

**Body (only if needed):**
- Skip when subject is self-explanatory
- Add body only for: non-obvious *why*, breaking changes, migration notes, linked issues
- Wrap at 72 chars. Bullets `-` not `*`
- References at end: `Closes #42`, `Refs #17`

**Never include:**
- "This commit does X", "I", "we", "now", "currently" — the diff says what
- Restating what changed — that's in the diff

## Examples

```
# Simple fix
fix(auth): handle expired token on refresh

# Feature with why
feat(orders): add idempotency key to place-order

POST retries without idempotency caused duplicate orders in prod.
Closes #847

# Breaking change
refactor(api)!: remove v1 endpoints

BREAKING CHANGE: /api/v1/* removed. Use /api/v2/*.
Migration: replace base URL in client config.
```

**REQUIRED COMPANION:** caveman — the base compressed-response mode.
