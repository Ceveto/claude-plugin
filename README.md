# Ceveto Claude Plugin

Claude Code plugin for connecting to the Ceveto business management API via MCP (Model Context Protocol).

## Install

```bash
claude plugin marketplace add https://github.com/Ceveto/claude-plugin
claude plugin install ceveto
```

## Setup

After installation, run the setup skill in your Ceveto project:

```
/ceveto:setup
```

The skill will automatically find the backend, generate API credentials, and configure everything.

## What you get

Once configured, Claude Code gets direct access to the Ceveto API. Tools are dynamically generated from the backend's OpenAPI schema — every API endpoint becomes an MCP tool automatically.

Modules include: Contacts, Tasks, Locations, Assets (CMMS), Time Tracking (Traksy), Teams, Skills, Tags, Documents, Payments, and more.

## Requirements

- Python 3.13+ with [uv](https://docs.astral.sh/uv/)
- A running Ceveto backend
- An account in the system

## How it works

```
Claude Code → MCP Server (ceveto_mcp/) → Ed25519 signed HTTP → Dashboard API
```

1. The MCP server runs as a subprocess launched by Claude Code
2. On startup it fetches the OpenAPI schema from the backend
3. Each API endpoint becomes an MCP tool with proper parameter schemas
4. Tools are filtered by the API user's permissions — read-only users only see read tools
5. Condition limits (e.g. `max_amount`) are shown in tool descriptions

## Multi-account

One API key can access multiple accounts. The MCP server auto-selects if only one account is available, or you can switch with:

- `whoami` — see current account
- `list_accounts` — see all available accounts
- `switch_default_account` — change active account

## Security

- **Ed25519 signatures** — no shared secrets, cryptographic request signing
- **Permission-filtered tools** — only tools the API key has access to are registered
- **Method-level filtering** — read-only keys don't see write tools
- **Credentials in .mcp.json** — automatically added to .gitignore during setup
