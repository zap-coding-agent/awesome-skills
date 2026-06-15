---
name: splunk-spl-authoring
description: Use when writing, fixing, or reviewing Splunk SPL searches — building a query from scratch, extracting fields, aggregating with stats/timechart/tstats, parsing raw logs, or when a search returns no/wrong results or runs slowly. Covers the search pipeline, command order, and idiomatic SPL.
---

# Splunk SPL Authoring

## Overview

SPL (Search Processing Language) is a pipeline: each `|` passes events to the next command. **The single most important habit: filter early, transform late.** Every search should narrow the dataset (index, sourcetype, time, terms) before any `stats`, `eval`, or `transaction`. Get this wrong and searches are slow, wrong, or both.

Core principle: **a good SPL search reads as `WHERE → SHAPE → AGGREGATE → ORDER`.**

## The Search Pipeline (canonical order)

```spl
index=web sourcetype=access_combined status>=500   ← 1. base filter (indexed fields)
| search uri_path="/api/*"                          ← 2. refine (extracted fields)
| eval is_checkout=if(like(uri_path,"/api/checkout%"),1,0)  ← 3. shape
| stats count, avg(response_time) as avg_rt by status, host ← 4. aggregate
| where count > 10                                  ← 5. post-aggregate filter
| sort - count                                      ← 6. order
| head 20                                           ← 7. limit
```

Rule of thumb for **what goes in the base search vs `| search`:** indexed fields (`index`, `sourcetype`, `source`, `host`) and raw keywords belong in the first line so the indexer prunes data before it reaches the search head. Search-time extracted fields can also go there, but conceptually they're a refinement.

## Quick Reference

| Goal | Command | Example |
|---|---|---|
| Count / aggregate | `stats` | `stats count by status` |
| Aggregate over time | `timechart` | `timechart span=5m count by host` |
| Fast aggregation on indexed/accelerated data | `tstats` | `tstats count where index=web by host` |
| Compute / derive a field | `eval` | `eval dur_s=duration/1000` |
| Filter after aggregation | `where` | `where count>100` |
| Extract fields at search time | `rex` | `rex "user=(?<user>\w+)"` |
| Pull structured fields | `spath` / `extract` | `spath input=_raw path=user.id` |
| Correlate events into sessions | `transaction` / `stats` | prefer `stats` (see below) |
| Enrich from a file/KV store | `lookup` | `lookup geo ip OUTPUT country` |
| Top / rare values | `top` / `rare` | `top 10 src_ip` |
| Dedupe | `dedup` | `dedup session_id` |

For the full command catalog with arguments and gotchas, see [spl-command-reference.md](spl-command-reference.md).

## Idioms That Matter

**Prefer `stats` over `transaction`.** `transaction` is memory-heavy and silently drops events past limits. For "group events by a key," `stats` is faster and complete:

```spl
# ❌ Expensive, lossy
... | transaction session_id maxspan=30m

# ✅ Same intent, scalable
... | stats earliest(_time) as start, latest(_time) as end,
            values(action) as actions, count by session_id
| eval duration=end-start
```

**Use `tstats` for scale.** When you only need indexed fields or are querying a data model, `tstats` runs against the tsidx files and is 10–100× faster:

```spl
| tstats count where index=firewall by _time span=1h, action
```

**Name everything with `as`.** `stats avg(rt) as avg_rt` — unnamed aggregates produce ugly field names (`avg(rt)`) that break downstream commands.

**`fields` early to drop weight.** After your base search, `| fields host status rt` keeps only what you need through the pipeline.

## Common Mistakes

| Symptom | Cause | Fix |
|---|---|---|
| No results | Wrong time range, or filtering on a field that isn't extracted yet | Widen time; verify field exists with `| table _raw` first |
| Field is empty after `stats` | Aggregated a field you didn't carry; or case mismatch | Field names are case-sensitive; values are not (unless quoted) |
| Slow search | Aggregating before filtering, or `transaction` | Filter in base search; switch to `stats`/`tstats` |
| `rex` matches nothing | Greedy/anchored regex, or escaping | Test on `_raw`; use named groups `(?<name>...)`; PCRE syntax |
| Counts look doubled | Joining or multivalue expansion | Use `stats ... by` instead of `join` where possible |
| `where field="x"` returns nothing | `where` compares fields; bare string is a field name | Quote the literal: `where field="x"` is correct; `where x` means field x |

**`search` vs `where`:** `search`/base uses keyword+wildcard matching (`status=5*`); `where` uses eval expressions and functions (`where like(uri,"/api/%")`, `where status>=500`). Don't mix mental models.

## Authoring Workflow

1. **Start at the base filter.** `index= sourcetype=` + time range. Confirm you see raw events.
2. **Verify fields exist.** `| table _time host status` (or `| fieldsummary`). Extract missing ones with `rex`/`spath`.
3. **Add the transform.** `stats`/`timechart`/`tstats` with named outputs.
4. **Refine and order.** `where`, `sort`, `head`.
5. **Optimize last.** Only after it's correct — see splunk-spl-optimization.

**REQUIRED COMPANION:** For performance-sensitive searches, use splunk-spl-optimization. For normalizing fields to standard names before aggregation, use splunk-cim-normalization.
