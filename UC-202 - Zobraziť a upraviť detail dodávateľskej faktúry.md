Prešiel som dodaný súbor. Je v ňom 15 odpovedí, z toho jedna prázdna, plus dve nové požiadavky, ktoré v pôvodnom UC neboli. Overil som ich voči 10b aj voči xls - upozorním na jednu vec, ktorá v ani jednom podklade nie je.

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
- Alternatívne toky
- Diagram tokov
- Výstupné podmienky
- Opis Obrazoviek + Validácie
- API
- Vysvetlivky pojmov

## Zapracované zmeny

**Uzavreté otázky**

1. **Vyhľadanie správneho dodávateľa** prebieha našepkávaním - spracovateľ začne písať názov alebo IČO a systém ponúkne zoznam zodpovedajúcich dodávateľov s údajmi, s ktorými sú založení v ODS.
2. **Oprava dodávateľa je dostupná vo všetkých statusoch**, v ktorých je faktúra editovateľná.
3. **Uloženie obrazu faktúry je povinné v statusoch 2 a 3**, v ostatných statusoch je vítané, ale nie je nutnosťou.
4. **Formát obrazu faktúry: PDF.** Biznis preferuje generovanie PDF, iný formát sa nerieši.
5. **Požiadavka na založenie dodávateľa a čísla účtu sa zadáva do iProc**, nie do UPI. Opravené v biznis zadaní.

**Nová požiadavka**

6. **Prílohy faktúry** sa majú preklápať spolu s obrazom faktúry do iProc a zároveň musí byť možné stiahnuť ich a uložiť na interné úložisko. Doplnené do biznis zadania, alternatívnych tokov, opisu obrazovky aj sekcie API.

**Opravy**

7. Vo vstupných podmienkach opravený odkaz na UC-06 na UC-201.
8. V hlavnom toku opravený preklep v kroku 1 (Detial na Detail).
9. Krížové odkazy na otázky prepísané na názvy otvorených bodov, aby sa pri prečíslovaní nerozišli.

## Otázky

| # | Otvorená otázka | Adresát |
|---|---|---|
| 1 | Má systém pri malom nesúlade sám a bez zásahu spracovateľa ponúknuť zoznam kandidátov na dodávateľa, ako to robí ABBYY? Našepkávanie pri písaní je potvrdené, proaktívne ponúknutie nie. | Biznis |
| 2 | Kľúč kontroly duplicity: bunka C1 xls uvádza DIČ + číslo faktúry + dátum vystavenia, dokument 10b uvádza dodávateľ + číslo faktúry + dátum vystavenia. Chybová hláška z iProc (E037) hovorí o „supplier or party". | Biznis |
| 3 | Rozpor pri editácii súm: na meetingu potvrdené, že sa sumy needitujú; podklad 10a pri zrážkovej dani uvádza editáciu základu dane. | Biznis, Michal Konečný |
| 4 | Aký status a poznámku dostane faktúra, keď kontrola A nájde IČO dodávateľa, ale nenájde Oracle vendor ID? Stavový model tento prípad nerozlišuje od situácie, keď sa dodávateľ nenájde vôbec. | Biznis |
| 5 | Ktorý atribút XML nesie IČ DPH odberateľa, ktoré sa v kontrole A1 porovnáva s hodnotou SK7020000944? | Architekt |
| 6 | Mapovanie na XML chýba pre polia Typ, IČO, IČ DPH, Suma na úhradu, Základ dane, Výška dane a Popis. | Architekt |
| 7 | Akým spôsobom sa vygeneruje PDF obrazu faktúry, ktorý si spracovateľ stiahne? Ide o rozpor so skorším stanoviskom architekta, že PDF sa negeneruje ani neukladá. | Architekt |
| 8 | Smeruje preklik z iProc na obraz faktúry priamo do eInvoice, alebo cez DMS? Ak cez DMS, čo sa tam ukladá a kto to tam ukladá? Ak priamo do eInvoice, ako sa zachová kompatibilita s existujúcimi linkami do DMS? | Architekt |
| 9 | Akým spôsobom sa prílohy preklápajú do iProc - spolu s dátami faktúry cez abbyy webservice, alebo iným kanálom? | Architekt |
| 10 | Odkiaľ prílohy pochádzajú? Prídu ako súčasť XML od Digitálneho poštára, alebo ich pridáva spracovateľ? Ani 10b, ani xls prílohy došlých faktúr neriešia. | Biznis, architekt |
| 11 | Spôsob potvrdenia prijatia faktúry z iProc (technicky). | Michal Konečný, vývoj |
| 12 | Spôsob a frekvencia načítania exportu dodávateľov z ODS a rozhranie pre našepkávanie pri vyhľadaní dodávateľa. | Architekt, vývoj |
| 13 | Hodnoty číselníka spôsobu zaplatenia - doplniť z ABBYY. | Michal Konečný |
| 14 | Zobrazenie a editácia dlhého popisu (skracovanie, limit znakov) - Andrea pošle príklad. | Biznis |
| 15 | Role a oprávnenia - kto smie editovať, meniť stavy, stornovať a odosielať do iProc. | Biznis |

