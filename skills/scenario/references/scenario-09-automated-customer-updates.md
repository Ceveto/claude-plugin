# Scenārijs 9: Automātiska klienta informēšana (L2/L3 roadmap)

## Situācijas apraksts

SIA "QuickFix" apkalpo biroju un tirdzniecības centru iekārtas. Viņu lielākais klients — tirdzniecības centrs "Alfa" — regulāri sūdzas: "Mēs nezinām, kad jūsu tehniķis ieradīsies un cik ilgi vēl jāgaida." Dispečere Anda pavada ~2h dienā, zvanot klientiem ar statusu atjauninājumiem. QuickFix vēlas šo procesu automatizēt.

> **Piezīme:** Šis scenārijs demonstrē L2/L3 iespējas, kas ir nākamais attīstības solis pēc MVP/L1.

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Servisa administrators | Vitālijs Rudzītis | Konfigurē notifikāciju noteikumus |
| Dispečere | Anda Sproģe | Vairs nav manuāli jāzvana klientiem |
| Tehniķis | Ēriks Grundulis | Strādā kā parasti — statusu maiņas trigero notifikācijas |
| Klients | Mārtiņš Kauls, Alfa apsaimniekotājs | Saņem automātiskus atjauninājumus |

---

## Plūsma pa soļiem

### 1. solis — Notifikāciju noteikumu konfigurēšana (vienreiz)

Vitālijs konfigurē event-based noteikumus klientam "Alfa":

| Notikums (trigger) | Darbība | Kanāls | Saņēmējs |
|--------------------|---------|--------|-----------|
| Statuss → DISPATCHED | SMS: "Tehniķis ir piešķirts jūsu pieteikumam. ETA: {eta}" | SMS + e-pasts | Klienta kontaktpersona |
| Statuss → ON_WAY | SMS: "Tehniķis {technician_name} ir ceļā. Plānotā ierašanās: {eta}" | SMS | Klienta kontaktpersona |
| Statuss → IN_PROGRESS | SMS: "Darbs uzsākts. Tehniķis {technician_name} ir uz vietas." | SMS | Klienta kontaktpersona |
| Statuss → COMPLETED | E-pasts: Servisa atskaite ar formas datiem un foto | E-pasts | Klienta kontaktpersona + vadītājs |
| ETA mainīts > 30 min | SMS: "Atvainojiet, jaunais ierašanās laiks: {new_eta}" | SMS | Klienta kontaktpersona |

### 2. solis — ETA aprēķināšana

Sistēma aprēķina ETA (Estimated Time of Arrival) ar vairākām metodēm:

- **Distance-based:** GPS attālums no tehniķa atrašanās vietas līdz objektam (vidējais ātrums 45 km/h pilsētā).
- **Schedule-based:** Ja tehniķim pirms tam ir cits uzdevums, ETA = iepriekšējā uzdevuma plānotais beigu laiks + brauciena laiks.
- **Historical:** Vidējais brauciena laiks uz šo objektu no vēsturiskajiem datiem.

### 3. solis — Reālā darba plūsma

**10:00** — Anda izveido uzdevumu un piešķir Ēriku.
- Statuss: DISPATCHED
- **Automātiski:** Mārtiņš saņem SMS: "Tehniķis ir piešķirts. Plānotā ierašanās: ap 11:15."

**11:05** — Ēriks pabeidz iepriekšējo darbu un pārslēdz statusu uz ON_WAY.
- **Automātiski:** Mārtiņš saņem SMS: "Tehniķis Ēriks Grundulis ir ceļā. Plānotā ierašanās: 11:25."

**11:22** — Ēriks ierodas un pārslēdz uz IN_PROGRESS.
- **Automātiski:** Mārtiņš saņem SMS: "Darbs uzsākts. Tehniķis Ēriks Grundulis ir uz vietas."

**12:45** — Ēriks pabeidz darbu, aizpilda formu, iegūst parakstu, pārslēdz uz COMPLETED.
- **Automātiski:** Mārtiņš un viņa vadītājs saņem e-pastu ar:
  - Servisa atskaites kopsavilkumu
  - Veikto darbu aprakstu
  - Foto pirms/pēc
  - Klienta paraksta apstiprinājumu

**Andai nav jāzvana nevienam.** Viss notiek automātiski.

### 4. solis — Dokumenta automātiska ģenerēšana (papildu)

Pie COMPLETED notikuma sistēma var arī automātiski ģenerēt:
- PDF servisa aktu no formas datiem
- CSV eksportu ar detalizētiem datiem
- Pielāgotu e-pasta šablonu ar klienta logo un formatējumu

### 5. solis — Klientu portāls (papildu)

Mārtiņš var arī pieslēgties klientu portālam (publiska lapa bez pilnas autentifikācijas) un redzēt:
- Aktīvos uzdevumus ar reāllaika statusu
- Vēsturiskos uzdevumus ar atskaitēm
- ETA karti ar tehniķa aptuveno atrašanās vietu

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto L2/L3) |
|---------|---------------------|----------------------|
| Klienta informēšana | Dispečere zvana manuāli (~2h/dienā) | Automātiski SMS/e-pasti |
| ETA | "Kādā stundā" (aptuveni) | Precīzs aprēķins no GPS/grafika |
| Servisa atskaite | Manuāli sagatavo un nosūta | Automātiski ģenerēta un nosūtīta |
| Klienta apmierinātība | "Mēs nezinām, kas notiek" | Reāllaika atjauninājumi |
| Dispečera slodze | Augsta (zvanu koordinēšana) | Būtiski samazināta |
| Profesionalitātes iespaids | Haotisks | Sistēmisks, uzticams |

---

## Izmantotās sistēmas funkcijas (L2/L3)

- **Event-based notifikāciju engine** — automātiski trigeri uz statusu maiņām
- **E-pasta un SMS šabloni** ar mainīgajiem (Jinja2)
- **ETA aprēķins** — distance, schedule, historical metodes
- **Dokumentu automātiska ģenerēšana** — PDF/CSV no uzdevuma datiem
- **Klientu portāls** — publisks piekļuves punkts bez pilnas auth
- **Idempotence** — novērš dubultu paziņojumu nosūtīšanu

---

*Saistītās idejas: 12, 13, 22, 23, 39*
