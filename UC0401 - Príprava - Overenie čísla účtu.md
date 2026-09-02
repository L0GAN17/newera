# UC0401 \- Príprava \- Overenie čísla účtu

## Obsah

## Info

**Otvorené otázky**

- **Zdroj príznaku povinného variabilného symbolu:** UC0401 má podľa dohody z review poskytnúť nadväzujúcemu UC0403 informáciu, či účet vyžaduje povinný variabilný symbol. Response AccountEnquiryEnterprise takýto príznak neobsahuje. Matúš Radušovský spomenul že ide o tabuľku TIDP014 na AS400. Treba potvrdiť, či ide o samostatný dopyt v rámci UC0401 alebo o rozšírenie AccountEnquiryEnterprise
- **Prechodné účty:** ako sú v CBS definované a či ich CashBox má akceptovať....

**Biznis zadanie**

Overenie existencie čísla účtu v CBS. Cieľom je zistiť, či zadané číslo účtu má správny tvar a formát a či sa nachádza v CBS. UC0401 overuje len formát a existenciu ako vstupnú podmienku pre nadväzujúce UC. Či je účet otvorený, klientsky, TB a neblokovaný rieši až UC0404 - Kontrola uskutočniteľnosti (vklady) / UC0504 - Kontrola uskutočniteľnosti (výbery).

Ak účet existuje, systém si z response rovno potiahne časti API, ktoré potrebujú nadväzujúce UC (aby sa dopyt nerobil duplicitne).


### Aktéri

| Aktér | Čo v tomto UC robí |
|---|---|
| **Teller** | Zadáva číslo účtu vo formáte BBAN. Je to jediná vstupná akcia v tomto UC. Pri chybe zostáva na obrazovke zadania, môže číslo opraviť a zadať znova, alebo z transakcie vystúpiť |
| **Supervízor-Teller** | V UC0401 nevykonáva žiadnu akciu. Uvedený je preto, že transakciu môže realizovať aj používateľ s rolou Supervízor-Teller, ktorý v tom prípade vystupuje v úlohe tellera. Žiadne schvaľovanie ani override sa v UC0401 nevyžaduje |
| **Systém** | Vykonáva všetky ostatné kroky. Lokálne validuje formát čísla účtu a MOD11, prevádza BBAN na IBAN, overuje dostupnosť CBS, volá rozhranie AccountEnquiryEnterprise, vyhodnocuje odpoveď, ukladá dáta pre nadväzujúce UC a zobrazuje hlášky |

## Vstupné podmienky

- Teller je prihlásený
- Pobočka je otvorená
- Pokladňa je otvorená
- Systém eviduje aktuálnu dostupnosť centrálneho bankového systému v tabuľke as400\_values (stĺpec is\_online). Ak je evidovaný ako nedostupný (is\_online = false), transakcia sa nezačína (viď AT2)



## Hlavný tok

** Hlavný scenár:**

1. Teller zadá do obrazovky číslo účtu vo formáte BBAN
2. Systém lokálne validuje formát čísla účtu, povolené sú len číselné znaky a dĺžka presne 10 znakov:
    - Ak formát sedí, UC pokračuje nasledujúcim krokom
    - Ak formát nesedí, tok pokračuje **AT1**
3. Systém vykoná kontrolu MOD11 nad celým 10-miestnym číslom účtu. Kontrola beží lokálne v CashBoxe pred dopytom do CBS:
    - Ak je číslo bezo zvyšku deliteľné 11, UC pokračuje nasledujúcim krokom
    - Ak číslo bezo zvyšku deliteľné 11 nie je, tok pokračuje **AT1**
4. Systém prevedie zadaný BBAN do tvaru IBAN pre volanie rozhrania
5. Systém overí dostupnosť centrálneho bankového systému podľa `as400_values.is_online` a vykoná dopyt cez rozhranie AccountEnquiryEnterprise 2v2 s parametrami podľa sekcie API:
    - Ak je `is_online = true` a systém odpovie, UC pokračuje nasledujúcim krokom
    - Ak je `is_online = false` alebo systém neodpovie, tok pokračuje **AT2**
