---
name: setup
description: Set up Ceveto MCP server connection. Use when configuring Ceveto API access, generating API keys, or updating MCP credentials. Also use when the user says "setup ceveto", "configure ceveto", "connect to ceveto".
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, AskUserQuestion
---

# Configure Ceveto MCP Server

Set up the Ceveto MCP server so Claude Code can interact with the Ceveto API.

## Steps

### 1. Find the backend

Look for the `new-ceveto` directory. Check these locations in order:
- `./new-ceveto/ceveto_mcp/__main__.py` (monorepo root)
- `./ceveto_mcp/__main__.py` (inside new-ceveto already)
- `../new-ceveto/ceveto_mcp/__main__.py` (sibling directory)

If not found, ask the user where their `new-ceveto` directory is.

### 2. Install dependencies

```bash
cd <backend-path> && uv sync 2>&1 | tail -3
```

### 3. Determine the backend URL

Check if the dev server is running. The default is `https://localhost:8400`. Ask the user to confirm.

### 4. Generate API credentials

Ask the user which account to use. To see available accounts:

```bash
cd <backend-path> && uv run python manage.py create_mcp_key --help
```

Then generate:

```bash
cd <backend-path> && uv run python manage.py create_mcp_key <account-slug> --name "Claude MCP"
```

Save the output — the private key is shown only once.

### 5. Write .mcp.json

Write to the project root `.mcp.json`:

```json
{
  "mcpServers": {
    "ceveto": {
      "command": "uv",
      "args": ["run", "--directory", "<backend-path>", "python", "-m", "ceveto_mcp"],
      "env": {
        "CEVETO_MCP_BASE_URL": "<base-url>",
        "CEVETO_MCP_USERNAME": "<username-from-output>",
        "CEVETO_MCP_PRIVATE_KEY": "<private-key-from-output>"
      }
    }
  }
}
```

If `.mcp.json` already exists, merge the `ceveto` server into existing config.

### 6. Protect secrets

Add `.mcp.json` to `.gitignore` if not already there.

### 7. Done

Tell the user to restart MCP with `/mcp` and test with the `whoami` tool.
