---
name: typescript-type-philosophy
description: Use when designing TypeScript type signatures, choosing between generics/unions/intersections/conditional types, deciding how much type coverage to add, debugging "type instantiation too deep" or unsound types, or understanding why TypeScript makes specific trade-offs (structural typing, soundness vs completeness). Covers the TypeScript team's design goals and the practical implications.
---

# TypeScript — Type System Design Philosophy

## The Governing Design Goals

The TypeScript team's stated goals, in order:

1. **Be useful to JavaScript programmers.** Types must work with JS idioms, not force Java patterns.
2. **Maintain a one-to-one correspondence with JS at runtime.** No runtime magic; types are erased entirely.
3. **Soundness is NOT a goal** — *usefulness* is. TS deliberately allows unsound type constructions when the alternative is unergonomic. The type system is a bug-finder, not a proof system.
4. **Structural typing, not nominal.** Two types are compatible if they have the same shape — regardless of names or ancestry.

> "TypeScript is JavaScript with syntax for types." — TypeScript homepage

## Structural Typing: the Core Bet

```ts
interface Point2D { x: number; y: number; }
interface Named   { name: string; }

function greet(n: Named) { console.log(n.name); }

// This works — {name, x, y} is structurally compatible with Named
const p = { name: "origin", x: 0, y: 0 };
greet(p);    // ✅ — JS objects are just bags of properties
```

**Why structural:** JavaScript objects are open and duck-typed. Nominal typing (Java/C#) would require explicit `implements Named` everywhere, breaking JS idioms like libraries that return object literals. Structural typing matches how JS actually works.

**The implication:** TypeScript does not distinguish `{ id: string }` coming from your UserService from `{ id: string }` coming from a ProductService. Both satisfy the same interface. This is usually right; use branded types when it's not:

```ts
type UserId = string & { readonly _brand: "UserId" };
type ProductId = string & { readonly _brand: "ProductId" };
// Now UserId and ProductId are incompatible even though both are strings
```

## Soundness vs Completeness: the Deliberate Trade-off

A sound type system would guarantee: if the program type-checks, it has no type errors at runtime. TypeScript does **not** guarantee this — and that's intentional:

```ts
const arr: string[] = [];
const first = arr[0];   // TypeScript infers: string (but at runtime: undefined)
```

If TS were sound here, you'd need `string | undefined` for every array access — correct but too noisy for the real-world JS codebase TS was designed to serve. `noUncheckedIndexedAccess` opts into stricter behavior if you want it.

**Practical implication:** TypeScript narrows the solution space of bugs but does not eliminate them. Design your code defensively at runtime boundaries (API responses, user input) regardless of types.

## The Advanced Type Toolkit

| Feature | When to use |
|---|---|
| `Union (A \| B)` | A value that is one of several alternatives |
| `Intersection (A & B)` | A value that satisfies multiple contracts |
| `Generic <T>` | Algorithm or structure that works over any type |
| `Conditional type (T extends U ? A : B)` | Types that change based on a type parameter |
| `Mapped type ({ [K in keyof T]: ... })` | Derive a new type by transforming each key of another |
| `Template literal type` | String-literal types that compose |
| `infer` in conditional types | Extract a type from a pattern |

**Use the simplest feature that expresses the constraint.** A union often replaces a complex conditional type. A simpler generic with a constraint often replaces a mapped type. Type complexity has maintenance cost; don't over-engineer types.

## Discriminated Unions (the Right Tool for State)

```ts
// ❌ optional fields mix states — loader state is confused
type State = { loading?: boolean; data?: User; error?: string };

// ✅ discriminated union — each state is exact
type State =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: string };

// TypeScript narrows automatically on the discriminant:
if (state.status === "success") {
  state.data;   // type: User (not User | undefined)
}
```

Every time you have optional fields that are only valid together, a discriminated union removes the possibility of the impossible state existing in the type system.

## `unknown` vs `any` — the Type Safety Boundary

- `any`: I give up. Opt out of the type system for this value. Contagious — passes anywhere without complaint.
- `unknown`: I received something I haven't characterized yet. Forces you to narrow before use.

```ts
// ❌ any bypasses all checks
function parse(raw: any) { return raw.user.id; }   // silently wrong if shape differs

// ✅ unknown forces handling
function parse(raw: unknown): User {
  if (!isUser(raw)) throw new Error("unexpected shape");
  return raw;
}
```

Use `unknown` at all runtime boundaries (JSON.parse, API responses, event handlers). Never use `any` except as a last resort with a comment explaining why.

## Type-Level Design Principles

1. **Make impossible states unrepresentable.** Use discriminated unions, not optional fields, for mutually-exclusive states.
2. **Parse, don't validate.** At boundaries (API, user input), parse raw values into typed domain objects once; trust the type inside. (Zod, io-ts, or manual narrowing.)
3. **Return types are contracts.** Annotate public function return types explicitly — inferred return types for public APIs leak implementation details and break callers on refactoring.
4. **Strict mode is the floor.** `strict: true` enables `strictNullChecks`, `noImplicitAny`, and others. Starting without it and enabling later is painful. Start strict.
5. **Avoid assertion casting (`as Foo`)** — it's an `any` you spelled differently. Use type guards or discriminated unions instead.
