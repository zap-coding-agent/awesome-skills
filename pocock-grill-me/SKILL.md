---
name: pocock-grill-me
description: Use when the user wants to stress-test a plan or design before writing code — applies Matt Pocock's discipline of resolving every design branch before touching the keyboard. Triggers on "grill me", "stress-test this", or when a plan has unresolved forks that will become bugs.
---

# Pocock — Grill Me

> "The cost of a bad decision discovered in code is 10× the cost of the same decision discovered in a question." — Matt Pocock

## Why This Exists

Pocock's engineering discipline: **never write code against an underspecified design.** Developers habitually start coding before the design is clear, making implicit choices at every fork — choices that accumulate into an architecture nobody intended. Grilling the design first surfaces those forks explicitly, resolves them deliberately, and gives you a clear picture before the first line is written.

The session is not interrogation — it's collaborative design. Every question comes with a recommended answer. The goal is a shared understanding, not catching the user out.

## The Process

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

For each question:
1. Ask one question only — no batching
2. Provide your recommended answer with brief reasoning
3. Wait for my response
4. If my answer opens a new branch, follow it before moving on
5. If a question can be answered by exploring the codebase, explore it instead of asking

## What to Look For

- **Underspecified inputs**: "user submits a form" — which user? what validation? what if they're not logged in?
- **Missing error paths**: what happens when the external API is down? when the DB write fails halfway?
- **Implicit sequencing**: does step A need to complete before step B can start?
- **Ownership ambiguity**: who owns this data? who can mutate it?
- **Terminology mismatch**: are "account" and "user" the same thing or different?
- **Scope creep signals**: "and then we could also..." — flag as a separate decision

## When It's Done

The session is complete when:
- Every major decision branch has a resolved answer
- Error paths are explicit
- Terminology is pinned
- You could hand the notes to a second developer and they'd build the same thing

**REQUIRED COMPANION:** grill-with-docs — extends this session with domain glossary and ADR cross-referencing. pocock-typescript — Pocock's broader TypeScript design philosophy.
