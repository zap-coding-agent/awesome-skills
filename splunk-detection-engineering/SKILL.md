---
name: splunk-detection-engineering
description: Use when building, tuning, or reviewing Splunk security detections — correlation searches, scheduled alerts, Enterprise Security notable events, risk-based alerting (RBA), or mapping detections to MITRE ATT&CK. Also when a detection is too noisy, misses true positives, or you need to reduce false positives. Covers detection logic, thresholds, throttling, and the detection lifecycle.
---

# Splunk Detection Engineering

## Overview

A detection is a saved search that turns events into a security signal. The hard part isn't the SPL — it's **fidelity**: high enough to catch real threats, low enough that analysts trust the alerts. A detection that fires 200 times a day gets muted, and a muted detection catches nothing.

Core principle: **every detection is a hypothesis about attacker behavior, expressed as a search, with an explicit response.** If you can't say what an analyst does when it fires, don't ship it.

## Anatomy of a detection

```
WHAT      The behavior (hypothesis) — "service account logging in interactively"
LOGIC     The SPL that finds it (built on CIM where possible)
THRESHOLD When it's suspicious (count, rate, rarity, deviation)
CONTEXT   Enrichment so the analyst can triage without pivoting
RESPONSE  Notable/risk/alert + what the analyst should do
TUNING    Allowlists, throttling, suppression of known-good
ATT&CK    Technique mapping for coverage tracking
```

## The detection lifecycle

```
1. HYPOTHESIZE  Pick a behavior worth detecting (threat-informed, not data-informed)
2. SCOPE DATA   Which sources/CIM model carry the evidence? Is coverage complete?
3. WRITE LOGIC  SPL over the data model; prefer tstats for scale
4. BASELINE     Run over historical data — how often does it fire? on what?
5. TUNE         Allowlist known-good, set threshold, add throttling
6. ENRICH       Add asset/identity/intel context to the result
7. DEPLOY       As notable / RBA / alert with a documented response
8. MEASURE      Track TP/FP rate; revisit. Detections decay.
```

**Baseline before deploy (step 4) is non-negotiable.** Run the bare logic over 30 days first — the result count tells you whether you have a detection or a noise generator.

## Writing detection logic

Prefer CIM data models so the detection spans every source and runs fast (see splunk-cim-normalization):

```spl
# Brute force → success: many failures then a success for the same user/src
| tstats summariesonly=t count from datamodel=Authentication
    where Authentication.action IN ("success","failure")
    by Authentication.user, Authentication.src, Authentication.action, _time span=5m
| rename Authentication.* as *
| stats sum(eval(if(action=="failure",count,0))) as failures,
        sum(eval(if(action=="success",count,0))) as successes by user, src
| where failures >= 10 AND successes >= 1
```

### Threshold strategies (pick the one that fits the behavior)

| Strategy | When | SPL pattern |
|---|---|---|
| Static count | Known-bad volume | `where count > 10` |
| Rate / velocity | Bursts | `bin _time span=1m | stats count by ... | where count>N` |
| Rarity / first-seen | New = suspicious | `stats earliest(_time) as first by process | where first > relative_time(now(),"-1d")` |
| Statistical deviation | Per-entity normal varies | `eventstats avg(c) stdev(c) | where c > avg+3*stdev` |
| Allowlist absence | Anything not approved | `| search NOT [|inputlookup approved | fields process]` |

Rarity and deviation detections outperform static thresholds for most behaviors — attackers don't know your threshold, but they do stand out from a baseline.

## Tuning — the work that makes detections usable

**Allowlist known-good with lookups, not inline `NOT`** (maintainable, auditable):

```spl
... | lookup approved_service_accounts user OUTPUT is_approved
| where isnull(is_approved)
```

**Throttle / suppress** so one campaign doesn't create 500 notables. In a saved search / ES correlation search set:
- **Throttle** (`alert.suppress`) by fields like `user`, `src` for a window (e.g., 1h) — one alert per entity per window.
- **Schedule window** with realtime-backfill or `cron` aligned to data latency.

