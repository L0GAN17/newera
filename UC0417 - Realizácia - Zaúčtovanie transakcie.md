Zapracoval som všetkých dvanásť pripomienok. Tri z nich menia UC podstatne - najmä Feriho poznámka o valute a o JIRA tickete. Excel navyše umožnil doplniť kompletné mapovanie request codes, čím odpadá jeden otvorený bod.

---

# UC0417 - Realizácia - Zaúčtovanie transakcie

## Obsah

- Obsah
- Info
  - Otvorené otázky
  - Biznis zadanie
  - Aktéri
- Vstupné podmienky
- Hlavný tok
- Alternatívny tok
- Biznis pravidlá
- Diagram tokov
- Výstupné podmienky
- Opis obrazoviek + Validácie
- API
- Mapping

---

## Info

### Otvorené otázky

**Vyriešené:**

- ~~Poradie odosielania pokynov~~ → **VYRIEŠENÉ (Feri):** hlavná transakcia, halierové vyrovnanie, poplatok, poplatok za vklad alebo výber mincí. Podrobnosti v BP02.
- ~~Status na úrovni pokynu alebo transakcie~~ → **VYRIEŠENÉ (Feri):** status sa eviduje na úrovni jednotlivého pokynu.
- ~~Uloženie statusu v databáze~~ → **VYRIEŠENÉ (Feri):** stĺpec `status` v tabuľke `transaction_journal`.
- ~~Interval opakovaného spracovania~~ → **VYRIEŠENÉ (Feri):** každých 15 minút počas celého dňa, kým sa pokyn nezaúčtuje alebo neuzavrie. Maximálny počet pokusov sa nestanovuje.
- ~~Klasifikácia chýb z CBS~~ → **VYRIEŠENÉ (Feri):** delenie na prechodné a trvalé chyby, neznámy kód sa vyhodnotí ako trvalá chyba.
- ~~Forma incidentu~~ → **VYRIEŠENÉ (Feri):** systém **nevytvára** JIRA ticket automaticky. Zobrazí odkaz na JIRA a odošle okamžitý email. Podrobnosti v BP07.
- ~~Mapovanie request codes na typy transakcií~~ → **VYRIEŠENÉ:** kompletné mapovanie je v sekcii API a vychádza zo súboru Transakcie.xlsx, záložka Potrebné trans.kódy. Ide o tabuľku, nie o mapovanie per jednotlivý prípad.
- ~~Čo sú naviazané transakčné kódy~~ → **UPRESNENÉ:** ide o poplatok za vklad alebo výber mincí, request code 28 a 29. Konkrétne kódy sú v BP01 a v sekcii API.

**Otvorené:**

**1. Co s pokynom ktory sa nezauctuje do konca dna → Feri**

```
Napisal si ze nemozeme zauctovat na druhy den, lebo podla VOP je valuta datum
prevzatia financnych prostriedkov, a ze to treba spracovat alebo riesit
nasledujuci den cez manualnu opravu.

Potrebujem to rozpisat, lebo teraz mam v UC ze sa to jednoducho prenesie do
dalsieho dna, co uz neplati:

1. Co presne je "manualna oprava"? Kto ju robi - teller, supervizor, alebo
   niekto z prevadzky?
2. Robi sa to v CashBoxe alebo mimo neho?
3. Ako sa zabezpeci spravna valuta, teda datum kedy klient peniaze realne
   priniesol? Posiela sa nejake pole s valutou, alebo to riesi ten kto robi
   opravu rucne?
4. A co ma system spravit na konci dna s pokynom ktory je stale v statuse T -
   necha ho tak a len na to upozorni, alebo ho nejako uzavrie?
```

Blokuje dokončenie UC. Dopad: mení sa BP04, AT1, AT5 a Výstupné podmienky.

**2. Kde najdeme ciselnik chybovych kodov CBS → Tomas Machacek**

```
V UC mam ze chyby z CBS delime na prechodne (skusime znova) a trvale (ide to
na rucne riesenie) a ze zaradenie konkretnych kodov bude v konfiguracnom
ciselniku.

Vyvojar sa pyta kde ten ciselnik najde. Odpoved je ze zatial neexistuje a
treba ho vytvorit. Ale potrebujem k tomu zoznam chybovych kodov ktore APSRV20
vracia, aby sme ich vedeli zaradit.

Vies mi ten zoznam poslat, alebo aspon povedat kde je zdokumentovany?
```

Blokuje implementáciu. Dopad: naplnenie číselníka.

**3. Nove hlasky do katalogu → Feri**

```
V UC0417 mi chybaju styri hlasky ktore nie su v schvalenom katalogu. Vyvojar
upozornil ze si ich nemame vymyslat, lebo potom budu chybat pri testoch.

Navrhujem tieto texty, potrebujem ich schvalit a pridelit kody:

1. Uspesny vyber: "Vydajte hotovost klientovi."
2. Transakcia caka na spracovanie: "Transakcia caka na spracovanie. Mozete
   pokracovat dalsou transakciou."
3. Chyba pri zauctovani: "Transakciu sa nepodarilo zauctovat. Chyba bola
   zaznamenana a odovzdana na riesenie."
4. Suhrnna hlaska pri hromadnom vklade - ak ju chceme, viz otazka nizsie

Najvyssi pouzity kod v katalogu je E056. Pozor, v UC0402 sme navrhli hlasku
o nedostupnosti sluzby, tak nech sa cislovanie nebije.
```

Blokuje dokončenie UC. Dopad: mení sa sekcia Opis obrazoviek.

**4. Suhrnna hlaska pri hromadnom vklade → Feri**

```
Pri hromadnom vklade sa dnes suhrnna hlaska nezobrazuje a teller vidi jednotlive
vklady az v zurnale. Ty si pisal ze ju mozeme vymysliet ak je potrebna.

Ma ju CashBox mat? Ak ano tak navrhujem text v style "Spracovanych vkladov: X.
Caka na spracovanie: Y. Neuspesnych: Z."

Alebo to nechame tak ako je?
```

Neblokuje. Dopad: prípadné doplnenie hlášky do AT6.

**5. Ake polia ma obsahovat email pri incidente → Feri**

```
Dohodli sme sa ze namiesto automatickeho JIRA ticketu posleme okamzity mail
a zobrazime odkaz na JIRA.

Potrebujem vediet:
1. Komu ten mail ide? Konkretna adresa alebo skupina?
2. Ma obsahovat to iste co malo mat ticket, teda detail transakcie, tellera
   a pobocky, chybovy kod z CBS a ktory zapis do DB zlyhal?
3. Ma sa mail poslat pri vsetkych troch situaciach (zlyhal zapis do DB, pokyn
   v trvalej chybe, pokyn sa nezauctoval do konca dna) alebo len pri niektorych?
```

Blokuje dokončenie UC. Dopad: mení sa BP07.

**6. Naviazane transakcne kody od pani Tibenskej → pani Tibenska**

```
Zo suboru Transakcie.xlsx uz viem ze poplatok za vklad minci ma request code 28
a poplatok za vyber minci 29. To pokryva to co sme nazyvali naviazane transakcne
kody.

Este stale ale cakame na ten subor s mapovanim naviazanych kodov o ktorom sme
sa bavili na calle. Vies ho poslat, alebo uz plati ze vsetko je v tom subore
Transakcie.xlsx a nic dalsie nepribudne?
```

Neblokuje, ak platí Transakcie.xlsx ako kompletný zdroj. Dopad: prípadné rozšírenie BP01.

