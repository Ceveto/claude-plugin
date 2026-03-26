---
name: explore
description: Explore Ceveto data — browse contacts, tasks, locations, assets. Use when the user says "show me contacts", "what tasks are open", "list locations", "explore ceveto data", "what's in ceveto", "show account data", or asks about their business data.
user-invocable: true
allowed-tools: mcp__ceveto__whoami, mcp__ceveto__list_accounts, mcp__ceveto__list_contacts, mcp__ceveto__list_tasks, mcp__ceveto__list_locations, mcp__ceveto__list_assets, mcp__ceveto__get_contact, mcp__ceveto__get_task, mcp__ceveto__get_location, mcp__ceveto__get_asset, mcp__ceveto__switch_default_account
argument-hint: "[contacts|tasks|locations|assets]"
---

# Explore Ceveto Data

Browse and search Ceveto business data using MCP tools.

## Available MCP Tools

These tools are dynamically generated from the Ceveto API. The exact tool names
depend on what was loaded, but common patterns:

| Action | Tool pattern | Parameters |
|--------|-------------|------------|
| List contacts | `list_contacts` | search, type, company, limit, offset |
| Get contact | `get_contact` | contact_id |
| List tasks | `list_tasks` | search, status, priority, assigned_to_id, limit |
| Get task | `get_task` | task_id |
| List locations | `list_locations` | search, type, country, city, limit |
| Get location | `get_location` | location_id |
| List assets | `list_assets` | search, type, status, limit |
| Get asset | `get_asset` | asset_id |

## Flow

### 1. Verify connection

Call `mcp__ceveto__whoami`. If fails, tell user to set up MCP first.

### 2. Determine what to explore

If $ARGUMENTS provided (e.g. "contacts"), explore that module.
Otherwise, give a quick overview — call list endpoints with limit=5 for each:

```
call list_contacts with limit=5
call list_tasks with limit=5
call list_locations with limit=5
```

### 3. Present data clearly

Format results as readable tables or summaries. For example:

**Contacts (16 total):**
| Name | Type | Company | Email |
|------|------|---------|-------|
| Andris Liepiņš | supplier | Company 10 UAB | ... |

**Tasks (42 total, 12 open):**
| Title | Status | Priority | Assigned |
|-------|--------|----------|----------|

### 4. Offer actions

After showing data, suggest what the user can do:
- "Want me to show details for any of these?"
- "I can filter by type, status, or search term"
- "I can create a new contact/task/location"

### 5. Drill down on request

When user asks for details, call the appropriate `get_*` tool with the ID
and show all fields in a readable format.