## Biznis zadanie

Spracovateľ (Prevádzková účtáreň) v prvej fáze skontroluje každú došlú faktúru: vyhodnotí, či je potrebné meniť editovateľné polia, obohatí dáta a označí faktúru do statusu 6. na odoslanie - eInvoice ju následne automaticky odošle do iProc. Automatické vyhodnotenie, ktoré faktúry môžu ísť priamo do iProc bez zásahu, je požiadavka na rozvoj do budúcna.

**Detail faktúry**

- Vzorom zoskupenia a logiky usporiadania polí je obrazovka z iProc (cover sheet / likvidačný list) - vizuál sa čo najviac približuje faktúre a obsahuje dáta, ktoré sa pri faktúre kontrolujú
- V detaile musia byť viditeľné všetky položky vizualizované v stĺpcoch zoznamu faktúr + Remittance information / Invoice note
- Farebná vizualizácia podľa xls platí aj pre detail (napr. červená bunka pri prázdnom Č. pôvodnej faktúry pre typy 381/383)
- História zmien sa zobrazuje priamo v detaile pod výsledkami automatizovaných kontrol (UC-203)
- Ak faktúra nevyžaduje žiadne úpravy, spracovateľ ju vie odoslať, odložiť alebo stornovať priamo z detailu, bez otvorenia obrazovky Úprava faktúry

**Úprava faktúry**

- Úprava údajov prebieha na samostatnej obrazovke, oddelenej od detailu, aby sa zobrazenie a editácia nemiešali (požiadavka vývoja)
- Na obrazovke Úprava faktúry musí spracovateľ vidieť obraz faktúry a údaje súčasne. Podľa obrazu dopĺňa údaje, ktoré vyťažené dáta neobsahujú alebo ktoré je potrebné overiť - napríklad spôsob úhrady (hotovosť, kreditná karta, vyúčtovacia faktúra) a popis
- Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto
- Ak spracovateľ opustí obrazovku Úprava faktúry a reálne niečo zmenil, systém ho vyzve, či si želá zmeny uložiť alebo zrušiť
- **Spracovateľ vie opraviť nesprávne identifikovaného dodávateľa vo všetkých statusoch, v ktorých je faktúra editovateľná.** Dodávateľa vyhľadá našepkávaním podľa názvu alebo IČO

**Obraz faktúry a prílohy**

- Obraz faktúry je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zostáva nemenný rovnako ako došlé XML, aj keď spracovateľ niektoré polia počas spracovania mení
- Obraz faktúry musí obsahovať všetky dáta, ktoré dodávateľ poslal
- Vykonané zmeny sa nepremietajú do obrazu faktúry; sú viditeľné v histórii zmien
- **Obraz faktúry musí byť možné uložiť ako PDF súbor. V statusoch 2 a 3 je táto možnosť povinná**, keďže obraz je potrebné priložiť k požiadavke do iProc na založenie dodávateľa alebo čísla bankového účtu. V ostatných statusoch je uloženie obrazu vítané, ale nie je nutnosťou
- Spracovateľ súbor stiahne na úložisko a priloží ho k požiadavke. Automatické priloženie priamo do iProc je vedené ako rozvoj do budúcna
- **Prílohy faktúry sa preklápajú spolu s obrazom faktúry do iProc. Zároveň musí byť možné prílohy stiahnuť a uložiť na interné úložisko**
- Obraz faktúry musí byť dostupný na preklik z aplikácie iProc. Spôsob prepojenia je otvorený

