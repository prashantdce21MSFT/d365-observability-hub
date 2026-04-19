---
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
