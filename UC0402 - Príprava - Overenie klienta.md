Prešiel som všetky Matúšove pripomienky. Sedem sa dá zapracovať priamo, jedna nie - píšem o nej na konci.

---

## 1. Jednoznačnosť hlavného toku (pripomienky 2 a 6)

Matúš má pravdu, že nie je jednoznačne určené, čo je hlavný tok. Riešim to dvomi vecami: doplnením vetvenia hneď na začiatok a explicitným návratom na konci.

**Hlavný tok - nahradiť predpoklad a krok 1**

```
**Vstup do UC.** UC0402 má dva vstupné body, ktoré sa navzájom vylučujú:

| Vstupný bod | Kedy nastáva | Kde je popísaný |
|---|---|---|
| Preklik z aplikácie GATE | Teller prišiel do CashBoxu preklikom z GATE, údaje o osobe prídu v prekliku | Hlavný tok |
| Priamy vstup do CashBoxu | Teller spustil transakciu priamo v CashBoxe bez prekliku, údaje o osobe nie sú k dispozícii | AT1 |

1. Systém vyhodnotí, akým spôsobom bol UC0402 vyvolaný:
   - Ak bol vyvolaný preklikom z aplikácie GATE, systém prevezme z prekliku identifikačné údaje osoby a výsledok vyhľadania v GATE. UC pokračuje nasledujúcim krokom
   - Ak bol vyvolaný priamym vstupom do CashBoxu, tok pokračuje **AT1**
```

**Hlavný tok - nahradiť krok 10**

```
10. Systém ukončí UC s úspechom a odovzdá nadväzujúcim UC overené údaje o osobe, príznak Klient / Neklient a príznak vzťahu k účtu. Tok pokračuje v UC, z ktorého bol UC0402 vyvolaný, teda v príslušnom UC vkladu, výberu alebo rozmieňania.
```

---

## 2. Kam sa ukladá odpoveď (pripomienka 3)

**Hlavný tok - nahradiť krok 3**

```
3. Systém uloží kompletnú odpoveď rozhrania tak, aby bola k dispozícii nadväzujúcim UC bez opakovaného volania GateGlobal. [OTVORENY BOD: či sa údaje zapisujú do pomocnej tabuľky deposit, alebo sa držia len v pamäti aplikácie počas rozpracovanej transakcie. Rovnaká otázka je otvorená v UC0401 - Príprava - Overenie čísla účtu, viď sekcia Mapping]
```

**Otvorené otázky - doplniť bod**

```
- **Kde sa ukladajú údaje počas rozpracovanej transakcie.** Vývojár k UC0401 uviedol, že pre vklady aj výbery existuje pomocná tabuľka deposit, ktorá slúži ako cache pre priebežné údaje transakcie. Zároveň pripustil, že môže ísť aj o dočasné držanie údajov v pamäti aplikácie bez zápisu do databázy. Tabuľka deposit v aktuálnom dátovom modeli CashBox nie je. Rozhodnutie sa týka UC0401, UC0402 aj UC0403 spoločne. Odpoveď od vývoja, téma na analytický meeting. Blokuje dokončenie UC. Dopad: mení sa krok 3 hlavného toku a sekcia Mapping.
```

---

## 3. Vzťah k účtu (pripomienka 4)

Matúš nerozumel kroku 6 a upozornil na väzbu na UC0431. Prepisujem, aby bolo jasné, čo sa porovnáva a načo to slúži.

**Hlavný tok - nahradiť kroky 5 a 6**

```
5. Systém zavolá rozhranie ProductCBIAuthorizedSubjectsService V5 s identifikáciou účtu, teda s číslom účtu vo formáte IBAN prevzatým z UC0401 - Príprava - Overenie čísla účtu. Rozhranie vráti zoznam osôb, ktoré majú k danému účtu oprávnenie, spolu s typmi ich oprávnení:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
6. Systém overí, či sa osoba prítomná pred tellerom nachádza v zozname oprávnených osôb k účtu:
   - Ak sa v zozname nachádza, systém uloží príznak, že osoba má vzťah k účtu
   - Ak sa v zozname nenachádza, systém uloží príznak, že osoba vzťah k účtu nemá
   
   Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok. Príznak využívajú nadväzujúce UC, najmä UC0431 - Príprava - Poplatok za vklad, kde absencia vzťahu k účtu znamená, že poplatok sa platí hneď pri vklade. [OTVORENY BOD: ktoré typy oprávnení sa považujú za vzťah k účtu]
```

