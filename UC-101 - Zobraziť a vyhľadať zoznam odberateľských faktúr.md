Prešiel som Evkinu špecifikáciu do detailu. Je stručnejšia než podklady pre dodávateľské faktúry, obsahuje vlastné otvorené otázky a na dvoch miestach nevyplnené polia, takže veľká časť UC zostane otvorená. Dôležité je, že som z nej vytiahol dve veci, ktoré menia obraz o rozsahu - píšem o nich pod špecifikáciou.

---

# UC-101 - Zobraziť a vyhľadať zoznam odberateľských faktúr

## Obsah

- Obsah
- Otázky
- Biznis zadanie
- Aktéri
- Spúšťač
- Vstupné podmienky
- Hlavný tok
- Alternatívny tok
- Diagram tokov
- Výstupné podmienky
- Opis Obrazoviek + Validácie
- API

## Otázky

| # | Otvorená otázka | Adresát |
|---|---|---|
| 1 | Ktoré konkrétne dáta z faktúry sa v zozname zobrazujú? V biznis špecifikácii sú na tomto mieste nevyplnené položky. | Biznis (Evka, Iveta) |
| 2 | Za aké obdobie sa faktúry v zozname zobrazujú - polrok, rok, iné? Otázka je otvorená priamo v biznis špecifikácii. | Biznis (Evka) |
| 3 | Dá sa vyhľadávať podľa každého stĺpca? V biznis špecifikácii je požiadavka označená otáznikom s odkazom na Ivetu. | Iveta |
| 4 | Je potrebné zoradenie zoznamu a aké sú technické možnosti? V biznis špecifikácii je celá požiadavka označená otáznikom s odkazom na Ivetu. | Iveta, vývoj |
| 5 | Ako je definované, ktoré faktúry sú „svoje"? Podľa oddelenia používateľa, podľa zdrojovej aplikácie, alebo podľa používateľa, ktorý faktúru vytvoril? Ako sa určí príslušnosť pri faktúrach a dobropisoch vytvorených priamo v eInvoice? | Biznis |
| 6 | Existuje rola, ktorá vidí faktúry všetkých oddelení - napríklad účtáreň, interná kontrola alebo podpora? | Biznis |
| 7 | Poskytuje Digitálny poštár informáciu o tom, že faktúru prijal poštár protistrany? Biznis špecifikácia túto požiadavku uvádza podmienene - „ak poštár takúto informáciu poskytuje". | Architekt |
| 8 | Aký je číselník stavov odosielania a stavov od poštára, ktoré sa v zozname zobrazujú? | Biznis, architekt |
| 9 | Ako sa má označiť faktúra, ktorá nebola odoslaná poštárom, ale len uložená? | Biznis (Evka) |
| 10 | Ktoré kritériá majú umožniť intervalové vyhľadávanie od-do? Biznis špecifikácia uvádza ako príklady číslo faktúry a dátum dodania. | Biznis |
| 11 | Zobrazujú sa v zozname aj dobropisy prijaté zo zdrojových aplikácií, alebo len tie vytvorené priamo v eInvoice? Biznis špecifikácia výslovne uvádza iba dobropisy inputované v eInvoice. | Biznis |
| 12 | Koľko záznamov sa zobrazuje naraz a akým spôsobom sa načítavajú ďalšie? | Biznis, vývoj |
| 13 | Je preklik na vystavenie dobropisu dostupný pri každej faktúre, alebo len pri faktúrach v určitom stave? | Biznis |
| 14 | Je zoznam pri otvorení predfiltrovaný alebo prednastavene zoradený? | Biznis |

## Biznis zadanie

Zdroj: biznis špecifikácia UC-101 od biznis architektky.

**Cieľ UC.** Poskytnúť zoznam (elektronický obraz) vystavených faktúr a možnosť vyhľadávať konkrétne faktúry podľa informácií uvedených v jednotlivých stĺpcoch prehľadu.

**Komu je zoznam určený.** Zoznam je určený primárne pre používateľov z jednotlivých oddelení, ktoré vystavujú faktúry. **Každý používateľ má vidieť len „svoje" faktúry**, vrátane svojich faktúr a dobropisov vytvorených priamo v eInvoice.

**Čo obrazovka poskytuje.** Používateľ dostane zoznam svojich faktúr a možnosť vyhľadávať podľa rôznych kritérií zodpovedajúcich zobrazeným stĺpcom. Musí byť možné nájsť konkrétnu faktúru - zadaním priamo čísla faktúry alebo iných zobrazených informácií, napríklad odberateľa.