**7. Tabulky BRANCH_JOURNAL nie su v datovom modeli → vyvoj**

```
UC0417 zapisuje do tabuliek BRANCH_JOURNAL, BRANCH_JOURNAL_CUSTOMER,
BRANCH_JOURNAL_OWNER, BRANCH_JOURNAL_CASH_BALANCE a BRANCH_JOURNAL_CHARGE.
Ani jedna z nich nie je v aktualnom datovom modeli cashbox_db.png.

V modeli su len transaction_journal a transaction_journal_cash_balance_internal.

Idu sa tie tabulky este vytvarat, alebo mam prepisat UC na tie co existuju?
```

Blokuje implementáciu. Dopad: mení sa sekcia Mapping.

**8. Naplnenie poli pri nevzniknutom poplatku → Matus Radusovsky**

```
Pri vklade z dovodu nefunkcneho bankomatu poplatok nevznika a zapisujeme
zaznam s kodom 2723 alebo 2724.

Potrebujem vediet co presne sa do toho zaznamu zapise. FEE_AMOUNT = 0? Aky
CHARGE_TYPE? Ktore polia sa nechavaju prazdne?

A este - ocakava tieto transakcie poplatkovy vypis z Tabisu? To by vedel
potvrdit aj Zoli.
```

Blokuje dokončenie UC. Dopad: mení sa BP08 a sekcia Mapping.

**9. Retencia IT logov → bezpecnost alebo prevadzka**

```
Zurnal mame na 10+1 rok, to je jasne. Ale IT logy nevieme ako dlho sa
uchovavaju. Vies mi to zistit alebo povedat na koho sa mam obratit?
```

Neblokuje dokončenie UC. Dopad: doplnenie hodnoty do sekcie Mapping.

**10. Zmeny na APSRV20 → Tomas Machacek**

```
Pisal si ze AS400 pripravili predstavu o nastaveni a bude sa to spolu overovat,
a ze zatial predpokladame ze na APSRV20 sa nebudu musiet robit zmeny.

Uz je to overene? Potrebujem vediet ci to mozem v UC napisat ako potvrdene
alebo to tam nechat ako predpoklad.
```

Neblokuje dokončenie UC. Dopad: prípadná zmena mapovania polí.

### Biznis zadanie

Zaúčtovanie transakcie do CBS cez rozhranie APSRV20.

UC0417 je záverečný krok realizácie transakcie. Keď sa UC0417 spustí, všetko je už potvrdené vrátane SPV override a klient odišiel s podpísaným potvrdením z DSA.

**Kľúčový princíp.** Klient odchádza s potvrdením **pred** zaúčtovaním, pretože tlač prebieha v DSA. Preto transakcia **musí** byť spracovaná. Systém nesmie transakciu potichu zahodiť. Každý neúspešný pokyn musí byť dohľadateľný a musí sa doňho vrátiť.

**Valuta transakcie.** Podľa VOP Tatra banky je valutou dátum prevzatia finančných prostriedkov od klienta. Transakcia sa preto **musí zaúčtovať v ten istý bankový deň**, v ktorom klient peniaze priniesol alebo prevzal. Ak sa to nepodarí, transakciu nemožno jednoducho zaúčtovať nasledujúci deň, ale rieši sa manuálnou opravou (potvrdil Feri). [OTVORENY BOD: postup manuálnej opravy, viď otázka 1]

**Rozsah UC0417.** Do UC0417 patrí zostavenie účtovacích pokynov, ich odoslanie do CBS, sledovanie stavu transakcie, spracovanie chýb a zápis do lokálnej databázy. Do UC0417 nepatrí:

| Čo | Rieši |
|---|---|
| Overenie realizovateľnosti transakcie, teda stav účtu, blokácie a prostriedky | UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady), UC0504 (výbery) |
| Výpočet halierového vyrovnania a určenie jeho transakčného kódu | UC0416 - Realizácia - Halierové vyrovnanie |
| Výpočet poplatku | UC0431 - Poplatok za vklad - Stanovenie výšky, UC0433 - Poplatok za mince - Stanovenie výšky |
| Tlač a podpis dokumentácie | UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA |
| Podmienky storna, teda kto ho môže vykonať a dokedy | Samostatný UC pre storno, blok 11 - Prehľady, položka Storno TXN |

**Vzťah k susedným UC.** UC0417 beží po UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA. Je posledným krokom transakcie.

**Použitie pre viaceré typy transakcií.** UC0417 je spoločný pre vklady, výbery aj rozmieňanie. Rozdiely podľa typu transakcie sú v sekcii Opis obrazoviek + Validácie.

### Aktéri

| Aktér | Čo v tomto UC robí |
|---|---|
| **Teller** | Prijíma informačnú hlášku o výsledku a potvrdzuje ju. Do priebehu zaúčtovania nezasahuje |
| **Systém** | Vykonáva všetky ostatné kroky. Zostavuje účtovacie pokyny, odosiela ich do CBS, sleduje status, spracúva chyby, zapisuje do lokálnej databázy a zobrazuje hlášky |

---

## Vstupné podmienky

- Teller je prihlásený, pobočka je otvorená, pokladňa je otvorená
- Transakcia je natypovaná (UC0403 - Príprava - Natypovanie transakcie pre vklady, UC0503 pre výbery)
- Poplatky sú vypočítané
- Halierové vyrovnanie je pripravené, ak vzniklo (UC0416 - Realizácia - Halierové vyrovnanie)
- Dokumentácia je vygenerovaná a podpísaná v DSA (UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA)
- SPV override je potvrdený, ak bol potrebný
- Realizovateľnosť transakcie je overená v UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery)
- UC0421 - Nad limit - Zistenie dodatočných informácií o vklade a UC0422 - Nad limit - Kontrola údajov supervízorom prebehli, ak boli potrebné
- Systém eviduje dostupnosť CBS v tabuľke `as400_values`, stĺpec `is_online`

---

## Hlavný tok

1. Systém zostaví zoznam účtovacích pokynov pre danú transakciu podľa BP01.
2. Systém pridelí každému pokynu vlastný identifikátor a všetkým pokynom spoločný konsolidačný kľúč podľa BP06.
3. Systém zapíše všetky pokyny do žurnálu so statusom S podľa BP03.
4. Systém odošle hlavný pokyn, teda vklad alebo výber, cez rozhranie MW_APP.AccountBookingRest a zmení jeho status na T podľa BP02.
5. Systém vyhodnotí odpoveď CBS pre hlavný pokyn:
   - Ak CBS potvrdí zaúčtovanie, status hlavného pokynu sa zmení na C a UC pokračuje nasledujúcim krokom
   - Ak CBS vráti chybu, status hlavného pokynu sa zmení na E a tok pokračuje **AT2**
   - Ak CBS neodpovie v stanovenom čase, status hlavného pokynu zostáva T a tok pokračuje **AT1**
6. Systém odošle závislé pokyny v poradí podľa BP02, teda halierové vyrovnanie, poplatok a poplatok za vklad alebo výber mincí, a nastaví im status T.
7. Systém vyhodnotí odpoveď CBS pre každý závislý pokyn samostatne:
   - Ak CBS potvrdí zaúčtovanie, status pokynu sa zmení na C
   - Ak CBS vráti chybu, status pokynu sa zmení na E a pokyn sa spracuje podľa **AT2**. Hlavná transakcia sa neruší
   - Ak CBS neodpovie, status pokynu zostáva T a pokyn sa spracuje podľa **AT1**. Hlavná transakcia sa neruší
