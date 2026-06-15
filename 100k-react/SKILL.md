---
name: 100k-react
description: Use when making architectural decisions for a React-based project, contributing to React-style open source, understanding why React's API is shaped the way it is, or navigating the React RFC process. Covers the React team's decision-making philosophy, backward-compatibility constraints, the "learn once write anywhere" principle, and how they evaluate proposals. Distinct from react-internals (which covers the Fiber technical model).
---

# React — Open Source Decision Philosophy (220k+ stars)

## The Governing Principle

> "We are not building a framework. We are building an understanding." — React team

React's popularity is inseparable from its restraint. Every time the React team *didn't* add a feature is as important as what they did add. The philosophy isn't "the best API" in isolation — it's the best API **given the constraints of a 220k-star project where millions of applications depend on stability**.

## "Learn Once, Write Anywhere" — not "Write Once, Run Anywhere"

React's original tagline was a deliberate rejection of Write Once, Run Anywhere. WORA (Java applets, PhoneGap) tried to hide the platform. React's bet: **teach developers a component model; let them write platform-specific code using that model.**

React DOM, React Native, React Three Fiber, React for email — each renders differently. The skill you learn is the same. The abstraction doesn't pretend platforms are identical; it gives you a consistent way to think across them.

**Why this matters for architectural decisions:** if you're choosing between a React-native pattern and a web-only shortcut, the React team would choose the native pattern that ports — even at higher upfront cost.

## The RFC Process — How React APIs Get Born

Large React changes go through an RFC (Request for Comments). The process exists because a bad API at React's scale cannot be un-shipped:

1. **Motivation**: what problem is unsolvable without this? If the current API can solve it with good patterns, the RFC dies here.
2. **Design constraints**: what must be true? (backward compatible, works with concurrent mode, SSR-safe, no added bundle cost for non-users)
3. **Prior art**: how did other systems solve this? (Vue, Svelte, Solid, Angular) — with explicit reasoning for why React's constraints require a different answer.
4. **Unresolved questions**: what is the team still unsure about? Shipping with known unknowns requires explicit acknowledgment.

The hooks RFC (Dan Abramov, 2018) is the canonical example: the proposal acknowledged the counter-intuitive "rules of hooks" upfront and explained why the constraint was necessary (deterministic hook ordering for the fiber model), rather than apologizing for it.

## The Stability Contract

React takes backward compatibility extremely seriously. The reasoning:

> "The best feature is the one you don't have to rewrite in two years."

Breaking changes at React's scale mean:
- Millions of applications need updates.
- Libraries that depend on React (Radix, Chakra, React Query) break.
- The community's tutorials, courses, and Stack Overflow answers become wrong.

Corollary for your own libraries and APIs: **if you have users, stability is a feature.** Adding an escape hatch (like `flushSync`, `unstable_` prefixes) buys you time to learn before committing. React's `unstable_` prefix is a promise: "we know this is right directionally, but the API may change — use at your own risk."

## How the React Team Evaluates a New Primitive

The checklist (derived from public RFCs and Dan Abramov's writing):

1. **Can existing patterns solve this without a new primitive?** (99% of proposals fail here)
2. **Does it compose with the existing primitives?** (hooks compose; class lifecycle methods don't)
3. **Does it make the pit of success easier to fall into?** A good API makes the right thing the easy thing.
4. **Is it expressible in JavaScript?** (no compiler requirement — this ruled out many reactive proposals pre-React Compiler)
5. **Does it work with Concurrent Mode?** (render can be interrupted and re-run; synchronous assumptions break this)
6. **What's the failure mode?** Explicit errors > silent wrong behavior.

## The "Algebraic Effects" Long Game

React has been moving toward a model where side effects (data fetching, routing, subscriptions) are expressed as values that the runtime handles — similar to algebraic effects in programming language theory. Suspense is the first visible piece. Server Components are another.

This explains decisions that look strange in isolation:
- Why `useEffect` is awkward: it's a stopgap. The right model involves effects as first-class values the scheduler controls.
- Why Suspense for data fetching isn't fully "done": the team is waiting for the effect model to crystallize before committing the API.

**Practical implication:** when the React team says "this is not the final API," trust them. Build on the stable parts; treat experimental APIs as directional signals.

## What to Take Into Your Own Projects

| React's decision principle | Apply when |
|---|---|
| Stability over features | You have external users; breaking changes have a real cost |
| Motivation-first design | Evaluating any new abstraction/library/pattern |
| Explicit constraints in proposals | Your team disagrees on an API — write down the constraints first |
| Pit of success design | Default behavior should be the correct behavior |
| Unstable_ prefix | You need to ship before the API is final |
| Learn once, not write once | Cross-platform work — share the model, not the code |
