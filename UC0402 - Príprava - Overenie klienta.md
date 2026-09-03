Zapracoval som všetky tri pripomienky. Najväčšia zmena je tretia - AT1 z UC0402 vypadáva, čím sa UC zužuje len na scenár prekliku.

---

# UC0402 - Príprava - Overenie klienta

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

**Vyriešené:**

- ~~Krajina vystavenia dokladu totožnosti~~ → **VYRIEŠENÉ:** pri prekliku z GATE sa hodnota prenesie z GATE v poli `idCardCountryCode`, pri manuálnom vypĺňaní teller vyberá z číselníka `ODS_SA.CCD_COUNTRY`.
- ~~Rodné číslo, dĺžka~~ → **VYRIEŠENÉ:** 9 až 10 miest.
- ~~Sekcia Iný subjekt (PO/FOP)~~ → **VYRIEŠENÉ:** v UC0402 sa nerieši, sekcia bola odstránená.
- ~~Vyhodnotenie whitelistu~~ → **PRESUNUTÉ:** rieši UC0404 - Príprava - Kontrola uskutočniteľnosti.
- ~~Vzťah osoby k účtu~~ → **VYRIEŠENÉ:** vyhodnocuje sa v UC0402 cez rozhranie ProductCBIAuthorizedSubjects.
- ~~Ktoré typy oprávnení sú relevantné~~ → **VYRIEŠENÉ (UC0431):** Majiteľ (M), Konateľ (K), Vkladateľ (V) a Disponent (D).
- ~~Dôsledok, ak osoba vzťah k účtu nemá~~ → **VYRIEŠENÉ (UC0431):** vyhodnotenie nie je blokujúce. Ak osoba vzťah k účtu nemá, ide o vklad treťou osobou a poplatok sa platí hneď pri vklade.
- ~~Verzia rozhrania ProductCBIAuthorizedSubjects~~ → **VYRIEŠENÉ:** používa sa v4. Verzia V5 prináša len identifikáciu kartového produktu podľa CardId, čo CashBox nepotrebuje.
- ~~Kto volá ProductCBIAuthorizedSubjects~~ → **ROZHODNUTÉ:** volá ho UC0402, jedenkrát pre celú transakciu. [POZNÁMKA: treba premietnuť do UC0431, ktorý vo vstupných a výstupných podmienkach uvádza, že službu volá on]
- ~~Poradie UC0401 a UC0402~~ → **ROZHODNUTÉ:** UC0401 beží ako prvý, UC0402 až po ňom. Nebežia súbežne.
- ~~Mapovanie údajov z prekliku GATE~~ → **VYRIEŠENÉ (Tomáš Macháček):** mapovanie je v UC0450 - Mapovacia tabuľka prekliku na vklad. Polia relevantné pre UC0402 sú v sekcii API.
- ~~Ako sa určí druh dokladu pri priamom vstupe~~ → **VYRIEŠENÉ (Feri):** druh dokladu vypĺňa teller vždy manuálne, ak transakcia nejde z prekliku. Systém nesmie použiť doklad uložený v GateGlobal, ale ten, ktorý osoba fyzicky predložila. Platí to rovnako pre klienta aj neklienta.
- ~~Prekryv s UC0407~~ → **VYRIEŠENÉ (Feri):** scenár priameho vstupu do CashBoxu patrí do UC0407 - Príprava - Overenie klienta - manuálne. Z UC0402 bol odstránený.

**Otvorené:**

**1. UC0431 treba opravit → autor UC0431**

```
Dohodli sme sa ze ProductCBIAuthorizedSubjects vola UC0402. UC0431 si v tom ale
protireci - v hlavnom toku pise ze vztah berie z UC0402 (co je spravne), ale vo
vystupnych podmienkach pise ze ho overil cez tu sluzbu a vo vstupnych podmienkach
vyzaduje jej dostupnost.

Ked sa to neopravi tak vyvojar moze naprogramovat volanie aj v UC0431 a sluzba
sa bude volat dvakrat.

Treba tam opravit:
- vstupne podmienky: namiesto dostupnosti sluzby dat ze je zname ci ma osoba
  vztah k uctu, dotiahnute v UC0402
- vystupne podmienky: namiesto "overeny cez sluzbu" dat "prevzaty z UC0402"
```

Neblokuje UC0402, blokuje UC0431. Dopad: mení sa UC0431.

**2. Ake opravnenia znamenaju Konatel, Vkladatel a Disponent → Matus Radusovsky**

```
UC0431 pracuje s hodnotami M, K, V, D. Majitela uz viem vyriesit - v podklade
k sluzbe je ze majitelske opravnenia sa daju rozpoznat podla priznaku
IS_OWNER_FLAG v ciselniku CCD_ROLE_TYPE, typicky su to MAJ1_PV, MAJ2_PV a CIFM_PV.

Ale konatel, vkladatel a disponent tam nie su. Podklad spomina este STAT_BODY
pre statutarny organ, co by mohol byt konatel, ale to je len moj odhad.

Vies mi poslat cely ciselnik CCD_ROLE_TYPE alebo aspon ktore kody patria
pod ktoru z tych styroch kategorii?
```

Blokuje dokončenie UC. Dopad: naplnenie konfiguračného zoznamu oprávnení.

**3. Preco nie je tabulka deposit v datovom modeli → vyvoj**

```
Pytal som sa kde sa ukladaju udaje pocas rozpracovanej transakcie a dostal som
odpoved ze je na to pomocna tabulka deposit. Pozrel som sa do toho a ta tabulka
naozaj existuje - bola aj v starsej verzii datoveho modelu so skoro rovnakymi
polami.

Lenze v aktualnom modeli cashbox_db.png uz nie je. Cize otazka nie je ci existuje,
ale preco zmizla z modelu. Je ten diagram nejako filtrovany, alebo v nom tabulka
chyba?

A este - plati ze sa tam ukladaju aj udaje o osobe z GateGlobal? Podla struktury
by to malo ist do stlpca depositor_info.
```