8. Systém vyhodnotí celkový status transakcie podľa BP03.
9. Systém vykoná zápisy do lokálnej databázy podľa sekcie Mapping:
   - Ak všetky zápisy prebehnú, UC pokračuje nasledujúcim krokom
   - Ak ktorýkoľvek zápis zlyhá, tok pokračuje **AT3**
10. Systém zobrazí tellerovi informačnú hlášku podľa typu transakcie a výsledného statusu.
11. Teller potvrdí hlášku tlačidlom OK.
12. Systém presmeruje tellera na homepage.
13. Systém ukončí UC.

---

## Alternatívny tok

### AT1 - Pokyn zostáva v statuse T

**Spúšťač:** CBS nepotvrdí zaúčtovanie pokynu v stanovenom čase.
**Platí pre:** hlavný aj závislé pokyny, všetky typy transakcií.
**Krok v hlavnom toku:** krok 5 alebo krok 7.

1. Pokyn zostáva v statuse T. Systém ho nezahadzuje ani neopakuje okamžite.
2. Systém vyhodnotí, o ktorý pokyn ide:
   - Ak ide o hlavný pokyn, systém neodosiela závislé pokyny. Tie zostávajú v statuse S a odošlú sa až po úspešnom potvrdení hlavného pokynu
   - Ak ide o závislý pokyn, hlavná transakcia sa neruší a spracovanie pokračuje
3. Systém pokračuje krokom 9 hlavného toku a zobrazí tellerovi hlášku o čakajúcom spracovaní. Teller môže pokračovať ďalšou transakciou.
4. Pokyn v statuse T sa ďalej spracúva opakovaným spracovaním podľa BP04, teda automaticky každých 15 minút počas celého bankového dňa a vynútene pri zatváraní pokladne.
5. Systém vyhodnotí výsledok opakovaného spracovania:
   - Ak CBS potvrdí zaúčtovanie, status pokynu sa zmení na C a systém dokončí prípadné chýbajúce zápisy do lokálnej databázy
   - Ak CBS vráti chybu, status pokynu sa zmení na E a tok pokračuje **AT2**
   - Ak CBS opäť neodpovie, pokyn zostáva v statuse T a opakované spracovanie sa vykoná znova
6. Ak sa pokyn nezaúčtuje do konca bankového dňa, tok pokračuje **AT5**.
7. Teller vidí transakciu so statusom T v žurnáli.

### AT2 - Pokyn skončí v statuse E

**Spúšťač:** CBS vráti chybu pri odoslaní pokynu alebo pri opakovanom spracovaní.
**Platí pre:** hlavný aj závislé pokyny, všetky typy transakcií.
**Krok v hlavnom toku:** krok 5, krok 7 alebo AT1 krok 5.

1. Systém zaznamená chybový kód a text z odpovede CBS do žurnálu a do IT logu.
2. Systém klasifikuje chybu podľa BP05:
   - Ak ide o prechodnú chybu, pokyn sa vráti do statusu T a spracuje sa opakovaným spracovaním podľa BP04
   - Ak ide o trvalú chybu, pokyn dostane finálny status E a UC pokračuje nasledujúcim krokom
3. Systém vykoná notifikáciu podľa BP07, teda odošle okamžitý email a tellerovi sprístupní odkaz na JIRA.
4. Systém vyhodnotí, o ktorý pokyn ide:
   - Ak ide o hlavný pokyn, závislé pokyny sa neodosielajú a zostávajú v statuse S. Systém ich označí ako nezrealizované s odkazom na chybu hlavného pokynu
   - Ak ide o závislý pokyn, hlavná transakcia sa neruší ani nestornuje. Zaúčtovaný zostáva ten pokyn, ktorý prešiel, a chýbajúci pokyn sa dorieši manuálne
5. Systém zobrazí tellerovi chybovú hlášku.
6. Ak status E nastal po tom, čo klient odišiel s podpísaným potvrdením z DSA, systém tento stav nerieši automaticky. Notifikácia podľa BP07 je podnetom na manuálne doriešenie prevádzkou, pretože opätovné zaúčtovanie alebo storno je účtovné rozhodnutie. Systém zabezpečuje, že takýto prípad je vždy zaznamenaný a dohľadateľný.

### AT3 - Zlyhanie zápisu do lokálnej databázy po úspešnom zaúčtovaní

**Spúšťač:** CBS potvrdil zaúčtovanie, ale zápis do lokálnej databázy zlyhal.
**Platí pre:** všetky typy transakcií.
**Krok v hlavnom toku:** krok 9.

1. Transakcia je z pohľadu banky aj klienta úspešná. Peniaze sú zaúčtované a klient má potvrdenie.
2. Chýba záznam v lokálnej databáze CashBoxu, teda transakcia nie je v žurnáli.
3. Systém vykoná notifikáciu podľa BP07 s celým detailom transakcie, tellera a pobočky, aby bolo možné zápis dodatočne doplniť.
4. Systém zapíše udalosť do IT logu s typom ERROR.
5. Systém zobrazí tellerovi štandardnú hlášku o úspechu transakcie. Teller ani klient nie sú informovaní o technickej chybe, pretože transakcia z ich pohľadu prebehla správne a teller nemá ako do situácie zasiahnuť. [OTVORENY BOD: potvrdiť správanie]
6. Zápis do lokálnej databázy sa dopĺňa dodatočne v rámci riešenia. Transakcia nesmie zostať bez záznamu.

### AT4 - Storno transakcie

**Spúšťač:** Požiadavka na storno už zaúčtovanej transakcie.
**Platí pre:** všetky typy transakcií.
**Krok v hlavnom toku:** nadväzuje na už zaúčtovanú transakciu, mimo hlavného toku UC0417.

1. Transakciu nie je možné stornovať počas priebehu UC0417. Do žurnálu sa zapisuje až po potvrdení z CBS.
2. Storno vytvára **novú transakciu** s vlastnými pokynmi, ktorú treba zapísať samostatne aj odoslať do CBS. Suma aj osoba sú zhodné s pôvodnou transakciou.
3. Systém identifikuje pokyny pôvodnej transakcie cez konsolidačný kľúč a pre každý vytvorí reverzný pokyn podľa BP06 s reverzným transakčným kódom, príznakom reverse_flag = Y a odkazom na identifikátor pôvodného pokynu. Reverzné kódy sú v sekcii API.
4. Stornovať sa musia všetky pokyny pôvodnej transakcie, teda hlavná transakcia, halierové vyrovnanie, poplatok aj poplatok za mince. Nie je prípustné stornovať len časť pokynov.
5. Podmienky storna, teda kto ho môže vykonať, dokedy a ktoré transakcie sa dajú stornovať, nie sú predmetom UC0417. Rieši ich samostatný UC pre storno, ktorý teller spúšťa zo žurnálu.

### AT5 - Pokyn sa nezaúčtoval do konca bankového dňa

**Spúšťač:** Pokyn je v statuse T alebo S a nastáva zatvorenie pokladne alebo koniec bankového dňa.
**Platí pre:** všetky typy transakcií.
**Krok v hlavnom toku:** nadväzuje na AT1.

