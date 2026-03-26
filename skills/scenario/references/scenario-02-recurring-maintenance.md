# Scenārijs 2: Plānotā profilakse — ikmēneša HVAC apkope

## Situācijas apraksts

SIA "KlimatServis" apkalpo 47 biroju ēku ventilācijas un kondicionēšanas sistēmas. Katru mēnesi katrā ēkā jāveic HVAC profilaktiskā apkope: filtru maiņa, aukstumnesēja līmeņa pārbaude, drenāžas sistēmas tīrīšana. Iepriekš dispečere Sandra katru mēnesi manuāli izveidoja 47 uzdevumus — tas aizņēma pusdienu.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Servisa vadītājs | Andris Liepiņš | Konfigurē recurring template |
| Dispečere | Sandra Kalēja | Pārbauda automātiski ģenerētos uzdevumus |
| Tehniķu brigāde | Kārlis (owner) + Edgars, Mārtiņš (participants) | Veic apkopi, fiksē darbu un laiku |

---

## Ātrais starts ar seed komandu

Ja negribi veidot visu manuāli — palaid seed komandu:

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 2 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, HVAC uzdevuma tipu, recurring template (ikmēneša), tehniķa lietotāju un 1 ģenerētu uzdevumu. Pēc tam vari pāriet uz [4. solis — Dispečeres pārskats](#4-solis--dispečeres-pārskats-sandra).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Kategorijas izveide (vienreiz)

Andris izveido uzdevumu kategoriju (ja vēl neeksistē no scenārija 1).

> **UI:** Sidebar → **Tasks** → **Task Categories** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | CMMS Maintenance |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `TaskCategory` objektu. Ja kategorija jau eksistē (no S1), šo soli var izlaist.

---

### 2. solis — Uzdevuma tipa izveide (vienreiz)

Andris izveido HVAC profilakses uzdevuma tipu.

> **UI:** Sidebar → **Tasks** → **Task Types** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | HVAC mēneša profilakse |
> | Code | PM-HVAC-001 |
> | Category | CMMS Maintenance *(izvēlies no dropdown)* |
> | Default Priority | Medium |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `TaskType` ar `default_priority=medium`. Šis tips tiks izmantots recurring template un arī manuāli veidotiem HVAC uzdevumiem.

---

### 3. solis — Recurring template izveide (vienreiz)

Andris izveido atkārtojošā uzdevuma šablonu, kas automātiski ģenerēs uzdevumus katru mēnesi.

> **UI:** Sidebar → **Tasks** → **Recurring Templates** → spied **"+"** pogu
>
> Atveras Create drawer. Ievadi:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | HVAC mēneša profilakse |
> | Task Type | HVAC mēneša profilakse *(izvēlies no dropdown)* |
> | Frequency | Monthly |
> | Interval | 1 |
> | Location | *(izvēlies lokāciju no dropdown)* |
> | Assigned to | Kārlis *(brigādes vadītājs)* |
>
> Template payload (JSON):
> ```json
> {
>   "title": "HVAC profilakse — {location}",
>   "priority": "medium",
>   "description": "Filtru maiņa, aukstumnesēja pārbaude, drenāžas tīrīšana"
> }
> ```
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `TaskRecurringTemplate` ar atkārtošanās grafiku. `next_run_at` tiek iestatīts uz nākamā mēneša 1. datumu. Sistēma periodiski pārbauda šablonus un ģenerē uzdevumus, kad pienāk laiks.

---

### 4. solis — Dispečeres pārskats (Sandra)

Mēneša 1. datumā sistēmas Dramatiq workers automātiski izpilda recurring ģenerāciju. Sandra no rīta atver Ceveto un redz jaunus uzdevumus.

> **UI:** Sidebar → **Tasks** → **Tasks**
>
> Filtrē sarakstu:
> - **Status** filtrs → "Todo"
> - **Task Type** filtrs → "HVAC mēneša profilakse"
>
> Redzēsi jaunu(-us) uzdevumu(-us) ar nosaukumu "HVAC profilakse — {lokācija}", prioritāti Medium, statusu Todo.
>
> Sandra klikšķina uz uzdevuma nosaukuma → atveras Detail Drawer.
>
> **Overview** tab → **Assignment** sekcija:
> - Assigned to: Kārlis (automātiski no template)
>
> Ja nepieciešams, Sandra var:
> - Mainīt assigned_to
> - Pievienot participants (skat. zemāk)

**Ko sistēma dara:**
- Automatizācija izveido `Task` objektus no `TaskRecurringTemplate`.
- Katrs uzdevums mantojumā saņem formu, prioritāti un termiņu no template.
- Izveido `TaskRecurringInstance` ierakstu, kas savieno uzdevumu ar šablonu.
- Izveido `TaskEvent` (type=`created`): "Automātiski ģenerēts no šablona".

---

### 5. solis — Participants pievienošana (Sandra)

Sandra pievieno brigādes dalībniekus uzdevumam.

> **UI:** Detail Drawer → **Overview** tab → **Participants** sekcija
>
> Spied **"Add Participant"** pogu:
>
> | Participant | Role |
> |-------------|------|
> | Edgars | contributor |
> | Mārtiņš | contributor |
>
> Spied **"Add"** katram.

**Ko sistēma dara:** Izveido `TaskParticipant` ierakstus. Edgars un Mārtiņš tagad redz šo uzdevumu savā sarakstā un var sekot progresam.

---

### 6. solis — Darba izpilde (Brigāde)

Kārlis sadala darbus brigādei. Katrs tehniķis atver savu uzdevumu.

> **UI:** Tasks sarakstā klikšķini uz uzdevuma → Detail Drawer
>
> **Kārlis sāk darbu:**
> Overview tab → spied **"Start"** → statuss mainās uz `in_progress`.
>
> **Form tab** → aizpilda profilakses check-list:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Filtru stāvoklis | Nomainīti |
> | Aukstumnesēja līmenis | Norma (95%) |
> | Drenāžas sistēma | Iztīrīta |
> | Vibrāciju tests | Norma |
> | Kopējais novērtējums | Labi |
>
> Spied **"Submit Form"**.
>
> **Comments tab** → pievieno novērojumu:
> ```
> B korpusa 3. stāvā filtru turētājs ir ieplīsusi — ieteicams nomainīt nākamajā reizē.
> ```
> Spied **"Add Comment"**.
>
> **Overview tab** → spied **"Complete"** → statuss mainās uz `completed`.

**Ko sistēma dara:**
- `TaskStatusTransition` fiksē katru statusa maiņu.
- `TaskStatusInterval` ar `is_active_time=True` sāk skaitīt aktīvo darba laiku no "Start" līdz "Complete".
- `TaskFormSubmission` saglabā check-list datus.
- `TaskComment` saglabā novērojumu ar laika zīmogu.
- `TaskEvent` ieraksti katrai darbībai.

---

### 7. solis — Laika uzskaite (automātiska + manuāla)

Pēc darba pabeigšanas Kārlis var pārskatīt laika metriku.

> **UI:** Detail Drawer → **Time** tab
>
> Redzēsi:
> - **Cycle time:** Kopējais laiks no izveides līdz pabeigšanai
> - **Active time:** Aktīvais darba laiks (tikai in_progress intervāli)
> - **Tracked time:** Manuāli pievienotās stundas
>
> Ja nepieciešams, pievieno manuālu laika ierakstu:
>
> Spied **"Add Time Entry"**:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Start time | 08:30 |
> | End time | 09:00 |
> | Description | Brauciens uz objektu |
>
> Spied **"Save"**.

**Ko sistēma dara:**
- `TaskStatusInterval` automātiski fiksē laiku no statusa maiņām.
- `TaskTimeEntry` saglabā manuālos papildinājumus (brauciens, sagatavošanās).
- **Time** tab apkopo abus datu avotus kopējā skatā.

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Uzdevumu izveide | Sandra veido 47 uzdevumus manuāli (~3h) | Sistēma ģenerē automātiski 5 sekundēs |
| Termiņu kontrole | Excel tabula | Automātiski termiņi no template |
| Darba laika uzskaite | Papīra lapas vai zvani | Automātiski no statusa intervāliem |
| Profilakses izlaišana | Var aizmirst lokāciju | Sistēma ģenerē uzdevumu katrai lokācijai |
| SLA pierādījumi | Nav | Pilna hronoloģija ar laikiem un datiem |
| Brigādes koordinācija | Telefona zvani | Participants + komentāri uzdevumā |

---

## Izmantotās sistēmas funkcijas

- **TaskRecurringTemplate** — šablons ar atkārtošanās grafiku
- **TaskRecurringInstance** — saite starp šablonu un ģenerēto uzdevumu
- **TaskType** — HVAC profilakses tips ar noklusēto prioritāti
- **TaskCategory** — uzdevumu tipu grupējums
- **TaskParticipant** — brigādes dalībnieku pārvaldība
- **TaskStatusInterval** — automātiska laika uzskaite
- **TaskTimeEntry** — manuālie laika ieraksti (brauciens u.c.)
- **TaskComment** — piezīmes un novērojumi pie uzdevuma
- **TaskEvent** — pilna aktivitāšu hronoloģija

---

*Saistītās idejas: 18, 20, 21, 26, 27, 28, 29, 38*
