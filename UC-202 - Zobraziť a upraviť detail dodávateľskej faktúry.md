Nižšie je celý UC-202 s opravami. Kontrolu konzistencie som spravil na konci - vypisujem, čo som overoval.

---

# UC-202 - Zobraziť a upraviť detail dodávateľskej faktúry

## Obsah

- Obsah
- Zapracované zmeny
- Otázky
- Biznis zadanie
- Aktéri
- Spúšťač
- Vstupné podmienky
- Automatizované kontroly pri zaevidovaní
- Hlavný tok
- Alternatívny tok
- Diagram tokov
- Výstupné podmienky
- Opis Obrazoviek + Validácie
- API
- Vysvetlivky pojmov

## Zapracované zmeny

**Uzavreté otvorené otázky**

1. **Vyhľadávanie dodávateľa** prebieha podľa názvu alebo IČO. Po zadaní časti hodnoty systém ponúkne všetkých zodpovedajúcich dodávateľov spolu s údajmi, s ktorými sú založení v ODS, a spracovateľ si vyberie vhodného.
2. **Dohľadanie dodávateľa je dostupné vo všetkých statusoch**, v ktorých je faktúra editovateľná.
3. **Požiadavka na založenie dodávateľa aj čísla účtu sa zadáva do iProc**, nie do UPI.
4. **Uloženie obrazu faktúry** je povinné v statusoch 2 a 3. V ostatných statusoch nie je nutnosťou, ale ak sa to technicky dá, biznis to uvíta.
5. **Formát obrazu faktúry** - preferuje sa generovanie PDF.
6. **Prílohy faktúry** sa preklápajú do iProc spolu s obrazom faktúry a musí byť možné ich stiahnuť na interné úložisko.

**Opravy z revízie zapracovania**

7. **AT 1d rozšírený na dve situácie.** Doteraz riešil len opravu nesprávne priradeného dodávateľa. Doplnená situácia, keď kontrola A dodávateľa nenašla vôbec a spracovateľ ho dohľadá manuálne - to je podľa stavového modelu jediná cesta, ako faktúru odblokovať zo statusu 2 bez čakania na automatické overenie ODS.
8. **AT 1a doplnené o odkaz na manuálne dohľadanie dodávateľa.** Doteraz uvádzalo len automatické overovanie a zadanie requestu.
9. **Editovateľnosť poľa Obchodný partner spresnená** vrátane statusov 2 a 3; doplnená otázka na potvrdenie.
10. **Odstránená nesprávna podmienka dostupnosti akcií na detaile.** Doteraz bolo uvedené, že akcie sú dostupné, „ak faktúra nevyžaduje úpravy" - to systém nevie posúdiť. Dostupnosť riadi stavový model, posúdenie potreby úprav je na spracovateľovi.
11. **Doplnené odloženie do AT 6a** - biznis spomenul aj dozistenie, čo zodpovedá statusu 5.

**Zúžená otázka**

12. Otázka, či systém pri malom nesúlade sám ponúkne kandidátov spôsobom, akým to robí ABBYY, zostáva otvorená. Vyhľadávanie na základe zadanej hodnoty je potvrdené, proaktívne ponúknutie bez zásahu spracovateľa nie.

## Otázky

| # | Otvorená otázka | Adresát |
|---|---|---|
| 1 | Úplný zoznam polí detailu a editovateľných polí - mapovanie na XML doplní Iveta (xls), údaje z hranatých zátvoriek doplní Andrea, revízia Michal. | Biznis |
| 2 | Rozpor editácie súm: na meetingu potvrdené, že sa needitujú sumy; podklad 10a pri zrážkovej dani uvádza editáciu základu dane a navýšenie finálnej sumy. | Biznis, Michal Konečný |
| 3 | Hodnoty číselníka spôsobu zaplatenia - doplniť z ABBYY. | Michal Konečný |
| 4 | Kľúč kontroly duplicity: bunka C1 xls uvádza DIČ + číslo faktúry + dátum vystavenia, dokument 10b uvádza dodávateľ + číslo faktúry + dátum vystavenia. Chybová hláška z iProc (E037) hovorí o „supplier or party". | Biznis |
| 5 | Variant kontroly čísla účtu: zastavenie v eInvoice vs návrh poslať do iProc s prázdnym poľom účtu. Má dopad na stav Cancelled. | Iveta, Michal |
| 6 | Kontrola duplicít: rozdelenie zodpovednosti eInvoice vs iProc. | Biznis |
| 7 | Kritérium automatického púšťania bezzávadových faktúr do iProc. | Eva, Michal |
| 8 | Zobrazenie a editácia dlhého popisu (skracovanie, limit znakov) - Andrea pošle príklad. | Biznis |
| 9 | Zvýraznenie problémových polí (vzor ABBYY červený flag) - rozsah potvrdí biznis. | Biznis |
| 10 | Spôsob potvrdenia prijatia faktúry z iProc (technicky). | Michal Konečný, vývoj |
| 11 | Spôsob a frekvencia načítania exportu dodávateľov z ODS. Rozhranie musí umožniť aj vyhľadávanie dodávateľa podľa časti názvu alebo IČO s vrátením údajov, s ktorými je dodávateľ založený v ODS. | Architekt, vývoj |
| 12 | Akým spôsobom sa technicky vygeneruje PDF s obrazom faktúry, ktoré si spracovateľ stiahne? Biznis preferuje PDF; náhradný formát neurčil. | Architekt |
| 13 | Ako sa technicky realizuje preklopenie príloh do iProc a ich stiahnutie na interné úložisko? Súvisí s FileNet DMS. | Architekt |
| 14 | Čo je obsahom príloh, keď dodávateľské faktúry prídu do eInvoice výhradne v XML? Ide o dokumenty vložené priamo v XML, alebo o samostatné súbory od poštára? | Biznis, architekt |
| 15 | Preklápajú sa prílohy do iProc automaticky spolu s odoslaním faktúry, alebo ich prikladá spracovateľ manuálne? | Biznis |
| 16 | Má systém pri malom nesúlade sám ponúknuť zoznam kandidátov na dodávateľa bez zásahu spracovateľa, ako to robí ABBYY? Vyhľadávanie na základe zadanej hodnoty je potvrdené. | Biznis |
| 17 | Ktorý atribút XML nesie IČ DPH odberateľa, ktoré sa v kontrole A1 porovnáva s hodnotou SK7020000944? | Architekt |
| 18 | **Nová:** Je pole Obchodný partner editovateľné aj v statusoch 2 a 3? V UC sú tam dnes editovateľné iba Poznámka spracovateľa, Číslo bankového účtu a Spôsob úhrady. Podľa odpovede biznisu má byť dohľadanie dodávateľa dostupné vo všetkých statusoch, kde je faktúra editovateľná - to by znamenalo doplniť Obchodného partnera medzi editovateľné polia pre statusy 2 a 3. | Biznis |
| 19 | Jazyk názvov polí (EN vs SK). | Biznis, UX |
| 20 | Role a oprávnenia (kto smie editovať, holdovať, rušiť, púšťať do iProc). | Biznis |
| 21 | Smeruje preklik z iProc na obraz faktúry priamo do eInvoice, alebo cez DMS? Ak cez DMS, čo sa tam ukladá a kto to tam ukladá? Ak priamo do eInvoice, ako sa zachová kompatibilita s existujúcimi linkami do DMS? | Architekt |