1. Systém pri zatváraní pokladne identifikuje všetky nespracované pokyny tellera.
2. Systém zobrazí hlášku **I012** so zoznamom nespracovaných transakcií a možnosťou spracovania jednotlivo alebo hromadne.
3. Systém vykoná opakované spracovanie pre vybrané pokyny podľa BP04.
4. Systém vyhodnotí výsledok každého pokynu podľa AT1 alebo AT2.
5. Ak pokyn ani po opakovanom spracovaní neskončí v statuse C, transakciu **nemožno zaúčtovať nasledujúci bankový deň**, pretože podľa VOP je valutou dátum prevzatia finančných prostriedkov. Pokyn sa preto rieši manuálnou opravou. [OTVORENY BOD: postup manuálnej opravy, kto ju vykonáva a ako sa zabezpečí správna valuta, viď otázka 1]
6. Systém vykoná notifikáciu podľa BP07.

### AT6 - Hromadný vklad, čiastočné zlyhanie

**Spúšťač:** Pri hromadnom vklade niektorý z čiastkových vkladov skončí v statuse T alebo E.
**Platí pre:** hromadný vklad, teda UC0415 - Realizácia - Hromadný vklad.
**Krok v hlavnom toku:** krok 5 alebo krok 7 pre konkrétny čiastkový vklad.

1. Systém nezastavuje spracovanie hromadného vkladu.
2. Systém zapíše nespracovaný vklad a pokračuje spracovaním ďalšieho vkladu v poradí.
3. Po spracovaní všetkých vkladov systém ukončí transakciu. Súhrnná hláška sa nezobrazuje. [OTVORENY BOD: či sa má súhrnná hláška vytvoriť, viď otázka 4]
4. Teller vidí jednotlivé vklady samostatne v žurnáli vrátane ich statusov.
5. Nespracované vklady sa riešia podľa AT1 alebo AT2 samostatne, každý so svojím konsolidačným kľúčom.

---

## Biznis pravidlá

### BP01 - Skladba účtovacích pokynov

Rozhranie APSRV20 neumožňuje grupovanie viacerých pohybov do jednej správy. CashBox preto pre každú transakciu pripravuje samostatné pokyny.

| # | Pokyn | Kedy sa vytvára | Request code |
|---|---|---|---|
| 1 | **Hlavná transakcia**, teda vklad alebo výber | Vždy | Podľa typu transakcie a meny, viď sekcia API |
| 2 | **Halierové vyrovnanie** | Ak vzniklo, viď UC0416 - Realizácia - Halierové vyrovnanie | 25 pri kladnom, 26 pri zápornom rozdiele |
| 3 | **Poplatok** | Len ak sa poplatok platí v hotovosti | 23, 24 alebo 27 podľa typu poplatku |
| 4 | **Poplatok za vklad alebo výber mincí** | Ak klient vkladá alebo vyberá mince a poplatok za mince vznikol, viď UC0433 - Poplatok za mince - Stanovenie výšky | 28 pri vklade mincí, 29 pri výbere mincí |

Transakčný kód vyhodnocuje **CashBox**, nie CBS. Platí to aj pre halierové vyrovnanie.

Ak sa poplatok neplatí v hotovosti, poplatkový pokyn sa nevytvára. Poplatok zaúčtuje CBS v rámci kapitalizácie.

**Zdroj transakčných kódov.** Kompletné mapovanie transakčných typov na request codes a nové transakčné kódy je v súbore Transakcie.xlsx, záložka Potrebné trans.kódy. Prehľad relevantný pre UC0417 je v sekcii API.

**Poznámka k pojmu naviazané transakčné kódy.** V predchádzajúcich verziách UC sa používal pojem naviazané transakčné kódy bez konkrétneho obsahu. Ide o poplatok za vklad alebo výber mincí, ktorý má v súbore Transakcie.xlsx pridelený request code 28 a 29. [OTVORENY BOD: či okrem poplatku za mince pribudnú ďalšie naviazané kódy z analýzy pani Tibenskej, viď otázka 6]

### BP02 - Poradie odosielania pokynov

Poradie potvrdil Feri.

Pokyny sa odosielajú **sekvenčne**, nie paralelne, v tomto poradí:

| Poradie | Pokyn | Dôvod |
|---|---|---|
| 1 | Hlavná transakcia | Základ transakcie, ostatné pokyny na ňu nadväzujú |
| 2 | Halierové vyrovnanie | Súvisí s hlavnou transakciou a jej účtovným dorovnaním |
| 3 | Poplatok | Hotovosť je už v pokladni |
| 4 | Poplatok za vklad alebo výber mincí | Nadväzuje na hotovostnú časť transakcie |

**Pravidlá odosielania:**

1. Ako prvý sa odosiela hlavný pokyn.
2. Závislé pokyny sa odosielajú až po úspešnom potvrdení hlavného pokynu, teda po dosiahnutí statusu C.
3. Ak hlavný pokyn skončí v statuse T alebo E, závislé pokyny sa neodosielajú a zostávajú v statuse S.

**Dôvod tohto poradia.** Zabraňuje vzniku osamotených poplatkových alebo vyrovnávacích pohybov bez hlavnej transakcie. Zaúčtovaný poplatok bez zaúčtovaného vkladu by bol účtovne nesprávny a jeho odstránenie by vyžadovalo storno.

**Čiastočné zlyhanie.** Ak hlavný pokyn prejde a niektorý závislý pokyn zlyhá, hlavná transakcia sa nestornuje ani nevracia. Zlyhaný pokyn sa spracuje samostatne podľa AT1 alebo AT2. Dôvod: klient už má peniaze a potvrdenie, spätné rušenie hlavnej transakcie by bolo pre neho horšie riešenie ako dodatočné doúčtovanie chýbajúceho pohybu.

### BP03 - Životný cyklus statusu

Statusy aj evidenciu na úrovni pokynu potvrdil Feri.

| Status | Význam |
|---|---|
| **S** | Štart transakcie. Pokyn je pripravený a zapísaný, ešte nebol odoslaný |
| **T** | Pokyn bol odoslaný do CBS, čaká sa na potvrdenie |
| **C** | CBS potvrdil zaúčtovanie, pokyn je úspešne zrealizovaný |
| **E** | CBS vrátil chybu |

Status sa eviduje **na úrovni jednotlivého pokynu**, nie na úrovni celej transakcie. Vyplýva to z toho, že transakcia sa skladá z viacerých samostatných pokynov, ktoré môžu skončiť rôzne.

**Vyhodnotenie statusu celej transakcie:**

| Podmienka | Status transakcie |
|---|---|
| Všetky pokyny sú v statuse C | C |
| Aspoň jeden pokyn je v statuse T alebo S a žiadny nie je v E | T |
| Aspoň jeden pokyn je v statuse E | E |

Status pokynu sa ukladá do stĺpca `status` v tabuľke `transaction_journal`, ktorý je typu varchar(35).

### BP04 - Opakované spracovanie nespracovaných pokynov

Opakované spracovanie je opätovné vyžiadanie informácie od CBS o tom, či bol pokyn zaúčtovaný. **Nejde o opätovné odoslanie pokynu.** Identifikátor pokynu zostáva rovnaký, takže CBS duplicitné zaúčtovanie nevykoná (BP06).

V TABISe automatický mechanizmus neexistuje. Forwardy sa kontrolujú pri odhlásení a nespracované transakcie sa forwardujú manuálne (potvrdil Feri).

**Interval a rozsah v CashBoxe** (potvrdil Feri):

| Vlastnosť | Hodnota |
|---|---|
| Interval automatického spracovania | Každých 15 minút |
| Trvanie | Počas celého bankového dňa, kým sa pokyn nezaúčtuje alebo neuzavrie |
| Maximálny počet pokusov | Nestanovuje sa. Spracovanie beží až do zaúčtovania alebo uzavretia pokynu |

