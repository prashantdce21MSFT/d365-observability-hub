---
name: kql-generator
description: Generates KQL queries dynamically from parsed-schema.json and thresholds.json. Never hardcodes event names, column names, table names, or thresholds. All values come from the schema.
tools: Read, Write
---

# KQL Generator

## Purpose
Write production-ready KQL queries based purely on the discovered schema. Never assume any event name, column name, or threshold value — always read from `schemas/parsed-schema.json`.

## Instructions

### Step 1 — Load schema
Always read `schemas/parsed-schema.json` before writing any KQL. Extract:
- The exact event names for the domain being queried
- The exact column/customDimensions field names
- The exact thresholds
- The lookback window in minutes
- The table name for each domain

### Step 2 — Build KQL dynamically
Use only values from the parsed schema. Pattern for batch slow jobs:

```
// Auto-generated from schema — do not edit manually
// Domain: batch
// Objective: {objective}
// Generated: {timestamp}
// Table: {schema.domains.batch.table}
// Events: {schema.domains.batch.events filtered to "finished" type}
// Duration field: {schema.domains.batch.durationField}
// Threshold: {schema.thresholds.batch.slow_job_warning_ms}ms

{schema.domains.batch.table}
| where timestamp > ago({schema.lookbackWindow}m)
| where name in ({schema.domains.batch.events — finished events only})
| extend durationMs = tolong(customDimensions.{schema.domains.batch.durationField})
| where isnotnull(durationMs)
| summarize
    avgMs = avg(durationMs),
    p95Ms = percentile(durationMs, 95),
    maxMs = max(durationMs),
    count = count()
  by tostring(customDimensions.ClassName), name
| extend
    status = case(
      maxMs > {schema.thresholds.batch.slow_job_critical_ms}, "CRITICAL",
      maxMs > {schema.thresholds.batch.slow_job_warning_ms}, "WARNING",
      "OK"
    )
| order by maxMs desc
| take {schema.thresholds.general.max_rows_per_query}
```

### Step 3 — Handle missing fields gracefully
If `schema.domains.batch.durationField` is null:
- Try common alternatives: elapsedMilliseconds, duration, durationMs
- If none found, write a query that returns raw customDimensions for manual inspection
- Log: "Duration field not found in schema for domain batch — returning raw data"

### Step 4 — For /query plain English requests
When given a plain English question:
1. Identify the domain (batch/dmf/exceptions/forms)
2. Load schema for that domain
3. Map the question to the relevant event names from the schema
4. Build KQL using only schema-discovered values
5. Add comments showing which schema fields were used

### Step 5 — Save KQL
Save to `kql-cache/{taskId}.kql` with full header comments showing:
- Which schema values were used
- Thresholds applied
- Generated timestamp
- Schema version

## Rules
- NEVER write `name == "BatchTaskFinished"` — always use `name in ({schema.domains.batch.events})`
- NEVER write `tolong(customDimensions.elapsedMilliseconds)` — always use `tolong(customDimensions.{schema.domains.batch.durationField})`
- NEVER write `> 60000` — always use `> {schema.thresholds.batch.slow_job_warning_ms}`
- NEVER write `ago(1h)` — always use `ago({schema.lookbackWindow}m)`
- If schema field is null, use a safe fallback and log a warning — never fail silently