Faktúry sa nemažú. Namiesto vymazania (dnešná prax v ABBYY pri duplicite) sa mení status na 7. stornované (Cancelled/Zamietnutá) s povinnou poznámkou. Všetky zmeny sú logované s TB spracovateľa; TB spracovateľa, ktorý označí faktúru na odoslanie, sa posiela do iProc cez abbyy webservice ako Verified by.

Popis stavov: Stavový model došlej faktúry

## Aktéri

- Hlavný aktér: Používateľ eInvoice (presné role OTVORENY BOD)
- Systém: eInvoice

## Spúšťač

Spracovateľ vyberie došlú faktúru zo zoznamu faktúr (UC-201) - klikne na detail pri vybranej faktúre.

## Vstupné podmienky

> - Spracovateľ je prihlásený a má oprávnenie na prístup
> - Spracovateľ vybral faktúru zo zoznamu (UC-201), alebo otvoril faktúru vyžadujúcu zásah
> - Faktúra existuje v DB eInvoice: prijatá od Poštára, XML uložené tak, ako prišlo
> - Systém vykonal automatizované kontroly a priradil faktúre status

### Automatizované kontroly pri zaevidovaní

eInvoice vykoná pri zaevidovaní validácie v uvedenom poradí a na základe výsledku priradí faktúre status.

| Kontrola | Predmet | Pravidlo | Výsledok |
|---|---|---|---|
| **A1** | Správnosť IČ DPH odberateľa | Odberateľom je pri dodávateľskej faktúre Tatra banka. Systém identifikuje IČ DPH odberateľa na faktúre a porovná ho s hodnotou **SK7020000944**. Kontrola neprebieha voči ODS - v dátach z ODS sú iba údaje dodávateľa | Pri zhode žiadna akcia. Pri nezhode faktúra pokračuje v procese, ale zobrazí sa oranžový warning vo Výsledkoch automatizovaných kontrol |
| **A** | Existencia dodávateľa | Kontrola podľa IČO alebo DIČ voči exportu SAP v ODS; musí nájsť aspoň jeden záznam | Ak nenájde záznam, status 2 |
| **B** | Existencia bankového účtu dodávateľa | Kontrola čísla účtu uvedeného na faktúre voči dátam v ODS | Ak nenájde, status 3 |
| **C** | Duplicita faktúry | DIČ + číslo faktúry + dátum vystavenia faktúry | Pri zhode status 4. Pre typ faktúry 384 je výsledok vždy DUPLICITA |

(OTVORENY BOD: kľúč kontroly C - rozpor medzi xls a 10b; atribút XML pre IČ DPH odberateľa)

**Výnimka z kontroly B:** pri kombinácii typ faktúry 381 (dobropis) a spôsob úhrady 1. No item selected (preddefinovaný) systém nemusí vykonať kontrolu čísla účtu a automaticky faktúre zaeviduje číslo účtu SK11 1100 0000 0020 0100 3800.

**Všeobecné pravidlo:** ak je faktúre pridelené číslo účtu SK11 1100 0000 0020 0100 3800, aplikácia kontrolu BU ignoruje.

**Chýbajúce DIČ v kmeňových dátach.** Ak eInvoice pri kontrole A dodávateľa nájde (napríklad podľa IČO), ale v dátach z ODS mu chýba DIČ, pole sa na obrazovke vyčervení. Ide o upozornenie pre spracovateľa, aby zabezpečil doplnenie DIČ do kmeňových dát. Spracovanie faktúry pokračuje ďalej a nezastavuje sa.

## Hlavný tok

