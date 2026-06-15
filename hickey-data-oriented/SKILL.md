---
name: hickey-data-oriented
description: Use when deciding between data-oriented and object-oriented design, building systems that process or transform data across boundaries, designing APIs that are easy to combine and extend, evaluating whether encapsulation is helping or hurting, or when a system is rigid and hard to extend. Applies Rich Hickey's data-oriented programming philosophy.
---

# Hickey — Data-Oriented Programming

## The Central Argument

> "We are complecting information and the mechanisms for dealing with it. Data is this beautiful, naked thing. It can go anywhere, do anything. But we clothe it in objects and then it can only go where those objects' methods say it can go." — Hickey

Object-oriented programming couples data and behavior. Hickey argues this coupling is almost always a mistake: it creates closed silos that resist combination, require conversion at every boundary, and make generic operations (logging, serialization, comparison, transmission) require special handling per type.

The alternative: **data is data.** Use the simplest possible representation (maps, vectors, sets, keywords). Behavior is separate functions that operate on that data. The world is much more composable.

## Data vs Objects — the Core Distinction

```
Object-Oriented world:
  - Data lives inside objects, hidden behind methods
  - To print an object: override toString() or implement Printable
  - To serialize it: implement Serializable or add Jackson annotations
  - To compare it: implement Comparable or Comparator
  - To store it in a cache: implement Cacheable (or write a special adapter)
  Each object is a closed world; every boundary requires a translation.

Data-Oriented world (Clojure / modern FP / good API design):
  - Data is a plain map: {:user/id "123" :user/name "Alice" :user/role :admin}
  - Printing: already works (it's just data)
  - Serialization: json/encode handles any map
  - Comparison: = on maps works by value
  - Caching: it's a value, cache it directly
  The same data flows through all of these without transformation.
```

## The "Information Model" vs the "Implementation Model"

Hickey distinguishes:
- **Information**: the facts. User 123 has name "Alice" and role "admin". This is eternal — it was true, is true, will be recorded.
- **Implementation**: how you process it. A class with methods, a struct, a database row. This is contingent — it depends on your technology choices.

The mistake: confusing the two. Designing your information model around your implementation language means your information can only be processed by that language, those classes, that framework.

**Design the information model first, independently.** Then choose an implementation that represents it cleanly.

## Spec / Schema at the Data Level

If data is a map, how do you validate it? With a schema at the data boundary — not with type annotations baked into a class hierarchy.

```clojure
;; Clojure spec — validates data, not objects
(s/def :user/id string?)
(s/def :user/name (s/and string? #(> (count %) 0)))
(s/def :user/role #{:admin :user :viewer})
(s/def ::user (s/keys :req [:user/id :user/name :user/role]))

;; Validate anywhere data crosses a boundary
(s/valid? ::user {:user/id "123" :user/name "Alice" :user/role :admin})  ; true
```

The equivalent in TypeScript: Zod schemas at API boundaries (as Pocock advocates) — the schema is the spec, and the type is derived from it. The data lives as plain values; the schema validates it at the edges.

## Open Systems vs Closed Systems

OOP's encapsulation creates **closed systems**: you can only do what the object's interface allows. This is good for protecting invariants. It is bad for:
- Extensibility: adding a new operation requires modifying the class (or the Open/Closed Principle headache)
- Generic processing: you can't log, serialize, or transmit without the class cooperating
- Combination: two objects can't be merged without a method designed for it

Hickey's preference: **design open systems where possible.** Functions that take data can be combined in ways the original author never anticipated. A function `(defn add-audit-trail [data] (assoc data :updated-at (now)))` works on any map — no class modification needed.

**The practical application:** design your internal APIs as functions over plain data structures. Reserve OOP-style encapsulation for the boundaries where invariants must be protected.

## Immutability — Values Over Places

```
Place-oriented (mutable):
  user = new User("Alice")
  user.role = "admin"   // mutation: the past is gone
  // What was user.role before? Unknown. Any other code holding a ref to user sees the change.

Value-oriented (immutable):
  user = {name: "Alice", role: "user"}
  adminUser = {...user, role: "admin"}  // new value; original unchanged
  // Both exist simultaneously; can be compared, logged, cached
```

Immutable data is **safe to share, safe to cache, safe to pass to any function.** Mutable data must be defended: synchronized, copied before passing, defensively cloned in tests. The entire defensive programming apparatus around mutable data is the cost of mutability.

**Where to apply in practice:** function arguments and return values should be values (immutable). State transitions produce new values, not mutations of old values. State lives at the boundary (Redux, Clojure atoms, database) — the processing in the middle is pure.

## The Generic Operations Advantage

When your data is plain maps and vectors, every generic operation "just works":

```js
// All of these work on plain data; none require class-specific code:
JSON.stringify(user)             // serialization
console.log(user)                // logging
deepEqual(user1, user2)         // comparison
{...user, role: "admin"}         // transformation
cache.set("user-123", user)      // caching
Object.assign({}, defaults, user) // merging
```

The moment you use a class with private fields and custom methods, each of these breaks unless the class implements a special interface. You've traded generic composability for encapsulation.

The Hickey principle: **make encapsulation a deliberate choice for protecting invariants, not the default for all data.**

**REQUIRED COMPANION:** hickey-simplicity covers the philosophical foundation. rich-hickey-simplicity is the overview of his "Simple Made Easy" framework. For the type-system application of these ideas, see pocock-typescript (discriminated unions are the TypeScript equivalent of tagged data).