## Biznis zadanie

Spracovateľ (Prevádzková účtáreň) v prvej fáze skontroluje každú došlú faktúru: vyhodnotí, či je potrebné meniť editovateľné polia, obohatí dáta a označí faktúru do statusu 6. na odoslanie - eInvoice ju následne automaticky odošle do iProc. Automatické vyhodnotenie, ktoré faktúry môžu ísť priamo do iProc bez zásahu, je požiadavka na rozvoj do budúcna.

**Detail faktúry**

- Vzorom zoskupenia a logiky usporiadania polí je obrazovka z iProc (cover sheet / likvidačný list) - vizuál sa čo najviac približuje faktúre a obsahuje dáta, ktoré sa pri faktúre kontrolujú
- V detaile musia byť viditeľné všetky položky vizualizované v stĺpcoch zoznamu faktúr + Remittance information / Invoice note
- Farebná vizualizácia podľa xls platí aj pre detail (napr. červená bunka pri prázdnom Č. pôvodnej faktúry pre typy 381/383)
- História zmien sa zobrazuje priamo v detaile pod výsledkami automatizovaných kontrol (UC-203)
- **Akcie na zmenu stavu sú dostupné aj na obrazovke detailu.** Ak spracovateľ posúdi, že faktúra nevyžaduje úpravy, vie ju odoslať, odložiť alebo stornovať priamo z detailu bez otvorenia obrazovky Úprava faktúry

**Úprava faktúry**

- Úprava údajov prebieha na samostatnej obrazovke, oddelenej od detailu, aby sa zobrazenie a editácia nemiešali (požiadavka vývoja)
- Na obrazovke Úprava faktúry musí spracovateľ vidieť obraz faktúry a údaje súčasne. Podľa obrazu dopĺňa údaje, ktoré vyťažené dáta neobsahujú alebo ktoré je potrebné overiť - napríklad spôsob úhrady (hotovosť, kreditná karta, vyúčtovacia faktúra) a popis
- Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto
- Ak spracovateľ opustí obrazovku Úprava faktúry a reálne niečo zmenil, systém ho vyzve, či si želá zmeny uložiť alebo zrušiť
- **Spracovateľ musí vedieť dohľadať a vybrať dodávateľa** vo všetkých statusoch, v ktorých je faktúra editovateľná. Platí to pre priradenie dodávateľa, ktorého kontrola A nenašla, aj pre opravu nesprávne priradeného dodávateľa

**Obraz faktúry**

- Obraz faktúry je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zostáva nemenný rovnako ako došlé XML, aj keď spracovateľ niektoré polia počas spracovania mení
- Obraz faktúry musí obsahovať všetky dáta, ktoré dodávateľ poslal
- Vykonané zmeny sa nepremietajú do obrazu faktúry; sú viditeľné v histórii zmien
- V statusoch 2 a 3 musí byť možné obraz faktúry uložiť ako súbor, preferovane vo formáte PDF. Spracovateľ ho stiahne na úložisko a priloží k požiadavke do iProc na založenie dodávateľa (status 2) alebo čísla bankového účtu (status 3). V ostatných statusoch uloženie obrazu nie je nutnosťou, ale ak to technicky pôjde, biznis to uvíta
- Automatické priloženie obrazu priamo do iProc je vedené ako rozvoj do budúcna
- Obraz faktúry musí byť dostupný na preklik z aplikácie iProc. Spôsob prepojenia je otvorený - viď otázka 21

**Prílohy faktúry**

- Prílohy k faktúre sa majú preklápať do iProc spolu s obrazom faktúry
- Prílohy musí byť možné stiahnuť a uložiť na interné úložisko

Faktúry sa nemažú. Namiesto vymazania (dnešná prax v ABBYY pri duplicite) sa mení status na 7. stornované (Cancelled/Zamietnutá) s povinnou poznámkou. Všetky zmeny sú logované s TB spracovateľa; TB spracovateľa, ktorý označí faktúru na odoslanie, sa posiela do iProc cez abbyy webservice ako Verified by.

Popis stavov tu: Stavový model došlej faktúry

## Aktéri

- Hlavný aktér: Používateľ eInvoice (presné role OTVORENY BOD - viď otázka 20)
- Systém: eInvoice

## Spúšťač

Spracovateľ vyberie došlú faktúru zo zoznamu faktúr (UC-201) - klikne na detail pri vybranej faktúre.

## Vstupné podmienky

> - Spracovateľ je prihlásený a má oprávnenie na prístup
> - Spracovateľ vybral faktúru zo zoznamu (UC-201), alebo otvoril faktúru vyžadujúcu zásah
> - Faktúra existuje v DB eInvoice: prijatá od Poštára, XML uložené tak, ako prišlo
> - Systém vykonal automatizované kontroly a priradil faktúre status

## Automatizované kontroly pri zaevidovaní

eInvoice vykoná pri zaevidovaní validácie v uvedenom poradí a na základe výsledku priradí faktúre status.

| Kontrola | Predmet | Pravidlo | Výsledok |
|---|---|---|---|
| **A1** | Správnosť IČ DPH odberateľa | Odberateľom je pri dodávateľskej faktúre Tatra banka. Systém identifikuje IČ DPH odberateľa na faktúre a porovná ho s hodnotou **SK7020000944**. Kontrola neprebieha voči ODS - v dátach z ODS sú iba údaje dodávateľa | Pri zhode žiadna akcia. Pri nezhode faktúra pokračuje v procese, ale zobrazí sa oranžový warning vo Výsledkoch automatizovaných kontrol |
| **A** | Existencia dodávateľa | Kontrola podľa IČO alebo DIČ voči exportu SAP v ODS; musí nájsť aspoň jeden záznam | Ak nenájde záznam, status 2 |
| **B** | Existencia bankového účtu dodávateľa | Kontrola čísla účtu uvedeného na faktúre voči dátam v ODS | Ak nenájde, status 3 |
| **C** | Duplicita faktúry | DIČ + číslo faktúry + dátum vystavenia faktúry (viď otázka 4) | Pri zhode status 4. Pre typ faktúry 384 je výsledok vždy DUPLICITA |

