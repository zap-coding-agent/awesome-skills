---
name: clean-architecture
description: Use when deciding which layer code belongs in, reviewing whether a controller/service/domain/repository is doing too much, untangling a "fat controller" or "anemic/God service", or enforcing the dependency rule between API, application, domain, and persistence layers. Technology-agnostic; the glue skill for the layered-* suite.
---

# Layered Architecture Boundaries

## Overview

Layered (a.k.a. clean / onion / hexagonal) architecture splits an app into rings with **one rule that governs everything: dependencies point inward, toward the domain.** Outer layers know about inner layers; inner layers know nothing about outer ones. The domain depends on nothing.

Core principle: **the question "where does this code go?" is answered by "what does it depend on?"** If a piece of logic needs the web framework, it's outer. If it needs the database, it's outer. If it's a business rule true regardless of how data arrives or is stored, it's the domain.

## The layers and the dependency rule

```
        ┌─────────────────────────────────────┐
        │   API / Controller  (HTTP, CLI, MQ)  │  ← knows: framework, DTOs, service
        │   ┌─────────────────────────────────┐│
        │   │   Application / Service          ││  ← knows: domain, repo interfaces
        │   │   ┌─────────────────────────────┐││
        │   │   │   Domain (entities, rules)  │││  ← knows: NOTHING outward
        │   │   └─────────────────────────────┘││
        │   └─────────────────────────────────┘│
        │   Persistence / Infra (DB, APIs)      │  ← implements domain interfaces
        └─────────────────────────────────────┘
        Dependencies point INWARD ─────────────►
```

**Dependency inversion at the bottom:** persistence is an *outer* layer, but the domain/service must not depend on it. So the domain *defines a repository interface* and persistence *implements* it. The arrow still points inward.

## What belongs where (the cheat sheet)

| Layer | Owns | Must NOT contain | Skill |
|---|---|---|---|
| **API / Controller** | Protocol concerns: routing, deserialization, auth check, status codes, calling one service method | Business rules, DB access, domain objects in signatures | api-controller |
| **Service / Application** | Use-case orchestration, transaction boundary, calling domain + repositories, events | HTTP/framework types, business invariants (those are domain), SQL | service-layer |
| **Domain** | Entities, value objects, aggregates, invariants, domain logic & events | Framework, ORM annotations leaking behavior, DB/HTTP knowledge | domain-model |
| **DTO / Mapping** | Request/response shapes, mapping to/from domain | Behavior, persistence concerns | dto-mapping |
| **Persistence / Repository** | Implementing repository interfaces, ORM, queries | Business rules, leaking ORM entities outward as domain | persistence-repository |

## The one flow, across every layer

The whole suite uses one use case — **place an order** — so you can see each layer's slice:

```
HTTP POST /orders {items, customerId}
  → Controller: deserialize → PlaceOrderRequest, validate shape, call service     [api-controller]
  → Service: load Customer (repo), Order.place(...) , save (repo), publish event  [service-layer]
  → Domain: Order aggregate enforces "total ≤ creditLimit", computes totals        [domain-model]
  → Mapping: PlaceOrderRequest → command; Order → OrderResponse                    [dto-and-mapping]
  → Repository: OrderRepository.save() implemented over the ORM                    [persistence-repository]
  ← Controller returns 201 + OrderResponse
```

## The boundary smells (what reviews should catch)

| Smell | Violation | Fix |
|---|---|---|
| Fat controller | Business logic in the API layer | Move to service/domain; controller calls one method |
| Anemic domain | Entities are bags of getters; all logic in services | Push invariants/behavior into the domain |
| God service | One service does validation, rules, persistence, mapping | Split use-cases; rules → domain; mapping → mapper |
| Domain imports framework/ORM | Inner depends on outer | Invert: define interface in domain, implement outside |
| ORM entity returned from API | Persistence leaks to the edge | Map to a DTO at the boundary |
| Controller hits repository directly | Skips the application layer | Route through a service use-case |
| Service returns HTTP status / throws HTTP errors | Framework leaking inward | Throw domain/app exceptions; translate at the edge |

## Deciding where new code goes

```
Does it depend on HTTP/CLI/queue shape?          → API/Controller layer
Does it depend on the database/ORM/external API? → Persistence/Infra layer
Is it a rule true regardless of I/O?             → Domain layer
Does it coordinate domain + I/O for one use-case?→ Service/Application layer
Is it just data shape crossing a boundary?       → DTO
```

If something seems to fit two layers, it's usually doing two things — split it.

## Common Mistakes

- **Treating persistence as "inner."** It's outer; the domain owns the *interface*, infra owns the *implementation*.
- **Layer-per-folder without the dependency rule.** Folders named `controller/`, `service/` mean nothing if a service imports the web framework. The rule is about *dependencies*, not directory names.
- **Mapping everywhere.** Map only at boundaries (edge ↔ DTO ↔ domain ↔ persistence model). Mapping mid-logic is a smell.
- **Over-layering small apps.** For a tiny service, controller→domain may be enough. Add layers when the seams earn their keep, not by ritual.

**REQUIRED COMPANION:** This is the overview. For each layer's detailed responsibilities and examples, use api-controller, service-layer, domain-model, dto-mapping, and persistence-repository.
