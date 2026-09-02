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

- ~~Krajina vystavenia dokladu totožnosti~~ → **VYRIEŠENÉ:** pri prekliku z GATE sa hodnota prenesie z GATE, pri manuálnom vypĺňaní teller vyberá z číselníka `ODS_SA.CCD_COUNTRY`.
- ~~Rodné číslo, dĺžka~~ → **VYRIEŠENÉ:** 9 až 10 miest.
- ~~Sekcia Iný subjekt (PO/FOP)~~ → **VYRIEŠENÉ:** v UC0402 sa nerieši, sekcia bola odstránená.
- ~~Vyhodnotenie whitelistu~~ → **PRESUNUTÉ:** rieši UC0404 - Príprava - Kontrola uskutočniteľnosti.
- ~~Vzťah osoby k účtu~~ → **VYRIEŠENÉ:** vyhodnocuje sa v UC0402 cez rozhranie ProductCBIAuthorizedSubjects.
- ~~Ktoré typy oprávnení sú relevantné~~ → **VYRIEŠENÉ (UC0431):** za vzťah k účtu sa považujú oprávnenia Majiteľ (M), Konateľ (K), Vkladateľ (V) a Disponent (D).
- ~~Dôsledok, ak osoba vzťah k účtu nemá~~ → **VYRIEŠENÉ (UC0431):** vyhodnotenie nie je blokujúce. Ak osoba vzťah k účtu nemá, ide o vklad treťou osobou a poplatok sa platí hneď pri vklade. Ak vzťah má, poplatok sa zaúčtuje v kapitalizácii.
- ~~Verzia rozhrania ProductCBIAuthorizedSubjects~~ → **VYRIEŠENÉ:** používa sa **v4**, nie V5. Verzia V5 prináša oproti v4 len možnosť identifikácie kartového produktu podľa CardId, čo CashBox nepotrebuje, keďže sa pýta na účet cez IBAN. Verzia v4 je zároveň v Interface Specifications uvedená ako používaná, kým V5 je v stave "Pripravovaný".
- ~~Zdroj druhu dokladu pri prekliku z GATE~~ → **VYRIEŠENÉ (Feriho matica):** pri prekliku z GATE prichádza druh dokladu z výsledku vyhľadania v GATE. Otvorený zostáva len formát hodnôt.
- ~~Mapovanie oprávnenia Majiteľ~~ → **VYRIEŠENÉ (podklad k rozhraniu):** majiteľské oprávnenia systém rozpozná podľa aktívneho príznaku `IS_OWNER_FLAG` v číselníku `CCD_ROLE_TYPE`. Typicky ide o oprávnenia MAJ1_PV, MAJ2_PV a CIFM_PV.

**Otvorené:**

**1. Kto vlastne vola ProductCBIAuthorizedSubjects → Matus Radusovsky**

```
V UC0402 mam ze tu sluzbu volame my a priznak vztahu k uctu len odovzdame dalej.
Lenze UC0431 si v tom protireci - v hlavnom toku pise ze vztah overi na zaklade
informacii z UC0402, ale vo vystupnych podmienkach ze ho overil cez tu sluzbu,
a vo vstupnych podmienkach vyzaduje jej dostupnost.

Cize kto ju vola? Ja by som ju nechal v UC0402 aby sa nevolala dvakrat, ako sme
sa dohodli na review. Navyse UC0402 bezi pri vsetkych transakciach, kym UC0431
len pri vkladoch.

Ale ma to jeden dosledok - tá sluzba potrebuje IBAN uctu z UC0401, takze UC0402
uz nemoze bezat subezne s UC0401 ale az po nom. To je zmena oproti tomu co sme
sa dohodli, tak to chcem mat potvrdene.
```

Blokuje dokončenie UC. Dopad: prípadné odstránenie krokov 5 a 6 z hlavného toku a zmena poradia UC0401 a UC0402.

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
polami (iban, deposit_currency, denominations_in, depositor_info a dalsie).

Lenze v aktualnom modeli cashbox_db.png uz nie je. Cize otazka nie je ci
existuje, ale preco zmizla z modelu. Je ten diagram nejako filtrovany, alebo
v nom tabulka chyba?

A este - plati ze sa tam ukladaju aj udaje o osobe z GateGlobal? Podla struktury
by to malo ist do stlpca depositor_info.
```

Blokuje dokončenie UC. Týka sa UC0401, UC0402 aj UC0403 spoločne. Dopad: mení sa krok 3 hlavného toku a sekcia Mapping.

**4. Ako teller najde cudzinca ktory nema rodne cislo → Feri**

```
V UC mam ze teller pri priamom vstupe zada rodne cislo a podla neho sa osoba
vyhlada. Matus upozornil ze to znie obmedzujuco a ze cudzinci rodne cislo nemaju.

Podla Interface Specifications ta sluzba CustomerCBIFind podporuje aj ICO, PID
a dalsie identifikatory. V tvojej matici je navyse rodne cislo oznacene ako
nepovinne, takze to vyzera ze sa s tym pocita.

