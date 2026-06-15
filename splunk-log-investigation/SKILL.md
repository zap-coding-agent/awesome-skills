---
name: splunk-log-investigation
description: Use when investigating an incident, outage, error spike, latency regression, or anomaly using Splunk logs — going from a symptom ("500s started at 2pm", "checkout is slow", "users can't log in") to root cause. Covers scoping, correlating across sources, tracing a request, and building the timeline.
---

# Splunk Log Investigation

## Overview

Investigation is **funnel, then trace**: narrow from "something is wrong" to the exact failing events, then follow one bad transaction end-to-end across services. The discipline that separates fast investigations from flailing: **establish the timeline before forming theories, and confirm root cause with evidence before declaring it.**

Core principle: **find the change.** Most incidents are a deviation from a known-good baseline — a deploy, a config push, a traffic shift, a dependency failure. Your job is to locate the deviation in time and in the data.

## Investigation Workflow

```
1. SCOPE      What, where, when? Pin the blast radius and a precise time window.
2. BASELINE   Compare now vs a healthy period. What changed?
3. ISOLATE    Narrow to the failing slice (host/service/endpoint/user cohort).
4. TRACE      Follow one failing request/session across all relevant sources.
5. ROOT CAUSE Confirm the cause with evidence — not the first correlation.
6. TIMELINE   Assemble the sequence; hand off / document.
```

Do not skip to step 5. The most common failure mode is locking onto the first suspicious log line and rationalizing it as the cause.

## 1. Scope — pin the window

```spl
index=* (error OR fail* OR exception OR 5*) earliest=-2h
| timechart span=1m count by index
```

Find *when* the spike starts. Then snap your window tightly around it (`earliest=-1h@m latest=now`). A precise window makes every later search faster and clearer.

## 2. Baseline — what changed?

Compare the incident window against a healthy one. `timewrap` is built for this:

```spl
index=web sourcetype=access_combined status>=500
| timechart span=5m count
| timewrap 1d                      ← overlay yesterday same time
```

Also check for deploys/config changes in the same window — these are the usual culprits:

```spl
index=ci OR index=deploy earliest=-3h
| table _time, service, version, action, user
| sort _time
```

## 3. Isolate — narrow the failing slice

Break the error down by every dimension until one stands out:

```spl
index=web status>=500 earliest=-1h@m
| stats count by host, uri_path, status
| sort - count
```

If `host=web-07` carries 90% of errors → host problem. If one `uri_path` dominates → code/dependency problem. If errors are spread evenly → upstream/shared dependency. **The distribution of failures tells you what kind of problem it is.**

## 4. Trace — follow one transaction

Pick one failing identifier (request_id, trace_id, session_id, order_id) and follow it across every source. This is where you see the actual failure, not just its symptom:

```spl
index=* (request_id="abc-123")
| sort _time
| table _time, index, sourcetype, host, level, message
```

When there's no shared ID, correlate by time + key with `stats` (not `join`):

```spl
index=app OR index=db earliest=-15m
| stats values(error) as errors, values(query_ms) as latencies,
        earliest(_time) as start, latest(_time) as end by session_id
| where isnotnull(errors)
```

For latency regressions, look at the distribution, not the average — averages hide tail latency:

```spl
index=app sourcetype=apptrace earliest=-1h
| stats perc50(latency_ms) as p50, perc95(latency_ms) as p95,
        perc99(latency_ms) as p99, count by endpoint
| sort - p99
```

## 5. Root cause — confirm, don't assume

Before declaring a cause, it must explain:
- **Timing** — the cause appears at/just before the symptom onset.
- **Scope** — the cause covers the same blast radius (same hosts/endpoints/users).
- **Mechanism** — there's a plausible causal chain, ideally visible in the trace.

If a candidate cause fails any of these, keep looking. Correlation in one host ≠ cause if the symptom spans all hosts.

## 6. Timeline — assemble and hand off

```spl
index=* (request_id="abc-123" OR host="web-07" OR action="deploy")
  earliest=-2h@m
| sort _time
| table _time, source, host, action, level, message
```

Produce: onset time, trigger/change, mechanism, blast radius, and the evidence search for each claim.

## Quick Reference — investigation searches

| Need | Search shape |
|---|---|
| When did it start | `... | timechart span=1m count` |
| Now vs healthy | `... | timechart span=5m count | timewrap 1d` |
| Which slice is failing | `... | stats count by host, endpoint, status | sort - count` |
| Follow one request | `index=* request_id="X" | sort _time | table _time index message` |
| Latency tails | `... | stats perc95(t) perc99(t) by endpoint` |
| Rate of change | `... | timechart count | delta count as change` |
| New/never-seen value | `... | stats earliest(_time) as first by error | where first > relative_time(now(),"-1h")` |
| Correlate without shared ID | `... | stats values(*) by session_id` (not `join`) |

## Common Mistakes

| Mistake | Why it bites | Do instead |
|---|---|---|
| Forming a theory before the timeline | You search to confirm bias and miss the real onset | Pin the spike start first |
| Trusting averages for latency | Tail latency hides in the mean | Use `perc95`/`perc99` |
| Using `join` to correlate | Subsearch limits silently drop events | `stats ... by <key>` |
| Stopping at first error log | Symptom ≠ cause | Trace the full transaction |
| Time window too wide | Slow, noisy, ambiguous | Snap tightly around onset |
| Ignoring deploys/config | The change is usually the cause | Always check the change log for the window |

**REQUIRED COMPANION:** Use splunk-spl-authoring for query construction and splunk-spl-optimization when investigation searches are too slow over large windows. For a structured, root-cause-first debugging mindset that applies beyond Splunk, see superpowers:systematic-debugging.