**Informácie o odoslaní a stave.** Používateľ dostane základné informácie faktúry a zároveň informáciu, či bola faktúra odoslaná poštárovi a v akom je stave. Ide o stav počas odosielania aj finálny stav od poštára, až po stav, kedy bola faktúra prijatá poštárom protistrany, ak poštár takúto informáciu poskytuje.

Zoznam poskytne:

- základné dáta faktúry,
- informáciu, či bola faktúra odoslaná poštárom, alebo len uložená,
- informácie o poslednom a aktuálnom stave odoslania poštárovi, teda stav, či poštár prijímateľa faktúru prijal a prešla jeho kontrolou, a dátum a čas tejto informácie.

**Vyhľadávanie v intervaloch.** Vyhľadávacie kritériá majú umožniť vyhľadávanie v intervaloch od-do, napríklad zobrazenie faktúr podľa čísla od-do alebo podľa dátumu dodania od-do.

**Preklik na nadväzujúce obrazovky.** Z konkrétnej faktúry v zozname sa používateľ dostane na detail faktúry, na obrazovku s detailmi komunikácie s poštárom, na obrazovku zmien vykonaných na faktúre po jej zamietnutí, na vystavenie dobropisu a na tlač PDF konkrétnej faktúry.

## Aktéri

**Hlavný aktér:** Používateľ oddelenia, ktoré vystavuje faktúry

Na rozdiel od zoznamu dodávateľských faktúr, ktorý používa jedno oddelenie, tento zoznam používajú používatelia viacerých oddelení a každý vidí iba faktúry svojho oddelenia.

**Systém:** eInvoice

**Vedľajší systém:** Digitálny poštár - zdroj informácií o stave odoslania a doručenia

(OTVORENY BOD: definícia príslušnosti faktúry k používateľovi a existencia role s prístupom naprieč oddeleniami - viď otázky 5 a 6)

## Spúšťač

Používateľ otvorí obrazovku zoznamu odberateľských faktúr v aplikácii eInvoice.

## Vstupné podmienky

> - Používateľ je prihlásený do aplikácie a má oprávnenie na prístup k zoznamu odberateľských faktúr
> - V DB eInvoice existujú odberateľské faktúry alebo dobropisy patriace používateľovi
> - Faktúra má priradený stav odoslania

## Hlavný tok

1. Používateľ otvorí zoznam odberateľských faktúr
2. Systém načíta faktúry a dobropisy, ktoré patria používateľovi
3. Systém zobrazí zoznam so základnými dátami faktúry
4. Systém pri každej faktúre zobrazí informáciu, či bola odoslaná poštárom, alebo len uložená
5. Systém pri každej faktúre zobrazí posledný a aktuálny stav odoslania poštárovi vrátane dátumu a času tejto informácie
6. Používateľ zadá vyhľadávacie kritériá; pri vybraných kritériách zadá interval od-do, napríklad číslo faktúry od-do alebo dátum dodania od-do
7. Systém zobrazí faktúry vyhovujúce zadaným kritériám
8. Používateľ vyberie konkrétnu faktúru zo zoznamu
9. Používateľ zvolí preklik na nadväzujúcu obrazovku

## Alternatívny tok

**AT 6a - Vyhľadanie konkrétnej faktúry podľa čísla**

Podmienka: Používateľ pozná číslo faktúry a hľadá jednu konkrétnu.

1. Používateľ zadá číslo faktúry priamo.
2. Systém zobrazí faktúru zodpovedajúcu zadanému číslu.

- Tok pokračuje krokom 8.

**AT 6b - Vyhľadanie podľa odberateľa a ďalších zobrazených údajov**

Podmienka: Používateľ nepozná číslo faktúry a hľadá podľa iných údajov.

1. Používateľ zadá hodnotu do niektorého z vyhľadávacích kritérií zodpovedajúcich zobrazeným stĺpcom, napríklad odberateľa.
2. Systém zobrazí faktúry vyhovujúce zadanej hodnote.

- Tok pokračuje krokom 8.
- (OTVORENY BOD: či je vyhľadávanie dostupné podľa každého stĺpca - viď otázka 3)

**AT 7a - Zoradenie zoznamu**

Podmienka: Používateľ potrebuje zoznam zoradiť.

1. Používateľ zvolí spôsob zoradenia, napríklad podľa abecedy alebo podľa sumy zostupne.
2. Systém zobrazí zoradený zoznam.

- Tok pokračuje krokom 8.
- (OTVORENY BOD: požiadavka na zoradenie nie je potvrdená; v biznis špecifikácii je označená otáznikom - viď otázka 4)

**AT 9a - Preklik na detail faktúry**

1. Používateľ zvolí preklik na detail vybranej faktúry.
2. Systém otvorí obrazovku detailu odchádzajúcej faktúry (UC-103).

