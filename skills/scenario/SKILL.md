---
name: scenario
description: Execute Ceveto business scenarios end-to-end using MCP tools. Use when the user wants to create tasks, onboard clients, set up maintenance workflows, handle emergencies, or any multi-step business process. Also trigger on "create a task", "new emergency repair", "onboard client", "set up workflow", "avārijas remonts", "izveidot uzdevumu", or any business scenario description.
user-invocable: true
argument-hint: "[scenario description or KB reference]"
---

# Ceveto Business Scenario Executor

Execute multi-step business scenarios by chaining MCP tools together.
This skill understands Ceveto's domain model and can create tasks,
contacts, locations, forms, and manage workflows.

## Before starting

### 1. Check permissions

Call `mcp__ceveto__whoami`. Parse `is_owner` and `permissions`.
Only proceed with operations the user has permission for.

### 2. Discover available tools

Use ToolSearch to find the tools needed for the scenario:
```
ToolSearch query: "mcp__ceveto__create"   → creation tools
ToolSearch query: "mcp__ceveto__list"     → lookup tools
ToolSearch query: "mcp__ceveto__transition" → status tools
```

Read tool descriptions to understand required parameters.

### 3. Check for KB scenarios

Look for scenario files in `CEVETO-KB/scenarios/` in the project root.
If the user's request matches a known scenario, follow that playbook.
If not, improvise using the domain model knowledge below.

## Ceveto Domain Model

The system has these key entities and their relationships:

```
Contact (client/vendor/supplier)
  └── Location (address, GPS)
        └── Asset (equipment at location)

TaskCategory (groups task types)
  └── TaskType (template: form, priority, gates)
        └── Task (actual work item)
              ├── assigned_to (team member)
              ├── client (contact)
              ├── location
              ├── TaskFormSubmission (filled form data)
              ├── TaskSignature (digital signatures)
              ├── TaskStatusTransition (audit trail)
              └── TaskComment (notes)

TaskCompletionGate (rules that block task completion)
  - signature_required
  - form_completed
  - approval_required

Task statuses: todo → in_progress → completed (or custom workflow)
Task priorities: low, medium, high, urgent, critical
```

## Scenario execution flow

### 1. Understand the request

Ask the user what they want to do. If vague, ask clarifying questions:
- "Who is the client?"
- "What type of work? (repair, inspection, installation)"
- "Where is the location?"
- "How urgent is this?"
- "Who should be assigned?"

### 2. Check existing data first

Before creating anything, search for existing records:
- Call `list_contacts` with search to find existing clients
- Call `list_locations` with search to find existing locations
- Call `list_task_types` to find matching task types

Show what was found and ask if the user wants to use existing records
or create new ones.

### 3. Create missing prerequisites

If task type, contact, or location don't exist, create them:

**Contact** (if new client):
```
create_contact(name="SIA Pārtikas Nams", type="client")
```

**Location** (if new site):
```
create_location(name="Rīga Centrs", address="Brīvības 120", client_id=<contact_uuid>)
```

**Task type** (if new workflow — rare, usually pre-configured):
```
create_task_type(name="Emergency Repair", code="EMERG-001", category_id=<cat_uuid>)
```

### 4. Create the task

```
create_task(
  title="Gaļas vitrīna neuztur temperatūru",
  task_type_id=<type_uuid>,
  priority="critical",
  description="Temperatūra +12°C, norma +2..+4°C",
  client_id=<contact_uuid>,
  location_id=<location_uuid>,
  assigned_to_id=<user_uuid>,
  due_date="2026-03-26"
)
```

### 5. Manage task lifecycle

Guide the user through status transitions:

```
transition_task(task_id=<uuid>, to_status="in_progress", notes="Tehniķis sācis darbu")
```

And when work is done:
```
transition_task(task_id=<uuid>, to_status="completed", notes="Remonts pabeigts")
```

If completion gates block, explain what's needed:
- "Task can't be completed — signature required"
- "Form must be submitted first"

### 6. Confirm and summarize

After each step, confirm what was created:
```
Created:
  ✓ Contact: SIA "Pārtikas Nams" (client)
  ✓ Location: Pārtikas Nams — Rīga Centrs
  ✓ Task: "Gaļas vitrīna KH-320 neuztur temperatūru"
    Priority: Critical
    Assigned to: Jānis Ozols
    Due: today
```

## Example scenarios

### Emergency repair
User: "iekārta sabojājusies pārtikas namā, vajag steidzamu remontu"
→ Search contact → search location → find task type → create task (critical)

### Scheduled maintenance
User: "jāplāno ikmēneša apkope visām iekārtām"
→ List locations → list assets → create tasks for each

### New client onboarding
User: "jauns klients Rīgas Piena Kombināts"
→ Create contact → create locations → assign default task types

### Inspection
User: "jāveic inspekcija Jelgavas noliktavā"
→ Find location → find inspection task type → create task → assign

## Language

The user may write in Latvian or English. Respond in the same language.
Ceveto is a Latvian product — many field values, names, and descriptions
will be in Latvian. This is normal.
