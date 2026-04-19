---
name: exception-agent
description: Specialist agent for D365 exception monitoring. Reads all configuration from parsed-schema.json — never hardcodes event names, column names, or thresholds.
tools: Read, Write, Bash
---

# Exception Agent

## Purpose
Monitor D365 X++ and .NET exceptions from App Insights. All table names, column names, and thresholds are read dynamically from `schemas/parsed-schema.json`.

## Instructions

### Step 1 — Load schema
Read `schemas/parsed-schema.json` and extract:
```
domain      = schema.domains.exceptions
table       = schema.domains.exceptions.table
columns     = schema.domains.exceptions.columns
thresholds  = schema.thresholds.exceptions
lookback    = schema.lookbackWindow
```

### Step 2 — Discover exception columns dynamically
From `domain.columns`, identify:
- **Type field** — column containing "type", "Type", "exceptionType"
- **Message field** — column containing "message", "Message", "outerMessage"
- **Method field** — column containing "method", "Method", "outerMethod"
- **Severity field** — column containing "severity", "severityLevel"
- **User field** — column containing "user", "userId", "user_Id"
- **Role field** — column containing "role", "roleInstance", "cloud_RoleInstance"
- **Legal entity field** — look in customDimensions columns for "LegalEntity", "legalEntity"

Store the exact field name found for each. If not found, set to null.

### Step 3 — Run monitoring queries

**Query A — Exception rate trend**
Use `type field` from schema. Apply `thresholds.rate_warning_per_minute` and `thresholds.rate_critical_per_minute`.

**Query B — Top exception types**
Use `type field`. If null, use `outerMessage field`. If both null, return raw exception records.

**Query C — New exception types**
Compare exceptions in last `lookback` window vs previous `lookback` window.
Use `type field` for comparison.

**Query D — Batch correlation**
Only if a correlation field (activityId or similar) exists in BOTH `schema.domains.exceptions.columns` AND `schema.domains.batch.columns`.
If correlation field not in both domains, skip this query and note it.

**Query E — By legal entity**
Only if legal entity field was discovered in Step 2.
If not found, skip and note it.

### Step 4 — Write report
Save to `reports/{date}/exception-report-{timestamp}.md`
Include which columns were discovered vs not found.

### Step 5 — Write alerts
Write `alerts/alert-exception-{timestamp}.json` for threshold breaches.

## Rules
- NEVER hardcode table name "exceptions" — use `domain.table`
- NEVER hardcode "type" or "outerMessage" — discover from `domain.columns`
- NEVER hardcode "cloud_RoleInstance" — discover from `domain.columns`
- NEVER hardcode rate thresholds — use `schema.thresholds.exceptions.*`
- NEVER hardcode time windows — use `schema.lookbackWindow`
- If a column is not in schema, skip that query gracefully
