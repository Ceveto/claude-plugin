# Ceveto Status

## Step 1: .mcp.json Configuration

Read from `/Users/cherijs/_REPO/ceveto/.mcp.json`:

| Setting     | Value                        |
|-------------|------------------------------|
| Mode        | Local (uv)                   |
| Environment | Dev (localhost:8400)         |
| Base URL    | `https://localhost:8400`     |
| Username    | `f52026f5...`                |
| Private Key | [REDACTED]                   |

The `ceveto` server is configured with a `command` key (`uv run --directory ... python -m ceveto_mcp`), confirming **Local** mode.

## Step 2: Connection Test (whoami)

Called `/company-api/me/` using Ed25519 signed request. Response:

```json
{
  "account_id": "c475113f-5c86-4c22-bfbc-47630561b96f",
  "account_name": "Pixels",
  "account_slug": "pixels",
  "user_id": "28d34402-d89a-4e2c-925c-50928684490e",
  "username": "f52026f583c24eb9a053f01fab5570e0",
  "is_owner": true,
  "permissions": {}
}
```

Connection successful.

## Step 3: Available Accounts

Called `/company-api/me/accounts/`. Response:

| # | Name   | Slug   | Owner |
|---|--------|--------|-------|
| 1 | Pixels | pixels | Yes   |

1 account available.

## Step 4: Available Tools

The `mcp__ceveto__*` tools are **not loaded** in this session (ToolSearch returned no results for `mcp__ceveto__`). This means the MCP server is either not connected in the current Claude Code session, or the tools have not been registered yet.

However, the OpenAPI schema at `/api/openapi.json` reports **466 endpoints** across **329 paths**, which would translate to approximately 466+ MCP tools when the server is fully connected (plus 3 meta tools: `whoami`, `list_accounts`, `switch_default_account`).

## Step 5: Summary

```
Ceveto MCP: Connected (API reachable, MCP tools not loaded in session)
Account:     Pixels (pixels)
Mode:        Local (uv)
Environment: Dev (localhost:8400)
Owner:       Yes
Accounts:    1
API Endpoints: 466 (across 329 paths)
MCP Tools:   0 registered in session (MCP server not connected)
```

### Troubleshooting

The API backend is running and responding correctly. The MCP tools are not available in this Claude Code session. To fix:

- Run `/mcp` to open the MCP dialog and check if the `ceveto` server is enabled
- Or restart the Claude Code session to re-initialize MCP connections

## Step 6: Connection State

Connection state should be saved to `~/.claude/plugins/data/ceveto/connection.json`:

```json
{
  "account_slug": "pixels",
  "account_name": "Pixels",
  "is_owner": true,
  "environment": "dev",
  "base_url": "https://localhost:8400",
  "last_checked": "2026-03-26T00:00:00Z",
  "tools_count": 0
}
```

Note: Directory creation for `~/.claude/plugins/data/ceveto/` was denied by sandbox permissions. The connection state file was not saved.
