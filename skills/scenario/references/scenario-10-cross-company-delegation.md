# Scenārijs 10: Sadarbība starp uzņēmumiem — uzdevumu deleģēšana partneriem (L3 roadmap)

## Situācijas apraksts

SIA "MegaServis" ir liels servisa uzņēmums, kas apkalpo rūpnieciskās iekārtas visā Latvijā. Dažos reģionos viņiem nav savu tehniķu, tāpēc deleģē darbus sertificētiem partneriem. Līdz šim deleģēšana notika pa e-pastu — ar kavēšanos, informācijas zudumiem un strīdiem par paveikto.

> **Piezīme:** Šis scenārijs demonstrē L3 iespējas — enterprise+ līmeņa funkcionalitāti.

---

## Iesaistītās lomas

| Loma | Persona | Uzņēmums | Darbības |
|------|---------|----------|----------|
| Dispečers | Andris Lācis | MegaServis (galvenais) | Izveido un deleģē uzdevumu |
| Partnera dispečers | Zane Ose | ReģionServis (partneris) | Pieņem un piešķir savam tehniķim |
| Partnera tehniķis | Oļegs Petrovs | ReģionServis | Izpilda darbu |
| Klients | Marta Vītola | SIA "RūpnīcaPlus" | Gala pasūtītājs |

---

## Plūsma pa soļiem

### 1. solis — Uzņēmumu savstarpējā saistīšana (vienreiz)

MegaServis nosūta sadarbības uzaicinājumu ReģionServis:

**Uzaicinājuma parametri:**
- Partnera tips: Apakšuzņēmējs
- Atļautie uzdevumu tipi: "Iekārtu apkope", "Avārijas remonts"
- Redzamības politika: Slēpt finanšu datus, rādīt formu un statusus
- Termiņš: 30 dienas pieņemšanai

Zane no ReģionServis pieņem uzaicinājumu un apstiprina sadarbības nosacījumus. Turpmāk MegaServis var deleģēt uzdevumus ReģionServis.

### 2. solis — Uzdevuma izveide un deleģēšana

Andris izveido uzdevumu:
- **Klients:** SIA "RūpnīcaPlus", Liepājas filiāle
- **Tips:** "Avārijas remonts — Kompresors"
- **Prioritāte:** HIGH
- **Apraksts:** Galvenais kompresors K-300 nedarbojas. Ražošana apstājusies.

Tā kā Liepājā MegaServis nav tehniķu, Andris deleģē uzdevumu ReģionServis:

**Deleģēšanas iestatījumi:**
- Partneris: ReģionServis
- Redzamība: Formas dati ✅, Finanšu dati ❌, Klienta kontakti ✅
- Termiņš pieņemšanai: 2 stundas (HIGH prioritāte)

### 3. solis — Partnera pieņemšana

Zane no ReģionServis saņem paziņojumu par jaunu deleģētu uzdevumu. Viņa redz:
- Uzdevuma aprakstu un formu
- Klienta kontaktus un adresi
- Prioritāti un termiņu
- **Neredz:** MegaServis iekšējās izmaksas un peļņas maržu

Zane pieņem uzdevumu un piešķir Oļegam.

**Statuss:** PENDING → ACCEPTED

### 4. solis — Darba izpilde (Partnera tehniķis)

Oļegs strādā kā parasti — ar savu Ceveto kontu ReģionServis vidē:
- Pārslēdz statusu uz IN_PROGRESS.
- Aizpilda formu ar diagnostikas un remonta datiem.
- Pievieno foto.
- Iegūst klienta parakstu.
- Pārslēdz uz COMPLETED.

**Ko sistēma dara automātiski:**
- Visi statusa maiņas un formas dati sinhronizējas atpakaļ uz MegaServis.
- MegaServis dispečers reāllaikā redz partnera progresu.
- Audit trail fiksē visas darbības ar norādi "ReģionServis / Oļegs Petrovs".

### 5. solis — Kontrole un apstiprināšana

Andris no MegaServis redz:
- ✅ Uzdevums pabeigts ar visiem pierādījumiem
- 📋 Formas dati ar foto un parakstu
- ⏱️ Izpildes laiks: 3h 20min
- 🏢 Izpildītājs: ReģionServis / Oļegs Petrovs

Andris var apstiprināt darbu vai pieprasīt papildinājumus pirms galīgās aizvēršanas.

### 6. solis — Vienota atskaite klientam

Marta no RūpnīcaPlus saņem atskaiti no MegaServis (nevis no ReģionServis) — jo MegaServis ir galvenais līgumspartneris:
- Servisa atskaite ar MegaServis logo
- Veiktie darbi, foto, paraksts
- Nav redzama informācija par apakšuzņēmēju (ja tā konfigurēts)

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto L3) |
|---------|---------------------|---------------------|
| Deleģēšana | E-pasts ar aprakstu | Strukturēts uzdevums ar kontrolētu piekļuvi |
| Partnera progress | "Zvaniet un pajautājiet" | Reāllaika statusa sinhronizācija |
| Datu drošība | Nosūta visu pa e-pastu | Redzamības politika (slēpt finanšu datus) |
| Pierādījumi | Partneris nosūta foto pa WhatsApp | Formas dati un foto sinhronizējas automātiski |
| Klienta atskaite | Jāsaliek manuāli | Vienota atskaite no galvenā uzņēmuma |
| Audit trail | Nav | Pilna ķēde: izveide → deleģēšana → izpilde → apstiprināšana |

---

## Izmantotās sistēmas funkcijas (L3)

- **CompanyRelationship** — kontrolēta partneru saistīšana ar uzaicinājumu plūsmu
- **ExternalTaskAssignment** — uzdevumu deleģēšana ar statusu sinhronizāciju
- **Visibility policy** — kontrolēta datu redzamība (slēpt finanšu datus)
- **Cross-company audit trail** — vienota notikumu ķēde
- **Approval gates** — galvenā uzņēmuma apstiprināšana pirms aizvēršanas
- **Skill/certification matching** — var pārbaudīt partnera tehniķa kvalifikāciju

---

*Saistītās idejas: 31, 32, 33, 34, 35*
