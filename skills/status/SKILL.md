---
name: status
description: Check Ceveto MCP connection status. Use when the user says "ceveto status", "is ceveto connected", "check connection", "which account", or wants to verify their MCP setup.
user-invocable: true
allowed-tools: Read, Bash, Glob, mcp__ceveto__whoami, mcp__ceveto__list_accounts
---

# Ceveto Connection Status

## Steps

### 1. Check .mcp.json config

Read `.mcp.json` from the project root. Determine:
- **Mode**: if has `command` key → Local, if has `url` key → Hosted
- **Environment**: parse `CEVETO_MCP_BASE_URL`:
  - `localhost` → Dev
  - `api.ceveto.dev` → Staging
  - `api.ceveto.com` → Production
- Show username (first 8 chars + `...`). Never show the private key.

If no `ceveto` server in `.mcp.json`, tell user to run `setup ceveto`.

### 2. Test connection via MCP tools

Call the `mcp__ceveto__whoami` tool. Parse the JSON response.

If it succeeds, extract:
- `account_name` and `account_slug`
- `is_owner` (true/false)
- `username`

If it fails with "No such tool available", MCP is not connected — tell user:
- Run `/mcp` to open MCP dialog and check if ceveto is enabled
- Or restart Claude Code session

If it fails with connection error, the backend is likely not running.

### 3. List accounts

Call `mcp__ceveto__list_accounts`. Show all available accounts with name, slug, and owner status.

### 4. Count available tools

Use ToolSearch to count `mcp__ceveto__*` tools:
```
ToolSearch query: "mcp__ceveto__"
```
Report the count.

### 5. Print summary

Format as a clean block:
```
Ceveto MCP: Connected ✓
Account:     {name} ({slug})
Mode:        Local (uv) / Hosted (SSE)
Environment: Dev / Staging / Production
Owner:       Yes / No
Accounts:    {count}
MCP Tools:   {count} registered
```

If something is wrong, show specific troubleshooting for the failure:
- "Backend not running" → `cd new-ceveto && make dev-backend`
- "MCP not connected" → `/mcp` restart
- "Invalid credentials" → `ceveto api-key` to regenerate
- "No permissions" → API user needs to be set as owner or given permissions

### 6. Save connection state

Save current connection info to `~/.claude/plugins/data/ceveto/connection.json`:
```json
{
  "account_slug": "pixels",
  "account_name": "Pixels",
  "is_owner": true,
  "environment": "dev",
  "base_url": "https://localhost:8400",
  "last_checked": "2026-03-26T22:00:00Z",
  "tools_count": 329
}
```

Create directory if needed: `mkdir -p ~/.claude/plugins/data/ceveto`

This lets other skills (explore, report) quickly know the current state
without re-calling whoami every time.
