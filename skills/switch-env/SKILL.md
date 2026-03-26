---
name: switch-env
description: Switch Ceveto MCP between environments. Use when the user says "switch to staging", "switch to production", "change environment", "use dev", "point to staging".
user-invocable: true
allowed-tools: Read, Write, Edit, AskUserQuestion
argument-hint: "[dev|staging|production]"
---

# Switch Ceveto Environment

Quickly switch the MCP connection between dev, staging, and production.

## Steps

### 1. Read current config

Read `.mcp.json` from the project root. If no `ceveto` server, tell user to run `/ceveto:setup` first.

### 2. Determine target environment

If $ARGUMENTS is provided (e.g. "staging"), use that. Otherwise ask:
- **dev** — `https://localhost:8400`
- **staging** — `https://api.ceveto.dev`
- **production** — `https://api.ceveto.com`

### 3. Update config

For **local mode** (uvx): update `CEVETO_MCP_BASE_URL` in the env section.

For **hosted mode** (SSE): update the URL:
- Staging: `https://mcp.ceveto.dev/sse`
- Production: `https://mcp.ceveto.com/sse`
- Dev: switch to local mode (hosted not available for dev)

### 4. Warn about credentials

API keys are per-environment. If switching environments, the user may need different credentials. Check if their current key works by reminding them:

"API keys are per-environment. Your current key was created on [old env]. You may need to create a new key on [new env]."

### 5. Restart

Tell the user to restart MCP with `/mcp` to apply changes.
