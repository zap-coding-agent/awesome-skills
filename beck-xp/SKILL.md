---
name: beck-xp
description: Use when structuring team engineering practices — pair programming, collective code ownership, sustainable pace, continuous integration (the original meaning), small releases, planning game, or when a team is moving slowly because of process overhead or quality problems. Applies Kent Beck's Extreme Programming practices beyond TDD.
---

# Beck — Extreme Programming (XP)

## Core Philosophy

> "XP is a discipline of software development based on values of simplicity, communication, feedback, and courage." — Kent Beck, Extreme Programming Explained

XP (1999, revised 2004) was radical when published: code reviews continuously (pair programming), integrate many times a day, release small increments often, let the customer change requirements at any time. These ideas, once controversial, are now the default in high-performing teams — they just aren't called XP anymore.

The underlying model: **software development is a social activity.** The practices exist to maximize feedback between team members, and between the team and the user.

## The 12 Core Practices

### 1. The Planning Game
Customers and developers collaborate on the release plan. Customers choose what to build (business value). Developers estimate how long it takes. Neither overrides the other.

**Why it works:** developers don't make business decisions. Customers don't make technical estimates. Each party does what they're actually good at.

The iteration: a 1-2 week cycle. At the start, the customer picks the stories to build from the estimated backlog. At the end, working software is demonstrated. The plan is always the current best guess — not a commitment to a year-long roadmap.

### 2. Small Releases
Ship the smallest possible thing that has business value. Release often. Get feedback. Adjust.

**The forcing function:** small releases force decisions about what matters. If you must release something valuable in 2 weeks, you'll cut features ruthlessly. Ruthless cutting produces clarity about what actually matters.

### 3. Metaphor
The team shares a system metaphor — a story that describes how the whole system works. A common vocabulary for the architecture. When everyone calls the thing that distributes work a "scheduler" (not a "job manager" or "work queue" or "dispatcher"), communication is faster.

Related to Domain-Driven Design's ubiquitous language: the model's vocabulary is the conversation's vocabulary.

### 4. Simple Design
At any point, the design is the simplest thing that works. No speculative generality. No "we might need this later." The four rules: passes tests, reveals intention, no duplication, fewest elements (see beck-tdd).

**The key tension:** simple design feels uncomfortable when you're used to "designing ahead." XP's answer: YAGNI. The design evolves as requirements evolve. Trust the refactoring muscle.

### 5. Test-Driven Development
Write the test first. (See beck-tdd for the full treatment.)

### 6. Refactoring
Continuously improve the design without changing behavior. The safety net (tests) makes this possible. (See fowler-refactoring for the catalog.)

### 7. Pair Programming

> "All production code is written by two people at one machine." — Beck

Driver types; navigator reviews in real-time. Switch roles frequently. Pair is not one person teaching and one person watching — both are actively working, just at different altitudes.

**What pairing does:**
- **Continuous code review**: bugs caught in real-time, not days later
- **Knowledge sharing**: no single points of failure; the codebase is collectively understood
- **Focus**: social pressure prevents the individual from context-switching to Slack
- **Mentorship**: junior-senior pairs transfer knowledge faster than code review

**The objection:** "pairs are 50% as productive." Beck's response: pairs produce code with far fewer bugs and far less technical debt; the re-work avoided more than compensates. Studies support this.

### 8. Collective Code Ownership

Any developer can improve any part of the codebase at any time. No "that's Joe's code." No asking permission. If you see a problem, fix it.

**The prerequisite:** good test coverage. Collective ownership without tests creates chaos. With tests, any developer can refactor confidently — the tests tell you if you broke something.

### 9. Continuous Integration (the Original Meaning)

Integrate your changes with the team's changes **multiple times per day**. Not "when your feature is done." Every time you step away from your desk.

Beck coined CI before CI servers existed. The principle: **the longer you delay integration, the harder integration becomes.** Feature branches that live for a week have a painful merge. Changes integrated every few hours never diverge far enough to conflict badly.

Modern CI (automated builds, GitHub Actions) implements this infrastructure — but the practice is what matters: small, frequent integrations to a shared main branch.

### 10. On-site Customer

A real customer (or proxy) is physically available to the team, answers questions, and writes acceptance tests. The team never has to guess what the customer wants — they can ask.

The modern equivalent: product manager embedded in the engineering team, direct access to users, or short feedback loops via analytics + user interviews.

### 11. 40-Hour Week (Sustainable Pace)

> "Work only as many hours as you can continue indefinitely." — Beck

Tired developers make mistakes. Overtime debt compounds. A team that works 50-hour weeks for a quarter will be slower in month 4 than a team that held 40 hours throughout.

The rule: no two consecutive weeks of overtime. If the project needs overtime, the plan is wrong — fix the plan, not the hours.

### 12. Coding Standards

The whole team writes code as if it was written by one person. Consistent style eliminates the friction of reading unfamiliar code. Not "my style is better" — one agreed style, automated by a linter.

## The XP Values (the Why Behind the Practices)

- **Communication**: practices that require talking reduce misunderstanding (pairing, on-site customer)
- **Simplicity**: practices that reduce complexity before it accretes (simple design, small releases)
- **Feedback**: practices that shorten the feedback loop (TDD, CI, small releases)
- **Courage**: practices that require bravery (refactoring aggressively, releasing early, showing unfinished work)
- **Respect**: all practices assume and reinforce that team members respect each other's judgment
