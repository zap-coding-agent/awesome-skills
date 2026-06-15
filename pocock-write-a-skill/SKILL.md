---
name: pocock-write-a-skill
description: Use when designing and writing a new agent skill — applies Matt Pocock's discipline around description-as-trigger, progressive disclosure, and bundling reference material. Triggers on "write a skill", "create a skill", or when reviewing whether an existing skill's description or structure is correct.
---

# Pocock — Write a Skill

> "The description is not a summary. It's an address. It tells the agent where to go, not what it will find when it gets there." — Matt Pocock

## The Core Insight: CSO (Claude Search Optimization)

Skills are loaded by agents based on the `description` field. The agent reads descriptions to decide *which* skill to invoke — it doesn't read the body until after it decides. This creates a critical constraint:

**If your description summarizes the workflow, the agent follows the description instead of reading the skill body.**

The description is a trigger condition, not a summary:

```yaml
# ❌ Summarizes workflow — agent follows this instead of reading SKILL.md
description: Walks you through writing optimized Splunk searches using CIM normalization and tstats.

# ✅ Trigger condition — agent loads the skill, then reads the body
description: Use when writing Splunk SPL, choosing between stats and tstats, applying CIM field names, or when a search is slow or returns unexpected results.
```

## The "Use when..." Pattern

Every description starts with "Use when..." followed by:
- Triggering symptoms ("search is slow", "getting a 403")
- Tools or commands ("writing SPL", "configuring nginx")
- Error messages ("CORS error", "Cannot read property of undefined")
- Triggering user intent ("designing a repository", "adding a new endpoint")

The test: would this description cause the agent to load the skill in exactly the situations where the skill is useful? No more, no less.

## Structure: Progressive Disclosure

Skills should be scannable. Put the most important guidance first — if the agent only reads the first 20 lines, those 20 lines should handle 80% of cases.

```
SKILL.md
├── Description (frontmatter) — the trigger
├── TL;DR or Quick Reference — for the common case
├── Main content — the full guidance
└── Edge cases and advanced scenarios

reference.md        ← heavy lookup content (command tables, syntax refs)
template.md         ← templates the skill fills in
scripts/            ← deterministic operations (don't make the LLM do math)
```

## When to Split Content

Keep `SKILL.md` under ~300 lines. When you have:
- Command reference tables → `reference.md`
- Output templates → `template.md`
- Scripts for counting/calculating → `scripts/`
- Step-by-step procedures → inline in `SKILL.md` with `REQUIRED COMPANION:` for depth

**REQUIRED COMPANION:** write-a-skill — the short-form invocation. pocock-typescript — Pocock's broader principle of letting types (descriptions) do the work.
