# UC0402 \- Príprava \- Overenie klienta

## Obsah


## Info

2. Systém zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) s parametrom brand = 001 (TB) a stiahne údaje o osobe. Vyhľadávanie prebehne podľa identifikátora, ktorý GATE odovzdal (CCAID alebo rodné číslo):
Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
Ak rozhranie nie je dostupné, tok pokračuje AT2

Ja tomu rozumiem, ale trošku sa obávam, že ti to vráti vývoj, alebo Lujza .. ze nemáš jednoznačne určený čo je hlavný tok 
 
3. Nemalo by byť kam uloží ? - toto je eśte otvorená otázka na analyticky (analitici, vs Vývoj, ci uz maju nejake riešenie a podobne ) 
 
4. Zase jednoznačnost HT , a popravde moc som ani neporozumel 
 
6. - Ak nema pri vklade vztah k účtu ma to vplyv na UC0431 (Vtedy plati poplatok hneď pri vklade ) 
 
7. Druh dokladu neprichádza z prekliku z GATEu ? 
 
10. trosku by som preformuloval, nieco ako mas a ze pokračuje v UC z ktorého bol UC0402 volaný 
 
AT1.  - a vyhľadá fyzickú osobu podľa zadaného rodného čísla to mi príde akoby bolo vyhladavanie mozne len cez rodné číslo. Je to naozaj tak ? aj cudzinci maju rodne cisla ? 
 
AT2 3 - Chýba hláška 
 
Legenda k spôsobu získania údaja:
GateGlobal - údaj sa dotiahne automaticky z GateGlobal (CustomerCBIDetail v6.2), prípadne zo SubRegu podľa dostupnosti - vyššie uvádzaš ze SubReg sa nepožíva ...  rozporuješ sám seba 

 
Aspoň takto zatial, nie je možnosť komentovať v conflu priamo 

**Otvorené otázky**

