---
name: service-layer
description: Use when writing or reviewing an application/service/use-case layer — orchestrating a business operation across domain objects and repositories, setting transaction boundaries, publishing events, and coordinating side effects. Catches "God services" stuffed with business rules, mapping, or framework/SQL code. Technology-agnostic.
---

# Service / Application Layer

## Overview

The service (application) layer **orchestrates a single use-case**: it loads what's needed, invokes domain behavior, persists the result, and triggers side effects — in the right order, inside the right transaction. It is the *conductor*, not the *musician*: the business rules live in the domain; the service just sequences the steps.

Core principle: **a service method tells a story of one use-case in plain steps. If it contains an `if` enforcing a business rule, that rule probably belongs in the domain.**

## What the service owns

1. **Orchestration** — the sequence of steps for one use-case.
2. **Transaction boundary** — what commits together, what rolls back together.
3. **Loading & saving** via repository *interfaces* (not the DB directly).
4. **Invoking domain behavior** — call methods on entities/aggregates that enforce rules.
5. **Side effects** — publish events, enqueue jobs, send notifications (often *after* commit).
6. **Cross-aggregate coordination** — when one use-case touches multiple aggregates.

## The shape of a clean service

```ts
// Application use-case: place an order
class OrderService {
  constructor(
    private customers: CustomerRepository,   // domain-defined interfaces
    private orders: OrderRepository,
    private events: EventPublisher,
    private tx: TransactionManager,
  ) {}

  async place(cmd: PlaceOrderCommand): Promise<Order> {
    return this.tx.run(async () => {                 // 2. transaction boundary
      const customer = await this.customers.byId(cmd.customerId); // 3. load
      if (!customer) throw new NotFoundError("customer");

      const order = customer.placeOrder(cmd.lines);  // 4. DOMAIN enforces rules
                                                      //    (credit limit, totals…)
      await this.orders.save(order);                  // 3. persist via interface

      this.events.publish(new OrderPlaced(order.id)); // 5. side effect
      return order;
    });
  }
}
```

Notice what's **absent**: no HTTP types, no SQL, no `total > creditLimit` check (that's in `customer.placeOrder`), no DTO mapping (that's at the edge). The service reads like a recipe.

## The "God service" anti-pattern

```ts
// ❌ everything crammed into the service
async place(req) {
  // shape validation (belongs in controller)
  if (!req.items?.length) throw new Error("no items");
  // business rule (belongs in domain)
  const total = req.items.reduce((s,i)=>s+i.price*i.qty, 0);
  const cust = await db.query(...);                 // SQL (belongs in repository)
  if (total > cust.credit_limit) throw new Error(...); // rule (belongs in domain)
  // mapping (belongs in a mapper)
  const dto = { id: ..., customerName: cust.name, ... };
  await db.query("INSERT ...");                     // SQL again
  return dto;
}
```

This method knows about HTTP shape, business math, the database, and presentation. It can't be tested without a DB, can't reuse the credit rule, and grows unboundedly. Split: rules → domain, SQL → repository, mapping → mapper.

## Transaction boundaries

The service decides the unit of consistency. Rules of thumb:

- **One use-case = one transaction** where possible.
- **One transaction should modify one aggregate.** Crossing aggregates? Prefer eventual consistency (publish an event, let another handler react) over a giant transaction.
- **Side effects that can't roll back** (emails, external API calls) go *after* commit — or use the outbox pattern so they're transactional with the state change.

```ts
await tx.run(async () => { ... });      // state changes commit atomically
await mailer.send(...);                 // non-transactional effect, after commit
```

## Services vs domain — who decides what

| Decision | Lives in |
|---|---|
| *Can* this order be placed (credit, stock, status)? | Domain |
| *In what order* do we load, validate, save, notify? | Service |
| What commits together? | Service (transaction) |
| How is total computed? | Domain |
| Which repository/queue/email provider? | Injected interfaces; wired outside |

If a "service" has lots of business `if`s, you likely have an **anemic domain** — push that logic into entities (see domain-model).

## Quick Reference

| Do | Don't |
|---|---|
| Orchestrate one use-case as ordered steps | Embed business rules (delegate to domain) |
| Depend on repository/port *interfaces* | Write SQL or call the ORM directly |
| Own the transaction boundary | Do shape validation (controller) or mapping (mapper) |
| Publish domain events for side effects | Call external systems inside the core transaction |
| Throw domain/app exceptions | Return or throw HTTP/framework types |

## Common Mistakes

- **Business rules in the service** → anemic domain. Move invariants into entities/aggregates.
- **Direct DB/ORM use** → couples the use-case to persistence. Depend on a repository interface.
- **Doing mapping here** → use a mapper; the service deals in domain objects and commands.
- **One transaction spanning many aggregates** → contention and coupling; prefer events + eventual consistency.
- **Non-rollback side effects inside the transaction** → inconsistent state on failure; do them after commit or via outbox.

**REQUIRED COMPANION:** Services invoke domain-model behavior, persist through persistence-repository interfaces, and receive commands built by dto-mapping. See clean-architecture for the dependency rule.
