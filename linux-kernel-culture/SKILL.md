---
name: linux-kernel-culture
description: Use when contributing to kernel-style projects, writing patch-based submissions, doing code review in a direct/high-signal culture, understanding Linus Torvalds' engineering philosophy, or designing systems where correctness and simplicity are absolute constraints. Covers the Linux development model, patch culture, and Linus's core engineering principles.
---

# Linux Kernel — Engineering Culture & Philosophy

## The Governing Principle

> "Talk is cheap. Show me the code." — Linus Torvalds

The Linux development model is empirical, not theoretical. Proposals without patches carry little weight. The code is the spec. If you can't show it in code, the idea isn't ready. This creates a culture of extreme concreteness: every design discussion anchors to actual implementation.

## Linus's Core Engineering Values

### 1. Good Taste in Code

Linus famously showed a singly-linked list node removal with and without "good taste" — the elegant version eliminates the special-case for removing the head by using a pointer-to-pointer:

```c
// ❌ lacks good taste — special cases the head
void remove_list_entry(Entry* entry) {
    Entry* prev = NULL;
    Entry* walk = head;
    while (walk != entry) { prev = walk; walk = walk->next; }
    if (!prev) head = entry->next;
    else prev->next = entry->next;
}

// ✅ good taste — indirect pointer eliminates the branch
void remove_list_entry(Entry* entry) {
    Entry** indirect = &head;
    while ((*indirect) != entry) indirect = &(*indirect)->next;
    *indirect = entry->next;
}
```

Good taste = fewer special cases, structures that naturally express the problem, code where the intent and the mechanism are the same thing.

### 2. Simplicity Is Not Optional

The kernel must run on millions of diverse machines with minimal resources. Complexity is not a stylistic problem — it's a correctness and security risk. Every abstraction must earn its existence:

- **No abstractions for abstraction's sake.** If removing a layer makes the code clearer, remove it.
- **No "clever" code.** Clever code that the next maintainer can't understand in 2 minutes is wrong code.
- **No over-engineering.** Design for today's requirements; let tomorrow's requirements drive tomorrow's changes.

### 3. Performance Is a Feature

> "Premature optimization is the root of all evil" is Knuth's, not Torvalds'. Linus's view: performance that matters is a correctness constraint, not an afterthought.

The kernel treats cache line alignment, avoiding unnecessary locks, syscall overhead, and memory layout as first-class design concerns from the start — not optimizations bolted on later.

### 4. Blame the Author, Not the Reviewer

Kernel review is direct to the point of bluntness. If code is wrong, Linus says so. The expectation: **the author is responsible for correctness.** Reviewers who find bugs are doing the author a favor. Reviewers who praise mediocre code are doing the project a disservice.

This is not cruelty — it's high standards applied consistently. The implicit contract: criticism is technical, not personal; it is given to improve the code, not to establish hierarchy.

## The Patch Culture

### A Good Patch Has:
1. **One logical change.** One patch = one reviewable unit. Don't mix whitespace fixes, refactoring, and features.
2. **A commit message that explains *why*, not what.** The diff shows what changed; the message explains why this change is correct, what problem it solves, and what alternatives were considered.
3. **Tests for the change** (when applicable to the subsystem).
4. **Signed-off-by chain.** A legal and social record of who reviewed and approved.

### Commit Message Format (kernel style):
```
subsystem: short summary (under 72 chars, imperative mood)

Body explains the motivation. What was broken? Why is this the right
fix? What are the performance implications? What alternatives were
considered and why were they rejected?

Signed-off-by: Your Name <you@example.com>
```

The imperative mood ("Fix", "Add", "Remove") is non-negotiable in kernel style — and a good convention everywhere.

### Review Ladder
Patches go through:
1. Subsystem maintainer review (technical correctness)
2. Andrew Morton / -mm tree (integration testing)
3. Linus's tree (final gate, merge window)

The **merge window** opens after a release and lasts two weeks. New features only during the merge window; only fixes after it closes. This creates predictable release cadence without a fixed date — release when it's ready.

## Linus's Law

> "Given enough eyeballs, all bugs are shallow." — Eric S. Raymond, attributed to Linus

The open development model — transparent code, distributed review, public mailing lists — exists to maximize the number of people who can spot problems. This is why the kernel development model is **email-patch-based, not proprietary-tooling-based**: the lowest barrier to review and contribution.

## What to Take Into Your Projects

| Kernel principle | Applied to any project |
|---|---|
| One patch = one logical change | One PR = one reviewable idea; no mixing concerns |
| Commit message explains *why* | "Fix typo" → "Fix: prevent null deref when user is unauthenticated" |
| Talk is cheap, show the code | Proposals as prototypes, not docs |
| Reviewer correctness > author feelings | Code review comments are about the code |
| No complexity without necessity | Earn every abstraction |
| Merge window model | Feature freeze before release, fixes only after |
