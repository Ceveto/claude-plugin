---
name: explore
description: Explore Ceveto data — browse contacts, tasks, locations, assets, teams, tags, payments, time entries, and more. Use when the user says "show me contacts", "what tasks are open", "list locations", "explore ceveto data", "what's in ceveto", "show account data", or asks about their business data.
user-invocable: true
argument-hint: "[module-name]"
---

# Explore Ceveto Data

Browse and search Ceveto business data using dynamically generated MCP tools.

## How MCP tools work

Ceveto MCP tools are generated from the backend's OpenAPI schema at startup.
Tool names follow the pattern `mcp__ceveto__{operation}` where operation is
derived from Django Ninja's operationId.

**To discover available tools**, use ToolSearch:
```
ToolSearch query: "mcp__ceveto__list"     → find all list tools
ToolSearch query: "mcp__ceveto__get"      → find all get/detail tools
ToolSearch query: "mcp__ceveto__create"   → find all create tools
ToolSearch query: "mcp__ceveto__contact"  → find contact-specific tools
ToolSearch query: "mcp__ceveto__task"     → find task-specific tools
```

## Available modules

These modules are typically available (depends on user permissions):

| Module | List tool | Get tool | Create tool |
|--------|-----------|----------|-------------|
| Contacts | `list_contacts` | `get_contact` | `create_contact` |
| Tasks | `list_tasks` | `get_task` | `create_task` |
| Locations | `list_locations` | `get_location` | `create_location` |
| Assets (CMMS) | `list_assets` | `get_asset` | `create_asset` |
| Teams | `list_teams` | `get_team` | `create_team` |
| Skills | `list_skills` | `get_skill` | `create_skill` |
| Tags | `list_tags` | `get_tag` | `create_tag` |
| Orders | `list_orders` | `get_order` | `create_order` |
| Payments | `list_payments` | `get_payment` | `create_payment` |
| Time entries | `list_time_entries` | `get_time_entry` | `create_time_entry` |
| Schedules | `list_schedules` | `get_schedule` | `create_schedule` |
| Task types | `list_task_types` | `get_task_type` | `create_task_type` |
| Task comments | `list_comments` | `get_comment` | `create_comment` |
| Documents | `list_documents` | `get_document` | `create_document` |
| Account members | `list_account_accesses` | `get_account_access` | — |
| Manufacturers | `list_manufacturers` | `get_manufacturer` | `create_manufacturer` |
| Suppliers | `list_suppliers` | `get_supplier` | `create_supplier` |
| Audit log | `list_audit_logs` | — | — |
| Inspections | `list_inspection_templates` | `get_inspection_template` | — |
| Recurring tasks | `list_recurring_templates` | `get_recurring_template` | — |
| Workflows | `list_workflows` | `get_workflow` | — |
| Task forms | `list_forms` | `get_form` | — |

Additional tools: `transition_task`, `archive_*`, `restore_*`, `update_*`, `delete_*`

## Common parameters

Most list tools accept:
- `search` — text search (name, title)
- `type` / `status` / `priority` — filter by enum value
- `limit` — max results (default varies, usually 20-50)
- `offset` — pagination offset
- `ordering` — sort field (prefix `-` for descending)

## Flow

### 1. Verify connection and check permissions

Call `mcp__ceveto__whoami`. Parse the response:

```json
{
  "account_name": "Pixels",
  "is_owner": true,
  "permissions": {
    "contacts": {"read": {}, "write": {}},
    "tasks": {"read": {}},
    "locations": {"read": {}, "write": {"max_amount": 5000}}
  }
}
```

**Permission rules:**
- `is_owner: true` → full access to everything, skip permission checks
- `permissions` dict maps module → allowed actions (read/write/admin)
- If module is missing from permissions → user has NO access to it
- `read` → can list and get, but NOT create/update/delete
- `write` → can list, get, create, update, delete
- `admin` → all of write + admin actions
- Conditions like `max_amount: 5000` are limits on write operations

**Before calling any tool**, check permissions:
- Only offer list/get for modules with `read` permission
- Only offer create/update/delete for modules with `write` permission
- If a module is not in permissions (and not owner), tell the user:
  "You don't have access to {module}. Contact your admin."
- If write has conditions (e.g. max_amount), mention the limit

Save permissions to `~/.claude/plugins/data/ceveto/permissions.json` so
other skill invocations can reuse without re-calling whoami.

### 2. Discover available tools

The MCP tools are dynamically generated from the API's OpenAPI schema.
Each tool has a description with parameters and return type info.

Use ToolSearch to find tools for the module the user wants:
```
ToolSearch query: "mcp__ceveto__contact"  → contact tools
ToolSearch query: "mcp__ceveto__task"     → task tools
```

Read the tool descriptions — they contain parameter names, types,
valid enum values, and any permission limits.

### 3. Determine what to explore

If $ARGUMENTS provided (e.g. "contacts"), explore that module.

If the user asks about something specific, use ToolSearch to find the right
tool. For example "show me task types" → search for `mcp__ceveto__task_type`.

If no specific request, give a quick overview — call list tools with limit=5
for the main modules.

### 3. Present data as readable tables

Format results with clear headers and columns relevant to the module.
Omit null fields and internal IDs unless the user asks for them.

### 4. Offer next steps

After showing data, suggest:
- "Want details on any of these? Give me the name or ID"
- "I can filter by type, status, or search term"
- "I can create, update, or delete entries"
- "I can show related data — e.g. tasks at a location, contacts for a company"

### 5. Chain tools for related data

When the user asks for related data, chain tools:
- Contact → tasks assigned to contact → time entries on those tasks
- Location → assets at location → maintenance tasks for those assets
- Team → team members → tasks assigned to team members