**Spúšťače opakovaného spracovania:**

| Spúšťač | Rozsah |
|---|---|
| Automatický interval počas dňa | Všetky pokyny v statuse T |
| Zatvorenie pokladne | Pokyny tellera v statuse T alebo S, viď AT5 |
| Manuálne z prehľadu | Konkrétny pokyn, ak to prehľad umožňuje |

**Koniec bankového dňa.** Pokyn, ktorý sa nezaúčtoval do konca bankového dňa, **nemožno zaúčtovať nasledujúci deň**. Podľa VOP Tatra banky je valutou dátum prevzatia finančných prostriedkov od klienta. Takýto pokyn sa rieši manuálnou opravou. [OTVORENY BOD: postup manuálnej opravy, viď otázka 1]

### BP05 - Klasifikácia chýb z CBS

Klasifikáciu potvrdil Feri.

| Kategória | Charakteristika | Spracovanie |
|---|---|---|
| **Prechodná chyba** | Nedostupnosť CBS, timeout na strane CBS, dočasná chyba spracovania | Pokyn sa vráti do statusu T a spracuje sa opakovaným spracovaním podľa BP04 |
| **Trvalá chyba** | Chyba dát, neplatný transakčný kód, chyba validácie na strane CBS | Pokyn dostane finálny status E, vykoná sa notifikácia podľa BP07, opakované spracovanie sa nevykonáva |

**Zdroj zaradenia chybových kódov.** Zaradenie konkrétnych chybových kódov CBS je definované v konfiguračnom číselníku v databáze CashBox. **Číselník zatiaľ neexistuje a je potrebné ho vytvoriť.** Napĺňa sa pri nasadení podľa zoznamu chybových kódov, ktoré vracia rozhranie APSRV20. [OTVORENY BOD: zoznam chybových kódov APSRV20, viď otázka 2]

Chybový kód, ktorý v číselníku nie je uvedený, sa štandardne vyhodnotí ako **trvalá chyba**, pretože je bezpečnejšie dostať transakciu pod ľudskú kontrolu než ju nechať zacykliť v opakovaných pokusoch.

**Biznisové chyby** ako zrušený účet, nedostatok prostriedkov, blokácia účtu alebo prekročenie limitov sa v UC0417 nevyskytujú. Sú odchytené v UC0404 - Príprava - Kontrola uskutočniteľnosti alebo UC0504 pred spustením realizácie.

### BP06 - Previazanie pokynov a ochrana pred dvojitým zaúčtovaním

**Previazanie.** Všetky pokyny jednej transakcie majú rovnaký **konsolidačný kľúč** (`consReference`). Cez neho systém identifikuje súvisiace pohyby pri storne, pri vyhodnocovaní celkového statusu transakcie a pri zobrazovaní v žurnáli.

**Ochrana pred dvojitým zaúčtovaním.** Každý pokyn má vlastný **identifikátor** (`application_transaction_id`), ktorý podľa mapovania rozhrania slúži na kontrolu duplicitného poslania z aplikácie. Identifikátor sa prideľuje pri vytvorení pokynu a **nemení sa pri opakovanom odoslaní ani pri opakovanom spracovaní**. Tým je zabezpečené, že pokyn nemôže byť zaúčtovaný dvakrát, ani keď CashBox nedostane odpoveď a pokus zopakuje.

**Storno.** Reverzný pokyn sa vytvára cez `reverse_flag = Y` a `original_transaction_id` s odkazom na pôvodný pokyn. Používa sa reverzný transakčný kód priradený k pôvodnému kódu podľa súboru Transakcie.xlsx, prehľad je v sekcii API.

### BP07 - Notifikácia pri chybe

**Systém nevytvára JIRA ticket automaticky** (rozhodol Feri). Automatické vytváranie by bolo rizikové z dvoch dôvodov: mohlo by vzniknúť veľké množstvo zbytočných incidentov, a odobralo by možnosť pridať k incidentu vlastný popis a snímky obrazovky.

**Systém namiesto toho vykoná:**

| Akcia | Popis |
|---|---|
| Okamžitý email | Systém odošle email bezprostredne po vzniku chyby. [OTVORENY BOD: príjemca a obsah, viď otázka 5] |
| Odkaz na JIRA | Systém sprístupní odkaz na JIRA, cez ktorý je možné incident založiť ručne s vlastným popisom a snímkami obrazovky |

**Situácie, v ktorých sa notifikácia vykoná:**

| Situácia | Zdroj |
|---|---|
| Zlyhal zápis do lokálnej databázy po úspešnom zaúčtovaní | AT3 |
| Pokyn skončil v trvalom statuse E | AT2 |
| Pokyn sa nezaúčtoval do konca bankového dňa | AT5 |

**Obsah notifikácie:** celý detail transakcie, tellera a pobočky, aby bolo možné nájsť chybu a dohľadať situáciu v kóde. Konkrétne identifikátor transakcie a pokynu, konsolidačný kľúč, transakčný kód, suma a mena, identifikátor tellera, identifikátor pobočky, časová pečiatka, status pokynu, chybový kód a text z CBS, ak existuje, a informácia o tom, ktorý zápis do databázy zlyhal.

**SLA.** Riešenie musí prebehnúť **okamžite**, teda v najkratšom možnom čase. Dôvodom je, že klient už má peniaze a potvrdenie, kým banka nemá transakciu zaúčtovanú alebo zaznamenanú (potvrdil Feri).

**Fallback pri nedostupnosti emailu.** Ak sa email nepodarí odoslať, systém zapíše udalosť do IT logu s typom ERROR a do žurnálu. Nespracovaná transakcia zostáva viditeľná v prehľade nespracovaných transakcií, takže sa nestratí ani bez notifikácie.

### BP08 - Nevzniknutý poplatok

Pri transakciách, kde poplatok nevzniká, napríklad pri vklade z dôvodu nefunkčného bankomatu, sa do evidencie poplatkov zapisuje záznam so samostatným transakčným kódom.

| Typ klienta | Pôvodný kód | Request code | Nový transakčný kód | Reverzný kód |
|---|---:|---:|---:|---:|
| Fyzická osoba a právnická osoba | 2723 | 9 | 4017 | 4018 |
| Fyzická osoba podnikateľ | 2724 | 10 | 4019 | 4020 |

[OTVORENY BOD: naplnenie polí záznamu pri nevzniknutom poplatku a potvrdenie, či sa tieto transakcie očakávajú v poplatkovom výpise, viď otázka 8]

---

## Diagram tokov

[OTVORENY BOD: diagram bude doplnený]

---

## Výstupné podmienky

**Úspech, transakcia v statuse C:**
- Všetky pokyny transakcie sú zaúčtované v CBS, každý v statuse C
- Transakcia je zaúčtovaná v ten istý bankový deň, v ktorom klient priniesol alebo prevzal peniaze
- Záznamy sú zapísané v lokálnej databáze podľa sekcie Mapping
- V evidencii poplatkov je záznam, buď o vzniknutom poplatku, alebo o nevzniknutom poplatku podľa BP08
- Tellerovi sa zobrazila informačná hláška podľa typu transakcie
- Teller je presmerovaný na homepage

