---
name: developer-productivity
description: Use when improving developer workflow, reducing friction in a codebase, designing local development environments, making CI faster, eliminating toil, evaluating developer experience, or thinking about how team practices (async communication, PR size, feedback loops) affect throughput. Covers individual and team-level productivity.
---

# Developer Productivity

## Core Philosophy

Productivity is not about working more hours. It is about **shortening feedback loops and removing friction from the path between an idea and working code in production.** A developer who can iterate locally in seconds, merge in minutes, and deploy in tens of minutes does more useful work in a day than one waiting hours for CI on a 1000-line PR.

> "Optimize for learning speed." — the unifying principle

## The Four Key Metrics (DORA)

DevOps Research and Assessment identifies four metrics that predict software delivery performance:

| Metric | Measures | Elite performer |
|---|---|---|
| **Deploy frequency** | How often you ship | Multiple/day |
| **Lead time for changes** | Commit to production | < 1 hour |
| **Change failure rate** | % deploys causing incidents | < 5% |
| **Time to restore** | Incident to recovery | < 1 hour |

These are correlated: teams that deploy frequently have lower change failure rates and faster recovery — the opposite of intuition. Small, frequent changes are safer than large, infrequent ones.

## Small Pull Requests: the Highest-Leverage Habit

A PR with 50 changed lines:
- Gets reviewed the same day.
- Reviewer can actually understand it.
- If it breaks something, rollback is easy.
- Goes green in minutes.

A PR with 800 changed lines:
- Waits days for a real review.
- Reviewer rubber-stamps.
- When it breaks prod, debugging 800 lines is the problem.
- Goes green in an hour.

**The rule:** a PR should do one thing. Refactoring, feature, and bug fix in one PR is three PRs. If the feature requires a refactoring, open a refactoring PR first; merge it; then the feature PR is clean and small.

**Stack PRs** when each step depends on the last: PR1 (data model) → PR2 (service) → PR3 (API) → PR4 (UI). Each is reviewable independently. Merge in order.

## Local Development: Make the Feedback Loop Instant

The local loop:
```
Make change → see result (< 5 seconds) → iterate
```

If this loop takes 30+ seconds (slow rebuild, manual restart, missing fixtures), developers stop iterating frequently and instead make large changes to amortize the cost. Large changes = risk.

**Investments that pay off fast:**
- Hot module reload / watch mode (Vite, webpack watch, air for Go, cargo-watch for Rust).
- `docker-compose` (or Tilt, devcontainer) for dependencies — one command to start everything.
- Test fixtures and seed data that are fast and deterministic.
- CLI scripts for common operations (`make migrate`, `make seed`, `make test`).
- Local `.env.example` that works out of the box.

**The new engineer test:** can someone clone the repo and have a working local environment in 15 minutes? If not, fix the developer experience.

## CI: Optimize for Speed and Signal

CI that takes 40 minutes is CI that nobody waits for. Optimizations:

1. **Parallelism**: shard tests across jobs; run lint, build, and tests concurrently.
2. **Caching**: dependency install (node_modules, go module cache, Maven .m2) is pure cache. Never reinstall from scratch on every run.
3. **Fail fast**: run the fastest checks first; short-circuit on failure.
4. **Scope to changes**: run only the tests for changed packages (Nx, Turborepo, Bazel).
5. **Test tiers**: unit tests (< 2min) on every commit; integration (< 10min) on main merge; E2E (20min) nightly or on release.

Target: < 10 minutes for the must-pass CI check on a PR. Anything over 10 minutes gets ignored.

## Async-First Communication

Synchronous communication (meetings, Slack pings) interrupts deep work. **Deep work is where productivity happens.** Optimize for async:

- **Decisions in writing**: RFC docs, GitHub discussions, ADRs. The work of thinking through a problem shouldn't evaporate in a Slack thread.
- **PRs as communication**: well-written PR descriptions reduce review back-and-forth. Context, motivation, and "what to look at first."
- **Meetings with agendas and async prep**: a 30-minute meeting where 10 people read the same doc vs. 10 separate 10-minute reads — math favors async.
- **Maker's schedule**: protect contiguous blocks (2+ hours) for deep work. Context-switching costs ~20 minutes per interruption.

## Automation Inventory: the Things to Eliminate

Toil that kills developer productivity:

| Toil | Automated with |
|---|---|
| Manual version bumps | `semantic-release`, `changesets` |
| Manually creating release notes | Conventional commits + `git-cliff` |
| Manually running DB migrations | Migration runner in deploy pipeline |
| Secret rotation by hand | Secret manager + rotation policy |
| Finding PR reviewer | CODEOWNERS |
| Onboarding setup steps | `mise` / `asdf` + a Makefile |
| Checking formatting on PR | Pre-commit hook / CI linter |

**The test:** is this work tracked in your todo list or is it happening automatically? If it's in your todo list, automate it.

## Code Review Ergonomics

Reviews are a latency bottleneck. Reduce the friction on both sides:

**For authors:**
- PR description answers: what changed, why, and what to look at first.
- One logical change per PR. No mixed concerns.
- Self-review before requesting: does the diff tell a clear story?
- Draft PR → review when ready, not speculative review.

**For reviewers:**
- Review within 4 working hours. Sitting reviews kill velocity.
- Comment on code, not the person. "This function is unclear" not "you wrote this wrong."
- Approve with minor nits rather than blocking; trust the author on taste.
- Distinguish: blocking (must fix before merge), suggestion (nice to have), question (need info).

## Personal Productivity Toolkit

| Habit | Effect |
|---|---|
| Morning deep work block (no Slack/meetings) | Most valuable engineering work |
| Time-boxing investigations (30-60min max) | Prevents rabbit holes |
| "Done" means merged + deployed + monitored | Not "open PR" or "local works" |
| Weekly retro on your own habits | Finds recurring friction to eliminate |
| Keyboard shortcuts for your whole toolchain | Compounds; saves hours/year |
