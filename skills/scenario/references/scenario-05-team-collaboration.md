# Scenārijs 5: Iekšējā sadarbība brigādē — komentāri un @mentions

## Situācijas apraksts

Tehniķis Normunds remontē liftu biroju ēkā. Darba gaitā atklājas, ka papildus mehāniskajam bojājumam ir arī elektriskā problēma un vajadzīga specifiska detaļa. Normundam ātri jāiesaista elektriķis un noliktavas darbinieks — bet visi ir dažādās vietās.

Iepriekš šāda koordinācija notika ar WhatsApp zvaniem un ziņām, kas pazuda starp citiem čatiem.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Galvenais tehniķis | Normunds Zariņš | Veic remontu, koordinē komandu |
| Elektriķis | Pēteris Saulītis | Pārbauda elektrodaļu, iekomentē risinājumu |
| Noliktavas darbinieks | Ilze Mežāre | Apstiprina detaļas pieejamību |
| Dispečers | Laura Vītoliņa | Seko progresam, reaģē uz eskalāciju |

---

## Ātrais starts ar seed komandu

```bash
cd backend && source .venv/bin/activate
python manage.py seed_scenario 5 --account-id <tavs-account-uuid>
```

Tas izveido: kategoriju, uzdevuma tipu, uzdevumu "Lifta mehānisma remonts — BC Alfa", 2 lietotājus (tehniķi), 2 komentārus (ar @mention), un 1 participant. Pēc tam vari pāriet uz [5. solis — Noliktavas atbilde](#5-solis--noliktavas-atbilde-ilze).

---

## Plūsma pa soļiem (no nulles)

### 1. solis — Uzdevuma izveide (Dispečers)

Laura izveido lifta remonta uzdevumu un piešķir Normundu.

> **UI:** Sidebar → **Tasks** → **Tasks** → spied **"+"** pogu
>
> | Lauks | Vērtība |
> |-------|---------|
> | Title | Lifta mehānisma remonts — BC Alfa |
> | Task Type | HVAC mēneša profilakse *(vai cits atbilstošs)* |
> | Priority | Medium |
> | Location | *(izvēlies biroju centra lokāciju)* |
> | Assigned to | Normunds Zariņš |
>
> Spied **"Create task"**.

**Ko sistēma dara:** Izveido `Task` ar statusu `todo`, piešķirts Normundam.

---

### 2. solis — Darba uzsākšana (Normunds)

Normunds ierodas objektā un sāk darbu.

> **UI:** Tasks sarakstā klikšķini uz uzdevuma → Detail Drawer
>
> Overview tab → spied **"Start"** → statuss mainās uz `in_progress`.

**Ko sistēma dara:** `TaskStatusTransition` (`todo` → `in_progress`), `TaskStatusInterval` sākas.

---

### 3. solis — Problēmas atklāšana un komentārs ar @mention (Normunds)

Normunds konstatē papildu bojājumu un pieprasa palīdzību caur komentāriem.

> **UI:** Detail Drawer → **Comments** tab
>
> Teksta laukā raksta:
> ```
> Atklāts papildu bojājums — frekvenču pārveidotājam kļūda E-07.
> @PēterisS vajag tavu palīdzību ar elektrisko daļu.
> @IlzeM — vai noliktavā ir frekvenču pārveidotājs Danfoss VLT 132F (11kW)?
> Mums vajag vai nu nomaiņu, vai remontkomplektu.
> ```
>
> Spied **"Add Comment"**.

**Ko sistēma dara:**
- Izveido `TaskComment` ar laika zīmogu un autora info.
- @mention sistēma nosūta push paziņojumus Pēterim un Ilzei.
- Abi automātiski tiek pievienoti kā `TaskParticipant` (ja vēl nav).
- `TaskEvent` (type=`comment_added`).

---

### 4. solis — Participants apskatīšana (jebkurš)

Pēc @mention sistēma automātiski pievienoja dalībniekus.

> **UI:** Detail Drawer → **Overview** tab → **Participants** sekcija
>
> Redzēsi:
>
> | Participant | Role |
> |-------------|------|
> | Normunds Zariņš | assigned (owner) |
> | Pēteris Saulītis | contributor |
> | Ilze Mežāre | contributor |
>
> Ja kāds nav automātiski pievienots, var manuāli pievienot ar **"Add Participant"**.

---

### 5. solis — Noliktavas atbilde (Ilze)

Ilze saņem paziņojumu un atbild komentāros.

> **UI:** Ilze atver paziņojumu → tiek atvērts uzdevuma Detail Drawer → **Comments** tab
>
> Raksta:
> ```
> Danfoss VLT 132F nav noliktavā. Ir pieejams VLT 132F-HC (ar heatsink) —
> derēs kā aizstājējs. Sērijas nr. DF-2025-7743.
> Varu sagatavot izsniegšanai pēc 30 min.
> ```
>
> Spied **"Add Comment"**.

**Ko sistēma dara:** Jauns `TaskComment` ar Ilzes info. Visi dalībnieki redz atbildi savā activity feed.

---

### 6. solis — Elektriķa atbilde un plānošana (Pēteris)

Pēteris apskata situāciju un iekomentē risinājumu.

> **UI:** Detail Drawer → **Comments** tab
>
> Raksta:
> ```
> E-07 parasti nozīmē jaudas moduļa bojājumu. VLT 132F-HC derēs.
> @NormundsZ es varu ierasties ap 14:00 — nomaiņa aizņems ~1h.
> Pagaidām atstāj veco pārveidotāju pieslēgtu, bet neieslēdz liftu.
> ```
>
> Spied **"Add Comment"**.

---

### 7. solis — Koordinēta izpilde

Darbs turpinās ar komentāru koordināciju.

> **UI:** Detail Drawer → **Comments** tab — viss redzams hronoloģiski:
>
> | Laiks | Komentārs |
> |-------|-----------|
> | 11:30 | Normunds: papildu bojājums, @mention |
> | 11:48 | Ilze: detaļas alternatīva |
> | 12:05 | Pēteris: plāno ierašanos 14:00 |
> | 13:30 | Ilze: "Detaļa sagatavota, gaida paņemšanu" |
> | 15:05 | Pēteris: "Pārveidotājs nomainīts. Lifts testēts 10 ciklus — darbojas normāli." |

---

### 8. solis — Uzdevuma pabeigšana (Normunds)

Normunds pabeidz mehānisko daļu un aizvērt uzdevumu.

> **UI:** Detail Drawer → **Overview** tab → spied **"Complete"**
>
> Statuss mainās uz `completed`.

**Ko sistēma dara:**
- `TaskStatusTransition` (`in_progress` → `completed`).
- `TaskStatusInterval` aizver — fiksē kopējo darba ilgumu.
- `task.completed_at` iestatīts.

---

### 9. solis — Dispečera pārskats (Laura)

Laura jebkurā brīdī var atvērt uzdevumu un redzēt pilnu ainu.

> **UI:** Detail Drawer → **Activity** tab
>
> Pilna hronoloģija:
>
> | Laiks | Notikums |
> |-------|----------|
> | 09:20 | Task created, assigned to Normunds |
> | 10:45 | Status: todo → in_progress |
> | 11:30 | Comment: Normunds @mention Pēteris, Ilze |
> | 11:35 | Pēteris added as participant |
> | 11:35 | Ilze added as participant |
> | 11:48 | Comment: Ilze — detaļas alternatīva |
> | 12:05 | Comment: Pēteris — plāno ierašanos |
> | 13:30 | Comment: Ilze — detaļa sagatavota |
> | 15:05 | Comment: Pēteris — darbs pabeigts |
> | 15:15 | Status: in_progress → completed |

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Komunikācija | WhatsApp grupa, zvani | Komentāri tieši pie uzdevuma |
| Detaļu pieejamība | Zvans noliktavai, gaida atzvanu | @mention ar tūlītēju paziņojumu |
| Papildu speciālistu iesaiste | Dispečerim jākoordinē manuāli | Tehniķis pats @mention kolēģi |
| Vēsture | Ziņas pazūd WhatsApp | Pilna hronoloģija pie uzdevuma |
| Laika uzskaite | Tikai galvenais tehniķis | Katram participant atsevišķi intervāli |
| Dispečera pārskats | "Kas tur notiek?" — jāzvana | Reāllaika aktivitāšu plūsma |

---

## Izmantotās sistēmas funkcijas

- **TaskComment** — komentāri ar @mentions un threading
- **TaskParticipant** — automātiska pievienošana no @mention
- **Push paziņojumi** — tūlītēja informēšana par @mention
- **TaskStatusInterval** — laika uzskaite pa dalībniekiem
- **TaskEvent** — pilns audit trail ar visiem notikumiem
- **Activity feed** — hronoloģisks skats uz visu, kas notika

---

*Saistītās idejas: 12, 27, 30, 37*
