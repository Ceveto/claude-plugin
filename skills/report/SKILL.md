---
name: report
description: Generate reports from Ceveto data. Use when the user says "report", "summary", "overview", "stats", "how many", "count", "analytics", or asks for aggregated business data.
user-invocable: true
allowed-tools: mcp__ceveto__whoami, mcp__ceveto__list_contacts, mcp__ceveto__list_tasks, mcp__ceveto__list_locations, mcp__ceveto__list_assets, mcp__ceveto__list_time_entries
argument-hint: "[contacts|tasks|overview]"
---

# Ceveto Report Generator

Generate business reports and summaries by querying Ceveto MCP tools.

## Available Data Sources

| Module | Tool | Key fields for reporting |
|--------|------|------------------------|
| Contacts | `list_contacts` | type, company_name, is_active |
| Tasks | `list_tasks` | status, priority, assigned_to, scheduled_date, due_date |
| Locations | `list_locations` | type, country, city, is_active |
| Assets | `list_assets` | type, status |
| Time entries | `list_time_entries` | date, hours, task_id |

## Flow

### 1. Verify connection

Call `mcp__ceveto__whoami`. Note the account name for the report header.

### 2. Determine report type

If $ARGUMENTS provided, generate that specific report. Otherwise ask what
the user wants:

- **Overview** — counts across all modules
- **Contacts** — breakdown by type, company
- **Tasks** — breakdown by status, priority, overdue analysis
- **Custom** — user specifies what they want

### 3. Gather data

Call relevant MCP tools with high limits to get full data:

```
list_contacts(limit=1000) → count by type
list_tasks(limit=1000) → count by status, priority
list_locations(limit=1000) → count by type, country
```

### 4. Compute aggregates

From the raw data, compute:
- **Totals**: count per module
- **Breakdowns**: group by type/status/priority
- **Trends**: overdue tasks, inactive contacts
- **Highlights**: top companies by contact count, busiest locations

### 5. Format report

Present as a structured report with the account name and date:

```markdown
# Ceveto Report — Pixels
Generated: 2026-03-26

## Overview
| Module | Count |
|--------|-------|
| Contacts | 16 |
| Tasks | 42 |
| Locations | 8 |

## Contacts by Type
| Type | Count |
|------|-------|
| client | 5 |
| supplier | 4 |
| manufacturer | 3 |

## Tasks by Status
| Status | Count | % |
|--------|-------|---|
| open | 12 | 29% |
| in_progress | 18 | 43% |
| completed | 12 | 29% |

## Attention Needed
- 3 tasks overdue (past due_date)
- 2 contacts inactive
```

### 6. Offer follow-up

"Want me to drill into any section? Or export this as a file?"