Blokuje dokončenie UC. Týka sa UC0401, UC0402 aj UC0403 spoločne. Dopad: mení sa krok 3 hlavného toku a sekcia Mapping.

**4. Mame v CashBoxe riesit black list, umrtie a exekuciu → Feri**

```
GateGlobal nam vracia aj priznaky ktore dnes nijako nepouzivame:

- BLACK_LIST_BLOCK - v podklade je popisany doslova ako "ma byt blokovana obsluha
  klienta". Cize je to priama instrukcia ktoru dnes ignorujeme.
- DEATH_FLAG a DEATH_DATE - umrtie klienta. Vklad alebo vyber na ucet zosnuleho
  ma pravne dosledky, ucet ide do dedicskeho konania.
- BLACK_LIST_FLAG - zaradenie na black list bez instrukcie blokovat.
- EXECUTION_FLAG - evidovana exekucia, mozno relevantne pri vyberoch.

Tie data uz mame, lebo stahujeme celu odpoved. Ide len o to ci ich mame
vyhodnocovat a s akym dosledkom - blokuje to transakciu, alebo len flagujeme
a riesi sa to inde?
```

Neblokuje. Dopad: prípadné rozšírenie hlavného toku o novú vetvu.

**5. Text hlasky pri nedostupnosti sluzby → Feri**

```
Ked vypadne GateGlobal, transakcia nemoze pokracovat a treba na to hlasku.
V katalogu taka nie je - pre CBS mame E026 "CBS je nedostupny, skuste to prosim
neskor", ale pre GateGlobal nic.

Navrhujem text: "Sluzba pre overenie klienta je nedostupna, skuste to prosim
neskor." Zamerne to pisem cez sluzbu a nie cez nazov GateGlobal, aby to pokrylo
aj tu druhu sluzbu na opravnenia a aby teller nemusel poznat nazvy internych
rozhrani.

Vies to schvalit a pridelit kod? Najvyssi pouzity je E056. Pozor, v UC0417 sme
uz navrhli E058, tak nech sa to nebije.
```

Blokuje dokončenie UC. Dopad: mení sa AT1.

**6. Tlacidla a povinne polia → Feri alebo Anka**

```
V UC nemam popisane cim teller uzavrie overenie osoby a co sa stane ked nevyplni
povinne polia. Vo Figme ma ten modal Overenie osoby tlacidla Zrusit a Potvrdit.

Navrhujem to iste co mame v UC0403 - Potvrdit je neaktivne kym nie su vyplnene
povinne polia a hlaska sa nezobrazuje. Sedi to?

A co sa stane ked da Zrusit - vrati sa na zaciatok transakcie alebo sa cela
transakcia zrusi?
```

Blokuje dokončenie UC, bez toho nie je testovateľný. Dopad: pribudnú kroky do hlavného toku a alternatívny tok pre zrušenie.

**7. Ako spoznat ze GateGlobal je nedostupny → Tomas Machacek**

```
Pri CBS to mame vyriesene cez tabulku as400_values kde je priznak is_online.
Pri GateGlobal nic take nemame a asi ani nedava zmysel budovat sledovanie stavu,
kedze je to synchronne rozhranie.

Navrhujem to riesit priamo z volania:
- vyprsi timeout volania (hodnota by bola konfiguracny parameter)
- alebo pride chybova odpoved mimo biznisovych chyb, teda mimo AUTH100

Sedi to tak? A aku hodnotu timeoutu mam napisat?
```

Blokuje dokončenie UC. Dopad: mení sa AT1.

**8. Aku hodnotu kanala posielame do GateGlobal → vyvoj**

```
Ta sluzba na opravnenia vyzaduje aby bol nas kanal registrovany v jej konfiguracii,
inak volanie skonci chybou ESBI102.

V UC0431 som nasiel ze pri volani FeeBe pouzivame konstantu Const.CASHBOX_GG_APPNAME
- to GG je GateGlobal, takze tu hodnotu uz asi mame v kode. Vies mi ju poslat?

A este jedna vec - v UC0431 je AppName priradena konstanta CHANEL_ID a ChannelID
zase konstanta APPNAME. Vyzera to na prehodene hodnoty, mozno sa na to pozri.
```

Blokuje implementáciu. Dopad: mení sa parameter requestu.

**9. Ci sa CIF a PID vobec zobrazuju → Feri**

```
V UC0450 mas napisane ze CIF a PID z prekliku pre vklad nechodia. V starsej verzii
UC0402 sme ich mali v tabulke poli, teraz tam uz nie su.

Cize zobrazujeme tellerovi CIF a PID vobec? Ak ano tak sa musia dotiahnut
z CustomerCBIDetail, nie z prekliku. Ak sa nezobrazuju tak je to v poriadku
tak ako to mam teraz.
```

Neblokuje. Dopad: prípadné doplnenie polí do Opisu obrazoviek.

**10. Kde sa riesia udaje o firme → Feri**

```
Sekciu Iny subjekt sme z UC0402 vyhodili, ale vo Figme je ta sekcia priamo
v modali Overenie osoby a UC701 sa na nu pri rozmieňani v mene firmy odvolava.

Cize dnes to nie je popisane nikde - v UC701 je len zmienka ze sa vyplna, ale
nie odkial sa udaje o firme beru ani podla coho sa firma vyhladava. Kde to ma byt?
```

Neblokuje UC0402, blokuje UC701. Dopad: mení sa odkaz v UC701.

**11. Co ked nie je preklik z GATE → Feri a Matej Pastucha**

