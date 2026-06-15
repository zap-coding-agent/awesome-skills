---
name: splunk-spl-optimization
description: Use when a Splunk search is slow, times out, hits subsearch/memory limits, scans too many events, or costs too much — or when reviewing SPL for performance before scheduling it as an alert/dashboard panel. Covers filtering early, tstats/data models, avoiding join/transaction, and search-cost diagnosis.
---

# Splunk SPL Optimization

## Overview

Splunk search cost is dominated by **how many events leave the indexers** and **how much work happens on the single search head**. Almost all optimization is one of two moves: (1) make the indexers return fewer events, and (2) avoid commands that centralize or re-scan data. 

Core principle: **the cheapest event is the one never read.** Filter on indexed fields and time before anything else.

## The Optimization Hierarchy (apply in order)

```
1. Narrow time range              ← biggest lever, almost free
2. Filter on indexed fields       ← index, sourcetype, source, host + keywords
3. Use tstats / accelerated DM    ← skip raw events entirely
4. Drop fields early (fields)     ← less data through the pipeline
5. Replace join/transaction       ← with stats / lookup
6. Move filters before transforms ← where after stats is too late
7. Limit subsearches              ← they block and truncate
```

## 1–2. Filter early on indexed fields

The first line of every search should prune at the indexer. Indexed fields (`index`, `sourcetype`, `source`, `host`) and raw keywords are evaluated before events are decompressed and parsed:

```spl
# ❌ Reads everything, filters late
index=* | search sourcetype=access_combined status=500 | stats count by host

# ✅ Indexer prunes by index+sourcetype+keyword first
index=web sourcetype=access_combined status=500 | stats count by host
```

Put literal keywords in the base search so they hit the index: `index=web error` is faster than `index=web | where like(_raw,"%error%")`.

## 3. tstats — the biggest win at scale

`tstats` runs against the `.tsidx` index files, never touching raw events. For counts/aggregations over indexed fields or accelerated data models it is **10–100× faster**:

```spl
# Slow: raw event scan
index=firewall action=blocked | timechart span=1h count by src_zone

# Fast: tsidx only
| tstats count where index=firewall action=blocked by _time span=1h, src_zone

# Against an accelerated data model
| tstats summariesonly=t count from datamodel=Network_Traffic
  where All_Traffic.action=blocked by All_Traffic.src_zone
```

`prestats=t` lets you pipe `tstats` into `timechart`/`chart`. `summariesonly=t` uses only accelerated summaries (fast, but misses un-summarized data — know the tradeoff).

## 4. Drop weight early

```spl
index=web sourcetype=access_combined status>=500
| fields host status response_time uri_path   ← carry only what you aggregate
| stats avg(response_time) as art by host, uri_path
```

`| fields` after the base search trims the per-event payload moving across the network and through each command.

## 5. Kill join and transaction

These are the two most common performance disasters:

```spl
# ❌ join: subsearch capped at 50k rows / 60s, silently truncates, slow
index=orders | join order_id [ search index=payments | fields order_id, status ]

# ✅ stats: single pass, complete, fast
index=orders OR index=payments
| stats values(status) as pay_status, values(amount) as amount,
        count by order_id

# ❌ transaction: memory-bound, drops events past maxopenevents
... | transaction session_id maxspan=30m

# ✅ stats reconstructs the session boundaries
... | stats earliest(_time) as start, latest(_time) as end,
            values(action) as actions, dc(action) as steps by session_id
| eval duration = end - start
```

Use `transaction` only when you genuinely need start/end conditions (`startswith`/`endswith`) or transaction-aware `duration`/`eventcount` that `stats` can't express.

## 6. Filter before you transform

`stats`/`chart`/`timechart` are transforming commands — they centralize data on the search head. Everything you can filter *before* them, you should:

```spl
# ❌ Aggregates all hosts, then throws most away
... | stats count by host | where host="web-07"

# ✅ Filter at the indexer
... host=web-07 | stats count
```

## 7. Tame subsearches

Subsearches run first and block the outer search; they autofinalize at 50k results / 60s and **truncate silently** — a subtle source of wrong answers, not just slowness:

```spl
# Risky: returns up to 50k IPs, may truncate
index=auth [ search index=threat_intel | fields src_ip ]

# Safer: large allow/deny lists belong in a lookup
index=auth | lookup threat_intel_ips src_ip OUTPUT threat_score
| where isnotnull(threat_score)
```

Adjust limits in `limits.conf` only as a last resort — usually the fix is restructuring with `lookup`/`stats`.

## Diagnosing a slow search

Use the **Job Inspector** (Job ▸ Inspect Job) on any run search. Read top-down:

| Field | Meaning | What it tells you |
|---|---|---|
| `eventCount` vs `scanCount` | events returned vs events read | Big gap = filtering too late |
| `command.search.index` time | time in tsidx lookup | High = broaden filter / use tstats |
| `command.search.rawdata` time | time decompressing raw events | High = reading too many raw events |
| `command.search.kv` time | field extraction cost | High = expensive search-time extractions |
| dispatch directory size | result volume | Large = returning too much |

`scanCount` ≫ `eventCount` is the classic "filter earlier" signal.

## Quick Reference

| Problem | Fix |
|---|---|
| Search times out | Narrow time; add indexed-field filter; switch to `tstats` |
| Subsearch truncated results | Replace with `lookup` or `inputlookup` |
| `join` slow / lossy | Rewrite as `stats ... by key` |
| `transaction` slow / drops events | Rewrite as `stats earliest/latest/values by key` |
| `scanCount` ≫ `eventCount` | Filter on indexed fields earlier |
| Dashboard panel slow | Accelerate a data model; use `tstats summariesonly=t`; base searches |
| Repeated expensive search | Schedule a report/summary index or accelerate it |

## Common Mistakes

- **`index=*`** with no other base filter — reads every index. Always name the index.
- **Wildcards at the start** (`*error`) — can't use the index; anchor them (`error*`).
- **`NOT`/negation as primary filter** — Splunk must read everything to exclude. Filter positively.
- **`where` for things `search` can do** — `where` runs after extraction; base `search` prunes at index time.

**REQUIRED COMPANION:** Use splunk-spl-authoring for correct query structure first — optimize only after the search is correct. For accelerating data models behind dashboards/detections, see splunk-cim-normalization.
