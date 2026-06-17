---
name: loop-design
description: Use when building an agent loop from scratch — designing the trigger, goal, actions, verification, and memory of an autonomous agent system. Applies when a recurring task should run unattended, when a single agent prompt isn't enough, or when you need compounding leverage from a system you build once.
---

# Loop Design

A loop is a program that prompts an agent on your behalf, checks whether the result is acceptable, and re-prompts if not. You build it once; it runs on its own schedule.

## The Five Elements Every Loop Needs

### 1. Trigger — what starts a run

```yaml
# Time-based
schedule: "0 8 * * 1-5"   # weekdays at 8am

# Event-based
on: pull_request.opened
on: test_suite.failed
on: loop.completed         # chained loops

# Manual
on: /bg run triage-loop    # Claude Code background command
```

**Rule:** if the trigger fires too often on a bad goal, it burns budget silently. Start with a long interval (daily or weekly) and tighten only after the loop is producing work you actually use.

### 2. Goal — a verifiable end state

The goal is not a description of what to do. It is the condition that proves it's done.

```
❌ Vague: "improve test coverage"
✅ Verifiable: "coverage for src/billing/ is at or above 90%, CI is green"

❌ Vague: "triage open issues"
✅ Verifiable: "zero P1 issues are unassigned and without an action plan comment"

❌ Vague: "fix the flaky tests"
✅ Verifiable: "the test suite passes 5 consecutive times with no skips"
```

Write the goal as a sentence the checker agent can evaluate against the output without needing any context about what the maker tried.

### 3. Actions — what the agent can do

List the tools explicitly. Scope them as narrowly as the goal allows:

```yaml
tools:
  - read_file           # always safe to include
  - write_file          # needed for code changes
  - bash                # needed for test execution; high-risk in broad scope
  - github_mcp          # needed for issue/PR operations
  - spawn_subagent      # needed for maker-checker pattern
```

**Principle of minimal capability:** don't give a triage loop write access to production systems. Don't give a test-writing loop access to the deployment pipeline.

### 4. Verification — how completion is confirmed

Never let the maker self-certify. Verification options in escalating trustworthiness:

| Method | How | Trust level |
|---|---|---|
| Test suite | Exit code 0 = done | High for code tasks |
| CI pipeline | All checks green | High |
| Checker agent | Fresh agent evaluates against goal | High for subjective tasks |
| Lint/type check | Zero errors | High for specific conditions |
| Human review | Loop surfaces findings; human approves | Highest; use for risky/irreversible |

For anything that merges to main or touches production: human review gate until the loop has a track record.

### 5. Memory — state that survives between runs

The model forgets everything when the session ends. State must live on disk.

```markdown
# triage-memory.md (read at start of each run, written at end)
Last run: 2026-06-17
Issues triaged this week: #847, #851, #863
Issues skipped (needs product input): #849
Open P1s assigned: #852 → @alice, #855 → @bob
```

**Minimum memory contents:**
- What was done in the last run (avoid duplicate work)
- What was explicitly deferred and why
- Current state toward the goal (so the loop can resume, not restart)

## Stopping Conditions — the Critical Engineering Work

Every loop needs a hard budget that triggers shutdown regardless of goal status:

```
Primary stop:  goal condition verified ✓
Fallback stop: max 20 turns reached  OR  cost > $2.00  OR  no progress in 5 turns
```

A loop without a fallback stop runs forever on a bad goal. See loop-stopping-conditions for full treatment.

## Structural Skeleton

```python
# Pseudocode — works in any orchestrator (Python, bash, Claude Code /bg)

def run_loop(goal, max_turns=20, budget_usd=2.0):
    memory = read_memory()
    turns = 0
    
    while turns < max_turns:
        context = build_context(goal, memory)
        result = call_agent(maker_prompt, context, tools)
        
        verdict = call_agent(checker_prompt, result, goal)   # fresh agent
        
        if verdict == "PASS":
            memory.record(result, status="done")
            write_memory(memory)
            return "completed"
        
        if no_progress(result, memory) or cost_exceeded(budget_usd):
            memory.record(result, status="blocked")
            write_memory(memory)
            open_draft_pr_with_findings(result)
            return "escalated"
        
        memory.record(result, status="in-progress")
        turns += 1
    
    escalate_to_human(memory)
    return "max-turns-reached"
```

## Real Example: Morning Triage Loop

```
Trigger:    Weekday 8am cron
Goal:       All P1 GitHub issues have an assigned owner and an action plan comment
Actions:    github_mcp (read issues, write comments, assign labels)
Verify:     Checker agent confirms zero unassigned P1s exist
Memory:     triage-log.md — issues handled per day, issues deferred with reason
Budget:     15 turns max, $1.50 max
```

**REQUIRED COMPANION:** cherny-loop — Boris Cherny's philosophy behind why loops replace prompts. loop-maker-checker — the checker agent pattern in detail. loop-stopping-conditions — writing stopping conditions that terminate reliably.
