# Scenārijs 12: Alvis Iznīcinātājs — kā Ceveto izglābj biroju no haosa

## Par situāciju

Servisa uzņēmumā **SIA "TechFix Pro"** strādā trīs leģendāras personības. Biroja koridoros klīst teiciens: *"Kur Alvis — tur darbs. Kur Alvis — tur DAUDZ darba."*

---

## Iesaistītās lomas

| Loma | Persona | Raksturojums |
|------|---------|-------------|
| Servisa tehniķis | **Alvis** — "Iznīcinātājs" | Cilvēks ar unikālu talantu — viņš spēj sabojāt lietas, kuras neviens cits pat nespētu atrast. Vienmēr ar labiem nodomiem, bet rezultāts... iepriekš neparedzams. Reiz mēģinot salabot kafijas automātu, noīsināja visu 3. stāvu. |
| Dispečers / Vadītājs | **Toms** — "Vulkāns" | Karstgalvīgs vadītājs, kurš jau 3 gadus mēģina saprast, kāpēc remonta budžets vienmēr ir 200% pārsniegts. Viņa iecienītākā frāze: *"ALVI!!! KO TU ATKAL ESI IZDARĪJIS?!"* Asinsspiediens paaugstinās katru reizi, kad tālrunī iedegas Alvja vārds. |
| Vecākais tehniķis | **Artūrs** — "Glābējs" | Mierīgs, metodisks profesionālis, kurš patiesībā salabo visu, ko Alvis sabojā, un vēl nedaudz vairāk. Dzīves filozofija: *"Nav tādas problēmas, ko nevar atrisināt. Ir tikai Alvja problēmas, kam vajag vairāk laika."* |

---

## Plūsma pa soļiem

### Epizode 1 — "Kafijas automāta incidents" (08:45)

**Situācija:** Klientam SIA "Baltic Office" saplīsis kafijas automāts. Vienkāršs izsaukums — nomainīt ūdens filtru.

**Toms** (Ceveto dispečera panelis):
Izveido uzdevumu un pievieno komentāru:
> *"Filtra maiņa. VIENKĀRŠA filtra maiņa. Lūdzu, Alvi, NEKUSTINI neko citu."*

- **Uzdevuma tips:** "Kafijas iekārtu apkope"
- **Prioritāte:** LOW (cik grūti var būt nomainīt filtru?)
- **Piešķirts:** Alvis
- **Completion gates:** form_completed (apkopes forma)

**Alvis** ierodas objektā, pārslēdz statusu uz **IN_PROGRESS**.

Alvis nomaina filtru. Darbojas! Bet tad viņš pamana, ka kafijas iztekas caurulīte "izskatās nedaudz vaļīga"...

Alvis nolemj "ātri pievelkam".

Alvis pārslēdz statusu uz **PAUSED** ar piezīmi: *"Pievelku caurulīti, 2 min"*

Rezultāts: caurulīte lūzt, karsts ūdens šļāciens applaucē servera telpu blakus, izsit 2 datoru monitorus un aktivizē ugunsdzēsības sistēmu visā stāvā.

**Alvis** Ceveto sistēmā pievieno komentāru:
> *"Neliela komplikācija. Vajag Artūru."*

### Epizode 2 — "Toma reakcija" (09:12)

**Toms** atver Ceveto un redz Alvja komentāru. Viņa rīcība:

1. **Maina uzdevuma prioritāti** no LOW → **CRITICAL** 🔴
2. Pievieno komentāru (kuru sistēma saglabā audit trail — mūžīgi):
   > *"ALVI. TAS BIJA FILTRS. F-I-L-T-R-S. KĀ TU SPĒJI APPLAUCĒT SERVERU TELPU AR KAFIJAS AUTOMĀTA FILTRU?! ARTŪR, LŪDZU BRAUC TŪLĪT."*
