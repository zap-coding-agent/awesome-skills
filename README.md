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

Folder prefix: person's **last name** (e.g. `beck-`, `fowler-`, `karpathy-`). Philosophy as operating principles — not biography, actionable technique. Skills that are also productivity tools appear in both this section and the Productivity section.

### Kent Beck (2 skills)
| Skill | Teaches |
|---|---|
| [beck-tdd](beck-tdd/SKILL.md) | Red/Green/Refactor, YAGNI, baby steps, four simple design rules, forcing questions. |
| [beck-xp](beck-xp/SKILL.md) | XP practices: pair programming, collective ownership, sustainable pace, CI, planning game. |

### Martin Fowler (3 skills)
| Skill | Teaches |
|---|---|
| [fowler-refactoring](fowler-refactoring/SKILL.md) | Code smells catalog, refactoring moves, safe mechanics, sequencing. |
| [fowler-microservices](fowler-microservices/SKILL.md) | MonolithFirst, service boundary design, strangler fig, inter-service communication. |
| [fowler-enterprise-patterns](fowler-enterprise-patterns/SKILL.md) | PoEAA: Active Record vs Data Mapper, Repository, Service Layer, Transaction Script. |

### Robert C. Martin — Uncle Bob (2 skills)
| Skill | Teaches |
|---|---|
| [martin-solid](martin-solid/SKILL.md) | SOLID principles — original formulations, failure modes, over-application traps. |
| [martin-clean-arch](martin-clean-arch/SKILL.md) | Concentric rings, Dependency Rule, use-case layer, policy vs mechanism, Humble Object. |

### Rich Hickey (2 skills)
| Skill | Teaches |
|---|---|
| [hickey-simplicity](hickey-simplicity/SKILL.md) | Simple vs easy, complect vs compose, information vs place, simplicity questions. |
| [hickey-data-oriented](hickey-data-oriented/SKILL.md) | Data vs objects, open systems, immutable values, generic operations, spec at boundaries. |

### Andrej Karpathy (4 skills)
| Skill | Teaches |
|---|---|
| [karpathy-neural-nets](karpathy-neural-nets/SKILL.md) | Recipe for training NNs, build-from-scratch for understanding, debugging discipline. |
| [karpathy-llm-intuitions](karpathy-llm-intuitions/SKILL.md) | How LLMs work intuitively: token prediction, context window, temperature, hallucination. |
| [karpathy-software-2](karpathy-software-2/SKILL.md) | Software 2.0 thesis: neural nets as software, what changes about engineering. |
| [karpathy-vibe-coding](karpathy-vibe-coding/SKILL.md) | Vibe coding philosophy: intent over syntax, when to trust generated code, limitations. |

### Matt Pocock (7 skills — also in Productivity)
| Skill | Teaches |
|---|---|
| [pocock-typescript](pocock-typescript/SKILL.md) | Inference over annotation, generics with constraints, discriminated unions, Zod. |
| [pocock-typescript-generics](pocock-typescript-generics/SKILL.md) | Generic constraints, conditional types, infer, mapped types, template literals. |
| [pocock-tsconfig](pocock-tsconfig/SKILL.md) | tsconfig options that matter: strict flags, module resolution, paths, composite. |
| [pocock-grill-me](pocock-grill-me/SKILL.md) | Stress-test a plan before coding — resolve every branch of the decision tree first. |
| [pocock-teach](pocock-teach/SKILL.md) | Mission-first stateful teaching: lessons, reference docs, storage strength over fluency. |
| [pocock-handoff](pocock-handoff/SKILL.md) | Compact a conversation into a clean handoff doc for the next agent session. |
| [pocock-write-a-skill](pocock-write-a-skill/SKILL.md) | CSO: description-as-trigger, progressive disclosure, when to split reference files. |

### Boris Cherny (2 skills — also in Loop Engineering)
| Skill | Teaches |
|---|---|
| [cherny-loop](cherny-loop/SKILL.md) | Loop engineering: the three stages, 6 components, why loops replace prompts, progressive trust escalation. |
| [cherny-maker-checker](cherny-maker-checker/SKILL.md) | Splitting the agent that writes from the agent that checks — the most important structural move in any loop. |