**Čakajúce spracovanie, transakcia v statuse T:**
- Aspoň jeden pokyn čaká na potvrdenie z CBS
- Transakcia je viditeľná pre tellera v žurnáli so statusom T
- Pokyn sa spracúva opakovane každých 15 minút počas celého bankového dňa a vynútene pri zatváraní pokladne
- Zápisy do lokálnej databázy sú vykonané v rozsahu, ktorý je možný pred potvrdením
- Teller môže pokračovať ďalšou transakciou

**Chyba, transakcia v statuse E:**
- Aspoň jeden pokyn skončil v trvalej chybe
- Bola vykonaná notifikácia podľa BP07, teda odoslaný email a sprístupnený odkaz na JIRA
- Transakcia je viditeľná pre tellera v žurnáli so statusom E
- Ďalšie riešenie prebieha manuálne

**Nezaúčtované do konca bankového dňa:**
- Pokyn zostal v statuse T alebo S aj po zatvorení pokladne
- Transakciu nemožno zaúčtovať nasledujúci bankový deň, rieši sa manuálnou opravou
- Bola vykonaná notifikácia podľa BP07
- [OTVORENY BOD: postup manuálnej opravy, viď otázka 1]

**Zlyhanie zápisu do lokálnej databázy:**
- Transakcia je zaúčtovaná v CBS, ale nie je zapísaná v žurnáli
- Bola vykonaná notifikácia podľa BP07
- Zápis sa dopĺňa dodatočne

---

## Opis obrazoviek + Validácie

UC0417 je prevažne backendový proces. Tellerovi sa zobrazuje len informačná hláška po ukončení zaúčtovania.

Katalóg hlášok: Info a chybové hlášky, AppLib, ConfluenceIT.

**Poznámka k chýbajúcim hláškam.** Tri zo štyroch hlášok v tomto UC nie sú v schválenom katalógu. Texty uvedené nižšie sú návrhy, ktoré treba dať schváliť a prideliť im kódy. Kým sa tak nestane, nie je možné podľa nich testovať. [OTVORENY BOD: viď otázka 3]

### Hláška po úspešnom vklade

Zobrazuje sa pri vklade vždy.

| Prvok | Hodnota |
|---|---|
| Kód | **I025** (schválená) |
| Typ | Info popup |
| Text | Prosim nezabudnite vziat od klienta peniaze. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška po úspešnom výbere

Zobrazuje sa pri výbere, ak všetko prebehlo v poriadku.

| Prvok | Hodnota |
|---|---|
| Kód | [OTVORENY BOD: hláška nie je v katalógu, ide o návrh] |
| Typ | Info popup |
| Text | Vydajte hotovosť klientovi. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška pri transakcii v statuse T

Zobrazuje sa, keď CBS nepotvrdí zaúčtovanie v stanovenom čase (AT1). Teller môže pokračovať ďalšou transakciou.

| Prvok | Hodnota |
|---|---|
| Kód | [OTVORENY BOD: hláška nie je v katalógu, ide o návrh] |
| Typ | Info popup |
| Text | Transakcia čaká na spracovanie. Môžete pokračovať ďalšou transakciou. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška pri chybe zaúčtovania

Zobrazuje sa, keď pokyn skončí v trvalom statuse E (AT2).

| Prvok | Hodnota |
|---|---|
| Kód | [OTVORENY BOD: hláška nie je v katalógu, ide o návrh] |
| Typ | Error popup |
| Text | Transakciu sa nepodarilo zaúčtovať. Chyba bola zaznamenaná a odovzdaná na riešenie. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |
| Doplnok | Odkaz na JIRA na založenie incidentu podľa BP07 |

### Hláška pri zlyhaní zápisu do lokálnej databázy

Samostatná hláška sa nezobrazuje. Tellerovi sa zobrazí štandardná hláška o úspechu podľa typu transakcie, pretože transakcia z jeho aj klientovho pohľadu prebehla správne a teller nemá možnosť do situácie zasiahnuť. Chyba sa rieši notifikáciou na pozadí podľa BP07. [OTVORENY BOD: potvrdiť správanie]

### Hláška pri opakovanom spracovaní počas zatvárania pokladne

Používa sa existujúca schválená hláška **I012** so zoznamom nespracovaných transakcií a možnosťou spracovania jednotlivo alebo hromadne.

### Hláška pri hromadnom vklade

Súhrnná hláška sa nezobrazuje. Po potvrdení a zápise sa transakcia ukončí a teller vidí jednotlivé vklady samostatne v žurnáli. [OTVORENY BOD: či sa má súhrnná hláška vytvoriť, viď otázka 4]

### Rozdiely podľa typu transakcie

| Typ transakcie | Rozdiely |
|---|---|
| UC0411 - Realizácia - Vklad na účet v mene účtu | Štandardný priebeh podľa hlavného toku |
| UC0412 - Realizácia - Vklad na účet v inej mene ako účtu | Štandardný priebeh. Ide o cross currency transakciu, platia povinné polia podľa sekcie API. Vzniká halierové vyrovnanie ako závislý pokyn |
| UC0413 - Realizácia - Vklad - Nefunkčný bankomat | Štandardný priebeh, ale poplatok nevzniká. Namiesto poplatkového pokynu sa vytvára záznam o nevzniknutom poplatku podľa BP08 |
| UC0415 - Realizácia - Hromadný vklad | UC0417 sa spúšťa postupne pre každý vklad, vrátane subtransakcií. Každý vklad má vlastnú sadu pokynov a vlastný konsolidačný kľúč. Pri zlyhaní jedného vkladu systém pokračuje ďalším, viď AT6 |
| Výbery, teda UC0511, UC0512, UC0515 a UC0518 | Štandardný priebeh. Rozdiel je v transakčných kódoch a v znení informačnej hlášky |
| Rozmieňanie, teda UC701 | Do CBS sa posiela len poplatok, request code 23. Samotné rozmieňanie sa neúčtuje, kód 621 je neúčtovný |

### Validácie

| # | Kontrola | Podmienka pre pokračovanie | Pri nesplnení | Testovateľné cez |
|---|---|---|---|---|
| 1 | Vstupné podmienky | Všetky splnené, teda podpis DSA, override, poplatky | UC0417 sa nespustí | Spustiť realizáciu bez podpísanej dokumentácie |
| 2 | Konsolidačný kľúč | Všetky pokyny transakcie majú rovnakú hodnotu | Chyba prípravy pokynov | Porovnať pokyny jednej transakcie |
| 3 | Identifikátor pokynu | Každý pokyn má jedinečný identifikátor, ktorý sa pri opakovaní nemení | Chyba prípravy pokynov | Vyvolať opakované spracovanie a porovnať identifikátory |
| 4 | Poradie odosielania | Závislé pokyny sa neodosielajú pred potvrdením hlavného | Pokyny zostávajú v statuse S | Simulovať zlyhanie hlavného pokynu |
| 5 | Výber request code | Použitý request code zodpovedá typu transakcie a mene podľa tabuľky v sekcii API | Trvalá chyba, AT2 | Vyvolať jednotlivé typy transakcií a porovnať odoslaný request code |
| 6 | Cross currency polia | Pri konverzii sú vyplnené kurz a sumy na oboch stranách | Trvalá chyba, AT2 | Odoslať cross currency pokyn bez kurzu |
| 7 | Zápis do lokálnej databázy | Všetky zápisy prebehli | AT3 | Simulovať výpadok databázy po potvrdení z CBS |
| 8 | Zaúčtovanie v ten istý bankový deň | Pokyn dosiahol status C do konca bankového dňa | AT5, manuálna oprava | Simulovať nedostupnosť CBS až do zatvorenia pokladne |

