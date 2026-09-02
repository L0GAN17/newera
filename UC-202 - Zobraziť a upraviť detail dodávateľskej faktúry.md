# UC\-202 \- Zobraziť a upraviť detail dodávateľskej faktúry

## Obsah





## Otázky


Podľa akých údajov spracovateľ vyhľadáva správneho dodávateľa pri oprave nesprávnej identifikácie - IČO, DIČ, názov, alebo kombinácia? Odpoved:spracovateľ vyhľadáva dodávateľa najčastejšie podľa názvu dodávateľa alebo podľa IČA dodávateľa napr. ak začnem písať OMV (prípadne číslo iča: 123… ) tak mi ponúkne všetkých dodávateľov s týmto názvom (ičom) kde su viditeľné údaje s ktorými su založený v ODS a spracovateľ si následnej vyberie vhodného dodávateľa.

Je oprava dodávateľa dostupná vo všetkých statusoch, kde je faktúra editovateľná, alebo len vo vybraných? Odpoved: ano vo všetkých statusoch kde je faktúra editovateľná

Spúšťa sa po manuálnej zmene dodávateľa aj kontrola C duplicita, alebo iba kontrola B? Odpoved: aj kontrola C aj kontrola B

Má systém pri malom nesúlade sám ponúknuť zoznam kandidátov na dodávateľa, ako to robí ABBYY, alebo dodávateľa vyhľadáva spracovateľ? Odpoved:

Zobrazí sa výzva na uloženie zmien vždy pri opustení obrazovky Úprava faktúry, alebo iba vtedy, keď spracovateľ reálne niečo zmenil? Odpoved: vyzva na za uloženie sa zobrazí iba vtedy ak spracovateľ realne niečo zmenil

Do ktorej záložky zoznamu sa spracovateľ vráti po odoslaní faktúry? Po odoslaní faktúra vypadne zo záložky Na spracovanie, takže v pôvodnej záložke ju už neuvidí. Odpoved: po odoslaní faktúry ,faktúra vypadne zo záložky Na spracovanie a spracovateľ sa vráti do zoznamu faktúr v danej záložke

Platí návrat na zoznam aj po odložení a stornovaní faktúry, alebo iba po odoslaní? Odpoved: ano aj po odložení a stornovaní

Zostávajú akcie na zmenu stavu aj na obrazovke detailu, alebo sú výhradne na obrazovke Úprava faktúry? Ak výhradne na úprave, spracovateľ bude musieť otvoriť úpravu aj pri samotnom stornovaní bez zmeny údajov. Odpoved:akcie na zmenu stavu možu byť aj na obrazovke detailu, ak na faktúre nebudu potrebné žiadne úpravy už z detailu budem vedieť faktúru odoslať na spracovanie, prípadne stornovanie,alebo dozistenie

Je akcia na uloženie obrazu faktúry dostupná len v statusoch 2 a 3, alebo vo všetkých statusoch? Odpoved: v statuse 2 a 3 sa musí dať uložiť obraz, v ostatných statusoch to nie je nutnosť ale ak sa to bude dať bolo by to fajn.

Ide o manuálne stiahnutie súboru spracovateľom, alebo má aplikácia obraz automaticky priložiť k požiadavke do iProc? Odpoved: ide o manuálne stiahnutie súboru spracovateľom na uložisko a následné priloženie do požiadavky do iProc. Na rozvoj do budúcna - možnosť priložiť priamo do iProc.

Ak sa PDF negeneruje, aký iný čitateľný formát je pre iProc akceptovateľný? Odpoved: preferujeme generovanie pdf


## Biznis zadanie

Spracovateľ (Prevádzková účtáreň) v prvej fáze skontroluje každú došlú faktúru: vyhodnotí, či je potrebné meniť editovateľné polia, obohatí dáta a označí faktúru do statusu 6. na odoslanie - eInvoice ju následne automaticky odošle do iProc. Automatické vyhodnotenie, ktoré faktúry môžu ísť priamo do iProc bez zásahu, je požiadavka na rozvoj do budúcna.

