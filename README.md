# Ceveto Claude Plugin

Connect Claude Code to your [Ceveto](https://ceveto.com) account. Claude gets direct access to your contacts, tasks, locations, assets, and more.

## Install

```bash
claude plugin marketplace add https://github.com/Ceveto/claude-plugin
claude plugin install ceveto
```

## Setup

In Claude Code, run:

```
/ceveto:setup
```

Choose **Hosted** (recommended) or **Local** mode, then follow the prompts.

## Connection Modes

### Hosted (recommended)

Connects to `mcp.ceveto.com` — no local install needed. Just your API credentials.

### Local

Runs MCP server locally via `uvx`. Requires Python 3.12+ and [uv](https://docs.astral.sh/uv/).

## What you can do

Once connected, Claude can manage your Ceveto data:

- Contacts, tasks, locations, assets (CMMS)
- Time tracking, schedules, teams, skills
- Tags, documents, payments, and more

Tools are generated dynamically from the API — new features appear automatically.

## Multi-account

- `whoami` — see current account
- `list_accounts` — see all accounts
- `switch_default_account` — change active account

## Security

- **Ed25519 signatures** on every API request
- **OAuth 2.1** browser-based authorization
- **Permission-filtered tools** — only what your key can access
- **Credentials stay local** — `.mcp.json` auto-added to `.gitignore`
