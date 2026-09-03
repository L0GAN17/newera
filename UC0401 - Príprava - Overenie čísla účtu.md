Vytvoril som UC0407. Mockup ale odhalil vec, ktorá mení pôvodnú predstavu z AT1 - píšem o tom hneď za UC.

---

# UC0407 - Príprava - Overenie klienta - manuálne

## Obsah

- Obsah
- Info
  - Otvorené otázky
  - Biznis zadanie
  - Aktéri
- Vstupné podmienky
- Hlavný tok
- Alternatívny tok
- Diagram tokov
- Výstupné podmienky
- Opis obrazoviek + Validácie
- API
- Mapping

---

## Info

### Otvorené otázky

**1. Vyhladava sa v UC0407 vobec? → Feri**

```
Toto je najdolezitejsia otazka a od nej zavisi cely tok.

V mockupe modalu Informacie o vkladatelovi nie je ziadne tlacidlo Vyhladat.
Vyzera to tak ze teller proste vyplni vsetko rucne a potvrdi.

Ty si ale pisal ze "len pri klientovi vieme ostatne polia doplnit ako je rodne
cislo, tituly atd". To znamena ze pri klientovi sa nieco dotahuje. Lenze
z mockupu nevidim co to spusti.

Cize ako to ma fungovat?

a) Teller vyplni vsetko rucne, nic sa nikam nevola. Vtedy ale nevieme ci je
   osoba klientom a nevieme ani jej vztah k uctu, co potrebuje UC0431.

b) Teller zada rodne cislo alebo CCAID a system sam dotiahne zvysok z GateGlobal.
   Vtedy ale potrebujem vediet ktore pole to spusti a ci sa to deje automaticky
   po vyplneni alebo tam ma byt tlacidlo Vyhladat ktore v mockupe chyba.

Bez toho neviem napisat hlavny tok.
```

Blokuje dokončenie UC. Dopad: mení sa celý hlavný tok a sekcia API.

**2. Ako zistime ze osoba je klientom → Feri**

```
V UC0402 to bolo jednoduche - v prekliku pride pole ccaId a podla neho vieme
ci je klient.

Tu ale preklik nemame. V modale je policko CCAID ktore vyzera ze ho moze
vyplnit teller. Cize:

- Vyplna CCAID teller rucne? Odkial ho zoberie?
- Alebo sa dopĺňa automaticky ked sa osoba najde v GateGlobal?
- A co ked ho teller necha prazdne, berieme to tak ze osoba nie je klientom?

To iste plati pre CIF a PID.
```

Blokuje dokončenie UC. Dopad: mení sa hlavný tok a Opis obrazoviek.

**3. Vola sa aj tu ProductCBIAuthorizedSubjects → Matus Radusovsky**

```
Pri UC0402 sme sa dohodli ze vztah osoby k uctu zistuje UC0402 volanim
ProductCBIAuthorizedSubjects a priznak potom pouziva UC0431 pri poplatku.

Ked UC0407 nahradza UC0402 pri priamom vstupe, tak by ho mal volat tiez, inak
by UC0431 nemal z coho brat priznak a poplatok by sa nevypocital spravne.

Sedi to tak? A co ked osoba nema CCAID - tá sluzba potrebuje identifikator
subjektu. Beriem to tak ze bez CCAID osoba nemoze mat vztah k uctu a teda ide
automaticky o vklad tretou osobou, cize poplatok sa plati hned. Je to spravna
uvaha?
```

Blokuje dokončenie UC. Dopad: mení sa hlavný tok a sekcia API.

**4. Ako teller identifikuje cudzinca → Feri**

```
V mockupe je rodne cislo ako nepovinne pole, co sedi s tym ze cudzinci ho nemaju.

Ale ak sa v UC0407 nieco vyhladava (viz otazka 1), tak podla coho sa vyhlada
cudzinec ktory je klientom TB ale rodne cislo nema? Podla CCAID? Podla PID?

A ak sa nevyhladava nic, tak je to bezpredmetne.
```

Blokuje dokončenie UC. Dopad: mení sa hlavný tok.

**5. Rozdiel medzi X a Zrusit → Anka**

