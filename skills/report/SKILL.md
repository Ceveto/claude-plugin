---
name: report
description: Generate reports from Ceveto data. Use when the user says "report", "summary", "overview", "stats", "how many", "count", "analytics", "dashboard", or asks for aggregated business data.
user-invocable: true
argument-hint: "[contacts|tasks|overview|custom]"
---

# Ceveto Report Generator

Generate business reports by querying and aggregating data from Ceveto MCP tools.

## How to find tools

MCP tools are dynamically generated. Use ToolSearch to discover:
```
ToolSearch query: "mcp__ceveto__list"   → all list endpoints
ToolSearch query: "mcp__ceveto__report" → any report endpoints
```

## Data sources for reporting

| Data | Tool | Group by | Count/aggregate |
|------|------|----------|-----------------|
| Contacts | `list_contacts` | type, company_name | active vs inactive |
| Tasks | `list_tasks` | status, priority, assigned_to | open/in_progress/completed, overdue |
| Locations | `list_locations` | type, country, city | active count |
| Assets | `list_assets` | type, status | operational vs maintenance |
| Time entries | `list_time_entries` | date, user | hours per day/week |
| Teams | `list_teams` | — | member count |
| Orders | `list_orders` | status | total value |
| Payments | `list_payments` | status | amounts |
| Task types | `list_task_types` | — | count per type |
| Audit log | `list_audit_logs` | — | recent activity |

## Flow

### 1. Verify connection, permissions, and discover tools

Call `mcp__ceveto__whoami`. From the response:
- Use `account_name` for report header
- Check `is_owner` — if true, can report on all modules
- Check `permissions` — only include modules the user has read access to

For each module in the report, verify the user has at least `read` permission
before calling list tools. Skip modules without access — don't show errors,
just omit them and note "X modules not accessible with current permissions".

Use ToolSearch to discover available list tools:
```
ToolSearch query: "mcp__ceveto__list"
```

Read tool descriptions to understand available filter parameters and
return fields — use these to build better aggregations.

### 2. Determine report scope

If $ARGUMENTS provided, focus on that. Otherwise offer:
- **Overview** — counts and highlights from all modules
- **Contacts** — breakdown by type, top companies
- **Tasks** — status distribution, priority, overdue, assignments
- **Time** — hours logged, by person, by task
- **Custom** — user specifies criteria

### 3. Gather data with high limits

Call list tools with `limit=500` or `limit=1000` to get comprehensive data.
Make parallel calls where possible:

```
list_contacts(limit=500)
list_tasks(limit=500)
list_locations(limit=500)
```

### 4. Compute aggregates in code

Process the JSON results to compute:
- **Totals** per module
- **Breakdowns** — group by type/status/priority, count each
- **Percentages** — each group as % of total
- **Trends** — overdue tasks (due_date < today), stale contacts
- **Top N** — busiest locations, most tasks by assignee

### 5. Format as structured report

```markdown
# Ceveto Report — {Account Name}
Generated: {date}

## Overview
| Module | Count |
|--------|-------|
| Contacts | 16 |
| Tasks | 42 (12 open) |
| Locations | 8 |
| Assets | 24 |

## Tasks by Status
| Status | Count | % |
|--------|-------|---|
| open | 12 | 29% |
| in_progress | 18 | 43% |
| completed | 12 | 29% |

## Tasks by Priority
| Priority | Count |
|----------|-------|
| urgent | 3 |
| high | 8 |
| medium | 22 |
| low | 9 |

## Contacts by Type
| Type | Count |
|------|-------|
| client | 5 |
| supplier | 4 |
| manufacturer | 3 |
| vendor | 2 |
| other | 2 |

## Attention Needed
- 3 tasks overdue
- 5 tasks high/urgent priority still open
- 2 contacts inactive
```

### 6. Save report snapshot

Save the raw aggregated data to `${CLAUDE_PLUGIN_DATA}/last-report.json`
(resolves to `~/.claude/plugins/data/ceveto/last-report.json`):

```json
{
  "account": "pixels",
  "generated_at": "2026-03-26T22:00:00Z",
  "totals": {"contacts": 16, "tasks": 42, "locations": 8},
  "tasks_by_status": {"open": 12, "in_progress": 18, "completed": 12},
  "contacts_by_type": {"client": 5, "supplier": 4},
  "alerts": ["3 tasks overdue", "2 contacts inactive"]
}
```

Create the directory if it doesn't exist:
```bash
mkdir -p ~/.claude/plugins/data/ceveto
```

### 7. Compare with previous report

Before generating, check if `~/.claude/plugins/data/ceveto/last-report.json` exists.
If it does, load it and compute deltas:

```
Since last report (2026-03-25):
  +3 new tasks (39 → 42)
  +1 new contact (15 → 16)
  -2 tasks completed
  1 new overdue task
```

Show deltas in an extra section at the top of the report.

### 8. Offer follow-up

- "Want me to drill into any section?"
- "I can generate a more detailed report for a specific module"
- "Want me to save this as a markdown file?"
- "I can compare specific modules over time"
