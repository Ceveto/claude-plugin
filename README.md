# Ceveto Claude Plugin

Claude Code plugin for connecting to the Ceveto business management API via MCP (Model Context Protocol).

## Install

```bash
claude plugin install https://github.com/Ceveto/claude-plugin
```

## Setup

After installation, run the setup skill:

```
/ceveto:setup /path/to/new-ceveto
```

This will:
1. Generate Ed25519 API credentials
2. Configure `.mcp.json` for your project
3. Add `.mcp.json` to `.gitignore`

## What you get

Once configured, Claude Code gets direct access to the Ceveto API with tools for:

- **Contacts** — list, create, update, delete
- **Tasks** — list, create, update, transition status
- **Locations** — list, create, update, delete
- **Assets** — CMMS asset management
- **Time tracking** — Traksy time entries and schedules
- **Teams & Skills** — team management
- **Tags, Documents, Payments** — and more

Tools are dynamically generated from the backend's OpenAPI schema — any new API endpoint automatically becomes available.

## Requirements

- Python 3.13+ with `uv`
- A running Ceveto backend (`new-ceveto`)
- An account in the system

## How it works

The MCP server (`ceveto_mcp/`) runs as a subprocess launched by Claude Code. It:

1. Connects to the backend via Ed25519-signed HTTP requests
2. Reads the OpenAPI schema at startup
3. Generates MCP tools for each API endpoint
4. Filters tools by the API user's permissions

## Security

- Ed25519 public-key cryptography (no shared secrets)
- Per-user permissions — tools filtered by what the API key can access
- `.mcp.json` contains credentials and should never be committed