- UC končí.

**AT 9b - Preklik na detaily komunikácie s poštárom**

1. Používateľ zvolí preklik na obrazovku s detailmi komunikácie s poštárom.
2. Systém zobrazí stavy, ktorými faktúra prechádzala, a časy od príchodu záznamu z pôvodnej aplikácie, prípadne od vytvorenia faktúry priamo v eInvoice.

- UC končí.
- Poznámka: rozsah zobrazených stavov a časov je predmetom dohody s Digitálnym poštárom.

**AT 9c - Preklik na zmeny vykonané na faktúre**

Podmienka: Faktúra bola poštárom zamietnutá a bolo potrebné na nej niečo zmeniť.

1. Používateľ zvolí preklik na obrazovku zmien vykonaných na faktúre.
2. Systém zobrazí zmenené polia spolu s informáciou, kto a kedy ich zmenil.

- UC končí.

**AT 9d - Preklik na vystavenie dobropisu**

1. Používateľ zvolí preklik na vystavenie dobropisu k vybranej faktúre.
2. Systém otvorí obrazovku vystavenia dobropisu (UC-105) s predvyplneným identifikátorom dobropisovanej faktúry.

- UC končí.
- (OTVORENY BOD: či je preklik dostupný pri každej faktúre - viď otázka 13)

**AT 9e - Tlač PDF konkrétnej faktúry**

1. Používateľ zvolí tlač PDF vybranej faktúry.
2. Systém vygeneruje PDF konkrétnej faktúry (UC-106).

- UC končí.

## Diagram tokov

## Výstupné podmienky

- Používateľ videl zoznam svojich faktúr a dobropisov so základnými dátami, s informáciou o odoslaní poštárom a s aktuálnym stavom odoslania vrátane dátumu a času tejto informácie
- Zoznam zodpovedá zadaným vyhľadávacím kritériám
- Zobrazenie a vyhľadávanie nemení stav ani obsah žiadnej faktúry; obrazovka je určená na čítanie a navigáciu
- Používateľ prešiel na jednu z nadväzujúcich obrazoviek, alebo pokračuje v práci so zoznamom

## Opis Obrazoviek + Validácie

FIGMA:

### Obrazovka č. 1: Zoznam odberateľských faktúr

**Popis obrazovky:** Prehľadová tabuľka vystavených faktúr a dobropisov patriacich používateľovi. Obsahuje vyhľadávaciu časť s možnosťou zadania intervalov od-do a preklik na nadväzujúce obrazovky. Obrazovka je určená na čítanie a navigáciu; údaje sa na nej neupravujú.

**Zobrazované údaje**

| Údaj | Popis | Stav špecifikácie |
|---|---|---|
| Základné dáta faktúry | Údaje z faktúry zobrazené v stĺpcoch prehľadu | (OTVORENY BOD: Chýba zoznam polí pre obrazovku - viď otázka 1) |
| Odoslanie poštárom | Informácia, či bola faktúra odoslaná poštárom, alebo len uložená | Potvrdené; označenie stavu pri neodoslaných faktúrach je otvorené (viď otázka 9) |
| Stav odoslania | Posledný a aktuálny stav odoslania poštárovi vrátane stavu, či poštár prijímateľa faktúru prijal a prešla jeho kontrolou | Potvrdené; číselník hodnôt otvorený (viď otázka 8), dostupnosť stavu od poštára protistrany podmienená (viď otázka 7) |
| Dátum a čas stavu | Dátum a čas informácie o stave odoslania | Potvrdené |

**Vyhľadávacie kritériá**

| Kritérium | Typ vyhľadávania | Poznámka |
|---|---|---|
| Číslo faktúry | Presná hodnota aj interval od-do | Uvedené v biznis špecifikácii ako príklad intervalového vyhľadávania |
| Odberateľ | Hodnota | Uvedené v biznis špecifikácii ako príklad vyhľadávania podľa zobrazených informácií |
| Dátum dodania | Interval od-do | Uvedené v biznis špecifikácii ako príklad intervalového vyhľadávania |
| Ostatné zobrazené stĺpce | (OTVORENY BOD: viď otázka 3) | Požiadavka vyhľadávať podľa každého stĺpca nie je potvrdená |

**Akcie na obrazovke**

