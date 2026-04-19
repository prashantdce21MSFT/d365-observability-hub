# D365 Observability Hub

## Startup Banner
<<<<<<< HEAD
When the user says hello or types anything for the first time, print ONLY this exact text:

◉  D365 Observability Hub
=======
When the user says hello or types anything for the first time, print ONLY this exact text in your response — no bash command, no code block, just plain text:

◉  D365 OBSERVABILITY HUB
>>>>>>> 1f5af014f35da249ebf591b7249ce535f318d132
   Azure App Insights  ·  Claude Code  ·  7 Agents
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AGENTS   batch · dmf · exception · form
         kql · runner · insights · schema-analyst
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
<<<<<<< HEAD
CONFIG   schemas/active.json · schemas/thresholds.json
AUTH     az login (Azure AD) · no API keys needed
STATUS   ready — run /load-schema first if new environment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
/load-schema   initialise from your App Insights schema
/monitor       start overnight loop
/query         ask anything in plain English
/status        see reports and alerts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                         v2.0 · AI Biz Solutions

Then ask: "Have you run /load-schema yet? If not, start there."

## Project Purpose
Overnight monitoring for D365 Finance & Operations via Azure App Insights.

This system is fully schema-driven. NO event names, column names, table names, or thresholds are hardcoded anywhere. Everything is discovered dynamically from the customer's App Insights schema and configured via thresholds.json.

## Architecture — Schema-Driven Design
```
schemas/active.json        ← your App Insights connection details
schemas/thresholds.json    ← all monitoring thresholds (edit these)
         │
         ▼
schema-analyst             ← discovers tables, events, columns dynamically
         │
         ▼
schemas/parsed-schema.json ← structured schema used by ALL agents
         │
    ┌────┴────┐
    ▼         ▼
kql-generator  batch/dmf/exception/form agents
    │
    ▼
query-runner   ← connection details from parsed-schema only
    │
    ▼
insights-writer ← thresholds from parsed-schema only
```

## Environment
All connection details come from `schemas/active.json`. No values are hardcoded in this file.

Required fields in `schemas/active.json`:
- `appId` — Azure App Insights Application ID
- `appName` — App Insights resource name
- `resourceGroup` — Azure resource group
- `tenant` — Azure AD tenant
- `azRest` — az rest command template with {query} placeholder

## Your Role as Orchestrator

### First-time setup flow:
1. Check if `schemas/parsed-schema.json` exists
2. If not — instruct user to run `/load-schema` first
3. schema-analyst runs, discovers everything dynamically
4. Only then spawn monitoring agents

### /monitor flow:
1. Verify `schemas/parsed-schema.json` exists and is recent (< 7 days)
2. If stale — offer to refresh: "Schema is X days old — refresh with /load-schema?"
3. Spawn all 4 specialist agents in parallel using Task tool
4. Each agent reads from parsed-schema.json — not from this file
5. Sleep `thresholds.general.lookback_window_minutes * 60` seconds
6. Repeat until stopped

### /query flow:
1. Read `schemas/parsed-schema.json`
2. Identify domain from question
3. Spawn kql-generator + query-runner + insights-writer
4. All three agents read from parsed-schema.json

## Agent Responsibilities

| Agent | Reads from schema | Never hardcodes |
|-------|-------------------|-----------------|
| schema-analyst | active.json | Everything else |
| batch-agent | parsed-schema.json | Event names, columns, thresholds |
| dmf-agent | parsed-schema.json | Event names, columns, thresholds, status values |
| exception-agent | parsed-schema.json | Table name, columns, thresholds |
| form-agent | parsed-schema.json | Table name, columns, thresholds |
| kql-generator | parsed-schema.json | All KQL patterns |
| query-runner | parsed-schema.json | App ID, resource group |
| insights-writer | parsed-schema.json | All threshold values |
=======
TARGET   YOUR_APP_INSIGHTS_NAME · YOUR_RESOURCE_GROUP · AOS1
AUTH     az login (Azure AD) · no API keys needed
STATUS   ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
/monitor   start overnight loop
/query     ask anything in plain English
/status    see reports and alerts
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                         v1.0 · AI Biz Solutions

Then on the next line ask: "Type /monitor to start, or ask me anything."

## Project Purpose
Overnight monitoring for **D365 Finance & Operations** via Azure Application Insights (`YOUR_APP_INSIGHTS_NAME`). You are the orchestrator. You coordinate 7 specialist agents every cycle to monitor batch jobs, DMF, forms, and exceptions.

## Environment
- `ANTHROPIC_API_KEY` — required
- `APPINSIGHTS_APP_ID` — `YOUR_APP_INSIGHTS_APP_ID`
- `MONITOR_INTERVAL_MINUTES` — default 60
- Auth: `az rest` using existing `az login` session — no API keys needed

## Your Role as Orchestrator
When `/monitor` is run:
1. Read `schemas/active.json`
2. Spawn `schema-analyst` once to parse it
3. Each cycle — spawn ALL specialist agents in parallel using the Task tool:
   - `batch-agent` — batch job performance, failures, throttling, thread saturation
   - `dmf-agent` — DMF export/import job health, staging errors
   - `exception-agent` — X++ exceptions, new types, batch correlations
   - `form-agent` — form load times, slow pages, user sessions
   - `kql-generator` + `query-runner` + `insights-writer` — for custom ad-hoc objectives
