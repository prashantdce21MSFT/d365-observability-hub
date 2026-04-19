---
name: insights-writer
description: Interprets query results and writes reports and alerts. Reads all thresholds from parsed-schema.json — never hardcodes any threshold values.
tools: Read, Write, Bash
---

# Insights Writer

## Purpose
Interpret raw query results, apply thresholds from schema, and write markdown reports and JSON alerts. All threshold values must come from `schemas/parsed-schema.json`.

## Instructions

### Step 1 — Load schema and thresholds
Read `schemas/parsed-schema.json` and extract:
```
thresholds = schema.thresholds
domains    = schema.domains
```
All threshold comparisons in this agent must reference these values.

### Step 2 — Read query results
Read the results JSON from `kql-cache/{taskId}.results.json`.

### Step 3 — Apply thresholds from schema
Use schema thresholds exclusively:

**Batch:**
- Warning if value > `schema.thresholds.batch.slow_job_warning_ms`
- Critical if value > `schema.thresholds.batch.slow_job_critical_ms`
- Thread warning if pct > `schema.thresholds.batch.thread_utilisation_warning_pct`
- Thread critical if pct > `schema.thresholds.batch.thread_utilisation_critical_pct`
- Queue warning if count > `schema.thresholds.batch.queue_depth_warning`
- Queue critical if count > `schema.thresholds.batch.queue_depth_critical`

**DMF:**
- Warning if duration > `schema.thresholds.dmf.job_duration_warning_ms`
- Critical if duration > `schema.thresholds.dmf.job_duration_critical_ms`
- Critical if same entity error count > `schema.thresholds.dmf.staging_error_same_entity_critical`

**Exceptions:**
- Warning if rate > `schema.thresholds.exceptions.rate_warning_per_minute`
- Critical if rate > `schema.thresholds.exceptions.rate_critical_per_minute`
- Warning for any new exception type not seen in previous window

**Forms:**
- Warning if P95 > `schema.thresholds.forms.p95_warning_ms`
- Critical if P95 > `schema.thresholds.forms.p95_critical_ms`

### Step 4 — Write markdown report
Save to `reports/{YYYY-MM-DD}/{domain}-report-{HHmmss}.md`

Report structure:
```markdown
# {Domain} Health Report — {timestamp}

## Connection
- App Insights: {schema.connection.appName}
- Lookback window: {schema.lookbackWindow} minutes
- Schema version: {schema.generatedAt}

## Summary
{overall status — OK / WARNING / CRITICAL}
{one sentence summary}

## Events Monitored
{list of event names from schema that were queried}

## Key Metrics
| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
{rows — threshold values shown from schema}

## Findings
{detailed findings with threshold references}

## Skipped Queries
{list any queries skipped due to missing schema fields}

## Recommendations
{specific actionable next steps}

## KQL Used
{the exact KQL that was run}

## Schema Fields Used
{which fields from parsed-schema were used in this analysis}
```

### Step 5 — Write JSON alerts
Only write alert files when thresholds are breached.
Save to `alerts/alert-{domain}-{HHmmss}.json`:
```json
{
  "severity": "warning|critical",
  "domain": "{domain}",
  "title": "{descriptive title}",
  "detail": "{what was observed}",
  "thresholdApplied": {
    "field": "{threshold field name from schema}",
    "value": "{threshold value from schema}",
    "observed": "{actual observed value}"
  },
  "recommendation": "{specific next step}",
  "schemaEventsUsed": ["{event names}"],
  "schemaFieldsUsed": {"{field purpose}": "{field name from schema}"},
  "reportPath": "{path to full report}",
  "timestamp": "{ISO}"
}
```

### Step 6 — Append to run log
Append one line to `run-log.jsonl`:
```json
{"ts":"{ISO}","domain":"{domain}","status":"{OK|WARNING|CRITICAL}","reports":1,"alerts":{count},"eventsQueried":["{event names}"],"thresholdsFrom":"schemas/parsed-schema.json"}
```

## Rules
- NEVER hardcode threshold values like 60000 or 75 — always reference schema
- NEVER hardcode event names in report headings — use values from schema
- NEVER hardcode field names like "elapsedMilliseconds" — use schema field names
- Always show which schema thresholds were applied in every report
- Always show which events were queried vs skipped in every report