**Biznis popis - doplniť odsek**

```
**Vzťah osoby k účtu.** Systém zisťuje, či osoba prítomná pred tellerom má oprávnenie k účtu, s ktorým sa transakcia realizuje. Vyhodnotenie nie je blokujúce a slúži ako vstup pre nadväzujúce UC. Pri vkladoch má vplyv na UC0431 - Príprava - Poplatok za vklad, kde platí, že ak osoba vzťah k účtu nemá, poplatok sa platí hneď pri vklade.
```

---

## 4. Druh dokladu z prekliku (pripomienka 5)

Matúšova otázka je namieste. V matici v Opise obrazoviek je pri prekliku z GATE uvedené *"Vyhľadanie v Gate"*, ale krok 7 hovorí, že systém druh dokladu určí z GateGlobal. To si protirečí.

**Hlavný tok - nahradiť krok 7**

```
7. Systém určí druh dokladu totožnosti:
   - Ak bol UC0402 vyvolaný preklikom z aplikácie GATE, druh dokladu sa prevezme z výsledku vyhľadania v GATE
   - Ak bol vyvolaný priamym vstupom do CashBoxu a osoba má CCAID, systém určí druh dokladu podľa toho, ktoré z polí OP_NUMBER, PAS_NUMBER, ID_NUMBER alebo PNP_NUMBER je v odpovedi GateGlobal vyplnené
   - Ak bol vyvolaný priamym vstupom a osoba CCAID nemá, druh dokladu vyplní teller manuálne
   
   Zistený druh systém namapuje na príslušný kód z číselníka `id_card_type`.
```

**Otvorené otázky - doplniť bod**

```
- **Druh dokladu pri prekliku z GATE.** Matica v sekcii Opis obrazoviek uvádza, že pri prekliku z GATE sa druh dokladu prenáša z výsledku vyhľadania v GATE. Treba potvrdiť, že GATE tento údaj naozaj odovzdáva, a ak áno, v akom formáte, aby sa dal namapovať na číselník id_card_type. Odpoveď od Feriho alebo Peťa Magyara. Blokuje dokončenie UC. Dopad: mení sa krok 7 hlavného toku.
```

---

## 5. Vyhľadávanie len podľa rodného čísla (pripomienka k AT1)

Matúš má pravdu, že to znie obmedzujúco, a otázka o cudzincoch je legitímna.

**AT1 - nahradiť krok 2 a 3**

```
2. Teller manuálne zadá identifikátor osoby, ktorá je pred ním, a zvolí Vyhľadať.
3. Systém zavolá rozhranie GateGlobal.CustomerCBIFind (v6.0) s parametrom brand = 001 (TB) a vyhľadá fyzickú osobu podľa zadaného identifikátora. Vždy sa vyhľadáva fyzická osoba, teda tá, ktorá vykonáva transakciu:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**

[OTVORENY BOD: podľa ktorých identifikátorov môže teller vyhľadávať. Rozhranie GateGlobal.CustomerCBIFind podporuje rodné číslo, IČO, PID, telefón a email. Doteraz sa v UC uvádzalo len rodné číslo, čo nepokrýva cudzincov bez slovenského rodného čísla]
```

**Otvorené otázky - doplniť bod**

```
- **Podľa čoho teller vyhľadáva osobu pri priamom vstupe.** V UC bolo doteraz uvedené vyhľadávanie výlučne podľa rodného čísla. Rozhranie GateGlobal.CustomerCBIFind podporuje aj IČO, PID, telefón a email. Rodné číslo zároveň nepokrýva cudzincov, ktorí slovenské rodné číslo nemajú. Treba určiť, podľa ktorých identifikátorov môže teller vyhľadávať a ako sa postupuje pri cudzincoch. Odpoveď od Feriho. Blokuje dokončenie UC. Dopad: mení sa AT1 a Opis obrazoviek.
```

---

## 6. Chýbajúca hláška v AT2 (pripomienka k AT2)

**Otvorené otázky - doplniť bod**

