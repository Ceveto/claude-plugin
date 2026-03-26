# Scenārijs 11: Vitrīnas gaismu krīze — SexyStyle intīmpreču veikals

## Par uzņēmumu

**SexyStyle** (SIA, reģ. nr. 50103216751) ir Latvijas vadošais intīmpreču mazumtirgotājs, kas darbojas kopš 2006. gada. Uzņēmumam ir fiziskais veikals Rīgā (Lāčplēša ielā 20a) un interneta veikali Latvijā, Lietuvā, Igaunijā un Eiropā. Sortimentā: fantāziju apģērbi, aksesuāri, kosmētika, apakšveļa, zeķes un citi intīmie produkti. Veikalā strādā ~17 darbinieki.

---

## Situācijas apraksts

Ceturtdienas vakarā plkst. 18:30 — tieši pirms "karstā piektdienas vakara" akcijas — veikala vadītāja **Kristīne Liepiņa** konstatē katastrofu: galvenajā tirdzniecības zālē nosprāgušas 4 no 6 vitrīnu LED apgaismojuma paneļiem. Veikals izskatās kā alā. Premium apakšveļas kolekcija "Midnight Velvet" (vērtība €4,200) ir pilnīgā tumsā, un rītdienas akcija paredz 30% atlaidi tieši šai kolekcijai.

Kristīne zvana servisa uzņēmumam, kas apkalpo veikalu.

> *"Mums rīt lielā akcija, bet veikals izskatās kā Helovīna murgs! Vajag gaismas TAGAD!"*

---

## Iesaistītās lomas

| Loma | Persona | Darbības |
|------|---------|----------|
| Klienta pārstāvis | Kristīne Liepiņa, veikala vadītāja | Ziņo par problēmu, dokumentē "tumšās zonas", paraksta darbu |
| Dispečers | Edgars Vanags | Izveido urgent uzdevumu, koordinē tehniķi un detaļas |
| Elektriķis/Tehniķis | Raivis Saulītis | Diagnosticē, nomaina paneļus, nodrošina "perfektu apgaismojumu" |
| Veikala darbiniece | Sabīne Ozoliņa | Asistē ar vitrīnu piekļuvi, pārstāj preču izvietojumu |

---

## Plūsma pa soļiem

### 1. solis — Uzdevuma izveide (Dispečers)

Edgars saņem Kristīnes zvanu un atver Ceveto:

- **Uzdevuma tips:** "Tirdzniecības apgaismojuma avārija"
- **Klients:** SIA "SexyStyle", veikals Lāčplēša 20a
- **Iekārta (asset):** LED vitrīnu paneļi VP-600 (4 gab., sērijas nr. SS-LED-001 līdz SS-LED-006)
- **Apraksts:** "4 no 6 vitrīnu LED paneļiem nav gaismas. Rīt plānota liela akcija — premium kolekcija tumsā. Klients ĻOTI satraukts."

**Ko sistēma dara automātiski:**
- Ielādē formu "Apgaismojuma diagnostika un remonts" ar laukiem: bojāto paneļu ID, foto pirms/pēc, elektroinstalācijas stāvoklis, nomainīto detaļu saraksts.
- Iestata prioritāti **CRITICAL** — jo uzdevuma tips "apgaismojuma avārija" tirdzniecības telpās automātiski saņem augstāko prioritāti.
- Pievieno completion gates: **form_completed** (diagnostikas forma) + **signature_required** (klienta pieņemšanas paraksts).

### 2. solis — Piešķiršana ar īpašu norādi (Dispečers)

Edgars piešķir uzdevumu Raivim Saulītim, kas specializējas LED sistēmās, un pievieno komentāru:

> *"⚠️ Klients ļoti nervozs — rīt liela akcija. Paņem līdzi 4x VP-600 paneļus no noliktavas. Jābūt gatavam līdz plkst. 21:00!"*

- Raivis saņem paziņojumu ar sarkanu CRITICAL indikatoru.
- Komentārs redzams uzdevuma plūsmā (TaskComment + TaskEvent).

### 3. solis — Darba uzsākšana (Tehniķis)

Raivis ierodas veikalā plkst. 19:15 un pārslēdz statusu uz **IN_PROGRESS**.

**Ko sistēma dara automātiski:**
- Sāk skaitīt aktīvo darba laiku (TaskStatusInterval).
- Fiksē ierašanās laiku — 45 min no pieteikuma (reaction time metric).

### 4. solis — Diagnostika un formas aizpildīšana (Tehniķis)

Raivis apskata vitrīnas un konstatē problēmu — mitrums no griestu kondensāta ir sabojājis LED draiverus. Viņš aizpilda diagnostikas formu mobilajā ierīcē:

| Lauks | Vērtība |
|-------|---------|
| Bojātie paneļi | SS-LED-001, SS-LED-003, SS-LED-004, SS-LED-005 |
| Bojājuma cēlonis | Mitruma iekļūšana LED draiveros no griestu kondensāta |
| Foto pirms remonta | 📷 6 attēli (tumšās vitrīnas ar "Midnight Velvet" kolekciju tumsā) |
| Veiktie darbi | 4x LED draiveru nomaiņa, mitruma izolācijas uzlabošana |
| Nomainītās detaļas | LED draiveris VPD-600 (4 gab.), silikona hermētiķis (1 tūba) |
| Foto pēc remonta | 📷 6 attēli (vitrīnas pilnā spožumā, kolekcija izskatās lieliska) |
| Papildus ieteikums | "Ieteicams veikt griestu kondensāta novēršanas darbus, lai problēma neatkārtojas" |

### 5. solis — Pauze un atsākšana (apakšuzdevums!)