(OTVORENY BOD: ktorý atribút XML nesie IČ DPH odberateľa - viď otázka 17)

**Výnimka z kontroly B:** pri kombinácii typ faktúry 381 (dobropis) a spôsob úhrady 1. No item selected (preddefinovaný) systém nemusí vykonať kontrolu čísla účtu a automaticky faktúre zaeviduje číslo účtu SK11 1100 0000 0020 0100 3800.

**Všeobecné pravidlo:** ak je faktúre pridelené číslo účtu SK11 1100 0000 0020 0100 3800, aplikácia kontrolu BU ignoruje.

**Chýbajúce DIČ v kmeňových dátach.** Ak eInvoice pri kontrole A dodávateľa nájde (napríklad podľa IČO), ale v dátach z ODS mu chýba DIČ, pole sa na obrazovke vyčervení. Ide o upozornenie pre spracovateľa, aby zabezpečil doplnenie DIČ do kmeňových dát. Spracovanie faktúry pokračuje ďalej a nezastavuje sa.

## Hlavný tok

1. Spracovateľ vyberie konkrétnu došlú faktúru zo zoznamu
2. Systém otvorí detail došlej faktúry a automaticky otvorí obraz faktúry; obraz je needitovaný a zobrazuje faktúru tak, ako prišla od dodávateľa
3. Systém zobrazí výsledky automatizovaných kontrol. Pri nesúlade v údajoch dodávateľa (IČ DPH dodávateľa, meno alebo názov, adresa) zobrazí aj hodnotu z ODS pre porovnanie. Pri nezhode IČ DPH odberateľa zobrazí oranžový warning s uvedením správnej hodnoty
4. Systém zobrazí históriu zmien faktúry pod výsledkami automatizovaných kontrol (UC-203)
5. Systém zobrazí sekciu s ostatnými údajmi z faktúry v poradí, ako sú uvedené v XML
6. Spracovateľ otvorí obrazovku Úprava faktúry
7. Systém zobrazí obrazovku Úprava faktúry spolu s obrazom faktúry tak, aby spracovateľ videl obraz a údaje súčasne
8. Spracovateľ prechádza a upravuje editovateľné polia; systém pri prechode tabulátorom podsvieti zdroj údaja v obraze faktúry nažlto
9. Spracovateľ zvolí akciu Na odoslanie
10. Systém uloží rozpracované zmeny, pri každej zmene uchová pôvodnú hodnotu a zaloguje ju s TB spracovateľa, a zmení status faktúry na 6. na odoslanie
11. Systém zapíše TB spracovateľa do poľa Verified by
12. Systém automaticky odošle dáta faktúry vrátane príloh a obrazu faktúry do iProc cez abbyy webservice a zmení status na 8. odoslané do iProc
13. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201); faktúra zo záložky Na spracovanie vypadne
14. Systém prijme z iProc informáciu o zaevidovaní s interným číslom faktúry (voucher number), zapíše ho do poľa Číslo interné a zmení status na 10. zaevidované v iProc

(OTVORENY BOD: či sa prílohy preklápajú automaticky pri odoslaní, alebo ich prikladá spracovateľ - viď otázka 15)

## Alternatívny tok

**AT 1a - Dodávateľ neexistuje (kontrola A)**

Podmienka: Kontrola A (existencia dodávateľa podľa IČO alebo DIČ voči exportu SAP v ODS) zlyhá.

1. Systém ponechá faktúru v statuse 2. dodávateľ nezaevidovaný.
2. Spracovateľ po nakliknutí do detailu pri uložení povinne vloží Poznámku spracovateľa s informáciou o spôsobe riešenia (zadanie requestu do iProc na založenie kmeňových dát).
3. Spracovateľ uloží obraz faktúry ako súbor a priloží ho k požiadavke do iProc (viď AT 2a).
4. **Spracovateľ môže alternatívne dodávateľa dohľadať a priradiť manuálne podľa AT 1d.** Ak sa dodávateľ v ODS nachádza a kontrola B prebehne validne, faktúra sa preradí do statusu 1 bez čakania na automatické overenie.
5. Systém automaticky overuje zadanie dodávateľa v ODS o 6:00 a 12:00. Po zaevidovaní dodávateľa aj čísla účtu vykoná kontrolu C duplicita a podľa výsledku pridelí faktúre status 4. duplicita alebo 1. na spracovanie.
6. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Faktúru preto nie je možné odblokovať automatickým overením v ten istý deň, v ktorom bol request na založenie zadaný.
7. Keď spracovateľ nadobudne všetky potrebné informácie na spracovanie faktúry, faktúru buď spracuje, alebo ju stornuje (status 7 s povinnou poznámkou).

- Tok pokračuje krokom 1.
- Poznámka: požiadavka na založenie nového dodávateľa aj nového čísla účtu sa zadáva do iProc.

**AT 1b - Bankové spojenie neexistuje alebo nie je schválené (kontrola B)**

Podmienka: Kontrola B (existencia bankového účtu dodávateľa voči dátam v ODS) zlyhá (stop spracovania).

1. Systém ponechá faktúru v statuse 3. BU neexistuje; platí rovnaká povinná poznámka a automatické overovanie ODS o 6:00 a 12:00 ako v AT 1a, vrátane pravidla, že údaje budú v ODS najskôr nasledujúci deň.
2. Spracovateľ uloží obraz faktúry ako súbor a priloží ho k požiadavke do iProc na založenie čísla účtu (viď AT 2a).
3. Ak bankové spojenie nie je schválené z dôvodu zmluvne dohodnutého iného účtu a faktúra neobsahuje daň: spracovateľ informuje dodávateľa, že jeho faktúru neakceptujeme, a požiada ho o opravu. Faktúru označí ako stornovanú (status 7, povinná poznámka).
4. Ak by takáto faktúra obsahovala daň: spracovateľ informuje dodávateľa, že faktúru neakceptujeme, s uvedením dôvodu, a požiada ho o vystavenie dobropisu. Faktúra sa zaeviduje s bankovým spojením SK11 1100 0000 0020 0100 3800 a spôsobom úhrady 4. Vyúčtovacia faktúra. Faktúra sa eviduje do iProc a po zaslaní dobropisu sa obe položky spárujú na forced approval holde.
5. Keď spracovateľ nadobudne všetky potrebné informácie, faktúru buď spracuje, alebo ju stornuje.

- Tok pokračuje krokom 6, resp. UC končí v statuse 7 (bod 3).

**AT 1c - Duplicitné číslo faktúry (kontrola C)**

Podmienka: Kontrola C zistí duplicitu (DIČ + číslo faktúry + dátum vystavenia), alebo je typ faktúry 384 (vždy duplicita).

