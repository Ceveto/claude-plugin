# Scenārijs 8: Dažādi klienti, viens servisa process — formu un workflow pielāgojumi

## Situācijas apraksts

SIA "UniServis" apkalpo trīs dažādus klientus ar atšķirīgām prasībām vienai un tai pašai pakalpojumu kategorijai — kondicionēšanas iekārtu apkopei:

- **"MiniMart"** — mazumtirdzniecības ķēde. Vienkāršas prasības: pārbaudīt, novērtēt, parakstīt.
- **"PharmaLV"** — farmācijas ražotne. Stingras prasības: detalizēta forma, GMP atbilstība, QA review process.
- **"BizPark"** — biroju centrs. Vidējas prasības: standarta forma + ēkas pārvaldnieka apstiprinājums.

Iepriekš UniServis uzturēja 3 dažādas Excel veidlapas un 3 dažādus procesus. Tas bija neefektīvi un radīja kļūdas.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Servisa administrators | Elīna Bergmane | Konfigurē formas, workflow un task types |
| Dispečers | Toms Jēgers | Izveido uzdevumus — sistēma pati rezolvē konfigurāciju |
| Tehniķi | Dažādi | Aizpilda atbilstošās formas |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 8 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, 3 formas (MiniMart 2-lauku, PharmaLV 7-lauku, BizPark 4-lauku), 3 uzdevumu tipus, 2 workflows (bāzes + PharmaLV ar QA), 3 kontaktus un 3 uzdevumus. Pēc tam vari pāriet uz [6. solis — Uzdevumu izveide](#6-solis--uzdevumu-izveide-toms).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Kategorijas izveide (vienreiz)

> **UI:** Sidebar → **Tasks** → **Task Categories** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | CMMS Maintenance |
>
> Spied **"Create"**.

---

### 2. solis — Formu izveide (Elīna, vienreiz)

Elīna izveido 3 dažādas formas — pa vienai katram klientam.

#### MiniMart forma (vienkārša — 2 lauki)

> **UI:** Sidebar → **Tasks** → **Forms** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | MiniMart — kondicionēšanas apkope |
> | Code | FORM-MINI-AC-001 |
>
> Sections (JSON):
> ```json
> [
>   {
>     "id": "general",
>     "name": "Vispārīgi",
>     "fields": [
>       {"id": "condition", "label": "Stāvoklis", "type": "select"},
>       {"id": "temp", "label": "Temperatūra", "type": "number"}
>     ]
>   }
> ]
> ```
>
> Spied **"Create"**, tad **"Publish"**.

#### PharmaLV forma (paplašināta — 7 lauki, 3 sekcijas)

> **UI:** Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | PharmaLV — GMP kondicionēšanas apkope |
> | Code | FORM-PHARMA-AC-001 |
>
> Sections (JSON):
> ```json
> [
>   {
>     "id": "general",
>     "name": "Vispārīgi",
>     "fields": [
>       {"id": "condition", "label": "Stāvoklis", "type": "select"},
>       {"id": "temp", "label": "Temperatūra", "type": "number"}
>     ]
>   },
>   {
>     "id": "gmp",
>     "name": "GMP atbilstība",
>     "fields": [
>       {"id": "cleanroom_class", "label": "Tīrtelpas klase", "type": "select"},
>       {"id": "pressure_diff", "label": "Spiediena diferenciālis", "type": "number"},
>       {"id": "particle_count", "label": "Daļiņu skaits", "type": "number"}
>     ]
>   },
>   {
>     "id": "calibration",
>     "name": "Kalibrēšana",
>     "fields": [
>       {"id": "cert_number", "label": "Sertifikāta nr.", "type": "text"},
>       {"id": "deviation", "label": "Novirze", "type": "number"}
>     ]
>   }
> ]
> ```
>
> Spied **"Create"**, tad **"Publish"**.

#### BizPark forma (vidēja — 4 lauki, 2 sekcijas)

> **UI:** Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | BizPark — biroja HVAC apkope |
> | Code | FORM-BIZPARK-AC-001 |
>
> Sections (JSON):
> ```json
> [
>   {
>     "id": "general",
>     "name": "Vispārīgi",
>     "fields": [
>       {"id": "condition", "label": "Stāvoklis", "type": "select"},
>       {"id": "temp", "label": "Temperatūra", "type": "number"}
>     ]
>   },
>   {
>     "id": "office",
>     "name": "Biroja specifika",
>     "fields": [
>       {"id": "noise_level", "label": "Trokšņa līmenis", "type": "number"},
>       {"id": "air_quality", "label": "Gaisa kvalitāte", "type": "select"}
>     ]
>   }
> ]
> ```
>
> Spied **"Create"**, tad **"Publish"**.

**Ko sistēma dara:** Izveido 3 `TaskForm` objektus ar statusu `published`. Katra forma ir pielāgota klienta specifiskajām prasībām — no vienkāršas 2-lauku formas līdz detalizētai 7-lauku GMP formai.

---

### 3. solis — Uzdevumu tipu izveide (Elīna, vienreiz)

Elīna izveido 3 uzdevumu tipus — katram klientam savu ar atbilstošo formu.

> **UI:** Sidebar → **Tasks** → **Task Types** → spied **"+"** pogu
>
> **MiniMart tips:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | MiniMart HVAC apkope |
> | Code | MAINT-MINI-AC-001 |
> | Category | CMMS Maintenance |
> | Base Form | MiniMart — kondicionēšanas apkope *(dropdown)* |
> | Default Priority | Medium |
>
> Spied **"Create"**.
>
> **PharmaLV tips:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | PharmaLV GMP apkope |
> | Code | MAINT-PHARMA-AC-001 |
> | Category | CMMS Maintenance |
> | Base Form | PharmaLV — GMP kondicionēšanas apkope *(dropdown)* |
> | Default Priority | High |
>
> Spied **"Create"**.
>
> **BizPark tips:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | BizPark biroja HVAC |
> | Code | MAINT-BIZPARK-AC-001 |
> | Category | CMMS Maintenance |
> | Base Form | BizPark — biroja HVAC apkope *(dropdown)* |
> | Default Priority | Medium |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido 3 `TaskType` objektus, katrs ar savu formu un prioritāti. Kad dispečers izveidos uzdevumu un izvēlēsies tipu — sistēma automātiski ielādēs pareizo formu.

---

### 4. solis — Workflow izveide (Elīna, vienreiz)

Elīna izveido 2 workflows — bāzes (vienkāršu) un PharmaLV (ar QA soli).

> **UI:** Sidebar → **Tasks** → **Workflows** → spied **"+"** pogu
>
> **Bāzes workflow (globāls):**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Bāzes workflow |
> | Scope | global |
>
> Statuses:
> ```json
> [
>   {"id": "todo", "name": "To Do", "canonical": "todo"},
>   {"id": "in_progress", "name": "In Progress", "canonical": "in_progress"},
>   {"id": "completed", "name": "Completed", "canonical": "completed"}
> ]
> ```
>
> Transitions:
> ```json
> [
>   {"from": "todo", "to": "in_progress"},
>   {"from": "in_progress", "to": "completed"}
> ]
> ```
>
> Spied **"Create"**.
>
> **PharmaLV workflow (ar QA):**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | PharmaLV workflow ar QA |
> | Scope | task_type |
> | Task Type | PharmaLV GMP apkope *(dropdown)* |
>
> Statuses:
> ```json
> [
>   {"id": "todo", "name": "To Do", "canonical": "todo"},
>   {"id": "in_progress", "name": "In Progress", "canonical": "in_progress"},
>   {"id": "awaiting_qa", "name": "Awaiting QA", "canonical": "paused"},
>   {"id": "completed", "name": "Completed", "canonical": "completed"},
>   {"id": "rework", "name": "Rework Required", "canonical": "in_progress"}
> ]
> ```
>
> Transitions:
> ```json
> [
>   {"from": "todo", "to": "in_progress"},
>   {"from": "in_progress", "to": "awaiting_qa"},
>   {"from": "awaiting_qa", "to": "completed"},
>   {"from": "awaiting_qa", "to": "rework"},
>   {"from": "rework", "to": "in_progress"}
> ]
> ```
>
> Spied **"Create"**.

**Ko sistēma dara:**
- **Bāzes workflow** (`global` scope) — attiecas uz visiem tipiem, kam nav specifiska workflow.
- **PharmaLV workflow** (`task_type` scope) — attiecas tikai uz "PharmaLV GMP apkope" tipu.
- Kanonisko statusu kartēšana (`canonical` lauks) nodrošina, ka analītika ir salīdzināma neatkarīgi no klienta specifiskā procesa. Piemēram, `awaiting_qa` → `paused`, `rework` → `in_progress`.

---

### 5. solis — Klientu kontaktu izveide (ja vēl nav)

> **UI:** Sidebar → **Contacts** → spied **"+"** pogu
>
> Izveido 3 kontaktus:
>
> | Name | Type |
> |------|------|
> | MiniMart SIA | Client |
> | PharmaLV SIA | Client |
> | BizPark SIA | Client |
>
> Spied **"Create"** katram.

---

### 6. solis — Uzdevumu izveide (Toms)

Toms izveido uzdevumus — vienkārši izvēlas uzdevuma tipu un klientu. Sistēma pati piemēro pareizo formu un workflow.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> **Uzdevums A — MiniMart:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Kondicionēšanas apkope — MiniMart SIA |
> | Task Type | MiniMart HVAC apkope *(dropdown)* |
> | Client | MiniMart SIA |
> | Priority | Low |
> | Location | *(izvēlies)* |
>
> Spied **"Create task"**.
>
> → Sistēma ielādē 2-lauku formu, bāzes workflow (todo→ip→completed).
>
> **Uzdevums B — PharmaLV:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Kondicionēšanas apkope — PharmaLV SIA |
> | Task Type | PharmaLV GMP apkope *(dropdown)* |
> | Client | PharmaLV SIA |
> | Priority | High |
> | Location | *(izvēlies)* |
>
> Spied **"Create task"**.
>
> → Sistēma ielādē 7-lauku GMP formu, PharmaLV workflow (ar awaiting_qa soli).
>
> **Uzdevums C — BizPark:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Kondicionēšanas apkope — BizPark SIA |
> | Task Type | BizPark biroja HVAC *(dropdown)* |
> | Client | BizPark SIA |
> | Priority | Medium |
> | Location | *(izvēlies)* |
>
> Spied **"Create task"**.
>
> → Sistēma ielādē 4-lauku biroja formu, bāzes workflow.

**Ko sistēma dara:** Katram uzdevumam automātiski piemēro atbilstošo formu un workflow no task type. **Dispečerim nav jādomā par konfigurāciju** — viņš vienkārši izvēlas tipu un klientu.

---

### 7. solis — MiniMart uzdevuma izpilde (vienkāršā plūsma)

> **UI:** Tasks sarakstā atver MiniMart uzdevumu → Detail Drawer
>
> 1. **Overview** tab → spied **"Start"** → `in_progress`
> 2. **Form** tab → redzēsi vienkāršo formu ar 2 laukiem:
>    - Stāvoklis → "Labi"
>    - Temperatūra → 21.5
>    - Spied **"Submit Form"**
> 3. **Overview** tab → spied **"Complete"** → `completed`
>
> Plūsma: `todo → in_progress → completed` (2 soļi)

---

### 8. solis — PharmaLV uzdevuma izpilde (QA plūsma)

> **UI:** Tasks sarakstā atver PharmaLV uzdevumu → Detail Drawer
>
> 1. **Overview** tab → spied **"Start"** → `in_progress`
>
> 2. **Form** tab → redzēsi paplašināto formu ar 3 sekcijām:
>
>    **Vispārīgi:**
>    - Stāvoklis → "Labi"
>    - Temperatūra → 20.8
>
>    **GMP atbilstība:**
>    - Tīrtelpas klase → "C"
>    - Spiediena diferenciālis → 12.5
>    - Daļiņu skaits → 3200
>
>    **Kalibrēšana:**
>    - Sertifikāta nr. → "CAL-2026-0443"
>    - Novirze → 0.3
>
>    Spied **"Submit Form"**.
>
> 3. **Overview** tab → allowed transitions rāda **"Awaiting QA"** (nevis Complete!)
>    Spied **"Awaiting QA"** → statuss mainās uz `awaiting_qa`
>
> 4. QA speciālists pārbauda datus. Ja viss ok:
>    **Overview** tab → spied **"Complete"** → `completed`
>
>    Ja jāpārstrādā:
>    **Overview** tab → spied **"Rework Required"** → `rework` → tehniķis labo → **"In Progress"** → utt.
>
> Plūsma: `todo → in_progress → awaiting_qa → completed` (3 soļi + iespējamais rework cikls)

**Ko sistēma dara:**
- Workflow nosaka, kādas statusa pārejas ir pieejamas.
- PharmaLV uzdevumam pēc `in_progress` nav pieejams `completed` — tikai `awaiting_qa`.
- `canonical` kartēšana: `awaiting_qa` = `paused` (sistēma zina, ka darbs gaida ārēju darbību).

---

### 9. solis — BizPark uzdevuma izpilde (vidējā plūsma)

> **UI:** Tasks sarakstā atver BizPark uzdevumu → Detail Drawer
>
> 1. **Overview** tab → spied **"Start"** → `in_progress`
> 2. **Form** tab → redzēsi 4-lauku formu:
>    - Stāvoklis → "Labi"
>    - Temperatūra → 22.1
>    - Trokšņa līmenis → 35
>    - Gaisa kvalitāte → "Laba"
>    - Spied **"Submit Form"**
> 3. **Overview** tab → spied **"Complete"** → `completed`

---

### 10. solis — Salīdzinošā analītika (Elīna)

Elīna var apskatīt vienotu analītiku pa visiem klientiem, pateicoties kanonisko statusu kartēšanai.

> **UI:** Tasks sarakstā var filtrēt un salīdzināt:
>
> | Klients | Task Type | Forma | Workflow | Cycle Time |
> |---------|-----------|-------|----------|------------|
> | MiniMart | MiniMart HVAC | 2 lauki | bāzes | ~1.5h |
> | PharmaLV | PharmaLV GMP | 7 lauki | ar QA | ~6.8h |
> | BizPark | BizPark HVAC | 4 lauki | bāzes | ~3.4h |
>
> PharmaLV ilgāks cikla laiks ir sagaidāms — QA review process pievieno gaidīšanu, bet aktīvais darba laiks ir līdzīgs visiem trim.

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Formu pārvaldība | 3 dažādi Excel/Word faili | 3 formas sistēmā, katrai savs task type |
| Jauna klienta pievienošana | Jāizveido jauna veidlapa no nulles | Jauna forma + task type — gatavs |
| Dispečera darbs | Jāatceras, kuram klientam kāda forma | Izvēlies task type — sistēma pati rezolvē |
| Analītika | Nesalīdzināmi dati starp klientiem | Kanoniskie statusi ļauj salīdzināt |
| Procesa izmaiņas | Jāpārtaisa visas veidlapas | Maina formu — visi jaunie uzdevumi saņem atjauninājumu |
| QA process | Manuāla e-pasta koordinācija | Workflow statuss `awaiting_qa` ar automātisku pāreju |

---

## Izmantotās sistēmas funkcijas

- **TaskForm** — 3 dažādas formas ar atšķirīgu sarežģītību
- **TaskType** — katram klientam savs tips ar piesaistītu formu
- **TaskStatusWorkflow** — `global` (bāzes) un `task_type` (PharmaLV ar QA) scope
- **Kanonisko statusu kartēšana** — `awaiting_qa` → `paused`, `rework` → `in_progress`
- **Allowed transitions** — workflow nosaka, kādas pogas redzamas Detail Drawer
- **TaskEvent** — audit trail katram uzdevumam neatkarīgi no klienta
- **Task.client** — klienta kontakta piesaiste uzdevumam

---

*Saistītās idejas: 01, 02, 03, 04, 10, 11, 25*