```
Modal ma vpravo hore kriz na zatvorenie a dole tlacidlo Zrusit. Robia to iste
alebo je medzi nimi rozdiel?

A co sa stane ked teller da Zrusit - vrati sa na obrazovku vkladu a moze modal
otvorit znova, alebo sa cela transakcia zrusi?
```

Blokuje dokončenie UC, bez toho nie je testovateľný. Dopad: mení sa alternatívny tok.

**6. Ma byt Potvrdit neaktivne kym nie su vyplnene povinne polia → Feri alebo Anka**

```
V UC0403 to tak mame - tlacidlo Potvrdit je neaktivne kym nie su vyplnene
povinne polia a chybova hlaska sa nezobrazuje.

Navrhujem to iste aj tu, nech je to konzistentne. V mockupe je Potvrdit modre
ale to je asi len ako vyzera ked uz je vsetko vyplnene.

Sedi to?
```

Blokuje dokončenie UC. Dopad: mení sa Opis obrazoviek.

**7. Kto rozhoduje ci sa zavola UC0402 alebo UC0407 → Feri a Matej Pastucha**

```
UC0402 riesi len preklik z GATE, UC0407 len priamy vstup do CashBoxu. Cize
niekto musi rozhodnut ktory z tych dvoch sa zavola.

Su to realizacne UC vkladu a vyberu, alebo je nad tym nieco ine? Aby to vyvojar
vedel naprogramovat.
```

Blokuje dokončenie UC. Dopad: mení sa Biznis zadanie a Vstupné podmienky.

**8. Preco nie je tabulka deposit v datovom modeli → vyvoj**

```
Rovnaka otazka ako pri UC0401, UC0402 a UC0403. Podla vyvojara sa udaje pocas
rozpracovanej transakcie ukladaju do pomocnej tabulky deposit, konkretne udaje
o osobe do stlpca depositor_info. Ta tabulka bola aj v starsej verzii datoveho
modelu, ale v aktualnom cashbox_db.png uz nie je.

Cize preco zmizla z modelu a plati ze sa tam udaje o osobe ukladaju?
```

Blokuje dokončenie UC. Dopad: mení sa sekcia Mapping.

**9. Ake opravnenia znamenaju Konatel, Vkladatel a Disponent → Matus Radusovsky**

```
Rovnaka otazka ako pri UC0402. Majitela viem rozpoznat podla priznaku
IS_OWNER_FLAG v ciselniku CCD_ROLE_TYPE, ale konatel, vkladatel a disponent
v podklade nie su.

Plati len ak sa potvrdi ze UC0407 vola ProductCBIAuthorizedSubjects.
```

Blokuje dokončenie UC, ak sa potvrdí volanie služby. Dopad: naplnenie konfiguračného zoznamu oprávnení.

**10. Kto je vlastnikom UC0407 → Matej Pastucha**

```
V status subore mas UC0407 pridelene ako TODO. Ja som ho teraz spisal, lebo
Feri rozhodol ze tam ma ist scenar priameho vstupu ktory som mal v UC0402
ako alternativny tok.

Nech si to nerobime dvakrat - berieš si to spat alebo to necham u seba?
```

Neblokuje. Dopad: žiadny na obsah UC.

### Biznis zadanie

UC slúži na overenie **osoby, ktorá je fyzicky prítomná pred tellerom** a vykonáva transakciu, v prípade, keď teller spustil transakciu **priamo v CashBoxe bez prekliku z aplikácie GATE**.

Cieľom je získať kompletnú sadu identifikačných údajov o tejto osobe rovnako ako pri UC0402, teda vrátane príznaku, či je osoba klientom banky, a jej vzťahu k účtu.

**Rozdiel oproti UC0402.** Pri prekliku z GATE prichádzajú údaje o osobe z aplikácie GATE. Tu neprichádza nič a teller ich zadáva ručne.

| Situácia | Rieši |
|---|---|
| Teller prišiel do CashBoxu preklikom z GATE | UC0402 - Príprava - Overenie klienta |
| Teller spustil transakciu priamo v CashBoxe | **UC0407** |

[OTVORENY BOD: kto rozhoduje, ktorý z týchto dvoch UC sa zavolá]

