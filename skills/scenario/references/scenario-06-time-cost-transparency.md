# Scenārijs 6: Darba laika un izmaksu caurspīdīgums

## Situācijas apraksts

SIA "ServisPro" klients — lielveikalu ķēde "FreshMart" — ikmēneša rēķinā apstrīd darba stundu skaitu. "Jūsu tehniķis bija tikai stundu, bet rēķinā ir 3 stundas!" Līdz šim nebija objektīvu pierādījumu — vārds pret vārdu. ServisPro vadība nolēma ieviest caurspīdīgu laika uzskaiti.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Tehniķis | Valdis Ābols | Veic darbu, statusi fiksē laiku automātiski |
| Dispečers | Inese Kalva | Pārbauda ierakstus, gatavo atskaites |
| Klienta menedžeris | Jānis Stradiņš | Izmanto datus rēķina pamatošanai |
| Klients | Aiga Priede, FreshMart operāciju vadītāja | Saņem detalizētu atskaiti |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 6 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, uzdevuma tipu, uzdevumu "Saldētavas remonts — FreshMart Āgenskalns" ar 5 statusa pārejām (todo→in_progress→paused→in_progress→completed), 3 statusa intervāliem un 1 manuālu laika ierakstu. Uzdevums jau ir `completed` statusā — var uzreiz apskatīt laika datus.

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Uzdevuma izveide (Dispečers)

Inese izveido remontu uzdevumu.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Saldētavas remonts — FreshMart Āgenskalns |
> | Task Type | *(izvēlies atbilstošu tipu)* |
> | Priority | Medium |
> | Location | FreshMart Āgenskalns |
> | Assigned to | Valdis Ābols |
>
> Spied **"Create task"**.

**Ko sistēma dara:** Izveido `Task` ar statusu `todo`.

---

### 2. solis — Darba uzsākšana (Valdis, 09:15)

Valdis ierodas FreshMart filiālē un sāk diagnostiku.

> **UI:** Tasks sarakstā klikšķini uz uzdevuma → Detail Drawer
>
> Overview tab → spied **"Start"** → statuss mainās uz `in_progress`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`todo` → `in_progress`) ar laika zīmogu 09:15.
- `TaskStatusInterval` sākas: status=`in_progress`, `is_active_time=True`.

---

### 3. solis — Pauze — brauciens pēc detaļas (Valdis, 10:00)

Valdim jābrauc uz noliktavu pēc rezerves detaļas.

> **UI:** Detail Drawer → **Overview** tab
>
> Spied **"Pause"** pogu (vai statusa dropdown → "Paused").
>
> Statuss mainās uz `paused`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`in_progress` → `paused`) ar laika zīmogu 10:00.
- Aizver iepriekšējo `TaskStatusInterval`: in_progress, 09:15–10:00, **aktīvais laiks: 45 min**.
- Jauns `TaskStatusInterval` sākas: status=`paused`, `is_active_time=False`.

---

### 4. solis — Turpinājums (Valdis, 10:40)

Valdis atgriežas ar detaļu un turpina darbu.

> **UI:** Detail Drawer → **Overview** tab
>
> Spied **"Resume"** (vai statusa dropdown → "In Progress").
>
> Statuss mainās atpakaļ uz `in_progress`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`paused` → `in_progress`) ar laika zīmogu 10:40.
- Aizver pauzes `TaskStatusInterval`: paused, 10:00–10:40, **pauzes laiks: 40 min**.
- Jauns `TaskStatusInterval`: in_progress, `is_active_time=True`.

---

### 5. solis — Darba pabeigšana (Valdis, 12:15)

Remonts ir pabeigts.

> **UI:** Detail Drawer → **Overview** tab → spied **"Complete"**
>
> Statuss mainās uz `completed`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`in_progress` → `completed`) ar laika zīmogu 12:15.
- Aizver pēdējo `TaskStatusInterval`: in_progress, 10:40–12:15, **aktīvais laiks: 1h 35min**.
- Iestata `task.completed_at`.

---

### 6. solis — Laika metriku apskatīšana