6. Systém vyhodnotí odpoveď rozhrania:
    - Ak odpoveď neobsahuje chybu, účet v CBS existuje a UC pokračuje nasledujúcim krokom
    - Ak odpoveď obsahuje chybu s kódom HIS0251, účet v CBS neexistuje a tok pokračuje **AT3**
7. Systém uloží do kontextu transakcie kompletnú odpoveď rozhrania vrátane všetkých vrátených údajov o účte (zoznam polí viď sekcia API). Uložené dáta sú k dispozícii nadväzujúcim UC bez opakovaného volania CBS
   -Pripomienka k bodu 7.: Vývojar: v ktorej Cashbox tabulke/ach su ulozene hodnoty?
   Vývojar ešte spomenul:
Vidím, že v UC je spomenuté "Do databázy sa nezapisuje nič" v sekcii Výstupné podmienky.
 Aale do kontextu tranzakcie by som chapal ze proste ten flow si drzi tie udaje a dalej ich pouziva 
Ale mame pri vkladoch aj vyberoch taku tabulku vymyslenu ktora sa nazyva ze deposit 
 
a tam sa ukladaju priebezne veci z toho vkladu, napriklad ziskas udaje o klientovy ulozia sa tam, nastavy sumu ktoru chce vybrat ulozi sa tam, nastavi sa kurz ktory sa tam tiez ulozi atd aby to nebolo len take ze na konci vkladu sa na BE posiela obrovsky request a tiez aby to bolo viac safe. Mozem ti poslat tu tabulku ako vyzera
 
deposit
  id: uuid
  status: varchar(20)
  bban: bigint
  deposit_currency: char(3)
  account_currency: char(3)
  deposit_currency_rate: numeric(20,10)
  overridden_rate: numeric(20,10)
  deposit_eur_rate: numeric(20,10)
  amount: numeric(20,2)
  fee_for_deposit_info: jsonb
  coin_handling_fee_info: jsonb
  fee_policy: varchar(20)
  white_list_flag: boolean
  performed_by: varchar(50)
  created_at: timestamp
  submitted_at: timestamp
  transaction_sequence_number: integer
  payment_details: jsonb
  denominations_in: jsonb
  denominations_out: jsonb
  fee_coins_in: jsonb
  fee_coins_out: jsonb
  depositor_info: jsonb
  account_info: jsonb

a vyzera to v tom cca takto 
 
status              bban        deposit_currency  account_currency  deposit_currency_rate  overridden_rate  deposit_eur_rate  amount
CANCELLED           2820002844  USD               USD               <null>                 <null>           1.1834000000      nezobrazené
SUBMITTED           2820002844  USD               USD               <null>                 <null>           1.1834000000      nezobrazené
DRAFT               <null>      <null>            <null>            <null>                 <null>           <null>            nezobrazené
DRAFT               <null>      <null>            <null>            <null>                 <null>           <null>            nezobrazené
SUBMITTED           2627074299  EUR               EUR               <null>                 <null>           <null>            nezobrazené
DENOMINATIONS_SET   2627074299  EUR               EUR               <null>                 <null>           <null>            nezobrazené
DENOMINATIONS_SET   2627074299  EUR               EUR               <null>                 <null>           <null>            nezobrazené
DRAFT               <null>      <null>            <null>            <null>                 <null>           <null>            nezobrazené
DRAFT               <null>      <null>            <null>            <null>                 <null>           <null>            nezobrazené
DRAFT               <null>      <null>            <null>            <null>                 <null>           <null>            nezobrazené
DENOMINATIONS_SET   2627074299  EUR               EUR               <null>                 <null>           <null>            nezobrazené

 
ale ta tabulka je skor taka pomocna k tomu ako sme to vymysleli ze to robime na BE, v podstate nieco ako cache pre ten vklad aby sme v priebehu celeho vkladu mali zaznamenane tie udaje ktore sa tam postupne vyplnaju a dotahuju
 
cize lujza to bude vediet najst ale teoreticky sa tam hovori ze si to ulozi do kontextu a to by mohlo byt aj nieco take ze si to appka len docasne pamata a neuklada sa to nikde
 
Treba spomenut na analytickom meetingu.

   
8. Systém ukončí UC s úspechom a odovzdá uložené údaje nadväzujúcim UC

## Alternatívny tok