1. Systém priradí faktúre status 4. duplicita.
2. Spracovateľ posúdi oprávnenosť duplicity.
3. Neoprávnená duplicita (napr. dodávateľ navýši sumu a pošle novú faktúru s tým istým číslom): spracovateľ preradí faktúru do statusu 7. stornované (povinná poznámka) a požiada dodávateľa o vystavenie dobropisu alebo ťarchopisu k pôvodne zaevidovanej faktúre; pri dobropise dodávateľ následne zašle novú faktúru s novým poradovým číslom.
4. Oprávnená duplicita (napr. dodávateľ bez poradového čísla používa číslo zmluvy - iná poistná udalosť alebo iné auto): spracovateľ upraví číslo faktúry doplnením hviezdičky a dátumu (pole je v statuse 4 editovateľné) a zmení status na 6. na odoslanie.

- Tok pokračuje krokom 10 (bod 4), resp. UC končí v statuse 7 (bod 3).

**AT 1d - Dohľadanie a výber dodávateľa**

Podmienka: Spracovateľ potrebuje priradiť alebo zmeniť dodávateľa faktúry. Nastáva v dvoch situáciách:

- kontrola A dodávateľa nenašla a faktúra je v statuse 2. dodávateľ nezaevidovaný, alebo
- kontrola A dodávateľa našla, ale spracovateľ pri porovnaní s obrazom faktúry zistí, že je nesprávny.

1. Spracovateľ začne zadávať názov dodávateľa alebo jeho IČO do vyhľadávacieho poľa (napríklad OMV, alebo začiatok IČO 123).
2. Systém priebežne ponúka všetkých dodávateľov z ODS, ktorí zadanej hodnote zodpovedajú, spolu s údajmi, s ktorými sú v ODS založení, aby ich spracovateľ vedel odlíšiť.
3. Spracovateľ vyberie vhodného dodávateľa z ponuky.
4. Systém prevezme od vybraného dodávateľa Číslo dodávateľa iProc (Oracle vendor ID) a Číslo dodávateľa SAP.
5. Systém vyhodnotí kontrolu B aj kontrolu C: kontrolu B nad bankovým účtom faktúry voči kmeňovým dátam vybraného dodávateľa a kontrolu C duplicita nad novým kľúčom. Ak je kontrola B negatívna, faktúra sa preradí do statusu 3. BU neexistuje. Ak kontrola C zistí duplicitu, faktúra sa preradí do statusu 4. duplicita. Ak sú obe kontroly validné, faktúra sa preradí do statusu 1. na spracovanie, prípadne v ňom zostáva.
6. Systém zaloguje zmenu dodávateľa s pôvodnou a novou hodnotou a s TB spracovateľa.

- Tok pokračuje krokom 8.
- Funkcia je dostupná vo všetkých statusoch, v ktorých je faktúra editovateľná. (OTVORENY BOD: editovateľnosť poľa Obchodný partner v statusoch 2 a 3 - viď otázka 18)
- (OTVORENY BOD: či systém pri malom nesúlade sám ponúkne kandidátov bez zadania hodnoty spracovateľom - viď otázka 16)

**AT 2a - Uloženie obrazu faktúry pre požiadavku do iProc**

Podmienka: Faktúra je v statuse 2. dodávateľ nezaevidovaný alebo 3. BU neexistuje a spracovateľ zadáva požiadavku do iProc na založenie kmeňových dát.

1. Spracovateľ zvolí akciu na uloženie obrazu faktúry.
2. Systém vygeneruje obraz faktúry ako súbor vo formáte PDF.
3. Spracovateľ súbor stiahne na úložisko a následne ho priloží k požiadavke do iProc na založenie dodávateľa alebo čísla bankového účtu.

- Tok pokračuje krokom 2.
- V statusoch 2 a 3 je uloženie obrazu povinnou funkcionalitou. V ostatných statusoch nie je nutnosťou; ak to technicky pôjde, biznis ho uvíta.
- Rozvoj do budúcna: možnosť priložiť obraz priamo do iProc bez manuálneho sťahovania.

**AT 2b - Stiahnutie príloh faktúry**

Podmienka: Spracovateľ potrebuje prílohy faktúry mimo aplikácie.

1. Spracovateľ zvolí akciu na stiahnutie príloh.
2. Systém sprístupní prílohy na stiahnutie.
3. Spracovateľ uloží prílohy na interné úložisko.

- Tok pokračuje krokom 2.
- (OTVORENY BOD: čo je obsahom príloh a v akej podobe prichádzajú - viď otázka 14)

**AT 6a - Spracovanie bez úpravy údajov**

Podmienka: Spracovateľ posúdi, že faktúra nevyžaduje žiadne úpravy editovateľných polí.

1. Spracovateľ neotvorí obrazovku Úprava faktúry.
2. Spracovateľ zvolí akciu Na odoslanie, Odložiť alebo Stornovať priamo na obrazovke detailu.
3. Systém vykoná zmenu stavu; pri statusoch vyžadujúcich poznámku ju povinne vyžiada.
4. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201).

- Tok pokračuje krokom 10 pri odoslaní, resp. UC končí pri odložení alebo stornovaní.
- Dostupnosť akcií riadi stavový model, nie posúdenie potreby úprav - to je na spracovateľovi.

**AT 8a - Spôsob úhrady 2 až 5**

Podmienka: Spracovateľ vyberie spôsob úhrady 2. Platené v hotovosti, 3. Služobná KK, 4. Vyúčtovacia faktúra alebo 5. Zaplatené zálohou.

1. Systém automaticky zaeviduje faktúre číslo účtu SK11 1100 0000 0020 0100 3800.
2. Systém vykoná nové kontroly: kontrolu B vyhodnotí ako validnú a pokračuje kontrolou C duplicita.
3. Systém podľa výsledku zmení status na 4. duplicita (ak existuje) alebo 1. na spracovanie.

- Tok pokračuje krokom 8.

**AT 8b - Dobropis (typ 381)**

Podmienka: Typ faktúry je dobropis (do abbyy webservicu sa posielajú záporné čísla; iProc pustí dobropis iba v zápornej hodnote).

1. Systém prerobí všetky čiastky faktúry na záporné hodnoty a zmenu zaloguje.
2. Poznámka: akceptovateľný, ale nepreferovaný variant je sprístupniť polia suma na úhradu, suma faktúry, základ dane a daň na manuálnu zmenu spracovateľom - z pohľadu chybovosti a prácnosti sa nepreferuje.

- Tok pokračuje krokom 8.

**AT 8c - Chýbajúci variabilný symbol**

Podmienka: Variabilný symbol nie je uvedený na faktúre.

