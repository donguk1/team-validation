# team-validation

Claude Code multi-agent validation & design plugin.

## Structure

- `agents/validation-*.md` — Read-only analysis agents (14)
- `agents/design-*.md` — Feature design agents (6)
- `commands/team-validation.md` — `/team-validation` orchestrator
- `commands/team-design.md` — `/team-design` orchestrator
- `.claude-plugin/plugin.json` — Plugin metadata + version

## Agent File Format

All agents follow this structure:

```markdown
---
name: {group}-{role}           # kebab-case, e.g. validation-security
description: English one-liner  # shown in plugin UI
model: sonnet
color: {color}                  # blue/cyan/green/yellow/magenta/red
tools: ["Read", "Glob", "Grep"] # minimal — only python/security get Bash
---

English body with 4 sections:
1. **Your Core Responsibilities:** — numbered list (5-6 items)
2. **Analysis/Design Process:** — numbered list (6 steps)
3. **Output Format:** — markdown template inside code block
4. **Important:** — READ-ONLY constraint + .env prohibition
```

### Tools Policy

- Default: `["Read", "Glob", "Grep"]`
- Bash allowed only for: `validation-python`, `validation-security`
- Task tool: never on agents (orchestrator-only)

### Output Section Order

1. Critical / Warning / Suggestion (severity first)
2. Domain-specific checklists/tables
3. Score: X/10

## Commands

- Orchestrators use `Task` tool with dedicated `subagent_type: "team-validation:{agent-name}"` (e.g. `"team-validation:validation-architect"`)
- Agent prompts are auto-loaded — Task prompt should contain only context (project path, scope, tech stack)
- No TeamCreate/TeamDelete (not available in Claude Code API)
- Empty `$ARGUMENTS` → AskUserQuestion for scope selection

## Versioning

- Semantic Versioning in `plugin.json`
- MINOR: agent/command add/remove
- PATCH: prompt edits, bug fixes
- Always sync `plugin.json` version ↔ `SCALING.md` version history
- Auto-update is enabled on marketplace

## Presets

### Validation (4~12 agents)

Required 4: architect, code-quality, bug-hunter, security

| Type | Additional | Total |
|------|-----------|-------|
| Python backend | backend, dedup, db-optimizer, python, testing | 9 |
| JS/TS frontend | frontend, dedup, product, testing | 8 |
| Game | game, testing | 6 |
| Data/ML | dedup, db-optimizer, python, data, testing | 9 |
| Fullstack | backend, frontend, dedup, db-optimizer, product, testing, devops + python(if exists) | 11~12 |

### Design (3~6 agents)

| Type | Agents | Total |
|------|--------|-------|
| Python backend | architect, api, data-model, task-breakdown | 4 |
| JS/TS frontend | architect, frontend, task-breakdown | 3 |
| Game | architect, task-breakdown, test-strategy | 3 |
| Fullstack | architect, api, data-model, frontend, task-breakdown, test-strategy | 6 |
| Data/ML | architect, data-model, task-breakdown | 3 |