AT1 - Číslo účtu je v nesprávnom tvare

**Spúšťač:** Zadané číslo účtu nespĺňa formát (krok 2) alebo neprejde kontrolou MOD11 (krok 3).  
  
Krok v hlavnom toku: krok 2 alebo krok 3.

1\. Systém vyhodnotí, že číslo účtu je v nesprávnom tvare - obsahuje nečíselné znaky, má inú dĺžku ako 10 znakov, alebo nie je bezo zvyšku deliteľné 11.  
2. Systém nevykoná konverziu na IBAN ani dopyt do CBS.  
3. Systém zobrazí hlášku typu error E027 - "Zadane \[input-cislo\] je v nespravnom tvare."  
4. Teller ostáva na obrazovke zadania čísla účtu. Zadaná hodnota zostáva viditeľná a editovateľná.  
5. Teller zadá opravené číslo účtu a UC pokračuje krokom 2 hlavného toku, alebo z transakcie vystúpi a UC končí neúspešne.

AT2 - CBS je nedostupný

**Spúšťač:** as400\_values.is\_online = false, alebo systém nedostane odpoveď na dopyt (krok 5).

1\. Systém zistí nedostupnosť centrálneho bankového systému. Dostupnosť je priebežne evidovaná v tabuľke as400\_values (stĺpec is\_online). Nedostupnosť sa spravidla zistí ešte pred zadaním čísla účtu, tento alternatívny tok však pokrýva aj výpadok počas už rozpracovanej transakcie.

2\. Systém zobrazí hlášku typu error E026 - "CBS je nedostupny, skuste to prosim neskor."

3\. Systém nevykoná žiadnu transakciu.

4\. Teller ostáva na obrazovke zadania čísla účtu, zadaná hodnota zostáva viditeľná a editovateľná.

5\. Teller môže zopakovať pokus (tok pokračuje krokom 5 hlavného scenára, lokálne validácie už prebehli), zadať iné číslo účtu (krok 2) alebo z transakcie vystúpiť.

AT3 - Účet v CBS neexistuje

**Spúšťač:** CBS vráti v odpovedi chybu s kódom HIS0251.  
**Platí pre:** vklady aj výbery.  
**Krok v hlavnom toku:** krok 6.

1. Systém vyhodnotí, že číslo účtu je formátovo správne, ale CBS vrátil chybu s kódom **HIS0251**, [AccountEnquiryEnterprise\_BBSS\_2v2.](https://rbinternational.sharepoint.com/:w:/r/sites/TBSK-EnterpriseArchitecture/AppLib/Integration%20Systems/MW_APP/2%20Interface%20Specifications/MW_APP.AccountEnquiryEnterprise/AccountEnquiryEnterprise_2v2/AccountEnquiryEnterprise_BBSS_2v2.docx?d=w5f9e5c6326264536acaa8cc527f20f11&csf=1&web=1&e=zEXqy7)
2. Systém vyhodnotí účet ako neexistujúci a ďalšie polia odpovede nevyhodnocuje.
3. Systém zobrazí hlášku W014 - "Zadane cislo uctu neexistuje."
4. Teller ostáva na obrazovke zadania čísla účtu. Zadaná hodnota zostáva viditeľná a editovateľná.
5. Teller zadá nové číslo účtu a UC pokračuje krokom 2 hlavného toku, alebo z transakcie vystúpi a UC končí neúspešne.

## Diagram tokov


## Výstupné podmienky

**Úspech:**

- Zadané číslo účtu má správny formát (10 číslic, prešlo kontrolou MOD11) a existuje v CBS
- Systém má v kontexte transakcie uloženú kompletnú odpoveď z rozhrania AccountEnquiryEnterprise vrátane všetkých údajov o účte
- Nadväzujúce UC môžu pracovať s uloženými dátami bez opakovaného volania CBS
- Do databázy sa nezapisuje nič

**Zlyhanie:**

| Druh zlyhania | Hláška | Čo sa nezapísalo | Kde sa teller nachádza |
| --- | --- | --- | --- |
| Číslo účtu má nesprávny tvar (AT1) | E027 | Nič, dopyt do CBS neprebehol | Na obrazovke zadania čísla účtu, môže zadať znova |
| CBS je nedostupný (AT2) | E026 | Nič, transakcia sa nevykonala | Na obrazovke zadania čísla účtu, môže zopakovať pokus |
| Účet v CBS neexistuje (AT3) | W014 | Nič, odpoveď sa neuložila | Na obrazovke zadania čísla účtu, môže zadať znova |

Vo všetkých prípadoch teller ostáva na obrazovke zadania a rozhoduje sa, či zadá nové číslo alebo z transakcie vystúpi.


## Opis Obrazoviek + Validácie

Teller pracuje s jedným vstupným poľom. Jedinou aktivitou tellera v tomto UC je zadanie čísla účtu, všetko ostatné sú systémové kontroly a volania.

| **Názov** | **Dátový typ** | **Validácia** | **M** | **E** | **Popis** | **Poznámka** |
| --- | --- | --- | --- | --- | --- | --- |
| Číslo účtu (BBAN) | Text | 1. len číselné znaky, dĺžka presne 10, deliteľné 11 (MOD11) 2. ak používateľ zadá napr. len 7 znakov, políčko (rám políčka) zčerveneje), musí nad políčkom bežať kontrola. 3. pri políčku môže byť ikona , ktora bude fungovať tak, že pri podržaní kurzora na ikone sa zobrazí text: *len číselné znaky, dĺžka presne 10* | Y | Y | Číslo účtu zadané tellerom | Do CBS sa posiela v tvare IBAN |



