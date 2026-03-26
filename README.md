# Ceveto Claude Plugin

Connect Claude Code to your [Ceveto](https://ceveto.com) account. Claude gets direct access to your contacts, tasks, locations, assets, and more.

## Install

```bash
claude plugin marketplace add https://github.com/Ceveto/claude-plugin
claude plugin install ceveto
```

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| **Setup** | "setup ceveto" | Configure MCP connection (hosted or local) |
| **Status** | "ceveto status" | Check connection, account info, tool count |
| **Explore** | "show me contacts" | Browse and search business data via MCP |
| **Report** | "ceveto report" | Generate business summaries and analytics |
| **Switch Env** | "switch to staging" | Switch between dev/staging/production |
| **Scenario** | "avārijas remonts" | Execute business scenarios (tasks, clients, workflows) |
| **API Key** | "create api key" | Generate API key (dev mode only) |

## Quick Start

```
/ceveto:setup
```

Choose local or hosted mode, paste your API credentials, done.

## What you can do

Once connected, Claude can manage your Ceveto data:

- Contacts, tasks, locations, assets (CMMS)
- Time tracking, schedules, teams, skills
- Tags, documents, payments, and more

Tools are generated dynamically from the API — new features appear automatically.

## Environments

| Environment | API | MCP Server |
|-------------|-----|------------|
| Dev | `localhost:8400` | local (stdio) |
| Staging | `api.ceveto.dev` | `mcp.ceveto.dev` |
| Production | `api.ceveto.com` | `mcp.ceveto.com` |

Switch with `/ceveto:switch-env staging`.

## Multi-account

- `whoami` — see current account
- `list_accounts` — see all accounts
- `switch_default_account` — change active account

## Security

- **Ed25519 signatures** on every API request
- **OAuth 2.1** browser-based authorization
- **Permission-filtered tools** — only what your key can access
- **Credentials stay local** — `.mcp.json` auto-added to `.gitignore`