- **Krajina vystavenia dokladu totožnosti:****[GateGlobal.CustomerCBIDetail (v6.2)](https://rbinternational.sharepoint.com/:f:/r/sites/TBSK-EnterpriseArchitecture/AppLib/Distribution%20Channels/GateGlobal/2%20Interface%20Specifications/GateGlobal.CustomerCBIDetail?csf=1&web=1&e=OuebWc)** toto pole neobsahuje. K dispozícii sú len `CITIZENSHIP` (krajina trvalého pobytu), `birthCountry` (krajina narodenia) a `TAX_DOMICIL` (daňový domicil) - ani jedno nie je krajina vydania dokladu. Treba rozhodnúť, či sa pole vypĺňa vždy manuálne, alebo existuje iný zdroj. → Tomáš Macháček / Feri
- ~~**Kontroly zo subjektových príznakov:****[GateGlobal.CustomerCBIDetail (v6.2)](https://rbinternational.sharepoint.com/:f:/r/sites/TBSK-EnterpriseArchitecture/AppLib/Distribution%20Channels/GateGlobal/2%20Interface%20Specifications/GateGlobal.CustomerCBIDetail?csf=1&web=1&e=OuebWc)** obsahuje príznaky, ktoré CashBox dnes nevyužíva - `BLACK_LIST_FLAG`, `BLACK_LIST_BLOCK` (má byť blokovaná obsluha klienta), `DEATH_FLAG` a `DEATH_DATE` (úmrtie klienta), `EXECUTION_FLAG` (evidovaná exekúcia). Treba rozhodnúť, či sa majú v CashBoxe vyhodnocovať a s akým dôsledkom. → Feri ~~

- **Rodné číslo - **dĺžka: 9 alebo 10 mieste rč
- Doriešiť vztah k účtu

**Biznis popis**

UC slúži na overenie **osoby, ktorá je fyzicky prítomná pred tellerom** a vykonáva transakciu. Cieľom je získať kompletnú sadu identifikačných údajov o tejto osobe a vyhodnotiť, či je klientom banky a či je whitelistovaná.

Spôsob získania údajov závisí od dvoch faktorov:

1. **Či má osoba pridelené CCAID** (je klientom TB) alebo nie
2. **Ako sa používateľ dostal do CashBoxu** - preklikom z aplikácie GATE alebo priamym vstupom do CashBoxu

Z toho vyplývajú **štyri scenáre**, ktoré určujú, ktoré údaje sa dotiahnu automaticky a ktoré musí teller vyplniť manuálne (viď sekcia Opis Obrazoviek + Validácie).

Vždy sa overuje **fyzická osoba** - tá, ktorá stojí pred tellerom a realizuje transakciu.

**Princíp sťahovania dát:** UC0402 si z GateGlobal uloží **kompletne celú odpoveď**, nie len vybrané polia. Nadväzujúce UC (UC0403, UC0404 / UC0504) potom GateGlobal nevolajú znova a pracujú s uloženými dátami.

**Vzťah k UC0401:** UC0402 rieši **osobu**, UC0401 rieši **účet**. Oba UC prebiehajú samostatne, nie sekvenčne za sebou. Nadväzujúci UC0404 / UC0504 pracuje s dátami z oboch.

**Vyhodnotenie klient / neklient a whitelist nie je blokujúce.** Ak osoba nie je klientom alebo nie je whitelistovaná, tok pokračuje ďalej. Tieto príznaky sú len vstupom do vyhodnotenia limitov v UC0404 / UC0504.

**Význam whitelistu (doplnené z review 10.8.2026):** whitelist je výnimka z dokladovania pôvodu peňazí. Štandardne pri vklade nad 10 000 EUR musí klient uviesť dôvod a zdroj peňazí, pri vklade nad 50 000 EUR musí zdroj zdokladovať (napríklad zmluvou). Whitelistovaní sú klienti s pravidelne vyššími obnosmi (napríklad tržby), aby toto nemuseli dokladovať pri každom vklade.

**Numerické subjekty:** reprezentant GateGlobal **nespracúva numerické subjekty** (subjekty s príznakom `IS_NUMERIC_FLAG = '1'`). Ide o klientov privátneho bankovníctva, ktorí nechcú byť menovaní. Tí si pri vklade alebo výbere prevedú prostriedky na klientsky účet. CashBox pracuje výlučne s klientskymi účtami, takže toto obmedzenie nie je prekážkou.

**Aktéri**

- Teller
- Supervízor-Teller
- System

## Vstupné podmienky

- Teller je prihlásený
- Pobočka je otvorená
- Pokladňa je otvorená
- UC je vyvolaný len počas klientskych transakcií (vklady, výbery, rozmieňanie)
- UC môže byť vyvolaný dvoma spôsobmi:
    - **a)** preklikom z aplikácie GATE - údaje o osobe prídu v prekliku
    - **b)** priamym vstupom do CashBoxu - teller osobu vyhľadáva (viď AT1)

  

## Hlavný Tok

**Predpoklad:** UC0402 sa vyvolá s preklikom z aplikácie GATE. Ak preklik neprebehol, tok pokračuje podľa AT1.

1. Systém prevezme z prekliku GATE identifikačné údaje osoby a výsledok vyhľadania v GATE.
2. Systém zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) s parametrom brand = 001 (TB) a stiahne údaje o osobe. Vyhľadávanie prebehne podľa identifikátora, ktorý GATE odovzdal (CCAID alebo rodné číslo):
    - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
    - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
3. Systém uloží do kontextu transakcie kompletnú odpoveď rozhrania. Uložené dáta sú k dispozícii nadväzujúcim UC bez opakovaného volania GateGlobal.
4. Systém vyhodnotí, či má osoba pridelené CCAID (atribút ccaIdTb):
    - Ak je atribút vyplnený, osoba je klientom TB
    - Ak supersubjekt neobsahuje CCA subjekt, atribút nie je vyplnený a osoba nie je klientom TB
Systém výsledok uloží ako príznak pre nadväzujúce UC. Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok.
5. Systém zavolá rozhranie ProductCBIAuthorizedSubjectsService V5 s identifikáciou účtu (IBAN) a zistí zoznam osôb oprávnených k danému účtu spolu s typmi ich oprávnení:
    - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
    - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