Tagad var apskatīt automātiski aprēķinātos laika datus.

> **UI:** Detail Drawer → **Time** tab
>
> Redzēsi:
>
> **Automātiski no statusiem:**
>
> | Intervāls | Status | Ilgums | Aktīvs? |
> |-----------|--------|--------|---------|
> | 09:15–10:00 | in_progress | 45 min | ✅ Jā |
> | 10:00–10:40 | paused | 40 min | ❌ Nē |
> | 10:40–12:15 | in_progress | 1h 35min | ✅ Jā |
>
> **Kopsavilkums:**
> - **Cycle time:** 3h 00min (no pirmā start līdz complete)
> - **Active time:** 2h 20min (tikai in_progress intervāli)
> - **Pause time:** 40min

---

### 7. solis — Manuāla laika ieraksta pievienošana (Valdis)

Valdis pievieno brauciena laiku, kas nav iekļauts statusa intervālos.

> **UI:** Detail Drawer → **Time** tab → spied **"Add Time Entry"**
>
> | Lauks | Vērtība |
> |-------|---------|
> | Start time | 10:00 |
> | End time | 10:40 |
> | Description | Brauciens uz centrālo noliktavu |
>
> Spied **"Save"**.

**Ko sistēma dara:**
- Izveido `TaskTimeEntry` ar manuāli norādīto laiku.
- **Time** tab tagad rāda gan automātiskos intervālus, gan manuālos ierakstus.
- Kopā: 2h 20min aktīvs + 40min brauciens = **3h 00min**.

---

### 8. solis — Dispečera pārbaude (Inese)

Inese pārbauda datus pirms atskaites.

> **UI:** Detail Drawer → **Time** tab — pārskata visus ierakstus
>
> **Activity** tab — pārbauda hronoloģiju:
> - 09:15 → in_progress (sāk darbu)
> - 10:00 → paused (brauc pēc detaļas)
> - 10:40 → in_progress (turpina)
> - 12:15 → completed (pabeigts)
>
> Viss sakrīt — apstiprina.

---

### 9. solis — Klienta diskusija (Jānis + Aiga)

Klients apstrīd stundu skaitu: "Tehniķis bija tikai stundu!"

> **UI:** Jānis atver uzdevuma Detail Drawer un parāda Aigai:
>
> **Time** tab:
> - 09:15 ieradās (sāka darbu)
> - 10:00 pauze (brauc pēc detaļas — var parādīt arī manuālo ierakstu ar aprakstu)
> - 10:40 turpināja
> - 12:15 pabeigts
>
> **Overview** tab:
> - Klienta paraksts plkst. 12:10 (ja piemērojams)
>
> Aiga: "Labi, tagad saprotu. Paldies par detalizēto informāciju."

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Laika uzskaite | Tehniķis pats raksta "3 stundas" | Automātiski no statusu intervāliem |
| Strīdi par stundām | Bieži, nav pierādījumu | Precīzi dati ar laika zīmogiem |
| Pauzes laiks | Nav redzams | Atsevišķi fiksēts (paused intervāls) |
| Brauciena laiks | Ieskaitīts kopējā laikā bez skaidrojuma | Atsevišķs manuāls ieraksts ar piezīmi |
| Mēneša atskaite | Excel ar aptuveniem skaitļiem | Detalizēti dati no sistēmas |
| Klienta uzticība | Zema (nav caurspīdīguma) | Augsta (viss pierādāms) |

---

## Izmantotās sistēmas funkcijas

- **TaskStatusInterval** — automātiski laika intervāli no statusu maiņām
- **TaskStatusTransition** — precīzi laika zīmogi katrai pārejai
- **TaskTimeEntry** — manuālie papildinājumi (brauciens, papildu darbi)
- **Cycle time / Active time** — divējāda laika metrika
- **TaskEvent** — pilna hronoloģija pierādījumiem
- **Time tab** — apkopo automātiskos un manuālos datus vienā skatā

---

*Saistītās idejas: 05, 06, 07, 16, 18, 28, 29, 38*
