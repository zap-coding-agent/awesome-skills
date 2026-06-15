---
name: 100k-kubernetes
description: Use when designing distributed system APIs, control planes, or systems that need eventual consistency — applying Kubernetes' declarative API design, controller/reconciliation loop pattern, operator pattern, or level-triggered vs edge-triggered design. Also when evaluating whether a system should be imperative or declarative.
---

# Kubernetes — API Design Philosophy

## The Governing Insight

> "Kubernetes is a platform for building platforms." — Kubernetes contributors

The architectural choices in Kubernetes weren't made for Kubernetes — they were made for building *any* distributed system that is reliable under failure. The declarative API, the controller loop, and the eventual consistency model are patterns applicable far beyond container orchestration.

## Declarative vs Imperative: the Core Decision

**Imperative:** tell the system *how* to achieve a goal.
```
kubectl run nginx --image=nginx     ← "do this action now"
```

**Declarative:** tell the system *what* the goal is; let it figure out how.
```yaml
# desired state stored in etcd
kind: Deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - image: nginx
```

The declarative API means: **the user expresses intent, not procedure.** The system is responsible for achieving and maintaining that intent. If a pod crashes, the system reconciles — not because someone issued a command, but because actual state diverged from desired state.

**Why this matters:** Declarative APIs are idempotent and self-healing. You can re-apply the same manifest a hundred times and get the same result. Imperative APIs require you to track what commands were issued and in what order — that state lives in your head, not the system.

## The Reconciliation Loop (Controller Pattern)

Every Kubernetes controller runs one loop:

```
loop forever:
  desired  = read from etcd (the spec)
  actual   = observe the real world
  if actual != desired:
    take the smallest action to move actual toward desired
```

This is **level-triggered** (respond to current state), not **edge-triggered** (respond to events/deltas). The difference:
- Edge-triggered: "do X when event Y fires" — miss the event, miss the action.
- Level-triggered: "ensure desired state is real, always" — tolerates missed events; every loop iteration is a full reconciliation.

**Why level-triggered wins for distributed systems:** Networks partition. Events get lost. Pods crash during a delta. A system that only reacts to deltas accumulates drift over time. A system that continuously reconciles to desired state is robust to all of these.

## API Design Principles (from the Kubernetes API Conventions)

### Resources, not Actions
Kubernetes APIs are CRUD on resources, not remote procedure calls. This keeps the API space small and consistent.

```
✅ PATCH /namespaces/default/deployments/web {"spec": {"replicas": 5}}
❌ POST /scale-deployment {"name": "web", "to": 5}
```

Action-based APIs multiply over time (scaleUp, scaleDown, scaleToZero, rollback...); resource-based APIs compose.

### Status vs Spec (the Two-Field Invariant)
Every Kubernetes resource has:
- `.spec` — desired state (set by the user / controller)
- `.status` — observed state (set by the controller, never by the user)

Controllers **never** write to `.spec` of their own resources. The contract: users own desired state; controllers own observed state. Merging these fields creates races and confusion.

### Defaulting and Validation at the API Server
Webhook defaulting and validation happen at admission time, before the object is stored. The stored object always has a canonical form. This means:
- Controllers read clean, defaulted objects — no defensive nil checks for fields that should have defaults.
- Validation errors surface to the user immediately, not after a failed reconciliation loop.

### The Operator Pattern
An **operator** is a controller that encodes the operational knowledge of a human operator into software. It extends the Kubernetes API with Custom Resource Definitions (CRDs) — new resource types — and controllers that reconcile them.

The pattern: domain-specific APIs (PostgresCluster, KafkaTopic) that express *what* the user wants; the operator handles *how* (provisioning, scaling, backups, failover). Users interact with Postgres concepts, not Kubernetes primitives.

## Seven Principles of Kubernetes API Design

1. **All reads from cache; all writes through the API server.** Never talk to etcd directly.
2. **Optimistic concurrency via resource version.** No distributed locks; retry on conflict.
3. **Idempotent operations.** Apply the same config a hundred times = same result.
4. **Eventual consistency is not a bug.** The system converges; it doesn't promise instantaneous synchrony.
5. **Fail open or fail closed deliberately.** Webhooks that fail should have explicit policies (ignore vs reject).
6. **Forward compatibility via strategic merge patch.** Adding optional fields doesn't break existing clients.
7. **API groups and versioning.** `apps/v1`, `batch/v1`; stability guarantees per version.

## What to Take Into Your Systems

| Kubernetes pattern | When to apply it |
|---|---|
| Declarative desired state | Any system that must self-heal |
| Reconciliation loop (level-triggered) | Any async work that must eventually be done |
| Spec/Status separation | Any system with user-desired vs system-observed state |
| Operator pattern | Domain-specific automation with a rich lifecycle |
| Resource-not-action APIs | Any API that will evolve over time |
| Idempotent mutations | Any distributed operation that may be retried |

**REQUIRED COMPANION:** For building controllers/operators in Go, pair with the Go patterns in the language suite. For distributed system debugging when reconciliation diverges, see splunk-log-investigation patterns applied to Kubernetes event logs.