```
AT1 sme presunuli do UC0407 a UC0402 teraz riesi len preklik z GATE. Cize ked
teller spusti transakciu priamo v CashBoxe, musi sa zavolat UC0407 namiesto UC0402.

Potrebujem vediet kto to rozhoduje - realizacne UC vkladu a vyberu, alebo je
nad tym nieco co si vyberie ktory z tych dvoch UC zavola? Aby to vyvojar vedel
naprogramovat.
```

Blokuje dokončenie UC. Dopad: mení sa Biznis zadanie a Vstupné podmienky.

### Biznis zadanie

UC slúži na overenie **osoby, ktorá je fyzicky prítomná pred tellerom** a vykonáva transakciu. Cieľom je získať kompletnú sadu identifikačných údajov o tejto osobe, vyhodnotiť, či je klientom banky, a zistiť jej vzťah k účtu, s ktorým sa transakcia realizuje.

**UC0402 rieši výlučne scenár, keď teller prišiel do CashBoxu preklikom z aplikácie GATE.** Ak preklik neprebehol a teller spustil transakciu priamo v CashBoxe, overenie osoby rieši **UC0407 - Príprava - Overenie klienta - manuálne**. [OTVORENY BOD: kto rozhoduje, ktorý z dvoch UC sa zavolá]

Spôsob získania údajov závisí od toho, či má osoba pridelené CCAID, teda či je klientom TB, alebo nie. Z toho vyplývajú **dva scenáre**, ktoré určujú, ktoré údaje prídu priamo z prekliku a ktoré si systém dotiahne z rozhrania GateGlobal (viď sekcia Opis obrazoviek + Validácie).

Vždy sa overuje **fyzická osoba**, tá, ktorá stojí pred tellerom a realizuje transakciu.

**Rozsah UC0402.** Do UC0402 patrí overenie osoby, získanie jej identifikačných údajov, vyhodnotenie príznaku Klient / Neklient a zistenie vzťahu osoby k účtu. Do UC0402 **nepatrí**:

| Čo | Rieši |
|---|---|
| Overenie osoby pri priamom vstupe do CashBoxu bez prekliku | UC0407 - Príprava - Overenie klienta - manuálne |
| Overovanie účtu, teda Brand, AccountStatus, AccountSubType, blokácie a disponibilný zostatok | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 (výbery) |
| Vyhodnotenie whitelistu | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 (výbery) |
| Výpočet poplatku za vklad | UC0431 - Poplatok za vklad - Stanovenie výšky |
| Dotiahnutie údajov o právnickej osobe alebo FOP | [OTVORENY BOD: v ktorom UC sa rieši] |

**Vzťah k susedným UC.** UC0402 rieši osobu, UC0401 - Príprava - Overenie čísla účtu rieši účet.

**Poradie je sekvenčné.** UC0401 beží ako prvý, UC0402 až po ňom. Dôvod: UC0402 volá rozhranie ProductCBIAuthorizedSubjects na zistenie vzťahu osoby k účtu a to vyžaduje na vstupe IBAN účtu, ktorý dotiahne UC0401. Pôvodná dohoda z review 10.8.2026 predpokladala súbežné spracovanie, tá už neplatí.

Nadväzujúci UC0404 - Príprava - Kontrola uskutočniteľnosti pracuje s dátami z oboch UC.

**Volanie rozhrania ProductCBIAuthorizedSubjects.** Rozhranie volá výlučne UC0402, jedenkrát pre celú transakciu. Nadväzujúce UC pracujú s uloženým príznakom vzťahu k účtu a rozhranie znova nevolajú. Platí to aj pre UC0431 - Poplatok za vklad - Stanovenie výšky.

**Princíp práce s dátami.** UC0402 si z rozhrania GateGlobal uloží **kompletne celú odpoveď**, nie len vybrané polia. Nadväzujúce UC potom rozhranie nevolajú znova a pracujú s uloženými dátami.

**Vyhodnotenie Klient / Neklient nie je blokujúce.** Ak osoba nie je klientom, tok pokračuje ďalej. Príznak je vstupom do vyhodnotenia limitov v UC0404 alebo UC0504.

**Vzťah osoby k účtu.** Systém zisťuje, či osoba prítomná pred tellerom má k účtu niektoré z oprávnení Majiteľ (M), Konateľ (K), Vkladateľ (V) alebo Disponent (D). Vyhodnotenie nie je blokujúce a slúži ako vstup pre nadväzujúce UC.

Pri vkladoch príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky:

| Vzťah k účtu | Dôsledok pre poplatok za vklad |
|---|---|
| Osoba vzťah k účtu nemá | Ide o vklad treťou osobou. Poplatok vypočítava CashBox volaním FeeBe a platí sa hneď pri vklade |
| Osoba vzťah k účtu má | Poplatok CashBox nevypočítava, zaúčtuje sa v procese kapitalizácie externým systémom |

**Numerické subjekty.** Reprezentant GateGlobal nespracúva numerické subjekty, teda subjekty s príznakom IS_NUMERIC_FLAG = '1'. Ide o klientov privátneho bankovníctva, ktorí nechcú byť menovaní. Tí si prostriedky prevedú na klientsky účet. CashBox pracuje výlučne s klientskymi účtami, takže toto obmedzenie nie je prekážkou.

### Aktéri

| Aktér | Čo v tomto UC robí |
|---|---|
| **Teller** | Skontroluje zobrazené údaje o osobe a v prípade potreby ich doplní alebo upraví. Vlastné zadávanie údajov v UC0402 neprebieha, keďže údaje prichádzajú z prekliku alebo z rozhrania GateGlobal |
| **Supervízor-Teller** | V UC0402 nevykonáva žiadnu akciu. Uvedený je preto, že transakciu môže realizovať aj používateľ s rolou Supervízor-Teller, ktorý v tom prípade vystupuje v úlohe tellera. Žiadne schvaľovanie ani override sa v UC0402 nevyžaduje |
| **Systém** | Vykonáva všetky ostatné kroky. Prevezme údaje z prekliku, volá rozhrania GateGlobal a ProductCBIAuthorizedSubjects, ukladá odpovede, vyhodnocuje príznak Klient / Neklient a vzťah k účtu, dopĺňa údaje podľa scenára a zobrazuje ich tellerovi |