**Doklad totožnosti.** Teller zapisuje doklad, ktorý osoba **fyzicky predložila**, nie doklad uložený v systéme banky. Tie sa môžu líšiť, napríklad ak osoba predloží pas a banka má v systéme občiansky preukaz. Preto sa údaje o doklade v UC0407 vypĺňajú vždy manuálne a systém ich z GateGlobal neodvodzuje. Platí to rovnako pre klienta aj neklienta (potvrdil Feri).

**Rozsah UC0407.** Do UC0407 patrí manuálne zadanie identifikačných údajov o osobe, vyhodnotenie príznaku Klient / Neklient a zistenie vzťahu osoby k účtu. Do UC0407 **nepatrí**:

| Čo | Rieši |
|---|---|
| Overenie osoby pri prekliku z GATE | UC0402 - Príprava - Overenie klienta |
| Overenie čísla účtu | UC0401 - Príprava - Overenie čísla účtu |
| Overovanie účtu, teda Brand, AccountStatus, AccountSubType, blokácie a disponibilný zostatok | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 (výbery) |
| Vyhodnotenie whitelistu | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 (výbery) |
| Výpočet poplatku za vklad | UC0431 - Poplatok za vklad - Stanovenie výšky |
| Dotiahnutie údajov o právnickej osobe alebo FOP | [OTVORENY BOD: v ktorom UC sa rieši] |

**Vzťah k susedným UC.** UC0407 beží **po** UC0401 - Príprava - Overenie čísla účtu, rovnako ako UC0402. Dôvod je zhodný: zistenie vzťahu osoby k účtu vyžaduje IBAN účtu, ktorý dotiahne UC0401.

**Výstup UC0407 je zhodný s výstupom UC0402.** Nadväzujúce UC nerozlišujú, ktorým z týchto dvoch UC boli údaje získané, a pracujú s rovnakými príznakmi.

### Aktéri

| Aktér | Čo v tomto UC robí |
|---|---|
| **Teller** | Zadáva všetky identifikačné údaje o osobe manuálne podľa dokladu, ktorý mu osoba fyzicky predložila. Potvrdzuje alebo ruší zadanie |
| **Supervízor-Teller** | V UC0407 nevykonáva žiadnu akciu. Uvedený je preto, že transakciu môže realizovať aj používateľ s rolou Supervízor-Teller, ktorý v tom prípade vystupuje v úlohe tellera. Žiadne schvaľovanie ani override sa v UC0407 nevyžaduje |
| **Systém** | Zobrazuje formulár, vyhodnocuje príznak Klient / Neklient a vzťah osoby k účtu, ukladá údaje pre nadväzujúce UC |

---

## Vstupné podmienky

- Teller je prihlásený
- Pobočka je otvorená
- Pokladňa je otvorená
- UC je vyvolaný len počas klientskych transakcií, teda vkladov, výberov alebo rozmieňania
- **Teller spustil transakciu priamo v CashBoxe.** Preklik z aplikácie GATE neprebehol. Ak by preklik prebehol, overenie osoby rieši UC0402 - Príprava - Overenie klienta a UC0407 sa nevyvoláva
- Prebehol UC0401 - Príprava - Overenie čísla účtu. Číslo účtu je overené a v kontexte transakcie je k dispozícii IBAN potrebný na zistenie vzťahu osoby k účtu
- V lokálnej databáze CashBox je naplnený číselník `id_card_type` a v ODS číselník `ODS_SA.CCD_COUNTRY`

---

## Hlavný tok

**Poznámka k rozsahu.** Nasledujúci tok je popísaný podľa mockupu obrazovky Informácie o vkladateľovi, ktorý neobsahuje funkciu vyhľadania osoby. Ak sa potvrdí, že v UC0407 prebieha dotiahnutie údajov z rozhrania GateGlobal, tok sa rozšíri o príslušné kroky. [OTVORENY BOD: viď otázka 1]

1. Systém zobrazí tellerovi modálne okno Informácie o vkladateľovi. Všetky polia sú prázdne a editovateľné.
2. Teller zadá identifikačné údaje o osobe podľa dokladu totožnosti, ktorý mu osoba fyzicky predložila:
   - priezvisko, meno a prípadne titul
   - rodné číslo, ak ho osoba má
   - dátum narodenia
   - druh dokladu, číslo dokladu a krajinu vystavenia dokladu