1. Systém automaticky doplní variabilný symbol ako prvých 10 číselných znakov poradového čísla faktúry, s podmienkou, že variabilný symbol nezačína 0.
2. Pole Variabilný symbol je v tomto prípade editovateľné spracovateľom; ak je VS uvedený na faktúre, pole je needitovateľné.

- Tok pokračuje krokom 8.

**AT 8d - Číslo faktúry obsahuje medzery**

Podmienka: V poradovom čísle faktúry sú medzery.

1. Systém v dátach určených na editáciu neželané medzery odstráni.
2. Ak sa odstránenie nedá technicky nastaviť, pole Číslo faktúry musí byť editovateľné vždy, keď sa medzery identifikujú.

- Tok pokračuje krokom 8.
- Poznámka: najmenej vhodným riešením je povoliť editáciu čísla faktúry naprieč všetkými statusmi.

**AT 9a - Uloženie zmien bez zmeny statusu**

Podmienka: Spracovateľ chce zmeny uložiť, ale faktúru zatiaľ neodoslať.

1. Spracovateľ zvolí akciu Uložiť.
2. Systém uloží zmeny, zaloguje ich s TB spracovateľa a zobrazí detail faktúry s aktualizovanými údajmi.

- UC končí (faktúra zostáva v pôvodnom statuse).

**AT 9b - Zrušenie rozpracovaných zmien**

Podmienka: Spracovateľ nechce vykonané zmeny uložiť.

1. Spracovateľ zvolí akciu Zrušiť.
2. Systém sa vráti na detail faktúry bez uloženia zmien; žiadna zmena sa nezaloguje.

- Tok pokračuje krokom 2.

**AT 9c - Opustenie obrazovky Úprava faktúry bez potvrdenia**

Podmienka: Spracovateľ opúšťa obrazovku Úprava faktúry bez toho, aby zvolil Uložiť alebo Zrušiť.

1. Ak spracovateľ v poliach reálne vykonal zmenu, systém zobrazí výzvu, či si želá zmeny uložiť alebo zrušiť.
2. Ak spracovateľ zvolí uloženie, systém pokračuje podľa AT 9a.
3. Ak spracovateľ zvolí zrušenie, systém pokračuje podľa AT 9b.
4. Ak spracovateľ na obrazovke nič nezmenil, výzva sa nezobrazí a systém sa vráti na detail faktúry.

**AT 9d - Odloženie faktúry**

Podmienka: Spracovateľ potrebuje faktúru odložiť.

1. Spracovateľ zvolí akciu Odložiť na obrazovke Úprava faktúry alebo na obrazovke detailu.
2. Systém uloží rozpracované zmeny a zmení status na 5. odložené; povinne vyžaduje vloženie Poznámky spracovateľa (vizualizácia príčiny pozastavenia).
3. Ak je poznámka už zadaná a spracovateľ mení status na status vyžadujúci poznámku, systém vyzve na pridanie ďalšej poznámky alebo edit existujúcej.
4. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201).

- UC končí (faktúra čaká na doriešenie).

**AT 9e - Stornovanie faktúry**

Podmienka: Faktúra sa nemá spracovať (napr. duplicita; faktúra prišla cez ABBYY aj cez eInvoice a je bez dane; nesprávne bankové spojenie s vrátením dodávateľovi; Compliance neschválil dodávateľa).

1. Spracovateľ zvolí akciu Stornovať na obrazovke Úprava faktúry alebo na obrazovke detailu.
2. Systém zmení status na 7. stornované; povinne vyžaduje vloženie Poznámky spracovateľa.
3. Systém faktúru neodosiela do iProc; faktúra sa nemaže a zostáva dohľadateľná.
4. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201).

- UC končí.

**AT 12a - Serverová chyba iProc**

1. Systém zapíše chybu do poľa Chyba z iProc a zmení status na 9. vrátené z iProc.
2. Systém automaticky opakuje zasielanie smerom do iProc v 15-minútových intervaloch, celkovo 3x.
3. Po 3 neúspešných pokusoch systém zašle notifikáciu na skupinu faktury_a_upomienky o serverovom probléme; faktúry zostávajú evidované v statuse 9.

**AT 12b - Kvalitatívna chyba v odoslaných dátach**

1. Systém zapíše chybu do poľa Chyba z iProc a zmení status na 9. vrátené z iProc.
2. Systém nevykonáva automatické opakovanie - je potrebný zásah spracovateľa.
3. Spracovateľ upraví dáta na obrazovke Úprava faktúry a zvolí akciu Na odoslanie; tok pokračuje krokom 10 hlavného toku.

Príklad chybovej hlášky pri neexistujúcom čísle objednávky v poli Objednávka číslo:

```
iProc - TBwsProc: NOK:E046 Invalid PO Number. P_H_PO_NUMBER = 86000753
```

**AT 12c - Chyba duplicity z iProc**

1. Systém pri chybovom hlásení o existujúcom zázname s daným poradovým číslom faktúry systémovo preradí položku do statusu 4. duplicita.
2. Spracovateľ ju rieši podľa AT 1c (storno s povinnou poznámkou, alebo úprava poradového čísla a status 6).
3. Ak by preradenie do statusu 4 nebolo možné a faktúra by zostala v statuse 9, číslo faktúry musí byť editovateľné aj v statuse 9.

Príklad chybovej hlášky pri duplicite:

```
Processing Notes
iProc - TBwsProc: NOK:E037 Invoice creation error - Invoice number for this supplier or party already exists
```

**Rozlíšenie pôvodu chyby podľa prefixu hlášky**

| Prefix chybovej hlášky | Strana chyby | Koho kontaktovať |
|---|---|---|
| `iProc - TBwsProc:` | ORACLE | RBI: konstantin.redko@rbinternational.com, arun.kumar-external@rbinternational.com |
| `iProc - TBwsProc FC:` | ABBYY | SysAppChannels (Morávek), prípadne dodávateľ (EXE) |

Poznámka pre vývoj: prefixy sú si veľmi podobné, líšia sa iba reťazcom `FC`. Pri implementácii je potrebné porovnávať presne, inak môže byť chyba z ABBYY vyhodnotená ako chyba z ORACLE.

Poznámka: uvedené kontakty sú aktuálne k dátumu spísania UC. Odporúča sa ich udržiavať v prevádzkovej dokumentácii, nie v UC špecifikácii.

**Známe chybové kódy**

| Kód | Význam | Alternatívny tok |
|---|---|---|
| E037 | Číslo faktúry pre daného dodávateľa alebo stranu už existuje (duplicita) | AT 12c |
| E046 | Neplatné číslo objednávky | AT 12b |

**AT 14a - Reexport faktúry zo statusu 10**

Podmienka: Faktúra je zaevidovaná v iProc (status 10) a je potrebná oprava vyťažených dát.