6. Systém porovná osobu prítomnú pred tellerom so zoznamom oprávnených osôb a vyhodnotí jej vzťah k účtu. Výsledok uloží ako príznak pre nadväzujúce UC. \[OTVORENY BOD: dôsledok, ak osoba vzťah k účtu nemá, a ktoré typy oprávnení sú relevantné\]
7. Systém určí druh dokladu totožnosti podľa toho, ktoré z polí dokladov je v odpovedi vyplnené (OP\_NUMBER, PAS\_NUMBER, ID\_NUMBER, PNP\_NUMBER), a namapuje ho na príslušný kód z číselníka `id_card_type`.
8. Systém doplní ostatné údaje o osobe podľa scenára (matica v sekcii Opis obrazoviek + Validácie). Údaje, ktoré sa nedotiahnu automaticky, ostávajú prázdne na manuálne vyplnenie tellerom.
9. Systém zobrazí údaje o osobe na obrazovke. Teller ich skontroluje a v prípade potreby doplní alebo upraví, všetky polia sú editovateľné. \[OTVORENY BOD: akou akciou teller overenie uzatvára\]
10. Systém ukončí UC s úspechom a odovzdá nadväzujúcim UC overené údaje o osobe, príznak Klient / Neklient a príznak vzťahu k účtu.

## Alternatívny Tok

#### AT1 - Priamy vstup do CashBoxu (bez prekliku z GATE)

**Spúšťač:** UC0402 bol vyvolaný priamo v CashBoxe, údaje o osobe nie sú vopred k dispozícii.  
**Platí pre:** vklady, výbery aj rozmieňanie.  
**Krok v hlavnom toku:** nahrádza kroky 1 a 2.

1. Systém zobrazí tellerovi formulár na zadanie údajov o osobe.
2. Teller manuálne zadá rodné číslo osoby, ktorá je pred ním, a zvolí Vyhľadať.
3. Systém zavolá rozhranie GateGlobal.CustomerCBIFind (v6.0) s parametrom brand = 001 (TB) a vyhľadá fyzickú osobu podľa zadaného rodného čísla. Vždy sa vyhľadáva fyzická osoba, teda tá, ktorá vykonáva transakciu:
    - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
    - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
4. Systém zo získaného identifikátora (ccaIdTb, prípadne ccxId) zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) a dotiahne kompletné údaje vrátane dokladov totožnosti. Volanie Detail je nutné, pretože CustomerCBIFind nevracia atribúty z tabuľky CCD\_DOCUMENT:
    - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
    - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
5. Systém vyhodnotí výsledok a doplní údaje podľa scenára:
    - Ak má osoba CCAID, z GateGlobal sa dotiahnu priezvisko, meno, titul, dátum narodenia, CCAID, CIF a PID. Rodné číslo a údaje o doklade totožnosti musí teller vyplniť manuálne
    - Ak osoba CCAID nemá, teller vyplní manuálne všetky údaje o osobe. Polia CCAID, CIF a PID sa nepoužívajú
6. Tok pokračuje krokom 3 hlavného toku.

#### AT2 - Rozhranie nie je dostupné

**Spúšťač:** systém nedostane odpoveď z rozhrania GateGlobal alebo ProductCBIAuthorizedSubjectsService.  
**Platí pre:** vklady, výbery aj rozmieňanie.  
**Krok v hlavnom toku:** krok 2 alebo krok 5 hlavného toku, krok 3 alebo krok 4 v AT1.

1. Systém zistí nedostupnosť rozhrania. \[OTVORENY BOD: mechanizmus detekcie\]
2. GateGlobal je jediný zdroj údajov o osobe. SubReg sa pre CashBox nepoužíva, CBS neobsahuje doklady totožnosti. Náhradný zdroj údajov neexistuje.
3. Systém zobrazí tellerovi chybovú hlášku o nedostupnosti služby. \[OTVORENY BOD: konkrétna hláška, v katalógu zatiaľ nie je\]
4. Osobu nie je možné overiť, transakcia nemôže pokračovať a UC končí neúspešne.

## Diagram tokov

## Výstupné podmienky

**Úspech:**

- Osoba fyzicky prítomná pred tellerom je overená a jej identifikačné údaje sú kompletné
- Systém má v kontexte transakcie uloženú kompletnú odpoveď z GateGlobal
- Systém má uložené príznaky, či je osoba klientom banky 
- Nadväzujúce UC môžu pracovať s uloženými dátami bez opakovaného volania GateGlobal

**Zlyhanie:**

- GateGlobal je nedostupný - osobu nie je možné overiť, transakcia nemôže pokračovať

## Opis Obrazoviek + Validácie

