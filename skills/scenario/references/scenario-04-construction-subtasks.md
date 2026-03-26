# Scenārijs 4: Būvprojekts ar parent/subtask struktūru

## Situācijas apraksts

SIA "ElektroMeistars" ir uzņēmusies elektroinstalācijas pilnīgu nomaiņu trīsstāvu biroju ēkā. Projekts ilgs 3 nedēļas, iesaistīti 4 tehniķi, un darbs jāsadala pa stāviem un sistēmām. Projekta vadītājam Renāram vajag redzēt kopējo progresu un neļaut "pazust" apakšuzdevumiem.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Projektu vadītājs | Renārs Grīnbergs | Izveido struktūru, seko progresam |
| Dispečers | Dace Siliņa | Piešķir tehniķus, kontrolē termiņus |
| Tehniķi | Ivars, Guntis, Raitis, Toms | Izpilda apakšuzdevumus |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 4 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, uzdevuma tipu, parent uzdevumu "Elektroinstalācijas nomaiņa — Brīvības 55" ar 3 subtaskiem (demontāža pa stāviem), tehniķa lietotāju. Pēc tam vari pāriet uz [5. solis — Darba izpilde](#5-solis--darba-izpilde-tehniķi).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Priekšnosacījumi (vienreiz)

> **UI:** Ja vēl nav:
>
> 1. Izveido kategoriju **CMMS Maintenance** (skat. S1, 1. solis)
> 2. Izveido uzdevuma tipu — šoreiz var izmantot esošo vai jaunu:
>
> Sidebar → **Tasks** → **Task Types** → spied **"+"**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | HVAC mēneša profilakse |
> | Code | PM-HVAC-001 |
> | Category | CMMS Maintenance |
> | Default Priority | Medium |
>
> Spied **"Create"**.

---

### 2. solis — Lokācijas izveide (ja vēl nav)

> **UI:** Sidebar → **Locations** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Biroju ēka — Brīvības 55 |
> | Address | Brīvības iela 55, Rīga |
>
> Spied **"Create"**.

---

### 3. solis — Parent uzdevuma izveide (Renārs)

Renārs izveido galveno projekta uzdevumu.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Elektroinstalācijas nomaiņa — Brīvības 55 |
> | Task Type | HVAC mēneša profilakse *(vai cits atbilstošs tips)* |
> | Priority | High |
> | Location | Biroju ēka — Brīvības 55 |
> | Description | Pilnīga elektroinstalācijas nomaiņa 3 stāvos ar jaunu sadales skapja uzstādīšanu. |
> | Due date | 2026-04-18 |
>
> Spied **"Create task"**.

**Ko sistēma dara:** Izveido `Task` ar statusu `todo` un prioritāti `high`. Šis būs parent uzdevums, zem kura grupēs subtaskus.

---

### 4. solis — Subtasku izveide (Renārs)

Renārs atver parent uzdevumu un izveido 3 apakšuzdevumus — pa vienam katram stāvam.

> **UI:** Tasks sarakstā klikšķini uz **"Elektroinstalācijas nomaiņa"** → Detail Drawer
>
> **Overview** tab → **Subtasks** sekcija → spied **"Add Subtask"** pogu
>
> **Subtask 1:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Demontāža — 1. stāvs |
> | Assigned to | Ivars |
>
> Spied **"Create"**.
>
> **Subtask 2:** (spied **"Add Subtask"** vēlreiz)
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Demontāža — 2. stāvs |
> | Assigned to | Guntis |
>
> Spied **"Create"**.
>
> **Subtask 3:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Demontāža — 3. stāvs |
> | Assigned to | Raitis |
>
> Spied **"Create"**.

**Ko sistēma dara:**
- Izveido 3 `Task` objektus ar `parent=parent_task`.
- Katrs subtask mantojumā saņem `task_type` un `location` no parent.
- Parent uzdevuma **Subtasks** sekcijā parādās 3 ieraksti ar statusu "Todo".
- Subtask count badge: **3 subtasks**.

---

### 5. solis — Darba izpilde (Tehniķi)

Ivars, Guntis un Raitis paralēli strādā savos stāvos.

> **UI:** Katrs tehniķis atver savu subtasku:
>
> Tasks sarakstā filtrē → var redzēt subtaskus vai atvērt parent → Subtasks sekcija → klikšķini uz subtaska
>
> **Ivars (1. stāvs):**
> 1. Detail Drawer → Overview tab → spied **"Start"** → statuss `in_progress`
> 2. Veic demontāžas darbus
> 3. Form tab → aizpilda check-list (ja uzdevuma tipam ir forma)
> 4. Overview tab → spied **"Complete"** → statuss `completed`
>
> **Guntis (2. stāvs):** Tāds pats process.
>
> **Raitis (3. stāvs):** Tāds pats process.

**Ko sistēma dara:**
- Katram subtaskam atsevišķi `TaskStatusTransition` un `TaskStatusInterval`.
- Kad subtask tiek pabeigts, parent **Subtasks** sekcijā statuss mainās uz "Completed" (zaļš).
- Parent uzdevuma progress: redzams kā "1/3", "2/3", "3/3 subtaski pabeigti".

---

### 6. solis — Parent uzdevuma progresu novērošana (Renārs)

Renārs jebkurā brīdī var atvērt parent uzdevumu un redzēt kopējo statusu.

> **UI:** Detail Drawer → parent uzdevums
>
> **Overview tab → Subtasks sekcija:**
>
> | Subtask | Status | Assigned to |
> |---------|--------|-------------|
> | Demontāža — 1. stāvs | ✅ Completed | Ivars |
> | Demontāža — 2. stāvs | ✅ Completed | Guntis |
> | Demontāža — 3. stāvs | 🔄 In Progress | Raitis |
>
> Progress: **2/3 subtaski pabeigti**

---

### 7. solis — Parent uzdevuma pabeigšana (Renārs)

Kad visi 3 subtaski ir completed, Renārs var pabeigt parent uzdevumu.

> **UI:** Detail Drawer → parent uzdevums → **Overview** tab
>
> Subtasks sekcija:
> - ✅ Demontāža — 1. stāvs
> - ✅ Demontāža — 2. stāvs
> - ✅ Demontāža — 3. stāvs
>
> Spied **"Complete"** → statuss mainās uz `completed`.
>
> **Comments** tab → Renārs pievieno kopsavilkumu:
> ```
> Projekts pabeigts termiņā. Visi 3 stāvi demontēti. Kopējais darba laiks: ~24h.
> ```

**Ko sistēma dara:**
- Pārbauda, vai visi subtaski ir `completed`.
- `TaskStatusTransition` parent uzdevumam.
- `task.completed_at` iestatīts.
- Pilns audit trail pa visiem subtaskiem pieejams caur parent.

---

### 8. solis — Projekta pārskats (Dace)

Dace var pārskatīt visu projektu.

> **UI:** Detail Drawer → parent uzdevums → katrs tab:
>
> **Overview tab:**
> - Status: Completed, Priority: High
> - 3/3 subtaski pabeigti
>
> **Time tab:**
> - Kopējais laiks pa visiem subtaskiem
> - Pa tehniķiem: Ivars Xh, Guntis Xh, Raitis Xh
>
> **Activity tab:**
> - Pilna hronoloģija: parent created → subtaski created → katrs subtask started → completed → parent completed

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Projekta struktūra | Excel tabula vai galvā | Hierarhiski uzdevumi (parent + subtaski) |
| Progress tracking | "Cik tālu esat?" pa telefonu | Reāllaika statuss: 2/3 gatavs |
| Darba laiks pa tehniķiem | Aptuvenas aplēses | Precīzi intervāli katram cilvēkam |
| Pabeigšanas kvalitāte | Var "aizmirst" subtasku | Parent nevar aizvērt, kamēr nav viss gatavs |
| Atskaitīšanās | Bieži "kur pazuda tas uzdevums?" | Viss redzams parent uzdevuma skatā |

---

## Izmantotās sistēmas funkcijas

- **Parent-subtask relationships** — hierarhiska uzdevumu struktūra
- **Task.parent** — FK uz parent uzdevumu
- **Subtask count / progress** — automātiski aprēķināts
- **TaskParticipant** — vairāki tehniķi uz vienu projektu
- **TaskStatusInterval** — laika uzskaite pa personām
- **TaskEvent** — audit trail katram subtaskam
- **Prioritātes badge** — HIGH/CRITICAL vizuāli izcelti

---

*Saistītās idejas: 19, 24, 27*
