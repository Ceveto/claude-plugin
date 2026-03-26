# Scenārijs 3: Darba nodošana ar klienta parakstu uz vietas

## Situācijas apraksts

SIA "TehnoFix" apkalpo rūpnīcas ražošanas līniju aprīkojumu. Pēc katras apkopes vai remonta klienta ražošanas vadītājam jāapliecina, ka darbs ir izpildīts un iekārta darbojas. Iepriekš tehniķi drukāja papīra aktus, kas bieži pazuda vai tika aizpildīti nepilnīgi. Reizēm klients apstrīdēja darba apjomu, jo nebija skaidru pierādījumu.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Tehniķis | Artūrs Vilks | Veic remontu, aizpilda formu, iegūst parakstu |
| Klienta pārstāvis | Liene Ozoliņa, ražošanas vadītāja | Pārbauda darbu, paraksta pieņemšanu |
| Dispečers | Kristīne Jansone | Pārbauda, ka uzdevums ir korekti aizvērts |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 3 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, remonta formu, uzdevuma tipu (ar paraksta prasību), 2 completion gates (forma + paraksts), un uzdevumu "Konveijera motora nomaiņa". Pēc tam vari pāriet uz [4. solis — Darba uzsākšana](#4-solis--darba-uzsākšana-artūrs).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Priekšnosacījumi (vienreiz)

Šis scenārijs izmanto tos pašus konfigurācijas objektus kā S1 — kategoriju, formu un uzdevuma tipu. Ja esi jau izpildījis S1, šo soli var izlaist.

> **UI:** Ja vēl nav izpildīts:
>
> 1. Izveido kategoriju **CMMS Maintenance** (skat. S1, 1. solis)
> 2. Izveido formu **Iekārtu remonta forma** / FORM-REPAIR-001 (skat. S1, 2. solis)
> 3. Izveido tipu **Aukstuma iekārtu avārija** / EMERGENCY-COLD-001 (skat. S1, 3. solis)

---

### 2. solis — Completion gates izveide (vienreiz)

Izveido divus vārtus, kas bloķēs uzdevuma pabeigšanu.

> **UI:** Sidebar → **Tasks** → **Completion Gates** → spied **"+"** pogu
>
> **Gate 1 — Forma obligāta:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Forma obligāta |
> | Gate Type | form_completed |
> | Task Type | Aukstuma iekārtu avārija *(dropdown)* |
> | Enforcement | hard_block |
>
> Spied **"Create"**.
>
> **Gate 2 — Paraksts obligāts:**
>
> Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Paraksts obligāts |
> | Gate Type | signature_required |
> | Task Type | Aukstuma iekārtu avārija |
> | Enforcement | hard_block |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido 2 `TaskCompletionGate` objektus. Abi ir `hard_block` — sistēma neļaus mainīt statusu uz `completed`, kamēr forma nav aizpildīta UN paraksts nav iegūts.

---

### 3. solis — Uzdevuma izveide (Dispečers)

Kristīne izveido remontu uzdevumu.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Konveijera motora nomaiņa |
> | Task Type | Aukstuma iekārtu avārija *(dropdown)* |
> | Priority | Critical *(automātiski no task type)* |
> | Location | *(izvēlies rūpnīcas lokāciju)* |
> | Assigned to | Artūrs Vilks |
>
> Spied **"Create task"**.

**Ko sistēma dara:** Izveido `Task` ar statusu `todo`. Automātiski piemēro abus completion gates no uzdevuma tipa.

---

### 4. solis — Darba uzsākšana (Artūrs)

Artūrs ierodas rūpnīcā un sāk darbu.

> **UI:** Tasks sarakstā klikšķini uz **"Konveijera motora nomaiņa"** → Detail Drawer
>
> Overview tab → spied **"Start"** → statuss mainās uz `in_progress`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`todo` → `in_progress`).
- `TaskStatusInterval` sāk skaitīt aktīvo darba laiku.

---

### 5. solis — Formas aizpildīšana (Artūrs)

Artūrs aizpilda remonta formu darba laikā.

> **UI:** Detail Drawer → **Form** tab
>
> **Diagnostika:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Temperatūra pirms | — |
> | Bojājuma veids | Motora pārslodze E-412 |
> | Foto pirms | nodilušais_gultnis.jpg |
>
> **Remonts:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Veiktie darbi | Motora nomaiņa, gultņa nomaiņa |
> | Izlietotās detaļas | Motors ABB-M3AA-132, sērija MO-2026-0289 |
> | Temperatūra pēc | — |
> | Foto pēc | uzstādītais_motors.jpg |
>
> Spied **"Submit Form"**.

**Ko sistēma dara:**
- `TaskFormSubmission` saglabā visus datus.
- `TaskEvent` (type=`form_submitted`).
- Completion gate "Forma obligāta" kļūst izpildīts ✅.