1. Spracovateľ preradí faktúru do statusu 11. Reexport faktúry; systém povinne vyžaduje vloženie Poznámky spracovateľa.
2. Spracovateľ upraví chybu v dátach na obrazovke Úprava faktúry; v statuse 11 je editovateľné aj poradové číslo faktúry.
3. Spracovateľ zvolí akciu Na odoslanie; tok pokračuje krokom 10.
4. Vyexportovaná položka dostane v iProc nové číslo voucher number.
5. Procesne spracovateľ pôvodne zaevidovanú faktúru v iProc vystornuje s dôvodom, že išla na opravu vstupných dát (činnosť v iProc, mimo eInvoice).

## Diagram tokov

## Výstupné podmienky

- Úspech: faktúra je zaevidovaná v iProc (status 10), pole Číslo interné obsahuje voucher number, pole Verified by obsahuje TB spracovateľa, všetky zmeny (manuálne aj systémové) sú zalogované, pôvodné XML aj obraz faktúry zostávajú nezmenené
- Alternatívne ukončenia: faktúra je v statuse 2, 3 alebo 5 (čaká na doriešenie, s povinnou poznámkou), v statuse 7 (stornovaná, neodosiela sa), alebo v statuse 9 (vrátená z iProc, čaká na retry alebo zásah)
- Po odoslaní, odložení aj stornovaní je spracovateľ vrátený do zoznamu faktúr v danej záložke (UC-201)

## Opis Obrazoviek + Validácie

FIGMA:

### Obrazovka 1: Detail dodávateľskej faktúry (zobrazenie)

**Popis:** Zoskupenie a usporiadanie polí podľa vzoru cover sheet z iProc; všetky položky zo stĺpcov zoznamu + Remittance information / Invoice note; výsledky automatizovaných kontrol so zobrazením hodnôt z ODS pri nesúlade v údajoch dodávateľa; história zmien pod výsledkami kontrol (UC-203); prílohy faktúry; farebná vizualizácia podľa xls. Obrazovka je určená na zobrazenie a kontrolu, údaje sa na nej neupravujú.

**Akcie na obrazovke**

| Akcia | Účel | Dostupnosť |
|---|---|---|
| Otvoriť Úpravu faktúry | Prechod na obrazovku, kde sa údaje menia | Podľa editovateľnosti faktúry v danom statuse |
| Na odoslanie | Odoslanie faktúry bez otvorenia obrazovky úpravy | Podľa stavového modelu |
| Odložiť | Odloženie faktúry; vyžaduje poznámku | Podľa stavového modelu |
| Stornovať | Stornovanie faktúry; vyžaduje poznámku | Podľa stavového modelu |
| Uložiť obraz faktúry | Stiahnutie PDF pre priloženie k požiadavke do iProc | Povinne v statusoch 2 a 3; v ostatných statusoch vítané |
| Stiahnuť prílohy | Uloženie príloh na interné úložisko | Ak faktúra obsahuje prílohy |

### Obrazovka 2: Úprava faktúry

**Popis:** Samostatná obrazovka určená na úpravu editovateľných polí, oddelená od detailu. Editovateľné polia sú vizuálne odlíšené (podsvietené). Obraz faktúry je zobrazený súčasne s údajmi, aby spracovateľ mohol podľa neho dopĺňať údaje ako spôsob úhrady a popis. Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto.

**Vyhľadanie a výber dodávateľa**

| Vlastnosť | Popis |
|---|---|
| Kedy sa používa | Pri priradení dodávateľa, ktorého kontrola A nenašla (status 2), aj pri oprave nesprávne priradeného dodávateľa |
| Vstup | Spracovateľ zadá časť názvu dodávateľa alebo časť IČO |
| Správanie systému | Systém priebežne ponúka všetkých dodávateľov z ODS, ktorí zadanej hodnote zodpovedajú |
| Obsah ponuky | Pri každom kandidátovi sú viditeľné údaje, s ktorými je založený v ODS, aby ich spracovateľ vedel odlíšiť |
| Výsledok výberu | Systém prevezme Oracle vendor ID a SAP ID a spustí kontrolu B aj C |
| Dostupnosť | Vo všetkých statusoch, v ktorých je faktúra editovateľná (viď otázka 18) |

**Rozloženie akčnej lišty**

| Umiestnenie | Akcie | Charakter |
|---|---|---|
| Vľavo | Na odoslanie, Odložiť, Stornovať | Menia stav faktúry |
| Vpravo | Uložiť, Zrušiť | Práca s rozpracovanými zmenami |

**Pravidlá pre akcie**

| Pravidlo |
|---|
| Akcie meniace stav najprv uložia rozpracované zmeny a až potom zmenia stav faktúry. Spracovateľ tak na jednej obrazovke uloží aj odošle faktúru |
| Po odoslaní, odložení aj stornovaní systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201) |
| Akcie Odložiť, Stornovať a reexport vyžadujú vloženie Poznámky spracovateľa |
| Pri opustení obrazovky bez zvolenia akcie systém vyzve spracovateľa, či si želá zmeny uložiť alebo zrušiť - iba ak reálne niečo zmenil |
| Akcia Zrušiť vráti spracovateľa na detail faktúry bez uloženia; žiadna zmena sa nezaloguje |

### Obrazovka 3: Obraz faktúry

**Popis:** Obraz faktúry sa automaticky otvára pri vstupe do detailu a je zobrazený aj na obrazovke Úprava faktúry. Je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zmeny vykonané počas spracovania sa doň nepremietajú. Usporiadanie polí vychádza z poradia, v akom sú uvedené v XML.

**Funkcie obrazu**

| Funkcia | Popis | Dostupnosť |
|---|---|---|
| Automatické otvorenie | Pri vstupe do detailu došlej faktúry | Vždy |
| Súčasné zobrazenie s údajmi | Na obrazovke Úprava faktúry | Vždy |
| Podsvietenie zdroja údaja | Pri prechode tabulátorom po editovateľných poliach sa zdroj podsvieti nažlto | Na obrazovke Úprava faktúry |
| Uloženie ako PDF | Spracovateľ stiahne obraz na úložisko a priloží ho k požiadavke do iProc na založenie dodávateľa alebo čísla účtu | Povinne v statusoch 2 a 3; v ostatných statusoch vítané, nie povinné |
| Preklik z iProc | Obraz musí byť dostupný z aplikácie iProc; spôsob prepojenia je otvorený (viď otázka 21) | Vždy |

### Polia

