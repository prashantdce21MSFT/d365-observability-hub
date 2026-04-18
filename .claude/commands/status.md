---
description: Show current monitoring status — recent reports, alert count, last run time
---

Show a status summary of the monitoring agent:

1. Read `run-log.jsonl` (last 20 lines) — show last run time and task counts
2. List files in `reports/` — count and show most recent
3. List files in `alerts/` — count and show any unacknowledged alerts with their severity
4. Show env status: APPINSIGHTS_APP_ID set or simulate mode
5. Print a clean summary table in the terminal
