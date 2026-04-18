<div align="center">

# ◉ D365 Observability Hub

### Overnight AI Monitoring for D365 Finance & Operations

[![Claude Code](https://img.shields.io/badge/Powered%20by-Claude%20Code-7C3AED?style=flat-square)](https://claude.ai/code)
[![Azure App Insights](https://img.shields.io/badge/Azure-App%20Insights-0078D4?style=flat-square&logo=microsoftazure)](https://azure.microsoft.com/en-us/products/monitor)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![D365 FO](https://img.shields.io/badge/D365-Finance%20%26%20Operations-0078D4?style=flat-square)](https://dynamics.microsoft.com)

**You close your laptop. 7 AI agents wake up.**
**By the time your morning coffee is ready — the reports are already there.**

[Quick Start](#quick-start) · [Architecture](#architecture) · [Agents](#the-7-agents) · [Commands](#commands) · [Outputs](#outputs)

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

## Quick Start

### Step 1 — Install Node.js

Download and install from [nodejs.org](https://nodejs.org) — version 18 or higher required.

```bash
node --version   # confirm v18+
```

---

### Step 2 — Install Claude Code CLI

```bash
npm install -g @anthropic-ai/claude-code

claude --version  # confirm it installed
```

---

### Step 3 — Set your Anthropic API key

Get your key from [console.anthropic.com](https://console.anthropic.com) → API Keys → Create Key

```bash
# Windows (permanent — survives terminal restarts)
setx ANTHROPIC_API_KEY "sk-ant-your-key-here"

# Mac/Linux
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
# Add to ~/.bashrc or ~/.zshrc to make permanent
```

> **Windows:** Close and reopen your terminal after running `setx` for the key to take effect.

---

### Step 4 — Install Azure CLI and login

Download Azure CLI from [learn.microsoft.com/cli/azure/install-azure-cli](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

```bash
# Login to your Azure tenant
az login --tenant YOUR_TENANT.onmicrosoft.com

# Confirm you are logged in
az account show --query user.name --output tsv
```

---

### Step 5 — Create your project folder

```bash
# Windows
mkdir d365-observability-hub
cd d365-observability-hub

# Mac/Linux
mkdir d365-observability-hub
cd d365-observability-hub
```

---

### Step 6 — Create the folder structure

```bash
# Windows — run each line separately
mkdir .claude
mkdir .claude\agents
mkdir .claude\commands
mkdir schemas
mkdir reports
mkdir alerts
mkdir kql-cache
mkdir docs

# Mac/Linux
mkdir -p .claude/agents .claude/commands schemas reports alerts kql-cache docs
```

---

### Step 7 — Clone or download the project files

**Option A — Clone with Git**
```bash
git clone https://github.com/prashantdce21MSFT/d365-observability-hub.git
cd d365-observability-hub
```

**Option B — Download ZIP**

Download the ZIP from the [Releases page](https://github.com/prashantdce21MSFT/d365-observability-hub/releases), extract it, and place the files in your folder.

Your folder should look like this:
```
d365-observability-hub/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
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
│   └── active.json
├── reports/
├── alerts/
├── kql-cache/
├── .env.example
├── SETUP.md
└── README.md
```

---

### Step 8 — Find your App Insights App ID

```bash
az monitor app-insights component show \
  --app YOUR_APP_INSIGHTS_NAME \
  --resource-group YOUR_RESOURCE_GROUP \
  --query appId \
  --output tsv
```

Or find it in Azure Portal:
```
Azure Portal → App Insights → YOUR_RESOURCE → Properties → Application ID
```

---

### Step 9 — Configure your App Insights details

Replace all `YOUR_*` placeholders in the following files with your real values:

| File | What to replace |
|------|-----------------|
| `CLAUDE.md` | `YOUR_APP_INSIGHTS_NAME`, `YOUR_APP_INSIGHTS_APP_ID` |
| `schemas/active.json` | All `YOUR_*` placeholders |
| `.claude/agents/query-runner.md` | App ID, resource group, app name |
| `.claude/agents/batch-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/dmf-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/exception-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/form-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |

> See [SETUP.md](SETUP.md) for detailed step-by-step configuration instructions.

---

### Step 10 — Start Claude Code

Navigate to your project folder and start Claude Code:

```bash
cd d365-observability-hub
claude
```

You will see the Claude Code header with the `>` prompt. You are now inside Claude Code.

---

### Step 11 — See the startup banner

Type at the `>` prompt:

```
hello
```

Claude reads `CLAUDE.md` automatically and prints the D365 Observability Hub banner — all 7 agents listed, your target environment, auth status, and available commands.

---

### Step 12 — Test with a one-off query

Before running the overnight loop, test your connection with a quick query:

```
/query "show me the last DMF export"
```

Three agents spin up, query your real App Insights using Azure AD, and return results in plain English. If you see real D365 data — you are connected.

---

### Step 13 — Start overnight monitoring

```
/monitor
```

All 7 agents spawn in parallel. Reports and alerts are written every 60 minutes. The loop runs until you press `Ctrl+C`.

---

### Run headless overnight (no terminal needed)

```bash
# Windows
start /B claude --print "/monitor 60" > monitor.log 2>&1

# Mac/Linux
nohup claude --print "/monitor 60" > monitor.log 2>&1 &

# Check on it anytime
tail -f monitor.log        # Mac/Linux
type monitor.log           # Windows

# Stop in the morning
taskkill /IM claude.exe /F   # Windows
kill $(pgrep claude)          # Mac/Linux
```

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

### Example /query questions

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

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `/monitor` not recognised | You are outside Claude Code. Run `claude` first, then type `/monitor` at the `>` prompt |
| Agents blocked — permission denied | Add `"Bash(az rest:*)"` to `.claude/settings.json` allow list |
| Simulate mode instead of live data | Run `az login --tenant YOUR_TENANT` before starting `claude` |
| Reports folder empty | Wait for first cycle to complete (~5 min after `/monitor`) |
| App Insights 403 error | Your Azure account needs Reader access on the App Insights resource |
| Banner not showing | Type `hello` at the `>` prompt to trigger it |
| Auth conflict warning | Run `/logout` then restart `claude` |

---

## Customising Objectives

Edit the monitoring objectives in `CLAUDE.md`, or create an `objectives.json`:

```json
[
  "BOMLEVELRECALCULATION lock durations last hour",
  "MasterPlanningRunController batch job trends",
  "Top slow LedgerJournalPost operations today"
]
```

Then run: `/monitor 60 schemas/active.json objectives.json`

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