**Legenda k spôsobu získania údaja:**

- **GateGlobal** - údaj sa dotiahne automaticky z GateGlobal (CustomerCBIDetail v6.2), prípadne zo SubRegu podľa dostupnosti
- **Vyhľadanie v Gate** - údaj sa prenesie z výsledku vyhľadania v aplikácii GATE
- **Manuálne** - údaj musí vyplniť teller ručne, nedopĺňa sa automaticky
- **-** - pole sa pre daný scenár nepoužíva

**M** = Mandatory (Y = povinné, N = nepovinné), **E** = Editable (Y = editovateľné, N = read-only)

#### Sekcia osoba (fyzická osoba prítomná pred tellerom)

  

| Názov | Dátový typ | Validácia | M | E | Popis | Osoba s CCAID - preklik z Gate | Osoba s CCAID - priamo v CashBoxe | Osoba bez CCAID - preklik z Gate | Osoba bez CCAID - priamo v CashBoxe |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Priezvisko | Text | - | Y | Y | Priezvisko osoby | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Meno | Text | - | Y | Y | Meno osoby | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Titul | Text | - | N | Y | Titul pred menom | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Rodné číslo | Text | 9 až 10 miest | N | Y | Vyhľadávací identifikátor fyzickej osoby | GateGlobal | Manuálne | Vyhľadanie v Gate | Manuálne |
| Dátum narodenia | Date (DD/MM/YYYY) | - | Y | Y | Dátum narodenia | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Druh dokladu | Dropdown | hodnoty z číselníka `id_card_type` | Y | Y | Druh dokladu totožnosti (občiansky preukaz, pas, povolenie na pobyt a ďalšie podľa číselníka) | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |
| Číslo dokladu | Text | - | Y | Y | Číslo dokladu totožnosti | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |
| Krajina vystavenia dokladu | Dropdown | skratka krajiny (napr. SK) | Y | Y | Krajina vystavenia dokladu | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |

**Pozn. k druhu dokladu:** hodnoty rozbaľovacieho zoznamu sa načítavajú z číselníka `id_card_type` v DB CashBox (stĺpce `code` a `description`), nie sú v aplikácii pevne zadané. Systém určí druh dokladu podľa toho, ktoré z polí `OP_NUMBER`, `PAS_NUMBER`, `ID_NUMBER` alebo `PNP_NUMBER` je v odpovedi GateGlobal vyplnené, a namapuje ho na príslušný kód z číselníka.

#### Sekcia Iný subjekt (PO / FOP)

\[OTVORENY BOD: patrí táto sekcia do UC0402? Matica od BA ju nepokrýva - Feri\]

| Názov | Dátový typ | Validácia | M | E | Popis |
| --- | --- | --- | --- | --- | --- |
| CCAID | Text | - | N | Y | CCAID právnickej osoby alebo FOP |
| CIF | Text | - | N | Y | CIF právnickej osoby alebo FOP |
| PID | Text | - | N | Y | PID právnickej osoby alebo FOP |

#### Zhrnutie pravidiel

- Pri osobách **s CCAID** sa základné osobné údaje (meno, priezvisko, titul, dátum narodenia) získavajú automaticky z GateGlobal.
- Údaje o **doklade totožnosti** sa pri prekliku z Gate preberajú z výsledku vyhľadania v Gate. Pri priamom vstupe do CashBoxu ich musí teller vyplniť manuálne.
- Pri osobách **bez CCAID** sa údaje pri prekliku z Gate preberajú z vyhľadania v Gate. Pri priamom vstupe do CashBoxu sa všetky údaje vypĺňajú manuálne.

  

## API

#### Rozhrania

| Rozhranie | Verzia | Účel | Použitie v UC0402 |
| --- | --- | --- | --- |
| **GateGlobal.CustomerCBIDetail** | v6.2 | Detail klienta (reprezentant GG\_SUBJ\_DATA) | Primárny zdroj údajov o osobe. Jediný zdroj údajov o dokladoch totožnosti |
| **GateGlobal.CustomerCBIFind** | v6.0 | Vyhľadanie klienta podľa identifikátorov (RČ, IČO, PID, telefón, email) | Použije sa pri manuálnom vyhľadaní v AT1. **Nevracia doklady totožnosti** |
| **GateGlobal.ProductCBIAuthorizedSubjects** | v4 | Oprávnenia na produkty | \[OTVORENY BOD: možná väzba na tému oprávnených osôb\] |
| **SubReg (Subjektový register)** | - | Záložný zdroj pri nedostupnosti GateGlobal | \[OTVORENY BOD: konkrétna služba. SubjectApi\_1\_9 sa nepoužíva\] |

