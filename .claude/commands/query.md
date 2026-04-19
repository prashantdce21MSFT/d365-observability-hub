---
description: Run a one-off monitoring query against App Insights. Usage: /query "your question in plain English"
argument-hint: "natural language query"
---

Run a single one-off App Insights query for: $1

1. Read `schemas/parsed.json` (or `schemas/active.json` if parsed doesn't exist yet, and run schema-analyst first)
2. Spawn `kql-generator` with objective: "$1" and task ID "ondemand-{timestamp}"
3. Spawn `query-runner` for that task
4. Spawn `insights-writer` for that task  
5. Print the findings directly in the terminal — don't just save to a file, show the key metrics and findings inline
6. Also save the report to `reports/ondemand/`