| Názov poľa | Validation | Mandatory | Editable | Popis | Poznámka |
|---|---|---|---|---|---|
| Poznámka spracovateľa | Povinná pri statusoch 2, 3, 5, 7 a pri reexporte zo statusu 10; pri opakovanej zmene statusu výzva na ďalšiu poznámku | Podmienene áno | Áno | Informácia o spôsobe riešenia položky | - |
| Obchodný partner (dodávateľ) | Kontrola A; vyhľadanie a výber dodávateľa z ODS podľa časti názvu alebo IČO | N/A | Áno - výber z ODS vo všetkých statusoch, v ktorých je faktúra editovateľná, vrátane statusov 2 a 3 | Názov dodávateľa | Pri zmene systém prevezme Oracle vendor ID a SAP ID a spustí kontrolu B aj C (AT 1d). V statuse 2 je to jediná cesta, ako faktúru odblokovať bez čakania na automatické overenie ODS. (OTVORENY BOD: viď otázka 18) |
| Číslo bankového účtu | Kontrola B | (xls) | Logika výberu podľa xls | - | NEŠPECIFIKOVANÉ detailne (xls) |
| Číslo faktúry | Kontrola C - duplicita: DIČ + číslo + dátum vystavenia - stop | N/A | Iba v statuse 4 (fallback: aj v statuse 9) a v statuse 11; inak needitovateľné | Úprava hviezdičkou a dátumom pri oprávnenej duplicite | Systém odstraňuje medzery (AT 8d) |
| Variabilný symbol | VS nezačína 0 (pri automatickom doplnení) | N/A | Iba ak VS nie je uvedený na faktúre; inak needitovateľné | Automatické doplnenie: prvých 10 číselných znakov čísla faktúry | - |
| Č. pôvodnej faktúry | Prázdne pri type 381/383 - bunka svieti načerveno; spracovateľ povinný naplniť | Podmienene áno (381/383) | Áno pre typy 381/383 | Automaticky doťahované | Ak je naplnené pri inom type, faktúra sa považuje za ťarchopis |
| Dátum dodania (dátum daňového dokladu) | (xls) | (xls) | Áno pre typy 381 a 383 a pre faktúry považované za ťarchopis | Systém prednastaví dátum prijatia faktúry ako dátum dodania | Obdobná logika ako v iProc |
| Spôsob úhrady | Výber 2-5 spúšťa automatiku účtu a nové kontroly (AT 8a) | N/A (default 1) | Áno (hodnoty 2-5) | 1. No item selected (prednastavené), 2. Platené v hotovosti, 3. Služobná KK, 4. Vyúčtovacia faktúra, 5. Zaplatené zálohou | Spracovateľ overuje voči obrazu faktúry; kód 54 a 55 = služobná KK, kód 10 = platené v hotovosti |
| Základ dane | (xls) | (xls) | Áno pre zálohové faktúry typu 386; inak needitovateľné | - | - |
| Výška dane | (xls) | (xls) | Áno pre zálohové faktúry typu 386; inak needitovateľné | - | - |
| Popis | (xls - stĺpec Popis) | (xls) | Áno | Text z faktúry min. z prvej položky; automatiky: číslo zákazníka na začiatok, PDP0DPH pri PDP alebo type AE, VS pri faktúrach od telekomunikačného operátora | Spracovateľ dopĺňa podľa obrazu faktúry |
| Číslo NO (Objednávka číslo) | (xls) | (xls) | Áno | Doplní spracovateľ; ideál: eInvoice nájde v texte hodnotu 86xxxxxx a doplní automaticky | Neexistujúce číslo objednávky vracia iProc s kódom E046 |
| Číslo zmluvy | (xls) | (xls) | Áno | Doplní spracovateľ; ideál: hodnota 86xxxxxx v kombinácii so spojením číslo zmluvy alebo CPA | - |
| IČ DPH dodávateľa | Kontrola voči ODS - upozornenie so zobrazením hodnoty z ODS | N/A | Nie | IČ DPH dodávateľa z faktúry | Ak dodávateľ v kmeňových dátach nemá DIČ, pole sa vyčervení; spracovanie pokračuje |
| IČ DPH odberateľa | Kontrola A1 - porovnanie s pevnou hodnotou SK7020000944; neporovnáva sa voči ODS | N/A | Nie | IČ DPH odberateľa uvedené dodávateľom na faktúre | Pri nezhode oranžový warning; faktúra pokračuje v procese. (OTVORENY BOD: atribút XML - viď otázka 17) |
| Číslo dodávateľa iProc | Read-only | N/A | Nie | Oracle vendor ID z ODS | Pri zmene dodávateľa sa prevezme od vybraného dodávateľa |
| Číslo dodávateľa SAP | Read-only | N/A | Nie | SAP ID z ODS | Pri zmene dodávateľa sa prevezme od vybraného dodávateľa |
| Verified by | Read-only | N/A | Nie | TB toho, kto označil faktúru na odoslanie; posiela sa do iProc cez abbyy webservice | - |
| Číslo interné | Read-only | N/A | Nie | Voucher number z iProc po zaevidovaní | - |
| Chyba z iProc | Read-only | N/A | Nie | Chybové hlásenie pri statuse 9; prichádza v poli Processing Notes | Pôvod chyby sa rozlišuje podľa prefixu hlášky |
| Prílohy faktúry | Read-only | N/A | Nie | Prílohy k faktúre; preklápajú sa do iProc spolu s obrazom faktúry a je možné ich stiahnuť na interné úložisko | (OTVORENY BOD: obsah a podoba príloh - viď otázka 14) |
| Výsledky kontrol | A1 IČ DPH odberateľa (voči pevnej hodnote), A dodávateľ, B bankové spojenie, C duplicita; upozornenia pri IČ DPH dodávateľa, mene alebo názve a adrese so zobrazením hodnoty z ODS | N/A | Nie | Inšpirácia ABBYY: červený flag polí; výber dodávateľa pri malom nesúlade | - |
| Dodávateľ-Zamestnanec | - | - | Rozvoj do budúcna | Dnes sa napĺňa priamo v iProc | Nie je súčasťou prvej fázy |
| Payment Reason Comment | - | - | Rozvoj do budúcna | | |

**Poznámka pre vývoj:** editovateľnosť polí je podmienená kombináciou statusu a typu faktúry - ide o dve nezávislé dimenzie, ktoré treba vyhodnocovať súčasne. V statusoch 8 a 10 nie je možné editovať žiadne dáta okrem prípadu reexportu. Statusy 2 a 3 nie sú manuálne meniteľné, spracovateľ ich opúšťa zmenou dát - dohľadaním dodávateľa, výberom iného bankového účtu alebo výberom spôsobu úhrady. Každá zmena poľa aj každá automatická systémová zmena musí byť zalogovaná s pôvodnou a novou hodnotou, dátumom a TB spracovateľa.