3. Teller prípadne zadá identifikátory osoby v banke, teda CCAID, CIF a PID. Polia sú nepovinné. [OTVORENY BOD: odkiaľ teller tieto hodnoty berie, viď otázka 2]
4. Systém priebežne vyhodnocuje, či sú vyplnené všetky povinné polia:
   - Ak povinné polia vyplnené nie sú, tlačidlo Potvrdiť zostáva neaktívne a chybová hláška sa nezobrazuje
   - Ak sú povinné polia vyplnené, systém sprístupní tlačidlo Potvrdiť
5. Teller zvolí Potvrdiť. Ak zvolí Zrušiť alebo modálne okno zatvorí, tok pokračuje **AT1**.
6. Systém overí formát zadaných údajov podľa tabuľky kontrol v sekcii Opis obrazoviek + Validácie:
   - Ak sú všetky údaje v správnom formáte, UC pokračuje nasledujúcim krokom
   - Ak niektorý údaj v správnom formáte nie je, tok pokračuje **AT2**
7. Systém vyhodnotí, či je osoba klientom banky:
   - Ak je vyplnené pole CCAID, osoba je klientom TB
   - Ak pole CCAID vyplnené nie je, osoba nie je klientom TB
   
   Systém výsledok uloží ako príznak pre nadväzujúce UC. Vyhodnotenie nie je blokujúce. [OTVORENY BOD: viď otázka 2]
8. Systém zavolá rozhranie ProductCBIAuthorizedSubjects (v4) s identifikáciou účtu, teda s číslom účtu vo formáte IBAN prevzatým z UC0401 - Príprava - Overenie čísla účtu, a s identifikátorom osoby. Rozhranie vráti zoznam osôb, ktoré majú k danému účtu oprávnenie:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT3**
   
   [OTVORENY BOD: či sa rozhranie v UC0407 volá, viď otázka 3]
9. Systém overí, či má osoba prítomná pred tellerom k účtu aspoň jedno oprávnenie zaradené v konfiguračnom zozname oprávnení zakladajúcich vzťah k účtu:
   - Ak také oprávnenie má, systém uloží príznak, že osoba má vzťah k účtu
   - Ak nemá žiadne, alebo ak osoba nemá CCAID, systém uloží príznak, že osoba vzťah k účtu nemá
   
   Vyhodnotenie nie je blokujúce. Príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky.
10. Systém uloží zadané údaje o osobe tak, aby boli k dispozícii nadväzujúcim UC. [OTVORENY BOD: či sa údaje zapisujú do pomocnej tabuľky `deposit`, viď sekcia Mapping]
11. Systém zatvorí modálne okno a ukončí UC s úspechom. Tok pokračuje v UC, z ktorého bol UC0407 vyvolaný, teda v príslušnom UC vkladu, výberu alebo rozmieňania.

---

## Alternatívny tok

### AT1 - Teller zruší zadanie

**Spúšťač:** Teller v kroku 5 hlavného toku zvolí Zrušiť alebo zatvorí modálne okno krížikom.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 5.

1. Systém zatvorí modálne okno bez uloženia zadaných údajov.
2. Systém neuloží žiadny príznak o osobe.
3. [OTVORENY BOD: kam sa teller dostane. Či sa vráti na obrazovku transakcie a môže modál otvoriť znova, alebo sa celá transakcia zruší. Viď otázka 5]

### AT2 - Zadaný údaj nie je v správnom formáte

**Spúšťač:** Systém v kroku 6 hlavného toku vyhodnotí, že niektorý zo zadaných údajov nespĺňa formát podľa tabuľky kontrol.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 6.

1. Systém zobrazí chybovú hlášku s popisom chyby. [OTVORENY BOD: v katalógu nie je schválená hláška pre nesprávny formát údajov o osobe]
2. Systém zvýrazní pole, ktoré chybu obsahuje.
3. Teller opraví údaj.
4. Teller zvolí Potvrdiť.
5. UC pokračuje krokom 6 hlavného toku.

### AT3 - Rozhranie ProductCBIAuthorizedSubjects nie je dostupné

**Spúšťač:** Systém nedostane odpoveď z rozhrania ProductCBIAuthorizedSubjects.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 8.