Cize ako ma teller najst cudzinca ktory je klientom TB a ma CCAID, ale nema
slovenske rodne cislo? Podla coho ho vyhlada?
```

Blokuje dokončenie UC. Dopad: mení sa AT1 a Opis obrazoviek.

**5. Mame v CashBoxe riesit black list, umrtie a exekuciu → Feri**

```
GateGlobal nam vracia aj priznaky ktore dnes nijako nepouzivame:

- BLACK_LIST_BLOCK - v podklade je popisany doslova ako "ma byt blokovana
  obsluha klienta". Cize je to priama instrukcia ktoru dnes ignorujeme.
- DEATH_FLAG a DEATH_DATE - umrtie klienta. Vklad alebo vyber na ucet zosnuleho
  ma pravne dosledky, ucet ide do dedicskeho konania.
- BLACK_LIST_FLAG - zaradenie na black list bez instrukcie blokovat.
- EXECUTION_FLAG - evidovana exekucia, mozno relevantne pri vyberoch.

Tie data uz mame, lebo stahujeme celu odpoved. Ide len o to ci ich mame
vyhodnocovat a s akym dosledkom - blokuje to transakciu, alebo len flagujeme
a riesi sa to inde?
```

Neblokuje. Dopad: prípadné rozšírenie hlavného toku o novú vetvu.

**6. Text hlasky pri nedostupnosti sluzby → Feri**

```
Ked vypadne GateGlobal, transakcia nemoze pokracovat a treba na to hlasku.
V katalogu taka nie je - pre CBS mame E026 "CBS je nedostupny, skuste to
prosim neskor", ale pre GateGlobal nic.

Navrhujem text: "Sluzba pre overenie klienta je nedostupna, skuste to prosim
neskor." Zamerne to pisem cez sluzbu a nie cez nazov GateGlobal, aby to
pokrylo aj tu druhu sluzbu na opravnenia a aby teller nemusel poznat nazvy
internych rozhrani.

Vies to schvalit a pridelit kod? Najvyssi pouzity je E056. Pozor, v UC0417
sme uz navrhli E058, tak nech sa to nebije.
```

Blokuje dokončenie UC. Dopad: mení sa AT2.

**7. Tlacidla a povinne polia → Feri alebo Anka**

```
V UC nemam popisane cim teller uzavrie overenie osoby a co sa stane ked
nevyplni povinne polia. Vo Figme ma ten modal Overenie osoby tlacidla
Zrusit a Potvrdit.

Navrhujem to iste co mame v UC0403 - Potvrdit je neaktivne kym nie su
vyplnene povinne polia a hlaska sa nezobrazuje. Sedi to?

A co sa stane ked da Zrusit - vrati sa na zaciatok transakcie alebo sa
cela transakcia zrusi?
```

Blokuje dokončenie UC, bez toho nie je testovateľný. Dopad: pribudnú kroky do hlavného toku a alternatívny tok pre zrušenie.

**8. Ako spoznat ze GateGlobal je nedostupny → Tomas Machacek**

```
Pri CBS to mame vyriesene cez tabulku as400_values kde je priznak is_online.
Pri GateGlobal nic take nemame a asi ani nedava zmysel budovat sledovanie
stavu, kedze je to synchronne rozhranie.

Navrhujem to riesit priamo z volania:
- vyprsi timeout volania (hodnota by bola konfiguracny parameter)
- alebo pride chybova odpoved mimo biznisovych chyb, teda mimo AUTH100

Sedi to tak? A akú hodnotu timeoutu mam napisat?
```

Blokuje dokončenie UC. Dopad: mení sa AT2.

**9. Aku hodnotu kanala posielame do GateGlobal → vyvoj**

```
Ta sluzba na opravnenia vyzaduje aby bol nas kanal registrovany v jej
konfiguracii, inak volanie skonci chybou ESBI102.

V UC0431 som nasiel ze pri volani FeeBe pouzivame konstantu
Const.CASHBOX_GG_APPNAME - to GG je GateGlobal, takze tu hodnotu uz asi mame
v kode. Vies mi ju poslat?

A este jedna vec - v UC0431 je AppName priradena konstanta CHANEL_ID a
ChannelID zase konstanta APPNAME. Vyzera to na prehodene hodnoty, mozno
sa na to pozri.
```

Blokuje implementáciu. Dopad: mení sa parameter requestu.

**10. Format druhu dokladu z GATE → Peto Magyar**

```
Feri v matici potvrdil ze pri prekliku z GATE nam druh dokladu prichadza z GATE.
Potrebujem este vediet v akom formate - aké konkretne hodnoty GATE posiela,
aby sa dali namapovat na nas ciselnik id_card_type.

Viem ze GATE integraciu riesime az pred LIVE, tak to nesuri, len nech na to
nezabudneme.
```

Neblokuje, GATE integrácia sa rieši pred LIVE. Dopad: mení sa krok 7 hlavného toku.

**11. Kde sa riesia udaje o firme → Feri**

```
Sekciu Iny subjekt sme z UC0402 vyhodili, ale vo Figme je ta sekcia priamo
v modali Overenie osoby a UC701 sa na nu pri rozmieňani v mene firmy odvolava.