---

## Vstupné podmienky

- Teller je prihlásený
- Pobočka je otvorená
- Pokladňa je otvorená
- UC je vyvolaný len počas klientskych transakcií, teda vkladov, výberov alebo rozmieňania
- **Teller prišiel do CashBoxu preklikom z aplikácie GATE.** Ak preklik neprebehol, overenie osoby rieši UC0407 - Príprava - Overenie klienta - manuálne a UC0402 sa nevyvoláva
- Prebehol UC0401 - Príprava - Overenie čísla účtu. Číslo účtu je overené a v kontexte transakcie je k dispozícii IBAN, ktorý UC0402 potrebuje na volanie rozhrania ProductCBIAuthorizedSubjects
- Kanál CashBox je registrovaný v konfigurácii služby ProductCBIAuthorizedSubjects [OTVORENY BOD: hodnota kanála]

---

## Hlavný tok

1. Systém prevezme z prekliku GATE identifikačné údaje osoby a údaje o transakcii podľa mapovania v UC0450 - Mapovacia tabuľka prekliku na vklad. Zoznam polí relevantných pre UC0402 je v sekcii API.
2. Systém vyhodnotí, či preklik obsahuje CCAID osoby, teda pole `ccaId`:
   - Ak `ccaId` je vyplnené, osoba je klientom TB. Osobné údaje v prekliku neprídu a systém si ich dotiahne z rozhrania GateGlobal. UC pokračuje krokom 3
   - Ak `ccaId` nie je vyplnené, osoba nie je klientom TB. Osobné údaje prídu priamo v prekliku a rozhranie GateGlobal sa nevolá. UC pokračuje krokom 5
3. Systém zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) s parametrom brand = 001 (TB) a s identifikátorom `ccaId` z prekliku a stiahne údaje o osobe:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT1**
4. Systém uloží kompletnú odpoveď rozhrania tak, aby bola k dispozícii nadväzujúcim UC bez opakovaného volania GateGlobal. [OTVORENY BOD: či sa údaje zapisujú do pomocnej tabuľky `deposit`, alebo sa držia len v pamäti aplikácie, viď sekcia Mapping]
5. Systém uloží príznak Klient / Neklient podľa toho, či bolo v prekliku vyplnené pole `ccaId`. Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok.
6. Systém zavolá rozhranie ProductCBIAuthorizedSubjects (v4) s identifikáciou účtu, teda s číslom účtu vo formáte IBAN prevzatým z UC0401 - Príprava - Overenie čísla účtu. Rozhranie volá výlučne UC0402, jedenkrát pre celú transakciu. Rozhranie vráti zoznam osôb, ktoré majú k danému účtu oprávnenie, spolu s typmi ich oprávnení:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT1**
7. Systém overí, či má osoba prítomná pred tellerom k účtu aspoň jedno oprávnenie zaradené v konfiguračnom zozname oprávnení zakladajúcich vzťah k účtu (viď sekcia API):
   - Ak také oprávnenie má, systém uloží príznak, že osoba má vzťah k účtu
   - Ak nemá žiadne, systém uloží príznak, že osoba vzťah k účtu nemá
   
   Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok. Príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky.
8. Systém doplní údaje o osobe podľa scenára, teda podľa matice v sekcii Opis obrazoviek + Validácie:
   - Údaje o doklade totožnosti sa v oboch scenároch preberajú z prekliku, teda z polí `idCardTypeCode`, `idCard` a `idCardCountryCode`
   - Osobné údaje sa pri osobe s CCAID preberajú z rozhrania GateGlobal, pri osobe bez CCAID z prekliku
9. Systém zobrazí údaje o osobe na obrazovke. Teller ich skontroluje a v prípade potreby doplní alebo upraví, všetky polia sú editovateľné. [OTVORENY BOD: akou akciou teller overenie uzatvára]
10. Systém ukončí UC s úspechom a odovzdá nadväzujúcim UC overené údaje o osobe, príznak Klient / Neklient a príznak vzťahu k účtu. Tok pokračuje v UC, z ktorého bol UC0402 vyvolaný, teda v príslušnom UC vkladu, výberu alebo rozmieňania.

---

## Alternatívny tok

### AT1 - Rozhranie nie je dostupné

**Spúšťač:** systém nedostane odpoveď z rozhrania GateGlobal alebo ProductCBIAuthorizedSubjects.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 3 alebo krok 6.

1. Systém zistí nedostupnosť rozhrania podľa vypršania timeoutu volania alebo podľa chybovej odpovede mimo biznisových chýb. [OTVORENY BOD: potvrdiť mechanizmus a hodnotu timeoutu]
2. GateGlobal je jediný zdroj údajov o osobe. SubReg sa pre CashBox nepoužíva, CBS neobsahuje doklady totožnosti. Náhradný zdroj údajov neexistuje.
3. Systém zobrazí tellerovi chybovú hlášku o nedostupnosti služby. Návrh textu: "Sluzba pre overenie klienta je nedostupna, skuste to prosim neskor." [OTVORENY BOD: hláška nie je v katalógu, ide o návrh]
4. Osobu nie je možné overiť, transakcia nemôže pokračovať a UC končí neúspešne.

---

## Diagram tokov

[OTVORENY BOD: diagram bude doplnený]

---

## Výstupné podmienky

