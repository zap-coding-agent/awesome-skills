---
name: splunk-cim-normalization
description: Use when normalizing Splunk fields to standard names so searches, dashboards, and detections work across data sources — mapping vendor fields to CIM (Common Information Model), building/validating data models, tagging events, or when a correlation search or Enterprise Security panel returns nothing because fields don't match. Covers CIM data models, field aliases, tags, eventtypes, and acceleration.
---

# Splunk CIM Normalization

## Overview

Different sources call the same thing different names: `src_ip`, `sourceIP`, `clientip`, `cs_ip` all mean "source address." The **Common Information Model (CIM)** is Splunk's shared schema that normalizes these to canonical fields (`src`, `dest`, `user`, `action`, …) so one search works across every vendor. Splunk Enterprise Security, prebuilt dashboards, and most detections are built **on top of CIM data models** — if your data isn't CIM-compliant, they show nothing.

Core principle: **normalize once at search time (via knowledge objects), query the standard names forever.** Don't bake vendor field names into 50 searches.

## How normalization is layered

```
Raw event
  → field EXTRACTION        (rex / props.conf → get clientip out of _raw)
  → field ALIAS             (clientip → src)            FIELDALIAS in props.conf
  → calculated field/EVAL   (derive action from status) EVAL- in props.conf
  → LOOKUP                  (enrich: src → src_category)
  → TAG / EVENTTYPE         (mark events as authentication, network, etc.)
  → DATA MODEL              (CIM model maps tagged events to its fields)
  → ACCELERATION            (tstats-ready summaries)
```

You make data CIM-compliant by ensuring its events (a) have the model's required fields under canonical names, and (b) carry the tags the model constrains on.

## The CIM data models you'll use most

| Data model | Covers | Key fields |
|---|---|---|
| `Authentication` | Logins, auth success/failure | `action` (success/failure), `user`, `src`, `dest`, `app` |
| `Network_Traffic` | Firewall, flow, proxy L3/L4 | `action`, `src`, `dest`, `src_port`, `dest_port`, `bytes`, `transport` |
| `Web` | Proxy / web server access | `action`, `src`, `dest`, `url`, `http_method`, `status`, `user` |
| `Endpoint` (Processes/Filesystem/Registry/Ports) | EDR/Sysmon | `process_name`, `process`, `parent_process_name`, `dest`, `user` |
| `Malware` | AV/EDR detections | `signature`, `action`, `dest`, `file_name`, `user` |
| `Intrusion_Detection` | IDS/IPS | `signature`, `severity`, `src`, `dest`, `ids_type` |
| `Change` | Config/account changes | `action`, `object`, `object_category`, `user`, `status` |

Field reference per model: Splunk docs "CIM data model reference tables." Required vs recommended fields are listed there — required fields must be present for the model to populate.

## Making a source CIM-compliant

**1. Alias vendor fields to CIM names** (props.conf on the search head):

```ini
[my_firewall]
FIELDALIAS-cim_src = source_address AS src
FIELDALIAS-cim_dest = dest_address AS dest
FIELDALIAS-cim_ports = source_port AS src_port dest_port AS dest_port
EVAL-action = case(verdict=="ALLOW","allowed", verdict=="DENY","blocked", 1==1, "unknown")
EVAL-bytes = bytes_in + bytes_out
```

**2. Tag events into the model.** The Network_Traffic model constrains on `tag=network tag=communicate`. Create an eventtype and tag it (eventtypes.conf + tags.conf):

```ini
# eventtypes.conf
[my_firewall_traffic]
search = sourcetype=my_firewall

# tags.conf
[eventtype=my_firewall_traffic]
network = enabled
communicate = enabled
```

**3. Validate** the model now populates:

```spl
| tstats summariesonly=f count from datamodel=Network_Traffic
  where nodename=All_Traffic by All_Traffic.action, sourcetype
```

If your sourcetype appears with sensible `action` values, it's compliant. Empty = missing tags or unaliased required fields.

## Querying via the data model (the payoff)

Once compliant, every detection/dashboard is source-agnostic and fast:

```spl
# Failed logins across ALL auth sources — no vendor field names anywhere
| tstats summariesonly=t count from datamodel=Authentication
  where Authentication.action=failure by Authentication.src, Authentication.user, _time span=1h
| rename Authentication.* as *
```

`datamodel()` for raw access, `from datamodel=X` with `tstats` for accelerated speed. **Always `rename <Model>.* as *`** so downstream commands see plain field names.

## Acceleration

Accelerate models that back dashboards/detections (Settings ▸ Data Models ▸ Edit Acceleration). This builds summaries that `tstats summariesonly=t` reads. Tradeoffs:

- `summariesonly=t` — fast, but only summarized (older/un-summarized data missing until backfill completes).
- `summariesonly=f` — complete, but falls back to raw search (slower).
- Acceleration consumes disk and indexer CPU; accelerate only models you actually query.

## Quick Reference

| Task | Where | Example |
|---|---|---|
| Rename a field to CIM | props.conf `FIELDALIAS` | `FIELDALIAS-x = clientip AS src` |
| Derive a CIM field | props.conf `EVAL-` | `EVAL-action = if(code<400,"allowed","blocked")` |
| Tag into a model | eventtypes.conf + tags.conf | `network=enabled` |
| Test model population | `tstats ... from datamodel=` | `summariesonly=f` to see all |
| Query accelerated model | `tstats summariesonly=t from datamodel=` | + `rename Model.* as *` |
| List a model's fields | Splunk docs CIM reference | required vs recommended |

## Common Mistakes

| Symptom | Cause | Fix |
|---|---|---|
| ES dashboard / detection empty | Source not CIM-tagged | Add eventtype + tags for the model |
| `tstats from datamodel` returns nothing | Required field missing or not aliased | Alias to canonical name; check required fields |
| Fields prefixed `Authentication.` downstream | Forgot to rename | `| rename Authentication.* as *` |
| Counts low vs raw search | `summariesonly=t` before backfill done | Use `summariesonly=f` or wait for acceleration |
| `action` values inconsistent | Vendor verdicts not normalized | `EVAL-action` mapping to CIM-allowed values |

**REQUIRED COMPANION:** Use splunk-spl-authoring for `tstats`/`rename` syntax. CIM compliance is a prerequisite for splunk-detection-engineering and the cross-source panels in splunk-dashboards.
