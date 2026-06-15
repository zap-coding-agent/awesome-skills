---
name: devops-engineer
description: Use when thinking about deployment pipelines, infrastructure-as-code, observability, reliability engineering, incident response, or the DevOps/SRE mindset — CI/CD design, toil reduction, SLOs, on-call, runbooks, blast radius reduction, and the philosophy of treating operations as a software engineering problem. Technology-agnostic.
---

# DevOps / SRE Engineer Mindset

## Core Philosophy

DevOps is not a role — it's the practice of **treating operations as a software engineering problem**: automatable, measurable, improvable. The three ways of DevOps (Gene Kim): flow (fast delivery), feedback (fast detection), continual learning.

SRE (Site Reliability Engineering, Google's operationalization of DevOps) adds one key principle: **toil has a budget.** Engineers should spend at most 50% of time on operational work; the rest must be engineering that permanently reduces future toil.

> "Hope is not a strategy." — SRE Book

## The Three Ways (Gene Kim's DevOps)

1. **Flow**: optimize the path from commit to production. Remove handoffs, automate gates, reduce batch size.
2. **Feedback**: shorten the loop from production back to development. Observe, alert, learn.
3. **Continual Learning**: post-mortems without blame; experiments; normalize failure as learning.

## CI/CD Pipeline Design

A pipeline is a specification of *how the code gets to production*. Design it to be:

- **Fast** — developers need feedback in <10 minutes. Parallelize tests; cache aggressively.
- **Opinionated** — every merge to main deploys to prod (or deploys to staging with a one-click promote). Manual gates introduce toil and delay.
- **Automated** — if a human must do something on every deploy, automate it or it will eventually be skipped.

Stages:
```
Commit → Build → Fast Tests (unit/lint) → Slow Tests (integration) → Artifact → Deploy Staging → Smoke Test → Deploy Prod → Canary Verification
```

**Deployment strategies:**
| Strategy | Blast radius | Rollback | Use when |
|---|---|---|---|
| Blue/Green | None (instant switch) | Instant | Stateless services |
| Canary | 1-5% of traffic | Remove canary | High traffic, stateful |
| Rolling | N pods at a time | Redeploy old | Kubernetes default |
| Feature flags | Per user/cohort | Toggle off | Long-running features |

## Infrastructure as Code (IaC)

**Principle:** the infrastructure definition is the source of truth. Clicking in a console is toil that produces an undocumented, un-reproducible state.

Good IaC:
- Is version-controlled (same review workflow as application code).
- Is idempotent: applying it twice = same result.
- Is modular: infrastructure is composed from reusable modules.
- Has tests: `terraform plan` / `pulumi preview` in CI before any apply.

```hcl
# Terraform example: the principle applies regardless of tool
module "web_service" {
  source       = "./modules/ecs_service"
  name         = "web"
  image        = var.image_tag         # ← parameterized, not hardcoded
  desired_count = 3
  min_healthy   = 50
}
```

Never manually modify production infrastructure. If you had to, immediately write the IaC that codifies the change.

## Observability: the Three Pillars

| Pillar | What it answers | Tool examples |
|---|---|---|
| **Logs** | What happened? | Splunk, ELK, CloudWatch Logs |
| **Metrics** | Is the system healthy? (aggregated numbers) | Prometheus, Datadog, CloudWatch Metrics |
| **Traces** | Where did this request spend time? | Jaeger, Zipkin, AWS X-Ray, Honeycomb |

**You are not observable if you can only answer "yes/no" to "is it working?"** Observable systems answer: *where* is it slow, *which* requests are failing, *what changed* at the time of the incident.

The four golden signals (Google SRE):
1. **Latency** — how long requests take (distinguish successful vs failing).
2. **Traffic** — how much demand the system handles.
3. **Errors** — rate of failed requests.
4. **Saturation** — how full the service is (CPU, memory, queue depth).

Alert on symptoms (user impact), not causes. "Error rate > 1%" is a user-impacting alert. "CPU > 80%" is an indicator, not a symptom — alert only if it predicts a user-impacting event.

## SLOs — The Contract with the Business

A **Service Level Objective (SLO)** is a target for a service level indicator (SLI) over a time window.

```
SLI: success rate = (successful requests) / (total requests)
SLO: success rate >= 99.9% over 30 days
Error budget: 0.1% * 30d * 24h * 60m = 43.2 minutes of downtime allowed

If error budget is being consumed fast → freeze non-critical deploys, focus on reliability.
If error budget is healthy → deploy features confidently; reliability is ahead of target.
```

Error budgets make the reliability vs velocity trade-off explicit and data-driven rather than a negotiation.

## Toil Reduction

Toil = manual, repetitive, automatable work with no lasting value. Examples: manually scaling a service, running a deployment script by hand, rotating credentials on a schedule.

**Toil reduction process:**
1. Identify: time-track operational work for one sprint.
2. Categorize: toil vs engineering work vs overhead.
3. Target: eliminate the single biggest toil source.
4. Measure: did the automation work? Did it actually reduce time?

If >50% of an SRE/DevOps engineer's time is toil, the system has a structural problem.

## Incident Response Mindset

During an incident: **minimize time-to-mitigate**, not time-to-root-cause. Mitigation = stopping the bleeding. Root cause follows.

```
1. DETECT    Alert fires / user report / monitoring.
2. RESPOND   Declare severity; assign IC (Incident Commander); open war room.
3. DIAGNOSE  Where is the user impact? What changed? (dashboards, logs, deploys)
4. MITIGATE  Rollback, failover, scale, disable feature — fastest path to user relief.
5. RESOLVE   Full fix deployed; monitoring confirms healthy.
6. POSTMORTEM Blameless analysis: timeline, contributing factors, action items with owners.
```

**The blameless postmortem:** the goal is to fix the *system*, not find the *person*. "Human error" is never a root cause — it's a signal that the system allowed or invited the error. Ask: what system design, process, or tooling would have prevented or caught this earlier?