**Úspech:**
- Osoba fyzicky prítomná pred tellerom je overená a jej identifikačné údaje sú kompletné
- Systém má uloženú kompletnú odpoveď z rozhrania GateGlobal, ak bolo volané
- Systém má uložený príznak, či je osoba klientom banky
- Systém má uložený príznak vzťahu osoby k účtu
- Systém má uložený typ subjektu, teda fyzická alebo právnická osoba, ktorý využíva UC0431 - Poplatok za vklad - Stanovenie výšky pri určení sadzby poplatku
- Nadväzujúce UC môžu pracovať s uloženými dátami bez opakovaného volania rozhraní
- Tok pokračuje v UC, z ktorého bol UC0402 vyvolaný

**Zlyhanie:**

| Druh zlyhania | Čo sa nezapísalo | Kde sa teller nachádza |
|---|---|---|
| GateGlobal je nedostupný (AT1) | Nič, údaje o osobe sa nedotiahli | Transakcia nemôže pokračovať |
| ProductCBIAuthorizedSubjects je nedostupný (AT1) | Príznak vzťahu k účtu sa neuložil | Transakcia nemôže pokračovať |

---

## Opis obrazoviek + Validácie

**Legenda k spôsobu získania údaja:**
- **Preklik** - údaj príde priamo v prekliku z aplikácie GATE, viď UC0450 - Mapovacia tabuľka prekliku na vklad
- **GateGlobal** - údaj sa dotiahne z rozhrania GateGlobal.CustomerCBIDetail (v6.2)

M = Mandatory (Y = povinné, N = nepovinné), E = Editable (Y = editovateľné, N = read-only)

### Sekcia osoba (fyzická osoba prítomná pred tellerom)

| Názov | Dátový typ | Validácia | M | E | Popis | Pole v prekliku | Osoba s CCAID | Osoba bez CCAID |
|---|---|---|---|---|---|---|---|---|
| Priezvisko | Text | - | Y | Y | Priezvisko osoby | `lastname` | GateGlobal | Preklik |
| Meno | Text | - | Y | Y | Meno osoby | `firstName` | GateGlobal | Preklik |
| Titul | Text | - | N | Y | Titul pred menom | `title` | GateGlobal | Preklik |
| Rodné číslo | Text | 9 až 10 miest | N | Y | Rodné číslo osoby | `birthNumber` | GateGlobal | Preklik |
| Dátum narodenia | Date (DD/MM/YYYY) | - | Y | Y | Dátum narodenia | `birthDate` | GateGlobal | Preklik |
| Druh dokladu | Dropdown | Hodnoty z číselníka `id_card_type` | Y | Y | Druh dokladu totožnosti | `idCardTypeCode` | Preklik | Preklik |
| Číslo dokladu | Text | - | Y | Y | Číslo dokladu totožnosti | `idCard` | Preklik | Preklik |
| Krajina vystavenia dokladu | Dropdown | Hodnoty z číselníka `ODS_SA.CCD_COUNTRY` | Y | Y | Krajina vystavenia dokladu | `idCardCountryCode` | Preklik | Preklik |

**Pozn. k dokladu totožnosti.** Údaje o doklade sa vždy vzťahujú na doklad, ktorý osoba **fyzicky predložila**, nie na doklad uložený v GateGlobal. Tie sa môžu líšiť, napríklad ak osoba predloží pas a banka má v systéme občiansky preukaz. Pri prekliku z GATE tieto údaje zadal teller v aplikácii GATE a prídu v prekliku v poliach `idCardTypeCode`, `idCard` a `idCardCountryCode`. Systém ich z GateGlobal neodvodzuje.

**Pozn. k osobným údajom.** Pri osobe s CCAID osobné údaje v prekliku neprídu a systém si ich dotiahne z rozhrania GateGlobal volaním CustomerCBIDetail. Pri osobe bez CCAID prídu priamo v prekliku a rozhranie sa nevolá.

[OTVORENY BOD: správanie tlačidiel a vynucovanie povinných polí]

---

## API

### Rozhrania

| Rozhranie | Verzia | Účel | Použitie v UC0402 |
|---|---|---|---|
| GateGlobal.CustomerCBIDetail | v6.2 | Detail klienta, reprezentant GG_SUBJ_DATA | Dotiahnutie osobných údajov, ak má osoba CCAID. Volá sa v kroku 3 |
| GateGlobal.ProductCBIAuthorizedSubjects | v4 | Zoznam oprávnených osôb k produktu spolu s typmi oprávnení | Zistenie vzťahu osoby k účtu v krokoch 6 a 7 |

**Poznámka k verzii ProductCBIAuthorizedSubjects.** Používa sa verzia v4, ktorá je v Interface Specifications uvedená ako používaná. Verzia V5 prináša oproti v4 len možnosť identifikácie **kartového produktu** podľa jednoznačného identifikátora karty. CashBox sa pýta na účet cez IBAN, nie na kartu, takže novinku V5 nepotrebuje. Verzia V5 je navyše v podklade uvedená v stave "Pripravovaný".

**Poznámka k rozhraniu CustomerCBIFind.** V UC0402 sa nepoužíva. Slúži na vyhľadanie klienta podľa identifikátorov a využíva ho UC0407 - Príprava - Overenie klienta - manuálne.

**SubReg (Subjektový register) sa pre CashBox nepoužíva.** GateGlobal je jediný zdroj údajov o osobe. CBS sa ako zdroj údajov o osobe nepoužíva, neobsahuje doklady totožnosti (potvrdil Feri).

### Údaje prichádzajúce z prekliku GATE

Zdroj: UC0450 - Mapovacia tabuľka prekliku na vklad (Tomáš Macháček). Nasledujúca tabuľka uvádza polia relevantné pre UC0402. Kompletné mapovanie vrátane polí pre nadväzujúce UC je v UC0450.

