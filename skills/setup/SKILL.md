---
name: setup
description: Set up or reconfigure Ceveto MCP connection. Use when the user says "setup ceveto", "configure ceveto", "connect to ceveto", "switch environment", "change to staging/production", or wants to modify their MCP connection.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
---

# Ceveto MCP Setup

## What this does

Sets up the Ceveto MCP connection so Claude Code can access the Ceveto API — contacts, tasks, locations, assets, time tracking, and more.

## Flow

### 1. Check existing config

Read `.mcp.json` in the current project root. If `ceveto` server already exists, show the current config and ask if the user wants to reconfigure or switch environment.

### 2. Choose connection mode

Ask the user:

**A) Local (recommended for developers)** — runs MCP server locally via `uvx`. Needs Python 3.12+ and uv.
**B) Hosted** — connects to `mcp.ceveto.com` or `mcp.ceveto.dev`. No local install needed.

### 3. Choose environment

Ask which environment:
- **Dev** — `https://localhost:8400` (local backend must be running)
- **Staging** — `https://api.ceveto.dev`
- **Production** — `https://api.ceveto.com`

### 4. Get credentials

Ask the user to paste their API credentials:
- **Username** — 32-char hex string from Ceveto dashboard (Settings → API Keys)
- **Private Key** — 64-char hex string (shown once when creating the key)

If the user doesn't have credentials yet, tell them:
1. Go to Ceveto dashboard → Settings → API Keys → Create
2. Save both the username and private key
3. Or for dev: run `cd new-ceveto && uv run python manage.py create_mcp_key <account-slug>`

### 5. Write config

**Local mode:**
```json
{
  "mcpServers": {
    "ceveto": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/Ceveto/mcp-server", "ceveto-mcp"],
      "env": {
        "CEVETO_MCP_BASE_URL": "<base-url>",
        "CEVETO_MCP_USERNAME": "<username>",
        "CEVETO_MCP_PRIVATE_KEY": "<private-key>"
      }
    }
  }
}
```

**Hosted mode:**
```json
{
  "mcpServers": {
    "ceveto": {
      "url": "<mcp-url>/sse"
    }
  }
}
```

MCP URLs:
- Staging: `https://mcp.ceveto.dev/sse`
- Production: `https://mcp.ceveto.com/sse`

If `.mcp.json` already exists, merge the `ceveto` entry — don't overwrite other servers.

### 6. Protect secrets

Add `.mcp.json` to `.gitignore` if not already there.

### 7. Verify

Tell the user to restart MCP with `/mcp` in Claude Code.

After restart, verify by calling MCP tools:

1. Call `mcp__ceveto__whoami` — should return account info
2. If successful, show: account name, slug, owner status
3. Call `mcp__ceveto__list_accounts` — show all available accounts
4. Try `mcp__ceveto__list_contacts` with limit=3 — verify data access

If `whoami` fails:
- "No such tool" → MCP not connected, try `/mcp` restart
- Connection error → backend not running (dev: `make dev-backend`)
- 401 → credentials invalid, regenerate with `ceveto api-key`
- 403 → API user has no permissions, set as owner

If verification succeeds, tell the user they can now:
- Say "explore ceveto data" to browse their data
- Say "ceveto report" for a business summary
- Ask anything about their contacts, tasks, locations etc.