Remonta laikā Raivis konstatē, ka vienam panelim vajag citu draiveri, kas nav līdzi.

- Raivis pārslēdz statusu uz **PAUSED** ar piezīmi: *"Gaidu SS-LED-005 draiveri — kolēģis ved no noliktavas."*
- Sistēma aptur aktīvā laika skaitītāju, bet turpina skaitīt kopējo laiku.
- Pēc 25 min draiveris tiek piegādāts — Raivis atsāk (**IN_PROGRESS**).

### 6. solis — Completion gates pārbaude

Raivis mēģina pabeigt uzdevumu. Sistēma pārbauda:

- ✅ Diagnostikas forma aizpildīta pilnībā
- ✅ Foto pirms UN pēc remonta pievienoti (6+6)
- ❌ Klienta paraksts vēl nav

**Sistēma bloķē:** *"Trūkst klienta paraksts. Gate: signature_required (hard_block)."*

### 7. solis — Klienta paraksts un sajūsma

Raivis izsauc Kristīni aplūkot rezultātu. Visas vitrīnas spīd pilnā spožumā.

> Kristīne: *"Tā ir maģija! 'Midnight Velvet' kolekcija vēl nekad nav izskatījusies tik labi! Rīt pārdošanas rekords garantēts!"*

Kristīne parakstās uz Raivja planšetes — TaskSignature ar GPS (56.9496° N, 24.1164° E — Lāčplēša 20a) un laika zīmogu (20:48).

### 8. solis — Uzdevuma pabeigšana

Visi gates izpildīti. Raivis pārslēdz uz **COMPLETED**.

**Ko sistēma dara automātiski:**
- Aptur laika skaitītāju.
- Kopējais laiks: 1h 33min (no tā aktīvais darbs: 1h 08min, pauze: 25 min).
- Izveido pilnu audit trail: izveidots → piešķirts → uzsākts → diagnosticēts → pauzēts → atsākts → forma aizpildīta → parakstīts → pabeigts.

### 9. solis — Pārskatīšana un proaktīvā apkope (Dispečers)

Edgars nākamajā rītā Ceveto paneļā redz:

- ✅ Uzdevums pabeigts 2h 18min pirms deadline (21:00)
- 📋 Pilna hronoloģija ar 6+6 foto — var parādīt klientam pirms/pēc salīdzinājumu
- ⚠️ Tehniķa ieteikums: griestu kondensāta problēma
- 💡 Edgars izveido **jaunu profilaktisko uzdevumu**: "SexyStyle — griestu kondensāta novēršana" ar zemāku prioritāti un plānotu izpildi nākamnedēļ.

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez sistēmas) | Pēc (ar Ceveto) |
|---------|---------------------|-----------------|
| Pieteikums | Panika pa telefonu, WhatsApp foto | Strukturēts CRITICAL uzdevums ar detaļu sarakstu |
| Reakcijas laiks | "Kad varēsiet?" — nezināms | Precīzi 45 min no pieteikuma līdz ierašanās |
| Pauzes izsekošana | Klients nervozē: "Kāpēc stāvat?!" | Pauzes iemesls dokumentēts, aktīvais laiks skaitīts atsevišķi |
| Pierādījumi | Foto pazūd čatā starp 100 citām bildēm | 12 foto piesaistīti uzdevumam ar pirms/pēc struktūru |
| Paraksts | "Mēs bijām apmierināti, bet tagad neatceramies" | Digitāls paraksts ar GPS un laika zīmogu |
| Profilakse | Nākamreiz atkal tā pati problēma | Tehniķis iesaka risinājumu → automātiski jauns uzdevums |
| Rēķina pamatojums | "Kāpēc tik dārgi?!" | Detalizēts: 1h 08min darbs + 4 detaļas + foto pierādījumi |

---

## Amizantais elements 🎭

Šis scenārijs ir lielisks demo materiāls, jo:

1. **Visiem saprotama situācija** — veikals tumsā pirms lielās akcijas = universāla panika.
2. **Emocionāls klients** — Kristīne ir tieši tas cilvēks, kuram vajag pierādījumus un caurskatāmību.
3. **Pauzes skaidrojums** — klasiskā situācija, kad klients domā, ka tehniķis "neko nedara", bet patiesībā gaida detaļu.
4. **Pirms/Pēc foto efekts** — premium apakšveļas kolekcija tumsā vs pilnā LED spožumā ir vizuāli iespaidīgs kontrasts.
5. **Proaktīvā apkope** — sistēma ne tikai risina krīzi, bet arī novērš nākamo, pateicoties tehniķa ieteikumam.

> *Fun fact: Kristīne nākamajā dienā ziņo, ka "Midnight Velvet" kolekcija izpārdota 4 stundās. Vai tas bija LED gaismu nopelns? Droši vien.*

---

## Izmantotās sistēmas funkcijas

- **TaskType** ar CRITICAL noklusēto prioritāti tirdzniecības telpām
- **Dinamiskā forma** — apgaismojuma diagnostika ar obligātiem foto laukiem
- **Completion gates** — form_completed (hard_block) + signature_required (hard_block)
- **TaskComment** — dispečera instrukcijas tehniķim
- **TaskSignature** ar GPS koordinātēm un laika zīmogu
- **TaskStatusInterval** — aktīvais darbs vs pauze atsevišķi
- **TaskStatusTransition** — pilns audit trail
- **TaskEvent** — visa aktivitāšu hronoloģija
- **Proaktīvā uzdevuma izveide** — no tehniķa ieteikuma uz jaunu plānotu darbu

---

*Saistītās idejas: 01, 04, 06, 08, 09, 10, 14, 15, 30*
