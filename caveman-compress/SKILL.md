---
name: caveman-compress
description: Use when a CLAUDE.md, memory file, or large instruction document is bloated with filler and needs to be compressed to save input tokens — rewrites it in caveman style while preserving all technical substance. Triggers on /caveman-compress FILEPATH or "compress my CLAUDE.md".
---

# Caveman Compress

Compress natural language memory files (CLAUDE.md, todos, preferences, instruction docs) into caveman format to reduce input tokens. Preserves all technical substance, code, URLs, and structure.

## Process

1. Read the target file
2. Create a human-readable backup at `<filename>.original.md`
3. Rewrite the file in caveman style — full intensity level
4. Overwrite the original file with the compressed version
5. Report: original token count, compressed token count, % reduction

## What to Compress

- Pleasantries and filler: "Please make sure to...", "It's important that...", "Always remember to..."
- Padding: "In order to" → "To", "at this point in time" → "now"
- Redundant explanation of obvious things
- Passive voice → active

## What to Preserve Verbatim

- All code blocks
- All URLs and paths
- All technical terms, API names, CLI commands
- All structured data (tables, lists with specific values)
- All constraint statements (things the agent must/must not do)
- Anything where paraphrase could change meaning

## Example

Before:
```
Please make sure to always run the test suite before committing changes to the repository. 
It's really important that you don't skip this step as it helps us maintain code quality 
and ensures that existing functionality continues to work correctly.
```

After:
```
Run tests before commit. No skip.
```

**REQUIRED COMPANION:** caveman — base compressed-response mode for ongoing sessions.
