---
description: Load and parse an App Insights schema file. Usage: /load-schema path/to/schema.json
argument-hint: path/to/schema.json
---

Load schema from: $1

1. Read the file at $1
2. Copy it to `schemas/active.json` (overwrite)
3. Spawn `schema-analyst` sub-agent to parse it into `schemas/parsed.json`
4. Print a summary of tables and columns found
5. Print the suggested objectives the schema-analyst identified