#### Tabuľka kontrol

| **#** | **Kontrola** | **Kde beží** | **Podmienka pre pokračovanie** | **Pri nesplnení** | **Testovateľné cez** |
| --- | --- | --- | --- | --- | --- |
| 1 | Formát čísla účtu - znaky | CashBox lokálne | Len číselné znaky | E027, AT1 | Zadať alfanumerický vstup |
| 2 | Formát čísla účtu - dĺžka | CashBox lokálne | Presne 10 znakov | E027, AT1 | Zadať 9 alebo 11 číslic |
| 3 | MOD11 | CashBox lokálne, pred dopytom do CBS | Celé 10-miestne číslo je bezo zvyšku deliteľné 11 | E027, AT1 | Zadať 10-ciferné číslo, ktoré nie je deliteľné 11 |
| 4 | Dostupnosť CBS | `as400_values.is_online` | `is_online = true` | E026, AT2 | Nastaviť `is_online = false`, prípadne simulovať výpadok |
| 5 | Existencia účtu | CBS, rozhranie AccountEnquiryEnterprise 2v2 | Odpoveď neobsahuje chybu HIS0251 | W014, AT3 | Zadať platný BBAN neexistujúci v CBS |

**Poznámka pre testovanie:** kontroly 1 až 3 musia zlyhať bez volania CBS. Ide o lokálnu pre-selekciu, ktorej účelom je nezaťažovať CBS dopytmi na zjavne chybné čísla.

## API

[2.3 AccountEnquiryEnterprise 2v2 - AppLib - ConfluenceIT](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19961248/2.3+AccountEnquiryEnterprise+2v2)

#### Rozhranie

**AccountEnquiryEnterprise 2v2** (AppLib, ConfluenceIT). Podrobná špecifikácia je v podklade *2.3 AccountEnquiryEnterprise 2v2*. Chybové kódy rozhrania nie sú predmetom tejto špecifikácie, platia podľa dokumentácie rozhrania.

#### Request - parametre posielané z CashBoxu

| **Element** | **Dátový typ** | **Hodnota z CashBoxu** |
| --- | --- | --- |
| RequestAuditInfo/ChannelID | string (číselník Business Architect List) |  |
| RequestAuditInfo/AppName | string (číselník Application List) |  |
| RequestAuditInfo/WorkstationID | string |  |
| RequestAuditInfo/PostTime | dateTime | Dátum a čas odoslania requestu |
| RequestAuditInfo/UserID | string | TB číslo zamestnanca |
| RequestAuditInfo/ClientID | string | CCAID klienta |
| RequestAuditInfo/ReferenceID | string |  |
| RequestAuditInfo/SessionID | string |  |
| RequestAuditInfo/Subversion | int | 2 (podľa BBSS posledná podporovaná subverzia) |
| DataSource | string, enum RTDS / BOTH / ONLINE | ONLINE (poskytuje aktuálne údaje a niektoré údaje naviac; pri RTDS môže ísť o niekoľkominútový sklz a niektoré údaje sa neposkytujú) |
| AccountNumberList/RetailAccountNumber | - | Neposiela sa z CashBoxu |
| AccountNumberList/IBANAccountNumber | string | IBAN tvar účtu, prevedený z BBAN (viď krok 4 hlavného toku) |