```ini
# savedsearches.conf
[Brute Force - Auth]
search = <detection SPL>
dispatch.earliest_time = -65m@m
dispatch.latest_time = -5m@m          ← lag for ingest delay; avoid partial buckets
cron_schedule = */10 * * * *
alert.suppress = 1
alert.suppress.fields = user, src
alert.suppress.period = 3600s
action.correlationsearch.enabled = 1
action.notable = 1
```

**Avoid partial buckets:** end the window a few minutes in the past (`latest=-5m@m`) so late-arriving events don't cause misses or duplicate fires.

## Risk-Based Alerting (RBA)

Instead of one notable per detection, **attribute risk to entities** and alert when risk accumulates. This collapses dozens of low-fidelity signals into one high-fidelity alert:

```spl
# Each detection writes a risk event via the Risk Analysis adaptive response action,
# or directly:
... | eval risk_score=40, risk_object=user, risk_object_type="user",
        risk_message="Service account interactive logon"
| collect index=risk
```

Then a single RBA correlation search fires when an entity crosses a threshold across multiple techniques:

```spl
| tstats summariesonly=t sum(All_Risk.calculated_risk_score) as risk,
        dc(All_Risk.annotations.mitre_attack.mitre_tactic_id) as tactics,
        values(All_Risk.annotations.mitre_attack.mitre_technique_id) as techniques
  from datamodel=Risk by All_Risk.risk_object, All_Risk.risk_object_type
| rename All_Risk.* as *
| where risk >= 100 AND tactics >= 3      ← crossed multiple kill-chain phases
```

RBA's power: a service account that triggers recon + lateral movement + exfil signals becomes one alert at the entity level, even if no single signal was alert-worthy.

## MITRE ATT&CK mapping

Annotate every detection with its technique so coverage is measurable and RBA can count distinct tactics:

```ini
# in the correlation search / savedsearches.conf
action.correlationsearch.annotations = {"mitre_attack": ["T1110.001"], "kill_chain_phases": ["exploitation"]}
```

Track coverage as a gap analysis: which ATT&CK techniques relevant to your environment have *no* detection? Prioritize those, not another variant of one you already cover.

## Quick Reference

| Need | Approach |
|---|---|
| Detect known-bad volume | Static threshold on CIM model |
| Detect novelty | First-seen / rarity (`earliest by entity`) |
| Detect per-entity anomaly | `eventstats avg/stdev` + N-sigma |
| Reduce duplicate alerts | `alert.suppress` by entity for a window |
| Reduce false positives | Allowlist lookup + `where isnull()` |
| Avoid missed/dup at window edges | `latest=-5m@m`, full buckets only |
| Combine weak signals | Risk-based alerting on `Risk` data model |
| Track coverage | MITRE ATT&CK annotations + gap analysis |

For a copy-paste correlation search skeleton with all the conf settings, see [detection-template.md](detection-template.md).

## Common Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| Deploy without historical baseline | Floods SOC, gets muted | Run logic over 30d first; count fires |
| Static threshold for variable behavior | Misses low-and-slow / floods | Use rarity or deviation |
| Inline `NOT host=...` allowlists | Unmaintainable, undocumented | Lookup + `where isnull()` |
| No throttling | One campaign = 100s of notables | `alert.suppress` by entity |
| Window includes partial bucket | Misses or duplicates | Lag the window end |
| Detecting what data you have, not what matters | Coverage theater | Threat-informed hypotheses + ATT&CK gaps |
| No documented response | Analyst can't act, alert ignored | Every detection ships with a runbook |

**REQUIRED COMPANION:** Detections should run on CIM data models — use splunk-cim-normalization first. Use splunk-spl-optimization to keep scheduled searches cheap, and splunk-threat-hunting to discover new detection hypotheses.
