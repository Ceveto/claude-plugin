---
name: scenario
description: Create tasks, manage work, and execute business processes in Ceveto via MCP. Triggers on "create task", "new task", "izveidot uzdevumu", "jauns uzdevums", "fix", "repair", "remonts", "apkope", "inspekcija", "onboard client", any request involving creating or managing work items, or when the user describes a problem that needs a task.
user-invocable: true
argument-hint: "[what needs to be done]"
---

# Ceveto Task & Workflow Manager

Help users create and manage tasks by asking the right questions,
looking up existing data, and filling in the gaps.

## Key principle

The user will give you minimal info like "uztaisi tasku Pārtikas Namam,
vitrīna sabojājusies". Your job is to:
1. Figure out what you already know from the request
2. Look up existing data to fill gaps
3. Ask only what you can't find yourself
4. Create everything needed

## Step 1: Parse what the user gave you

Extract whatever you can from the request:
- **Client hint** — any company name, person name, or reference
- **Problem/work** — what needs to be done
- **Location hint** — any address, site name
- **Urgency hint** — words like "steidzami", "urgent", "critical", "ASAP"
- **Assignee hint** — any person mentioned for the work

Most fields will be missing — that's fine.

## Step 2: Check connection and permissions

Call `mcp__ceveto__whoami`. If not connected, tell user to set up first.
Check `is_owner` and `permissions` — need at least `tasks: write`.

## Step 3: Look up existing data (don't ask if you can find it)

For each hint from step 1, search via MCP tools:

**Client:**
```
list_contacts(search="Pārtikas") → find matching contact
```
If found → use it, don't ask.
If multiple matches → show options, ask which one.
If not found → ask: "Klientu '{name}' neatradu. Izveidot jaunu? Kāds tips — client, vendor, supplier?"

**Location:**
```
list_locations(search="Pārtikas") → find matching location
```
If found → use it.
If not found and client was found → ask: "Kur tieši? Adrese?"

**Task type:**
```
list_task_types(limit=50) → get all available types
```
Show the list and ask: "Kāds uzdevuma tips? Pieejamie:" + list names.
If the user's description clearly matches one (e.g. "remonts" → repair type), suggest it.

**Assignee:**
```
list_account_accesses(limit=50) → get team members
```
If user mentioned someone → search for them.
If not → ask: "Kam piešķirt? Pieejamie:" + list names.
Or suggest: "Vai atstāt nepiešķirtu?"

## Step 4: Ask only what's missing

After searching, you should have most fields. Ask about the rest in ONE message:

"Lūdzu precizē:
- **Uzdevuma tips**: [suggest best match] — vai cits?
- **Prioritāte**: medium (vai critical, jo tu minēji 'steidzami')?
- **Termiņš**: šodien?
- **Apraksts**: [paraphrase what user said] — vai papildināt?"

Let the user confirm or change. Don't ask one question at a time.

## Step 5: Create everything

Create in order (dependencies first):

1. **Contact** (if new): `create_contact(name=..., type="client")`
2. **Location** (if new): `create_location(name=..., address=..., client_id=...)`
3. **Task**: `create_task(title=..., task_type_id=..., priority=..., client_id=..., location_id=..., assigned_to_id=..., description=..., due_date=...)`

## Step 6: Confirm

Show what was created:

```
Uzdevums izveidots:
  Nosaukums: Gaļas vitrīna KH-320 neuztur temperatūru
  Tips: Aukstuma iekārtu avārija
  Prioritāte: Critical
  Klients: SIA "Pārtikas Nams"
  Lokācija: Pārtikas Nams — Rīga Centrs
  Piešķirts: Jānis Ozols
  Termiņš: šodien

Vai vēlies ko mainīt?
```

## Step 7: Follow-up actions

After creating, offer relevant next steps:
- "Mainīt statusu uz 'in progress'?"
- "Pievienot komentāru?"
- "Izveidot vēl vienu līdzīgu uzdevumu?"

## Language

Respond in the same language the user uses. Latvian is default for
Ceveto users. Field values (names, descriptions) should match the
user's language.

## Common patterns

**User says:** "tasku par remontu pārtikas namam"
→ search "pārtikas" in contacts → found → search locations for that client
→ list task types → suggest repair type → ask priority + assignee → create

**User says:** "new task for Nordic Equipment, compressor inspection"
→ search "Nordic" in contacts → found → search locations → list task types
→ suggest inspection type → ask assignee → create

**User says:** "steidzami, iekārta nestrādā Jelgavā"
→ urgency=critical → search locations with "Jelgava" → ask client
→ list task types → suggest emergency → create with critical priority
