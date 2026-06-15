---
name: handoff
description: Use when a long conversation needs to be passed to a fresh agent session — compacts the current context into a handoff document another agent can pick up without losing thread. Triggers on "handoff", "summarise for next session", or "prepare context for new agent".
---

# Handoff

Write a handoff document summarising the current conversation so a fresh agent can continue the work without losing context.

## Process

1. Save the document to the OS temp directory — not the current workspace
2. Do not duplicate content already captured elsewhere (PRDs, ADRs, plans, commits, diffs) — reference them by path or URL
3. Redact sensitive information: API keys, passwords, PII
4. If the user passed arguments, treat them as a description of what the next session will focus on — tailor the document accordingly

## Document Structure

```markdown
# Handoff — [brief description]

## What we were doing
[1-3 sentence summary of the task and current state]

## Decisions made
[Key decisions with brief rationale — don't duplicate ADRs, just link]

## Current state
[What is done, what is in progress, what is blocked]

## Next steps
[Ordered list of what the fresh agent should do first]

## Suggested skills
[Skills the next agent should invoke — by name]

## Relevant files
[Paths or URLs to the most important artifacts]

## Open questions
[Anything unresolved that the next agent needs to address]
```

**REQUIRED COMPANION:** pocock-handoff — Matt Pocock's framing of why clean handoffs matter for multi-session work.
