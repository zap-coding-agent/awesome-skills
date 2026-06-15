---
name: pocock-typescript-generics
description: Use when TypeScript generics feel too hard or too loose — writing generic utilities, using conditional types with infer, mapping over types, building type-safe function overloads, or when you get "Type instantiation is excessively deep" errors. Applies Pocock's advanced type-level programming approach.
---

# Pocock — Advanced TypeScript Generics

## The Mental Model: Types Are a Programming Language

TypeScript has two layers: the value layer (JavaScript) and the type layer. **The type layer is Turing-complete and supports its own form of programming:** conditionals (`extends ? : `), mapping (`[K in keyof T]`), recursion, and pattern matching (`infer`). Mastering TypeScript means getting comfortable writing programs at the type level.

## Generic Constraints — the "Lock and Key" Pattern

```ts
// ❌ T is unconstrained — TypeScript can't help inside the function
function pluck<T>(arr: T[], key: string): unknown[] {
  return arr.map(item => (item as any)[key]);   // forced to use any
}

// ✅ K extends keyof T — two type parameters constrain each other
function pluck<T, K extends keyof T>(arr: T[], key: K): T[K][] {
  return arr.map(item => item[key]);   // fully typed, no casts
}

const names = pluck([{name: "Alice", age: 30}], "name");   // string[]
const ages  = pluck([{name: "Alice", age: 30}], "age");    // number[]
const bad   = pluck([{name: "Alice", age: 30}], "foo");    // ❌ compile error
```

The lock-and-key pattern: `K extends keyof T` locks K to the actual keys of T. TypeScript can then infer that `T[K]` is the value type for that key. This pattern powers most useful generic utilities.

## `infer` — Extract Types from Patterns

`infer` captures a type from inside a conditional type. It's the workhorse of utility types:

```ts
// Extract the return type of any function
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
//                                                     ^^^^ infer R captures the return type

type A = ReturnType<() => string>;           // string
type B = ReturnType<(x: number) => boolean>; // boolean

// Extract the element type of an array or Promise
type Awaited<T>  = T extends Promise<infer U> ? U : T;
type Element<T>  = T extends (infer U)[] ? U : never;

type C = Awaited<Promise<string>>;   // string
type D = Element<number[]>;          // number

// Extract first argument of a function
type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;
type E = FirstArg<(a: string, b: number) => void>;  // string
```

**The pattern:** `T extends <pattern with infer X> ? X : fallback`. TypeScript tries to match T against the pattern; if it matches, X is set to the matched type.

## Mapped Types — Transform Every Key

```ts
// The standard library implements these with mapped types:
type Partial<T>  = { [K in keyof T]?: T[K] };          // all optional
type Required<T> = { [K in keyof T]-?: T[K] };          // remove optional (- removes the ?)
type Readonly<T> = { readonly [K in keyof T]: T[K] };   // make immutable
type Mutable<T>  = { -readonly [K in keyof T]: T[K] };  // remove readonly

// Custom: only allow certain key types
type OnlyStrings<T> = { [K in keyof T]: T[K] extends string ? T[K] : never };
type StringKeys<T>  = { [K in keyof T]: T[K] extends string ? K : never }[keyof T];

const user = { id: 1, name: "Alice", email: "a@b.com", age: 30 };
type StringFields = StringKeys<typeof user>;   // "name" | "email"
```

**Key remapping with `as`** (TS 4.1):
```ts
// Prefix every key with "get"
type Getters<T> = { [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K] };
type UserGetters = Getters<{name: string; age: number}>;
// { getName: () => string; getAge: () => number }
```

## Distributive Conditional Types

When a conditional type's input is a union, it **distributes** — runs separately for each union member:

```ts
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string | number>;   // "yes" | "no"  ← distributes!
// Not: (string | number) extends string ? "yes" : "no"  ← "no"
// But: (string extends string ? "yes") | (number extends string ? "no")  ← "yes" | "no"

// Use [T] to prevent distribution:
type IsStringNoDist<T> = [T] extends [string] ? "yes" : "no";
type B = IsStringNoDist<string | number>;   // "no" ← treats union as a whole
```

**Building `Exclude` and `Extract`** using distribution:
```ts
type Exclude<T, U> = T extends U ? never : T;
// Exclude<"a" | "b" | "c", "a"> → never | "b" | "c" → "b" | "c"

type Extract<T, U> = T extends U ? T : never;
// Extract<string | number | boolean, string | number> → string | number
```

## Template Literal Types — String Manipulation at the Type Level

```ts
type EventName<T extends string> = `on${Capitalize<T>}`;
type A = EventName<"click" | "hover">;   // "onClick" | "onHover"

// Parse a route string into parameter names:
type ParseRoute<T extends string> =
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | ParseRoute<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
    ? Param
    : never;

type Params = ParseRoute<"/users/:userId/posts/:postId">;   // "userId" | "postId"
```

Template literal types + `infer` = string parsing at compile time. Used in routing libraries (tRPC, Hono) to infer URL parameter names.

## Recursive Types (and the "Type instantiation is excessively deep" error)

```ts
// Deep Readonly — recursively apply Readonly
type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

// Deep Partial
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;
```

**"Type instantiation is excessively deep"** = your recursive type hit TypeScript's depth limit (~100 levels). Fixes:
1. Add a depth counter parameter: `type Deep<T, Depth extends number[] = []> = Depth["length"] extends 10 ? T : ...`
2. Use `interface` instead of `type` for self-referential structures (interfaces are lazily evaluated).
3. Add a base case that short-circuits for primitives (already done above with `T extends object`).

**REQUIRED COMPANION:** pocock-typescript covers the foundations (inference, discriminated unions, Zod). pocock-tsconfig covers the compiler settings that make these techniques work correctly.