#### Vyhodnotenie odpovede

| **Výsledok** | **Kritérium** | **Ďalší postup** |
| --- | --- | --- |
| Účet neexistuje | Odpoveď obsahuje chybu s kódom **HIS0251**, textový popis "Účet neexistuje", zdrojový systém AS400 | AT3, ďalšie polia odpovede sa nevyhodnocujú |
| Účet existuje | Odpoveď neobsahuje chybu | Krok 7 hlavného toku, uloží sa kompletná odpoveď |

`AccountStatus` sa v UC0401 nevyhodnocuje, je to predmetom UC0404 - Príprava - Kontrola uskutočniteľnosti alebo UC0504.


#### Response - údaje ukladané do kontextu transakcie

Ukladá sa kompletná odpoveď. Nasledujúce tabuľky sú úplný zoznam polí, ktoré rozhranie vracia, spolu s informáciou, ktorý nadväzujúci UC dané pole využíva.

**Identifikácia účtu**

| **Pole (AccDetail/)** | **Typ** | **Popis** | **Využíva** |
| --- | --- | --- | --- |
| RetailAccountNumber | decimal | Retailové číslo účtu | UC0403, realizačné UC |
| IBANAccountNumber | string | IBAN tvar účtu | UC0403, realizačné UC |
| CBSPresentBranch | string | Aktuálna pobočka, identifikátor v CBS (3 znaky) |  |
| Brand | string | 001 = Tatra banka, 002 = Raiffeisen | UC0404 / UC0504 |
| BrandClientID | long | CCA\_ID majiteľa účtu (podľa CIF) | UC0402 |
| CurrencyCode | string | Kód meny účtu | UC0403 (predvolená mena transakcie) |
| AccountName | string | Názov účtu | UC0403 |
| AlternativeName | string | Alternatívny názov účtu (pri sporiacom účte) |  |
| OpenDate | date | Dátum otvorenia účtu |  |
| Note | string | Poznámka |  |

**Stav a typ účtu**

| **Pole (AccDetail/)** | **Typ** | **Popis** | **Využíva** |
| --- | --- | --- | --- |
| AccountStatus | string | O = open, C = close | UC0404 / UC0504 |
| AccountSubType | string | N = nostro, L = loro, S = sporiaci, C = klientsky, NU = numerický, X = ostatné, V / V1 / V2 = vkladná knižka | UC0404 / UC0504 |
| BlockDB | string (Y/N) | Blokované debetné operácie | UC0504 (výbery) |
| BlockCR | string (Y/N) | Blokované kreditné operácie | UC0404 (vklady) |
| SepaSecCode | string | 1 = bez ochrany, 2 = podmienené inkaso, 3 = nemožno inkasovať |  |
| SpecialAccAttribute | string | BD = bytový dom, NU = notárska úschova, SVB = spoločenstvo vlastníkov, SUN = franchisista |  |
| BundleId | string | ID balíka (kód balíka) |  |
| StatementOwnerFlag | string (Y/N) | Význam neurčený v podklade |  |
| LastClosedBankingDate | date | Dátum uzatvoreného bankového dňa, len pre neuzatvorené účty |  |

**Zostatky a limity**

