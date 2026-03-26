# Scenārijs 7: Drošības priekšnosacījumi pirms darba uzsākšanas

## Situācijas apraksts

SIA "IndustriaServis" apkalpo ķīmiskās rūpnīcas iekārtas. Noteiktos objektos darbu drīkst uzsākt tikai pēc drošības procedūru izpildes: drošības instruktāža, individuālo aizsardzības līdzekļu (IAL) pārbaude, darba atļaujas saņemšana. Iepriekš šīs prasības bija "uz papīra" — tehniķi dažreiz tās ignorēja, jo steidzās.

Pēc neliela incidenta (tehniķis uzsāka darbu bez aizsargbrillēm tīrāmatā zonā), uzņēmums nolēma ieviest sistēmisku kontroli.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Drošības speciālists | Anita Liepa | Konfigurē drošības prasības sistēmā |
| Dispečers | Roberts Vējš | Izveido uzdevumu |
| Tehniķis | Gunārs Celms | Mēģina uzsākt darbu, izpilda prasības |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 7 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, remonta formu, uzdevuma tipu (EMERGENCY-COLD-001), 3 completion gates (form hard_block, signature hard_block, time_logged soft_warning), un uzdevumu "Reaktora bloka R-7 apkope — ķīmiskā zona". Pēc tam vari pāriet uz [5. solis — Tehniķis mēģina uzsākt darbu](#5-solis--tehniķis-mēģina-uzsākt-darbu-gunārs).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Priekšnosacījumi (vienreiz)

Ja nav izpildīts S1, izveido bāzes objektus.

> **UI:**
> 1. Kategorija **CMMS Maintenance** (skat. S1, 1. solis)
> 2. Forma **Iekārtu remonta forma** / FORM-REPAIR-001 (skat. S1, 2. solis)
> 3. Tips **Aukstuma iekārtu avārija** / EMERGENCY-COLD-001 (skat. S1, 3. solis)

---

### 2. solis — Drošības gates izveide (Anita, vienreiz)

Anita konfigurē 3 completion gates, kas bloķēs darba uzsākšanu un pabeigšanu.

> **UI:** Sidebar → **Tasks** → **Completion Gates** → spied **"+"** pogu
>
> **Gate 1 — Drošības instruktāža (hard_block):**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Drošības instruktāža |
> | Gate Type | form_completed |
> | Task Type | Aukstuma iekārtu avārija *(dropdown)* |
> | Enforcement | hard_block |
> | Failure Message | Drošības forma nav aizpildīta |
>
> Spied **"Create"**.
>
> **Gate 2 — Darba atļaujas paraksts (hard_block):**
>
> Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Darba atļaujas paraksts |
> | Gate Type | signature_required |
> | Task Type | Aukstuma iekārtu avārija |
> | Enforcement | hard_block |
> | Failure Message | Darba atļauja nav parakstīta |
>
> Spied **"Create"**.
>
> **Gate 3 — Riska novērtējums (soft_warning):**
>
> Spied **"+"** vēlreiz:
>
> | Lauks | Vērtība |
> |-------|---------|
> | Name | Riska novērtējums |
> | Gate Type | time_logged |
> | Task Type | Aukstuma iekārtu avārija |
> | Enforcement | soft_warning |
> | Failure Message | Ieteicams aizpildīt riska novērtējumu |
>
> Spied **"Create"**.

**Ko sistēma dara:** Izveido 3 `TaskCompletionGate` objektus:
- 2 ar `hard_block` — sistēma fiziski bloķē statusa maiņu, kamēr nav izpildīti
- 1 ar `soft_warning` — sistēma brīdina, bet ļauj turpināt

---

### 3. solis — Uzdevuma izveide (Roberts)

Roberts izveido apkopes uzdevumu ķīmiskajā zonā.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Reaktora bloka R-7 apkope — ķīmiskā zona |
> | Task Type | Aukstuma iekārtu avārija *(vai drošības specifisks tips)* |
> | Priority | Critical |
> | Location | *(izvēlies ķīmiskās rūpnīcas lokāciju)* |
> | Assigned to | Gunārs Celms |
>
> Spied **"Create task"**.

**Ko sistēma dara:** Izveido `Task` ar statusu `todo`. Automātiski piemēro visus 3 gates no uzdevuma tipa.

---

### 4. solis — Gates statusa pārskats (jebkurš)

Pirms darba var apskatīt, kādas prasības jāizpilda.

> **UI:** Tasks sarakstā klikšķini uz uzdevuma → Detail Drawer
>
> **Overview** tab → **Completion Gates** sekcija:
>
> | Gate | Status | Enforcement |
> |------|--------|-------------|
> | Drošības instruktāža | ❌ Nav izpildīts | hard_block |
> | Darba atļaujas paraksts | ❌ Nav izpildīts | hard_block |
> | Riska novērtējums | ⚠️ Nav izpildīts | soft_warning |
>
> Badge: **"Some gates not satisfied"** (sarkans)

---

### 5. solis — Tehniķis mēģina uzsākt darbu (Gunārs)

Gunārs ierodas objektā un mēģina sākt darbu.

> **UI:** Detail Drawer → **Overview** tab → spied **"Start"**
>
> Sistēma bloķē pāreju! Kļūdas ziņojums:
>
> ```
> Nevar uzsākt darbu. Izpildiet prasības:
> ❌ Drošības instruktāža — drošības forma nav aizpildīta
> ❌ Darba atļaujas paraksts — darba atļauja nav parakstīta
> ⚠️ Riska novērtējums — ieteicams aizpildīt (nav obligāts)
> ```

**Ko sistēma dara:** Pārbauda visus gates. `hard_block` gates nav izpildīti — atgriež HTTP 400, statuss paliek `todo`.

---

### 6. solis — Drošības formas aizpildīšana (Gunārs)

Gunārs aizpilda drošības instruktāžas formu.

> **UI:** Detail Drawer → **Form** tab
>
> **Diagnostika** (vai drošības instruktāžas sekcija):
>
> | Lauks | Vērtība |
> |-------|---------|
> | Temperatūra pirms | — |
> | Bojājuma veids | Plānotā apkope |
> | Foto pirms | IAL_komplekts.jpg *(selfijs ar aizsargbrillēm, respiratoru, cimdiem)* |
>
> **Remonts:**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Veiktie darbi | *(aizpildīs vēlāk)* |
> | Izlietotās detaļas | — |
> | Temperatūra pēc | — |
> | Foto pēc | — |
>
> Spied **"Submit Form"**.

**Ko sistēma dara:**
- `TaskFormSubmission` ar drošības datiem un IAL foto.
- Gate "Drošības instruktāža" kļūst izpildīts ✅.

---

### 7. solis — Darba atļaujas paraksts (Gunārs + drošības inženieris)

Gunārs atrod objekta drošības inženieri, kas pārbauda IAL un parakstās.

> **UI:** Parakstu pievieno caur API:
>
> ```
> POST /api/tasks/tasks/{task_id}/signatures?account_id={acct_id}
> {
>   "signature_data": "data:image/png;base64,iVBORw0KGgo=",
>   "metadata": {
>     "signer_role": "Drošības inženieris",
>     "signer_name": "Māris Zālītis",
>     "signature_type": "darba_atļauja",
>     "gps_lat": 56.9496,
>     "gps_lon": 24.1052
>   }
> }
> ```
>
> Pēc API izsaukuma **Signatures** sekcijā parādīsies paraksts.

**Ko sistēma dara:**
- `TaskSignature` ar parakstītāja datiem un lomu "Drošības inženieris".
- Gate "Darba atļaujas paraksts" kļūst izpildīts ✅.

---

### 8. solis — Atkārtots mēģinājums uzsākt darbu (Gunārs)

Tagad abi hard_block gates ir izpildīti.

> **UI:** Detail Drawer → **Overview** tab
>
> Completion Gates sekcija tagad:
> - ✅ "Drošības instruktāža" — izpildīts
> - ✅ "Darba atļaujas paraksts" — izpildīts
> - ⚠️ "Riska novērtējums" — soft_warning (var ignorēt)
>
> Spied **"Start"** → šoreiz izdodas! Statuss mainās uz `in_progress`.

**Ko sistēma dara:**
- Hard_block gates izpildīti — pāreja atļauta.
- Soft_warning tiek parādīts, bet nebloķē.
- `TaskStatusTransition` (`todo` → `in_progress`).
- `TaskStatusInterval` sākas.

---

### 9. solis — Darba izpilde un pabeigšana (Gunārs)

Gunārs veic apkopi un pabeidz darbu.

> **UI:** Detail Drawer:
>
> 1. **Form** tab → papildina formu ar remonta datiem (veiktie darbi, detaļas, foto pēc)
> 2. **Overview** tab → spied **"Complete"** → statuss mainās uz `completed`

---

### 10. solis — Drošības audita pārskats (Anita)

Anita var pārbaudīt, vai tehniķis izpildīja visas prasības PIRMS darba uzsākšanas.

> **UI:** Detail Drawer → **Activity** tab
>
> Hronoloģija:
>
> | Laiks | Notikums |
> |-------|----------|
> | 08:30 | Task created |
> | 08:42 | Form submitted (drošības instruktāža + IAL foto) |
> | 08:51 | Signature added (Māris Zālītis, drošības inženieris) |
> | 08:53 | Status: todo → in_progress (darbs uzsākts) |
> | 11:15 | Form updated (remonta dati) |
> | 11:30 | Status: in_progress → completed |
>
> **Svarīgi:** Formas aizpildīšana un paraksts notika PIRMS darba uzsākšanas — sistēma to garantē caur hard_block gates.
>
> **Overview** tab → Completion Gates:
> - ✅✅⚠️ — abi hard_block izpildīti, soft_warning ignorēts
>
> **Signatures** sekcija:
> - Paraksts ar datumu, GPS, parakstītāja lomu

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Drošības instruktāža | "Esmu izlasījis" mutiskā apliecinājuma | Digitāla forma ar datiem |
| IAL kontrole | Nav pierādījumu | Foto ar laika zīmogu un GPS |
| Darba atļauja | Papīra veidlapa | Digitāls paraksts no drošības inženiera |
| Izpildes kontrole | Atkarīga no cilvēka disciplīnas | Sistēma fiziski bloķē darba sākšanu |
| Incidenta izmeklēšana | "Nav pierādījumu, ko tehniķis darīja" | Pilna hronoloģija ar foto, parakstiem, laikiem |
| Atbilstība regulām | Grūti pierādīt | Sistēmisks pierādījumu trail |

---

## Izmantotās sistēmas funkcijas

- **TaskCompletionGate** ar `hard_block` — bloķē darba uzsākšanu
- **TaskCompletionGate** ar `soft_warning` — brīdina, bet neblokē
- **Gate tipi:** `form_completed`, `signature_required`, `time_logged`
- **TaskSignature** — digitāls paraksts ar lomu un kontekstu
- **TaskFormSubmission** — drošības forma ar IAL foto
- **TaskEvent** — audit trail ar precīziem laikiem (pierāda secību)
- **Gate status UI** — vizuāls pārskats par izpildītajām/neizpildītajām prasībām

---

*Saistītās idejas: 09, 10, 11, 17, 36*