**CBS sa ako zdroj údajov o osobe nepoužíva** - neobsahuje doklady totožnosti (potvrdil Feri).

#### Číselníky z DB CashBox

| Tabuľka | Stĺpce | Použitie v UC0402 |
| --- | --- | --- |
| `id_card_type` | `code` varchar(20), `description` varchar(20) | Číselník druhov dokladov totožnosti pre pole Druh dokladu (občiansky preukaz, pas, povolenie na pobyt a ďalšie) |

#### Parametre requestu

| Parameter | Hodnota z CashBoxu | Poznámka |
| --- | --- | --- |
| **brand** | **001** (TB) | Reprezentant sa počíta zo subjektových obrazov daného brandu. 001 = TB, 002 = RB. Ak sa brand nepošle, vráti sa super-supersubjekt naprieč brandmi |
| Vyhľadávací identifikátor | CCAID / rodné číslo / IČO | Podľa scenára |

#### Vyhľadávanie

| Identifikátor | Atribút | Poznámka |
| --- | --- | --- |
| CCAID | `ccaIdTb` (TB), `ccaIdRb` (SB) | `ccaIdTb` je CCA subjekt s najnižším ID v rámci supersubjektu. **Ak supersubjekt neobsahuje CCA subjekt, atribút nie je vyplnený** - to zodpovedá scenáru osoby bez CCAID |
| Rodné číslo | `personalId` | Vždy sa vyhľadáva fyzická osoba pred tellerom |
| IČO | `ico` | Vyhľadávanie na 8 miest, kratšie sa doplní na 8 miest. Pre sekciu Iný subjekt pri PO |

**Upozornenie k identifikátorom:** podľa podkladu sa identifikátor supersubjektu (`ccdIdTb`) a super-supersubjektu (`ccxId`) môžu v čase meniť spárovaním alebo rozpárovaním subjektov. **Nie je vhodné sa na ne priamo odkazovať.** CashBox pracuje s `ccaIdTb`.

#### Obmedzenia reprezentanta

- **Numerické subjekty** (`IS_NUMERIC_FLAG = '1'`) reprezentant nespracúva. Nie je to pre CashBox prekážka, pracuje sa len s klientskymi účtami.
- Zdrojom hodnôt je **aktívny CCA obraz supersubjektu** (`ACTIVE_FLAG = 1`). Ak má supersubjekt viac aktívnych CCA obrazov, berie sa ten s najmenším ID.
- Algoritmus výpočtu reprezentanta zohľadňuje **GDPR stav** obrazov supersubjektu - obrazy s nižšou prioritou sú z výpočtu vylúčené.

#### Sťahovanie dát

UC0402 sťahuje **kompletne celú odpoveď** rozhrania a ukladá ju do kontextu transakcie. Nadväzujúce UC z nej čerpajú bez opakovaného volania.

Chybové kódy rozhrania nie sú predmetom tejto špecifikácie, platia podľa dokumentácie rozhrania.

#### Nadväznosť

- **Vstup:** preklik z GATE alebo priamy vstup v CashBoxe
- **Súbežne:** UC0401 - Príprava - Overenie čísla účtu (rieši účet, volá AccountEnquiryEnterprise)
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 (vklady) alebo UC0504 (výbery)
- **Analogický UC:** UC0502 - Príprava - Overenie klienta (výbery), zatiaľ nevytvorený
- **Možný prekryv:** UC0407 - Príprava - Overenie klienta - manuálne (nedostupný GATE)

---

### Mapping

Zdroj: podklad *GG Dátový reprezentant FO PO, verzia 25.5* (rozhranie CustomerCBIDetail v6.2, reprezentant GG\_SUBJ\_DATA).