---

## API

### Rozhranie

Zaúčtovanie prebieha cez **MW_APP.AccountBookingRest**, čo je REST rozhranie nad APSRV20. APSRV20 sa nazýva aj AccountBooking, špecifikácia je v MW_APP.AccountBooking. REST verzia je obálka nad MQ verziou, endpoint `/esb/restapi/midas-account/booking`.

Timeout volania si definuje APSRV20, na strane CashBoxu sa nedefinuje.

**Stav integrácie.** AS400 pripravili predstavu o nastavení a prebieha overovanie. Zatiaľ sa predpokladá, že na APSRV20 sa nebudú musieť robiť zmeny. [OTVORENY BOD: potvrdiť po overení, viď otázka 10]

### Mapovanie transakčných typov na request codes

Zdroj: súbor Transakcie.xlsx, záložka Potrebné trans.kódy. Mapovanie má podobu tabuľky, nie vyhodnotenia per jednotlivý prípad. CashBox vyberie riadok podľa typu transakcie a odošle príslušný request code.

**Vklady na účet v mene účtu**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Vklad na BÚ v mene účtu | 200 | 1 | 4001 | 4002 |
| Vklad na BÚ v mene účtu, živnostník | 280 | 2 | 4003 | 4004 |
| Vklad na BÚ v mene účtu, hromadný vklad | 1745 | 3 | 4005 | 4006 |
| Vklad na BÚ v mene účtu, živnostník, hromadný vklad | 1749 | 4 | 4007 | 4008 |
| Vklad na BÚ v mene účtu s poplatkom v hotovosti | 2704 | 5 | 4009 | 4010 |
| Vklad na BÚ v mene účtu, živnostník, s poplatkom v hotovosti | 2708 | 6 | 4011 | 4012 |
| Vklad na BÚ v mene účtu, hromadný vklad s poplatkom v hotovosti | 2711 | 7 | 4013 | 4014 |
| Vklad na BÚ v mene účtu, živnostník, hromadný vklad s poplatkom v hotovosti | 2713 | 8 | 4015 | 4016 |

**Vklady pri nefunkčnom bankomate**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Vklad na BÚ, pokazený bankomat, PO a FO | 2723 | 9 | 4017 | 4018 |
| Vklad na BÚ, pokazený bankomat, FOP | 2724 | 10 | 4019 | 4020 |

**Vklady na účet v inej mene ako mena účtu**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Vklad na BÚ v inej mene ako mena účtu | 202 | 11 | 4021 | 4022 |
| Vklad na BÚ v inej mene ako mena účtu, bez poplatku | 212 | 12 | 4023 | 4024 |
| Vklad na BÚ v inej mene ako mena účtu, živnostník | 282 | 13 | 4025 | 4026 |
| Vklad na BÚ v inej mene ako mena účtu s poplatkom v hotovosti | 2705 | 14 | 4027 | 4028 |
| Vklad na BÚ v inej mene ako mena účtu, živnostník, s poplatkom v hotovosti | 2709 | 15 | 4029 | 4030 |

**Výbery**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Výber z BÚ v mene účtu | 300 | 16 | 4031 | 4032 |
| Výber z BÚ v mene účtu, bez poplatku, nefunkčný ATM | 309 | 17 | 4033 | 4034 |
| Výber z BÚ v inej mene ako mena účtu | 307 | 18 | 4035 | 4036 |

**Poplatky**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Poplatky hotovosť | 982 | 23 | 4045 | 4046 |
| Poplatky hotovosť | 983 | 24 | 4047 | 4048 |
| Poplatok vkladateľa v hotovosti | 923 | 27 | 4053 | 4054 |
| Poplatok za vklad mincí | 934 | 28 | 4055 | 4056 |
| Poplatok za výber mincí | 935 | 29 | 4057 | 4058 |
| Exchange charges | 910 | 30 | 4059 | 4060 |

**Halierové vyrovnanie**

| Znamienko rozdielu | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Kladné | 924 | 25 | 4049 | 4050 |
| Záporné | 925 | 26 | 4051 | 4052 |

**Dotácie a odvody**

| Typ transakcie | Pôvodný kód | Request code | Nový kód | Reverzný |
|---|---:|---:|---:|---:|
| Dotácia | 600 | 19 | 4037 | 4038 |
| Odvod | 601 | 20 | 4039 | 4040 |
| Teller short | 610 | 21 | 4041 | 4042 |
| Teller over | 611 | 22 | 4043 | 4044 |

**Kódy bez request code**

| Skupina | Kódy | Poznámka |
|---|---|---|
| Interné transakčné kódy | 920, 921, 930, 940 | Nemajú request code, označené ako interný transakčný kód |
| Poplatky bez nového kódu | 931, 932, 936 | V zozname označené hodnotou "nie", nový ani reverzný kód nemajú |
| Neúčtovné transakčné kódy | 605 (interná dotácia), 606 (interný odvod), 621 (rozmieňanie), 622 (interné rozmieňanie) | Do CBS sa neposielajú, účtovanie prebieha výlučne v CashBoxe. Nový kód majú 4061 až 4068 |

### Kľúčové polia účtovacieho pokynu

| Pole (REST JSON path) | Hodnota |
|---|---|
| channel_id | CBX |
| request_code | Podľa tabuľky vyššie |
| version | 7 |
| reverse_flag | N, pri storne Y, viď AT4 |
| transaction_amount | Suma pohybu |
| transaction_currency | Mena pohybu |
| exchange_rate | Povinné pri cross currency transakciách |
| debit_part.bank_code, account_prefix, account_number | Vyplniť, ak je na debetnej strane retailový účet, teda pri výbere |
| credit_part.bank_code, account_prefix, account_number | Vyplniť, ak je na kreditnej strane retailový účet, teda pri vklade |
| credit_part.variable_symbol, specific_symbol | Variabilný a špecifický symbol, ak sa neposiela endToEndID |
| constant_symbol | Konštantný symbol, ak sa neposiela endToEndID |
| info_beneficiary | Popis transakcie, maximálne 140 znakov |
| application_transaction_id | Identifikátor pokynu, kontrola duplicitného poslania podľa BP06 |
| original_transaction_id | Pri reverznej transakcii identifikátor pôvodného pokynu |
| consReference | Konsolidačný kľúč, spoločný pre všetky pokyny transakcie podľa BP06 |
| transaction_branch | 3-znakový kód pobočky |
| parcial_payment | N |

**Polia, ktoré sa neplnia:** debit_part.narrative, credit_part.narrative, credit_part.title, payment_reference, debit_part.specific_symbol, debit_part.variable_symbol, debit_part.title, debit_part.info, credit_part.info, info_bank.

**Symboly a endToEndID.** Posiela sa buď endToEndID, alebo variabilný, konštantný a špecifický symbol samostatne, z ktorých CBS endToEndID vyskladá. Nikdy nie oboje. Používa sa vždy kreditná strana variabilného a konštantného symbolu, aj pri vkladoch aj pri výberoch.

### Cross currency transakcie

Pri transakciách s konverziou mien sú povinné tieto polia:

| Pole | Obsah |
|---|---|
| exchange_rate | Kurz použitý na konverziu. Typ kurzu určuje kategória transakcie podľa UC0403 - Príprava - Natypovanie transakcie |
| debit_part.amount | Suma na debetnej strane |
| debit_part.currency | Mena debetnej strany |
| credit_part.amount | Suma na kreditnej strane |
| credit_part.currency | Mena kreditnej strany |

