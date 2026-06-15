---
name: domain-model
description: Use when designing or reviewing the domain layer — entities, value objects, aggregates, invariants, domain logic and domain events. Catches the "anemic domain model" (logic-less data bags with all rules in services) and domain code that leaks framework/ORM/persistence concerns. Technology-agnostic; DDD building blocks.
---

# Domain Model Layer

## Overview

The domain layer is **the business rules, independent of how data arrives or is stored.** It's the innermost ring — it depends on nothing outward (no framework, no database, no HTTP). Everything else exists to deliver requests to it and persist its results.

Core principle: **objects own their invariants. A domain object must be impossible to put into an invalid state.** If callers can mutate fields freely and rules live elsewhere, you have an anemic model — data without behavior.

## Building blocks

| Block | Identity? | Mutable? | Purpose |
|---|---|---|---|
| **Entity** | Yes (id) | Yes (through methods) | A thing tracked over time (Order, Customer) |
| **Value Object** | No (equality by value) | No (immutable) | A descriptive value (Money, Address, Email) |
| **Aggregate** | Root entity + members | Via root only | A consistency boundary; outside code touches only the root |
| **Domain Event** | — | Immutable | A fact that happened (OrderPlaced) |
| **Domain Service** | — | Stateless | Logic that doesn't belong to one entity (e.g., a transfer between two accounts) |

## Rich vs anemic — the central distinction

```ts
// ❌ ANEMIC: a data bag; rules live in some service far away
class Order {
  id; customerId; status; total;
  items = [];
  // only getters/setters — anyone can set status="SHIPPED" on an empty order
}

// ✅ RICH: the object protects its own invariants
class Order {
  private constructor(readonly id, private status, private lines) {}

  static place(customer, lines) {
    if (lines.length === 0) throw new DomainRuleError("order needs at least one line");
    const total = Money.sum(lines.map(l => l.subtotal()));
    if (total.greaterThan(customer.availableCredit()))
      throw new DomainRuleError("credit limit exceeded");   // invariant enforced HERE
    const order = new Order(OrderId.next(), "PLACED", lines);
    order.record(new OrderPlaced(order.id, total));         // domain event
    return order;
  }

  ship() {
    if (this.status !== "PLACED") throw new DomainRuleError("only placed orders ship");
    this.status = "SHIPPED";
  }
  get total() { return Money.sum(this.lines.map(l => l.subtotal())); }
}
```

The rich `Order` cannot exist over its credit limit, cannot be empty, and cannot ship out of sequence — **the rules are inseparable from the data.** The service can't accidentally bypass them.

## Value objects — make illegal values unrepresentable

```ts
class Money {                          // immutable, self-validating
  private constructor(readonly cents, readonly currency) {}
  static of(amount, currency) {
    if (amount < 0) throw new DomainRuleError("money cannot be negative");
    return new Money(Math.round(amount * 100), currency);
  }
  add(o) { this.assertSame(o); return new Money(this.cents + o.cents, this.currency); }
  greaterThan(o) { this.assertSame(o); return this.cents > o.cents; }
  equals(o) { return this.cents === o.cents && this.currency === o.currency; }
}
```

Value objects eliminate whole classes of bugs: no negative money, no mixing currencies silently, equality by value. Prefer them over primitives (`string`/`number`) for domain concepts — this is "primitive obsession" avoidance.

## Aggregates — the consistency boundary

An aggregate is a cluster of objects treated as one unit for changes. **Outside code references only the root**; the root enforces the rules for everything inside.

```
Order (aggregate root)
 ├── OrderLine, OrderLine        ← only reachable via Order; no external refs
 └── ShippingAddress (VO)

Rules:
• Reference other aggregates by ID, never by object (Order holds customerId, not Customer).
• One transaction modifies one aggregate (see service-layer).
• Invariants that must always hold → keep inside one aggregate.
```

## Keeping the domain pure

The domain must not import the web framework, the ORM, or know about JSON/HTTP/SQL.

| Tempting leak | Why it's wrong | Instead |
|---|---|---|
| ORM annotations driving behavior | Couples domain to persistence | Map at the persistence layer (persistence-repository) |
| `@JsonProperty` / serialization hints | Couples domain to transport | Map to DTOs at the edge (dto-mapping) |
| Calling a repository from an entity | Inner depends on outer | Load in the service, pass data in |
| `throw new HttpError(422)` | Framework leaks inward | `throw new DomainRuleError(...)`; translate at controller |

A pure domain is testable with plain object construction — no database, no HTTP, no mocks of frameworks.

## Quick Reference

| Goal | Building block |
|---|---|
| A thing with identity and lifecycle | Entity |
| A descriptive, comparable-by-value concept | Value Object (immutable) |
| A change-consistency boundary | Aggregate (act through the root) |
| "This happened" fact for side effects | Domain Event |
| Logic spanning entities, stateless | Domain Service |
| Reference to another aggregate | Hold its **ID**, not the object |

## Common Mistakes

- **Anemic model** — getters/setters only, rules in services. Move invariants into the entity; make state changes go through intention-revealing methods (`order.ship()`, not `order.status = ...`).
- **Primitive obsession** — `string email`, `number amount`. Wrap domain concepts in value objects.
- **Public setters** — let callers reach invalid states. Mutate only through methods that enforce rules.
- **Framework/ORM/HTTP leaking in** — keep the domain dependency-free; map at the boundaries.
- **Aggregates referencing each other by object** — hold IDs; load via repositories in the service.
- **Giant aggregates** — model the smallest cluster that must stay consistent together.

**REQUIRED COMPANION:** The service-layer invokes domain behavior; persistence-repository maps aggregates to storage without leaking; dto-mapping exposes domain data outward. See clean-architecture for the dependency rule.
