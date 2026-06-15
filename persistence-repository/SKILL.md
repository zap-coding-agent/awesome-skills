---
name: persistence-repository
description: Use when designing or reviewing the persistence layer — repository interfaces and implementations, the repository vs DAO distinction, mapping domain aggregates to/from ORM entities or rows, query placement, and keeping the database out of the domain. Catches ORM entities leaking as domain models and business logic in repositories. Technology-agnostic.
---

# Persistence / Repository Layer

## Overview

The repository is **the collection-like illusion over storage**: the domain and service ask for and hand back whole aggregates as if from an in-memory collection, and the repository hides the database, ORM, SQL, and mapping behind that. Persistence is an *outer* layer, but via dependency inversion the **interface lives with the domain and the implementation lives in infrastructure** — so the dependency arrow still points inward.

Core principle: **the domain depends on a repository interface it owns; it never knows the database exists.** Swap Postgres for an API or an in-memory fake and the domain/service code doesn't change.

## Dependency inversion in practice

```ts
// in the DOMAIN layer — an interface phrased in domain terms
interface OrderRepository {
  byId(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
  ordersForCustomer(id: CustomerId): Promise<Order[]>;
}

// in the INFRASTRUCTURE layer — the implementation, the only place the ORM appears
class SqlOrderRepository implements OrderRepository {
  async byId(id) {
    const row = await this.orm.orders.findUnique({ where: { id: id.value }, include: { lines: true }});
    return row ? OrderRecordMapper.toDomain(row) : null;   // ← map ORM row → domain aggregate
  }
  async save(order) {
    const record = OrderRecordMapper.toRecord(order);      // ← map domain → ORM row
    await this.orm.orders.upsert({ where: { id: record.id }, create: record, update: record });
  }
}
```

The service depends on `OrderRepository` (the interface); wiring injects `SqlOrderRepository`. Tests inject an in-memory implementation.

## Repository vs DAO (don't conflate them)

| | Repository | DAO |
|---|---|---|
| Granularity | Whole **aggregate** (Order + its lines) | Single table / row |
| Vocabulary | Domain terms (`ordersForCustomer`) | CRUD (`insert`, `selectById`) |
| Returns | Domain objects | Rows / records |
| Lives behind | A domain-owned interface | Data-access utility |

Prefer **repositories** at the domain boundary. A DAO can exist underneath as a thin table helper, but the service should talk to the repository in aggregate terms.

## The ORM-entity-as-domain-model trap

The most common persistence mistake: using ORM entities as your domain model.

```ts
// ❌ ORM entity doubles as the domain model
@Entity()
class Order {                          // annotations couple domain to the ORM
  @PrimaryColumn() id;
  @Column() status;                    // public, mutable — no invariants
  @OneToMany(...) lines;               // lazy proxies, persistence concerns everywhere
}
```

Why it bites: the domain now depends on the ORM (violates the dependency rule), invariants can't be enforced (the ORM needs a no-arg constructor and public fields), lazy-loading leaks into business code, and serializing it leaks DB structure to clients. Keep a **persistence model (record/entity) separate from the domain aggregate**, and map between them in the repository. (For tiny apps you may accept the coupling deliberately — but name the tradeoff.)

## What belongs in the repository — and what doesn't

| Belongs | Does NOT belong |
|---|---|
| Queries, ORM/SQL, transactions at the data level | Business rules / invariants (domain) |
| Mapping ORM rows ↔ domain aggregates | Use-case orchestration (service) |
| Hiding pagination/caching/storage details | HTTP/DTO concerns (edge) |
| Returning fully-formed aggregates | Returning half-loaded entities that break invariants |

```ts
// ❌ business logic creeping into the repository
async save(order) {
  if (order.total > customer.creditLimit) throw ...   // ← domain rule in the wrong place
}
// ✅ repository just persists; the domain already enforced the rule before save
```

## Queries beyond CRUD: read models

When a screen needs data spanning many aggregates, don't contort the domain to assemble it. Use a **read model / query service** that selects exactly what the view needs (often a projection or denormalized query) — separate from the write-side repositories. This is the read/write split at the heart of CQRS, used lightly:

```ts
// read side returns a flat view DTO, not domain aggregates
interface OrderSummaryQueries { recentForCustomer(id): Promise<OrderSummaryView[]>; }
```

## Quick Reference

| Do | Don't |
|---|---|
| Define the repo **interface in the domain** | Put the interface in infra (inverts the rule) |
| Phrase methods in domain terms, per aggregate | Expose generic `query(sql)` to the domain |
| Map ORM rows ↔ domain aggregates in the repo | Use ORM entities as the domain model |
| Return complete aggregates | Return partially-loaded objects |
| Use read models for cross-aggregate views | Force reporting queries through write repos |
| Inject an in-memory repo in tests | Hit a real DB for unit tests of services |

## Common Mistakes

- **ORM entity = domain model** — couples domain to persistence and kills invariants. Separate the two; map in the repo.
- **Repository interface in infrastructure** — the domain must own it (dependency inversion).
- **Business logic in the repository** — rules belong in the domain; repos only persist/fetch.
- **Generic `findAll`/`query` exposed to services** — leaks persistence thinking; expose intention-revealing, aggregate-shaped methods.
- **Returning leaky/lazy objects** — lazy proxies blow up later in business code; return materialized aggregates.
- **Forcing reports through write repositories** — use a dedicated read model/query service.

**REQUIRED COMPANION:** The service-layer depends on these interfaces; domain-model defines the aggregates being persisted (and owns the interface); dto-mapping handles the *outbound* (API) mapping while this skill handles the *storage* mapping. See clean-architecture.