### Zdrojové tabuľky lokálnej databázy CashBox

| Tabuľka | Použitie v UC0417 |
|---|---|
| `as400_values` | Overenie dostupnosti CBS pred odoslaním pokynu, stĺpec `is_online` |
| Konfiguračný číselník chybových kódov CBS | Klasifikácia chýb podľa BP05. [OTVORENY BOD: číselník zatiaľ neexistuje] |

Poznámka: AS400 a CBS označujú ten istý centrálny bankový systém (potvrdil Matúš Radušovský). V texte UC sa používa pojem CBS, názvy technických objektov zostávajú nezmenené.

---

## Mapping

### Zápisy do lokálnej databázy

| Tabuľka | Obsah zápisu | Kedy |
|---|---|---|
| `transaction_journal` | Jeden riadok pre každý pokyn vrátane statusu, transakčného kódu, sumy, meny, účtu a konsolidačného kľúča | Pri vytvorení pokynu so statusom S, aktualizuje sa pri zmene statusu |
| `transaction_journal_cash_balance_internal` | Detail mincovky k transakcii na úrovni jednotlivých nominálov | Po potvrdení hlavného pokynu |
| BRANCH_JOURNAL | Záznam transakcie | Po potvrdení hlavného pokynu |
| BRANCH_JOURNAL_CUSTOMER | Údaje o klientovi k transakcii | Po potvrdení hlavného pokynu |
| BRANCH_JOURNAL_OWNER | Údaje o vlastníkovi účtu | Po potvrdení hlavného pokynu |
| BRANCH_JOURNAL_CASH_BALANCE | Mincovka k transakcii. Pri vklade v inej mene ako EUR vznikajú dva záznamy, jeden pre menu transakcie a jeden pre poplatok v EUR, rozlíšené cez identifikátor meny | Po potvrdení hlavného pokynu |
| CASHBOX_TOTAL_REALIZED | Zmeny po realizácii | Po potvrdení hlavného pokynu |
| BRANCH_JOURNAL_CHARGE | Záznam o poplatku pre poplatkový výpis, aj pri nevzniknutom poplatku podľa BP08 | Po potvrdení príslušného pokynu |

**Poznámka pre vývoj.** Tabuľky BRANCH_JOURNAL, BRANCH_JOURNAL_CUSTOMER, BRANCH_JOURNAL_OWNER, BRANCH_JOURNAL_CASH_BALANCE a BRANCH_JOURNAL_CHARGE **nie sú v aktuálnom dátovom modeli CashBox**. Je potrebné ich vytvoriť alebo UC zosúladiť s existujúcimi tabuľkami `transaction_journal` a `transaction_journal_cash_balance_internal`. [OTVORENY BOD: viď otázka 7]

### Zápis pri nevzniknutom poplatku

Pri transakciách, kde poplatok nevzniká, sa zapisuje záznam s transakčným kódom 4017 alebo 4019 podľa BP08.

[OTVORENY BOD: naplnenie jednotlivých polí záznamu, viď otázka 8. Zo štruktúry evidencie poplatkov popísanej v UC701 - Rozmieňanie hotovosti vyplýva, že záznam obsahuje typ poplatku, identifikátor poplatku, sadzbu, sumu poplatku, číslo účtu, názov aplikácie a identifikátor brandu, nie je však určené, aké hodnoty sa plnia pri nulovom poplatku]

### Halierové vyrovnanie

Halierové vyrovnanie sa odosiela ako samostatný účtovací pokyn previazaný s hlavnou transakciou cez konsolidačný kľúč. Suma, transakčný kód a cieľový účet sa určujú podľa UC0416 - Realizácia - Halierové vyrovnanie. UC0417 zabezpečuje odoslanie, sledovanie statusu a spracovanie chýb tohto pokynu rovnako ako pri ostatných pokynoch.

### Logovanie a auditing

**Žurnál CashBox** (rozsah potvrdil Feri):
- Loguje sa celý priebeh UC0417, teda vytvorenie pokynov, odoslanie, prijaté odpovede z CBS, zmeny statusu, pokusy o opakované spracovanie a zápisy do databázy
- Zaznamenáva sa finálny status každého pokynu aj celej transakcie a stav účtovania
- Pri chybe sa zaznamenáva chybový kód a text z CBS
- **Retencia:** žurnál sa uchováva 10 + 1 rok. Po jedenástich rokoch sa záznamy mažú po dňoch

**IT logy.** Zapisujú sa do nástroja typu Splunk. Formát záznamu:

```
[DATUM][CAS] [ID_POUZIVATELA] [TYP_LOGU] [KOD_LOGU] - [POPIS_LOGU]
```

Typy logov: INFO, ERROR, WARNING, OVERIDE.

Do IT logu sa zapisuje odoslanie a výsledok každého pokynu s typom INFO, chyby z CBS s typom ERROR, zlyhanie zápisu do lokálnej databázy s typom ERROR, pokusy o opakované spracovanie s typom INFO a vykonanie notifikácie s typom ERROR.

[OTVORENY BOD: retencia IT logov, viď otázka 9]

### DSA

Tlač potvrdenky prebieha v DSA **pred** zaúčtovaním (UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA). UC0417 už len potvrdzuje výsledok spracovania po návrate z DSA do CashBoxu.

---

## Poznámky pre teba - nekopírovať do UC

### Tri pripomienky, ktoré zmenili UC podstatne

**1. Valuta a koniec bankového dňa.** Feriho poznámka o VOP je najzávažnejšia. Doteraz sme mali, že pokyny v statuse T sa jednoducho prenesú do ďalšieho dňa. To je podľa neho nesprávne, lebo valutou je dátum prevzatia peňazí. Prepísal som BP04, doplnil nový AT5 a novú výstupnú podmienku. Zároveň to vytvorilo najdôležitejšiu otvorenú otázku, lebo pojem "manuálna oprava" nie je nikde definovaný.

**2. Žiadny automatický JIRA ticket.** BP07 je prepísané celé. Namiesto automatického ticketu ide okamžitý email a odkaz na JIRA. Feriho argument je vecný: pri opakovanej chybe by vzniklo množstvo duplicitných incidentov a človek by nemal ako pridať vlastný popis a snímky.

**3. Opakované spracovanie bez limitu pokusov.** Pôvodne som navrhoval maximálne 10 pokusov. Feri to zmenil na neobmedzené počas celého dňa. Odstránil som limit z BP04 aj z BP07.

### Čo priniesol Excel

Kompletné mapovanie request codes odpovedá naraz na Feriho otázku (*"ako si to predstavuje vývoj"*) aj na otázku vývojára o naviazaných kódoch. Mapovanie je tabuľka, nie logika per prípad.

Zároveň sa ukázalo, že **naviazané transakčné kódy** = poplatok za vklad alebo výber mincí, request code 28 a 29. Ten pojem bol v UC roky bez obsahu.

Zostáva overiť u pani Tibenskej, či Transakcie.xlsx je kompletný zdroj, alebo ešte niečo pribudne.

### Čo som nechal ako otvorené napriek pripomienke

Vývojár pri hláškach napísal, že si ich *"vieme vymyslieť ale potom to bude chýbať pri testoch"*. Má pravdu, preto som texty ponechal ako návrhy a nepridelil im kódy. Priradenie kódov patrí správcovi katalógu, inak vzniknú konflikty s číslovaním, ktoré sme už použili v UC0402 a v starších návrhoch.