### Garry Tan (3 skills)
| Skill | Teaches |
|---|---|
| [tan-product](tan-product/SKILL.md) | Founder mode, talking to users, design+engineering integration, when to build vs buy. |
| [tan-design](tan-design/SKILL.md) | Design as product thinking — clarity, hierarchy, when design is a competitive moat. |
| [tan-startup](tan-startup/SKILL.md) | Early-stage decisions: hiring bar, fundraising timing, focus vs optionality. |

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

## Productivity & Learning Modes

Interactive modes and session tools. Skills from famous engineers appear in **both** this section and their personality suite — same skill, different discovery path.

### Planning & Design
| Skill | Use when… |
|---|---|
| [grill-me](grill-me/SKILL.md) | Stress-testing a plan before writing code — one question at a time. |
| [handoff](handoff/SKILL.md) | Compacting a long conversation into a handoff doc for a fresh agent session. |
| [write-a-skill](write-a-skill/SKILL.md) | Creating a new agent skill with proper description format and structure. |

### Learning
| Skill | Use when… |
|---|---|
| [teach](teach/SKILL.md) | Learning a new concept over multiple sessions with missions, lessons, and reference docs. |

### Caveman Suite — Token Compression (by [JuliusBrussee](https://github.com/JuliusBrussee/caveman))
| Skill | Use when… |
|---|---|
| [caveman](caveman/SKILL.md) | Responses are too verbose — cuts ~65% output tokens, full technical accuracy. |
| [caveman-commit](caveman-commit/SKILL.md) | Writing git commit messages — terse Conventional Commits format. |
| [caveman-review](caveman-review/SKILL.md) | Reviewing a PR — one-line-per-finding comments with severity prefix. |
| [caveman-compress](caveman-compress/SKILL.md) | Compressing a CLAUDE.md or memory file to save input tokens. |

---

## Loop Engineering

Agent loops — the paradigm Boris Cherny described as the next wave after prompt engineering. Skills that appear in the personality section (cherny-*) are mirrored here for discoverability.

### Philosophy (also in Personality)
| Skill | Use when… |
|---|---|
| [cherny-loop](cherny-loop/SKILL.md) | Deciding whether to build a loop, understanding the three stages, designing the full loop architecture. |
| [cherny-maker-checker](cherny-maker-checker/SKILL.md) | An agent self-certifying its own work, or needing an independent quality gate inside a loop. |

### Loop Design Patterns
| Skill | Use when… |
|---|---|
| [loop-design](loop-design/SKILL.md) | Building a loop from scratch — trigger, goal, actions, verification, memory, structural skeleton. |
| [loop-maker-checker](loop-maker-checker/SKILL.md) | Implementing the checker agent: prompt design, verdict contract, escalation on repeated failure. |
| [loop-stopping-conditions](loop-stopping-conditions/SKILL.md) | A loop doesn't terminate, burns budget, or needs a reliable halt strategy — four-part goal contract. |

---

## Roadmap

Planned suites — same depth-first approach.

### Language Suites (queued)
- **Java + Spring Boot** — DI, JPA, REST, testing, security
- **Rust** — ownership, lifetimes, async, error handling, idiomatic patterns
- **Go** — goroutines, interfaces, errors, idiomatic patterns, performance
- **TypeScript (deep)** — advanced types, generics, utility types, module patterns

### More Famous Philosophies (queued)
- `carmack-optimization` — game dev, fast math, profiling discipline
- `dhh-convention` — convention over configuration, Shape Up methodology
- `graham-simplicity` — startup thinking, simple solutions, taste in software

### More 100k+ Repos (queued)
- `100k-vue` — Vue 3's reactivity system vs React's model
- `100k-postgresql` — MVCC, query planner, extension design

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
- **Duplicates are intentional** — personality skills and productivity skills can coexist for the same concept (different discovery path, same quality).