---

### 6. solis — Mēģinājums pabeigt — gate bloķē (Artūrs)

Artūrs mēģina uzdevumu aizvērt.

> **UI:** Detail Drawer → **Overview** tab → spied **"Complete"**.
>
> Sistēma parāda kļūdu!
>
> Completion Gates sekcija:
> - ✅ "Forma obligāta" — izpildīts
> - ❌ "Paraksts obligāts" — **nav izpildīts**

**Ko sistēma dara:** Pārbauda visus `hard_block` gates. Paraksts nav iegūts — atgriež HTTP 400.

---

### 7. solis — Darba demonstrācija klientam (Artūrs + Liene)

Artūrs uzaicina Lieni apskatīt rezultātu.

> **UI:** Artūrs parāda Lienei Detail Drawer savā ierīcē:
>
> - **Form tab** — kādus darbus veica, kādu detaļu uzstādīja, foto pirms/pēc
> - **Overview tab** — uzdevuma statuss un detaļas
>
> Liene ir apmierināta ar darbu.

---

### 8. solis — Klienta paraksts (Artūrs + Liene)

Liene parakstās uz Artūra planšetes ekrāna.

> **UI:** Detail Drawer → **Overview** tab → **Signatures** sekcija
>
> Šobrīd parakstu pievieno caur API:
>
> ```
> POST /api/tasks/tasks/{task_id}/signatures?account_id={acct_id}
> {
>   "signature_data": "data:image/png;base64,iVBORw0KGgo=",
>   "metadata": {
>     "signer_role": "Ražošanas vadītāja",
>     "signer_name": "Liene Ozoliņa",
>     "gps_lat": 56.9496,
>     "gps_lon": 24.1052
>   }
> }
> ```
>
> Pēc API izsaukuma **Signatures** sekcijā parādīsies paraksts ar datumu un parakstītāja info.

**Ko sistēma dara:**
- `TaskSignature` ar paraksta datiem, GPS koordinātēm, laika zīmogu un parakstītāja lomu.
- `TaskEvent` (type=`signature_added`).
- Completion gate "Paraksts obligāts" tagad ir izpildīts ✅.

---

### 9. solis — Uzdevuma pabeigšana (Artūrs)

Tagad abi gates ir izpildīti.

> **UI:** Detail Drawer → **Overview** tab → spied **"Complete"**.
>
> Šoreiz izdodas! Statuss mainās uz `completed`.
>
> Completion Gates:
> - ✅ "Forma obligāta"
> - ✅ "Paraksts obligāts"
> - Badge: **"All gates satisfied"** (zaļš)

**Ko sistēma dara:**
- `TaskStatusTransition` (`in_progress` → `completed`).
- `TaskStatusInterval` aizver aktīvo intervālu.
- `task.completed_at` iestatīts.

---

### 10. solis — Dispečera pārbaude (Kristīne)

Kristīne birojā pārbauda darba rezultātu.

> **UI:** Detail Drawer → pārbaudi katru tab:
>
> **Overview tab:**
> - Status: Completed
> - Visi gates ✅
> - Signature ar Lienes vārdu, datumu, GPS
>
> **Form tab:**
> - Aizpildītie dati: motora tips, sērijas numurs, foto
>
> **Activity tab:**
> - Task created → Status: todo → in_progress → Form submitted → Signature added → Status: completed
>
> Ja klients mēnesi vēlāk jautā "vai tiešām nomainījāt motoru?", Kristīne 10 sekundēs atrod pierādījumu ar foto, parakstu un detaļas sērijas numuru.

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Darba akts | Papīra forma, bieži nepilnīga | Digitāla forma ar visiem obligātajiem laukiem |
| Klienta paraksts | Uz papīra, var pazust | Digitāls ar GPS un laika zīmogu |
| Pierādījumi pēc mēneša | "Es neatceros, ko jūs darījāt" | Foto + forma + paraksts uzdevumā |
| Kvalitātes kontrole | Atkarīga no cilvēka disciplīnas | Sistēma neļauj aizvērt bez pierādījumiem |
| Strīdu risināšana | Vārds pret vārdu | Objektīvi dati ar laika zīmogiem |

---

## Izmantotās sistēmas funkcijas

- **TaskForm** — dinamiskā forma ar diagnostikas un remonta sekcijām
- **TaskCompletionGate** — `hard_block` uz `form_completed` + `signature_required`
- **TaskSignature** — digitālais paraksts ar GPS, laika zīmogu, lomu
- **TaskFormSubmission** — formas datu iesniegums
- **TaskStatusInterval** — automātiska darba laika uzskaite
- **TaskEvent** — pilna aktivitāšu hronoloģija

---

*Saistītās idejas: 08, 09, 11, 25, 33, 34*
