---
name: cherny-loop
description: Use when designing an agentic system, moving beyond single prompts to autonomous agent loops, deciding whether to build a loop or write a prompt, or when work should run unattended on a schedule. Applies Boris Cherny's loop engineering framework — the paradigm shift from prompt-operator to loop-architect.
---

# Cherny — Loop Engineering

## The Core Statement

> "I don't prompt Claude anymore. I have loops that are running. They're the ones that are prompting Claude and figuring out what to do." — Boris Cherny, creator of Claude Code, June 2026

Cherny's insight: prompt engineering optimizes a single instruction you type by hand. Loop engineering optimizes the autonomous system that decides what to prompt, when, and whether the result is acceptable. The developer stops being inside the loop and becomes the author of it.

## The Three Stages of Developer Evolution

**Stage 1 — Directed tool:** You write code; the model is autocomplete. You are always in the loop, initiating everything.

**Stage 2 — Parallel sessions:** You run multiple agent sessions manually, juggling context windows. You're still the initiating force — you start every task.

**Stage 3 — Loop architect:** You write programs that prompt agents on your behalf. The loop reads your GitHub, your CI, your issue tracker — and decides what to work on. You review results, not initiate tasks.

> "My job is to write the loop."

The practical evidence: in the last 30 days of December 2025, 100% of Cherny's contributions to Claude Code were written by Claude Code itself. 259 pull requests. He was the loop author, not the coder.

## What a Loop Actually Is

A loop is a small program with five responsibilities:

```
Trigger  →  Goal check  →  Actions  →  Verification  →  Memory update
   ↑                                                            |
   └────────────────────────────────────────────────────────────┘
                    (if not done, re-enter)
```

1. **Trigger** — what starts a run: a cron schedule, a PR opening, a test failing, another loop completing
2. **Goal** — a verifiable end state, not vague intent: "all tests pass", "zero unassigned P1s", "bundle under 200KB"
3. **Actions** — tools the agent uses: read/write files, run bash, call APIs, spawn sub-agents
4. **Verification** — how completion is confirmed: test exit codes, a checker agent, CI results — not self-attestation
5. **Memory** — persistent state on disk: what was done, what remains, what was decided — the model forgets everything between runs, so the state must live on disk

## The Six Structural Components

| Component | Purpose |
|---|---|
| **Automations** | Scheduled/event triggers that surface work without manual prompts |
| **Worktrees** | Isolated git checkouts so parallel agents don't clobber each other |
| **Skills** | Durable project knowledge so agents don't re-derive context from scratch |
| **Connectors (MCP)** | Integrations to GitHub, Slack, CI, databases — action beyond the filesystem |
| **Sub-agents** | Specialized roles, especially the maker-checker split |
| **Memory** | Persistent markdown/state files that survive context window resets |

## Loop vs Prompt — When to Use Which

```
Use a prompt when:
- Task is one-shot, result is immediate
- Human judgment needed in the loop
- Exploratory / unknown shape

Use a loop when:
- Task recurs (daily triage, weekly coverage check)
- Success condition is objectively verifiable
- Parallelism matters (many independent files/tasks)
- You want compounding leverage — build once, runs forever
```

## Cherny's Practical Rules

**"Your actual engineering work is making sure they halt."**  
Loops that don't terminate are the main failure mode. Every loop needs a hard budget (max turns, max tokens, max cost) that triggers shutdown even if the goal isn't met.

**Progressive trust escalation:**  
`findings only → draft PR → gated PR (human approves) → auto-merge (low-risk only)`  
Don't skip stages. Trust is earned by observable track record, not by the model's confidence.

**Start slow, watch the cost:**  
Run a new loop on a long schedule with a tight goal for a few days before scaling. A loop that runs hourly on a bad goal burns budget and produces noise.

**REQUIRED COMPANION:** loop-design — the practical blueprint for building a loop from scratch. cherny-maker-checker — the single most important structural pattern inside any loop. loop-stopping-conditions — how to write stopping conditions that actually terminate.
