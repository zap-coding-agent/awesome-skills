---
name: pocock-tsconfig
description: Use when configuring TypeScript — setting up tsconfig.json for a new project, understanding what strict mode enables, configuring path aliases, setting up project references for monorepos, debugging "cannot find module" or declaration file errors, or understanding which tsconfig options actually matter. Applies Pocock's (Total TypeScript) tsconfig guidance.
---

# Pocock — TypeScript Configuration (tsconfig)

## Core Principle

> "Most TypeScript projects should start with `strict: true` and add from there. Not from a permissive base." — Pocock

tsconfig is a compiler contract. The decisions you make here determine what TypeScript can catch and what it silently misses. Start strict and relax intentionally; the reverse (start loose, tighten later) means fixing hundreds of errors you could have prevented.

## The Minimum Viable Strict Config

```jsonc
// tsconfig.json — start here, deviate intentionally
{
  "compilerOptions": {
    // Strictness — these should almost always be on
    "strict": true,              // enables all strict-family checks (see below)
    "noUncheckedIndexedAccess": true,   // arr[0] is T | undefined, not T (Pocock strongly recommends)
    "exactOptionalPropertyTypes": true, // { x?: string } means x is string | undefined, not assignable to string

    // Module system — modern Node/bundler setup
    "module": "NodeNext",        // or "Bundler" for Vite/webpack
    "moduleResolution": "NodeNext",  // must match module
    "target": "ES2022",          // emit modern JS; adjust per runtime support

    // Output
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,         // emit .d.ts (required if publishing a library)
    "declarationMap": true,      // source maps for .d.ts (enables "go to definition" to source)
    "sourceMap": true,

    // Quality
    "noImplicitReturns": true,   // all code paths must return
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,      // error on unused variables
    "noUnusedParameters": true,  // error on unused function params
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

## What `strict: true` Actually Enables

`strict` is a shorthand that turns on a family of checks. Know what you're getting:

| Flag enabled by `strict` | What it catches |
|---|---|
| `strictNullChecks` | `null`/`undefined` can't be used as if they're the type |
| `strictFunctionTypes` | Function parameter types are contravariant (catches subtle callback bugs) |
| `strictBindCallApply` | `bind`/`call`/`apply` are type-checked |
| `strictPropertyInitialization` | Class properties must be initialized or marked `!` |
| `noImplicitAny` | Variables/params that can't be inferred require explicit annotation |
| `noImplicitThis` | `this` in functions must be typed |
| `useUnknownInCatchVariables` | `catch (e)` gives `e: unknown`, not `e: any` |
| `alwaysStrict` | Emits `"use strict"` in every file |

**The one Pocock adds beyond `strict`:** `noUncheckedIndexedAccess`. Without it, `arr[0]` is typed as `T` even though it might be `undefined`. This catches real bugs:

```ts
const arr: string[] = [];
// Without noUncheckedIndexedAccess:
const first = arr[0];   // type: string — but at runtime: undefined
first.toUpperCase();    // runtime crash, no TS error

// With noUncheckedIndexedAccess:
const first = arr[0];   // type: string | undefined
first?.toUpperCase();   // safe
```

## Module Resolution — the Most Confusing tsconfig Section

The `module` and `moduleResolution` options must be compatible. The modern pairings:

```jsonc
// For Node.js (ESM, node 18+)
"module": "NodeNext",
"moduleResolution": "NodeNext"

// For bundlers (Vite, webpack, Rollup, esbuild)
"module": "ESNext",
"moduleResolution": "Bundler"

// For Node.js (CommonJS, legacy)
"module": "CommonJS",
"moduleResolution": "Node"

// ❌ Outdated combination still seen in old projects:
"module": "CommonJS",
"moduleResolution": "Node10"  // same as "Node", doesn't understand exports field
```

**`NodeNext` requires explicit extensions in imports:**
```ts
// NodeNext module resolution
import { foo } from "./foo";       // ❌ Must specify .js extension
import { foo } from "./foo.js";    // ✅ (TypeScript knows .js → .ts mapping)
```

This trips up projects migrating from CommonJS to ESM. It's intentional: NodeNext mirrors what Node.js actually does.

## Path Aliases

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"]
    }
  }
}
```

```ts
// Instead of:
import { Button } from "../../../components/Button";
// Use:
import { Button } from "@components/Button";
```

**Critical:** tsconfig paths only affect type-checking. The *bundler* (Vite, webpack) or *runtime* (via `tsconfig-paths`) must also be configured to resolve these aliases. Forgetting this causes "module not found" errors at runtime that TypeScript didn't catch.

## Project References (Monorepos)

When you have multiple packages in a monorepo, project references let TypeScript build them incrementally and in dependency order:

```jsonc
// packages/core/tsconfig.json
{
  "compilerOptions": { "composite": true, "outDir": "./dist" },
  "include": ["src"]
}

// packages/app/tsconfig.json  
{
  "compilerOptions": { "outDir": "./dist" },
  "references": [{ "path": "../core" }],   // app depends on core
  "include": ["src"]
}

// root tsconfig.json (orchestration only)
{
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/app" }
  ]
}
```

`tsc --build` (or `tsc -b`) resolves the dependency graph and builds in order. `composite: true` enables incremental compilation.

## Library vs Application Config

```jsonc
// Library (publishing to npm)
{
  "compilerOptions": {
    "declaration": true,        // emit .d.ts — required for consumers
    "declarationMap": true,     // source maps for .d.ts — "go to definition" works
    "sourceMap": true,
    "module": "NodeNext",       // or "ESNext" for bundler-first libs
    "moduleResolution": "NodeNext"
  }
}

// Application (consumed only by you)
{
  "compilerOptions": {
    "declaration": false,       // no need — you're not publishing
    "noEmit": true              // if bundler handles emit (Vite, Next.js)
  }
}
```

For library authors: if you don't emit `.d.ts` files, consumers get `Could not find declaration file`. Always emit declarations.

## Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `Cannot find module './foo'` | Missing `.js` extension (NodeNext) or missing path alias in bundler | Add `.js` extension; configure bundler aliases |
| `Property 'x' has no initializer` | `strictPropertyInitialization` — class property not initialized | Initialize in constructor or add `= value` or `!` (if guaranteed) |
| `Object is possibly undefined` | `noUncheckedIndexedAccess` or `strictNullChecks` | Add null check; use `??` or optional chaining |
| `Type instantiation is excessively deep` | Recursive generic type | Add base case; use `interface`; limit depth |
| `Could not find declaration file` | Library not emitting `.d.ts` | Add `"declaration": true` to library tsconfig |
