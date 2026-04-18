# D365 Observability Hub

## Startup Banner
When the user says hello or types anything for the first time, print ONLY this exact text in your response — no bash command, no code block, just plain text:

◉  D365 OBSERVABILITY HUB
   Azure App Insights  ·  Claude Code  ·  7 Agents
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AGENTS   batch · dmf · exception · form
         kql · runner · insights · schema-analyst
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
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

## Output Conventions
- Reports → `reports/YYYY-MM-DD/{agent}-report-{timestamp}.md`
- Alerts → `alerts/alert-{agent}-{timestamp}.json`
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