| Pole v prekliku | Údaj | Poznámka |
|---|---|---|
| `lastname` | Priezvisko | Príde, len ak osoba nemá CCAID. Pri CCAID sa volá CustomerCBIDetail |
| `firstName` | Meno | Príde, len ak osoba nemá CCAID |
| `title` | Titul | Príde, len ak osoba nemá CCAID |
| `birthNumber` | Rodné číslo | Príde, len ak osoba nemá CCAID |
| `birthDate` | Dátum narodenia | Príde, len ak osoba nemá CCAID |
| `idCardTypeCode` | Druh dokladu | Príde vždy |
| `idCard` | Číslo dokladu | Príde vždy |
| `idCardCountryCode` | Krajina vystavenia dokladu | Príde vždy |
| `ccaId` | CCAID predkladateľa, teda osoby pred tellerom | Podľa jeho vyplnenia sa určuje príznak Klient / Neklient |
| `ownerCcaId` | CCAID vlastníka účtu | Pri transakcii v zastúpení je iné ako `ccaId` |
| `account` | Číslo účtu vo formáte BBAN | Spracúva UC0401 - Príprava - Overenie čísla účtu |
| `isEmployee` | Príznak zamestnanca TB | Nie je pole na formulári. Využíva sa pri kurzoch |

**Polia, ktoré z prekliku pre vklad neprídu:** CIF, PID, mena, hotovosť, vklad, variabilný symbol, špecifický symbol, konštantný symbol. [OTVORENY BOD: či sa CIF a PID v UC0402 vôbec zobrazujú]

**Poznámka k `ownerCcaId`.** Pole obsahuje CCAID vlastníka účtu a pri transakcii v zastúpení sa líši od `ccaId`. Vzťah osoby k účtu sa napriek tomu vyhodnocuje cez rozhranie ProductCBIAuthorizedSubjects, pretože porovnanie `ccaId` a `ownerCcaId` by pokrylo len majiteľa, nie disponenta ani ďalšie oprávnenia.

### Číselníky

| Zdroj | Tabuľka | Použitie v UC0402 |
|---|---|---|
| DB CashBox | `id_card_type` (`code` varchar(20), `description` varchar(20)) | Rozbaľovací zoznam Druh dokladu |
| ODS | `ODS_SA.CCD_COUNTRY` | Rozbaľovací zoznam Krajina vystavenia dokladu |
| GateGlobal | `CCD_ROLE_TYPE` | Typy oprávnení k produktu, vyhodnotenie vzťahu k účtu |

### Vyhodnotenie vzťahu k účtu

Systém vyhodnotí, že osoba má vzťah k účtu, ak má k nemu aspoň jedno oprávnenie zaradené v **konfiguračnom zozname oprávnení zakladajúcich vzťah k účtu**. Zoznam sa napĺňa hodnotami z číselníka `CCD_ROLE_TYPE` a zodpovedá kategóriám Majiteľ (M), Konateľ (K), Vkladateľ (V) a Disponent (D) podľa UC0431 - Poplatok za vklad - Stanovenie výšky.

**Majiteľské oprávnenia** systém rozpozná podľa aktívneho príznaku `IS_OWNER_FLAG` v číselníku `CCD_ROLE_TYPE`. Podľa podkladu k rozhraniu ide typicky o oprávnenia MAJ1_PV, MAJ2_PV a CIFM_PV.

**Ďalšie typy oprávnení uvedené v podklade:**

| Kód | Význam |
|---|---|
| MAJ1_PV, MAJ2_PV, CIFM_PV | Majiteľské oprávnenia, príznak IS_OWNER_FLAG |
| DRK1_PK, DRK2_PK | Držiteľské oprávnenia k platobnej karte |
| ZKZ1_PV | Zákonný zástupca |
| STAT_BODY | Štatutárny orgán |
| KUV_OWNER | Konečný užívateľ výhod |

[OTVORENY BOD: zaradenie oprávnení pre kategórie Konateľ, Vkladateľ a Disponent. Podklad uvádza kód STAT_BODY pre štatutárny orgán, čo pravdepodobne zodpovedá kategórii Konateľ, potvrdené to však nie je]

| Výsledok | Kritérium | Príznak pre nadväzujúce UC |
|---|---|---|
| Osoba má vzťah k účtu | Osoba je v zozname oprávnených osôb s oprávnením zo zoznamu | Má vzťah k účtu |
| Osoba nemá vzťah k účtu | Osoba nie je v zozname, alebo služba vráti chybu AUTH100 | Nemá vzťah k účtu, ide o vklad treťou osobou |

### GateGlobal - parametre requestu

| Parameter | Hodnota z CashBoxu | Poznámka |
|---|---|---|
| brand | 001 (TB) | Reprezentant sa počíta zo subjektových obrazov daného brandu. 001 = TB, 002 = RB |
| Vyhľadávací identifikátor | `ccaId` z prekliku | |

**Upozornenie k identifikátorom:** podľa podkladu sa identifikátor supersubjektu (ccdIdTb) a super-supersubjektu (ccxId) môžu v čase meniť spárovaním alebo rozpárovaním subjektov. Nie je vhodné sa na ne priamo odkazovať. CashBox pracuje s ccaIdTb.

### GateGlobal - obmedzenia reprezentanta

- Numerické subjekty, teda IS_NUMERIC_FLAG = '1', reprezentant nespracúva
- Zdrojom hodnôt je aktívny CCA obraz supersubjektu, teda ACTIVE_FLAG = 1. Ak má supersubjekt viac aktívnych CCA obrazov, berie sa ten s najmenším ID
- Algoritmus výpočtu reprezentanta zohľadňuje GDPR stav obrazov supersubjektu, obrazy s nižšou prioritou sú z výpočtu vylúčené

### ProductCBIAuthorizedSubjects - parametre requestu

