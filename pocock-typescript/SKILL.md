---
name: pocock-typescript
description: Use when writing advanced TypeScript — squeezing maximum type safety from inference, building utility types, using generics with precision, reading complex error messages, or when TypeScript feels like it's fighting you instead of helping. Applies Matt Pocock's (Total TypeScript) approach: inference over annotation, constraints over casts, type-level programming as a first-class skill.
---

# Matt Pocock — TypeScript Mastery

## Core Philosophy

> "TypeScript is a tool for making JavaScript safer. The goal is not to type everything — it's to catch the bugs that matter." — Matt Pocock

Pocock's approach (Total TypeScript, ts-reset, Zod, t3-oss) centers on one idea: **TypeScript is a programming language at two levels — the value level (JS) and the type level.** Mastering TypeScript means getting good at both. Most developers write JS at the value level and fight TypeScript at the type level. Flip that: embrace type-level programming as a skill worth developing.

## Inference Over Annotation — the First Principle

```ts
// ❌ Over-annotated: fighting TypeScript
const user: { id: string; name: string; role: "admin" | "user" } = {
  id: "123", name: "Alice", role: "admin"
};
// TypeScript already knows this. You're repeating yourself.

// ✅ Let inference work; annotate only at boundaries
const user = { id: "123", name: "Alice", role: "admin" as const };
type User = typeof user;   // derive the type from the value
```

Annotate function *parameters* and *return types* (the public contract). Let TypeScript infer everything inside.

## `as const` — Unlock Literal Types

Without `as const`, TypeScript widens literals to their primitive type:
```ts
const config = { env: "production", port: 3000 };
// TypeScript infers: { env: string; port: number }  ← too wide

const config = { env: "production", port: 3000 } as const;
// TypeScript infers: { readonly env: "production"; readonly port: 3000 }  ← exact
```

`as const` is the primary tool for turning runtime values into precise types. Essential for discriminated unions, lookup tables, and config objects.

## Generics — Constraints Are the Point

The most common generic mistake: using `<T>` when you should use `<T extends Something>`.

```ts
// ❌ T is too wide — inside the function, TypeScript knows nothing about T
function getProperty<T>(obj: T, key: string) { return (obj as any)[key]; }

// ✅ constrain T so TypeScript can reason about it
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];   // return type is T[K] — exact, inferred, correct
}
const name = getProperty(user, "name");   // type: string ✓
const bad  = getProperty(user, "foo");    // compile error ✓
```

The pattern `K extends keyof T` is Pocock's "lock and key" — the two type parameters constrain each other, giving TypeScript enough information to infer the output type precisely.

## Discriminated Unions — Prefer Over Optional Fields

```ts
// ❌ optional fields: impossible states are representable
type Result<T> = { data?: T; error?: string; loading?: boolean };

// ✅ discriminated union: each state is exact
type Result<T> =
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

// TypeScript narrows automatically:
if (result.status === "success") result.data;   // T, not T | undefined
```

Discriminated unions make impossible states unrepresentable at the type level. The status field (or `kind`, `type`, `_tag`) is the discriminant.

## Reading TypeScript Errors — the Skill Nobody Teaches

Pocock's method for complex TS errors:
1. **Find the deepest error.** TypeScript errors cascade — the first line is often the symptom, not the cause. Read bottom-up.
2. **Look for "Type X is not assignable to type Y."** X is what you gave. Y is what was expected. The mismatch reveals the gap.
3. **Hover over intermediate types.** In VS Code, hover over every variable on the path to the error — find where the type first diverged from what you expected.
4. **Simplify.** Extract the failing expression into a variable; annotate it with what you think it should be; TypeScript will tell you what it actually is.

```ts
// Technique: annotate to reveal the actual type
const result: ExpectedType = someComplexExpression;   // error will show actual type
```

## The `satisfies` Operator (TS 4.9)

```ts
// ❌ annotation widens the type and loses literal inference
const palette: Record<string, string[]> = {
  red: ["#ff0000", "#cc0000"],
  blue: "#0000ff",             // error — but palette.red is now string[], not the literal tuple
};

// ✅ satisfies: checks the constraint without widening
const palette = {
  red: ["#ff0000", "#cc0000"],
  blue: "#0000ff",             // error caught ✓
} satisfies Record<string, string[]>;
// palette.red is still ["#ff0000", "#cc0000"] (literal) not string[]
```

`satisfies` validates that a value matches a type without widening the inferred type. Use it for config objects, route tables, event maps — anywhere you want to validate shape and keep literals.

## Utility Types Worth Understanding Deeply

```ts
// Build these once from scratch to understand them forever
type Partial<T>  = { [K in keyof T]?: T[K] };       // all optional
type Required<T> = { [K in keyof T]-?: T[K] };       // remove optional
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
type Record<K extends keyof any, V> = { [P in K]: V };
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;
type NonNullable<T> = T extends null | undefined ? never : T;
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : never;
```

`infer` inside a conditional type extracts a part of a type. `ReturnType` shows the pattern: "if T is a function type, infer its return type R, else never."

## Zod — Runtime + Compile-Time Safety Together

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["admin", "user"]),
});

type User = z.infer<typeof UserSchema>;   // type derived from schema — single source of truth
// User = { id: string; email: string; role: "admin" | "user" }

// At the runtime boundary (API response, form input):
const result = UserSchema.safeParse(rawData);
if (result.success) result.data;   // type: User ✓
```

Pocock's principle: **never write a TypeScript type for data that crosses a runtime boundary. Write a Zod schema; derive the type from it.** One source of truth; validation and types stay in sync.

## Common TypeScript Traps Pocock Highlights

| Trap | Why | Fix |
|---|---|---|
| `as Type` everywhere | Silences errors; masks real bugs | Use type guards, `satisfies`, or fix the upstream type |
| `any` for "hard" types | Turns off TypeScript for that branch | `unknown` + narrowing |
| Annotating instead of inferring | Doubles the work; can disagree with runtime | Infer; annotate only the public API |
| Not using `as const` for configs | Widens literals; breaks discriminated unions | `as const` on all literal objects/arrays |
| Optional chaining over discriminated unions | Hides which state you're in | Discriminated union + exhaustive switch |
