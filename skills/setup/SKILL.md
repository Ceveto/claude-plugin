---
name: setup
description: Set up Ceveto MCP connection. Use when the user says "setup ceveto", "configure ceveto", "connect to ceveto", or wants to add Ceveto API access.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
---

# Configure Ceveto MCP Connection

Connect Claude Code to a Ceveto account via MCP.

## What you need

Get your API credentials from the Ceveto dashboard:
1. Go to **Settings → API Keys**
2. Click **Create API Key**
3. Save the **Username** and **Private Key** (shown only once)

## Steps

### 1. Ask for credentials

Ask the user for:
- **Username** — the API key username from their Ceveto dashboard
- **Private Key** — the Ed25519 private key (64-char hex string)
- **Base URL** — default is `https://api.ceveto.com`, confirm with user

### 2. Write .mcp.json

Write to the current project root `.mcp.json`:

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

If `.mcp.json` already exists, merge the `ceveto` entry into the existing `mcpServers` object.

### 3. Protect secrets

Add `.mcp.json` to `.gitignore` if not already there. This file contains credentials.

### 4. Done

Tell the user:
- Restart MCP with `/mcp` in Claude Code
- The `ceveto` server will appear with tools like `whoami`, `list_contacts`, etc.
- First time may take ~10 seconds while `uvx` downloads the package