```
- **Hláška pri nedostupnosti rozhrania.** V AT2 sa má zobraziť chybová hláška o nedostupnosti služby, ale v schválenom katalógu hlášok taká pre GateGlobal ani pre ProductCBIAuthorizedSubjectsService nie je. Pre CBS existuje E026 "CBS je nedostupny, skuste to prosim neskor", analogická hláška pre GateGlobal chýba. Treba navrhnúť textáciu, dať ju schváliť a prideliť kód. Odpoveď od Feriho. Blokuje dokončenie UC. Dopad: mení sa AT2.
```

---

## 7. Rozpor so SubRegom v legende

**Opis obrazoviek - nahradiť prvú odrážku legendy**

```
- **GateGlobal** - údaj sa dotiahne automaticky z rozhrania GateGlobal.CustomerCBIDetail (v6.2)
```

---

## Jedna vec, ktorú takto zapracovať neviem

Matúš pri pripomienke 4 napísal: *"Ak nema pri vklade vztah k účtu ma to vplyv na UC0431 (Vtedy plati poplatok hneď pri vklade)"*.

Zapracoval som to ako **väzbu na UC0431**, teda že príznak sa odovzdáva a UC0431 ho vyhodnocuje. Neviem ale overiť, či to tak naozaj je - v UC0431 sme spolu nepracovali a nemám k nemu podklad. Je možné, že tam táto logika ešte nie je popísaná.

Odporúčam si to overiť, prípadne priložiť k otázke pre Feriho:

```
Matus, k tomu vztahu k uctu - napisal si ze ked osoba nema vztah k uctu tak plati poplatok hned pri vklade a ma to vplyv na UC0431. Zapracoval som to tak ze UC0402 ten priznak len ulozi a vyhodnocuje ho UC0431. Je to v UC0431 uz popisane, alebo to tam treba doplnit? Ja som s tym UC nepracoval.
```
















Overil som to a **Matúš má pravdu** - moja obava bola zbytočná. UC0431 tú logiku už popísanú má a na UC0402 sa výslovne odvoláva.

## Ako to v UC0431 funguje

Krok 3 hovorí: *"Systém overí na základe informácií získaných v UC0402 vzťah klienta k účtu"*. Vetvenie je takéto:

| Vzťah k účtu | Čo sa stane |
|---|---|
| Osoba vzťah **nemá** (hlavný tok) | CashBox nastaví Fee_ID, zavolá FeeBe a poplatok sa platí hneď pri vklade. Ide o vklad treťou osobou |
| Osoba vzťah **má** (alternatíva 3.a.I) | Poplatok CashBox nevypočítava, zaúčtuje sa v procese kapitalizácie externým systémom |

Formulácia, ktorú som do UC0402 dal, teda sedí.

## Čo to navyše uzatvára

UC0431 definuje aj **konkrétne typy oprávnení**: *"Vzťah k účtu (M, K, V, D): Majiteľ, Konateľ, Vkladateľ, Disponent"*. Tým sa dá uzavrieť otvorený bod, ktorý sme v UC0402 mali pri kroku 6.

---

## Sekcie na prekopírovanie do UC0402

### Hlavný tok - nahradiť krok 6

```
6. Systém overí, či má osoba prítomná pred tellerom k účtu niektoré z oprávnení Majiteľ (M), Konateľ (K), Vkladateľ (V) alebo Disponent (D):
   - Ak má niektoré z týchto oprávnení, systém uloží príznak, že osoba má vzťah k účtu
   - Ak nemá žiadne z nich, systém uloží príznak, že osoba vzťah k účtu nemá
   
   Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok. Príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky.
```

### Biznis popis - nahradiť odsek o vzťahu k účtu

```
**Vzťah osoby k účtu.** Systém zisťuje, či osoba prítomná pred tellerom má k účtu niektoré z oprávnení Majiteľ (M), Konateľ (K), Vkladateľ (V) alebo Disponent (D). Vyhodnotenie nie je blokujúce a slúži ako vstup pre nadväzujúce UC.

Pri vkladoch príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky:

| Vzťah k účtu | Dôsledok pre poplatok za vklad |
|---|---|
| Osoba vzťah k účtu nemá | Ide o vklad treťou osobou. Poplatok vypočítava CashBox volaním FeeBe a platí sa hneď pri vklade |
| Osoba vzťah k účtu má | Poplatok CashBox nevypočítava, zaúčtuje sa v procese kapitalizácie externým systémom |
```

### Výstupné podmienky - doplniť do časti Úspech