| Objekt | Parameter | Hodnota z CashBoxu | Poznámka |
|---|---|---|---|
| RequestCVO | channel | [OTVORENY BOD: hodnota kanála. V CashBoxe existuje konštanta CASHBOX_GG_APPNAME] | Kanál musí byť registrovaný v konfigurácii služby, inak volanie skončí chybou ESBI102 |
| RequestCVO | timestamp | Časová pečiatka správy | |
| RequestCVO | version | v4 | |
| RequestCVO | subversion | 0 | |
| ProductAuthorizedSubjectsRequestCVO | brand | 001 (GATE TB) | 001 = GATE TB, 002 = SUN |
| ProductAuthorizedSubjectsRequestCVO | productNumber/Account | IBAN účtu | IBAN je dotiahnutý v UC0401 - Príprava - Overenie čísla účtu |
| ProductAuthorizedSubjectsRequestCVO | derivedRolesFlag | Neposiela sa | Odvodené oprávnenia CashBox nepotrebuje. Aktívny príznak pri produkte z brandu RB (002) vedie k chybe ESBI105 |
| ProductAuthorizedSubjectsRequestCVO | subjectIdentification | `ccaId` osoby z prekliku | |

### ProductCBIAuthorizedSubjects - obmedzenia služby

Podľa podkladu:
- Služba nepublikuje oprávnenia, ktoré sú v konfigurovateľnom zozname zakázaných k publikácii (príznak RESTR_PUBLIC_FLAG v číselníku CCD_ROLE_TYPE), sú viazané na subjektový obraz s neaktívnym GDPR stavom, alebo sú exspirované
- Služba vracia informácie k všetkým produktom v databáze bez ohľadu na ich stav a aktívnosť
- Služba neoveruje a nekontroluje stav PIDov
- Informácie sú získavané výhradne z databázy, služba nevolá žiadny OIF
- Ak k produktu nie sú žiadne oprávnené osoby, služba vracia prázdny zoznam

### Chybové kódy rozhraní

| Kód | Význam | Použitie v UC0402 |
|---|---|---|
| AUTH100 | Pre zadaný identifikátor neexistuje oprávnená osoba | Vyhodnotenie vzťahu k účtu, krok 7 |
| ESBI102 | Zadaný kanál nie je súčasťou konfigurácie | Predpoklad volania služby, vstupná podmienka |

Ostatné chybové kódy nie sú predmetom tejto špecifikácie, platia podľa dokumentácie rozhraní.

### Nadväznosť

- **Predchádza:** UC0401 - Príprava - Overenie čísla účtu, ktorý overí číslo účtu a dotiahne IBAN. UC0402 beží až po ňom, nie súbežne
- **Vstup:** preklik z aplikácie GATE a IBAN účtu z UC0401
- **Alternatíva k UC0402:** UC0407 - Príprava - Overenie klienta - manuálne, ktorý sa vyvolá, ak preklik z GATE neprebehol
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery). Príznak vzťahu k účtu a typ subjektu využíva UC0431 - Poplatok za vklad - Stanovenie výšky
- **Súvisiaci podklad:** UC0450 - Mapovacia tabuľka prekliku na vklad
- **Analogický UC:** UC0502 - Príprava - Overenie klienta (výbery), zatiaľ nevytvorený

---

## Mapping

Zdroj: podklad *GG Dátový reprezentant FO PO, verzia 25.5*, rozhranie CustomerCBIDetail v6.2, reprezentant GG_SUBJ_DATA.

### Uloženie údajov počas rozpracovanej transakcie

UC0402 potrebuje odpoveď rozhrania sprístupniť nadväzujúcim UC bez opakovaného volania GateGlobal. Podľa vývojára slúži na priebežné údaje transakcie pomocná tabuľka `deposit`, do ktorej sa postupne ukladajú údaje o klientovi, suma, kurz a ďalšie hodnoty. Údaje o osobe by patrili do stĺpca `depositor_info`.

Tabuľka `deposit` bola v staršej verzii dátového modelu CashBox so zhodnou štruktúrou. V aktuálnom modeli `cashbox_db.png` uvedená nie je.

[OTVORENY BOD: prečo tabuľka nie je v aktuálnom dátovom modeli a či sa do nej ukladajú aj údaje o osobe]

### Osobné údaje - tabuľka CCD_PERSON (GS_PERSON)

Platí pre scenár, keď má osoba CCAID a systém volá CustomerCBIDetail.

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
|---|---|---|---|
| Meno | FIRSTNAME / firstName | FIRSTNAME | Ak je PERSON_NAME prázdne, meno sa vyskladá ako SURNAME + medzera + FIRSTNAME |
| Priezvisko | SURNAME / surname | SURNAME | |
| Titul | TITLE_INF / title | TITLE_INF | Titul pred menom. Reprezentant eviduje aj titleBehind, v CashBoxe sa nepoužíva |
| Rodné číslo | PERSONAL_ID / personalId | PERSONAL_ID | Hodnota z aktívneho CCA obrazu subjektu z GATE TB |
| Dátum narodenia | BIRTH_DATE / birthDate | BIRTH_DATE | |

### Doklady totožnosti

Údaje o doklade totožnosti sa v UC0402 **preberajú výlučne z prekliku** z polí `idCardTypeCode`, `idCard` a `idCardCountryCode`. Vzťahujú sa na doklad, ktorý osoba fyzicky predložila.

Reprezentant GateGlobal síce obsahuje polia OP_NUMBER, PAS_NUMBER, ID_NUMBER a PNP_NUMBER z tabuľky CCD_DOCUMENT, ale **CashBox ich nepoužíva**. Doklad uložený v GateGlobal sa môže líšiť od dokladu, ktorý osoba práve predložila, a pre overenie totožnosti je rozhodujúci ten predložený.

