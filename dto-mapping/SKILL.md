---
name: dto-mapping
description: Use when designing DTOs / request-response models / commands, or mapping between them and domain objects — at the API edge or any boundary. Catches leaking ORM/domain entities to clients, over-posting/mass-assignment, behavior creeping into DTOs, and mapping logic scattered through controllers/services. Technology-agnostic.
---

# DTO & Mapping Layer

## Overview

A DTO (Data Transfer Object) is a **flat, behavior-free data shape that crosses a boundary** — the wire format for a request, response, or message. Mapping is the translation between DTOs and domain objects. The reason this exists: **your internal model and your external contract must be free to change independently.** Bind them together and every refactor breaks clients (or worse, leaks internals).

Core principle: **never expose a domain entity or ORM model directly across a boundary, and never let an inbound DTO mutate your domain directly.** Always translate.

## Why DTOs exist (the failure they prevent)

```ts
// ❌ returning the domain/ORM entity directly
return res.json(order);   // leaks: internal field names, password hashes, ORM lazy proxies,
                          // computed-only fields, future fields you didn't mean to expose
```

Problems: you leak internal/sensitive fields, you couple the API contract to the schema (rename a column → break clients), serialization triggers lazy-loading surprises, and you can't version the API independently.

```ts
// ✅ explicit response contract
class OrderResponse {
  static from(order: Order): OrderResponse {
    return {
      id: order.id.value,
      status: order.status,
      total: order.total.toString(),       // VO → string, controlled format
      lines: order.lines.map(l => ({ sku: l.sku, qty: l.qty })),
      // password/credit fields simply never appear
    };
  }
}
```

## The three DTO roles

| DTO | Direction | Purpose | Maps to/from |
|---|---|---|---|
| **Request DTO** | inbound | What the client may send | → Command / domain input |
| **Command** | inbound | A validated intent the service acts on | built from Request DTO |
| **Response DTO** | outbound | The public contract for results | ← domain object |

Keeping Request DTO and Command separate lets you accept a loose wire shape but pass a precise, internal intent inward.

## Inbound: avoid over-posting / mass assignment

Never bind the raw request straight onto a domain object or entity — clients can set fields they shouldn't (`isAdmin`, `status`, `price`):

```ts
// ❌ mass assignment: client controls every field
const order = new Order(req.body);

// ✅ accept only the allowed fields, then build a precise command
const dto = PlaceOrderRequest.from(req.body);   // only customerId + lines exist on it
const cmd = dto.toCommand(authenticatedUserId); // server sets identity, not the client
```

The DTO is an **allowlist** of what the outside may influence. Server-controlled values (ids, timestamps, the authenticated user, status) are set internally, never accepted from the client.

## Mapping — keep it at the boundary, keep it explicit

```ts
// Mapping lives in a dedicated mapper / static factory — not sprinkled in controllers
class OrderMapper {
  static toResponse(o: Order): OrderResponse { ... }
  static toCommand(dto: PlaceOrderRequest, userId: string): PlaceOrderCommand { ... }
}
```

Guidance:
- **Map at edges only**: edge ↔ DTO, DTO ↔ domain, domain ↔ persistence model. Not in the middle of business logic.
- **Explicit over magic**: hand-written or schema-checked mapping is debuggable; deep auto-mappers hide leaks and break silently on rename. If you use an auto-mapper, assert the mapping in a test.
- **One direction per method**: `toResponse`, `toCommand` — don't build bidirectional mega-mappers.
- **DTOs are dumb**: no business logic, no persistence, no calling services. Data + trivial shaping only.

## DTOs vs domain vs persistence model — three distinct shapes

```
PlaceOrderRequest  (API contract, loose, public)
      │ map
PlaceOrderCommand  (internal intent, validated)
      │ →
Order              (domain: behavior + invariants)
      │ map
OrderRecord/Entity (persistence model: columns, ORM annotations)
```

These often look similar early on — resist merging them. The moment the API needs a field the DB doesn't have (or vice versa), separate shapes save you. (Persistence mapping detail: persistence-repository.)

## Quick Reference

| Do | Don't |
|---|---|
| Define explicit Request/Response DTOs | Return domain/ORM entities over the wire |
| Build a Command from the request DTO | Bind `req.body` onto a domain object (mass assignment) |
| Set server-controlled fields internally | Trust client-supplied ids/status/price |
| Map in a dedicated mapper at the boundary | Scatter mapping through controllers/services |
| Keep DTOs behavior-free | Put logic/validation rules in DTOs |
| Version response DTOs for API evolution | Couple the contract to the schema |

## Common Mistakes

- **Leaking entities** — exposes internals/secrets and couples contract to schema. Always map to a Response DTO.
- **Mass assignment** — binding raw input to domain/entity lets clients set protected fields. DTO = allowlist.
- **One model for everything** (API = domain = table) — convenient until any layer needs to change alone; then it's a cascade.
- **Mapping scattered everywhere** — hard to audit what's exposed. Centralize in mappers.
- **Smart DTOs** — DTOs that validate business rules or call services. Keep them inert; rules live in the domain.
- **Blind auto-mapping** — silently exposes new fields on rename. Prefer explicit; test the mapping if auto.

**REQUIRED COMPANION:** DTOs cross the api-controller boundary, become commands for the service-layer, and expose domain-model data. Persistence-side mapping is persistence-repository. See clean-architecture.