1. Spracovateľ zvolí možnosť Detail pri konkrétnej došlej faktúre zo zoznamu
2. Systém otvorí detail došlej faktúry a automaticky otvorí obraz faktúry ako PDF na ďalšej obrazovke; obraz je needitovaný a zobrazuje faktúru tak, ako prišla od dodávateľa
3. Systém zobrazí výsledky automatizovaných kontrol. Pri nesúlade v údajoch dodávateľa (IČ DPH dodávateľa, meno alebo názov, adresa) zobrazí aj hodnotu z ODS pre porovnanie. Pri nezhode IČ DPH odberateľa zobrazí oranžový warning s uvedením správnej hodnoty
4. Systém zobrazí históriu zmien faktúry pod výsledkami automatizovaných kontrol (UC-203)
5. Systém zobrazí sekciu s ostatnými údajmi z faktúry v poradí, ako sú uvedené v XML, a prílohy faktúry
6. Spracovateľ otvorí obrazovku Úprava faktúry
7. Systém zobrazí obrazovku Úprava faktúry spolu s obrazom faktúry tak, aby spracovateľ videl obraz a údaje súčasne
8. Spracovateľ prechádza a upravuje editovateľné polia; systém pri prechode tabulátorom podsvieti zdroj údaja v obraze faktúry nažlto
9. Spracovateľ zvolí akciu Na odoslanie
10. Systém uloží rozpracované zmeny, pri každej zmene uchová pôvodnú hodnotu a zaloguje ju s TB spracovateľa, a zmení status faktúry na 6. na odoslanie
11. Systém zapíše TB spracovateľa do poľa Verified by
12. Systém automaticky odošle dáta faktúry vrátane príloh do iProc cez abbyy webservice a zmení status na 8. odoslané do iProc
13. Systém vráti spracovateľa do zoznamu faktúr v danej záložke (UC-201); faktúra zo záložky Na spracovanie vypadne
14. Systém prijme z iProc informáciu o zaevidovaní s interným číslom faktúry (voucher number), zapíše ho do poľa Číslo interné a zmení status na 10. zaevidované v iProc

## Alternatívne toky

**AT 1a - Dodávateľ neexistuje (kontrola A)**

Podmienka: Kontrola A (existencia dodávateľa podľa IČO alebo DIČ voči exportu SAP v ODS) zlyhá.

1. Systém ponechá faktúru v statuse 2. dodávateľ nezaevidovaný.
2. Spracovateľ po nakliknutí do detailu pri uložení povinne vloží Poznámku spracovateľa s informáciou o spôsobe riešenia (zadanie requestu do iProc na založenie kmeňových dát).
3. Spracovateľ uloží obraz faktúry ako PDF a priloží ho k požiadavke do iProc (viď AT 2a).
4. Systém automaticky overuje zadanie dodávateľa v ODS o 6:00 a 12:00. Po zaevidovaní dodávateľa aj čísla účtu vykoná kontrolu C duplicita a podľa výsledku pridelí faktúre status 4. duplicita alebo 1. na spracovanie.
5. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Faktúru preto nie je možné odblokovať v ten istý deň, v ktorom bol request na založenie zadaný.
6. Keď spracovateľ nadobudne všetky potrebné informácie na spracovanie faktúry, faktúru buď spracuje, alebo ju stornuje (status 7 s povinnou poznámkou).

- Tok pokračuje krokom 1.

**AT 1b - Bankové spojenie neexistuje alebo nie je schválené (kontrola B)**

Podmienka: Kontrola B (existencia bankového účtu dodávateľa voči dátam v ODS) zlyhá (stop spracovania).

1. Systém ponechá faktúru v statuse 3. BU neexistuje; platí rovnaká povinná poznámka a automatické overovanie ODS o 6:00 a 12:00 ako v AT 1a, vrátane pravidla, že údaje budú v ODS najskôr nasledujúci deň.
2. Spracovateľ uloží obraz faktúry ako PDF a priloží ho k požiadavke do iProc na založenie čísla účtu (viď AT 2a).
3. Ak bankové spojenie nie je schválené z dôvodu zmluvne dohodnutého iného účtu a faktúra neobsahuje daň: spracovateľ informuje dodávateľa, že jeho faktúru neakceptujeme, a požiada ho o opravu. Faktúru označí ako stornovanú (status 7, povinná poznámka).
4. Ak by takáto faktúra obsahovala daň: spracovateľ informuje dodávateľa, že faktúru neakceptujeme, s uvedením dôvodu, a požiada ho o vystavenie dobropisu. Faktúra sa zaeviduje s bankovým spojením SK11 1100 0000 0020 0100 3800 a spôsobom úhrady 4. Vyúčtovacia faktúra. Faktúra sa eviduje do iProc a po zaslaní dobropisu sa obe položky spárujú na forced approval holde.
5. Keď spracovateľ nadobudne všetky potrebné informácie, faktúru buď spracuje, alebo ju stornuje.

