# Skills

A world-class, vendor-neutral library of agent skills. Each skill is a directory with a `SKILL.md` — plain YAML frontmatter (`name` + `description`) per the [agentskills.io spec](https://agentskills.io/specification), no framework lock-in, usable with any AI agent.

## Agent-Agnostic by Design

Skills are **not tied to Claude Code or any specific agent.** The installer shim links this repo into whatever agent toolchains you use — one canonical source, many consumers.

### Install with `bin/skills`

```bash
./bin/skills list                              # list every skill
./bin/skills install                           # symlink → Claude Code + Codex (default)
./bin/skills install --claude                  # just ~/.claude/skills
./bin/skills install --codex                   # just ~/.agents/skills
./bin/skills install --target ~/.cursor/skills # any agent / Cursor / custom dir
./bin/skills install --all                     # every known target
./bin/skills install --all --copy              # copy instead of symlink (sandboxes)
./bin/skills install --dry-run                 # preview, change nothing
./bin/skills uninstall --codex                 # remove from a specific target
./bin/skills manifest                          # (re)generate skills.json
```

- Default targets: `~/.claude/skills` (Claude Code) and `~/.agents/skills` (Codex).
- Override with `CLAUDE_SKILLS_DIR` / `CODEX_SKILLS_DIR` env vars.
- Symlinks mean edits here are live everywhere; re-run `install` only after adding new skill dirs.

[`skills.json`](skills.json) — a generated, agent-neutral index (name · path · description). Regenerate with `./bin/skills manifest`.

---

## Splunk — Log Analysis, Detections & Dashboards

A deep, interlocking suite. Foundation skills (authoring, optimization, CIM) are referenced by the rest.

| Skill | Use when… |
|---|---|
| [splunk-spl-authoring](splunk-spl-authoring/SKILL.md) | Writing/fixing SPL — pipeline, commands, field extraction, idioms. **Foundation.** |
| [splunk-spl-optimization](splunk-spl-optimization/SKILL.md) | Search is slow, times out, or costs too much. |
| [splunk-cim-normalization](splunk-cim-normalization/SKILL.md) | Normalizing fields to CIM so dashboards/detections work across sources. |
| [splunk-log-investigation](splunk-log-investigation/SKILL.md) | Investigating an incident/outage from symptom → root cause. |
| [splunk-detection-engineering](splunk-detection-engineering/SKILL.md) | Building correlation searches, RBA, MITRE ATT&CK mapping. |
| [splunk-threat-hunting](splunk-threat-hunting/SKILL.md) | Proactive, hypothesis-driven hunting for threats alerts missed. |
| [splunk-dashboards](splunk-dashboards/SKILL.md) | Dashboards, KPIs, fast panels, drilldowns, visualization choices. |

---

## Clean Architecture — Layered Code Design

Technology-agnostic. All six skills share one "place an order" example so they interlock.

| Skill | Use when… |
|---|---|
| [clean-architecture](clean-architecture/SKILL.md) | Deciding which layer code belongs in; enforcing the dependency rule. **Overview.** |
| [api-controller](api-controller/SKILL.md) | Writing the API/controller layer; killing fat controllers. |
| [service-layer](service-layer/SKILL.md) | Application/use-case orchestration, transactions; avoiding God services. |
| [domain-model](domain-model/SKILL.md) | Entities, value objects, aggregates, invariants; avoiding anemic domain. |
| [dto-mapping](dto-mapping/SKILL.md) | DTOs, commands, boundary mapping; stopping entity leaks & mass assignment. |
| [persistence-repository](persistence-repository/SKILL.md) | Repository interfaces/impls, ORM↔domain mapping; keeping DB out of domain. |

---

## React + Next.js

| Skill | Use when… |
|---|---|
| [react-internals](react-internals/SKILL.md) | Understanding *why* React works — Fiber, hooks model, Concurrent Mode, RSC. |
| [react-patterns](react-patterns/SKILL.md) | Component design, custom hooks, compound components, state colocation. |
| [react-performance](react-performance/SKILL.md) | Excessive re-renders, slow lists, large bundle — profiling first, then fixes. |
| [nextjs-app-router](nextjs-app-router/SKILL.md) | Server Components, data fetching, caching model, Server Actions, streaming. |
| [nextjs-patterns](nextjs-patterns/SKILL.md) | Route structure, auth/middleware, providers, env vars, error boundaries. |

---

## Famous Engineering Philosophies

Folder prefix: person's **last name** (e.g. `beck-`, `fowler-`, `karpathy-`). Philosophy as operating principles — not biography, actionable technique.

| Skill | Teaches |
|---|---|
| [beck-tdd](beck-tdd/SKILL.md) | Red/Green/Refactor, YAGNI, baby steps, four simple design rules, forcing questions. |
| [fowler-refactoring](fowler-refactoring/SKILL.md) | Code smells catalog, refactoring moves, safe mechanics, sequencing. |
| [martin-solid](martin-solid/SKILL.md) | SOLID principles — original formulations, failure modes, over-application traps. |
| [hickey-simplicity](hickey-simplicity/SKILL.md) | Simple vs easy, complect vs compose, information vs place, simplicity questions. |
| [karpathy-neural-nets](karpathy-neural-nets/SKILL.md) | Recipe for training NNs, Software 2.0 philosophy, build-from-scratch for understanding. |
| [pocock-typescript](pocock-typescript/SKILL.md) | Inference over annotation, generics with constraints, discriminated unions, Zod. |
| [tan-product](tan-product/SKILL.md) | Founder mode, talking to users, design+engineering integration, when to build vs buy. |

---

## 100k+ Repo Design Philosophies

Folder prefix: **`100k-`** (e.g. `100k-react`, `100k-linux`). What the teams behind famous repos *decided* and *why* — design principles, not usage guides. Duplicates with language skills are intentional (different angle).

| Skill | Repo | Stars |
|---|---|---|
| [100k-react](100k-react/SKILL.md) | `facebook/react` | 220k — decision-making philosophy, RFC process, stability contract |
| [100k-linux](100k-linux/SKILL.md) | `torvalds/linux` | 170k — engineering values, patch culture, commit conventions |
| [100k-kubernetes](100k-kubernetes/SKILL.md) | `kubernetes/kubernetes` | 110k — declarative APIs, reconciliation loop, operator pattern |
| [100k-typescript](100k-typescript/SKILL.md) | `microsoft/TypeScript` | 100k — structural typing rationale, soundness trade-offs |

---

## Engineering Personas

How senior engineers in specific roles think about problems — mindset, strategy, and the practices that matter most in that role.

| Skill | Persona |
|---|---|
| [qa-engineer](qa-engineer/SKILL.md) | Test strategy, shift-left, test pyramid, testability as design signal, CI gates. |
| [devops-engineer](devops-engineer/SKILL.md) | CI/CD design, IaC, observability, SLOs, incident response, toil reduction. |
| [ml-engineer](ml-engineer/SKILL.md) | Problem framing, data leakage, evaluation design, experiment tracking, model monitoring. |
| [developer-productivity](developer-productivity/SKILL.md) | DORA metrics, small PRs, fast feedback loops, async communication, toil automation. |

---

## Roadmap

Planned suites — same depth-first approach.

### Language Suites (queued)
- **Java + Spring Boot** — DI, JPA, REST, testing, security
- **Rust** — ownership, lifetimes, async, error handling, idiomatic patterns
- **Go** — goroutines, interfaces, errors, idiomatic patterns, performance
- **TypeScript (deep)** — advanced types, generics, utility types, module patterns

### More Famous Philosophies (queued)
- `linus-torvalds-systems` — performance, hardware-aware programming
- `john-carmack-optimization` — game dev, fast math, LOD, profiling discipline
- `dhh-convention` — convention over configuration, omakase, Shape Up methodology
- `joel-spolsky-strategy` — developer environment, hiring, software estimation

### More 100k+ Repos (queued)
- `vue-reactivity-design` — Vue 3's reactivity system vs React's model
- `postgresql-internals` — MVCC, query planner, extension design
- `tensorflow-mlir` — compiler-based ML, graph optimization philosophy

### SDLC
- `requirements-to-spec`, `code-review-discipline`, `release-management`, `incident-postmortem`, `observability-design`

### InfoSec
- `threat-modeling`, `secure-coding`, `secrets-management`, `incident-response`, `cloud-security`

---

## Authoring Conventions

- **Description = when, not what.** Triggers and symptoms only; never summarize the workflow.
- **Keyword-rich** — error messages, symptoms, tool/command names aid discovery.
- **Active, verb-first names** — `splunk-log-investigation`, not `splunk-investigation-helper`.
- **One excellent example** over many mediocre ones; real, runnable, commented for *why*.
- **Heavy reference → separate file** (e.g., command references, templates).
- **Cross-reference** with `REQUIRED COMPANION:` markers — never `@`-link (force-loads and burns context).
