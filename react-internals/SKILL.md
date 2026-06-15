---
name: react-internals
description: Use when you need to understand *why* React works the way it does — Fiber reconciliation, the rendering model, Concurrent Mode, hooks internals, or when debugging unexpected re-renders, stale closures, batching behavior, or tearing. Understanding the internals predicts behavior without memorizing rules.
---

# React Architecture & Design Philosophy

## The Core Bet

React's defining decision: **UI = f(state).** The entire library is built to make that equation true and efficient. Every design choice — reconciliation, Fiber, Concurrent Mode, hooks — is a consequence of this commitment.

> "React is a JavaScript library for building user interfaces." — deceptively simple; the radical part is *how*.

Before Fiber (React 15): one synchronous call to re-render the whole tree. This was simple but blocking — a large update would freeze the UI. React 16 rewrote the engine to fix this without changing the f(state) model.

## Fiber: the Reconciler

Fiber is React's **unit of work** — a linked list of work nodes, one per component, that can be interrupted, paused, and resumed. Unlike the old stack-based reconciler (recursive, uninterruptible), Fiber lets React yield to the browser between units.

Key concepts:
- **Work-in-progress tree**: Fiber builds a new tree in the background, diffing against the current tree.
- **Commit phase**: Once the entire tree is computed, React applies changes to the DOM in one synchronous pass. The commit is never interrupted — partial DOM updates would be visible as glitches.
- **Two phases**: Render phase (interruptible, pure, can run many times) + Commit phase (synchronous, side-effect-safe once).

**Why this matters for you:** Any work you do in render-phase hooks (`useMemo`, `useReducer`, render itself) must be **pure and idempotent** — React may call it multiple times in Concurrent Mode. Side effects belong only in `useEffect` (commit-phase) or event handlers.

## Reconciliation & the Diffing Heuristics

React doesn't diff the entire tree — that would be O(n³). It applies two heuristics that bring it to O(n):

1. **Type determines identity.** If a node changes type (e.g., `<div>` → `<span>`), React tears down and rebuilds the entire subtree. Don't swap types to do "transitions."
2. **Key determines identity within lists.** Without a key, React assumes position = identity. A key tells React "this is the same logical element even if it moved." Use stable, unique keys (not array index for dynamic lists).

**Gotcha:** Using the array index as a key on a list that reorders, inserts, or deletes causes React to patch items in place rather than move them — wrong output, missed animations, lost input state.

## Hooks: the Mental Model

Hooks are **not magic** — they're an array of values attached to a Fiber node, indexed in order. This is why the rules are structural:

- **Don't call hooks conditionally or in loops** — the call order must be stable across renders so the index correctly identifies each hook's state slot.
- `useState` → stores `[value, setter]` at slot N.
- `useEffect` → stores `[callback, deps]` at slot N; fires after commit if deps changed.

**Closures and staleness:** A `useEffect` callback captures the state values from the render it was scheduled in. If state has changed by the time the effect fires, those values are stale. The fix is the deps array — listing a value forces React to re-schedule the effect when it changes. For values that change often and you need fresh in a callback, `useRef` stores a mutable box that is the same object across renders.

## Concurrent Features (React 18+)

Concurrent Mode means React can render multiple versions of the UI at the same time:

- **Transitions** (`startTransition`): mark updates as non-urgent. React renders the old UI while preparing the new one, never showing a half-updated state.
- **Suspense**: components declare data-dependency boundaries; React shows a fallback while waiting. Works with async/await at the framework level (Next.js, Remix).
- **Automatic batching**: all state updates in the same event — including in Promises and timeouts — are now batched into one re-render.

**Implication:** Concurrent rendering makes the "render can run multiple times" guarantee real, not theoretical. Side effects in render (mutating refs, calling non-idempotent functions) will cause bugs in Concurrent Mode that didn't appear in sync mode.

## The Unidirectional Data Flow Design Decision

State flows down via props and context. Events flow up via callbacks. This is not arbitrary — it makes the UI predictable: given state, you always know the UI. Two-way binding (Angular v1, Vue v-model for non-trivial cases) breaks this predictability because state can change from anywhere in the tree.

**Context is not a performance tool.** It's a way to skip prop-drilling for infrequently-changing values (theme, locale, auth). Using context for high-frequency state (e.g., every keystroke) causes all consumers to re-render. Use Zustand, Jotai, or Redux for granular subscriptions.

## Why Server Components (RSC) Fit the Model

RSC extends f(state) to the server: components that run *only* on the server, never shipped to the browser, with direct access to data sources. The result: zero client-side bundle cost for data-fetching components. The mental model stays identical — you're still writing components that return UI.

The key invariant: **RSC cannot use state, effects, or browser APIs** — they have no client-side lifecycle because they never run there. When you need interactivity, add `"use client"` to create a client component that RSC renders into.

**REQUIRED COMPANION:** For using React day-to-day, see react-patterns and react-performance. For Next.js's RSC integration, see nextjs-app-router.
