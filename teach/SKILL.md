---
name: teach
description: Use when the user wants to learn a new skill or concept over multiple sessions — sets up a stateful teaching workspace with missions, lessons, and reference documents. Triggers on "teach me", "I want to learn", or "explain X from scratch".
---

# Teach

Teach the user a new skill or concept. This is stateful — treat the current directory as a teaching workspace across sessions.

## Teaching Workspace Files

- `MISSION.md` — why the user wants to learn this. Ground every lesson here.
- `./lessons/*.html` — individual lessons, each self-contained, saved as `0001-<name>.html`
- `./reference/*.html` — cheat sheets, glossaries, syntax references — designed for quick lookup
- `./learning-records/*.md` — key insights and non-obvious lessons, titled `0001-<name>.md`
- `RESOURCES.md` — high-quality external resources to cite and assign
- `NOTES.md` — user preferences and working notes

## First Session

If `MISSION.md` is not populated, ask why the user wants to learn this before anything else. A lesson divorced from the user's real-world goal will feel abstract and won't stick.

## Zone of Proximal Development

Each lesson should challenge the user *just enough* — not so easy it's boring, not so hard it overwhelms working memory. Calibrate by reading `./learning-records/` to understand what they already know.

## Lesson Format

Each lesson is one self-contained HTML file:
- Short — completable in one sitting
- One tangible win that builds on previous lessons
- Knowledge first, then skill practice via interactive feedback loop
- A primary source recommendation (best external resource on this specific thing)
- Citations for every non-obvious claim
- Links to related lessons and reference docs
- A reminder to ask the agent follow-up questions

Lessons are **beautiful** — clean typography, readable layout. The user will return to them. Think Tufte.

Open the lesson file for the user after creating it.

## Knowledge vs Skills vs Wisdom

- **Knowledge** — what to know. Teach it first; difficulty is the enemy here
- **Skills** — making knowledge stick through retrieval practice, spacing, interleaving. Difficulty is the tool here
- **Wisdom** — comes from real-world practice. Point the user toward communities and real projects

**REQUIRED COMPANION:** pocock-teach — Matt Pocock's full philosophy behind this teaching system.