1. Systém zistí nedostupnosť rozhrania podľa vypršania timeoutu volania alebo podľa chybovej odpovede mimo biznisových chýb. [OTVORENY BOD: potvrdiť mechanizmus a hodnotu timeoutu]
2. Systém nemá ako zistiť vzťah osoby k účtu a náhradný zdroj neexistuje.
3. Systém zobrazí tellerovi chybovú hlášku o nedostupnosti služby. Návrh textu: "Sluzba pre overenie klienta je nedostupna, skuste to prosim neskor." [OTVORENY BOD: hláška nie je v katalógu, ide o návrh. Zhodná s návrhom v UC0402]
4. Transakcia nemôže pokračovať a UC končí neúspešne.

---

## Diagram tokov

[OTVORENY BOD: diagram bude doplnený]

---

## Výstupné podmienky

**Úspech:**
- Identifikačné údaje o osobe sú kompletné a zadané tellerom
- Systém má uložený príznak, či je osoba klientom banky
- Systém má uložený príznak vzťahu osoby k účtu
- Nadväzujúce UC môžu pracovať s uloženými dátami
- Tok pokračuje v UC, z ktorého bol UC0407 vyvolaný
- Výstup je zhodný s výstupom UC0402 - Príprava - Overenie klienta

**Zlyhanie:**

| Druh zlyhania | Čo sa nezapísalo | Kde sa teller nachádza |
|---|---|---|
| Teller nevyplnil povinné polia | Nič, UC neprešiel do kroku 6 | V modálnom okne, tlačidlo Potvrdiť je neaktívne |
| Zadaný údaj nemá správny formát (AT2) | Nič | V modálnom okne s vyznačeným chybným poľom |
| Teller zrušil zadanie (AT1) | Nič | [OTVORENY BOD: viď AT1] |
| ProductCBIAuthorizedSubjects je nedostupný (AT3) | Príznak vzťahu k účtu sa neuložil | Transakcia nemôže pokračovať |

---

## Opis obrazoviek + Validácie

### Modálne okno Informácie o vkladateľovi

Modálne okno sa zobrazuje nad obrazovkou transakcie. Na pozadí zostáva viditeľná obrazovka vkladu s poliami Číslo účtu BBAN, Mena, Variabilný symbol, Referencia platiteľa a Popis, ktoré rieši UC0403 - Príprava - Natypovanie transakcie, a karta s vypočítaným poplatkom za vklad, ktorú rieši UC0431 - Poplatok za vklad - Stanovenie výšky.

Rozloženie polí je do troch stĺpcov. Zdroj: Figma mockup Informácie o vkladateľovi.

M = Mandatory (Y = povinné, N = nepovinné), E = Editable (Y = editovateľné, N = read-only)

| Názov | Dátový typ | Validácia | M | E | Popis |
|---|---|---|---|---|---|
| Priezvisko | Text | - | Y | Y | Priezvisko osoby podľa predloženého dokladu |
| Meno | Text | - | Y | Y | Meno osoby podľa predloženého dokladu |
| Titul | Text | - | N | Y | Titul pred menom |
| Rodné číslo | Text | 9 až 10 miest | N | Y | Rodné číslo osoby, ak ho osoba má |
| Dátum narodenia | Date (DD/MM/YYYY) | Formát DD/MM/YYYY, výber cez kalendár | Y | Y | Dátum narodenia |
| Druh dokladu | Dropdown | Hodnoty z číselníka `id_card_type` | Y | Y | Druh dokladu, ktorý osoba fyzicky predložila |
| Číslo dokladu | Text | - | Y | Y | Číslo predloženého dokladu |
| Krajina vystavenia dokladu | Dropdown | Hodnoty z číselníka `ODS_SA.CCD_COUNTRY` | Y | Y | Krajina vystavenia predloženého dokladu |
| CCAID | Text | - | N | Y | Identifikátor klienta v TB, ak je osoba klientom [OTVORENY BOD: odkiaľ teller hodnotu berie] |
| CIF | Text | - | N | Y | Identifikátor CIF [OTVORENY BOD: odkiaľ teller hodnotu berie] |
| PID | Text | - | N | Y | Identifikátor PID [OTVORENY BOD: odkiaľ teller hodnotu berie] |

