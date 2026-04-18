# Contributing to D365 Observability Hub

Contributions are welcome! Here are some ways you can help:

## Adding New Agents

Create a new `.md` file in `.claude/agents/` following the pattern of existing agents:

```markdown
---
name: your-agent-name
description: When to invoke this agent and what it does
tools: Read, Write, Bash
---

# Your Agent Name

## App Insights Config
- App ID: (from schemas/active.json)
- Auth: az rest (Azure CLI)

## Events You Monitor
...

## KQL Patterns
...

## Thresholds
...
```

## Adding New Slash Commands

Create a `.md` file in `.claude/commands/` with a YAML frontmatter block:

```markdown
---
description: What the command does
argument-hint: [optional-arg]
---

Instructions for Claude when this command is run...
```

## Improving Thresholds

Edit the threshold tables in `CLAUDE.md` and in the relevant agent `.md` file.

## Submitting

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-agent-name`
3. Commit your changes
4. Open a pull request with a description of what your agent monitors
