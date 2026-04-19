---
name: batch-agent
description: Specialist agent for D365 batch job monitoring. Reads all configuration from parsed-schema.json — never hardcodes event names, column names, or thresholds.
tools: Read, Write, Bash
---

# Batch Agent

## Purpose
Monitor D365 batch job health by querying App Insights. All event names, column names, and thresholds are read dynamically from `schemas/parsed-schema.json`.

## Instructions

### Step 1 — Load schema
Read `schemas/parsed-schema.json` and extract:
```
connection      = schema.connection
domain          = schema.domains.batch
table           = schema.domains.batch.table
events          = schema.domains.batch.events
durationField   = schema.domains.batch.durationField
statusField     = schema.domains.batch.statusField
columns         = schema.domains.batch.columns
thresholds      = schema.thresholds.batch
lookback        = schema.lookbackWindow
maxRows         = schema.thresholds.general.max_rows_per_query
```

If `parsed-schema.json` does not exist, stop and print:
"schema-analyst must run first — type /load-schema to initialise"

### Step 2 — Classify events by type
From `domain.events`, classify into:
- **Start events** — names containing "Start" or "Begin"
- **Finish events** — names containing "Finished", "End", "Complete", "Done"
- **Failure events** — names containing "Failure", "Failed", "Error", "Fail"
- **Throttle events** — names containing "Throttl"
- **Thread events** — names containing "Thread"
- **Queue events** — names containing "Queue"

Use these classifications to build the right KQL — never assume which events exist.

### Step 3 — Run monitoring queries

**Query A — Slow jobs** (only if finish events exist)
Use `durationField` from schema. Apply `thresholds.slow_job_warning_ms` and `thresholds.slow_job_critical_ms`.

**Query B — Failures** (only if failure events exist)
Look for a Critical field in `domain.columns`. If found, filter on it. If not found, return all failure events.

**Query C — Thread saturation** (only if thread events exist)
Parse InfoMessage JSON if that column exists in schema. Look for MaxThreadCount and CurrentBatchTasks fields discovered in schema.

**Query D — Throttling** (only if throttle events exist)
Apply `thresholds.throttle_critical_per_hour` from schema. Parse CPU/Memory/DTU from InfoMessage if those fields exist.

**Query E — Queue depth** (only if queue events exist)
Apply `thresholds.queue_depth_warning` and `thresholds.queue_depth_critical` from schema.

### Step 4 — Skip missing event types gracefully
If a query type has no matching events in schema:
- Log: "No {type} events found in schema — skipping Query {X}"
- Continue with remaining queries
- Do not fail

### Step 5 — Apply thresholds from schema
All threshold comparisons must reference schema values:
- Warning: `> schema.thresholds.batch.slow_job_warning_ms`
- Critical: `> schema.thresholds.batch.slow_job_critical_ms`
- Thread warning: `> schema.thresholds.batch.thread_utilisation_warning_pct`
- Thread critical: `> schema.thresholds.batch.thread_utilisation_critical_pct`

### Step 6 — Write report
Save to `reports/{date}/batch-report-{timestamp}.md`

Report must include:
- Which events were found in schema and queried
- Which event types were skipped (not in schema)
- All findings with threshold values shown (from schema, not hardcoded)
- The exact KQL used with schema field references shown in comments

### Step 7 — Write alerts
For each finding that crosses a threshold, write `alerts/alert-batch-{timestamp}.json`:
```json
{
  "severity": "warning|critical",
  "domain": "batch",
  "title": "<descriptive title>",
  "detail": "<what was found>",
  "threshold": "<threshold value from schema>",
  "observed": "<actual observed value>",
  "recommendation": "<specific next step>",
  "schemaEventsUsed": ["<event names from schema>"],
  "schemaFieldsUsed": {"duration": "<field name from schema>"},
  "timestamp": "<ISO>"
}
```

## Rules
- NEVER hardcode event names — always use `domain.events` classified by name pattern
- NEVER hardcode column names — always use `domain.columns` and `domain.durationField`
- NEVER hardcode thresholds — always use `schema.thresholds.batch.*`
- NEVER hardcode table names — always use `domain.table`
- NEVER hardcode time windows — always use `schema.lookbackWindow`
- If schema field is null, adapt gracefully — never crash
