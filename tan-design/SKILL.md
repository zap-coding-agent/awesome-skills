---
name: tan-design
description: Use when making design decisions for a product, evaluating whether a UI is good, working with designers as an engineer, applying design thinking to technical problems, or understanding how Garry Tan's design background shapes his product philosophy. Covers taste, visual hierarchy, the engineer-designer collaboration, and how good design compounds.
---

# Tan — Design Thinking for Engineers

## Core Philosophy

> "Design is not decoration. Design is the decision of what to include and what to leave out." — Garry Tan

Garry Tan studied design at Stanford, worked as a designer, and built Posterous (which had design as a core differentiator). His view: the best technical founders understand design not as aesthetics but as **product decisions made visible**.

The competitive advantage for engineer-founders who learn design: they can close the loop between "what does the user see" and "what does the code do" without a translation layer. The friction of design-to-engineering handoff disappears.

## The Designer's Eye — What to Train

Design intuition is learnable. The elements:

### Visual Hierarchy
Everything on a screen competes for attention. Good design directs attention deliberately:
- **Size**: larger = more important
- **Contrast**: high contrast = more important (dark text on light, primary color on neutral)
- **Position**: top-left to bottom-right is reading order; place the most important thing first
- **Whitespace**: space around an element makes it feel important; cramming things together reduces the value of each

```
❌ Bad hierarchy: 5 buttons the same size, same color, same weight
✅ Good hierarchy: 1 primary action (filled, prominent), 2-3 secondary (outlined), rest as links
```

### Consistency as Trust
Visual inconsistency signals carelessness. Users unconsciously notice:
- Different font sizes for similar content
- Buttons with different border radii
- Icons with different visual weights
- Colors that are almost-but-not-quite the same (#333 vs #3a3a3a)

**The rule:** use a design system or define your tokens once. Every spacing value from a scale (4, 8, 12, 16, 24, 32…). Every color from a palette. Every font size from a type scale. "Close enough" is not.

### The Gestalt Principles (the non-obvious ones)
- **Proximity**: things that are close together are perceived as related. Use spacing to group and separate.
- **Common region**: a border or background groups things more powerfully than proximity alone.
- **Continuity**: the eye follows lines and curves. Use alignment to create visual flow.
- **Similarity**: things that look alike are understood as the same type of thing. Use consistent styling for similar actions.

## The "Is This Good?" Checklist (Tan's criteria)

1. **Can a new user understand what to do in 5 seconds?** First-time user test — not "can they figure it out eventually," but "is it immediately obvious."
2. **Is there one clear primary action?** If everything is equally prominent, nothing is.
3. **Does the empty state communicate value?** First-run experience is where most users churn. An empty state should show what's possible, not just "you have no items."
4. **Does it work at the extremes?** 100 items, 0 items, very long text, very short text, mobile width.
5. **Does the feedback confirm the action?** After a user acts, they need confirmation. Missing loading states, error states, and success states are design bugs.
6. **Does it feel fast?** Perceived performance is design. Optimistic UI, skeleton screens, and instant visual feedback are as important as actual load time.

## Design Tokens — the Engineering Implementation of Design Consistency

Tan's view: engineers who understand design implement systems that enforce consistency at the code level:

```ts
// ❌ Magic numbers: inconsistency compounds
<div style={{ padding: "11px", marginBottom: "13px", fontSize: "15px" }} />

// ✅ Design tokens: consistency is automatic
// tokens.ts
export const spacing = { xs: 4, sm: 8, md: 16, lg: 24, xl: 32 } as const;
export const fontSize = { sm: 12, base: 16, lg: 20, xl: 24, "2xl": 32 } as const;
export const colors = {
  primary: { 500: "#6366f1", 600: "#4f46e5" },
  neutral: { 50: "#f9fafb", 900: "#111827" }
} as const;

// Components use tokens; spacing and color never drift
<div style={{ padding: spacing.md, marginBottom: spacing.sm, fontSize: fontSize.base }} />
```

Tailwind CSS is a token system. Radix UI + a theme is a token system. The principle holds regardless of tool.

## The Designer-Engineer Collaboration Anti-Patterns

Tan has seen these at hundreds of YC companies:

| Anti-pattern | What happens | Fix |
|---|---|---|
| Designer hands off final spec, engineer implements exactly | Engineer implements something visually different because constraints weren't communicated | Engineer and designer work together on the hard parts |
| Engineer adds a feature without design review | 3 months of features → inconsistent, cluttered product | Every user-visible change gets a design review, even a quick one |
| Designer uses non-existent font sizes / spacing | Engineer has to "approximate" → consistency breaks | Design system with tokens both parties use |
| "We'll polish it later" | Later never comes | Ship polished or don't ship. Polish is not a separate phase. |
| Design and engineering in separate sprints | Engineer discovers design constraint during implementation, too late | Engineers in design reviews; designers in engineering planning |

## Taste Is Trainable

> "Taste is the ability to recognize what's good and what's not, before you can explain why." — Tan

How to train design taste as an engineer:
1. **Screenshot great UIs.** Maintain a swipe file of interfaces you admire. Study what they have in common.
2. **Annotate the bad.** When a UI frustrates you, articulate why. "The buttons are too close together" is more useful than "it's bad."
3. **Copy before you create.** Reproduce a UI you admire from scratch. Understanding how it was built gives you vocabulary.
4. **Get feedback fast.** Show your designs to someone who wasn't involved. The first 30 seconds of their interaction reveals everything.