**Pozn. k dokladu totožnosti.** Údaje o doklade sa vždy vzťahujú na doklad, ktorý osoba fyzicky predložila, nie na doklad uložený v systéme banky. Systém ich z GateGlobal neodvodzuje ani nepredvypĺňa. Platí to rovnako pre klienta aj neklienta (potvrdil Feri).

**Pozn. k rodnému číslu.** Pole je nepovinné, pretože ho cudzinci nemajú.

### Tabuľka kontrol

| # | Kontrola | Kde beží | Podmienka pre pokračovanie | Pri nesplnení | Testovateľné cez |
|---|---|---|---|---|---|
| 1 | Vyplnenie povinných polí | CashBox lokálne | Vyplnené sú priezvisko, meno, dátum narodenia, druh dokladu, číslo dokladu a krajina vystavenia dokladu | Tlačidlo Potvrdiť zostáva neaktívne, hláška sa nezobrazuje | Nechať prázdne pole Priezvisko a skúsiť potvrdiť |
| 2 | Formát rodného čísla | CashBox lokálne | 9 až 10 miest, ak je pole vyplnené | AT2 | Zadať 8 alebo 11 znakov |
| 3 | Formát dátumu narodenia | CashBox lokálne | Platný dátum vo formáte DD/MM/YYYY | AT2 | Zadať neplatný dátum |
| 4 | Hodnota druhu dokladu | CashBox lokálne | Hodnota je z číselníka `id_card_type` | AT2 | Nevyberať z rozbaľovacieho zoznamu |
| 5 | Hodnota krajiny vystavenia | CashBox lokálne | Hodnota je z číselníka `ODS_SA.CCD_COUNTRY` | AT2 | Nevyberať z rozbaľovacieho zoznamu |

Všetky kontroly bežia lokálne v CashBoxe.

### Správanie tlačidiel

| Tlačidlo | Správanie |
|---|---|
| Potvrdiť | Neaktívne, kým nie sú vyplnené všetky povinné polia. Chybová hláška o nevyplnených povinných poliach sa nezobrazuje. Kontrola formátu prebieha až po kliknutí. [OTVORENY BOD: potvrdiť, navrhnuté podľa vzoru z UC0403] |
| Zrušiť | Zatvorí modálne okno bez uloženia údajov, viď AT1 |
| Krížik vpravo hore | [OTVORENY BOD: či robí to isté ako Zrušiť] |

---

## API

### Rozhrania

| Rozhranie | Verzia | Účel | Použitie v UC0407 |
|---|---|---|---|
| GateGlobal.ProductCBIAuthorizedSubjects | v4 | Zoznam oprávnených osôb k produktu spolu s typmi oprávnení | Zistenie vzťahu osoby k účtu v krokoch 8 a 9. [OTVORENY BOD: či sa v UC0407 volá] |
| GateGlobal.CustomerCBIFind | v6.0 | Vyhľadanie klienta podľa identifikátorov | [OTVORENY BOD: podľa mockupu sa nepoužíva, viď otázka 1] |
| GateGlobal.CustomerCBIDetail | v6.2 | Detail klienta | [OTVORENY BOD: podľa mockupu sa nepoužíva, viď otázka 1] |

**Poznámka k verzii ProductCBIAuthorizedSubjects.** Používa sa verzia v4, ktorá je v Interface Specifications uvedená ako používaná. Verzia V5 prináša oproti v4 len možnosť identifikácie kartového produktu podľa identifikátora karty, čo CashBox nepotrebuje.

### Číselníky

| Zdroj | Tabuľka | Použitie v UC0407 |
|---|---|---|
| DB CashBox | `id_card_type` (`code` varchar(20), `description` varchar(20)) | Rozbaľovací zoznam Druh dokladu |
| ODS | `ODS_SA.CCD_COUNTRY` | Rozbaľovací zoznam Krajina vystavenia dokladu |
| GateGlobal | `CCD_ROLE_TYPE` | Typy oprávnení k produktu, vyhodnotenie vzťahu k účtu |

### Vyhodnotenie vzťahu k účtu

Zhodné s UC0402 - Príprava - Overenie klienta.

