---
name: pocock-teach
description: Use when the user wants to learn something deeply over multiple sessions — applies Matt Pocock's mission-first, stateful teaching system with lessons, reference docs, and learning records. Triggers on "teach me", "I want to learn X properly", or when learning needs to persist across sessions.
---

# Pocock — Teach

> "Fluency without storage strength is an illusion of mastery." — Matt Pocock

## The Philosophy

Pocock's teaching system is built around one insight: **most learning tools optimise for fluency** (can you answer this now?) **not storage strength** (will you remember this in a month?). Fluency fades. Storage strength compounds.

The three ingredients for deep learning:
- **Knowledge** — facts, concepts, syntax. Difficulty is the enemy here. Make acquisition frictionless.
- **Skills** — making knowledge durable through effortful retrieval. Difficulty is the tool here.
- **Wisdom** — can't be taught; comes from real-world interaction with communities and practitioners.

## The Mission-First Rule

Every teaching session starts with the mission: *why* does the user want to learn this? A lesson that isn't tied to a real goal feels abstract and doesn't stick. If the mission isn't clear, clarifying it is the first job.

The mission lives in `MISSION.md` and grounds every lesson decision: what to teach, in what order, at what depth.

## The Teaching Workspace

The current directory is a persistent workspace:
- `MISSION.md` — the user's reason for learning this
- `./lessons/*.html` — individual lessons (`0001-<name>.html`), self-contained, beautiful
- `./reference/*.html` — reference sheets: syntax, algorithms, glossaries, cheat sheets
- `./learning-records/*.md` — key non-obvious insights (`0001-<name>.md`), used to track zone of proximal development
- `RESOURCES.md` — high-quality external sources to cite and assign
- `NOTES.md` — user preferences and session notes

## Zone of Proximal Development

The sweet spot: challenging *just enough*. Too easy and the user isn't building storage strength. Too hard and working memory fills before the concept lands.

Calibrate via `./learning-records/` — what has the user already internalised? Build the next lesson from there.

## Lesson Design Principles

- **One tangible win** per lesson — don't teach three things at once
- **Knowledge → practice** — teach the concept, then make the user retrieve it
- **Interactive feedback loop** — quizzes should have same-length answers (no formatting clues). Interactive exercises should give immediate feedback.
- **Citations everywhere** — every non-obvious claim links to a primary source
- **Never trust parametric knowledge** — find and cite a real resource before teaching it
- **Beautiful HTML output** — the user will return to these lessons. Clean typography. Think Tufte.

## Acquiring Wisdom

When a user's question requires lived experience — community norms, war stories, edge cases that only appear in production — the answer is a community referral. Find a high-reputation community (forum, Discord, local group) where they can test skills in the real world.

**REQUIRED COMPANION:** teach — the short-form invoke. pocock-typescript — Pocock's TypeScript teaching philosophy specifically.
