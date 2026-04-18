---
description: Start the D365 monitoring loop. Runs all 7 agents every cycle. Usage: /monitor [interval_minutes]
argument-hint: [interval_minutes]
---

Start the D365 Monitoring Agent overnight loop. Interval: $1 minutes (default 60).

## Startup sequence

1. **Check environment**
```bash
echo "ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY:+SET}"
echo "az login status: $(az account show --query user.name -o tsv 2>/dev/null || echo NOT LOGGED IN)"
```

2. **Load and parse schema**
Read `schemas/active.json`, spawn `schema-analyst` sub-agent.

3. **Print banner**
```
╔══════════════════════════════════════════════════════════╗
║   D365 Monitoring Agent  •  YOUR_APP_INSIGHTS_NAME              ║
║   7 agents  •  Claude Code  •  Overnight loop            ║
╚══════════════════════════════════════════════════════════╝
Agents:  batch · dmf · exception · form · kql · runner · insights
Interval: [N] minutes
Auth:     az rest (Azure AD)
```

## Monitoring loop (repeat until stopped)

Each cycle, spawn ALL of these in parallel using the Task tool:

**Specialist agents (always run):**
- `batch-agent` — batch job health
- `dmf-agent` — DMF export/import health  
- `exception-agent` — X++ exception analysis
- `form-agent` — form load time and user sessions

**General agents (for default objectives):**
- `kql-generator` → `query-runner` → `insights-writer` for any additional objectives

After each cycle:
- Print summary: how many reports written, how many alerts filed, any criticals
- Print `⏳ Sleeping [N] minutes...`
- Run `bash -c "sleep $((N*60))"`
- Repeat

## Graceful stop
When user interrupts, finish current cycle then print final summary of all reports and alerts written.
