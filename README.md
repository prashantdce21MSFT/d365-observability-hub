<div align="center">

# ◉ D365 Observability Hub

### Overnight AI Monitoring for D365 Finance & Operations

[![Claude Code](https://img.shields.io/badge/Powered%20by-Claude%20Code-7C3AED?style=flat-square)](https://claude.ai/code)
[![Azure App Insights](https://img.shields.io/badge/Azure-App%20Insights-0078D4?style=flat-square&logo=microsoftazure)](https://azure.microsoft.com/en-us/products/monitor)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![D365 FO](https://img.shields.io/badge/D365-Finance%20%26%20Operations-0078D4?style=flat-square)](https://dynamics.microsoft.com)

**You close your laptop. 7 AI agents wake up.**  
**By the time your morning coffee is ready — the reports are already there.**

[Getting Started](#getting-started) · [Architecture](#architecture) · [Agents](#the-7-agents) · [Commands](#commands) · [Outputs](#outputs)

</div>

---

## What Is This?

D365 Observability Hub is a **Claude Code native** overnight monitoring system for Dynamics 365 Finance & Operations. It uses AI agent architecture to automatically query your Azure App Insights, detect performance issues, and write reports — while you sleep.

> **No server. No Node.js app. No API keys. Just markdown files and Claude Code.**

### What It Monitors

| Domain | Events | What It Detects |
|--------|--------|-----------------|
| **Batch Jobs** | BatchTaskStart/Finished/Failure, BatchThrottled, BatchThreadInfo, BatchQueuesDetails | Slow jobs, failures, thread saturation, CPU/DTU throttling |
| **DMF** | DMFExportJobStart/End, DMFExportStagingEnd, DMFImportJobStart/End | Failed exports, staging errors, slow jobs, aborted runs |
| **Exceptions** | exceptions table | X++ errors, new exception types, batch correlations |
| **Forms** | pageViews table | Slow form loads, P95/P99, active sessions, regional issues |

---

## Architecture

```
You (/monitor or /query)
        │
        ▼
Claude Code — Orchestrator
reads CLAUDE.md · reads schema · spawns agents
        │
        ├──────────────────────────────────────┐
        │    Specialist agents (parallel)       │
        ├─ batch-agent                          │
        ├─ dmf-agent                            │
        ├─ exception-agent                      │
        └─ form-agent                           │
                                               │
        General agents (/query + custom)        │
        ├─ kql-generator                        │
        ├─ query-runner                         │
        └─ insights-writer                      │
                │                              │
                ▼                              │
        Azure App Insights (az rest · Azure AD) │
                │                              │
                ▼                              │
        Outputs (reports/ alerts/ kql-cache/)  │
                │                              │
                └──── sleep 60 min ────────────┘
                         ↻ repeat all night
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- [Claude Code](https://claude.ai/code) — `npm install -g @anthropic-ai/claude-code`
- Azure CLI — [install guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- An Anthropic API key — [console.anthropic.com](https://console.anthropic.com)
- Azure App Insights resource with D365 FO telemetry

### Installation

**1. Clone the repository**
```bash
git clone https://github.com/YOUR_USERNAME/d365-observability-hub.git
cd d365-observability-hub
```

**2. Set your Anthropic API key**
```bash
# Windows (permanent)
setx ANTHROPIC_API_KEY "sk-ant-YOUR_KEY_HERE"

# Mac/Linux
export ANTHROPIC_API_KEY="sk-ant-YOUR_KEY_HERE"
```

**3. Login to Azure**
```bash
az login --tenant YOUR_TENANT.onmicrosoft.com
az account show  # confirm you're logged in
```

**4. Configure your App Insights**

Edit `schemas/active.json` with your real schema, or run:
```
/load-schema path/to/your-schema.json
```

Update `query-runner.md` with your App Insights App ID:
```bash
az resource list --resource-type "microsoft.insights/components" \
  --query "[].{name:name, resourceGroup:resourceGroup}" --output table
```

**5. Start monitoring**
```bash
claude
```
Then at the `>` prompt:
```
/monitor
```

---

## The 7 Agents

| Agent | Type | Purpose |
|-------|------|---------|
| `schema-analyst` | sub-task | Runs once at startup. Maps tables, columns, event types from your schema |
| `batch-agent` | specialist | Batch jobs — speed, failures, thread saturation, throttling, queue depth |
| `dmf-agent` | specialist | DMF exports/imports — staging status, duration, entity errors |
| `exception-agent` | specialist | X++ exceptions — rate, new types, correlation to batch failures |
| `form-agent` | specialist | Form load times — P95/P99, slow pages, user sessions, regional issues |
| `kql-generator` | sub-agent | Writes KQL from plain English. Only uses your real schema columns |
| `query-runner` | sub-agent | Executes KQL via `az rest`. Azure AD auth — no API keys |
| `insights-writer` | sub-agent | Applies D365 thresholds. Writes reports and alerts |

---

## Commands

| Command | Description |
|---------|-------------|
| `/monitor` | Start the overnight loop. All 7 agents run every 60 min |
| `/monitor 30` | Run every 30 minutes instead of default 60 |
| `/query "..."` | One-off question in plain English. Runs 3 general agents |
| `/status` | Show recent reports, alert count, last run time |
| `/load-schema path` | Load and parse a new App Insights schema |

### Example queries
```
/query "show me the last DMF export"
/query "which batch jobs took more than 1 minute today"
/query "any BatchTaskFailure in the last 4 hours"
/query "show batch thread utilisation trend"
/query "what are the slowest forms right now"
/query "any new exception types today"
```

---

## Outputs

| Path | Contents |
|------|----------|
| `reports/YYYY-MM-DD/` | One markdown report per agent per cycle. Never overwritten |
| `alerts/` | JSON alert per warning/critical finding |
| `kql-cache/` | Every generated KQL saved. Open directly in App Insights Logs |
| `run-log.jsonl` | Append-only structured event log |

### Detection Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Batch job duration | > 1 minute | > 5 minutes |
| Batch thread utilisation | > 75% | > 95% |
| Task queue depth | > 10 | > 50 |
| BatchThrottled events | Any | 3+ per hour |
| BatchTaskFailure Critical=True | Any | Any |
| DMF job duration | > 5 minutes | > 30 minutes |
| DMF StagingStatus=Error | Any | 3+ same entity |
| Form P95 load time | > 3 seconds | > 10 seconds |
| Exception rate | > 10/min | > 50/min |
| New exception type | Any | — |

---

## Headless Overnight Mode

```bash
# Windows
start /B claude --print "/monitor 60" > monitor.log 2>&1

# Mac/Linux  
nohup claude --print "/monitor 60" > monitor.log 2>&1 &
echo "PID: $!"

# Check on it
tail -f monitor.log

# Stop in the morning
taskkill /IM claude.exe /F   # Windows
kill <PID>                    # Mac/Linux
```

---

## Project Structure

```
d365-observability-hub/
├── CLAUDE.md                          ← Orchestrator brain (always loaded)
├── .claude/
│   ├── settings.json                  ← Tool permissions (az rest must be allowed)
│   ├── agents/
│   │   ├── schema-analyst.md
│   │   ├── batch-agent.md
│   │   ├── dmf-agent.md
│   │   ├── exception-agent.md
│   │   ├── form-agent.md
│   │   ├── kql-generator.md
│   │   ├── query-runner.md
│   │   └── insights-writer.md
│   └── commands/
│       ├── monitor.md
│       ├── query.md
│       ├── status.md
│       └── load-schema.md
├── schemas/
│   └── active.json                    ← Your App Insights schema (edit this)
├── reports/                           ← Written by agents (gitignored)
├── alerts/                            ← Written by agents (gitignored)
├── kql-cache/                         ← Written by agents (gitignored)
└── docs/
    └── architecture.png
```

---

## Customising Objectives

Edit the `## Default Monitoring Objectives` section in `CLAUDE.md`, or create an `objectives.json`:

```json
[
  "BOMLEVELRECALCULATION lock durations last hour",
  "HDMMasterPlanningRunController batch job trends",
  "Top slow LedgerJournalPost operations today",
  "AOS heap usage from customMetrics last 4h"
]
```

Then: `/monitor 60 schemas/active.json objectives.json`

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `/monitor` not recognised | You're outside Claude Code. Run `claude` first |
| Agents blocked — permission denied | Add `"Bash(az rest:*)"` to `.claude/settings.json` allow list |
| Simulate mode instead of live | Run `az login` before starting `claude` |
| Reports folder empty | Wait for first cycle to complete (~5 min after `/monitor`) |
| App Insights 403 error | Your Azure account needs Reader access on the App Insights resource |

---

## Built With

- [Claude Code](https://claude.ai/code) — Anthropic's CLI agent framework
- [Azure Application Insights](https://azure.microsoft.com/en-us/products/monitor) — D365 telemetry
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/) — `az rest` for Azure AD auth
- [KQL](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/) — Kusto Query Language

---

## Author

**Prashant Verma** — Principal Consultant, AI Business Solutions  
Built with Claude Code · Recognised by Microsoft Cowork Programme

---

## License

MIT — see [LICENSE](LICENSE) for details.
