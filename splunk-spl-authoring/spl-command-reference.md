# SPL Command Reference

Reference for the most-used SPL commands, grouped by purpose. Splunk command types matter for performance: **streaming** commands run on indexers in parallel; **transforming** commands (`stats`, `chart`, `timechart`, `top`, `rare`) run on the search head and force data centralization; **centralized streaming** (`dedup`, `head`, `streamstats` in some modes) run on the search head in order. Push filtering before transforming commands.

## Filtering & Selecting

| Command | Purpose | Notes / Gotchas |
|---|---|---|
| `search` | Keyword + field match | Implicit as first command. Supports wildcards `*`, booleans `AND OR NOT`, comparisons on fields. |
| `where` | Eval-expression filter | Use functions: `like()`, `match()`, `isnull()`, `in()`. `where x="y"` compares field x to literal y. |
| `fields` | Keep/drop fields | `fields host status` keeps; `fields - _raw` drops. Do early to reduce payload. **Removing `_raw`/`_time` breaks some downstream commands.** |
| `table` | Select fields as columns | Like `fields` but defines display order; drops `_raw`. Use for final output, not mid-pipeline. |
| `dedup` | Remove duplicates | `dedup 1 session_id` keeps first. `sortby` controls which. Centralized — can be slow on huge sets. |
| `head` / `tail` | First / last N | `head 100`. `tail` reverses then takes N (expensive). |

## Aggregation & Reporting

| Command | Purpose | Notes |
|---|---|---|
| `stats` | Aggregate by groups | The workhorse. `count`, `dc()` (distinct count), `sum`, `avg`, `min`, `max`, `values()`, `list()`, `earliest()`, `latest()`, `perc95()`. Always alias with `as`. |
| `eventstats` | Aggregate but keep all events | Adds aggregate as a new field on every row (e.g., add `avg` next to each event). |
| `streamstats` | Running/cumulative aggregates | Order-dependent: running totals, deltas. `streamstats current=f window=5 ...`. |
| `chart` | 2-D matrix | `chart count over status by host` → rows × columns. |
| `timechart` | Aggregate over `_time` | `timechart span=1h count by host`. `span`, `bins`, `limit` (top N series). |
| `top` / `rare` | Most/least common | `top limit=10 url by host`. Returns `count` and `percent`. |
| `tstats` | Aggregate on tsidx / data models | Fastest aggregation. `| tstats count where index=x by host`. `prestats=t` to chain into `timechart`. Only indexed/data-model fields. |
| `timewrap` | Compare periods | `timechart ... | timewrap 1w` for week-over-week. |

`stats` aggregate functions cheat sheet: `count`, `count(field)` (non-null), `dc(field)`, `sum`, `avg`, `stdev`, `median`, `perc95(field)`, `values(field)` (unique, sorted, multivalue), `list(field)` (all, ordered), `first`/`last` (input order), `earliest`/`latest` (by time), `mode`, `range`.

## Field Extraction & Parsing

| Command | Purpose | Notes |
|---|---|---|
| `rex` | Regex extraction | `rex field=_raw "user=(?<user>\S+)"`. PCRE. `max_match=0` for all matches → multivalue. `mode=sed` to rewrite: `rex mode=sed "s/\d{4}/XXXX/g"`. |
| `spath` | Parse JSON/XML | `spath input=payload path=data.items{}.id`. Auto-runs on JSON if fields referenced. |
| `extract` / `kv` | Apply field extractions | `extract pairdelim="," kvdelim="="`. |
| `xmlkv` / `multikv` | Parse XML / tabular | `multikv` parses `top`-style command output. |
| `makemv` / `mvexpand` | Multivalue handling | `makemv delim="," tags` splits; `mvexpand tags` → one row per value (watch row explosion). |
| `eval` | Compute fields | See eval functions below. |

## Enrichment & Lookups

| Command | Purpose | Notes |
|---|---|---|
| `lookup` | Enrich from lookup table/KV | `lookup users uid OUTPUT name dept`. Automatic lookups configured in props.conf. |
| `inputlookup` | Read a lookup as events | `| inputlookup assets.csv`. Use `append=t` to union. Great as a subsearch allowlist. |
| `outputlookup` | Write results to a lookup | `| outputlookup known_hosts.csv`. Persist hunt/baseline state. |
| `join` | SQL-style join | **Avoid when possible** — subsearch limits (50k rows, autofinalize), slow. Prefer `stats by` or `lookup`. |
| `append` / `appendcols` | Union / side-by-side | `appendcols` zips columns positionally (fragile). `append` unions rows. |

## Correlation & Sequencing

| Command | Purpose | Notes |
|---|---|---|
| `transaction` | Group events into transactions | Memory-heavy, lossy at limits. Use only when you need `duration`/`eventcount` semantics or start/end conditions (`startswith`, `endswith`). Otherwise use `stats`. |
| `transpose` | Rows ↔ columns | `transpose 5` for small result reshaping. |
| `map` | Loop a search per row | Powerful but slow; avoid in production dashboards. |

## eval Functions (most used)

```
Conditional:  if(cond,a,b), case(c1,v1,c2,v2,...), coalesce(a,b,c), validate()
String:       len(), lower(), upper(), substr(), replace(), trim(), split(),
              mvjoin(), mvindex(), mvcount(), spath()
Match:        like(field,"foo%"), match(field,"regex"), searchmatch()
Math:         round(x,2), ceiling(), floor(), abs(), pow(), exp(), log()
Time:         now(), strftime(_time,"%F %T"), strptime(s,"%Y-%m-%d"),
              relative_time(now(),"-1d@d"), tonumber(), tostring(x,"duration")
Type/null:    isnull(), isnotnull(), isnum(), typeof(), nullif()
Network:      cidrmatch("10.0.0.0/8", ip)
```

`case()` has no default — add a trailing `1=1, "default"` to catch the rest. `coalesce()` returns the first non-null, ideal for unifying field name variants.

## Time Modifiers

```
earliest=-24h@h latest=now          ← snap to hour boundary
earliest=-7d@d  latest=@d           ← last 7 full days
earliest=@mon                       ← start of this month
[ ... | return earliest_time ]      ← dynamic from subsearch
```

`@` snaps to a unit boundary (`@d` = midnight, `@w0` = Sunday). Relative times: `s m h d w mon y`.

## Subsearches

Run first, return results to outer search. Format-dependent:
- In `search`: returns `(field=v1 OR field=v2 ...)` — capped at 50k rows / 60s by default.
- With `| inputlookup` or `[search ... | fields ip]`: produces a filter list.
- Use `| format` to control output; `| return` to emit a single value/field.

```spl
index=auth action=failure
  [ search index=threat_intel | fields src_ip ]   ← only IPs on the intel list
| stats count by src_ip
```

Keep subsearches small and fast — they block the outer search and silently truncate.