**Detail faktúry**

- Vzorom zoskupenia a logiky usporiadania polí je obrazovka z iProc (cover sheet / likvidačný list) - vizuál sa čo najviac približuje faktúre a obsahuje dáta, ktoré sa pri faktúre kontrolujú
- V detaile musia byť viditeľné všetky položky vizualizované v stĺpcoch zoznamu faktúr + Remittance information / Invoice note
- Farebná vizualizácia podľa xls platí aj pre detail (napr. červená bunka pri prázdnom Č. pôvodnej faktúry pre typy 381/383)
- História zmien sa zobrazuje priamo v detaile pod výsledkami automatizovaných kontrol (UC-203)
- **Ak faktúra nevyžaduje žiadne úpravy, spracovateľ ju vie odoslať, odložiť alebo stornovať priamo z detailu**, bez otvorenia obrazovky Úprava faktúry

**Úprava faktúry**

- Úprava údajov prebieha na samostatnej obrazovke, oddelenej od detailu, aby sa zobrazenie a editácia nemiešali (požiadavka vývoja)
- Na obrazovke Úprava faktúry musí spracovateľ vidieť obraz faktúry a údaje súčasne. Podľa obrazu dopĺňa údaje, ktoré vyťažené dáta neobsahujú alebo ktoré je potrebné overiť - napríklad spôsob úhrady (hotovosť, kreditná karta, vyúčtovacia faktúra) a popis
- Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto
- Ak spracovateľ opustí obrazovku Úprava faktúry a **reálne niečo zmenil**, systém ho vyzve, či si želá zmeny uložiť alebo zrušiť

**Obraz faktúry**

- Obraz faktúry je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zostáva nemenný rovnako ako došlé XML, aj keď spracovateľ niektoré polia počas spracovania mení
- Obraz faktúry musí obsahovať všetky dáta, ktoré dodávateľ poslal
- Vykonané zmeny sa nepremietajú do obrazu faktúry; sú viditeľné v histórii zmien
- Obraz faktúry musí byť možné uložiť ako súbor v čitateľnom formáte (preferovane PDF). Spracovateľ ho stiahne na úložisko a priloží k požiadavke na založenie dodávateľa (status 2) alebo čísla bankového účtu (status 3). Automatické priloženie priamo do iProc je vedené ako rozvoj do budúcna
- Obraz faktúry musí byť dostupný na preklik z aplikácie iProc. Spôsob prepojenia je otvorený - viď otázka 23

Faktúry sa nemažú. Namiesto vymazania (dnešná prax v ABBYY pri duplicite) sa mení status na 7. stornované (Cancelled/Zamietnutá) s povinnou poznámkou. Všetky zmeny sú logované s TB spracovateľa; TB spracovateľa, ktorý označí faktúru na odoslanie, sa posiela do iProc cez abbyy webservice ako Verified by.