### Identifikátory - tabuľka CCD_SUBJECT (GS_SUBJECT) a CCD_IAAP

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
|---|---|---|---|
| CCAID | CCAID_TB / ccaIdTb | ID | Identifikátor CCA subjektu v TB. V UC0402 sa preberá z prekliku, pole `ccaId` |
| PID | PID_TB / pidTb | P_NUMBER | Z prekliku pre vklad nepríde. [OTVORENY BOD: či sa v UC0402 zobrazuje] |
| CIF | MAIN_CIF / mainCif | MAIN_CIF | Z prekliku pre vklad nepríde. [OTVORENY BOD: či sa v UC0402 zobrazuje] |
| Typ subjektu | sType / C_TYPE | C_TYPE | P = fyzická osoba, C = právnická osoba, E = zamestnanec TB, B = banka, A = anonym, I = pobočky a bankomaty, R, Q. Využíva UC0431 na rozlíšenie FO a PO pri určení sadzby poplatku |

### Ďalšie dostupné atribúty, ktoré CashBox zatiaľ nevyužíva

| Atribút | Popis | Možné využitie |
|---|---|---|
| INDUSTRY_CODE | Industry Code, číselník CCD_INDUSTRY | Zdroj pre rozlíšenie živnostníka v UC0416 - Realizácia - Halierové vyrovnanie |
| BLACK_LIST_FLAG | Subjekt je na black liste, hodnota 1 = áno | [OTVORENY BOD] |
| BLACK_LIST_BLOCK | Má byť blokovaná obsluha klienta, hodnota 1 = áno | [OTVORENY BOD] |
| DEATH_FLAG, DEATH_DATE | Úmrtie klienta | [OTVORENY BOD] |
| EXECUTION_FLAG | Evidovaná exekúcia | [OTVORENY BOD] |
| CITIZENSHIP | Krajina trvalého pobytu (FO) alebo sídla (PO) | |
| tbConsistencyFlag | Príznak konzistentnosti klienta | |

**Mimo rozsah UC0402:**
- Overenie osoby pri priamom vstupe bez prekliku, rieši UC0407 - Príprava - Overenie klienta - manuálne
- Overovanie účtu, rieši UC0404 alebo UC0504
- Vyhodnotenie whitelistu, rieši UC0404 alebo UC0504
- Výpočet poplatku za vklad, rieši UC0431 - Poplatok za vklad - Stanovenie výšky
- Údaje o právnickej osobe alebo FOP, [OTVORENY BOD: v ktorom UC sa riešia]

---

## Poznámky pre teba - nekopírovať do UC

### Čo sa zapracovaním zmenilo

| Pripomienka | Dopad |
|---|---|
| Mapping z prekliku (Feri, Tomáš) | Nová sekcia v API s poľami z UC0450, prepísaná matica v Opise obrazoviek na názvy polí, prepísaný krok 1 |
| Druh dokladu vždy manuálne bez prekliku (Feri) | **Odstránená celá logika odvodenia z OP_NUMBER, PAS_NUMBER, ID_NUMBER a PNP_NUMBER.** Doklad ide vždy z prekliku, alebo ho vypĺňa teller v UC0407 |
| AT1 patrí do UC0407 (Feri) | AT1 odstránené, AT2 prečíslované na AT1, UC0402 rieši len preklik, matica sa zúžila zo štyroch scenárov na dva |

### Zmena, ktorú UC0450 priniesla navyše

Mapovacia tabuľka odhalila logiku, ktorú sme v UC nemali: **pri osobe s CCAID osobné údaje v prekliku vôbec neprídu** a musí sa volať CustomerCBIDetail. Pri osobe bez CCAID prídu v prekliku a rozhranie sa nevolá vôbec.

To znamená, že hlavný tok sa vetví hneď v kroku 2 a rozhranie GateGlobal sa v jednom z dvoch scenárov **nevolá**. Predtým UC predpokladal, že sa volá vždy. Prepísal som to.

### Čo treba odovzdať Matejovi pre UC0407

Presunom AT1 sa z UC0402 stráca časť, ktorá bola už rozpracovaná. Nech sa nezabudne, patrí do UC0407:

- Volanie GateGlobal.CustomerCBIFind (v6.0) na vyhľadanie osoby a následné volanie CustomerCBIDetail, keďže Find nevracia doklady
- Scenáre: osoba s CCAID, kde sa dotiahnu osobné údaje a doklad vypĺňa teller, a osoba bez CCAID, kde teller vypĺňa všetko
- Otvorená otázka, podľa ktorých identifikátorov teller vyhľadáva a ako sa postupuje pri cudzincoch bez rodného čísla
- Pravidlo od Feriho, že doklad vypĺňa teller vždy manuálne a použije ten, ktorý osoba fyzicky predložila

Návrh správy:

```
Ahoj Matej, Feri rozhodol ze scenar priameho vstupu do CashBoxu ide do tvojho
UC0407 a z UC0402 som ho vyhodil. Nech to nezacinas od nuly, mal som to uz
rozpracovane, tak ti to poslem:

- vola sa CustomerCBIFind na vyhladanie osoby a potom este CustomerCBIDetail,
  lebo Find nevracia doklady totoznosti
- dva scenare: osoba s CCAID (dotiahnu sa osobne udaje, doklad vyplna teller)
  a osoba bez CCAID (teller vyplna vsetko)
- Feri potvrdil ze doklad vyplna teller vzdy rucne a zapisuje ten co mu osoba
  fyzicky predlozila, nie ten co mame ulozeny v GateGlobal
- ostava otvorene podla coho teller vyhladava, lebo cudzinci nemaju rodne cislo

A este jedna vec - ked UC0402 riesi len preklik a tvoj UC0407 len priamy vstup,
kto rozhoduje ktory z nich sa zavola? Realizacne UC, alebo je nad tym nieco ine?
```