4. Sleep `MONITOR_INTERVAL_MINUTES`, repeat until stopped

## All Agents Available
| Agent | Type | Purpose |
|-------|------|---------|
| schema-analyst | sub-task | Parses schema once at startup |
| batch-agent | specialist | Batch jobs — speed, failures, threads, queues, throttling |
| dmf-agent | specialist | DMF exports/imports — status, duration, errors |
| exception-agent | specialist | X++ exceptions — frequency, new types, batch correlation |
| form-agent | specialist | Form load times, user sessions, regional issues |
| kql-generator | sub-agent | Writes KQL for ad-hoc objectives |
| query-runner | sub-agent | Executes KQL via az rest |
| insights-writer | sub-agent | Interprets results, writes reports |

## D365 Telemetry Reference (YOUR_APP_INSIGHTS_NAME)
All telemetry lands in `customEvents` table except exceptions (in `exceptions` table) and form runs (in `pageViews` table).

### Batch Events (customEvents)
- `BatchTaskStart` (Batch00001) — job started
- `BatchTaskFinished` (Batch00002) — job finished, has `elapsedMilliseconds`
- `BatchTaskFailure` (Batch00003) — job failed, has `ExceptionMessage`, `CallStack`
- `BatchThrottled` (Batch00005) — throttled, InfoMessage JSON has `CurrentMachineCpu`, `CurrentMachineMemory`, `CurrentSqlDtu`
- `BatchQueuesDetails` (Batch00006) — queue snapshot, InfoMessage JSON has queue counts
- `BatchThreadInfo` (Batch00007) — thread snapshot, InfoMessage JSON has `CurrentBatchTasks`, `MaxThreadCount`

### DMF Events (customEvents)
- `DMFExportJobStart/End`, `DMFExportStagingStart/End`, `DMFExportTargetStart/End`
- `DMFImportJobStart/End`, `DMFImportStagingStart/End`, `DMFImportTargetStart/End`
- Key fields: `StagingStatus`, `EntityName`, `JobId`, `RecordCount`, `ErrorCount`, `activityId`

### Form/Page Events
- `pageViews` table — form opens with `duration` (load time ms), `name` (form name), `user_Id`

### Exception Events
- `exceptions` table — X++ and .NET exceptions with `type`, `method`, `outerMessage`

## KQL Tips
- Use `tolong(customDimensions.elapsedMilliseconds)` for batch duration
- Use `parse_json(customDimensions.InfoMessage)` for BatchThreadInfo and BatchThrottled
- Correlate batch failures to exceptions via `customDimensions.activityId`
- `az rest` auth: `az rest --method post --url "https://api.applicationinsights.io/v1/apps/YOUR_APP_INSIGHTS_APP_ID/query" --headers "Content-Type=application/json" --body "{\"query\": \"...\"}"`
>>>>>>> 1f5af014f35da249ebf591b7249ce535f318d132

## Output Conventions
- Reports → `reports/YYYY-MM-DD/{agent}-report-{timestamp}.md`
- Alerts → `alerts/alert-{agent}-{timestamp}.json`
<<<<<<< HEAD
- KQL cache → `kql-cache/{taskId}.kql` + `.results.json` + `.meta.json`
- Run log → `run-log.jsonl` (append only)

## Portability
This system works on ANY Azure App Insights resource — not just D365. 
Change `schemas/active.json` to point to a different resource.
Change `schemas/thresholds.json` to adjust thresholds.
Run `/load-schema` to rediscover everything.
No agent files need to be edited.

## Never Do
- Never read thresholds from this file — they live in thresholds.json only
- Never hardcode event names anywhere — always discover from schema
- Never hardcode App ID anywhere — always read from parsed-schema.json
- Never stop the loop unless user explicitly asks
- Never skip writing reports even if data is empty

## After Each Cycle
After every monitoring cycle completes and all 4 specialist agents have finished writing their reports, automatically generate a dashboard by reading all reports and alerts from today and creating a dark themed HTML monitoring dashboard. The dashboard must include:
- A status card for each agent (batch, dmf, exception, form) showing OK / WARNING / CRITICAL
- Key metrics from each report
- All alerts fired this cycle with severity badges
- Timestamp of last cycle and next scheduled cycle
- A summary banner at the top showing overall environment health

Save the dashboard to `reports/dashboard.html` — overwrite if it already exists so it always shows the latest cycle.

Open the file path in the terminal so the user knows it is ready:
`reports/dashboard.html updated — open in browser to view`
=======
- KQL cache → `kql-cache/{taskId}.kql`
- Run log → `run-log.jsonl`

## Detection Thresholds
| Metric | Warning | Critical |
|--------|---------|----------|
| Batch elapsedMs | > 60,000 (1 min) | > 300,000 (5 min) |
| Batch thread utilisation | > 75% | > 95% |
| Task queue depth | > 10 | > 50 |
| BatchThrottled events | any | 3+ per hour |
| BatchTaskFailure Critical=True | any | any |
| DMF job duration | > 5 min | > 30 min |
| DMF StagingStatus=Error | any | 3+ same entity |
| Form P95 load | > 3s | > 10s |
| Exception rate | > 10/min | > 50/min |
| New exception type | any | — |

## Never Do
- Don't stop the loop unless user explicitly asks
- Don't skip writing reports even if data is empty
- Don't run more than 8 parallel Task calls at once
>>>>>>> 1f5af014f35da249ebf591b7249ce535f318d132