Popis stavov tu: [Stavový model došlej faktúry](https://tbsk.atlassian.net/wiki/x/GBoyAQ)

## Aktéri

- Hlavný aktér: Používateľ eInvoice ( presné role OTVORENY BOD)
- Systém: eInvoice

## Spúšťač

Spracovateľ vyberie došlú faktúru zo zoznamu faktúr [(UC-201)](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/20058927/UC-201+-+Zobrazi+a+vyh+ada+zoznam+dod+vate+sk+ch+fakt+r) - klikne na detail pri vybranej faktúre

## Vstupné podmienky

Používateľ vyberie faktúru zo zoznamu (UC-06), alebo otvorí faktúru vyžadujúcu zásah

- Spracovateľ je prihlásený a má oprávnenie na prístup
- Spracovateľ vybral faktúru zo zoznamu (UC-201)
- Faktúra existuje v DB eInvoice: prijatá od Poštára, XML uložené tak, ako prišlo
- Systém vykonal automatizované kontroly a priradil faktúre status

### Automatizované kontroly pri zaevidovaní

eInvoice vykoná pri zaevidovaní validácie v uvedenom poradí a na základe výsledku priradí faktúre status.

| KontrolaPredmetPravidloVýsledok**A1**Správnosť IČ DPH odberateľaOdberateľom je pri dodávateľskej faktúre Tatra banka. Systém identifikuje IČ DPH odberateľa na faktúre a porovná ho s hodnotou **SK7020000944**. Kontrola neprebieha voči ODS - v dátach z ODS sú iba údaje dodávateľaPri zhode žiadna akcia. Pri nezhode faktúra pokračuje v procese, ale zobrazí sa oranžový warning vo Výsledkoch automatizovaných kontrol**A**Existencia dodávateľaKontrola podľa IČO alebo DIČ voči exportu SAP v ODS; musí nájsť aspoň jeden záznamAk nenájde záznam, status 2**B**Existencia bankového účtu dodávateľaKontrola čísla účtu uvedeného na faktúre voči dátam v ODSAk nenájde, status 3**C**Duplicita faktúryDIČ + číslo faktúry + dátum vystavenia faktúry (viď otázka 4)Pri zhode status 4. Pre typ faktúry 384 je výsledok vždy DUPLICITA**Výnimka z kontroly B:** pri kombinácii typ faktúry 381 (dobropis) a spôsob úhrady 1. No item selected (preddefinovaný) systém nemusí vykonať kontrolu čísla účtu a automaticky faktúre zaeviduje číslo účtu SK11 1100 0000 0020 0100 3800.**Všeobecné pravidlo:** ak je faktúre pridelené číslo účtu SK11 1100 0000 0020 0100 3800, aplikácia kontrolu BU ignoruje.**Chýbajúce DIČ v kmeňových dátach.** Ak eInvoice pri kontrole A dodávateľa nájde (napríklad podľa IČO), ale v dátach z ODS mu chýba DIČ, pole sa na obrazovke vyčervení. Ide o upozornenie pre spracovateľa, aby zabezpečil doplnenie DIČ do kmeňových dát. Spracovanie faktúry pokračuje ďalej a nezastavuje sa. |  |  |  |
| --- | --- | --- | --- |
| Kontrola | Predmet | Pravidlo | Výsledok |
| **A1** | Správnosť IČ DPH odberateľa | Odberateľom je pri dodávateľskej faktúre Tatra banka. Systém identifikuje IČ DPH odberateľa na faktúre a porovná ho s hodnotou **SK7020000944**. Kontrola neprebieha voči ODS - v dátach z ODS sú iba údaje dodávateľa | Pri zhode žiadna akcia. Pri nezhode faktúra pokračuje v procese, ale zobrazí sa oranžový warning vo Výsledkoch automatizovaných kontrol |
| **A** | Existencia dodávateľa | Kontrola podľa IČO alebo DIČ voči exportu SAP v ODS; musí nájsť aspoň jeden záznam | Ak nenájde záznam, status 2 |
| **B** | Existencia bankového účtu dodávateľa | Kontrola čísla účtu uvedeného na faktúre voči dátam v ODS | Ak nenájde, status 3 |
| **C** | Duplicita faktúry | DIČ + číslo faktúry + dátum vystavenia faktúry (viď otázka 4) | Pri zhode status 4. Pre typ faktúry 384 je výsledok vždy DUPLICITA |

## Hlavný tok

1. Spracovateľ zvolí možnosť Detial pri konkrétnej došlej faktúre zo zoznamu
2. Systém otvorí detail došlej faktúry a automaticky otvorí obraz faktúry ako PDF na ďalšej obrazovke; obraz je needitovaný a zobrazuje faktúru tak, ako prišla od dodávateľa (otvorený bod na vývoj, bude sa to dať??)
3. Systém zobrazí výsledky automatizovaných kontrol. Pri nesúlade v údajoch dodávateľa (IČ DPH dodávateľa, meno alebo názov, adresa) zobrazí aj hodnotu z ODS pre porovnanie. Pri nezhode IČ DPH odberateľa zobrazí oranžový warning s uvedením správnej hodnoty
4. Systém zobrazí históriu zmien faktúry pod výsledkami automatizovaných kontrol (UC-203)
5. Systém zobrazí sekciu s ostatnými údajmi z faktúry v poradí, ako sú uvedené v XML
6. Spracovateľ otvorí obrazovku Úprava faktúry
7. Systém zobrazí obrazovku Úprava faktúry spolu s obrazom faktúry tak, aby spracovateľ videl obraz a údaje súčasne
8. Spracovateľ prechádza a upravuje editovateľné polia; systém pri prechode tabulátorom podsvieti zdroj údaja v obraze faktúry nažlto
9. Spracovateľ zvolí akciu Na odoslanie
10. Systém uloží rozpracované zmeny, pri každej zmene uchová pôvodnú hodnotu a zaloguje ju s TB spracovateľa, a zmení status faktúry na 6. na odoslanie
11. Systém zapíše TB spracovateľa do poľa Verified by
12. Systém automaticky odošle dáta faktúry do iProc cez abbyy webservice a zmení status na 8. odoslané do iProc
13. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201); faktúra zo záložky Na spracovanie vypadne
14. Systém prijme z iProc informáciu o zaevidovaní s interným číslom faktúry (voucher number), zapíše ho do poľa Číslo interné a zmení status na 10. zaevidované v iProc