| Akcia | Cieľová obrazovka | Poznámka |
|---|---|---|
| Otvoriť detail faktúry | UC-103 Zobraziť detail odchádzajúcej faktúry | - |
| Zobraziť komunikáciu s poštárom | Obrazovka stavov, ktorými faktúra prechádzala | Vrátane časov od príchodu záznamu z pôvodnej aplikácie alebo od vytvorenia v eInvoice |
| Zobraziť zmeny na faktúre | Obrazovka zmenených polí s údajom, kto a kedy ich zmenil | Relevantné pri faktúrach zamietnutých poštárom |
| Vystaviť dobropis | UC-105 Vytvoriť a upraviť dobropis | Identifikátor dobropisovanej faktúry sa predvyplní |
| Tlačiť PDF | UC-106 Vygenerovať PDF odberateľskej faktúry | - |

**Pravidlá zobrazenia**

| Pravidlo | Stav špecifikácie |
|---|---|
| Používateľ vidí iba faktúry a dobropisy, ktoré sú „jeho" | Potvrdené; definícia príslušnosti otvorená (viď otázka 5) |
| Zoznam obsahuje aj dobropisy vytvorené priamo v eInvoice | Potvrdené; zaradenie dobropisov zo zdrojových aplikácií otvorené (viď otázka 11) |
| Obdobie zobrazovaných faktúr | (OTVORENY BOD: viď otázka 2) |
| Zoradenie zoznamu | (OTVORENY BOD: požiadavka nie je potvrdená - viď otázka 4) |
| Počet záznamov a načítanie ďalších | (OTVORENY BOD: viď otázka 12) |

## API

- Dáta zoznamu pochádzajú z DB eInvoice - faktúry prijaté zo zdrojových aplikácií aj faktúry a dobropisy vytvorené priamo v eInvoice
- Stavy odoslania a doručenia pochádzajú z komunikácie s Digitálnym poštárom
- (OTVORENY BOD: rozsah stavov, ktoré Digitálny poštár poskytuje, vrátane informácie o prijatí poštárom protistrany - viď otázka 7)
- (OTVORENY BOD: technický spôsob obmedzenia rozsahu dát podľa príslušnosti používateľa - viď otázka 5)

---

## Čo som pri analýze našiel a odporúčam riešiť

**1. Sú tam dve rôzne obrazovky histórie, nie jedna.**

Biznis špecifikácia rozlišuje dva prekliky, ktoré vyzerajú podobne, ale sú to iné veci:

| Preklik | Čo zobrazuje | Zodpovedá |
|---|---|---|
| Detaily komunikácie s poštárom | Stavy, ktorými faktúra prechádzala, a časy | História stavov - Evka ju potvrdila ako samostatnú obrazovku |
| Zmeny na faktúre | Zmenené polia, kto a kedy ich zmenil | História zmien - to, čo dnes máme ako UC-203 |

V UC mape máme dnes iba jednu históriu. Ak sa toto potvrdí, potrebujeme dve: históriu stavov a históriu zmien polí. Odporúčam to zaradiť do stromu skôr, než sa začnú písať ďalšie UC odberateľských faktúr.

**2. Obmedzenie rozsahu dát podľa používateľa je nová vec, ktorá nemá obdobu v dodávateľskej vetve.**

Pri dodávateľských faktúrach vidí Prevádzková účtáreň všetko. Tu má každý používateľ vidieť len faktúry svojho oddelenia. To znamená, že prístup nie je len o tom, kto sa dostane do aplikácie, ale aj o tom, ktoré riadky vidí. Pre vývoj je to podstatný rozdiel a mal by to vedieť skôr, než začne.

Zároveň to súvisí s otázkou, ktorá nám visí od začiatku - matica oprávnení. Bez odpovede na to, ako sa určuje príslušnosť faktúry k oddeleniu, sa nedá dokončiť ani UC-101, ani matica.

**3. Dva zoznamy prekliknutí v špecifikácii sa mierne líšia.**

V úvodnej časti sú uvedené tri prekliky (detail, dobropis, PDF), v požiadavkách na obrazovku štyri (detail, komunikácia s poštárom, zmeny, PDF). Dobropis v druhom zozname chýba, obe histórie chýbajú v prvom. Zlúčil som ich do piatich, keďže vecne si neodporujú, ale stojí za to si to s Evkou potvrdiť.

**4. Kandidát na chýbajúci zoznam stĺpcov.**

Nevypĺňal som ho, keďže si žiadal držať sa biznis špecifikácie. Hárok pre odberateľské faktúry v xls od biznisu však obsahuje deväť stĺpcov, ktoré eInvoice dopĺňa - dátum a čas prijatia, zdrojová aplikácia, doručenie cez Peppol, dátum odoslania poštárovi, dátum odpovede od poštára, status faktúry od poštára a poznámka k odoslaniu. Časť z nich presne zodpovedá tomu, čo Evka popisuje slovami. Odporúčam to s ňou a s Ivetou porovnať - je možné, že odpoveď na otázku 1 už existuje.
