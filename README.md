# Ceveto Claude Plugin

Connect Claude Code to your [Ceveto](https://ceveto.com) account. Claude gets direct access to your contacts, tasks, locations, assets, and more.

## Install

```bash
claude plugin marketplace add https://github.com/Ceveto/claude-plugin
claude plugin install ceveto
```

## Setup

1. Go to your Ceveto dashboard → **Settings → API Keys** → **Create**
2. Save the username and private key
3. In Claude Code, run:

```
/ceveto:setup
```

4. Paste your credentials when prompted
5. Restart MCP with `/mcp`

That's it — Claude can now access your Ceveto data.

## What you can do

Once connected, Claude can:

- List and manage contacts, tasks, locations
- View and update assets (CMMS)
- Track time entries and schedules
- Manage teams, skills, tags
- And more — all Ceveto modules are available

Tools are generated dynamically from the API — when Ceveto adds new features, your tools update automatically.

## Requirements

- [uv](https://docs.astral.sh/uv/) (Python package manager) — installed automatically by Claude Code
- A Ceveto account with API key access

## How it works

The plugin runs a lightweight MCP server locally via `uvx`. It connects to your Ceveto instance with Ed25519-signed requests, reads the API schema, and creates tools for each endpoint. Only tools you have permission to use are shown.

## Multi-account

If your API key has access to multiple accounts, use:

- `whoami` — see current account
- `list_accounts` — see all accounts
- `switch_default_account` — change active account

## Security

- **Ed25519 cryptographic signatures** on every API request
- **Permission-filtered tools** — only what your key can access
- **Credentials stay local** — stored only in `.mcp.json` (auto-added to `.gitignore`)