- Tok pokračuje krokom 6, resp. UC končí v statuse 7 (bod 3).

**AT 1c - Duplicitné číslo faktúry (kontrola C)**

Podmienka: Kontrola C zistí duplicitu, alebo je typ faktúry 384 (vždy duplicita).

1. Systém priradí faktúre status 4. duplicita.
2. Spracovateľ posúdi oprávnenosť duplicity.
3. Neoprávnená duplicita (napr. dodávateľ navýši sumu a pošle novú faktúru s tým istým číslom): spracovateľ preradí faktúru do statusu 7. stornované (povinná poznámka) a požiada dodávateľa o vystavenie dobropisu alebo ťarchopisu k pôvodne zaevidovanej faktúre; pri dobropise dodávateľ následne zašle novú faktúru s novým poradovým číslom.
4. Oprávnená duplicita (napr. dodávateľ bez poradového čísla používa číslo zmluvy - iná poistná udalosť alebo iné auto): spracovateľ upraví číslo faktúry doplnením hviezdičky a dátumu (pole je v statuse 4 editovateľné) a zmení status na 6. na odoslanie.

- Tok pokračuje krokom 10 (bod 4), resp. UC končí v statuse 7 (bod 3).

**AT 1d - Nesprávne identifikovaný dodávateľ**

Podmienka: Kontrola A dodávateľa nájde, ale spracovateľ pri porovnaní s obrazom faktúry zistí, že systém priradil nesprávneho dodávateľa.

Dostupnosť: vo všetkých statusoch, v ktorých je faktúra editovateľná.

1. Spracovateľ na obrazovke Úprava faktúry začne písať názov dodávateľa alebo jeho IČO.
2. Systém našepkávaním ponúkne zoznam dodávateľov zodpovedajúcich zadanému názvu alebo IČO. Pri každom dodávateľovi v zozname zobrazí údaje, s ktorými je založený v ODS, aby spracovateľ vedel vybrať toho správneho.
3. Spracovateľ vyberie vhodného dodávateľa zo zoznamu.
4. Systém prevezme od vybraného dodávateľa Číslo dodávateľa iProc (Oracle vendor ID) a Číslo dodávateľa SAP.
5. Systém vyhodnotí kontrolu B aj kontrolu C: kontrolu B nad bankovým účtom faktúry voči kmeňovým dátam nového dodávateľa a kontrolu C duplicita nad novým kľúčom. Ak je kontrola B negatívna, faktúra sa preradí do statusu 3. BU neexistuje. Ak kontrola C zistí duplicitu, faktúra sa preradí do statusu 4. duplicita. Ak sú obe kontroly validné, faktúra pokračuje v spracovaní.
6. Systém zaloguje zmenu dodávateľa s pôvodnou a novou hodnotou a s TB spracovateľa.

- Tok pokračuje krokom 8.
- (OTVORENY BOD: či systém pri malom nesúlade sám ponúkne kandidátov bez zásahu spracovateľa)

**AT 2a - Uloženie obrazu faktúry a príloh**

Podmienka: Spracovateľ potrebuje obraz faktúry alebo prílohy mimo aplikácie. V statusoch 2 a 3 ide o povinnú funkcionalitu, keďže obraz je potrebné priložiť k požiadavke do iProc na založenie kmeňových dát.

1. Spracovateľ zvolí akciu na uloženie obrazu faktúry, prípadne príloh.
2. Systém vygeneruje obraz faktúry ako PDF.
3. Spracovateľ súbor stiahne na interné úložisko.
4. Spracovateľ priloží súbor k požiadavke do iProc na založenie dodávateľa alebo čísla bankového účtu.

- Tok pokračuje krokom 2.
- V ostatných statusoch je uloženie obrazu vítané, ale nie je nutnosťou.
- Rozvoj do budúcna: možnosť priložiť obraz priamo do iProc bez manuálneho sťahovania.

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

Podmienka: Spracovateľ opúšťa obrazovku Úprava faktúry bez toho, aby zvolil Uložiť alebo Zrušiť, a v poliach reálne vykonal zmenu.

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