**Poznámka pre testovanie:** overiť editovateľnosť každého poľa vo všetkých statusoch a pre všetky typy faktúr; dohľadanie dodávateľa v statuse 2 aj opravu nesprávneho dodávateľa v statuse 1 vrátane spustenia kontroly B aj C; automatické doplnenie variabilného symbolu vrátane podmienky, že nezačína 0; odstránenie medzier z čísla faktúry; červenú bunku pri prázdnom Č. pôvodnej faktúry pre typy 381 a 383; vyhodnotenie faktúry ako ťarchopisu pri naplnenom Č. pôvodnej faktúry pri inom type; prepočet dobropisu na záporné hodnoty; automatiku účtu pri spôsobe úhrady 2 až 5; oranžové upozornenie pri nezhode IČ DPH odberateľa; uloženie obrazu faktúry v statusoch 2 a 3; návrat do zoznamu po odoslaní, odložení aj stornovaní; a celý tok odoslania 6 → 8 → 10 vrátane troch chybových scenárov.

## API

- **iProc:** odoslanie dát faktúry na existujúce API iProc vrátane príloh a obrazu faktúry; spätné potvrdenie s interným číslom dokladu a číslom dodávateľa; rozlíšenie chyby iProc vs eInvoice. Chybové hlásenie prichádza v poli Processing Notes; pôvod chyby sa rozlišuje podľa prefixu (`iProc - TBwsProc:` = ORACLE, `iProc - TBwsProc FC:` = ABBYY). Známe kódy: E037 duplicita, E046 neplatné číslo objednávky. (OTVORENY BOD: úplná špecifikácia API, zoznam chybových kódov, technický spôsob spätného prenosu, spôsob preklopenia príloh)
- **ODS:** export dodávateľov (názov, bankové spojenie, IČO, IČ DPH, adresa) ako referenčné dáta kontrol. V ODS sú iba údaje dodávateľa - kontrola A1 nad IČ DPH odberateľa preto voči ODS neprebieha. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Rozhranie musí umožniť vyhľadávanie dodávateľa podľa časti názvu alebo IČO a vrátiť údaje, s ktorými je dodávateľ v ODS založený, aby spracovateľ vedel kandidátov odlíšiť (AT 1d). (OTVORENY BOD: technický spôsob a frekvencia - viď otázka 11)
- **Obraz faktúry:** dodávateľské faktúry prídu do eInvoice výhradne v XML. Polia sa v obraze usporiadajú podľa poradia, v akom sú uvedené v XML; mapovanie atribútov sa nevykonáva. Polymorfné XML atribúty vyriešia architekt a vývoj. Obraz musí byť generovateľný ako PDF na stiahnutie, povinne v statusoch 2 a 3 (viď otázka 12), a dostupný na preklik z iProc (viď otázka 21).
- **Prílohy faktúry:** prílohy sa preklápajú do iProc spolu s obrazom faktúry a musí byť možné ich stiahnuť a uložiť na interné úložisko. (OTVORENY BOD: technická realizácia a väzba na FileNet DMS - viď otázka 13; obsah a podoba príloh - viď otázka 14)
- **Digitálny poštár:** zdroj došlej faktúry (XML uložené tak, ako prišlo)

## Vysvetlivky pojmov

| Pojem | Význam |
|---|---|
| BU | Bankové spojenie (bankový účet dodávateľa) |
| VS | Variabilný symbol |
| Dobropis | Doklad, ktorým dodávateľ znižuje pôvodne fakturovanú sumu. V podkladoch sa preň používajú skratky DBP (pri type faktúry 381) aj DBS - obe označujú ten istý doklad. V texte UC sa používa slovo dobropis |
| Ťarchopis | Doklad, ktorým dodávateľ zvyšuje pôvodne fakturovanú sumu (typ faktúry 383) |
| PDP | Prenesenie daňovej povinnosti |
| PDP0DPH | Text, ktorý sa automaticky dopĺňa na začiatok poľa Popis pri faktúre s prenesením daňovej povinnosti alebo type AE |
| K4O | Verifikácia editovaných polí iným používateľom, než ktorý ich editoval. V eInvoice sa neuplatňuje, kontrola správnosti prebieha v iProc |
| CPA | Označenie používané pri čísle zmluvy |
| Voucher number | Interné číslo faktúry pridelené v iProc; zapisuje sa do poľa Číslo interné |
| Forced approval hold | Stav v iProc, na ktorom sa spárujú faktúra a k nej vystavený dobropis |
| Oracle vendor ID | Číslo dodávateľa v iProc; povinný údaj pre zaevidovanie faktúry |
| Polymorfné XML atribúty | Peppol štruktúra umožňuje, aby ten istý atribút niesol rôzne typy obsahu; riešenie je na strane architektúry |

---

## Čo som pri kontrole overoval

**Krížové odkazy na otázky.** Prešiel som všetkých 21 otázok a každý odkaz v texte. Odkazy na otázky 4, 11, 12, 13, 14, 15, 16, 17, 18, 20 a 21 sedia. Otázky 1 až 3, 5 až 10 a 19 sa v texte odkazom nespomínajú, čo je v poriadku - sú to samostatné otvorené body.

**Odkazy na alternatívne toky.** AT 1a odkazuje na AT 1d a AT 2a, AT 1b na AT 2a, AT 12c na AT 1c, AT 9c na AT 9a a AT 9b. Všetky cieľové toky existujú.

**Odkazy na kroky hlavného toku.** Overil som, že AT 6a, AT 8a až 8d, AT 9a až 9e, AT 12b a AT 14a odkazujú na kroky, ktoré v štrnásťkrokovom toku existujú a dávajú zmysel.

**Konzistencia troch nových pravidiel.** Dohľadanie dodávateľa je teraz opísané rovnako v biznis zadaní, v AT 1a, v AT 1d, v tabuľke Obrazovky 2 aj v riadku Obchodný partner. Dostupnosť akcií na detaile je rovnako v biznis zadaní, AT 6a a v tabuľke akcií Obrazovky 1. Návrat do zoznamu je uvedený v kroku 13, AT 6a, AT 9d, AT 9e, vo výstupných podmienkach aj v pravidlách Obrazovky 2.

**Odkazy na iné UC.** Všade UC-201 a UC-203 podľa aktuálneho stromu v Confluence, žiadne pozostatky po UC-06 ani UC-902.

**Čo zostalo neuzavreté a vedome.** Poznámka pre vývoj hovorí, že statusy 2 a 3 nie sú manuálne meniteľné, a zároveň je v nich dostupné dohľadanie dodávateľa. Nie je to rozpor - status sa nemení priamo akciou, ale zmenou dát, ktorá spustí kontroly. Formuloval som to tak, aby to bolo zrejmé, ale ak by to pri review niekoho zaskočilo, je to vysvetliteľné.
