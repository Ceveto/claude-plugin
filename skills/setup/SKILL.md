---
name: setup
description: Set up Ceveto MCP server connection. Use when configuring Ceveto API access, generating API keys, or updating MCP credentials. Also use when the user says "setup ceveto", "configure ceveto", "connect to ceveto".
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
argument-hint: "[backend-path]"
---

# Configure Ceveto MCP Server

Set up the Ceveto MCP server so Claude Code can interact with the Ceveto API.

## Prerequisites

- Python 3.13+ with `uv` package manager
- A running Ceveto backend (new-ceveto)
- Access to an account in the Ceveto system

## Steps

### 1. Find the backend path

If $ARGUMENTS is provided, use it as the backend path. Otherwise, ask the user where their `new-ceveto` directory is located.

Verify the path exists and has `ceveto_mcp/` inside it:
```bash
ls <path>/ceveto_mcp/__main__.py
```

### 2. Check if MCP dependencies are installed

```bash
cd <path> && uv sync 2>&1 | tail -3
```

### 3. Generate API credentials

Ask the user which account to create the API key for. Run:

```bash
cd <path> && uv run python manage.py create_mcp_key <account-slug> --name "Claude MCP"
```

This outputs:
- `CEVETO_MCP_USERNAME` — the API user's username
- `CEVETO_MCP_PRIVATE_KEY` — the Ed25519 private key (shown once)
- `CEVETO_MCP_BASE_URL` — the backend URL

### 4. Ask for the base URL

The default is `https://localhost:8400`. Ask the user to confirm or change.

### 5. Write .mcp.json

Write the MCP configuration to the project root `.mcp.json`:

```json
{
  "mcpServers": {
    "ceveto": {
      "command": "uv",
      "args": ["run", "--directory", "<backend-path>", "python", "-m", "ceveto_mcp"],
      "env": {
        "CEVETO_MCP_BASE_URL": "<base-url>",
        "CEVETO_MCP_USERNAME": "<username>",
        "CEVETO_MCP_PRIVATE_KEY": "<private-key>"
      }
    }
  }
}
```

If `.mcp.json` already exists, merge the `ceveto` server into the existing config.

### 6. Add .mcp.json to .gitignore

Check if `.gitignore` exists and add `.mcp.json` if not already present (it contains secrets).

### 7. Verify

Tell the user to restart MCP via `/mcp` in Claude Code, then test with `whoami`.

## Important

- The private key is shown ONLY ONCE during generation. If lost, regenerate with `create_mcp_key`.
- `.mcp.json` contains secrets — never commit it to git.
- The backend must be running for MCP tools to work.
- The API user is created as account owner by default. Adjust permissions in the Django admin if needed.
