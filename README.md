<div align="center">

# ◉ D365 Observability Hub

![D365 Observability Hub](docs/D365-Observability-Hub.gif)

### Overnight AI Monitoring for D365 Finance & Operations

[![Claude Code](https://img.shields.io/badge/Powered%20by-Claude%20Code-7C3AED?style=flat-square)](https://claude.ai/code)
[![Azure App Insights](https://img.shields.io/badge/Azure-App%20Insights-0078D4?style=flat-square&logo=microsoftazure)](https://azure.microsoft.com/en-us/products/monitor)
[![Version](https://img.shields.io/badge/Version-2.0-7C3AED?style=flat-square)](https://github.com/prashantdce21MSFT/d365-observability-hub/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![D365 FO](https://img.shields.io/badge/D365-Finance%20%26%20Operations-0078D4?style=flat-square)](https://dynamics.microsoft.com)

**You close your laptop. 7 AI agents wake up.**
**By the time your morning coffee is ready — the reports are already there.**

[What's New in v2.0](#whats-new-in-v20) · [Quick Start](#quick-start) · [Architecture](#architecture) · [Agents](#the-7-agents) · [Commands](#commands) · [Outputs](#outputs)

</div>

---

## What Is This?

D365 Observability Hub is a **Claude Code native** overnight monitoring system for Dynamics 365 Finance & Operations. It uses AI agent architecture to automatically query your Azure App Insights, detect performance issues, and write reports — while you sleep.

> **No server. No Node.js app. No API keys. No hardcoding. Just markdown files and Claude Code.**

### What It Monitors

| Domain | Source Table | What It Detects |
|--------|-------------|-----------------|
| **Batch Jobs** | customEvents | Slow jobs, failures, thread saturation, CPU/DTU throttling |
| **DMF** | customEvents | Failed exports, staging errors, slow jobs, aborted runs |
| **Exceptions** | exceptions | X++ errors, new exception types, batch correlations |
| **Forms** | pageViews | Slow form loads, P95/P99, active sessions, regional issues |

> All table names, event names, column names and field mappings are **discovered dynamically from your App Insights** — nothing is hardcoded.

---

## What's New in v2.0

### v1.0 was hardcoded

```
batch-agent.md had:
  name == "BatchTaskFinished"                        ← hardcoded event name
  tolong(customDimensions.elapsedMilliseconds)       ← hardcoded field
  > 60000                                            ← hardcoded threshold
  ago(1h)                                            ← hardcoded time window
  App ID: 9d57480a-...                               ← hardcoded in every agent
```

### v2.0 is fully schema-driven

```
batch-agent.md now reads:
  name in (schema.domains.batch.events)                          ← discovered at runtime
  tolong(customDimensions.{schema.domains.batch.durationField})  ← from schema
  > schema.thresholds.batch.slow_job_warning_ms                  ← from thresholds.json
  ago({schema.lookbackWindow}m)                                  ← from thresholds.json
  App ID from schemas/active.json only                           ← never in agent files
```

### Key improvements

| Feature | v1.0 | v2.0 |
|---------|------|------|
| Event names | Hardcoded in agent files | Discovered live from App Insights |
| Column names | Hardcoded in agent files | Discovered from customDimensions |
| Thresholds | Hardcoded in CLAUDE.md | `schemas/thresholds.json` — auto-generated |
| App ID | Hardcoded in every agent | `schemas/active.json` only |
| Status values | Hardcoded ("Finished", "Error") | Discovered dynamically |
| Portability | D365 FO only | Any Azure App Insights resource |
| Dashboard | Manual only | Auto-generated after every cycle |
| schema-analyst | Read pre-defined schema | Queries App Insights live at startup |

---

## Architecture

### How the schema flows

```
schemas/active.json        ← YOU fill this in (App ID + tenant only)
         │
         ▼
/load-schema → schema-analyst runs
         │
         ├── Queries App Insights live  → discovers tables
         ├── Queries each table         → discovers event names
         ├── Queries each event         → discovers customDimensions fields
         ├── Reads or auto-generates    → schemas/thresholds.json
         └── Saves everything to       → schemas/parsed-schema.json
                    │
                    ▼
         parsed-schema.json  ← contains BOTH:
                               · thresholds (from thresholds.json)
                               · discovered schema (from live queries)
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
   All 7 agents          query-runner
   read from here        reads App ID
   — no hardcoding       from here only
```

### How /monitor works every cycle

```
/monitor starts
    │
    ▼
Reads schemas/parsed-schema.json
    │
    ▼
4 specialist agents spawn in parallel
├── batch-agent      → queries customEvents using discovered batch events
├── dmf-agent        → queries customEvents using discovered DMF events
├── exception-agent  → queries exceptions table
└── form-agent       → queries pageViews table
    │
    ▼
Reports written  → reports/YYYY-MM-DD/{agent}-report-{timestamp}.md
    │
    ▼
Alerts written   → alerts/alert-{agent}-{timestamp}.json
                   (only when threshold breached)
    │
    ▼
Dashboard auto-generated → reports/dashboard.html  ← NEW in v2.0
    │
    ▼
Sleep 60 minutes
    │
    ▼
Repeat all night ↻
```

---

## Quick Start

### Prerequisites

- Node.js v18+ — [nodejs.org](https://nodejs.org)
- Azure CLI — [learn.microsoft.com/cli/azure](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- Anthropic API key — [console.anthropic.com](https://console.anthropic.com)
- Azure App Insights resource with D365 FO telemetry enabled

---

### Step 1 — Install Node.js

Download and install from [nodejs.org](https://nodejs.org) — version 18 or higher required.

```bash
node --version   # confirm v18+
```

---

### Step 2 — Install Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code
claude --version   # confirm installed
```

---

### Step 3 — Set your Anthropic API key

Get your key from [console.anthropic.com](https://console.anthropic.com) → API Keys → Create Key

```bash
# Windows (permanent — survives restarts)
setx ANTHROPIC_API_KEY "sk-ant-your-key-here"

# Mac/Linux
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
# Add to ~/.bashrc or ~/.zshrc to make permanent
```

> **Windows:** Close and reopen your terminal after `setx` for the key to take effect.

---

### Step 4 — Login to Azure

```bash
az login --tenant YOUR_TENANT.onmicrosoft.com

# Confirm you are logged in
az account show --query user.name --output tsv
```

> The system uses Azure AD authentication via `az rest` — no App Insights API keys needed.

---

### Step 5 — Clone the project

```bash
git clone https://github.com/prashantdce21MSFT/d365-observability-hub.git
cd d365-observability-hub
```

Or download the ZIP from [Releases](https://github.com/prashantdce21MSFT/d365-observability-hub/releases) and extract it.

---

### Step 6 — Configure your App Insights connection

Open `schemas/active.json` — this is the **only file you need to edit**:

```json
{
  "appId": "YOUR_APP_INSIGHTS_APP_ID",
  "appName": "YOUR_APP_INSIGHTS_NAME",
  "resourceGroup": "YOUR_RESOURCE_GROUP",
  "tenant": "YOUR_TENANT.onmicrosoft.com",
  "azRest": "az rest --method post --url \"https://api.applicationinsights.io/v1/apps/YOUR_APP_INSIGHTS_APP_ID/query\" --headers \"Content-Type=application/json\" --body \"{\\\"query\\\": \\\"{query}\\\"}\""
}
```

Find your App ID:

```bash
az monitor app-insights component show \
  --app YOUR_APP_INSIGHTS_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --query appId \
  --output tsv
```

Or in Azure Portal: **App Insights → YOUR_RESOURCE → Properties → Application ID**

---

### Step 7 — Start Claude Code

```bash
claude
```

---

### Step 8 — See the startup banner

```
> hello
```

---

### Step 9 — Load schema (always run this first)

```
> /load-schema
```

Schema-analyst will:
1. Read your App ID from `schemas/active.json`
2. Query App Insights live to discover all tables
3. Discover all event names in each table
4. Discover all `customDimensions` fields per event
5. Auto-generate `schemas/thresholds.json` with D365 defaults
6. Save everything to `schemas/parsed-schema.json`

When prompted press **2** — Yes, allow all edits this session.

```
Schema loaded — 3 tables, N events discovered. Ready for /monitor or /query.
- customEvents — N events (batch + DMF)
- exceptions   — N types
- pageViews    — N form events
```

> Edit `schemas/thresholds.json` to tune warning/critical levels for your environment.

---

### Step 10 — Test with a one-off query

```
> /query "show me the last DMF export"
```

---

### Step 11 — Start overnight monitoring

```
> /monitor
```

All 7 agents spawn in parallel. Every 60 minutes: reports written, alerts filed, dashboard generated.

---

### Run headless overnight

```bash
# Windows
start /B claude --dangerously-skip-permissions --print "/monitor 60" > monitor.log 2>&1

# Mac/Linux
nohup claude --dangerously-skip-permissions --print "/monitor 60" > monitor.log 2>&1 &

# Check on it
type monitor.log        # Windows
tail -f monitor.log     # Mac/Linux

# Stop in the morning
taskkill /IM claude.exe /F    # Windows
kill $(pgrep claude)           # Mac/Linux
```

---

## Understanding thresholds.json

Auto-generated on first `/load-schema`. Edit to tune for your environment — no agent files need changing.

```json
{
  "batch": {
    "slow_job_warning_ms": 60000,
    "slow_job_critical_ms": 300000,
    "thread_utilisation_warning_pct": 75,
    "thread_utilisation_critical_pct": 95,
    "queue_depth_warning": 10,
    "queue_depth_critical": 50,
    "throttle_critical_per_hour": 3
  },
  "dmf": {
    "job_duration_warning_ms": 300000,
    "job_duration_critical_ms": 1800000,
    "staging_error_same_entity_critical": 3
  },
  "exceptions": {
    "rate_warning_per_minute": 10,
    "rate_critical_per_minute": 50
  },
  "forms": {
    "p95_warning_ms": 3000,
    "p95_critical_ms": 10000
  },
  "general": {
    "lookback_window_minutes": 60,
    "max_rows_per_query": 100
  }
}
```

Common tuning examples:
- Batch jobs normally take 3 min → set `slow_job_warning_ms` to `180000`
- DMF exports legitimately take 15 min → set `job_duration_warning_ms` to `900000`
- Run every 30 min → set `lookback_window_minutes` to `30`

---

## Understanding parsed-schema.json

Auto-generated by schema-analyst — do not edit manually. Re-run `/load-schema` to refresh.

Contains everything agents need:
- Connection details (from `active.json`)
- All thresholds (from `thresholds.json`)
- Discovered tables, event names, column names
- Field mappings per event (duration field, status field, entity field)
- Warnings about your specific environment

Every agent reads exclusively from this file. No connection details or field names exist anywhere else.

---

## The 7 Agents

| Agent | Type | Purpose |
|-------|------|---------|
| `schema-analyst` | sub-task | Runs once. Discovers tables, events, columns live from App Insights. Auto-generates thresholds.json |
| `batch-agent` | specialist | Batch jobs — uses discovered event names and duration field |
| `dmf-agent` | specialist | DMF exports/imports — uses discovered status field and correlation field |
| `exception-agent` | specialist | X++ exceptions — uses discovered column names |
| `form-agent` | specialist | Form load times — uses discovered duration and name fields |
| `kql-generator` | sub-agent | Writes KQL from plain English using schema only — never hardcodes |
| `query-runner` | sub-agent | Executes KQL via az rest — reads App ID from parsed-schema only |
| `insights-writer` | sub-agent | Applies thresholds from parsed-schema — writes reports, alerts, dashboard |

---

## Commands

| Command | Description |
|---------|-------------|
| `/load-schema` | **Run this first.** Discovers schema, auto-generates thresholds.json |
| `/monitor` | Start overnight loop. All 7 agents every 60 min + auto dashboard |
| `/monitor 30` | Run every 30 minutes |
| `/query "..."` | One-off question in plain English |
| `/status` | Show recent reports, alert count, last run |

### Example /query questions

```
/query "show me the last DMF export"
/query "which batch jobs took more than 1 minute today"
/query "any batch job failures in the last 4 hours"
/query "how many DMF jobs ran in the last 7 days"
/query "what are the slowest forms right now"
/query "any new exception types today"
/query "show batch thread utilisation trend"
```

---

## Outputs

| Path | Contents | Written when |
|------|----------|-------------|
| `reports/YYYY-MM-DD/` | Markdown report per agent per cycle | Every cycle, always |
| `alerts/` | JSON alert per warning/critical finding | Only when threshold breached |
| `reports/dashboard.html` | Dark themed HTML dashboard | Auto after every cycle |
| `kql-cache/` | Every generated KQL + results | Every query |
| `run-log.jsonl` | Append-only structured event log | Every agent action |

### Report vs Alert

| | Report | Alert |
|--|--------|-------|
| Written | Every cycle — always | Only when threshold crossed |
| Format | Full markdown — summary, metrics, KQL, recommendations | Short JSON — severity, title, detail, recommendation |
| Purpose | Morning reading — full story | Immediate action — urgent findings |
| Analogy | Shift handover document | Pager notification |

---

## Project Structure

```
d365-observability-hub/
├── CLAUDE.md                      ← Orchestrator brain (always loaded)
├── .claude/
│   ├── settings.json              ← Tool permissions
│   ├── agents/
│   │   ├── schema-analyst.md      ← Discovers schema + auto-generates thresholds
│   │   ├── batch-agent.md         ← Batch monitoring (schema-driven)
│   │   ├── dmf-agent.md           ← DMF monitoring (schema-driven)
│   │   ├── exception-agent.md     ← Exception monitoring (schema-driven)
│   │   ├── form-agent.md          ← Form monitoring (schema-driven)
│   │   ├── kql-generator.md       ← KQL from plain English (schema-driven)
│   │   ├── query-runner.md        ← Executes KQL (App ID from schema only)
│   │   └── insights-writer.md     ← Reports + alerts + dashboard
│   └── commands/
│       ├── monitor.md
│       ├── query.md
│       ├── status.md
│       └── load-schema.md
├── schemas/
│   ├── active.json                ← YOU edit this (App ID + tenant only)
│   ├── thresholds.json            ← Auto-generated. Edit to tune.
│   └── parsed-schema.json         ← Auto-generated. Do not edit.
├── reports/                       ← Runtime (gitignored)
├── alerts/                        ← Runtime (gitignored)
├── kql-cache/                     ← Runtime (gitignored)
├── docs/
│   └── D365-Observability-Hub.gif
├── .env.example
├── SETUP.md
├── CONTRIBUTING.md
└── README.md
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `/load-schema` fails 403 | Your Azure account needs Reader access on App Insights resource |
| `/monitor` not recognised | You are outside Claude Code. Run `claude` first |
| Agents asking permission repeatedly | Press 2 (allow all edits this session) or use `--dangerously-skip-permissions` |
| Simulate mode instead of live data | Run `az login --tenant YOUR_TENANT` before starting `claude` |
| Reports folder empty | Wait ~5 min for first cycle to complete after `/monitor` |
| thresholds.json not generated | Check `schemas/active.json` has valid App ID then re-run `/load-schema` |
| parsed-schema.json stale | Re-run `/load-schema` — recommended every 7 days |
| Banner not showing | Type `hello` at the `>` prompt |

---

## Portability

Works on **any Azure App Insights resource** — not just D365 FO.

To point at a different environment:
1. Update `schemas/active.json` with the new App ID
2. Delete `schemas/thresholds.json` and `schemas/parsed-schema.json`
3. Run `/load-schema` — everything rediscovered automatically

No agent files need editing.

---

## Built With

- [Claude Code](https://claude.ai/code) — Anthropic's CLI agent framework
- [Azure Application Insights](https://azure.microsoft.com/en-us/products/monitor) — D365 telemetry
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/) — `az rest` for Azure AD auth
- [KQL](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/) — Kusto Query Language

---

## Author

**Prashant Verma** — Principal Consultant, AI Business Solutions


---

## License

MIT — see [LICENSE](LICENSE) for details.
