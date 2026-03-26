---
name: setup
description: Set up Ceveto MCP connection. Use when the user says "setup ceveto", "configure ceveto", "connect to ceveto", or wants to add Ceveto API access.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
---

# Configure Ceveto MCP Connection

Connect Claude Code to a Ceveto account via MCP.

## Steps

### 1. Ask the user how they want to connect

Two options:

**A) Hosted (recommended)** — connect to `mcp.ceveto.com`. No local install needed.
**B) Local** — run MCP server locally via `uvx`. Needs Python + uv.

### 2A. Hosted setup

Ask for API credentials from the Ceveto dashboard (Settings → API Keys → Create):
- **Username**
- **Private Key**

Write to `.mcp.json`:

```json
{
  "mcpServers": {
    "ceveto": {
      "url": "https://mcp.ceveto.com/sse"
    }
  }
}
```

Note: In hosted mode, the user authenticates by calling the `connect` tool
with their OAuth access token after connecting. The MCP server handles
auth per-session.

For now (before OAuth browser flow), users can also use local mode.

### 2B. Local setup

Ask for:
- **Username** — API key username
- **Private Key** — Ed25519 private key (64-char hex)
- **Base URL** — default `https://api.ceveto.com`

Write to `.mcp.json`:

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

If `.mcp.json` already exists, merge the `ceveto` entry into existing `mcpServers`.

### 3. Protect secrets

Add `.mcp.json` to `.gitignore` if not already there.

### 4. Done

Tell the user to restart MCP with `/mcp` in Claude Code.