Systém vyhodnotí, že osoba má vzťah k účtu, ak má k nemu aspoň jedno oprávnenie zaradené v konfiguračnom zozname oprávnení zakladajúcich vzťah k účtu. Zoznam sa napĺňa hodnotami z číselníka `CCD_ROLE_TYPE` a zodpovedá kategóriám Majiteľ (M), Konateľ (K), Vkladateľ (V) a Disponent (D) podľa UC0431 - Poplatok za vklad - Stanovenie výšky.

Majiteľské oprávnenia systém rozpozná podľa aktívneho príznaku `IS_OWNER_FLAG` v číselníku `CCD_ROLE_TYPE`. Podľa podkladu k rozhraniu ide typicky o oprávnenia MAJ1_PV, MAJ2_PV a CIFM_PV.

[OTVORENY BOD: zaradenie oprávnení pre kategórie Konateľ, Vkladateľ a Disponent]

**Osobitosť UC0407.** Ak osoba nemá CCAID, nemá pridelený identifikátor subjektu a nemôže mať k účtu evidované oprávnenie. Systém v takom prípade uloží príznak, že osoba vzťah k účtu nemá, a rozhranie nevolá. [OTVORENY BOD: potvrdiť túto úvahu]

### ProductCBIAuthorizedSubjects - parametre requestu

| Objekt | Parameter | Hodnota z CashBoxu | Poznámka |
|---|---|---|---|
| RequestCVO | channel | [OTVORENY BOD: hodnota kanála. V CashBoxe existuje konštanta CASHBOX_GG_APPNAME] | Kanál musí byť registrovaný v konfigurácii služby, inak volanie skončí chybou ESBI102 |
| RequestCVO | timestamp | Časová pečiatka správy | |
| RequestCVO | version | v4 | |
| RequestCVO | subversion | 0 | |
| ProductAuthorizedSubjectsRequestCVO | brand | 001 (GATE TB) | 001 = GATE TB, 002 = SUN |
| ProductAuthorizedSubjectsRequestCVO | productNumber/Account | IBAN účtu | IBAN je dotiahnutý v UC0401 - Príprava - Overenie čísla účtu |
| ProductAuthorizedSubjectsRequestCVO | derivedRolesFlag | Neposiela sa | Aktívny príznak pri produkte z brandu RB (002) vedie k chybe ESBI105 |
| ProductAuthorizedSubjectsRequestCVO | subjectIdentification | CCAID zadané tellerom | [OTVORENY BOD: čo sa pošle, ak teller CCAID nezadal] |

### Chybové kódy rozhraní

| Kód | Význam | Použitie v UC0407 |
|---|---|---|
| AUTH100 | Pre zadaný identifikátor neexistuje oprávnená osoba | Vyhodnotenie vzťahu k účtu, krok 9 |
| ESBI102 | Zadaný kanál nie je súčasťou konfigurácie | Predpoklad volania služby |

Ostatné chybové kódy nie sú predmetom tejto špecifikácie, platia podľa dokumentácie rozhraní.

### Nadväznosť

- **Predchádza:** UC0401 - Príprava - Overenie čísla účtu, ktorý overí číslo účtu a dotiahne IBAN
- **Alternatíva k UC0407:** UC0402 - Príprava - Overenie klienta, ktorý sa vyvolá, ak teller prišiel preklikom z aplikácie GATE
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery). Príznak vzťahu k účtu využíva UC0431 - Poplatok za vklad - Stanovenie výšky

---

## Mapping

### Uloženie údajov počas rozpracovanej transakcie

UC0407 potrebuje zadané údaje sprístupniť nadväzujúcim UC. Podľa vývojára slúži na priebežné údaje transakcie pomocná tabuľka `deposit`, do ktorej sa postupne ukladajú údaje o klientovi, suma, kurz a ďalšie hodnoty. Údaje o osobe by patrili do stĺpca `depositor_info`.

Tabuľka `deposit` bola v staršej verzii dátového modelu CashBox. V aktuálnom modeli `cashbox_db.png` uvedená nie je.

[OTVORENY BOD: prečo tabuľka nie je v aktuálnom dátovom modeli a či sa do nej ukladajú aj údaje o osobe. Rovnaká otázka je otvorená v UC0401, UC0402 a UC0403]

### Údaje zadané tellerom

Všetky údaje o osobe v UC0407 zadáva teller manuálne. Systém ich z externých rozhraní nedotiahne ani nepredvyplní.