#### Osobné údaje - tabuľka CCD\_PERSON (GS\_PERSON)

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
| --- | --- | --- | --- |
| Meno | `FIRSTNAME` / `firstName` | FIRSTNAME | Ak je PERSON\_NAME prázdne, meno sa vyskladá ako SURNAME + medzera + FIRSTNAME |
| Priezvisko | `SURNAME` / `surname` | SURNAME |  |
| Titul | `TITLE_INF` / `title` | TITLE\_INF | Titul pred menom. Reprezentant eviduje aj `titleBehind` (titul za menom) - v CashBoxe sa zatiaľ nepoužíva |
| Rodné číslo | `PERSONAL_ID` / `personalId` | PERSONAL\_ID | Hodnota z aktívneho CCA obrazu subjektu z GATE TB |
| Dátum narodenia | `BIRTH_DATE` / `birthDate` | BIRTH\_DATE |  |

#### Doklady totožnosti - tabuľka CCD\_DOCUMENT (GS\_DOCUMENT)

Reprezentant nemá jedno pole pre druh dokladu. Má **samostatné pole pre každý typ dokladu** a druh sa určí podľa toho, ktoré je vyplnené. Zistený druh sa namapuje na kód z číselníka `id_card_type`.

| Druh dokladu | Atribút reprezentanta | Zdrojový atribút | Pravidlo výberu |
| --- | --- | --- | --- |
| Občiansky preukaz | `OP_NUMBER` | DOC\_NUMBER | Prvý vyplnený podľa stavu v poradí PLA, NED, NEP, POD, EXP, AUK, kde ACTIVE\_FLAG = '1', len doklad viazaný na CCA subjekt. Ak supersubjekt nemá aktívny doklad daného typu, pole je prázdne |
| Pas | `PAS_NUMBER` | DOC\_NUMBER | Rovnaké pravidlo, typ dokladu PAS |
| ID karta | `ID_NUMBER` | DOC\_NUMBER | Rovnaké pravidlo, typ dokladu ID a všetky ID\_% |
| Povolenie na pobyt | `PNP_NUMBER` | DOC\_NUMBER | Rovnaké pravidlo, typ dokladu PNP |

Poznámka: stav dokladu NULL sa vyhodnocuje ako Nedefinovaný (NED).

**Krajina vystavenia dokladu:** reprezentant toto pole neobsahuje. \[OTVORENY BOD: zdroj alebo manuálne vypĺňanie\]

#### Identifikátory - tabuľka CCD\_SUBJECT (GS\_SUBJECT) a CCD\_IAAP

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
| --- | --- | --- | --- |
| CCAID | `CCAID_TB` / `ccaIdTb` | ID | Identifikátor CCA subjektu v TB. Ak supersubjekt neobsahuje CCA subjekt, nie je vyplnený |
| PID | `PID_TB` / `pidTb` | P\_NUMBER | PID v stave aktívny (STATUS = 'PI0'). Pri viacerých sa vráti text MANY\_PIDS |
| CIF | `MAIN_CIF` / `mainCif` | MAIN\_CIF | Pri viacerých rôznych MAIN\_CIF subjektoch sa vráti text MANY\_MAINCIFS |

####   
Ďalšie dostupné atribúty (CashBox zatiaľ nevyužíva)

| Atribút | Popis | Možné využitie |
| --- | --- | --- |
| `sType` / `C_TYPE` | Typ subjektu: P = fyzická osoba, C = právnická osoba, E = zamestnanec TB, B = banka, A = anonym, I = pobočky a bankomaty, R, Q | Rozlíšenie FO/PO, identifikácia zamestnanca TB |
| `INDUSTRY_CODE` | Industry Code, číselník CCD\_INDUSTRY | Zdroj pre rozlíšenie živnostníka v UC0416 |
| `BLACK_LIST_FLAG` | Subjekt je na black liste (1 = áno) | \[OTVORENY BOD\] |
| `BLACK_LIST_BLOCK` | Má byť blokovaná obsluha klienta (1 = áno) | \[OTVORENY BOD\] |
| `DEATH_FLAG`, `DEATH_DATE` | Úmrtie klienta | \[OTVORENY BOD\] |
| `EXECUTION_FLAG` | Evidovaná exekúcia | \[OTVORENY BOD\] |
| `CITIZENSHIP` | Krajina trvalého pobytu (FO) / sídla (PO) |  |
| `tbConsistencyFlag` | Príznak konzistentnosti klienta |  |

  

##
