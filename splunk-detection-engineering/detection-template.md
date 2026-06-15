# Correlation Search Template

A reusable skeleton for a Splunk Enterprise Security correlation search (or a plain scheduled alert). Copy, fill the bracketed parts, and tune. Keep the documentation block — future-you and the SOC need it.

## Detection documentation block (keep with the search)

```
Name:        [Tactic] - [Behavior] - Rule
Hypothesis:  What attacker behavior does this detect, and why does it stand out?
Data source: [sourcetypes / CIM data model], required fields: [...]
ATT&CK:      [Txxxx.xxx], tactic: [...]
Threshold:   [static N / rarity / N-sigma] — rationale
Known FPs:   [legit behaviors that look similar] → handled by [allowlist lookup]
Response:    What does the analyst do when this fires? (link runbook)
Owner / last reviewed: [name] / [date]
```

## SPL skeleton

```spl
| tstats summariesonly=t count
    from datamodel=[Model]
    where [Model].action=[...]                        ← filter at the model
    by [Model].user, [Model].src, [Model].dest, _time span=[5m]
| rename [Model].* as *

``` then shape, allowlist, threshold, enrich, and emit:

```spl
| stats [aggregations] by [entity fields]

| lookup [approved_entities].csv [key] OUTPUT is_approved   ← allowlist known-good
| where isnull(is_approved)

| where [threshold condition]                                ← the decision

| lookup asset_lookup dest OUTPUT category, priority         ← enrichment
| lookup identity_lookup user OUTPUT bunit, watchlist

| eval risk_score=[N], risk_object=[user|dest|src],
       risk_object_type="[user|system]",
       risk_message="[human-readable, includes key values]"

| eval mitre_technique="[Txxxx]"
| table _time, risk_object, risk_score, mitre_technique, [evidence fields], risk_message
```

## savedsearches.conf settings

```ini
[ [Tactic] - [Behavior] - Rule ]
search = <the SPL above>
description = <the hypothesis>
disabled = 0
enableSched = 1
cron_schedule = */10 * * * *
dispatch.earliest_time = -65m@m
dispatch.latest_time = -5m@m                 ; lag for ingest delay; whole buckets only
dispatch.ttl = 2p

# Throttling — one alert per entity per window
alert.suppress = 1
alert.suppress.fields = risk_object
alert.suppress.period = 3600s

# Enterprise Security correlation search wiring
action.correlationsearch.enabled = 1
action.correlationsearch.label = [Tactic] - [Behavior] - Rule
action.correlationsearch.annotations = {"mitre_attack":["Txxxx"],"kill_chain_phases":["[phase]"]}

# Adaptive responses (choose what fits your model)
action.notable = 1                            ; create a notable event
action.notable.param.severity = high
action.risk = 1                               ; write to Risk data model for RBA
action.risk.param._risk = [{"risk_object_field":"risk_object","risk_object_type":"user","risk_score":40}]
```

## Pre-deploy checklist

- [ ] Ran bare logic over 30 days — fire count is sane (not hundreds/day)
- [ ] Reviewed the actual events it fired on — are they true positives?
- [ ] Known-good behaviors allowlisted via lookup (not inline `NOT`)
- [ ] Threshold justified (why this number / this rarity window)
- [ ] Throttling set so a campaign won't flood the SOC
- [ ] Window lags ingest delay; only whole buckets
- [ ] Enriched with asset + identity context
- [ ] ATT&CK technique annotated
- [ ] Response runbook written and linked
- [ ] Owner + review date recorded

## Tuning loop (after deploy)

```spl
# How often is each detection firing, and is it noisy?
index=notable earliest=-7d
| stats count, dc(risk_object) as entities by source
| sort - count
| eval per_day = round(count/7,1)
```

Detections with high `count` and low `entities` are firing repeatedly on the same thing → tighten allowlist or threshold. Revisit any detection that hasn't been reviewed in 90 days; environments drift and so does fidelity.
