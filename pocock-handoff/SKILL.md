---
name: pocock-handoff
description: Use when a long conversation needs to be handed off to a fresh agent — applies Matt Pocock's discipline of clean context transfer so the next session starts with full understanding rather than re-deriving everything. Triggers on "handoff", "prepare for next session", or "summarise for a new agent".
---

# Pocock — Handoff

> "A handoff document is the diff between your conversation and the next agent's starting point." — Matt Pocock

## The Problem This Solves

Long agent conversations accumulate context that isn't written down anywhere — implicit decisions, abandoned approaches, current state. When you start a fresh session, you either lose that context (and the next agent re-derives it expensively) or you paste in the whole conversation (and pay for it in tokens). A handoff document is the compression of the conversation into exactly what the next session needs.

Pocock's principle: **the handoff document is a product artifact.** It should be good enough that a developer who wasn't in the conversation could read it and continue the work without asking clarifying questions.

## What Makes a Good Handoff

- **Decisions, not deliberations**: record what was decided, not every option that was considered
- **State over history**: what is true now, not the full story of how you got here
- **Next-step ordered**: the first thing in the document is the first thing the next agent should do
- **Non-duplicative**: link to PRDs, ADRs, plans, commits — don't copy-paste them
- **Skill-aware**: suggest which skills the next agent should invoke to continue effectively

## Process

1. Save the document to the OS temp directory — not the current workspace
2. Do not duplicate content already captured in other artifacts — reference by path or URL
3. Redact sensitive information (API keys, passwords, PII)
4. If the user passed an argument describing the next session's focus — tailor the document to that

## Document Structure

```markdown
# Handoff — [brief description]

## What we were doing
[1-3 sentence summary of the task and where we are]

## Decisions made
[Key choices with brief reasoning — link to ADRs where they exist]

## Current state
[Done / in-progress / blocked]

## Next steps
[Ordered list — the fresh agent starts here]

## Suggested skills
[Skill names the next agent should invoke]

## Relevant files
[Paths or URLs to the most important artifacts]

## Open questions
[Unresolved items the next agent needs to address]
```

**REQUIRED COMPANION:** handoff — the short-form version. pocock-typescript — Pocock's broader design discipline.