## Alternatívne toky

**AT 1a - Dodávateľ neexistuje (kontrola A)**

Podmienka: Kontrola A (existencia dodávateľa podľa IČO alebo DIČ voči exportu SAP v ODS) zlyhá.

1. Systém ponechá faktúru v statuse 2. dodávateľ nezaevidovaný.
2. Spracovateľ po nakliknutí do detailu pri uložení povinne vloží Poznámku spracovateľa s informáciou o spôsobe riešenia (zadanie requestu na založenie kmeňových dát).
3. Spracovateľ uloží obraz faktúry ako súbor a priloží ho k požiadavke (viď AT 2a).
4. Systém automaticky overuje zadanie dodávateľa v ODS o 6:00 a 12:00. Po zaevidovaní dodávateľa aj čísla účtu vykoná kontrolu C duplicita a podľa výsledku pridelí faktúre status 4. duplicita alebo 1. na spracovanie.
5. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Faktúru preto nie je možné odblokovať v ten istý deň, v ktorom bol request na založenie zadaný.
6. Keď spracovateľ nadobudne všetky potrebné informácie na spracovanie faktúry, faktúru buď spracuje, alebo ju stornuje (status 7 s povinnou poznámkou).

- Tok pokračuje krokom 1.
- (OTVORENY BOD: či sa požiadavka zadáva do iProc alebo do UPI - viď otázka 17) Odpoved: požiadavka na založenie nového dodávateľa a nového čísla účtu sa zadáva do Iproc

**AT 1b - Bankové spojenie neexistuje alebo nie je schválené (kontrola B)**

Podmienka: Kontrola B (existencia bankového účtu dodávateľa voči dátam v ODS) zlyhá (stop spracovania).

1. Systém ponechá faktúru v statuse 3. BU neexistuje; platí rovnaká povinná poznámka a automatické overovanie ODS o 6:00 a 12:00 ako v AT 1a, vrátane pravidla, že údaje budú v ODS najskôr nasledujúci deň.
2. Spracovateľ uloží obraz faktúry ako súbor a priloží ho k požiadavke na založenie čísla účtu (viď AT 2a).
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

**AT 1d - Nesprávne identifikovaný dodávateľ**