3. Izveido **3 apakšuzdevumus** (jo Alvja "filtra maiņa" tagad ir pilna katastrofa):
   - 🔧 "Ūdens noplūdes novēršana — kafijas automāts"
   - 💻 "Servera telpas žāvēšana un aprīkojuma pārbaude"
   - 🔥 "Ugunsdzēsības sistēmas atiestatīšana"
4. Pievieno **Artūru** kā dalībnieku visos 3 apakšuzdevumos
5. Pievieno **completion gate**: signature_required (hard_block) — *"Šoreiz es PERSONĪGI parakstīšu, ka viss ir kārtībā"*

### Epizode 3 — "Artūrs glābj situāciju" (09:45)

**Artūrs** ierodas un pārslēdz pirmo apakšuzdevumu uz **IN_PROGRESS**.

Artūra komentārs (mierīgi, profesionāli):
> *"Situācija novērtēta. Alvja 'pievelkšana' izraisīja montāžas bloka šķelšanu. Ūdens apturēts. Sāku žāvēšanu."*

Artūrs strādā cauri visiem 3 apakšuzdevumiem, aizpildot diagnostikas formas:

**Apakšuzdevums 1 — Ūdens noplūde:**
| Lauks | Vērtība |
|-------|---------|
| Bojājuma cēlonis | "Neautorizēta caurulītes manipulācija" (Artūrs ir diplomātisks) |
| Foto pirms remonta | 📷 Alvja katastrofas ainava — serveru telpa šļakatu pilna |
| Veiktie darbi | Montāžas bloka nomaiņa, jauna iztekas caurulīte |
| Foto pēc remonta | 📷 Kafijas automāts strādā perfekti |

**Apakšuzdevums 2 — Serveru telpa:**
| Lauks | Vērtība |
|-------|---------|
| Bojājuma apjoms | 2 monitori bojāti, serveris OK (bija ūdensizturīgā skapī) |
| Veiktie darbi | Žāvēšana, monitoru nomaiņa, kabeļu pārbaude |
| Papildus ieteikums | "Ieteicams kafijas automātu PĀRVIETOT tālāk no serveru telpas" |

**Apakšuzdevums 3 — Ugunsdzēsības sistēma:**
| Lauks | Vērtība |
|-------|---------|
| Veiktie darbi | Sistēmas atiestatīšana, sensoru pārbaude |
| Statuss | Pilnībā funkcionāla |

Katru apakšuzdevumu Artūrs pabeidz ar laika ierakstu (TaskTimeEntry):
- Ūdens noplūde: 45 min
- Serveru telpa: 1h 20min
- Ugunsdzēsība: 25 min
- **Kopā: 2h 30min** (vietā Alvja oriģinālajām "2 min")

### Epizode 4 — "Parakstu ceremonija" (12:30)

Artūrs mēģina pabeigt galveno uzdevumu, bet sistēma bloķē:

> **"Trūkst paraksts: Toma personīgais paraksts (hard_block)"**

Toms ierodas uz vietas. Apskata Artūra darbu. Viss ir perfekti.

Toms parakstās ar TaskSignature (GPS: Baltic Office birojs).

Bet pirms tam viņš pievieno komentāru:
> *"Artūr — tu esi leģenda. Alvi — nākamnedēļ tev būs apmācības par to, kā NEPIESKARTIES lietām, kuras tev nav lūgts salabot. Es to ieplānošu Ceveto kā recurring task."*

### Epizode 5 — "Recurring apmācība" (Toms ir nopietns)

Toms atver Ceveto un izveido **recurring template**:

- **Nosaukums:** "Alvja drošības instruktāža — NEAIZTIEC to, kas nav tavs uzdevums"
- **Frekvence:** monthly (reizi mēnesī)
- **Uzdevuma tips:** "Darbinieku apmācība"
- **Completion gates:**
  - form_completed (hard_block): "Alvim jāaizpilda tests par to, ko viņš ir iemācījies"
  - signature_required (hard_block): "Alvja paraksts, ka viņš saprot"
