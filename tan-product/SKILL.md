---
name: tan-product
description: Use when making product decisions, evaluating startup ideas, deciding whether to build something, prioritizing features, designing for early users, or thinking about the relationship between engineering and product strategy. Applies Garry Tan's (YC president, ex-Initialized Capital, ex-Posterous) product and founder philosophy — founder mode, talking to users, design-engineering integration.
---

# Garry Tan — Product & Founder Philosophy

## Core Philosophy

> "Make something people want." — Paul Graham / YC axiom, operationalized by Tan

Garry Tan's work — as a founder (Posterous), investor (Initialized Capital, backing Coinbase, Instacart, Dropbox early), and YC president — consistently returns to one idea: **the best technology companies are built by people who are simultaneously engineers and product designers, who talk to users relentlessly, and who remain in the details long after most leaders delegate their way out.**

## Founder Mode — Stay in the Details

The 2024 essay by Paul Graham on "founder mode" (which Tan has championed and operationalized at YC) challenges the conventional wisdom that scaling a company means delegating everything:

> "The right way to run a company is more like running a family than managing a corporation." — PG, describing what founders do vs professional managers

Founder mode means:
- **Skip-level access**: the CEO talks directly to engineers, designers, and customers — not only to direct reports. Direct reports can hide problems; skip-levels surface them.
- **Being in the design review**: not just approving final specs, but being in the room where decisions are made. Steve Jobs did design reviews. Tan argues most successful technical founders do too.
- **Doing the thing yourself first**: before hiring for a function, understand it well enough to evaluate who does it well. Founders who never did sales can't hire or manage a sales team effectively.
- **High-resolution feedback**: "this is wrong" is not founder mode feedback. "The button's in the wrong place because users scan top-left first and the CTA needs to be where their eye already is" — that is.

**The failure mode it prevents:** Potemkin-village management — where everyone downstream optimizes for looking good to the layer above them rather than actually solving the problem.

## Talking to Users — the Discipline, Not the Activity

Talking to users is not a one-time exercise. Tan treats it as a continuous practice with a method:

1. **Talk to the user who churned first.** Not the happy user who will validate your assumptions — the one who left. They have the highest-signal feedback and the least incentive to be polite.
2. **Ask about their life, not your product.** "Walk me through the last time you tried to do X" → what they actually do. "What do you think of feature Y?" → what they think you want to hear.
3. **Listen for the problem behind the complaint.** A user says "the onboarding is confusing." The underlying problem might be "I don't trust that this will work before I invest time setting it up." Fix the trust problem, not the confusion.
4. **The Mom Test (via Rob Fitzpatrick):** Would your mom lie to spare your feelings? Apply the same test to every user conversation — if the question can be answered with validation, you're asking the wrong question.

## Design + Engineering = Founder's Competitive Advantage

Tan (who was a designer before a developer) argues that the most defensible early-stage companies have founders who **hold design and engineering in their head simultaneously**:

- **Design is not polish.** Design is the decision about what to build and what to cut. A feature nobody uses that ships perfectly is a design failure.
- **Engineering is design.** The technical decisions — what to build in-house vs. off-the-shelf, what the data model is, how fast the core action is — are product decisions.
- **The gap is where bad products live.** When design hands a spec to engineering, both sides optimize for their local incentives (beautiful mockup vs. clean code). The product that ships is owned by neither. Founders in the details prevent this.

```
Bad process:
  PM writes spec → designer creates mockup → engineer implements → product is nobody's baby

Good process:
  Founder understands the problem → designer, engineer, and founder in the same room →
  tradeoffs made explicitly → everyone owns the outcome
```

## YC's Framework for Evaluating Ideas

From YC office hours and Tan's public commentary:

| Question | What it filters |
|---|---|
| *Who specifically is the customer?* | Vague answers → vague product |
| *How do they solve this problem today?* | No status quo → no real problem |
| *Why hasn't someone built this already?* | Good answers: new tech, new regulation, insight others lack |
| *How do you get your first 10 users?* | If you can't name them, the idea isn't concrete enough |
| *Would they be devastated if this went away?* | Nice-to-have vs. need-to-have |
| *What's the simplest version that proves demand?* | Week 1 test, not month 6 launch |

**The nuclear option (Tan's version):** Call 10 potential customers before writing a line of code. If you can't get 10 people to take a 20-minute call about their problem, the problem isn't real enough to build a company on.

## Do Things That Don't Scale — the Technical Translation

Paul Graham's essay, applied by Tan to engineering decisions early on:

- **Manual before automated.** Run the process manually first. If nobody uses it manually, nobody will use the automated version. Build the automation after you've proven the manual process is worth automating.
- **Concierge MVP.** Do the thing for the user yourself (Airbnb taking photos of apartments, Stripe manually setting up payment accounts). This exposes every friction point before you code it.
- **Instrument before optimizing.** Don't optimize the signup flow before you have 100 people going through it and you understand where they drop off. Data drives optimization; optimization without data is guessing.
- **Fight the urge to generalize.** The first version of the product should solve one user's problem perfectly, not solve many users' problems adequately. You can generalize once you know what "solve perfectly" actually looks like.

## When to Build vs Buy vs Not Build

Tan's heuristic: **build only the thing that is your core differentiation.**

```
Core differentiation (build):
  → The thing users choose you FOR. Unique, defensible, improving over time.

Everything else (buy / off-the-shelf / defer):
  → Auth, payments, email, infra, analytics, CRM.
  → The cost of building it yourself is time not spent on differentiation.
  → "We built our own auth" is not a competitive advantage unless you're Okta.
```

The traps: building infra too early (pre-optimization), re-implementing off-the-shelf tools (NIH syndrome), and solving the wrong generalization (building a platform before you have one product that works).