Podmienka: Kontrola A dodávateľa nájde, ale spracovateľ pri porovnaní s obrazom faktúry zistí, že systém priradil nesprávneho dodávateľa.

1. Spracovateľ na obrazovke Úprava faktúry vyhľadá správneho dodávateľa v dátach z ODS.
2. Spracovateľ vyberie správneho dodávateľa zo zoznamu nájdených záznamov.
3. Systém prevezme od vybraného dodávateľa Číslo dodávateľa iProc (Oracle vendor ID) a Číslo dodávateľa SAP.
4. Systém vyhodnotí **kontrolu B aj kontrolu C**: kontrolu B nad bankovým účtom faktúry voči kmeňovým dátam nového dodávateľa a kontrolu C duplicita nad novým kľúčom. Ak je kontrola B negatívna, faktúra sa preradí do statusu 3. BU neexistuje. Ak kontrola C zistí duplicitu, faktúra sa preradí do statusu 4. duplicita. Ak sú obe kontroly validné, faktúra pokračuje v spracovaní.
5. Systém zaloguje zmenu dodávateľa s pôvodnou a novou hodnotou a s TB spracovateľa.

- Tok pokračuje krokom 8.
- (OTVORENY BOD: podľa akých údajov sa dodávateľ vyhľadáva, v ktorých statusoch je funkcia dostupná a či systém sám ponúka kandidátov - viď otázky 18 až 20) Odpoved: spracovateľ vyhľadáva dodávateľa najčastejšie podľa názvu dodávateľa alebo podľa IČA dodávateľa napr. ak začnem písať  OMV (prípadne číslo iča: 123… ) tak mi ponúkne všetkých dodávateľov s týmto názvom (ičom) kde su viditeľné údaje s ktorými su založený v ODS a spracovateľ si následnej vyberie vhodného dodávateľa. Toto by malo byť dostupné vo všetkých statusoch kde je faktúra editovateľná.

**AT 2a - Uloženie obrazu faktúry pre požiadavku na založenie kmeňových dát**

Podmienka: Faktúra je v statuse 2. dodávateľ nezaevidovaný alebo 3. BU neexistuje a spracovateľ zadáva požiadavku na založenie kmeňových dát.

1. Spracovateľ zvolí akciu na uloženie obrazu faktúry.
2. Systém vygeneruje obraz faktúry ako súbor v čitateľnom formáte, preferovane PDF.
3. Spracovateľ súbor stiahne na úložisko a následne ho priloží k požiadavke na založenie dodávateľa alebo čísla bankového účtu.

- Tok pokračuje krokom 2.
- Rozvoj do budúcna: možnosť priložiť obraz priamo do iProc bez manuálneho sťahovania.
- (OTVORENY BOD: dostupnosť akcie v ostatných statusoch a akceptovateľný formát - viď otázky 21 a 22)

**AT 6a - Faktúra nevyžaduje úpravy**

Podmienka: Spracovateľ pri kontrole detailu zistí, že faktúra nevyžaduje žiadne úpravy editovateľných polí.

1. Spracovateľ nezobrazí obrazovku Úprava faktúry.
2. Spracovateľ zvolí akciu Na odoslanie, Odložiť alebo Stornovať priamo na obrazovke detailu.
3. Systém vykoná zmenu stavu; pri statusoch vyžadujúcich poznámku ju povinne vyžiada.

- Tok pokračuje krokom 10, resp. UC končí podľa zvolenej akcie.

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

Podmienka: Spracovateľ opúšťa obrazovku Úprava faktúry bez toho, aby zvolil Uložiť alebo Zrušiť, a **v poliach reálne vykonal zmenu**.