- **Apraksts:** *"Ikmēneša atgādinājums Alvim par robežām starp 'salabot filtru' un 'iznīcināt stāvu'. Artūrs vada apmācību."*

---

## Pirms / Pēc salīdzinājums

| Aspekts | Pirms (bez Ceveto) | Pēc (ar Ceveto) |
|---------|--------------------|-----------------|
| Alvja katastrofas | Nav dokumentētas, Toms tikai kliedz | Pilns audit trail ar foto — pierādījumi mūžīgi 📸 |
| Toma dusmas | Izgaist gaisā, nekas nemainās | Konstruktīvi novirzītas: apakšuzdevumi, gates, recurring apmācība |
| Artūra darbs | "Artūr, brauc kaut kur, kaut kas salūza" | Strukturēti uzdevumi ar formām, laika uzskaiti un prioritātēm |
| Izmaksu pārskatāmība | "Kāpēc rēķins €800 par filtra maiņu?" | Detalizēts: 45min ūdens + 1h20min serveri + 25min ugunsdzēsība + 2 monitori |
| Atkārtošanās novēršana | Alvis nākammēnes sabojā kaut ko citu | Monthly recurring apmācība ar testu un parakstu |
| Klienta komunikācija | "Mums te bija neliels incidents..." | Profesionāla atskaite ar foto, laiku un veiktajiem darbiem |

---

## Morāle

Ceveto nespēj novērst to, ka Alvis sabojā lietas. **Bet Ceveto var:**

1. ✅ **Dokumentēt katastrofu** — ar foto, laika zīmogiem un GPS
2. ✅ **Eskalēt pareizi** — prioritātes maiņa no LOW uz CRITICAL ar vienu klikšķi
3. ✅ **Sadalīt haosu** — viens "filtra maiņas" uzdevums kļūst par 3 strukturētiem apakšuzdevumiem
4. ✅ **Izsekot izmaksas** — klients zina, kāpēc rēķins ir €800 nevis €30
5. ✅ **Novērst nākamo reizi** — recurring apmācības ar obligātu testu
6. ✅ **Saglabāt Toma veselību** — dusmas tiek novirzītas uz komentāriem, nevis uz cilvēkiem

> *Epilogs: Nākamajā mēnesī Alvis veiksmīgi nokārto ikmēneša apmācības testu ar 100% rezultātu. Tajā pašā nedēļā viņš mēģina "ātri salabot" biroja printeri un izraisa papīra iestrēgumu, kas bloķē visus 4 stāva printerus. Toms Ceveto sistēmā izveido jaunu uzdevuma tipu: "Alvja Speciālais Gadījums" ar obligātu completion gate: "Artūram jāapstiprina PIRMS Alvis pieskaras iekārtai."*

---

## Izmantotās sistēmas funkcijas

- **TaskType** ar dažādām prioritātēm (LOW → CRITICAL eskalācija)
- **Parent/Subtask hierarhija** — 1 sākotnējais uzdevums → 3 apakšuzdevumi
- **TaskComment** — komunikācija starp Tomu, Alvi un Artūru (ar emocionālu kontekstu)
- **Completion gates** — form_completed + signature_required (hard_block)
- **TaskSignature** — Toma personīgais paraksts ar GPS
- **TaskStatusInterval** — PAUSED → IN_PROGRESS (Alvja "2 min" kas kļūst par stundām)
- **TaskTimeEntry** — detalizēta laika uzskaite pa apakšuzdevumiem
- **TaskRecurringTemplate** — ikmēneša apmācības ar obligātu testu
- **TaskEvent** — pilna audit trail (ieskaitot Toma leģendāros komentārus)
- **Prioritātes maiņa** — LOW → CRITICAL dinamiska eskalācija

---

*Saistītās idejas: 01, 04, 06, 08, 09, 10, 12, 14, 15, 18, 19, 20, 27, 30*