- Úspech: faktúra je zaevidovaná v iProc (status 10), pole Číslo interné obsahuje voucher number, pole Verified by obsahuje TB spracovateľa, prílohy sú preklopené do iProc, všetky zmeny (manuálne aj systémové) sú zalogované, pôvodné XML aj obraz faktúry zostávajú nezmenené
- Alternatívne ukončenia: faktúra je v statuse 2, 3 alebo 5 (čaká na doriešenie, s povinnou poznámkou), v statuse 7 (stornovaná, neodosiela sa), alebo v statuse 9 (vrátená z iProc, čaká na retry alebo zásah)
- Po odoslaní, odložení aj stornovaní je spracovateľ vrátený do zoznamu faktúr v danej záložke (UC-201)

## Opis Obrazoviek + Validácie

FIGMA:

#### Obrazovka 1: Detail dodávateľskej faktúry (zobrazenie)

**Popis:** Zoskupenie a usporiadanie polí podľa vzoru cover sheet z iProc; všetky položky zo stĺpcov zoznamu + Remittance information / Invoice note; výsledky automatizovaných kontrol so zobrazením hodnôt z ODS pri nesúlade v údajoch dodávateľa; história zmien pod výsledkami kontrol (UC-203); prílohy faktúry; farebná vizualizácia podľa xls. Obrazovka je určená na zobrazenie a kontrolu, údaje sa na nej neupravujú.

**Akcie na obrazovke**

| Akcia | Účel |
|---|---|
| Otvoriť Úpravu faktúry | Prechod na obrazovku, kde sa údaje menia |
| Na odoslanie | Dostupné, ak faktúra nevyžaduje úpravy |
| Odložiť | Dostupné, ak faktúra nevyžaduje úpravy; vyžaduje poznámku |
| Stornovať | Dostupné, ak faktúra nevyžaduje úpravy; vyžaduje poznámku |
| Uložiť obraz faktúry | Stiahnutie PDF; povinné v statusoch 2 a 3 |
| Uložiť prílohy | Stiahnutie príloh na interné úložisko |

#### Obrazovka 2: Úprava faktúry

**Popis:** Samostatná obrazovka určená na úpravu editovateľných polí, oddelená od detailu. Editovateľné polia sú vizuálne odlíšené (podsvietené). Obraz faktúry je zobrazený súčasne s údajmi, aby spracovateľ mohol podľa neho dopĺňať údaje ako spôsob úhrady a popis. Pri prechode tabulátorom po editovateľných poliach sa v obraze faktúry podsvieti zdroj údaja nažlto. Obrazovka obsahuje aj vyhľadanie dodávateľa našepkávaním.

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

**Vyhľadanie dodávateľa**

| Vlastnosť | Popis |
|---|---|
| Spôsob vyhľadania | Našepkávanie pri písaní - spracovateľ začne písať názov dodávateľa alebo jeho IČO |
| Obsah ponuky | Zoznam dodávateľov zodpovedajúcich zadanému výrazu, pri každom údaje, s ktorými je založený v ODS |
| Dostupnosť | Vo všetkých statusoch, v ktorých je faktúra editovateľná |
| Následná akcia systému | Prevzatie Oracle vendor ID a SAP ID, spustenie kontroly B aj C, zápis do histórie zmien |

#### Obrazovka 3: Obraz faktúry a prílohy

**Popis:** Obraz faktúry sa automaticky otvára pri vstupe do detailu a je zobrazený aj na obrazovke Úprava faktúry. Je needitovaný a zobrazuje faktúru presne v podobe, v akej prišla od dodávateľa. Zmeny vykonané počas spracovania sa doň nepremietajú. Usporiadanie polí vychádza z poradia, v akom sú uvedené v XML.

**Funkcie obrazu a príloh**

| Funkcia | Popis |
|---|---|
| Automatické otvorenie | Pri vstupe do detailu došlej faktúry |
| Súčasné zobrazenie s údajmi | Na obrazovke Úprava faktúry |
| Podsvietenie zdroja údaja | Pri prechode tabulátorom po editovateľných poliach sa zdroj podsvieti nažlto |
| Uloženie obrazu ako PDF | Povinné v statusoch 2 a 3, keďže obraz sa prikladá k požiadavke do iProc na založenie kmeňových dát. V ostatných statusoch vítané, ale nie je nutnosťou |
| Uloženie príloh | Prílohy musí byť možné stiahnuť a uložiť na interné úložisko |
| Preklopenie príloh do iProc | Prílohy sa preklápajú spolu s obrazom faktúry do iProc |
| Preklik z iProc | Obraz musí byť dostupný z aplikácie iProc; spôsob prepojenia je otvorený |