1. Systém zobrazí výzvu, či si spracovateľ želá zmeny uložiť alebo zrušiť.
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
3. Po 3 neúspešných pokusoch systém zašle notifikáciu na skupinu faktury\_a\_upomienky o serverovom probléme; faktúry zostávajú evidované v statuse 9.

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
| --- | --- | --- |
| `iProc - TBwsProc:` | ORACLE | RBI: <konstantin.redko@rbinternational.com>, <arun.kumar-external@rbinternational.com> |
| `iProc - TBwsProc FC:` | ABBYY | SysAppChannels (Morávek), prípadne dodávateľ (EXE) |

Poznámka pre vývoj: prefixy sú si veľmi podobné, líšia sa iba reťazcom `FC`. Pri implementácii je potrebné porovnávať presne, inak môže byť chyba z ABBYY vyhodnotená ako chyba z ORACLE.

Poznámka: uvedené kontakty sú aktuálne k dátumu spísania UC. Odporúča sa ich udržiavať v prevádzkovej dokumentácii, nie v UC špecifikácii.

**Známe chybové kódy**

| Kód | Význam | Alternatívny tok |
| --- | --- | --- |
| E037 | Číslo faktúry pre daného dodávateľa alebo stranu už existuje (duplicita) | AT 12c |
| E046 | Neplatné číslo objednávky | AT 12b |

**AT 14a - Reexport faktúry zo statusu 10**

Podmienka: Faktúra je zaevidovaná v iProc (status 10) a je potrebná oprava vyťažených dát.

1. Spracovateľ preradí faktúru do statusu 11. Reexport faktúry; systém povinne vyžaduje vloženie Poznámky spracovateľa.
2. Spracovateľ upraví chybu v dátach na obrazovke Úprava faktúry; v statuse 11 je editovateľné aj poradové číslo faktúry.
3. Spracovateľ zvolí akciu Na odoslanie; tok pokračuje krokom 10.
4. Vyexportovaná položka dostane v iProc nové číslo voucher number.
5. Procesne spracovateľ pôvodne zaevidovanú faktúru v iProc vystornuje s dôvodom, že išla na opravu vstupných dát (činnosť v iProc, mimo eInvoice).

## Výstupné podmienky

- Úspech: faktúra je zaevidovaná v iProc (status 10), pole Číslo interné obsahuje voucher number, pole Verified by obsahuje TB spracovateľa, všetky zmeny (manuálne aj systémové) sú zalogované, pôvodné XML aj obraz faktúry zostávajú nezmenené
- Alternatívne ukončenia: faktúra je v statuse 2, 3 alebo 5 (čaká na doriešenie, s povinnou poznámkou), v statuse 7 (stornovaná, neodosiela sa), alebo v statuse 9 (vrátená z iProc, čaká na retry alebo zásah)
- Po odoslaní, odložení aj stornovaní je spracovateľ vrátený do zoznamu faktúr v danej záložke (UC-201)

## Opis Obrazoviek + Validácie



FIGMA: 

#### Obrazovka 1: Detail dodávateľskej faktúry (zobrazenie)

Zoskupenie a usporiadanie polí podľa vzoru cover sheet z iProc; všetky položky zo stĺpcov zoznamu + Remittance information / Invoice note; výsledky automatizovaných kontrol so zobrazením hodnôt z ODS pri nesúlade v údajoch dodávateľa; história zmien pod výsledkami kontrol (UC-203); farebná vizualizácia podľa xls. Obrazovka je určená na zobrazenie a kontrolu, údaje sa na nej neupravujú.

Obsahuje akciu na otvorenie obrazovky Úprava faktúry a akcie na zmenu statusu (Na odoslanie, Odložiť, Stornovať). Viď otázka 17.


**Akcie na obrazovke**

| Akcia | Účel |
| --- | --- |
| Otvoriť Úpravu faktúry | Prechod na obrazovku, kde sa údaje menia |
| Na odoslanie | Dostupné, ak faktúra nevyžaduje úpravy |
| Odložiť | Dostupné, ak faktúra nevyžaduje úpravy; vyžaduje poznámku |
| Stornovať | Dostupné, ak faktúra nevyžaduje úpravy; vyžaduje poznámku |
| Uložiť obraz faktúry | Stiahnutie súboru pre priloženie k požiadavke na založenie kmeňových dát |

