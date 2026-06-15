---
name: splunk-threat-hunting
description: Use when proactively hunting for threats in Splunk that existing alerts didn't catch — forming a hypothesis, exploring data for anomalies, looking for lateral movement, beaconing, living-off-the-land, persistence, or suspicious patterns, and turning findings into detections. Distinct from incident investigation (no known alert) and from writing detections (hunting discovers what to detect).
---

# Splunk Threat Hunting

## Overview

Threat hunting is **proactive, hypothesis-driven search for adversary activity that automated detections missed.** Unlike investigation (you have an alert/symptom) or detection engineering (you're codifying a known pattern), hunting starts from "if an attacker were doing X here, what would the data look like?" and goes looking.

Core principle: **a hunt is a falsifiable hypothesis plus a data plan.** "Look for bad stuff" is not a hunt. "Adversaries use scheduled tasks for persistence; find new scheduled tasks created by non-admin users on servers" is.

## The hunt loop

```
1. HYPOTHESIZE  Threat-informed: pick a TTP (MITRE ATT&CK) plausible for your environment
2. SCOPE        What data would show it? Do you even have that telemetry?
3. HUNT         Explore — baseline normal, surface the abnormal
4. TRIAGE       Investigate hits: benign, known, or genuinely suspicious?
5. OUTCOME      Finding → incident; pattern → new detection; gap → telemetry request
6. DOCUMENT     Hypothesis, data, queries, result — even "found nothing" is a result
```

Every hunt ends in one of four outcomes: an **incident** (escalate), a **detection** (codify it — see splunk-detection-engineering), a **visibility gap** (you lacked the data — request it), or a **documented negative** (hypothesis tested, nothing found — still valuable, prevents re-hunting).

## Hunting techniques (the analytical moves)

Detections use thresholds; hunts use **distribution and rarity**. You're looking for the outlier, the first-time, the thing that breaks the pattern.

### Stack counting (long tail = suspicious)

Aggregate and sort ascending — the rare values at the bottom are where adversaries hide:

```spl
index=endpoint EventCode=4688            ← process creation
| stats count, values(user) as users, dc(host) as hosts by parent_process, process_name
| sort + count                            ← rarest first
```

Common processes have huge counts; a `count=1` process spawned by `winword.exe` is worth a look.

### First-seen / new-in-environment

```spl
index=endpoint EventCode=4688 earliest=-30d
| stats earliest(_time) as first_seen, dc(host) as hosts by process_name
| where first_seen > relative_time(now(), "-1d")     ← appeared in last 24h
| convert ctime(first_seen)
```

### Beaconing (regular C2 callbacks)

Beacons phone home on an interval. Hunt for low-variance time deltas to a destination:

```spl
index=proxy OR index=firewall
| sort 0 src, dest, _time
| streamstats current=f last(_time) as prev by src, dest
| eval delta = _time - prev
| stats count, avg(delta) as avg_int, stdev(delta) as jitter,
        dc(bytes) as byte_variety by src, dest
| where count > 20 AND jitter < 30        ← very regular interval = likely beacon
| sort + jitter
```

Low `jitter` (consistent interval) + steady byte sizes + a long-lived connection is a classic beacon signature. Layer in rare/uncategorized destination domains.

### Living-off-the-land (LOLBins)

Legit binaries used maliciously — hunt by suspicious command-line, not by binary presence:

```spl
index=endpoint (process_name IN ("powershell.exe","certutil.exe","rundll32.exe","mshta.exe","regsvr32.exe"))
| search (CommandLine="*-enc*" OR CommandLine="*DownloadString*" OR CommandLine="*FromBase64*"
          OR CommandLine="*-urlcache*" OR CommandLine="*http*")
| stats count, values(CommandLine) as cmds by host, user, process_name
| sort - count
```

### Lateral movement (account/host fan-out)

```spl
| tstats summariesonly=t count from datamodel=Authentication
    where Authentication.action=success by Authentication.user, Authentication.dest
| rename Authentication.* as *
| stats dc(dest) as hosts_accessed, values(dest) as hosts by user
| where hosts_accessed > 10               ← one account touching many hosts fast
| sort - hosts_accessed
```

### Outlier by deviation (per-entity baseline)

```spl
index=proxy earliest=-30d
| bin _time span=1d
| stats sum(bytes_out) as daily_out by user, _time
| eventstats avg(daily_out) as avg, stdev(daily_out) as sd by user
| where daily_out > avg + 3*sd            ← user's own normal, exceeded
```

## Scoping: do you even have the data?

Before hunting a TTP, confirm the telemetry exists and is complete. A "clean" hunt result over data you don't actually collect is a false negative, not a win:

```spl
| tstats count where index=* by sourcetype, index    ← what am I actually collecting?
| sort - count
```

Map your hypothesis to required telemetry (e.g., persistence-via-scheduled-task needs Windows Event ID 4698 or Sysmon). If it's missing, the hunt outcome is a **visibility gap** — document and request the data source.

## From hunt to detection

When a hunt finds a repeatable pattern with acceptable fidelity, hand it to splunk-detection-engineering: baseline its fire rate, set a threshold/allowlist, annotate ATT&CK, and schedule it. **The goal of hunting is to put yourself out of a job for that TTP** — automate what you found so you can hunt the next thing.

## Quick Reference

| Hunting move | SPL signature | Finds |
|---|---|---|
| Stack count | `stats count by x | sort + count` | Rare values (long tail) |
| First-seen | `stats earliest(_time) by x | where first > -1d` | New entities |
| Beaconing | `streamstats last(_time) | eval delta | stats stdev(delta)` | Regular C2 intervals |
| LOLBin abuse | filter binaries + suspicious CommandLine | Living-off-the-land |
| Lateral movement | `stats dc(dest) by user | where >N` | Account fan-out |
| Deviation | `eventstats avg/stdev by entity | where >avg+3sd` | Per-entity anomaly |
| Data coverage | `tstats count by sourcetype, index` | Visibility gaps |

## Common Mistakes

| Mistake | Fix |
|---|---|
| "Hunt" with no hypothesis | State a falsifiable TTP-based hypothesis first |
| Hunting data you have, not threats that matter | Threat-model the environment; pick relevant ATT&CK TTPs |
| Static thresholds in a hunt | Hunting is about rarity/distribution, not fixed limits |
| Assuming empty result = safe | Confirm you have the telemetry; else it's a visibility gap |
| Finding a pattern, never codifying it | Promote good hunts to detections |
| Not documenting negatives | Record "tested, nothing found" — prevents re-hunting |

**REQUIRED COMPANION:** Use splunk-spl-authoring/optimization for the heavy exploratory searches, splunk-cim-normalization so hunts span all sources, and splunk-detection-engineering to operationalize findings.
