# Setup Guide

## Step 1 — Find your App Insights App ID

```bash
az login --tenant YOUR_TENANT.onmicrosoft.com

az resource list \
  --resource-type "microsoft.insights/components" \
  --query "[].{name:name, appId:properties.AppId, rg:resourceGroup}" \
  --output table
```

Copy the App ID from the output.

## Step 2 — Update these files with your values

| File | What to replace |
|------|-----------------|
| `CLAUDE.md` | `YOUR_APP_INSIGHTS_NAME`, `YOUR_APP_INSIGHTS_APP_ID` |
| `schemas/active.json` | All `YOUR_*` placeholders |
| `.claude/agents/query-runner.md` | `YOUR_APP_INSIGHTS_APP_ID`, `YOUR_APP_INSIGHTS_NAME`, `YOUR_RESOURCE_GROUP` |
| `.claude/agents/batch-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/dmf-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/exception-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |
| `.claude/agents/form-agent.md` | `YOUR_APP_INSIGHTS_APP_ID` |

## Step 3 — Load your real schema

In Claude Code run:
```
/load-schema path/to/your-schema.json
```

Or export your schema from App Insights:
```bash
az monitor app-insights component show \
  --app YOUR_APP_INSIGHTS_NAME \
  --resource-group YOUR_RESOURCE_GROUP
```

## Step 4 — Set your API key

```bash
# Windows
setx ANTHROPIC_API_KEY "sk-ant-your-key-here"

# Mac/Linux
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
```

## Step 5 — Start monitoring

```bash
cd d365-observability-hub
claude
```

Then at the `>` prompt:
```
/monitor
```
