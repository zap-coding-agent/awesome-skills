---
name: fowler-microservices
description: Use when deciding whether to use microservices, how to decompose a monolith, setting service boundaries, designing inter-service communication, or when a microservices system is causing more pain than it's solving. Applies Martin Fowler's nuanced take — he coined the term but also wrote "MonolithFirst".
---

# Fowler — Microservices

## The Counterintuitive Opening

> "Don't start a new project with microservices, even if you're sure your application will be big enough to make it worthwhile." — Martin Fowler

Fowler co-defined "microservices architecture" in a landmark 2014 article with James Lewis — and then spent the next several years warning people against over-applying it. His view is deliberately nuanced: microservices solve real problems at scale and create new problems everywhere else.

## What Microservices Actually Are

The original definition:
> "An approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API."

Key properties from Fowler and Lewis's original characterization:
- Organized around **business capabilities**, not technical tiers (not "frontend service, backend service, db service")
- **Independently deployable** — a change to Service A doesn't require redeploying Service B
- **Decentralized data** — each service owns its own database; no shared schema
- **Smart endpoints, dumb pipes** — the logic is in the services, not the message bus (unlike SOA)
- Teams can be aligned to services — **Conway's Law** applied intentionally

## The Monolith First Argument (Fowler's Own Counterweight)

```
Microservices require you to get the service boundary right.
You don't know the right service boundary until you've run the system.
Therefore, start with a monolith; refactor to services as boundaries become clear.
```

The cost of a wrong service boundary: you've distributed a monolith. The coupling that made your monolith hard to maintain is now a network call, with all the distributed systems failure modes added on top (partial failure, latency, serialization).

**Fowler's rule:** if you wouldn't know how to decompose a monolith into coherent services after 6 months of building it, you don't know enough to decompose it into services from the start.

## Service Boundary Design — the Hard Part

How to find the right seam:

**Domain-Driven Design alignment** (the primary method): services map to Bounded Contexts — areas of the domain with their own language, models, and consistency requirements. The Order Context and the Inventory Context have different models for "product"; a service boundary between them is natural.

**The strangler fig pattern** — migrating a monolith:
```
1. Put the monolith behind a facade (reverse proxy or API gateway)
2. Identify one high-value, low-coupling capability to extract
3. Build the new service; route traffic to it via the facade
4. When the new service handles all traffic for that capability, remove it from the monolith
5. Repeat for the next capability
```
The monolith "dies" gradually as services strangle it. You never do a big-bang migration.

## Inter-Service Communication Patterns

| Pattern | When | Trade-offs |
|---|---|---|
| **Synchronous REST/gRPC** | Request needs a response now | Simple, but creates coupling — caller waits, failure propagates |
| **Async messaging (events)** | Caller doesn't need to wait | Decoupled, resilient, but harder to debug and trace |
| **Choreography** | Services react to events independently | Low coupling, high autonomy — but no central view of the flow |
| **Orchestration** | A coordinator drives the flow | Clear flow visibility — but coordinator becomes a coupling point |

The recurring principle: **prefer async messaging** where the use case allows it. Synchronous chains create cascading failure (if Service C fails, Service B fails, Service A fails). Async breaks the failure chain.

## The Distributed Systems Tax

Microservices are a distributed system. Every distributed systems problem applies:

```
Things that are simple in a monolith that become hard in microservices:
- Transactions: no ACID across services → Saga pattern (compensating transactions)
- Joins: can't join across databases → denormalize or accept round trips
- Consistency: two services updating related data → eventual consistency
- Debugging: a request that fails went through 5 services → distributed tracing required
- Testing: integration test requires all services running → service virtualization
- Latency: a local function call is microseconds; a service call is milliseconds
```

Fowler's assessment: microservices are a **productivity multiplier for large teams** and a **productivity drain for small teams**. The overhead of distributed systems engineering is fixed; the benefit scales with team size and deployment frequency.

## The "Microservices Premium" Threshold

Fowler's rough heuristic: microservices are worth the premium when:
1. **Multiple teams** need to deploy independently without coordination
2. **Components have different scaling requirements** (the checkout service needs 100× the capacity of the admin service)
3. **Components have different technology requirements** (ML service in Python, API in Go, frontend in Node)
4. **You've already built and understood the domain** (monolith first was respected)

Below this threshold, a **modular monolith** — clear module boundaries, separate deployable units, but not separate processes — gives most of the benefit without the distributed systems complexity.

**REQUIRED COMPANION:** fowler-enterprise-patterns covers the data access patterns (Repository, Data Mapper) that work within service boundaries. fowler-refactoring applies when extracting services from a monolith.
