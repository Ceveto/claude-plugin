---
name: status
description: Check Ceveto MCP connection status. Use when the user says "ceveto status", "is ceveto connected", "check connection", "which account", or wants to verify their MCP setup.
user-invocable: true
allowed-tools: Read, Bash, Glob
---

# Ceveto Connection Status

## Steps

### 1. Check .mcp.json

Read `.mcp.json` from the project root. If no `ceveto` server configured, tell the user to run `/ceveto:setup`.

### 2. Show current config

Display:
- **Mode**: Local (uvx) or Hosted (SSE)
- **Environment**: Dev / Staging / Production (based on URL)
- **Base URL**: the API endpoint

For local mode, show the username (first 8 chars + ...).
Never show the private key.

### 3. Test connection

Try calling the `whoami` MCP tool. If it works, show:
- Account name and slug
- Whether the user is an owner
- Number of available MCP tools

If it fails, show troubleshooting tips:
- Backend not running (dev)
- Invalid credentials
- Network issues

### 4. Show available accounts

Call `list_accounts` to show all accounts this API key has access to.

### 5. Summary

Print a clean status summary:
```
Ceveto MCP: Connected
Account: Pixels (pixels)
Mode: Local (uvx)
Environment: Dev (localhost:8400)
Owner: Yes
Tools: 384 registered
```
