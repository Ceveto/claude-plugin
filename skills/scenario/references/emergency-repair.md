# Scenārijs 1: Avārijas remonts — aukstuma vitrīna veikalā

## Situācijas apraksts

Piektdienas rītā plkst. 7:15 veikala vadītāja Inga Bērziņa pamana, ka gaļas vitrīna nav noturējusi temperatūru nakts laikā. Digitālais termometrs rāda +12°C (norma: +2..+4°C). Produkti var sabojāties — nepieciešama tūlītēja reakcija.

Inga zvana servisa dispečerim.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Klienta pārstāvis | Inga Bērziņa, veikala vadītāja | Ziņo par problēmu, paraksta darbu |
| Dispečers | Māris Kalniņš | Izveido uzdevumu, piešķir tehniķi |
| Servisa tehniķis | Jānis Ozols | Veic remontu, aizpilda formu, iegūst parakstu |

---

## Ātrais starts ar seed komandu

Ja negribi veidot visu manuāli — palaid seed komandu:

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 1 --account-id <tavs-account-uuid>
```

Tas izveido visu nepieciešamo: kategoriju, formu, uzdevuma tipu, 2 completion gates, kontaktu un 1 uzdevumu. Pēc tam vari pāriet uzreiz uz [7. solis — Uzdevuma izveide](#7-solis--uzdevuma-izveide-dispečers).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Kategorijas izveide (vienreiz)

Māris atver Ceveto un izveido uzdevumu kategoriju, zem kuras grupēs visus CMMS remonta uzdevumus.

> **UI:** Sidebar → **Tasks** → **Task Categories** → spied **"+"** pogu augšā pa kreisi
>
> Atveras Create drawer. Ievadi:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | CMMS Maintenance |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `TaskCategory` objektu, kas kalpo kā grupējums uzdevumu tipiem. Kategorija ir koplietojama — tā tiks izmantota arī citos scenārijos.

---

### 2. solis — Formas izveide (vienreiz)

Māris izveido remonta formu ar diagnostikas un remonta sekcijām. Šo formu pēc tam piesaistīs uzdevuma tipam — katrs tehniķis to redzēs uzdevumā.

> **UI:** Sidebar → **Tasks** → **Forms** → spied **"+"** pogu
>
> Atveras Create drawer. Ievadi:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Iekārtu remonta forma |
> | Code | FORM-REPAIR-001 |
>
> Sections (JSON):
> ```json
> [
>   {
>     "id": "diagnostics",
>     "name": "Diagnostika",
>     "fields": [
>       {"id": "temp_before", "label": "Temperatūra pirms", "type": "number"},
>       {"id": "fault_type", "label": "Bojājuma veids", "type": "select"},
>       {"id": "photo_before", "label": "Foto pirms", "type": "file"}
>     ]
>   },
>   {
>     "id": "repair",
>     "name": "Remonts",
>     "fields": [
>       {"id": "work_done", "label": "Veiktie darbi", "type": "text"},
>       {"id": "parts_used", "label": "Izlietotās detaļas", "type": "text"},
>       {"id": "temp_after", "label": "Temperatūra pēc", "type": "number"},
>       {"id": "photo_after", "label": "Foto pēc", "type": "file"}
>     ]
>   }
> ]
> ```
>
> Spied **"Create"**. Pēc tam formu sarakstā atver to un spied **"Publish"**, lai tā būtu aktīva.

**Ko sistēma dara:** Izveido `TaskForm` ar statusu `published`. Forma definē, kādus datus tehniķim jāaizpilda darba laikā. Tā tiks automātiski ielādēta katrā uzdevumā, kura tips norāda uz šo formu.

---

### 3. solis — Uzdevuma tipa izveide (vienreiz)

Māris izveido uzdevuma tipu "Aukstuma iekārtu avārija", kas apvieno kategoriju, formu, noklusēto prioritāti un paraksta prasību.

> **UI:** Sidebar → **Tasks** → **Task Types** → spied **"+"** pogu
>
> Atveras Create drawer. Ievadi:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Aukstuma iekārtu avārija |
> | Code | EMERGENCY-COLD-001 |
> | Category | CMMS Maintenance *(izvēlies no dropdown)* |
> | Base Form | Iekārtu remonta forma *(izvēlies no dropdown)* |
> | Default Priority | Critical |
> | Requires Signature | ✅ (ieslēgts) |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `TaskType` ar `default_priority=critical`, `requires_signature=True` un piesaistītu formu. Katru reizi, kad kāds izveidos uzdevumu ar šo tipu, sistēma automātiski iestatīs prioritāti un formu.

---

### 4. solis — Completion gates izveide (vienreiz)

Māris izveido divus "vārtus" (gates), kas neļaus aizvērt uzdevumu, kamēr nav izpildītas prasības.

> **UI:** Sidebar → **Tasks** → **Completion Gates** → spied **"+"** pogu
>
> **Gate 1 — Klienta paraksts:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Klienta paraksts obligāts |
> | Gate Type | signature_required |
> | Task Type | Aukstuma iekārtu avārija *(izvēlies no dropdown)* |
> | Enforcement | hard_block |
> | Failure Message | Trūkst klienta paraksts |
>
> Spied **"Create"**.
>
> **Gate 2 — Forma aizpildīta:**
>
> Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Forma jāaizpilda |
> | Gate Type | form_completed |
> | Task Type | Aukstuma iekārtu avārija |
> | Enforcement | hard_block |
> | Failure Message | Remonta forma nav aizpildīta |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido 2 `TaskCompletionGate` objektus, kas ir piesaistīti uzdevuma tipam. `hard_block` nozīmē, ka sistēma fiziski neļaus mainīt statusu uz `completed`, kamēr abas prasības nav izpildītas. Tie automātiski attiecas uz visiem uzdevumiem ar šo tipu.

---

### 5. solis — Klienta kontakta izveide (ja vēl nav)

Māris pievieno klientu sistēmā.

> **UI:** Sidebar → **Contacts** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | SIA "Pārtikas Nams" |
> | Type | Client |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `Contact` ar `type=client`. Kontaktu var piesaistīt uzdevumiem, lai zinātu, kuram klientam darbs tiek veikts.

---

### 6. solis — Lokācijas izveide (ja vēl nav)

Māris pievieno darba vietu.

> **UI:** Sidebar → **Locations** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Pārtikas Nams — Rīga Centrs |
> | Address | Brīvības iela 120, Rīga |
> | Client | SIA "Pārtikas Nams" *(izvēlies no dropdown)* |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido `Location` objektu. Uzdevumam obligāti jābūt piesaistītai lokācijai vai asset — tas nodrošina, ka katrs darbs ir ģeogrāfiski identificējams.

---

### 7. solis — Uzdevuma izveide (Dispečers)

Māris atver Ceveto un izveido jaunu avārijas uzdevumu.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu augšā pa kreisi
>
> Atveras Create Task drawer. Ievadi:
>
> **Basic Information:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Gaļas vitrīna KH-320 neuztur temperatūru |
> | Task Type | Aukstuma iekārtu avārija *(izvēlies no dropdown)* |
> | Priority | Critical *(automātiski aizpildās no task type, var mainīt)* |
>
> **Assignment:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Client | SIA "Pārtikas Nams" |
> | Location | Pārtikas Nams — Rīga Centrs |
> | Assigned to | Jānis Ozols *(izvēlies tehniķi no dropdown)* |
>
> **Details:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Description | Temperatūra +12°C, norma +2..+4°C. Produktu zuduma risks. |
>
> Spied **"Create task"**.

**Ko sistēma dara:**
- Izveido `Task` ar statusu `todo` un prioritāti `critical` (sarkans badge).
- Automātiski piesaista formu no uzdevuma tipa (`base_form`).
- Automātiski piemēro abus completion gates, jo tie ir piesaistīti šim task type.
- Izveido `TaskEvent` (type=`created`) un `TaskStatusTransition` (→ `todo`).
- Uzdevums parādās sarakstā ar sarkanu prioritātes badge.

---

### 8. solis — Darba uzsākšana (Tehniķis)

Jānis ierodas objektā un sāk darbu.

> **UI:** Tasks sarakstā klikšķini uz uzdevuma **nosaukuma** → atveras Detail Drawer labajā pusē.
>
> **Overview** tab augšā redzēsi pogu **"Start"** (vai "In Progress").
>
> Spied **"Start"** — statuss mainās no `todo` uz `in_progress`.

**Ko sistēma dara:**
- Izveido `TaskStatusTransition` (`todo` → `in_progress`).
- Izveido `TaskStatusInterval` ar `is_active_time=True` — sāk skaitīt darba laiku.
- Izveido `TaskEvent` (type=`status_changed`).
- Badge mainās no "Todo" uz "In Progress".

---

### 9. solis — Formas aizpildīšana (Tehniķis)

Jānis aizpilda diagnostikas un remonta formu.

> **UI:** Detail Drawer → pārslēdzies uz **Form** tab.
>
> Redzēsi formu **"Iekārtu remonta forma"** v1 ar 2 sekcijām:
>
> **Diagnostika:**
>
> | Lauks | Ievadi |
> |-------|--------|
> | Temperatūra pirms | 12.3 |
> | Bojājuma veids | Kompresora kļūme |
> | Foto pirms | photo_01.jpg |
>
> **Remonts:**
>
> | Lauks | Ievadi |
> |-------|--------|
> | Veiktie darbi | Kompresora nomaiņa, gāzes uzpilde |
> | Izlietotās detaļas | Kompresors XR-440 (1 gab.) |
> | Temperatūra pēc | 3.1 |
> | Foto pēc | photo_02.jpg |
>
> Spied **"Submit Form"** apakšā.

**Ko sistēma dara:**
- Izveido `TaskFormSubmission` ar visiem datiem.
- Izveido `TaskEvent` (type=`form_submitted`).
- Completion gate "Forma jāaizpilda" tagad ir izpildīts ✅.

---

### 10. solis — Mēģinājums pabeigt — gate bloķē (Tehniķis)

Jānis mēģina uzdevumu pabeigt.

> **UI:** Detail Drawer → atgriezies uz **Overview** tab.
>
> Spied **"Complete"** pogu.
>
> Sistēma parāda kļūdu — uzdevumu nevar pabeigt!
>
> Paskaties **Completion Gates** sekcijā (Overview tab apakšā):
> - ✅ "Forma jāaizpilda" — zaļš (izpildīts)
> - ❌ "Klienta paraksts obligāts" — sarkans (nav izpildīts)
> - Badge: **"Some gates not satisfied"** (sarkans)

**Ko sistēma dara:** Pārbauda visus `hard_block` gates priekš šī task type. Ja kaut viens nav izpildīts, atgriež HTTP 400 un neļauj mainīt statusu uz `completed`.

---

### 11. solis — Klienta paraksts (Tehniķis + Klients)

Jānis parāda paveikto Ingai. Inga parakstās uz planšetes ekrāna.

> **UI:** Detail Drawer → **Overview** tab → **Signatures** sekcija.
>
> Šobrīd parakstu pievieno caur API (signature UI komponents ir read-only):
>
> ```
> POST /api/tasks/tasks/{task_id}/signatures?account_id={acct_id}
> {
>   "signature_data": "data:image/png;base64,iVBORw0KGgo=",
>   "metadata": {
>     "signer_role": "Klienta pārstāvis",
>     "gps_lat": 56.9496,
>     "gps_lon": 24.1052
>   }
> }
> ```
>
> Pēc API izsaukuma **Signatures** sekcijā parādīsies paraksts ar datumu un parakstītāja info.

**Ko sistēma dara:**
- Izveido `TaskSignature` ar paraksta datiem, GPS koordinātēm un laika zīmogu.
- Izveido `TaskEvent` (type=`signature_added`).
- Completion gate "Klienta paraksts obligāts" tagad ir izpildīts ✅.

---

### 12. solis — Uzdevuma pabeigšana (Tehniķis)

Tagad visi gates ir izpildīti. Jānis pabeidz darbu.

> **UI:** Detail Drawer → **Overview** tab.
>
> Spied **"Complete"** pogu.
>
> Šoreiz izdodas! Statuss mainās uz `completed`.
>
> Paskaties:
> - Status badge: **"Completed"**
> - Completion Gates: ✅✅ **"All gates satisfied"** (zaļš)

**Ko sistēma dara:**
- Izveido `TaskStatusTransition` (`in_progress` → `completed`).
- Aizver `TaskStatusInterval` — aprēķina darba ilgumu sekundēs.
- Iestata `task.completed_at` laika zīmogu.
- Izveido `TaskEvent` (type=`status_changed`).

---

### 13. solis — Pārskatīšana (Dispečers)

Māris Ceveto paneļā pārbauda darba rezultātu.

> **UI:** Detail Drawer → pārbaudi katru tab:
>
> **Overview tab:**
> - Status: Completed, Priority: Critical
> - Visi gate ✅ zaļi
> - Signature ar datumu un GPS
>
> **Activity tab:**
> - Pilns audit trail hronoloģiskā secībā:
>   - Task created
>   - Status changed: todo → in_progress
>   - Form submitted
>   - Signature added
>   - Status changed: in_progress → completed
>
> **Time tab:**
> - Cycle time (minūtes no sākuma līdz beigām)
> - Tracked time (aktīvais darba laiks)
> - Var pievienot manuālas laika ievades ar start/end laikiem
>
> **Form tab:**
> - Aizpildītā forma ar visiem datiem (read-only)
> - Last submission datums un iesniedzēja vārds
>
> **Comments tab:**
> - Var pievienot komentāru par darba kvalitāti vai jautājumiem

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Pieteikums | Telefona zvans, WhatsApp ziņa | Strukturēts uzdevums ar prioritāti |
| Pierādījumi | Foto tālrunī, kas pazūd | Fotofiksācija piesaistīta uzdevumam |
| Klienta paraksts | Papīra akts, ko var pazaudēt | Digitāls paraksts ar laika zīmogu |
| Darba laiks | Manuāla atzīmēšana | Automātiska uzskaite no statusiem |
| Strīdi par paveikto | Bieži ("mēs nezinām, ko jūs darījāt") | Pilna audit trail ar foto un parakstu |
| Reakcijas ātrums | Nav izsekojams | Precīzi mērīts un analizējams |

---

## Izmantotās sistēmas funkcijas

- **TaskCategory** — uzdevumu tipu grupējums
- **TaskForm** — dinamiskā forma ar sekcijām un laukiem
- **TaskType** ar noklusēto prioritāti, formu un paraksta prasību
- **TaskCompletionGate** — hard_block vārti (forma + paraksts)
- **Task** — galvenais uzdevuma objekts ar statusu, prioritāti, piešķīrumu
- **TaskFormSubmission** — formas datu iesniegums
- **TaskSignature** — digitāls paraksts ar GPS un laika zīmogu
- **TaskStatusInterval** — automātiska darba laika uzskaite
- **TaskStatusTransition** — statusa pāreju audit trail
- **TaskEvent** — pilna aktivitāšu hronoloģija

---

*Saistītās idejas: 01, 04, 08, 09, 10, 14, 15, 30*