| Pole | Zdroj |
|---|---|
| Priezvisko, Meno, Titul | Teller podľa predloženého dokladu |
| Rodné číslo | Teller, ak ho osoba má |
| Dátum narodenia | Teller podľa predloženého dokladu |
| Druh dokladu, Číslo dokladu, Krajina vystavenia dokladu | Teller podľa dokladu, ktorý osoba fyzicky predložila |
| CCAID, CIF, PID | Teller [OTVORENY BOD: odkiaľ hodnoty berie] |

### Príznaky odovzdávané nadväzujúcim UC

| Príznak | Ako sa určí |
|---|---|
| Klient / Neklient | Podľa vyplnenia poľa CCAID |
| Vzťah k účtu | Podľa odpovede rozhrania ProductCBIAuthorizedSubjects, krok 9 |

**Mimo rozsah UC0407:**
- Overenie osoby pri prekliku z GATE, rieši UC0402 - Príprava - Overenie klienta
- Overovanie účtu, rieši UC0404 alebo UC0504
- Vyhodnotenie whitelistu, rieši UC0404 alebo UC0504
- Výpočet poplatku za vklad, rieši UC0431 - Poplatok za vklad - Stanovenie výšky
- Údaje o právnickej osobe alebo FOP, [OTVORENY BOD: v ktorom UC sa riešia]

---

## Poznámky pre teba - nekopírovať do UC

### Mockup zmenil pôvodnú predstavu z AT1

V AT1 v UC0402 sme mali, že teller zadá identifikátor, zvolí **Vyhľadať**, systém zavolá CustomerCBIFind a potom CustomerCBIDetail.

**V mockupe žiadne tlačidlo Vyhľadať nie je.** Modál vyzerá ako čisto manuálne vypĺňanie: teller zadá všetko a potvrdí.

To si ale protirečí s Feriho vetou *"len pri klientovi vieme ostatné polia doplniť ako je rodné číslo, tituly atď"*. Ak sa nič nevyhľadáva, nemá to ako doplniť.

Preto som hlavný tok napísal podľa mockupu, teda ako manuálne vypĺňanie, a otázku o vyhľadávaní dal ako **otázku číslo 1**, lebo od nej závisí celý tok. Ak sa potvrdí, že vyhľadávanie existuje, pribudnú kroky a dve rozhrania do sekcie API.

Nechcel som to domyslieť, lebo by som napísal tok, ktorý nezodpovedá ani mockupu, ani Feriho vete.

### Čo mockup naopak potvrdil

| Vec | Zistenie |
|---|---|
| Tlačidlá | Zrušiť a Potvrdiť, plus krížik. Tým sa čiastočne uzatvára otázka, ktorú sme mali otvorenú aj v UC0402 |
| Povinné polia | Priezvisko, Meno, Dátum narodenia, Druh dokladu, Číslo dokladu, Krajina vystavenia. Zhoduje sa s Feriho maticou |
| Rodné číslo | Nepovinné, čo podporuje scenár s cudzincami |
| CCAID, CIF, PID | **Na obrazovke sú** a vyzerajú ako editovateľné. Tým sa uzatvára otázka z UC0402, či sa vôbec zobrazujú |

### Dôsledok pre UC0402

Keďže mockup ukazuje CCAID, CIF a PID ako polia na obrazovke, do UC0402 by sa mali vrátiť do tabuľky polí. Odstránil som ich tam predtým, lebo v UC0450 je uvedené, že z prekliku pre vklad neprídu. Ale to znamená len to, že sa musia dotiahnuť z CustomerCBIDetail, nie že sa nezobrazujú.

Navrhujem do matice v UC0402 doplniť tri riadky:

```
| CCAID | Text | - | N | Y | Identifikátor klienta v TB | `ccaId` | Preklik | - |
| CIF | Text | - | N | Y | Identifikátor CIF | z prekliku nepríde | GateGlobal | - |
| PID | Text | - | N | Y | Identifikátor PID | z prekliku nepríde | GateGlobal | - |
```

### Vlastníctvo UC0407

V status súbore je UC0407 pridelený **Matejovi Pastuchovi** so stavom TODO. Ty si ho teraz napísal. Oplatí sa to s ním vyjasniť skôr, než na ňom začne pracovať aj on.