| **Pole (AccDetail/)** | **Typ** | **Popis** | **Využíva** |
| --- | --- | --- | --- |
| Balances/ClosedClearBal | amount | Zostatok po uzávierke, bez pohybov s doprednou valutou |  |
| Balances/ClosedLdgBal | amount | Zostatok po uzávierke, s pohybmi s doprednou valutou |  |
| Balances/ActualClearBal | amount | Aktuálny zostatok, bez doprednej valuty |  |
| Balances/ActualLdgBal | amount | Aktuálny zostatok, s doprednou valutou |  |
| Balances/DispoClearBal | amount | Disponibilný zostatok, bez doprednej valuty | UC0504 |
| Balances/DispoLdgBal | amount | Disponibilný zostatok, s doprednou valutou | UC0504 |
| OverdraftLimit | amount | Výška povoleného prečerpania | UC0504 |
| HeldAmount | amount | Výška zadržiavaných prostriedkov | UC0504 |
| MinimumBalance | amount | Minimálny zostatok na účte | UC0504 |
| OverdraftDetail/Preference | text (Y/N) | Má účet nastavenú preferenciu |  |
| OverdraftDetail/OverDraftFlag | text (Y/N) | Balíkové voliteľné prečerpanie |  |
| OverdraftDetail/PreferredLimit | amount | Preferovaná výška limitu voliteľného prečerpania |  |

**Cashpool zostatky**

| **Pole (AccDetail/CashpoolBalances/)** | **Typ** | **Popis** |
| --- | --- | --- |
| ClosedClearBal, ClosedLdgBal | amount | Zostatok po uzávierke (bez / s doprednou valutou) |
| ActualClearBal, ActualLdgBal | amount | Aktuálny zostatok (bez / s doprednou valutou) |
| DispoClearBal, DispoLdgBal | amount | Disponibilný zostatok (bez / s doprednou valutou) |

**Sporiaca schéma (pri sporiacom účte)**

| **Pole (AccDetail/SavingSchema/)** | **Typ** | **Popis** |
| --- | --- | --- |
| SavingSchemaFlag | string (Y/N) | Je aktívna sporiaca schéma |
| SavingsMinTransfer | amount | Minimálna hodnota sporiaceho prevodu |
| SavingsMaxTransfer | amount | Maximálna hodnota sporiaceho prevodu |
| PosVolume | decimal | Percento objemu POS transakcií (max. 999,99 %) |
| SavingsMinBalance | amount | Minimálny účtovný zostatok na BÚ po sporiacom prevode |
| BalanceType | string | Typ zostatku: U = účtovný, D = disponibilný |
| LastChangeDatetime | string | Dátum a čas poslednej zmeny schémy (counter pre AccountSavingSchemeAPI) |

**Časové pečiatky**

| **Pole (AccDetail/OfflineTimeStamps/)** | **Typ** | **Popis** |
| --- | --- | --- |
| LastAccountBalanceChange | dateTime | Dátum poslednej aktualizácie zostatku na bežnom účte |
| LastBalanceChange | dateTime | Dátum poslednej aktualizácie zostatku v RTDS na ľubovoľnom bežnom účte |

#### Zdrojové tabuľky lokálnej databázy CashBox

| **Tabuľka** | **Stĺpce** | **Použitie v UC0401** |
| --- | --- | --- |
| `as400_values` | `is_online` boolean, `cbd` date | Overenie dostupnosti CBS v kroku 5 a vo vstupných podmienkach, vyhodnotenie AT2. Stĺpec `cbd` obsahuje aktuálny bankový deň |

Poznámka: AS400 a CBS označujú ten istý centrálny bankový systém (potvrdil Matúš Radušovský). V texte UC sa používa pojem CBS, názvy technických objektov zostávajú nezmenené.

#### Cieľové tabuľky

UC0401 **do databázy nezapisuje**. Odpoveď rozhrania je držaná v kontexte transakcie a odovzdaná nadväzujúcim UC.

#### Nadväznosť

- **Vstup:** teller zadá číslo účtu vo formáte BBAN
- **Súbežne:** UC0402 - Príprava - Overenie klienta (rieši osobu, nie účet, volá rozhranie GateGlobal)
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 - Príprava - Kontrola uskutočniteľnosti (výbery)
- **Analogický UC:** UC0501 - Príprava - Overenie čísla účtu (výbery). Oba UC sú funkčne zhodné a treba ich udržiavať zosúladené

---

### Mapping

UC0401 nezapisuje údaje do databázy, preto samostatné mapovanie na stĺpce tabuliek nie je relevantné. Prehľad polí odpovede rozhrania a informácia o tom, ktoré nadväzujúce UC dané pole využíva, sú v sekcii API, časť Response.
