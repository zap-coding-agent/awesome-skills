---
name: write-a-skill
description: Use when the user wants to create a new agent skill — gathers requirements, drafts SKILL.md with proper description format, adds reference files if needed, and reviews with the user. Triggers on "create a skill", "write a skill", "build a new skill", or "add a skill".
---

# Write a Skill

Create a new agent skill with proper structure following the agentskills.io specification.

## Process

**1. Gather requirements**
- What task or domain does the skill cover?
- What specific use cases and triggers should it handle?
- Does it need reference files (command tables, templates, checklists)?
- Any executable scripts for deterministic operations?

**2. Draft the skill**

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Optional: tables, commands, lookup content
└── template.md        # Optional: templates the skill fills out
```

**3. Write the description correctly**

The `description` field is how agents decide whether to load this skill. It must:
- Start with "Use when..." — triggering conditions and symptoms only
- Never summarize the workflow (agents follow the description instead of reading the body)
- Include error messages, tool names, or command names that would appear in context
- Be keyword-rich for discovery

```yaml
# ✅ Correct
description: Use when writing Splunk SPL — authoring searches, fixing pipeline errors, choosing between stats/tstats, applying CIM field names, or the search returns unexpected results.

# ❌ Wrong — summarizes workflow
description: Guides you through writing optimized Splunk searches using best practices and CIM normalization.
```

**4. Review with the user**
- Does this cover your use cases?
- Anything missing or unclear?
- Does the description trigger in the right situations?

## Skill Body Guidelines

- **One excellent example** over many mediocre ones — real, runnable
- **Heavy reference content** → separate file (keeps SKILL.md scannable)
- **Cross-reference** related skills with `REQUIRED COMPANION:` — never `@`-link
- **No workflow summaries in the description** — that's what the body is for

**REQUIRED COMPANION:** pocock-write-a-skill — Matt Pocock's philosophy behind skill authoring.
