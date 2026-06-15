---
name: karpathy-vibe-coding
description: Use when working with AI coding assistants (Claude, Copilot, Cursor, etc.) as a core part of development — deciding what to delegate to AI vs write yourself, maintaining code understanding while using AI generation, catching AI mistakes, and applying Karpathy's philosophy of "vibe coding" for the right tasks and manual coding for the ones where understanding matters.
---

# Karpathy — Vibe Coding

## The Concept

> "There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists." — Karpathy, Feb 2025

Karpathy coined the term to describe a genuine new mode of software development: you describe what you want, the AI generates it, you accept or steer. You don't read every line. You're operating at the intent level.

The key insight: **this is legitimately useful for a specific class of problems, and actively harmful for another class.** The skill is knowing which is which.

## When Vibe Coding Is Correct

```
Good candidates for vibe coding:
✅ One-off scripts you'll run once and throw away
✅ Prototyping an idea to test feasibility (you'll rewrite it if it works)
✅ Boilerplate generation (CRUD, config files, test fixtures)
✅ Familiar domains where you can verify correctness by inspection
✅ UI scaffolding / layouts you'll visually verify
✅ Converting between formats (JSON→CSV, XML→JSON, SQL→code)
✅ Code in a language/framework you're learning (learning while doing)
```

The common thread: **you can verify the output without deeply understanding every line.** The loop runs: describe → generate → test/verify → ship or discard.

## When Vibe Coding Is Dangerous

```
Bad candidates for vibe coding:
❌ Security-sensitive code (auth, crypto, input validation, SQL)
❌ Core business logic you'll maintain for years
❌ Code at the boundary of external systems (API clients, payment flows)
❌ Performance-critical paths you'll need to optimize
❌ Any code where a subtle bug causes data corruption
❌ Anything you can't verify quickly (complex algorithms with non-obvious invariants)
```

**The reason:** AI-generated code looks correct before you understand it. Code you don't understand is code you can't debug, can't maintain, and can't extend safely. For short-lived scripts that's acceptable. For production systems, it's technical debt that's hidden and dangerous.

## Karpathy's Workflow for Effective AI-Assisted Coding

He describes his actual workflow:

1. **Start with intent, not syntax.** Describe what you want at the level of behavior, not implementation. "A function that rate-limits API calls to 10/second per user" is better than "a function with a TokenBucket struct."

2. **Stay in the driver's seat.** Don't accept multi-hundred-line diffs blindly. Review at the diff level — what changed, why, what could go wrong.

3. **Test immediately.** The verification loop must be fast. Vibe coding without fast tests is driving without a windshield.

4. **When something breaks, read the code.** Bugs are the forcing function to understand. "It broke" + "I don't know the code" = full stop, read it now.

5. **Maintain the mental model.** You may not have written every line, but you must be able to explain every major component. If you can't explain it, you don't own it.

## The Understanding Tax

Karpathy is explicit about what you trade away in vibe coding: **deep understanding of the code.** This is a real cost, not a philosophical one.

```
Systems you built line by line:
  Debugging: you have a mental model → you know where to look
  Optimization: you know the hot paths → you know what to change
  Extension: you understand the invariants → you know what's safe to add

Systems you vibed into existence:
  Debugging: no mental model → read the code now, under pressure
  Optimization: no profile of which AI-generated code is expensive
  Extension: unclear what the AI assumed → easy to break invariants
```

**The heuristic:** vibe coding shifts understanding-cost from build-time to debug-time. For throw-away code, that's a good trade. For maintained code, it's usually a bad trade.

## The AI as Pair Programmer Frame (vs. the AI as Contractor Frame)

Two ways to think about AI coding assistance:

**Contractor frame (wrong for learning):** "AI builds it, I verify it ships." Optimizes for short-term velocity. Erodes understanding over time. Works for tasks where understanding doesn't matter.

**Pair programmer frame (right for growth):** "AI and I work together. AI handles syntax and boilerplate. I handle architecture and invariants. I understand everything we build." Maintains understanding. Suitable for anything you'll maintain.

Karpathy explicitly advocates the pair programmer frame as the default, with the contractor frame reserved for the vibe coding candidates above.

## Prompting AI for Code — Karpathy's Principles

1. **Specify the interface before the implementation.** "Write a function `rate_limit(user_id, limit_per_second)` that returns True if the call should proceed, False if rate-limited. Use Redis for state." Giving the signature constrains the AI to what you actually want.

2. **Provide the context that matters.** Error messages, relevant type definitions, the existing code it needs to fit into. The AI doesn't know your codebase; you do.

3. **Ask for tests first.** "Write tests for this behavior before writing the implementation." Forces the AI to specify what it's building before building it — essentially AI-TDD.

4. **Iterate on the output, don't accept the first pass.** "This is good but the rate limiter resets per-second, not per-window. Modify it to use a sliding window." You're the reviewer; the AI is the author.

5. **When stuck, explain the problem, not the solution.** "I'm trying to prevent duplicate order submissions. Here's the current flow: [describe]. What are the approaches?" Let the AI generate the design space; you choose the approach.