#### Polia

| Názov poľa | Validation | Mandatory | Editable | Popis | Poznámka |
|---|---|---|---|---|---|
| Poznámka spracovateľa | Povinná pri statusoch 2, 3, 5, 7 a pri reexporte zo statusu 10; pri opakovanej zmene statusu výzva na ďalšiu poznámku | Podmienene áno | Áno | Informácia o spôsobe riešenia položky | - |
| Obchodný partner (dodávateľ) | Kontrola A; možnosť vyhľadať správneho dodávateľa našepkávaním podľa názvu alebo IČO | N/A | Áno - výber z ODS | Názov dodávateľa | Dostupné vo všetkých statusoch, kde je faktúra editovateľná. Pri zmene systém prevezme Oracle vendor ID a SAP ID a spustí kontrolu B aj C (AT 1d) |
| Číslo bankového účtu | Kontrola B | (xls) | Logika výberu podľa xls | - | NEŠPECIFIKOVANÉ detailne (xls) |
| Číslo faktúry | Kontrola C - duplicita - stop | N/A | Iba v statuse 4 (fallback: aj v statuse 9) a v statuse 11; inak needitovateľné | Úprava hviezdičkou a dátumom pri oprávnenej duplicite | Systém odstraňuje medzery (AT 8d) |
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
| IČ DPH odberateľa | Kontrola A1 - porovnanie s pevnou hodnotou SK7020000944; neporovnáva sa voči ODS | N/A | Nie | IČ DPH odberateľa uvedené dodávateľom na faktúre | Pri nezhode oranžový warning; faktúra pokračuje v procese |
| Číslo dodávateľa iProc | Read-only | N/A | Nie | Oracle vendor ID z ODS | Pri zmene dodávateľa sa prevezme od nového dodávateľa |
| Číslo dodávateľa SAP | Read-only | N/A | Nie | SAP ID z ODS | Pri zmene dodávateľa sa prevezme od nového dodávateľa |
| Verified by | Read-only | N/A | Nie | TB toho, kto označil faktúru na odoslanie; posiela sa do iProc cez abbyy webservice | - |
| Číslo interné | Read-only | N/A | Nie | Voucher number z iProc po zaevidovaní | - |
| Chyba z iProc | Read-only | N/A | Nie | Chybové hlásenie pri statuse 9; prichádza v poli Processing Notes | Pôvod chyby sa rozlišuje podľa prefixu hlášky |
| Prílohy faktúry | Read-only | N/A | Nie | Prílohy k faktúre | Preklápajú sa spolu s obrazom faktúry do iProc; musí byť možné ich stiahnuť na interné úložisko |
| Výsledky kontrol | A1 IČ DPH odberateľa (voči pevnej hodnote), A dodávateľ, B bankové spojenie, C duplicita; upozornenia pri IČ DPH dodávateľa, mene alebo názve a adrese so zobrazením hodnoty z ODS | N/A | Nie | Inšpirácia ABBYY: červený flag polí; výber dodávateľa pri malom nesúlade | - |
| Dodávateľ-Zamestnanec | - | - | Rozvoj do budúcna | Dnes sa napĺňa priamo v iProc | Nie je súčasťou prvej fázy |
| Payment Reason Comment | - | - | Rozvoj do budúcna | | |

## API