#### Obrazovka 2: Úprava faktúry

**Popis:** Samostatná obrazovka určená na úpravu editovateľných polí, oddelená od detailu. Editovateľné polia sú vizuálne odlíšené (podsvietené). Obraz faktúry je zobrazený súčasne s údajmi, aby spracovateľ mohol podľa neho dopĺňať údaje ako spôsob úhrady a popis. Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto.

**Rozloženie akčnej lišty**

| Akcie |  |
| --- | --- |
| Na odoslanie, Odložiť, Stornovať | Menia stav faktúry |
| Uložiť, Zrušiť | Práca s rozpracovanými zmenami |

**Pravidlá pre akcie**

| Pravidlo |
| --- |
| Akcie meniace stav najprv uložia rozpracované zmeny a až potom zmenia stav faktúry. Spracovateľ tak na jednej obrazovke uloží aj odošle faktúru |
| Po odoslaní, odložení aj stornovaní systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201) |
| Akcie Odložiť, Stornovať a reexport vyžadujú vloženie Poznámky spracovateľa |
| Pri opustení obrazovky bez zvolenia akcie systém vyzve spracovateľa, či si želá zmeny uložiť alebo zrušiť - iba ak reálne niečo zmenil |
| Akcia Zrušiť vráti spracovateľa na detail faktúry bez uloženia; žiadna zmena sa nezaloguje |

#### Obrazovka 3: Obraz faktúry Odpoved/Popis potrebný zapracovat do UC: Tento obraz faktúry potrebujeme v statuse 2 ( nový dodávateľ) a 3 ( nové číslo účtu) vedieť uložiť, najlepšie vo formáte pdf (prípadne v inom čitateľnom formáte) nakoľko tento obraz je potrebné vložiť k požiadavke do Iprocu na založenie dodávateľa alebo čisla účtu. 

Popis: Obraz faktúry sa automaticky otvára pri vstupe do detailu a je zobrazený aj na obrazovke Úprava faktúry. Je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zmeny vykonané počas spracovania sa doň nepremietajú. Usporiadanie polí vychádza z poradia, v akom sú uvedené v XML. Obraz musí byť možné otvoriť aj z aplikácie iProc cez DMS linku na preklik.

**Funkcie obrazu**

| **Funkcia** | **Popis** |
| --- | --- |
| Automatické otvorenie | Pri vstupe do detailu došlej faktúry |
| Súčasné zobrazenie s údajmi | Na obrazovke Úprava faktúry |
| Podsvietenie zdroja údaja | Pri prechode tabulátorom po editovateľných poliach sa zdroj podsvieti nažlto |
| Uloženie ako súbor | Preferovane PDF; spracovateľ ho stiahne na úložisko a priloží k požiadavke na založenie kmeňových dát v statusoch 2 a 3 |
| Preklik z iProc | Obraz musí byť dostupný z aplikácie iProc; spôsob prepojenia je otvorený (viď otázka 23) |


Polia

