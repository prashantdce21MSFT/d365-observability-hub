---
<<<<<<< HEAD
description: Initialise the system by running schema-analyst to discover all tables, event names, columns and fields dynamically from your App Insights resource. Run this first before /monitor or /query.
---

Run the schema-analyst agent to:
1. Read connection details from schemas/active.json
2. Discover all tables dynamically from App Insights
3. Discover all event names in each table
4. Discover all customDimensions fields per event
5. Load thresholds from schemas/thresholds.json
6. Save everything to schemas/parsed-schema.json

Print progress at each step. When complete print:
"Schema loaded — {table count} tables, {event count} events discovered. Ready for /monitor or /query."
=======
description: Load and parse an App Insights schema file. Usage: /load-schema path/to/schema.json
argument-hint: path/to/schema.json
---

Load schema from: $1

1. Read the file at $1
2. Copy it to `schemas/active.json` (overwrite)
3. Spawn `schema-analyst` sub-agent to parse it into `schemas/parsed.json`
4. Print a summary of tables and columns found
5. Print the suggested objectives the schema-analyst identified
>>>>>>> 1f5af014f35da249ebf591b7249ce535f318d132
