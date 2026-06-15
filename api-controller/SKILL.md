---
name: api-controller
description: Use when writing or reviewing an API/controller/handler layer (REST, GraphQL resolver, gRPC, CLI command, message consumer) — request deserialization, input validation, auth checks, status codes, error mapping, and keeping controllers thin. Catches "fat controllers" with business logic or direct DB access. Technology-agnostic.
---

# API / Controller Layer

## Overview

The controller is the **translator at the edge**: it converts a protocol-specific request (HTTP, gRPC, CLI args, a queue message) into a call to one application use-case, and converts the result back into a protocol response. That's it.

Core principle: **a controller has no business logic and no persistence. If you deleted the web framework, every business rule should still exist elsewhere.** A good controller method is mostly a single line of real work surrounded by translation.

## What the controller owns

1. **Deserialize** the request into a DTO / input object.
2. **Validate shape** — required fields, types, formats, ranges (not business rules).
3. **AuthN/AuthZ check** — is this caller allowed to invoke this use-case? (delegate the *decision* where it's non-trivial).
4. **Call exactly one** application/service method.
5. **Map the result** to a response DTO and **choose the status code**.
6. **Translate errors** from domain/app exceptions to protocol responses.

Everything else belongs deeper.

## The shape of a thin controller

```ts
// POST /orders
async function placeOrder(req, res) {
  const dto = PlaceOrderRequest.from(req.body);   // 1. deserialize
  dto.validate();                                 // 2. shape validation
  requireScope(req.user, "orders:write");         // 3. authz

  const order = await orderService.place(          // 4. ONE use-case call
    dto.toCommand(req.user.id)
  );

  return res.status(201).json(OrderResponse.from(order)); // 5. map + status
}
// 6. error translation handled by a shared error middleware (see below)
```

Compare to a **fat controller** (the thing to refactor away):

```ts
// ❌ business logic + persistence in the controller
async function placeOrder(req, res) {
  const customer = await db.query("SELECT * FROM customers WHERE id=?", [req.body.customerId]);
  let total = 0;
  for (const i of req.body.items) total += i.price * i.qty;   // ← domain rule
  if (total > customer.credit_limit) return res.status(400).json({error: "over limit"}); // ← domain rule
  await db.query("INSERT INTO orders ...");                   // ← persistence
  // ...
}
```

The fat version can't be reused by a CLI, a queue consumer, or a test without a web request — because the logic is trapped in the controller.

## Shape validation vs business validation

| Shape (controller) | Business (domain/service) |
|---|---|
| `email` is a valid email format | this email isn't already registered |
| `quantity` is a positive integer | enough stock to fulfill `quantity` |
| `items` array is non-empty | customer's credit covers the order total |

Controllers reject malformed requests (`400`). The domain rejects requests that are well-formed but break a rule — surfaced as an app/domain exception the controller translates.

## Status codes & error translation

Map *categories* of failure once, centrally — don't sprinkle status codes through business code:

| Outcome | Status |
|---|---|
| Created | 201 |
| Success, no body | 204 |
| Validation/shape error | 400 |
| Unauthenticated / unauthorized | 401 / 403 |
| Domain rule violation (e.g., over credit) | 409 or 422 |
| Not found | 404 |
| Unexpected | 500 (don't leak internals) |

```ts
// shared error middleware — the ONLY place that knows HTTP status codes
function toHttp(err) {
  if (err instanceof ValidationError)  return { status: 400, body: err.details };
  if (err instanceof NotFoundError)    return { status: 404, body: {error: err.message} };
  if (err instanceof DomainRuleError)  return { status: 422, body: {error: err.message} };
  return { status: 500, body: {error: "internal error"} };   // never leak stack/SQL
}
```

The service throws `DomainRuleError("credit limit exceeded")`; it does **not** know that maps to 422. That keeps the framework out of the inner layers.

## Quick Reference

| Do | Don't |
|---|---|
| Deserialize → validate shape → call one service method → map response | Put loops of business logic in the handler |
| Translate exceptions to status codes centrally | Return `res` from service code |
| Return DTOs | Return ORM/domain entities directly |
| Keep methods short and uniform | Access the database/repository directly |
| Pass a command/DTO inward | Pass `req`/`res`/framework objects inward |

## Common Mistakes

- **Business rules in the handler** — move to domain/service; the controller should not compute totals or check limits.
- **Direct DB/repository access** — route through a service; controllers don't know persistence exists.
- **Returning entities** — leaks internal model + persistence fields; map to a response DTO (see dto-mapping).
- **Status codes scattered in business code** — centralize error→status translation.
- **Passing `req`/`res` deeper** — the framework must not leak past the controller; pass plain commands/DTOs.

**REQUIRED COMPANION:** The controller calls the service-layer and uses dto-mapping for request/response shapes. See clean-architecture for the dependency rule.