Cize dnes to nie je popisane nikde - v UC701 je len zmienka ze sa vyplna, ale
nie odkial sa udaje o firme beru ani podla coho sa firma vyhladava.
Kde to ma byt?
```

Neblokuje UC0402, blokuje UC701. Dopad: mení sa odkaz v UC701.

**12. UC0402 verzus UC0407 → Matej Pastucha**

```
V zozname UC je UC0407 Overenie klienta - manualne pre nedostupny GATE a mas
ho ako TODO. To co mam v UC0402 ako AT1, teda priamy vstup do CashBoxu bez
prekliku, je vlastne to iste.

Kedze UC0407 este nie je napisany, dohodnime sa teraz nech nevznikne duplicita.
Ma to zostat u mna v UC0402 ako alternativny tok, alebo to vyhodim a bude to
cely tvoj UC0407?

Este poznamka - UC0407 nie je v subore cashbox-uc-strom.md, len v status
subore. Mozno by sa mal doplnit aj do stromu.
```

Neblokuje. Dopad: prípadné odstránenie AT1 z UC0402.

### Biznis zadanie

UC slúži na overenie **osoby, ktorá je fyzicky prítomná pred tellerom** a vykonáva transakciu. Cieľom je získať kompletnú sadu identifikačných údajov o tejto osobe, vyhodnotiť, či je klientom banky, a zistiť jej vzťah k účtu, s ktorým sa transakcia realizuje.

Spôsob získania údajov závisí od dvoch faktorov:
1. Či má osoba pridelené CCAID, teda či je klientom TB, alebo nie
2. Ako sa používateľ dostal do CashBoxu, teda preklikom z aplikácie GATE alebo priamym vstupom do CashBoxu

Z toho vyplývajú **štyri scenáre**, ktoré určujú, ktoré údaje sa dotiahnu automaticky a ktoré musí teller vyplniť manuálne (viď sekcia Opis obrazoviek + Validácie).

Vždy sa overuje **fyzická osoba**, tá, ktorá stojí pred tellerom a realizuje transakciu.

**Rozsah UC0402.** Do UC0402 patrí overenie osoby, získanie jej identifikačných údajov, vyhodnotenie príznaku Klient / Neklient a zistenie vzťahu osoby k účtu. Do UC0402 **nepatrí**:

| Čo | Rieši |
|---|---|
| Overovanie účtu, teda Brand, AccountStatus, AccountSubType, blokácie a disponibilný zostatok | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 - Príprava - Kontrola uskutočniteľnosti (výbery) |
| Vyhodnotenie whitelistu | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 - Príprava - Kontrola uskutočniteľnosti (výbery) |
| Výpočet poplatku za vklad | UC0431 - Poplatok za vklad - Stanovenie výšky |
| Dotiahnutie údajov o právnickej osobe alebo FOP | [OTVORENY BOD: v ktorom UC sa rieši] |

**Vzťah k susedným UC.** UC0402 rieši osobu, UC0401 - Príprava - Overenie čísla účtu rieši účet. Nadväzujúci UC0404 - Príprava - Kontrola uskutočniteľnosti pracuje s dátami z oboch. [OTVORENY BOD: kontrola vzťahu k účtu vyžaduje IBAN z UC0401, čo mení pôvodne dohodnuté súbežné poradie UC0401 a UC0402]

**Princíp práce s dátami.** UC0402 si z rozhrania GateGlobal uloží **kompletne celú odpoveď**, nie len vybrané polia. Nadväzujúce UC potom rozhranie nevolajú znova a pracujú s uloženými dátami. Tým sa zabraňuje duplicitným dopytom.

**Vyhodnotenie Klient / Neklient nie je blokujúce.** Ak osoba nie je klientom, tok pokračuje ďalej. Príznak je vstupom do vyhodnotenia limitov v UC0404 alebo UC0504.

**Vzťah osoby k účtu.** Systém zisťuje, či osoba prítomná pred tellerom má k účtu niektoré z oprávnení Majiteľ (M), Konateľ (K), Vkladateľ (V) alebo Disponent (D). Vyhodnotenie nie je blokujúce a slúži ako vstup pre nadväzujúce UC.

Pri vkladoch príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky:

| Vzťah k účtu | Dôsledok pre poplatok za vklad |
|---|---|
| Osoba vzťah k účtu nemá | Ide o vklad treťou osobou. Poplatok vypočítava CashBox volaním FeeBe a platí sa hneď pri vklade |
| Osoba vzťah k účtu má | Poplatok CashBox nevypočítava, zaúčtuje sa v procese kapitalizácie externým systémom |

**Numerické subjekty.** Reprezentant GateGlobal nespracúva numerické subjekty, teda subjekty s príznakom IS_NUMERIC_FLAG = '1'. Ide o klientov privátneho bankovníctva, ktorí nechcú byť menovaní. Tí si pri vklade alebo výbere prevedú prostriedky na klientsky účet. CashBox pracuje výlučne s klientskymi účtami, takže toto obmedzenie nie je prekážkou.

### Aktéri

| Aktér | Čo v tomto UC robí |
|---|---|
| **Teller** | Pri prekliku z GATE skontroluje zobrazené údaje o osobe a v prípade potreby ich doplní alebo upraví. Pri priamom vstupe do CashBoxu zadáva identifikátor osoby, spúšťa vyhľadanie a dopĺňa údaje, ktoré sa nedotiahli automaticky |
| **Supervízor-Teller** | V UC0402 nevykonáva žiadnu akciu. Uvedený je preto, že transakciu môže realizovať aj používateľ s rolou Supervízor-Teller, ktorý v tom prípade vystupuje v úlohe tellera. Žiadne schvaľovanie ani override sa v UC0402 nevyžaduje |
| **Systém** | Vykonáva všetky ostatné kroky. Volá rozhrania GateGlobal a ProductCBIAuthorizedSubjects, ukladá odpovede, vyhodnocuje príznak Klient / Neklient a vzťah k účtu, určuje druh dokladu, dopĺňa údaje podľa scenára a zobrazuje ich tellerovi |

---

## Vstupné podmienky

- Teller je prihlásený
- Pobočka je otvorená
- Pokladňa je otvorená
- UC je vyvolaný len počas klientskych transakcií, teda vkladov, výberov alebo rozmieňania
- Je známy IBAN účtu, s ktorým sa transakcia realizuje, dotiahnutý v UC0401 - Príprava - Overenie čísla účtu. Údaj je potrebný na zistenie vzťahu osoby k účtu. [OTVORENY BOD: väzba na UC0401 a poradie UC]
- Kanál CashBox je registrovaný v konfigurácii služby ProductCBIAuthorizedSubjects [OTVORENY BOD: hodnota kanála]

---

## Hlavný tok

**Vstup do UC.** UC0402 má dva vstupné body, ktoré sa navzájom vylučujú:

| Vstupný bod | Kedy nastáva | Kde je popísaný |
|---|---|---|
| Preklik z aplikácie GATE | Teller prišiel do CashBoxu preklikom z GATE, údaje o osobe prídu v prekliku | Hlavný tok |
| Priamy vstup do CashBoxu | Teller spustil transakciu priamo v CashBoxe bez prekliku, údaje o osobe nie sú k dispozícii | AT1 |

1. Systém vyhodnotí, akým spôsobom bol UC0402 vyvolaný:
   - Ak bol vyvolaný preklikom z aplikácie GATE, systém prevezme z prekliku identifikačné údaje osoby a výsledok vyhľadania v GATE. UC pokračuje nasledujúcim krokom
   - Ak bol vyvolaný priamym vstupom do CashBoxu, tok pokračuje **AT1**
2. Systém zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) s parametrom brand = 001 (TB) a stiahne údaje o osobe. Vyhľadávanie prebehne podľa identifikátora, ktorý GATE odovzdal, teda CCAID alebo rodné číslo:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
3. Systém uloží kompletnú odpoveď rozhrania tak, aby bola k dispozícii nadväzujúcim UC bez opakovaného volania GateGlobal. [OTVORENY BOD: či sa údaje zapisujú do pomocnej tabuľky `deposit`, alebo sa držia len v pamäti aplikácie, viď sekcia Mapping]
4. Systém vyhodnotí, či má osoba pridelené CCAID, teda atribút ccaIdTb:
   - Ak je atribút vyplnený, osoba je klientom TB
   - Ak supersubjekt neobsahuje CCA subjekt, atribút nie je vyplnený a osoba nie je klientom TB
   
   Systém výsledok uloží ako príznak pre nadväzujúce UC. Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok.
5. Systém zavolá rozhranie ProductCBIAuthorizedSubjects (v4) s identifikáciou účtu, teda s číslom účtu vo formáte IBAN prevzatým z UC0401 - Príprava - Overenie čísla účtu. Rozhranie vráti zoznam osôb, ktoré majú k danému účtu oprávnenie, spolu s typmi ich oprávnení:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
6. Systém overí, či má osoba prítomná pred tellerom k účtu aspoň jedno oprávnenie zaradené v konfiguračnom zozname oprávnení zakladajúcich vzťah k účtu (viď sekcia API):
   - Ak také oprávnenie má, systém uloží príznak, že osoba má vzťah k účtu
   - Ak nemá žiadne, systém uloží príznak, že osoba vzťah k účtu nemá
   
   Vyhodnotenie nie je blokujúce, tok pokračuje bez ohľadu na výsledok. Príznak využíva UC0431 - Poplatok za vklad - Stanovenie výšky.
7. Systém určí druh dokladu totožnosti:
   - Ak bol UC0402 vyvolaný preklikom z aplikácie GATE, druh dokladu sa prevezme z výsledku vyhľadania v GATE
   - Ak bol vyvolaný priamym vstupom do CashBoxu a osoba má CCAID, systém určí druh dokladu podľa toho, ktoré z polí OP_NUMBER, PAS_NUMBER, ID_NUMBER alebo PNP_NUMBER je v odpovedi GateGlobal vyplnené
   - Ak bol vyvolaný priamym vstupom a osoba CCAID nemá, druh dokladu vyplní teller manuálne
   
   Zistený druh systém namapuje na príslušný kód z číselníka `id_card_type`.
8. Systém doplní ostatné údaje o osobe podľa scenára, teda podľa matice v sekcii Opis obrazoviek + Validácie. Údaje, ktoré sa nedotiahnu automaticky, ostávajú prázdne na manuálne vyplnenie tellerom.
9. Systém zobrazí údaje o osobe na obrazovke. Teller ich skontroluje a v prípade potreby doplní alebo upraví, všetky polia sú editovateľné. [OTVORENY BOD: akou akciou teller overenie uzatvára]
10. Systém ukončí UC s úspechom a odovzdá nadväzujúcim UC overené údaje o osobe, príznak Klient / Neklient a príznak vzťahu k účtu. Tok pokračuje v UC, z ktorého bol UC0402 vyvolaný, teda v príslušnom UC vkladu, výberu alebo rozmieňania.

---

## Alternatívny tok

### AT1 - Priamy vstup do CashBoxu (bez prekliku z GATE)

**Spúšťač:** UC0402 bol vyvolaný priamo v CashBoxe, údaje o osobe nie sú vopred k dispozícii.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 1, nahrádza aj krok 2.

1. Systém zobrazí tellerovi formulár na zadanie údajov o osobe.
2. Teller manuálne zadá identifikátor osoby, ktorá je pred ním, a zvolí Vyhľadať.
3. Systém zavolá rozhranie GateGlobal.CustomerCBIFind (v6.0) s parametrom brand = 001 (TB) a vyhľadá fyzickú osobu podľa zadaného identifikátora. Vždy sa vyhľadáva fyzická osoba, teda tá, ktorá vykonáva transakciu:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
   
   [OTVORENY BOD: podľa ktorých identifikátorov môže teller vyhľadávať. Rozhranie podporuje rodné číslo, IČO, PID a ďalšie. Rodné číslo nepokrýva cudzincov]
4. Systém zo získaného identifikátora, teda ccaIdTb alebo ccxId, zavolá rozhranie GateGlobal.CustomerCBIDetail (v6.2) a dotiahne kompletné údaje vrátane dokladov totožnosti. Volanie Detail je nutné, pretože CustomerCBIFind nevracia atribúty z tabuľky CCD_DOCUMENT:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
5. Systém vyhodnotí výsledok a doplní údaje podľa scenára:
   - Ak má osoba CCAID, z GateGlobal sa dotiahnu priezvisko, meno, titul, dátum narodenia, CCAID, CIF a PID. Rodné číslo a údaje o doklade totožnosti musí teller vyplniť manuálne
   - Ak osoba CCAID nemá, teller vyplní manuálne všetky údaje o osobe. Polia CCAID, CIF a PID sa nepoužívajú
6. Tok pokračuje krokom 3 hlavného toku.

### AT2 - Rozhranie nie je dostupné

**Spúšťač:** systém nedostane odpoveď z rozhrania GateGlobal alebo ProductCBIAuthorizedSubjects.
**Platí pre:** vklady, výbery aj rozmieňanie.
**Krok v hlavnom toku:** krok 2 alebo krok 5 hlavného toku, krok 3 alebo krok 4 v AT1.

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
- Systém má uloženú kompletnú odpoveď z rozhrania GateGlobal
- Systém má uložený príznak, či je osoba klientom banky
- Systém má uložený príznak vzťahu osoby k účtu
- Systém má uložený typ subjektu, teda fyzická alebo právnická osoba, ktorý využíva UC0431 - Poplatok za vklad - Stanovenie výšky pri určení sadzby poplatku
- Nadväzujúce UC môžu pracovať s uloženými dátami bez opakovaného volania rozhraní
- Tok pokračuje v UC, z ktorého bol UC0402 vyvolaný

**Zlyhanie:**

| Druh zlyhania | Čo sa nezapísalo | Kde sa teller nachádza |
|---|---|---|
| GateGlobal je nedostupný (AT2) | Nič, údaje o osobe sa nedotiahli | Transakcia nemôže pokračovať |
| ProductCBIAuthorizedSubjects je nedostupný (AT2) | Príznak vzťahu k účtu sa neuložil | Transakcia nemôže pokračovať |

---

## Opis obrazoviek + Validácie

**Legenda k spôsobu získania údaja:**
- **GateGlobal** - údaj sa dotiahne automaticky z rozhrania GateGlobal.CustomerCBIDetail (v6.2)
- **Vyhľadanie v Gate** - údaj sa prenesie z výsledku vyhľadania v aplikácii GATE
- **Manuálne** - údaj musí vyplniť teller ručne, nedopĺňa sa automaticky
- **-** - pole sa pre daný scenár nepoužíva

M = Mandatory (Y = povinné, N = nepovinné), E = Editable (Y = editovateľné, N = read-only)

### Sekcia osoba (fyzická osoba prítomná pred tellerom)

| Názov | Dátový typ | Validácia | M | E | Popis | Osoba s CCAID - preklik z Gate | Osoba s CCAID - priamo v CashBoxe | Osoba bez CCAID - preklik z Gate | Osoba bez CCAID - priamo v CashBoxe |
|---|---|---|---|---|---|---|---|---|---|
| Priezvisko | Text | - | Y | Y | Priezvisko osoby | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Meno | Text | - | Y | Y | Meno osoby | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Titul | Text | - | N | Y | Titul pred menom | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Rodné číslo | Text | 9 až 10 miest | N | Y | Vyhľadávací identifikátor fyzickej osoby | GateGlobal | Manuálne | Vyhľadanie v Gate | Manuálne |
| Dátum narodenia | Date (DD/MM/YYYY) | - | Y | Y | Dátum narodenia | GateGlobal | GateGlobal | Vyhľadanie v Gate | Manuálne |
| Druh dokladu | Dropdown | Hodnoty z číselníka `id_card_type` | Y | Y | Druh dokladu totožnosti, teda občiansky preukaz, pas, povolenie na pobyt a ďalšie podľa číselníka | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |
| Číslo dokladu | Text | - | Y | Y | Číslo dokladu totožnosti | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |
| Krajina vystavenia dokladu | Dropdown | Hodnoty z číselníka `ODS_SA.CCD_COUNTRY` | Y | Y | Krajina vystavenia dokladu totožnosti | Vyhľadanie v Gate | Manuálne | Vyhľadanie v Gate | Manuálne |

**Pozn. k druhu dokladu:** hodnoty rozbaľovacieho zoznamu sa načítavajú z číselníka `id_card_type` v DB CashBox, stĺpce code a description, nie sú v aplikácii pevne zadané. Pri prekliku z GATE sa druh dokladu prenáša z GATE. Pri priamom vstupe systém určí druh dokladu podľa toho, ktoré z polí OP_NUMBER, PAS_NUMBER, ID_NUMBER alebo PNP_NUMBER je v odpovedi GateGlobal vyplnené.

**Pozn. ku krajine vystavenia dokladu:** pri prekliku z aplikácie GATE sa hodnota prenesie z GATE. Pri manuálnom vypĺňaní teller vyberá z rozbaľovacieho zoznamu naplneného z číselníka `ODS_SA.CCD_COUNTRY`.

### Zhrnutie pravidiel

- Pri osobách s CCAID sa základné osobné údaje, teda meno, priezvisko, titul a dátum narodenia, získavajú automaticky z rozhrania GateGlobal
- Údaje o doklade totožnosti sa pri prekliku z GATE preberajú z výsledku vyhľadania v GATE. Pri priamom vstupe do CashBoxu ich musí teller vyplniť manuálne
- Pri osobách bez CCAID sa údaje pri prekliku z GATE preberajú z vyhľadania v GATE. Pri priamom vstupe do CashBoxu sa všetky údaje vypĺňajú manuálne

[OTVORENY BOD: správanie tlačidiel a vynucovanie povinných polí]

---

## API

### Rozhrania

| Rozhranie | Verzia | Účel | Použitie v UC0402 |
|---|---|---|---|
| GateGlobal.CustomerCBIDetail | v6.2 | Detail klienta, reprezentant GG_SUBJ_DATA | Primárny a jediný zdroj údajov o osobe. Jediný zdroj údajov o dokladoch totožnosti |
| GateGlobal.CustomerCBIFind | v6.0 | Vyhľadanie klienta podľa identifikátorov, teda rodného čísla, IČO, PID a ďalších | Použije sa pri manuálnom vyhľadaní v AT1. **Nevracia doklady totožnosti** |
| GateGlobal.ProductCBIAuthorizedSubjects | **v4** | Zoznam oprávnených osôb k produktu spolu s typmi oprávnení | Zistenie vzťahu osoby k účtu v krokoch 5 a 6 hlavného toku |

**Poznámka k verzii ProductCBIAuthorizedSubjects.** Používa sa verzia **v4**, ktorá je v Interface Specifications uvedená ako používaná. Verzia V5 prináša oproti v4 len možnosť identifikácie **kartového produktu** podľa jednoznačného identifikátora karty. CashBox sa pýta na účet cez IBAN, nie na kartu, takže novinku V5 nepotrebuje. Verzia V5 je navyše v podklade uvedená v stave "Pripravovaný".

**SubReg (Subjektový register) sa pre CashBox nepoužíva.** GateGlobal je jediný zdroj údajov o osobe. CBS sa ako zdroj údajov o osobe nepoužíva, neobsahuje doklady totožnosti (potvrdil Feri).

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

[OTVORENY BOD: zaradenie oprávnení pre kategórie Konateľ, Vkladateľ a Disponent. Podklad uvádza kód STAT_BODY pre štatutárny orgán, čo pravdepodobne zodpovedá kategórii Konateľ, potvrdené to však nie je. Kompletný číselník CCD_ROLE_TYPE nie je v projektových podkladoch]

| Výsledok | Kritérium | Príznak pre nadväzujúce UC |
|---|---|---|
| Osoba má vzťah k účtu | Osoba je v zozname oprávnených osôb s oprávnením zo zoznamu | Má vzťah k účtu |
| Osoba nemá vzťah k účtu | Osoba nie je v zozname, alebo služba vráti chybu AUTH100 | Nemá vzťah k účtu, ide o vklad treťou osobou |

### GateGlobal - parametre requestu

| Parameter | Hodnota z CashBoxu | Poznámka |
|---|---|---|
| brand | 001 (TB) | Reprezentant sa počíta zo subjektových obrazov daného brandu. 001 = TB, 002 = RB. Ak sa brand nepošle, vráti sa super-supersubjekt naprieč brandmi |
| Vyhľadávací identifikátor | CCAID alebo rodné číslo | Podľa scenára |

### GateGlobal - vyhľadávanie

| Identifikátor | Atribút | Poznámka |
|---|---|---|
| CCAID | ccaIdTb (TB), ccaIdRb (SB) | ccaIdTb je CCA subjekt s najnižším ID v rámci supersubjektu. Ak supersubjekt neobsahuje CCA subjekt, atribút nie je vyplnený, čo zodpovedá scenáru osoby bez CCAID |
| Rodné číslo | personalId | Vždy sa vyhľadáva fyzická osoba pred tellerom |

**Upozornenie k identifikátorom:** podľa podkladu sa identifikátor supersubjektu (ccdIdTb) a super-supersubjektu (ccxId) môžu v čase meniť spárovaním alebo rozpárovaním subjektov. Nie je vhodné sa na ne priamo odkazovať. CashBox pracuje s ccaIdTb.

### GateGlobal - obmedzenia reprezentanta

- Numerické subjekty, teda IS_NUMERIC_FLAG = '1', reprezentant nespracúva. Nie je to pre CashBox prekážka, pracuje sa len s klientskymi účtami
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
| ProductAuthorizedSubjectsRequestCVO | productNumber/Account | IBAN účtu | Vyhľadávanie produktu typu účet prebieha cez IBAN. IBAN je dotiahnutý v UC0401 - Príprava - Overenie čísla účtu |
| ProductAuthorizedSubjectsRequestCVO | derivedRolesFlag | Neposiela sa | Odvodené oprávnenia CashBox nepotrebuje. Aktívny príznak pri produkte z brandu RB (002) vedie k chybe ESBI105 |
| ProductAuthorizedSubjectsRequestCVO | subjectIdentification | ccaId osoby | Identifikácia je možná aj cez ccdId, ccxId, cif, ico, personalID alebo pid |

### ProductCBIAuthorizedSubjects - obmedzenia služby

Podľa podkladu:
- Služba nepublikuje oprávnenia, ktoré sú v konfigurovateľnom zozname zakázaných k publikácii (príznak RESTR_PUBLIC_FLAG v číselníku CCD_ROLE_TYPE), sú viazané na subjektový obraz s neaktívnym GDPR stavom, alebo sú exspirované
- Služba vracia informácie k všetkým produktom v databáze bez ohľadu na ich stav a aktívnosť
- Služba neoveruje a nekontroluje stav PIDov
- Informácie sú získavané výhradne z databázy, služba nevolá žiadny OIF
- Ak k produktu nie sú žiadne oprávnené osoby, služba vracia prázdny zoznam

### Chybové kódy rozhraní

Chybové kódy rozhraní nie sú predmetom tejto špecifikácie, platia podľa dokumentácie rozhraní. Výnimkou sú:

| Kód | Význam | Použitie v UC0402 |
|---|---|---|
| AUTH100 | Pre zadaný identifikátor neexistuje oprávnená osoba | Vyhodnotenie vzťahu k účtu, krok 6 |
| ESBI102 | Zadaný kanál nie je súčasťou konfigurácie | Predpoklad volania služby, vstupná podmienka |

### Nadväznosť

- **Vstup:** preklik z aplikácie GATE alebo priamy vstup v CashBoxe. IBAN účtu z UC0401 - Príprava - Overenie čísla účtu
- **Súbežne:** UC0401 - Príprava - Overenie čísla účtu. [OTVORENY BOD: súbežnosť je dotknutá potrebou IBAN pre kontrolu vzťahu k účtu]
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery). Príznak vzťahu k účtu a typ subjektu využíva UC0431 - Poplatok za vklad - Stanovenie výšky
- **Analogický UC:** UC0502 - Príprava - Overenie klienta (výbery), zatiaľ nevytvorený
- **Možný prekryv:** UC0407 - Príprava - Overenie klienta - manuálne (nedostupný GATE), vlastník Matej Pastucha, stav TODO

---

## Mapping

Zdroj: podklad *GG Dátový reprezentant FO PO, verzia 25.5*, rozhranie CustomerCBIDetail v6.2, reprezentant GG_SUBJ_DATA.

### Uloženie údajov počas rozpracovanej transakcie

UC0402 potrebuje odpoveď rozhrania sprístupniť nadväzujúcim UC bez opakovaného volania GateGlobal. Podľa vývojára slúži na priebežné údaje transakcie pomocná tabuľka `deposit`, do ktorej sa postupne ukladajú údaje o klientovi, suma, kurz a ďalšie hodnoty. Údaje o osobe by patrili do stĺpca `depositor_info`.

Tabuľka `deposit` bola v staršej verzii dátového modelu CashBox so zhodnou štruktúrou. V aktuálnom modeli `cashbox_db.png` uvedená nie je.

[OTVORENY BOD: prečo tabuľka nie je v aktuálnom dátovom modeli a či sa do nej ukladajú aj údaje o osobe. Týka sa UC0401, UC0402 aj UC0403 spoločne]

### Osobné údaje - tabuľka CCD_PERSON (GS_PERSON)

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
|---|---|---|---|
| Meno | FIRSTNAME / firstName | FIRSTNAME | Ak je PERSON_NAME prázdne, meno sa vyskladá ako SURNAME + medzera + FIRSTNAME |
| Priezvisko | SURNAME / surname | SURNAME | |
| Titul | TITLE_INF / title | TITLE_INF | Titul pred menom. Reprezentant eviduje aj titleBehind, teda titul za menom, v CashBoxe sa zatiaľ nepoužíva |
| Rodné číslo | PERSONAL_ID / personalId | PERSONAL_ID | Hodnota z aktívneho CCA obrazu subjektu z GATE TB |
| Dátum narodenia | BIRTH_DATE / birthDate | BIRTH_DATE | |

### Doklady totožnosti - tabuľka CCD_DOCUMENT (GS_DOCUMENT)

Reprezentant nemá jedno pole pre druh dokladu. Má samostatné pole pre každý typ dokladu a druh sa určí podľa toho, ktoré je vyplnené. Zistený druh sa namapuje na kód z číselníka `id_card_type`.

| Druh dokladu | Atribút reprezentanta | Zdrojový atribút | Pravidlo výberu |
|---|---|---|---|
| Občiansky preukaz | OP_NUMBER | DOC_NUMBER | Prvý vyplnený podľa stavu v poradí PLA, NED, NEP, POD, EXP, AUK, kde ACTIVE_FLAG = '1', len doklad viazaný na CCA subjekt. Ak supersubjekt nemá aktívny doklad daného typu, pole je prázdne |
| Pas | PAS_NUMBER | DOC_NUMBER | Rovnaké pravidlo, typ dokladu PAS |
| ID karta | ID_NUMBER | DOC_NUMBER | Rovnaké pravidlo, typ dokladu ID a všetky ID_% |
| Povolenie na pobyt | PNP_NUMBER | DOC_NUMBER | Rovnaké pravidlo, typ dokladu PNP |

Poznámka: stav dokladu NULL sa vyhodnocuje ako Nedefinovaný (NED).

**Krajina vystavenia dokladu:** reprezentant GateGlobal toto pole neobsahuje. Hodnota sa pri prekliku prenesie z aplikácie GATE, pri manuálnom vypĺňaní teller vyberá z číselníka `ODS_SA.CCD_COUNTRY`.

### Identifikátory - tabuľka CCD_SUBJECT (GS_SUBJECT) a CCD_IAAP

| Pole v UC | Atribút reprezentanta | Zdrojový atribút | Poznámka |
|---|---|---|---|
| CCAID | CCAID_TB / ccaIdTb | ID | Identifikátor CCA subjektu v TB. Ak supersubjekt neobsahuje CCA subjekt, nie je vyplnený |
| PID | PID_TB / pidTb | P_NUMBER | PID v stave aktívny, teda STATUS = 'PI0'. Pri viacerých sa vráti text MANY_PIDS |
| CIF | MAIN_CIF / mainCif | MAIN_CIF | Pri viacerých rôznych MAIN_CIF subjektoch sa vráti text MANY_MAINCIFS |
| Typ subjektu | sType / C_TYPE | C_TYPE | P = fyzická osoba, C = právnická osoba, E = zamestnanec TB, B = banka, A = anonym, I = pobočky a bankomaty, R, Q. Využíva UC0431 - Poplatok za vklad - Stanovenie výšky na rozlíšenie FO a PO pri určení sadzby poplatku (OWNER_TYPE) |

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
- Overovanie účtu, rieši UC0404 - Príprava - Kontrola uskutočniteľnosti alebo UC0504
- Vyhodnotenie whitelistu, rieši UC0404 - Príprava - Kontrola uskutočniteľnosti alebo UC0504
- Výpočet poplatku za vklad, rieši UC0431 - Poplatok za vklad - Stanovenie výšky
- Údaje o právnickej osobe alebo FOP, [OTVORENY BOD: v ktorom UC sa riešia]