| Názov poľa | Validation | Mandatory | Editable | Popis | Poznámka |
| --- | --- | --- | --- | --- | --- |
| Poznámka spracovateľa | Povinná pri statusoch 2, 3, 5, 7 a pri reexporte zo statusu 10; pri opakovanej zmene statusu výzva na ďalšiu poznámku | Podmienene áno | Áno | Informácia o spôsobe riešenia položky | - |
| Obchodný partner (dodávateľ) | Kontrola A; možnosť manuálneho výberu správneho dodávateľa z ODS | N/A | Áno - výber z ODS | Názov dodávateľa | Pri zmene systém prevezme Oracle vendor ID a SAP ID a spustí kontrolu B aj C (AT 1d) |
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
| IČ DPH odberateľa | Kontrola A1 - porovnanie s pevnou hodnotou SK7020000944; neporovnáva sa voči ODS | N/A | Nie | IČ DPH odberateľa uvedené dodávateľom na faktúre | Pri nezhode oranžový warning; faktúra pokračuje v procese. (OTVORENY BOD: atribút XML - viď otázka 16) |
| Číslo dodávateľa iProc | Read-only | N/A | Nie | Oracle vendor ID z ODS | Pri zmene dodávateľa sa prevezme od nového dodávateľa |
| Číslo dodávateľa SAP | Read-only | N/A | Nie | SAP ID z ODS | Pri zmene dodávateľa sa prevezme od nového dodávateľa |
| Verified by | Read-only | N/A | Nie | TB toho, kto označil faktúru na odoslanie; posiela sa do iProc cez abbyy webservice | - |
| Číslo interné | Read-only | N/A | Nie | Voucher number z iProc po zaevidovaní | - |
| Chyba z iProc | Read-only | N/A | Nie | Chybové hlásenie pri statuse 9; prichádza v poli Processing Notes | Pôvod chyby sa rozlišuje podľa prefixu hlášky |
| Výsledky kontrol | A1 IČ DPH odberateľa (voči pevnej hodnote), A dodávateľ, B bankové spojenie, C duplicita; upozornenia pri IČ DPH dodávateľa, mene alebo názve a adrese so zobrazením hodnoty z ODS | N/A | Nie | Inšpirácia ABBYY: červený flag polí; výber dodávateľa pri malom nesúlade | - |
| Dodávateľ-Zamestnanec | - | - | Rozvoj do budúcna | Dnes sa napĺňa priamo v iProc | Nie je súčasťou prvej fázy |
| Payment Reason Comment | - | - | Rozvoj do budúcna |  |  |


 

## API

- 
    - **iProc:** odoslanie dát faktúry na existujúce API iProc; spätné potvrdenie s interným číslom dokladu a číslom dodávateľa; rozlíšenie chyby iProc vs eInvoice. Chybové hlásenie prichádza v poli Processing Notes; pôvod chyby sa rozlišuje podľa prefixu (`iProc - TBwsProc:` = ORACLE, `iProc - TBwsProc FC:` = ABBYY). Známe kódy: E037 duplicita, E046 neplatné číslo objednávky. (OTVORENY BOD: úplná špecifikácia API, zoznam chybových kódov, technický spôsob spätného prenosu)
    - **ODS:** export dodávateľov (názov, bankové spojenie, IČO, IČ DPH, adresa) ako referenčné dáta kontrol. V ODS sú iba údaje dodávateľa - kontrola A1 nad IČ DPH odberateľa preto voči ODS neprebieha. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Rozhranie musí umožniť aj vyhľadanie dodávateľa spracovateľom pri oprave nesprávnej identifikácie (AT 1d). (OTVORENY BOD: technický spôsob a frekvencia)
    - **Obraz faktúry:** dodávateľské faktúry prídu do eInvoice výhradne v XML. Polia sa v obraze usporiadajú podľa poradia, v akom sú uvedené v XML; mapovanie atribútov sa nevykonáva. Polymorfné XML atribúty vyriešia architekt a vývoj. Obraz musí byť generovateľný ako súbor na stiahnutie (viď otázka 12) a dostupný na preklik z iProc (viď otázka 23).
    - **FileNet DMS:** prílohy faktúry. (OTVORENY BOD: dostupnosť príloh na detaile; súvisí aj s otázkou 23) Odpoved ktoru treba do UC zapracovat: prílohy k fakúre by sa mali preklápať spolu s obrazom faktúry do Iproc. A mali by sa dať stiahnuť a uložiť na interné úložisko.
    - **Digitálny poštár:** zdroj došlej faktúry (XML uložené tak, ako prišlo)