```
- Systém má uložený príznak vzťahu osoby k účtu, teda či má niektoré z oprávnení M, K, V alebo D
- Systém má uložený typ subjektu, teda fyzická alebo právnická osoba, ktorý využíva UC0431 - Poplatok za vklad - Stanovenie výšky pri určení sadzby poplatku
```

### Mapping - presunúť sType z tabuľky nevyužívaných atribútov

Riadok `sType / C_TYPE` už nepatrí medzi nevyužívané. Nahradiť ho v tabuľke využívaných:

```
| Typ subjektu | `sType` / `C_TYPE` | C_TYPE | P = fyzická osoba, C = právnická osoba, E = zamestnanec TB, B = banka, A = anonym, I = pobočky a bankomaty, R, Q. Využíva UC0431 - Poplatok za vklad - Stanovenie výšky na rozlíšenie FO a PO pri určení sadzby poplatku (OWNER_TYPE) |
```

---

## Ale otvorilo sa niečo iné

**UC0431 si v otázke, kto volá ProductCBIAuthorizedSubjectsService, protirečí sám sebe.**

| Miesto v UC0431 | Čo hovorí |
|---|---|
| Hlavný tok, krok 3 | *"Systém overí **na základe informácií získaných v UC0402** vzťah klienta k účtu"* - teda volá UC0402 |
| Vstupné podmienky | *"Je dostupná služba ProductCBIAuthorizedSubjectsService"* |
| Výstupné podmienky | *"Bol overený vzťah klienta k účtu **prostredníctvom služby** Gate_Global ProductCBIAuthorizedSubjectsService"* - teda volá UC0431 |

Ak rozhranie volá UC0402 aj UC0431, ide o duplicitné volanie, čomu sme sa mali princípom z review vyhnúť.

### Otvorené otázky - doplniť bod

```
- **Kto volá ProductCBIAuthorizedSubjectsService.** UC0402 má podľa tejto verzie rozhranie volať a odovzdať príznak vzťahu k účtu nadväzujúcim UC. UC0431 - Poplatok za vklad - Stanovenie výšky si však protirečí: v hlavnom toku uvádza, že vzťah overuje na základe informácií získaných v UC0402, ale vo výstupných podmienkach uvádza, že vzťah bol overený prostredníctvom služby ProductCBIAuthorizedSubjectsService, a vo vstupných podmienkach vyžaduje jej dostupnosť. Treba rozhodnúť, či rozhranie volá UC0402 raz pre celú transakciu, alebo si ho volá UC0431 samostatne. Odpoveď od Matúša Radušovského a autora UC0431. Blokuje dokončenie UC. Dopad: prípadné odstránenie krokov 5 a 6 z hlavného toku UC0402.
```

---

## Dve poznámky k UC0431 samotnému

Nie sú to veci UC0402, ale keď som ho čítal, všimol som si ich.

**Rozpor v identifikátore poplatku.** Hlavný tok aj tabuľka poplatkov uvádzajú `Fee_ID = 191`, ale tabuľka FeeCalcRequest v sekcii API uvádza `feeID = 210`. Jedno z toho bude nesprávne.

**Neúplná odpoveď na otázku o vzťahu.** UC0431 pracuje s hodnotami M, K, V, D, ale nie je uvedené, ako sa tieto štyri hodnoty mapujú na typy oprávnení, ktoré vracia ProductCBIAuthorizedSubjectsService. Tá služba vracia oprávnenia ako MAJ1_PV, CIFM_PV, ZKZ1_PV a podobne. Prepojenie medzi nimi zatiaľ nikde nie je.

Návrh otázky:

```
Matus, dakujem, UC0431 to naozaj popisane ma a odvolava sa na UC0402, takze to sedi. Ale vyplynuli mi z toho dve veci:

1. UC0431 si v jednom mieste protireci - v hlavnom toku pise ze vztah overi na zaklade informacii z UC0402, ale vo vystupnych podmienkach ze ho overil cez ProductCBIAuthorizedSubjectsService. Cize kto tu sluzbu vlastne vola? Ja som ju dal do UC0402 aby sa nevolala dvakrat, ale chcem to mat potvrdene.

2. UC0431 pracuje s hodnotami M, K, V, D (majitel, konatel, vkladatel, disponent). Tá sluzba ale vracia opravnenia ako MAJ1_PV, CIFM_PV, ZKZ1_PV a podobne. Kde je mapovanie medzi tym? Bez toho nevieme co presne z odpovede citat.
```
