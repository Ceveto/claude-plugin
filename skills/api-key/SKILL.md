---
name: api-key
description: Generate a Ceveto API key for MCP. Use when the user says "create api key", "generate key", "new api key", "rotate key", or needs credentials for MCP setup. Only works with local backend (dev mode).
user-invocable: true
allowed-tools: Bash, Read, Glob, AskUserQuestion
argument-hint: "[account-slug]"
---

# Generate Ceveto API Key

Creates an API key for MCP server usage. Requires a running local backend.

## Steps

### 1. Find backend

Look for `new-ceveto` directory:
- `./new-ceveto/manage.py`
- `../new-ceveto/manage.py`
- The current directory if it has `manage.py` + `ceveto_mcp/`

If not found, tell the user this only works with a local backend checkout.

### 2. Get account slug

If $ARGUMENTS is provided, use it as the account slug.

Otherwise, try to list available accounts:
```bash
cd <backend-path> && DB_PASSWORD=postgres uv run python manage.py create_mcp_key --help
```

Ask the user which account to create the key for.

### 3. Generate key

```bash
cd <backend-path> && DB_PASSWORD=postgres uv run python manage.py create_mcp_key <account-slug> --name "Claude MCP"
```

### 4. Show credentials

Display the output clearly:
- **Username**: the 32-char hex string
- **Private Key**: the 64-char hex string (shown once!)

Remind the user to save the private key — it cannot be shown again.

### 5. Offer to configure

Ask if the user wants to automatically update `.mcp.json` with the new credentials. If yes, update the `CEVETO_MCP_USERNAME` and `CEVETO_MCP_PRIVATE_KEY` values.

### 6. Set as owner (optional)

Ask if this API key should be account owner (full access) or have limited permissions. If owner:
```bash
cd <backend-path> && DJANGO_SETTINGS_MODULE=config.settings DB_PASSWORD=postgres uv run python -c "
import django; django.setup()
from user.models import AccountAccess, User
user = User.objects.get(username='<username>')
access = AccountAccess.objects.get(user=user)
access.is_owner = True
access.save(update_fields=['is_owner'])
print('Set as owner.')
"
```