- **iProc:** odoslanie dát faktúry vrátane príloh na existujúce API iProc; spätné potvrdenie s interným číslom dokladu a číslom dodávateľa; rozlíšenie chyby iProc vs eInvoice. Chybové hlásenie prichádza v poli Processing Notes; pôvod chyby sa rozlišuje podľa prefixu (`iProc - TBwsProc:` = ORACLE, `iProc - TBwsProc FC:` = ABBYY). Známe kódy: E037 duplicita, E046 neplatné číslo objednávky. Požiadavka na založenie nového dodávateľa a nového čísla účtu sa zadáva do iProc. (OTVORENY BOD: úplná špecifikácia API, zoznam chybových kódov, technický spôsob spätného prenosu, spôsob preklopenia príloh)
- **ODS:** export dodávateľov (názov, bankové spojenie, IČO, IČ DPH, adresa) ako referenčné dáta kontrol. V ODS sú iba údaje dodávateľa - kontrola A1 nad IČ DPH odberateľa preto voči ODS neprebieha. Systém je nastavený tak, že dodávateľ a číslo účtu budú v ODS najskôr nasledujúci deň po prijatí faktúry. Rozhranie musí umožniť našepkávanie pri vyhľadaní dodávateľa podľa názvu alebo IČO vrátane zobrazenia údajov, s ktorými je dodávateľ založený v ODS (AT 1d). (OTVORENY BOD: technický spôsob a frekvencia)
- **Obraz faktúry a prílohy:** dodávateľské faktúry prídu do eInvoice výhradne v XML. Polia sa v obraze usporiadajú podľa poradia, v akom sú uvedené v XML; mapovanie atribútov sa nevykonáva. Polymorfné XML atribúty vyriešia architekt a vývoj. Obraz musí byť generovateľný ako PDF na stiahnutie a dostupný na preklik z iProc. Prílohy sa preklápajú spolu s obrazom faktúry do iProc a musí byť možné stiahnuť ich na interné úložisko. (OTVORENY BOD: spôsob generovania PDF, spôsob preklopenia príloh, pôvod príloh)
- **FileNet DMS:** (OTVORENY BOD: úloha DMS pri obraze faktúry a prílohách - súvisí s otázkou o prekliku z iProc)
- **Digitálny poštár:** zdroj došlej faktúry (XML uložené tak, ako prišlo)

## Vysvetlivky pojmov

| Pojem | Význam |
|---|---|
| BU | Bankové spojenie (bankový účet dodávateľa) |
| VS | Variabilný symbol |
| Dobropis | Doklad, ktorým dodávateľ znižuje pôvodne fakturovanú sumu. V podkladoch sa preň používajú skratky DBP (pri type faktúry 381) aj DBS - obe označujú ten istý doklad |
| Ťarchopis | Doklad, ktorým dodávateľ zvyšuje pôvodne fakturovanú sumu (typ faktúry 383) |
| PDP | Prenesenie daňovej povinnosti |
| PDP0DPH | Text automaticky dopĺňaný na začiatok poľa Popis pri faktúre s prenesením daňovej povinnosti alebo type AE |
| K4O | Verifikácia editovaných polí iným používateľom, než ktorý ich editoval. V eInvoice sa neuplatňuje, kontrola správnosti prebieha v iProc |
| CPA | Označenie používané pri čísle zmluvy |
| Voucher number | Interné číslo faktúry pridelené v iProc; zapisuje sa do poľa Číslo interné |
| Forced approval hold | Stav v iProc, na ktorom sa spárujú faktúra a k nej vystavený dobropis |
| Oracle vendor ID | Číslo dodávateľa v iProc; povinný údaj pre zaevidovanie faktúry |

---

**Tri veci, na ktoré ťa upozorňujem**

**Prílohy nie sú v žiadnom podklade.** Overil som to v 10b aj v xls - ani jeden dokument prílohy došlých faktúr nerieši. Požiadavka je teda úplne nová a nevieme z nej dve podstatné veci: odkiaľ prílohy pochádzajú (prídu s XML od poštára, alebo ich pridáva spracovateľ?) a akým kanálom sa preklápajú do iProc. Doplnil som to ako otázky 9 a 10 a odporúčam to prebrať skôr než neskôr, keďže to môže znamenať rozšírenie abbyy webservicu.

**Vo verzii, ktorú si poslal, chýba časť otázok.** V Confluence zostalo 11 otázok, pôvodne ich bolo 23 - chýbajú tie biznisové, napríklad kľúč duplicity, editácia súm pri zrážkovej dani a matica oprávnení. Keďže sú stále nezodpovedané a jedna z nich je doložený rozpor medzi xls a 10b, vrátil som ich späť. Ak ich chceš z Confluence trvalo odstrániť, treba ich vyriešiť, nie zmazať - inak sa stratia.

**Sekciu Vysvetlivky pojmov som ponechal**, hoci v tvojej verzii nebola. Ak ju v Confluence nechceš, jednoducho ju nekopíruj.
