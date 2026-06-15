---
name: splunk-dashboards
description: Use when building or improving Splunk dashboards, reports, or visualizations — Dashboard Studio or Simple XML, choosing the right chart, adding inputs/filters/drilldowns, defining KPIs, making panels load fast with base searches or accelerated data models, or fixing dashboards that are slow, cluttered, or hard to read.
---

# Splunk Dashboards

## Overview

A dashboard answers a question at a glance and lets the viewer drill from "something's wrong" to "here's why." The two failure modes are **slow** (every panel reruns a heavy search) and **noisy** (20 panels, no hierarchy, nobody knows where to look). Both are design problems, not Splunk limitations.

Core principle: **a dashboard has one primary question.** Lead with the answer (the KPI/trend), support it with breakdowns, and make everything drillable. If you can't name the question, you're building a data dump.

## Dashboard Studio vs Simple XML

| | Dashboard Studio | Simple XML |
|---|---|---|
| Layout | Absolute/grid, pixel control | Row/panel flow |
| Look | Modern, themeable | Classic |
| Definition | JSON | XML |
| Best for | New dashboards, visual polish, custom layout | Legacy, heavy form logic, some advanced tokens |
| Use when | Default for anything new | Maintaining existing, or features Studio lacks |

Default to **Dashboard Studio** for new work unless you need a Simple-XML-only feature or are editing an existing XML dashboard.

## Design: lead with the answer

```
┌─────────────────────────────────────────────┐
│  KPI    KPI    KPI    KPI      ← single-value, the headline numbers
├─────────────────────────────────────────────┤
│  ███ primary trend over time ███             ← the main question
├──────────────────────┬──────────────────────┤
│  breakdown by dim     │  breakdown by dim    ← supporting context
├──────────────────────┴──────────────────────┤
│  detail table (drilldown target)             ← the "why", row-level
└─────────────────────────────────────────────┘
```

Top-left is the most valuable real estate — put the headline there. Reading order is top→bottom, left→right; arrange panels so the eye flows from summary to detail.

## Choose the right visualization

| Question | Visualization | SPL shape |
|---|---|---|
| One number now | Single value (+ trendline/sparkline) | `stats count` / `... | timechart count` |
| Trend over time | Line / area (`timechart`) | `timechart span=5m count by x` |
| Compare categories | Bar / column | `stats count by category | sort - count` |
| Part-to-whole | Pie (≤5 slices) or bar (prefer bar) | `top limit=5 x` |
| Distribution | Histogram / column | `... | bin field | stats count by field` |
| Two dimensions | Heatmap / `chart over by` | `chart count over hour by weekday` |
| Geographic | Choropleth / cluster map | `... | iplocation src | geostats count` |
| Row-level detail | Table (with sparklines/color) | `stats ... by entity` |

**Avoid pie charts with many slices** and **dual-axis line charts** — both are hard to read. When in doubt, a sorted bar chart or a table beats a clever visualization.

## Performance: the #1 dashboard problem

A dashboard with 10 panels = 10 searches firing on load. Make them cheap:

**1. Base searches + post-process (Simple XML) / base+chain (Studio).** Run the expensive search once; derive panels from it:

```xml
<search id="base">
  <query>index=web sourcetype=access_combined | fields status host response_time uri_path</query>
  <earliest>$tr.earliest$</earliest><latest>$tr.latest$</latest>
</search>
<!-- panels post-process the base, no re-scan -->
<chart><search base="base"><query>| stats count by status</query></search></chart>
<chart><search base="base"><query>| timechart count by host</query></search></chart>
```

Post-process must be **non-retrieving** (transforming/streaming only) — no new `index=` search.

**2. Use accelerated data models + `tstats`** for panels over large data (see splunk-cim-normalization):

```spl
| tstats summariesonly=t count from datamodel=Web by Web.status, _time span=5m
| rename Web.* as *
```

**3. Schedule reports / summary indexing** for panels that don't need to be real-time. Back the panel with a saved report run on a cron instead of a live search.

**4. Set sensible time defaults** — don't default a dashboard to "All time."

## Inputs, tokens, and drilldown

**Inputs** (dropdowns, time, text) set **tokens** that filter every panel:

```xml
<input type="time" token="tr"><default><earliest>-24h@h</earliest><latest>now</latest></default></input>
<input type="dropdown" token="env" searchWhenChanged="true">
  <label>Environment</label>
  <choice value="*">All</choice>
  <fieldForLabel>env</fieldForLabel><fieldForValue>env</fieldForValue>
  <search><query>| inputlookup environments | dedup env</query></search>
</input>
<!-- panels reference $env$ and $tr.earliest$ -->
```

**Drilldown** turns a static panel into an investigation tool — click a row to pass its value to another dashboard/search:

```xml
<drilldown>
  <link target="_blank">/app/search/host_detail?form.host=$click.value$&form.tr.earliest=$tr.earliest$</link>
</drilldown>
```

Guard tokens that may be unset with `<change>`/`eval` token logic or `default` values so panels don't error on first load.

## Defining KPIs

A good KPI is **a single number with a clear definition, a comparison, and a threshold.** "Errors: 412" is data; "Error rate: 0.8% (▲ vs 0.5% yesterday), threshold 1%" is a KPI.

```spl
index=web sourcetype=access_combined
| stats count as total, count(eval(status>=500)) as errors
| eval error_rate = round(errors/total*100, 2)
```

Add color ranges (green/amber/red) on single-value/gauge panels tied to the threshold. For SLA/health dashboards, ITSI formalizes this (KPIs, glass tables, health scores) — but most needs are met with single-value panels + thresholds.

## Quick Reference

| Goal | Approach |
|---|---|
| New dashboard | Dashboard Studio, lead with KPIs |
| Slow dashboard | Base search + post-process; `tstats`/accelerated DM; scheduled reports |
| Filter all panels | Inputs → tokens referenced in every search |
| Click-to-investigate | Drilldown passing `$click.value$` + time tokens |
| One headline number | Single value with trend + color threshold |
| Compare to baseline | `timewrap`, or KPI with delta vs prior period |
| Don't default to "All time" | Set a bounded default time input |

## Common Mistakes

| Mistake | Fix |
|---|---|
| Every panel its own heavy search | Base search + post-process / chained search |
| Live searches over huge data | Accelerated data model + `tstats summariesonly=t` |
| Pie chart with 12 slices | Sorted bar chart |
| No hierarchy — 20 equal panels | One question; KPIs top, detail bottom |
| Default time = All time | Bounded default (e.g., last 24h) |
| Static panels, no drilldown | Add drilldown to detail view |
| Token unset errors on load | Provide defaults / guard with `<change>` |
| Numbers without context | KPI = number + comparison + threshold |

**REQUIRED COMPANION:** Use splunk-spl-authoring for panel queries, splunk-spl-optimization and splunk-cim-normalization to make panels fast at scale, and splunk-log-investigation to design the drilldown paths analysts actually need.
