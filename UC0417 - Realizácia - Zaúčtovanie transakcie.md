# UC0417 \- Realizácia \- Zaúčtovanie transakcie

## Obsah





## Otázky

Biznis zadanie

Zaúčtovanie transakcie do CBS cez rozhranie APSRV20.

UC0417 je záverečný krok realizácie transakcie. Keď sa UC0417 spustí, všetko je už potvrdené vrátane SPV override a klient odišiel s podpísaným potvrdením z DSA.

**Kľúčový princíp.** Klient odchádza s potvrdením **pred** zaúčtovaním, pretože tlač prebieha v DSA. Preto transakcia **musí** byť nakoniec spracovaná. Systém nesmie transakciu potichu zahodiť. Každý neúspešný pokyn musí byť dohľadateľný a musí sa doňho vrátiť.

**Vzťah k susedným UC.** UC0417 beží po UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA. Je posledným krokom transakcie.

**Použitie pre viaceré typy transakcií.** UC0417 je spoločný pre vklady, výbery aj rozmieňanie. Rozdiely podľa typu transakcie sú v sekcii Opis obrazoviek + Validácie.

## Aktéri

- Systém
- Teller (prijíma informačnú hlášku o výsledku)

## Vstupné podmienky

- Teller je prihlásený, pobočka je otvorená, pokladňa je otvorená
- Transakcia je natypovaná (UC0403 - Príprava - Natypovanie transakcie pre vklady, UC0503 pre výbery)
- Poplatky sú vypočítané
- Halierové vyrovnanie je pripravené, ak vzniklo (UC0416 - Realizácia - Halierové vyrovnanie)
- Dokumentácia je vygenerovaná a podpísaná v DSA (UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA)
- SPV override je potvrdený, ak bol potrebný
- Realizovateľnosť transakcie je overená v UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery)
- UC0421 - Príprava - Zistenie dodatočných informácií o vklade a UC0422 - Príprava - Kontrola údajov supervízorom prebehli, ak boli potrebné
- Systém eviduje dostupnosť CBS v tabuľke `as400_values`, stĺpec `is_online`


## Hlavný tok

1. Systém zostaví zoznam účtovacích pokynov pre danú transakciu podľa BP01.
2. Systém pridelí každému pokynu vlastný identifikátor a všetkým pokynom spoločný konsolidačný kľúč podľa BP06.
3. Systém zapíše všetky pokyny do žurnálu so statusom S podľa BP03.
4. Systém odošle hlavný pokyn, teda vklad alebo výber, cez rozhranie MW\_APP.AccountBookingRest a zmení jeho status na T podľa BP02.
5. Systém vyhodnotí odpoveď CBS pre hlavný pokyn:
    - Ak CBS potvrdí zaúčtovanie, status hlavného pokynu sa zmení na C a UC pokračuje nasledujúcim krokom
    - Ak CBS vráti chybu, status hlavného pokynu sa zmení na E a tok pokračuje **AT2**
    - Ak CBS neodpovie v stanovenom čase, status hlavného pokynu zostáva T a tok pokračuje **AT1**
6. Systém odošle závislé pokyny, teda halierové vyrovnanie, poplatok a naviazané transakčné kódy (pripomienka od vývojara:toto najdeme kde ? alebo co to je zac ?), podľa BP02 a nastaví im status T.
7. Systém vyhodnotí odpoveď CBS pre každý závislý pokyn samostatne:
    - Ak CBS potvrdí zaúčtovanie, status pokynu sa zmení na C
    - Ak CBS vráti chybu, status pokynu sa zmení na E a pokyn sa spracuje podľa **AT2**. Hlavná transakcia sa neruší
    - Ak CBS neodpovie, status pokynu zostáva T a pokyn sa spracuje podľa **AT1**. Hlavná transakcia sa neruší
8. Systém vyhodnotí celkový status transakcie podľa BP03.
9. Systém vykoná zápisy do lokálnej databázy podľa BP04:
    - Ak všetky zápisy prebehnú, UC pokračuje nasledujúcim krokom
    - Ak ktorýkoľvek zápis zlyhá, tok pokračuje **AT3**
10. Systém zobrazí tellerovi informačnú hlášku podľa typu transakcie a výsledného statusu (viď Opis obrazoviek + Validácie).
11. Teller potvrdí hlášku tlačidlom OK.
12. Systém presmeruje tellera na homepage.
13. Systém ukončí UC.


## Alternatívny tok

####  AT1 - Pokyn zostáva v statuse T

**Spúšťač:** CBS nepotvrdí zaúčtovanie pokynu v stanovenom čase.  
**Platí pre:** hlavný aj závislé pokyny, všetky typy transakcií.  
**Krok v hlavnom toku:** krok 5 alebo krok 7.

1. Pokyn zostáva v statuse T. Systém ho nezahadzuje ani neopakuje okamžite.
2. Systém vyhodnotí, o ktorý pokyn ide:
    - Ak ide o hlavný pokyn, systém neodosiela závislé pokyny. Tie zostávajú v statuse S a odošlú sa až po úspešnom potvrdení hlavného pokynu
    - Ak ide o závislý pokyn, hlavná transakcia sa neruší a spracovanie pokračuje
3. Systém pokračuje krokom 9 hlavného toku a zobrazí tellerovi hlášku o čakajúcom spracovaní. Teller môže pokračovať ďalšou transakciou.
4. Pokyn v statuse T sa ďalej spracúva mechanizmom opakovaného spracovania podľa BP04, teda automaticky v pravidelnom intervale a vynútene pri zatváraní pokladne.
5. Systém vyhodnotí výsledok opakovaného spracovania:
    - Ak CBS potvrdí zaúčtovanie, status pokynu sa zmení na C a systém dokončí prípadné chýbajúce zápisy do lokálnej databázy
    - Ak CBS vráti chybu, status pokynu sa zmení na E a tok pokračuje **AT2**
    - Ak CBS opäť neodpovie, pokyn zostáva v statuse T a opakované spracovanie sa vykoná znova. Po prekročení maximálneho počtu pokusov podľa BP04 systém vytvorí incident podľa BP07
6. Teller vidí transakciu so statusom T v žurnáli.

### AT2 - Pokyn skončí v statuse E

**Spúšťač:** CBS vráti chybu pri odoslaní pokynu alebo pri opakovanom spracovaní.  
**Platí pre:** hlavný aj závislé pokyny, všetky typy transakcií.  
**Krok v hlavnom toku:** krok 5, krok 7 alebo AT1 krok 5.

1. Systém zaznamená chybový kód a text z odpovede CBS do žurnálu a do IT logu.
2. Systém klasifikuje chybu podľa BP05:
    - Ak ide o prechodnú chybu, pokyn sa vráti do statusu T a spracuje sa opakovaným spracovaním podľa BP04
    - Ak ide o trvalú chybu, pokyn dostane finálny status E a UC pokračuje nasledujúcim krokom
3. Systém vytvorí incident podľa BP07. Incident obsahuje chybový kód z CBS, detail transakcie, tellera a pobočky.
4. Systém vyhodnotí, o ktorý pokyn ide:
    - Ak ide o hlavný pokyn, závislé pokyny sa neodosielajú a zostávajú v statuse S. Systém ich označí ako nezrealizované s odkazom na chybu hlavného pokynu
    - Ak ide o závislý pokyn, hlavná transakcia sa neruší ani nestornuje. Zaúčtovaný zostáva ten pokyn, ktorý prešiel, a chýbajúci pokyn sa dorieši cez incident
5. Systém zobrazí tellerovi chybovú hlášku (viď Opis obrazoviek + Validácie).
6. Ak status E nastal po tom, čo klient odišiel s podpísaným potvrdením z DSA, systém tento stav nerieši automaticky. Vytvorený incident je podnetom na manuálne doriešenie prevádzkou, pretože opätovné zaúčtovanie alebo storno je účtovné rozhodnutie. Systém zabezpečuje, že takýto prípad je vždy zaznamenaný a dohľadateľný.

### AT3 - Zlyhanie zápisu do lokálnej databázy po úspešnom zaúčtovaní

**Spúšťač:** CBS potvrdil zaúčtovanie, ale zápis do lokálnej databázy zlyhal.  
**Platí pre:** všetky typy transakcií.  
**Krok v hlavnom toku:** krok 9.

1. Transakcia je z pohľadu banky aj klienta úspešná. Peniaze sú zaúčtované a klient má potvrdenie.
2. Chýba záznam v lokálnej databáze CashBoxu, teda transakcia nie je v žurnáli.
3. Systém vytvorí incident podľa BP07 s celým detailom transakcie, tellera a pobočky, aby bolo možné zápis dodatočne doplniť.
4. Systém zapíše udalosť do IT logu s typom ERROR.
5. Systém zobrazí tellerovi štandardnú hlášku o úspechu transakcie. Teller ani klient nie sú informovaní o technickej chybe, pretože transakcia z ich pohľadu prebehla správne a teller nemá ako do situácie zasiahnuť. \[OTVORENY BOD: potvrdiť správanie\]
6. Zápis do lokálnej databázy sa dopĺňa dodatočne v rámci riešenia incidentu. Transakcia nesmie zostať bez záznamu.

### AT4 - Storno transakcie

**Spúšťač:** Požiadavka na storno už zaúčtovanej transakcie.  
**Platí pre:** všetky typy transakcií.  
**Krok v hlavnom toku:** nadväzuje na už zaúčtovanú transakciu, mimo hlavného toku UC0417.

1. Transakciu nie je možné stornovať počas priebehu UC0417. Do žurnálu sa zapisuje až po potvrdení z CBS.
2. Storno vytvára **novú transakciu** s vlastnými pokynmi, ktorú treba zapísať samostatne aj odoslať do CBS. Suma aj osoba sú zhodné s pôvodnou transakciou.
3. Systém identifikuje pokyny pôvodnej transakcie cez konsolidačný kľúč a pre každý vytvorí reverzný pokyn podľa BP06 s reverzným transakčným kódom, príznakom reverse\_flag = Y a odkazom na identifikátor pôvodného pokynu.
4. Stornovať sa musia všetky pokyny pôvodnej transakcie, teda hlavná transakcia, halierové vyrovnanie, poplatok aj naviazané transakčné kódy. Nie je prípustné stornovať len časť pokynov.
5. Podmienky storna, teda kto ho môže vykonať, dokedy a ktoré transakcie sa dajú stornovať, nie sú predmetom UC0417. Rieši ich samostatný UC pre storno, ktorý teller spúšťa zo žurnálu.

### AT5 - Opakované spracovanie pri zatváraní pokladne

**Spúšťač:** Teller zatvára pokladňu a existujú pokyny v statuse T alebo S.  
**Platí pre:** všetky typy transakcií.  
**Krok v hlavnom toku:** nadväzuje na AT1, mimo hlavného toku UC0417.

1. Systém pri zatváraní pokladne identifikuje všetky nespracované pokyny tellera.
2. Systém zobrazí hlášku **I012** so zoznamom nespracovaných transakcií a možnosťou spracovania jednotlivo alebo hromadne.
3. Systém vykoná opakované spracovanie pre vybrané pokyny podľa BP04.
4. Systém vyhodnotí výsledok každého pokynu podľa AT1 alebo AT2.
5. Pokyny, ktoré ani po opakovanom spracovaní neskončia v statuse C, dostanú finálny status E. Systém pre ne vytvorí incident podľa BP07 a zároveň vytvorí e-mail na riadenie servisu, keďže tieto prípady sa riešia individuálne.

\[OTVORENY BOD: či opakované spracovanie patrí do UC0417 alebo do samostatného UC\]

### AT6 - Hromadný vklad, čiastočné zlyhanie

**Spúšťač:** Pri hromadnom vklade niektorý z čiastkových vkladov skončí v statuse T alebo E.  
**Platí pre:** hromadný vklad.  
**Krok v hlavnom toku:** krok 5 alebo krok 7 pre konkrétny čiastkový vklad.

1. Systém nezastavuje spracovanie hromadného vkladu.
2. Systém zapíše nespracovaný vklad a pokračuje spracovaním ďalšieho vkladu v poradí.
3. Po spracovaní všetkých vkladov systém ukončí transakciu. Súhrnná hláška sa nezobrazuje
4. Teller vidí jednotlivé vklady samostatne v žurnáli vrátane ich statusov.
5. Nespracované vklady sa riešia podľa AT1 alebo AT2 samostatne, každý so svojím konsolidačným kľúčom.

## Biznis pravidlá

### BP01 - Skladba účtovacích pokynov

Rozhranie APSRV20 neumožňuje grupovanie viacerých pohybov do jednej správy. CashBox preto pre každú transakciu pripravuje samostatné pokyny.

| **#** | **Pokyn** | **Kedy sa vytvára** | **Zdroj transakčného kódu** |
| --- | --- | --- | --- |
| 1 | Hlavná transakcia, teda vklad alebo výber | Vždy | Podľa typu transakcie a meny, zdroj Transakcie.xlsx |
| 2 | Halierové vyrovnanie | Ak vzniklo (UC0416 - Realizácia - Halierové vyrovnanie) | Request code 25 alebo 26, transakčný kód 4049 alebo 4051 |
| 3 | Poplatok | Len ak sa poplatok platí v hotovosti | Podľa typu poplatku |
| 4 | Naviazané transakčné kódy | Podľa typu transakcie, napríklad poplatok za vklad mincí | Konfiguračný číselník naviazaných kódov |

**Konfiguračný číselník naviazaných transakčných kódov** definuje, ktoré doplnkové pokyny sa vytvárajú pre daný typ transakcie. Číselník je konfigurovateľný a napĺňa sa hodnotami z analýzy transakčných kódov. Zdrojom je súbor Transakcie.xlsx, sekcie "Hlavné tr.kódy - rozklad" (mapovanie hlavný kód na naviazané kódy podľa poradia) a "Potrebné trans.kódy" (zoznam použitých kódov s request code) - UC0417 sa musí držať v rámci transakčných kódov definovaných v tomto súbore.

**Štruktúra číselníka.** Pre každý hlavný transakčný kód (MAINCOD) číselník uvádza poradie (ORDER), v akom sa naviazané pokyny vytvárajú, ich transakčný kód (TRDCOD) a popis (TRDESC). Poradie 1 je vždy hlavný pokyn.

**Príklad pre hlavný kód 200 (vklad na BÚ v mene účtu):**

| Poradie | Transakčný kód | Popis |
| --- | --- | --- |
| 1 | 200 | Current account deposit (hlavný pokyn) |
| 2 | 923 | Poplatok vkladateľa v hotovosti |
| 3 | 932 | Popl. za zvýš. prácnosť |
| 4 | 934 | Popl. za vklad mincí |

**Príklad pre hlavný kód 202 (vklad na BÚ v inej mene ako mena účtu):**

| Poradie | Transakčný kód | Popis |
| --- | --- | --- |
| 1 | 202 | CA deposit in non ac. ccy (hlavný pokyn) |
| 2 | 920 | Deposit non ac. ccy - cash box |
| 3 | 931 | Poplatok vkladateľa v hotovosti |
| 4 | 930 | FX pos and fx profit - buy |
| 5 | 940 | FX pos and FX profit sell |
| 6 | 924 | Halierové vyrovnanie |
| 7 | 936 | Popl. za zvýš. prácnosť |
| 8 | 934 | Popl. za vklad mincí |

Kompletný číselník pre všetky hlavné transakčné kódy (vklady, výbery, hromadný vklad, nefunkčný bankomat) je v prílohe **Transakcie.xlsx**, záložka "Hlavné tr.kódy - rozklad".

Ak sa poplatok neplatí v hotovosti, poplatkový pokyn sa nevytvára. Poplatok zaúčtuje CBS v rámci kapitalizácie.

Transakčný kód vyhodnocuje **CashBox**, nie CBS. Platí to aj pre halierové vyrovnanie


BP02 - Poradie odosielania pokynov

\[NÁVRH NA POTVRDENIE: nasledujúce pravidlo je návrh riešenia, ktorý nebol potvrdený. Otázka na poradie pokynov, čakanie na potvrdenie a správanie pri čiastočnom zlyhaní zostáva otvorená.\] (pripomienka od Feriho: súhlasím s týmto poradím:

hlavná transakcia

musí byť halierové lebo súvisí s hlavnou transakciou a jej dorovnaním účtovne

poplatok, lebo ho máme v kase

vklad alebo výber míncí)

Pokyny sa odosielajú **sekvenčne**, nie paralelne:

1. Ako prvý sa odosiela hlavný pokyn, teda vklad alebo výber.
2. Závislé pokyny sa odosielajú až po úspešnom potvrdení hlavného pokynu, teda po dosiahnutí statusu C.
3. Ak hlavný pokyn skončí v statuse T alebo E, závislé pokyny sa neodosielajú a zostávajú v statuse S.
4. Závislé pokyny sa medzi sebou odosielajú v poradí: halierové vyrovnanie, poplatok, naviazané transakčné kódy.

**Dôvod tohto poradia.** Zabraňuje vzniku osamotených poplatkových alebo vyrovnávacích pohybov bez hlavnej transakcie. Zaúčtovaný poplatok bez zaúčtovaného vkladu by bol účtovne nesprávny a jeho odstránenie by vyžadovalo storno.

**Čiastočné zlyhanie.** Ak hlavný pokyn prejde a niektorý závislý pokyn zlyhá, hlavná transakcia sa nestornuje ani nevracia. Zlyhaný pokyn sa spracuje samostatne podľa AT1 alebo AT2. Dôvod: klient už má peniaze a potvrdenie, spätné rušenie hlavnej transakcie by bolo pre neho horšie riešenie ako dodatočné doúčtovanie chýbajúceho phyb


BP03 - Životný cyklus statusu


| **Status** | **Význam** |
| --- | --- |
| **S** | Štart transakcie. Pokyn je pripravený, ešte nebol odoslaný |
| **T** | Pokyn je zapísaný do databázy a odoslaný do CBS, čaká sa na potvrdenie |
| **C** | CBS potvrdil zaúčtovanie, transakcia je úspešne zrealizovaná |
| **E** | CBS vrátil chybu |

\[NÁVRH NA POTVRDENIE: status sa eviduje **na úrovni jednotlivého pokynu**, nie na úrovni celej transakcie. Vyplýva to z toho, že transakcia sa skladá z viacerých samostatných pokynov, ktoré môžu skončiť rôzne.\] (pripomienka od Feriho:áno súhlas)

**Vyhodnotenie statusu celej transakcie:**

| **Podmienka** | **Status transakcie** |
| --- | --- |
| Všetky pokyny sú v statuse C | C |
| Aspoň jeden pokyn je v statuse T alebo S a žiadny nie je v E | T |
| Aspoň jeden pokyn je v statuse E | E |

Status pokynu sa ukladá do stĺpca `status` v tabuľke `transaction_journal`, ktorý je typu varchar(35). \[OTVORENY BOD: potvrdiť s Matúšom Radušovským (pripomienka od Feriho: áno)


 BP04 - Opakované spracovanie nespracovaných pokynov

Opakované spracovanie je opätovné vyžiadanie informácie od CBS o tom, či bol pokyn zaúčtovaný. **Nejde o opätovné odoslanie pokynu.** Identifikátor pokynu zostáva rovnaký, takže CBS duplicitné zaúčtovanie nevykoná (BP06).

V TABISe automatický mechanizmus neexistuje. Forwardy sa kontrolujú pri odhlásení a nespracované transakcie sa forwardujú manuálne 

**Spúšťače pre CashBox:**

| **Spúšťač** | **Rozsah** | **Stav** |
| --- | --- | --- |
| Zatvorenie pokladne | Pokyny tellera v statuse T alebo S |  |
| Automatický interval počas dňa | Všetky pokyny v statuse T |  |
| Manuálne z prehľadu | Konkrétny pokyn | Ak to prehľad umožňuje |

\[NÁVRH NA POTVRDENIE: automatický interval 15 minút, maximálny počet automatických pokusov 10. Obe hodnoty sú konfiguračné parametre. Po prekročení maximálneho počtu pokusov systém prestane pokyn automaticky spracúvať a vytvorí incident podľa BP07. Pokyn zostáva v statuse T a je naďalej viditeľný v prehľade nespracovaných transakcií.\] (pripomienka od Feriho: súhlas každých 15 minúť ale počas celého dňa pokiaľ nedôjde k zaúčtovaniu alebo zatvoreniu pokynu)

**Prenos cez bankový deň.** Pokyny v statuse T sa pri zatvorení pobočky neanulujú a prenášajú sa do ďalšieho bankového dňa . (pripomienka od Feriho: toto musíme spracovať alebo riešiť nasledujúci deň cez manuálnu opravu, nemôžeme zaúčtovať na druhý deň (podľa VOP TB je valuta dátum prevzatia fin. prostriedkov).)

### BP05 - Klasifikácia chýb z CBS

\[NÁVRH NA POTVRDENIE: nasledujúce pravidlo je návrh riešenia. Zoznam chybových kódov CBS a ich zaradenie nie sú k dispozícii.\] (pripomienka od Feriho: súhlas)

| **Kategória** | **Charakteristika** | **Spracovanie** |
| --- | --- | --- |
| **Prechodná chyba** | Nedostupnosť CBS, timeout na strane CBS, dočasná chyba spracovania | Pokyn sa vráti do statusu T a spracuje sa opakovaným spracovaním podľa BP04 |
| **Trvalá chyba** | Chyba dát, neplatný transakčný kód, chyba validácie na strane CBS | Pokyn dostane finálny status E, vytvára sa incident podľa BP07, opakované spracovanie sa nevykonáva |

Zaradenie konkrétnych chybových kódov CBS je definované v konfiguračnom číselníku (pripomienka od vývojára: ten najdeme kde ?), ktorý sa napĺňa pri nasadení. Chybový kód, ktorý v číselníku nie je uvedený, sa štandardne vyhodnotí ako **trvalá chyba**, pretože je bezpečnejšie dostať transakciu pod ľudskú kontrolu než ju nechať zacykliť v opakovaných pokusoch.

**Biznisové chyby** ako zrušený účet, nedostatok prostriedkov, blokácia účtu alebo prekročenie limitov sa v UC0417 nevyskytujú. Sú odchytené v UC0404 - Príprava - Kontrola uskutočniteľnosti alebo UC0504 pred spustením realizácie 

### BP06 - Previazanie pokynov a ochrana pred dvojitým zaúčtovaním

**Previazanie.** Všetky pokyny jednej transakcie majú rovnaký **konsolidačný kľúč** (`consReference`). Cez neho systém identifikuje súvisiace pohyby pri storne, pri vyhodnocovaní celkového statusu transakcie a pri zobrazovaní v žurnáli.

**Ochrana pred dvojitým zaúčtovaním.** Každý pokyn má vlastný **identifikátor** (`application_transaction_id`), ktorý podľa mapovania rozhrania slúži na kontrolu duplicitného poslania z aplikácie. Identifikátor sa prideľuje pri vytvorení pokynu a **nemení sa pri opakovanom odoslaní ani pri opakovanom spracovaní**. Tým je zabezpečené, že pokyn nemôže byť zaúčtovaný dvakrát, ani keď CashBox nedostane odpoveď a pokus zopakuje.

**Storno.** Reverzný pokyn sa vytvára cez `reverse_flag = Y` a `original_transaction_id` s odkazom na pôvodný pokyn. Používa sa reverzný transakčný kód priradený k pôvodnému kódu podľa súboru Transakcie.xlsx.


 BP07 - Incident management

Systém automaticky vytvára incident v podobe JIRA ticketu v týchto prípadoch (pripomienka od Feriho: TUto sme sa dohodli ze pridame len link na Jiru, automaticky vytvoreny ticket na incident by bol nebezpecny aby sa nam nenastali pripady kedy by sa zbytocne vytvorilo x incidentov alebo tiez by sme tymto odobrali moznost pridanie custom popisu a screenshotov ):

| **Situácia** | **Zdroj** |
| --- | --- |
| Zlyhal zápis do lokálnej databázy po úspešnom zaúčtovaní | AT3 |
| Pokyn skončil v trvalom statuse E | AT2 |
| Pokyn prekročil maximálny počet pokusov opakovaného spracovania | BP04, AT5 |

**Obsah incidentu**: celý detail transakcie, tellera a pobočky, aby bolo možné nájsť chybu a dohľadať situáciu v kóde. Konkrétne identifikátor transakcie a pokynu, konsolidačný kľúč, transakčný kód, suma a mena, identifikátor tellera, identifikátor pobočky, časová pečiatka, status pokynu, chybový kód a text z CBS, ak existuje, a informácia o tom, ktorý zápis do databázy zlyhal.

**Príjemca** : osoba zodpovedná za CashBox, ktorá vie problém riešiť. Konkrétny JIRA projekt a komponent sú konfiguračné parametre nastavené pri nasadení.

**SLA.** Riešenie incidentu podlieha prísnemu SLA vzhľadom na to, že klient už má peniaze a potvrdenie. \[OTVORENY BOD: konkrétne hodnoty SLA\]

**Fallback pri nedostupnosti JIRA.** Ak sa incident nepodarí vytvoriť, systém zapíše udalosť do IT logu s typom ERROR a do žurnálu. Nespracovaná transakcia zostáva viditeľná v prehľade nespracovaných transakcií, takže sa nestratí ani bez incidentu. \[OTVORENY BOD: či má systém navyše aktívne notifikovať o nespracovaní transakcie\]

**Technická realizácia.** Predpokladá sa, že CashBox dokáže vytvoriť JIRA ticket priamo, keďže ide o rovnakú platformu ako Safebox, ktorý vytvára JIRA tickety pre OSC. \[OTVORENY BOD: potvrdiť s vývojom\]

(pripomienka od Feriho:má byť okamžitý mail. SLA - je hneď aleob najnižšie možné.Tu potrebujeme riešiť ASAP. )

### B BP08 - Nevzniknutý poplatok

Pri transakciách, kde poplatok nevzniká, napríklad pri vklade z dôvodu nefunkčného bankomatu, sa do evidencie poplatkov zapisuje záznam so samostatným transakčným kódom 

| **Typ klienta** | **Transakčný kód** |
| --- | --- |
| Fyzická osoba a právnická osoba | 2723 |
| Fyzická osoba podnikateľ | 2724 |

\[OTVORENY BOD: naplnenie polí záznamu pri nevzniknutom poplatku a potvrdenie, či sa tieto transakcie očakávajú v poplatkovom výpise\]

**Úspech, transakcia v statuse C:**

- Všetky pokyny transakcie sú zaúčtované v CBS, každý v statuse C
- Záznamy sú zapísané v lokálnej databáze podľa BP04
- V evidencii poplatkov je záznam, buď o vzniknutom poplatku, alebo o nevzniknutom poplatku podľa BP08
- Tellerovi sa zobrazila informačná hláška podľa typu transakcie
- Teller je presmerovaný na homepage

**Čakajúce spracovanie, transakcia v statuse T:**

- Aspoň jeden pokyn čaká na potvrdenie z CBS
- Transakcia je viditeľná pre tellera v žurnáli so statusom T
- Pokyn sa spracúva opakovane podľa BP04 a vynútene pri zatváraní pokladne
- Zápisy do lokálnej databázy sú vykonané v rozsahu, ktorý je možný pred potvrdením
- Teller môže pokračovať ďalšou transakciou

**Chyba, transakcia v statuse E:**

- Aspoň jeden pokyn skončil v trvalej chybe
- Je vytvorený incident podľa BP07 s chybovým kódom z CBS
- Transakcia je viditeľná pre tellera v žurnáli so statusom E
- Ďalšie riešenie prebieha manuálne v rámci incidentu

**Zlyhanie zápisu do lokálnej databázy:**

- Transakcia je zaúčtovaná v CBS, ale nie je zapísaná v žurnáli
- Je vytvorený incident podľa BP07
- Zápis sa dopĺňa dodatočne v rámci riešenia incidentu

## Opis Obrazoviek + Validácie


UC0417 je prevažne backendový proces. Tellerovi sa zobrazuje len informačná hláška po ukončení zaúčtovania.

Katalóg hlášok: Info a chybové hlášky, AppLib, ConfluenceIT.

### Hláška po úspešnom vklade

Zobrazuje sa pri vklade vždy 

| **Prvok** | **Hodnota** |
| --- | --- |
| Kód | I025 |
| Typ | Info popup |
| Text | Prosim nezabudnite vziat od klienta peniaze. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška po úspešnom výbere

Zobrazuje sa pri výbere, ak všetko prebehlo v poriadku 

| **Prvok** | **Hodnota** |
| --- | --- |
| Kód | \[OTVORENY BOD: hláška nie je v katalógu\] (pripomienka od vývojára: toto treba zistit aj osatatne co chybaju. Vieme si tam nieco vymysliet ale potom to bude chybat pri testoch) |
| Typ | Info popup |
| Text | Vydajte hotovosť klientovi. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška pri transakcii v statuse T

Zobrazuje sa, keď CBS nepotvrdí zaúčtovanie v stanovenom čase (AT1). Teller môže pokračovať ďalšou transakciou 

| **Prvok** | **Hodnota** |
| --- | --- |
| Kód | \[OTVORENY BOD: hláška nie je v katalógu\] |
| Typ | Info popup |
| Text | Transakcia čaká na spracovanie. Môžete pokračovať ďalšou transakciou. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška pri chybe zaúčtovania

Zobrazuje sa, keď pokyn skončí v trvalom statuse E (AT2).

| **Prvok** | **Hodnota** |
| --- | --- |
| Kód | \[OTVORENY BOD: hláška nie je v katalógu\] |
| Typ | Error popup |
| Text | Transakciu sa nepodarilo zaúčtovať. Chyba bola zaznamenaná a odovzdaná na riešenie. |
| Tlačidlo | OK |
| Akcia po OK | Presmerovanie na homepage |

### Hláška pri zlyhaní zápisu do lokálnej databázy

Samostatná hláška sa nezobrazuje. Tellerovi sa zobrazí štandardná hláška o úspechu podľa typu transakcie, pretože transakcia z jeho aj klientovho pohľadu prebehla správne a teller nemá možnosť do situácie zasiahnuť. Chyba sa rieši incidentom na pozadí podľa BP07. \[OTVORENY BOD: potvrdiť správanie\]

### Hláška pri opakovanom spracovaní počas zatvárania pokladne

Používa sa existujúca schválená hláška **I012** so zoznamom nespracovaných transakcií a možnosťou spracovania jednotlivo alebo hromadne.

### Hláška pri hromadnom vklade

Súhrnná hláška sa nezobrazuje. Po potvrdení a zápise sa transakcia ukončí a teller vidí jednotlivé vklady samostatne v žurnáli

### Rozdiely podľa typu transakcie

| **Typ transakcie** | **Rozdiely** |
| --- | --- |
| UC0411 - Realizácia - Vklad na bežný účet | Štandardný priebeh podľa hlavného toku |
| UC0412 - Realizácia - Vklad na účet v inej mene ako účtu | Štandardný priebeh. Ide o cross currency transakciu, platia povinné polia podľa sekcie API. Vzniká halierové vyrovnanie ako závislý pokyn |
| UC0413 - Realizácia - Vklad - Nefunkčný bankomat | Štandardný priebeh, ale poplatok nevzniká. Namiesto poplatkového pokynu sa vytvára záznam o nevzniknutom poplatku podľa BP08 |
| UC0415 - Realizácia - Hromadný vklad | UC0417 sa spúšťa postupne pre každý vklad, vrátane subtransakcií. Každý vklad má vlastnú sadu pokynov a vlastný konsolidačný kľúč. Pri zlyhaní jedného vkladu systém pokračuje ďalším, viď AT6 |
| Výbery (UC0511, UC0512, UC0515, UC0518) | Štandardný priebeh. Rozdiel je v transakčných kódoch a v znení informačnej hlášky |
| Rozmieňanie (UC701) | Do CBS sa posiela len poplatok. Samotné rozmieňanie sa neúčtuje, ide o zmenu stavu mincovky |

### Validácie

| **#** | **Kontrola** | **Podmienka pre pokračovanie** | **Pri nesplnení** | **Testovateľné cez** |
| --- | --- | --- | --- | --- |
| 1 | Vstupné podmienky | Všetky splnené, teda podpis DSA, override, poplatky | UC0417 sa nespustí | Spustiť realizáciu bez podpísanej dokumentácie |
| 2 | Konsolidačný kľúč | Všetky pokyny transakcie majú rovnakú hodnotu | Chyba prípravy pokynov | Porovnať pokyny jednej transakcie |
| 3 | Identifikátor pokynu | Každý pokyn má jedinečný identifikátor, ktorý sa pri opakovaní nemení | Chyba prípravy pokynov | Vyvolať opakované spracovanie a porovnať identifikátory |
| 4 | Poradie odosielania | Závislé pokyny sa neodosielajú pred potvrdením hlavného | Pokyny zostávajú v statuse S | Simulovať zlyhanie hlavného pokynu |
| 5 | Cross currency polia | Pri konverzii sú vyplnené kurz a sumy na oboch stranách | Trvalá chyba, AT2 | Odoslať cross currency pokyn bez kurzu |
| 6 | Zápis do lokálnej databázy | Všetky zápisy prebehli | AT3 | Simulovať výpadok databázy po potvrdení z CBS |

## API


### Rozhranie

Zaúčtovanie prebieha cez **MW\_APP.AccountBookingRest**, čo je REST rozhranie nad APSRV20. APSRV20 sa nazýva aj AccountBooking, špecifikácia je v MW\_APP.AccountBooking. REST verzia je obálka nad MQ verziou, endpoint `/esb/restapi/midas-account/booking`.

Timeout volania si definuje APSRV20, na strane CashBoxu sa nedefinuje.

**Stav integrácie.** AS400 pripravili predstavu o nastavení a prebieha overovanie. Zatiaľ sa predpokladá, že na APSRV20 sa nebudú musieť robiť zmeny (Tomáš Macháček).

### Request codes

Zdroj: súbor Transakcie.xlsx, záložka Potrebné trans.kódy.

| **Pohyb** | **Request code** |
| --- | --- |
| Vklad na bežný účet v mene účtu | 1 |
| Vklad v inej mene ako mena účtu | podľa typu transakcie |
| Poplatok | 22 alebo 23 |
| Poplatok za mince | 26 |
| Halierové vyrovnanie | 25 alebo 26 |

\[OTVORENY BOD: kompletné mapovanie request codes na typy transakcií podľa súboru Transakcie.xlsx\] (pripomienka od Feriho: toto budeme mapovať tu per detail, ktorý môže nastať? ako si to predstavuje vývoj?)

### Kľúčové polia účtovacieho pokynu

| **Pole (REST JSON path)** | **Hodnota** |
| --- | --- |
| channel\_id | CBX |
| request\_code | Podľa typu pohybu |
| version | 7 |
| reverse\_flag | N, pri storne Y (viď AT4) |
| transaction\_amount | Suma pohybu |
| transaction\_currency | Mena pohybu |
| exchange\_rate | Povinné pri cross currency transakciách |
| debit\_part.bank\_code, account\_prefix, account\_number | Vyplniť, ak je na debetnej strane retailový účet, teda pri výbere |
| credit\_part.bank\_code, account\_prefix, account\_number | Vyplniť, ak je na kreditnej strane retailový účet, teda pri vklade |
| credit\_part.variable\_symbol, specific\_symbol | Variabilný a špecifický symbol, ak sa neposiela endToEndID |
| constant\_symbol | Konštantný symbol, ak sa neposiela endToEndID |
| info\_beneficiary | Popis transakcie, maximálne 140 znakov |
| application\_transaction\_id | Identifikátor pokynu, kontrola duplicitného poslania (BP06) |
| original\_transaction\_id | Pri reverznej transakcii identifikátor pôvodného pokynu |
| consReference | Konsolidačný kľúč, spoločný pre všetky pokyny transakcie (BP06) |
| transaction\_branch | 3-znakový kód pobočky |
| parcial\_payment | N |

**Polia, ktoré sa neplnia:** debit\_part.narrative, credit\_part.narrative, credit\_part.title, payment\_reference, debit\_part.specific\_symbol, debit\_part.variable\_symbol, debit\_part.title, [debit\_part.info](http://debit_part.info), [credit\_part.info](http://credit_part.info), info\_bank.

**Symboly a endToEndID.** Posiela sa buď endToEndID, alebo variabilný, konštantný a špecifický symbol samostatne, z ktorých CBS endToEndID vyskladá. Nikdy nie oboje. Používa sa vždy kreditná strana variabilného a konštantného symbolu, aj pri vkladoch aj pri výberoch.

### Cross currency transakcie

Pri transakciách s konverziou mien sú povinné tieto polia:

| **Pole** | **Obsah** |
| --- | --- |
| exchange\_rate | Kurz použitý na konverziu. Typ kurzu určuje kategória transakcie podľa UC0403 - Príprava - Natypovanie transakcie |
| debit\_part.amount | Suma na debetnej strane |
| debit\_part.currency | Mena debetnej strany |
| credit\_part.amount | Suma na kreditnej strane |
| credit\_part.currency | Mena kreditnej strany |

### Zdrojové tabuľky lokálnej databázy CashBox

| **Tabuľka** | **Použitie v UC0417** |
| --- | --- |
| `as400_values` | Overenie dostupnosti CBS pred odoslaním pokynu, stĺpec `is_online` |

Poznámka: AS400 a CBS označujú ten istý centrálny bankový systém (potvrdil Matúš Radušovský). V texte UC sa používa pojem CBS, názvy technických objektov zostávajú nezmenené.

---

## Mapping

### Zápisy do lokálnej databázy

| **Tabuľka** | **Obsah zápisu** | **Kedy** |
| --- | --- | --- |
| `transaction_journal` | Jeden riadok pre každý pokyn vrátane statusu, transakčného kódu, sumy, meny, účtu a konsolidačného kľúča | Pri vytvorení pokynu so statusom S, aktualizuje sa pri zmene statusu |
| `transaction_journal_cash_balance_internal` | Detail mincovky k transakcii na úrovni jednotlivých nominálov | Po potvrdení hlavného pokynu |
| BRANCH\_JOURNAL | Záznam transakcie | Po potvrdení hlavného pokynu |
| BRANCH\_JOURNAL\_CUSTOMER | Údaje o klientovi k transakcii | Po potvrdení hlavného pokynu |
| BRANCH\_JOURNAL\_OWNER | Údaje o vlastníkovi účtu | Po potvrdení hlavného pokynu |
| BRANCH\_JOURNAL\_CASH\_BALANCE | Mincovka k transakcii. Pri vklade v inej mene ako EUR vznikajú dva záznamy, jeden pre menu transakcie a jeden pre poplatok v EUR, rozlíšené cez identifikátor meny | Po potvrdení hlavného pokynu |
| CASHBOX\_TOTAL\_REALIZED | Zmeny po realizácii | Po potvrdení hlavného pokynu |
| BRANCH\_JOURNAL\_CHARGE | Záznam o poplatku pre poplatkový výpis, aj pri nevzniknutom poplatku podľa BP08 | Po potvrdení príslušného pokynu |

**Poznámka pre vývoj.** Tabuľky BRANCH\_JOURNAL, BRANCH\_JOURNAL\_CUSTOMER, BRANCH\_JOURNAL\_OWNER, BRANCH\_JOURNAL\_CASH\_BALANCE a BRANCH\_JOURNAL\_CHARGE **nie sú v aktuálnom dátovom modeli CashBox**. Je potrebné ich vytvoriť alebo UC zosúladiť s existujúcimi tabuľkami `transaction_journal` a `transaction_journal_cash_balance_internal`. \[OTVORENY BOD: viď sekcia Otvorené otázky\]

### Zápis pri nevzniknutom poplatku

Pri transakciách, kde poplatok nevzniká, sa zapisuje záznam s transakčným kódom 2723 alebo 2724 podľa BP08.

\[OTVORENY BOD: naplnenie jednotlivých polí záznamu potvrdí Matúš Radušovský. Zo štruktúry evidencie poplatkov popísanej v UC701 - Rozmieňanie hotovosti vyplýva, že záznam obsahuje typ poplatku, identifikátor poplatku, sadzbu, sumu poplatku, číslo účtu, názov aplikácie a identifikátor brandu, nie je však určené, aké hodnoty sa plnia pri nulovom poplatku\]

### Halierové vyrovnanie

Halierové vyrovnanie sa odosiela ako samostatný účtovací pokyn previazaný s hlavnou transakciou cez konsolidačný kľúč. Suma, transakčný kód a cieľový účet sa určujú podľa UC0416 - Realizácia - Halierové vyrovnanie. UC0417 zabezpečuje odoslanie, sledovanie statusu a spracovanie chýb tohto pokynu rovnako ako pri ostatných pokynoch.

### Logovanie a auditing

**Žurnál CashBox** 

- Loguje sa celý priebeh UC0417, teda vytvorenie pokynov, odoslanie, prijaté odpovede z CBS, zmeny statusu, pokusy o opakované spracovanie a zápisy do databázy
- Zaznamenáva sa finálny status každého pokynu aj celej transakcie a stav účtovania
- Pri chybe sa zaznamenáva chybový kód a text z CBS
- **Retencia:** žurnál sa uchováva 10 + 1 rok. Po jedenástich rokoch sa záznamy mažú po dňoch

**IT logy.** Zapisujú sa do nástroja typu Splunk. Formát záznamu:

`[DATUM][CAS] [ID_POUZIVATELA] [TYP_LOGU] [KOD_LOGU] - [POPIS_LOGU]`

Typy logov: INFO, ERROR, WARNING, OVERIDE.

Do IT logu sa zapisuje odoslanie a výsledok každého pokynu s typom INFO, chyby z CBS s typom ERROR, zlyhanie zápisu do lokálnej databázy s typom ERROR, pokusy o opakované spracovanie s typom INFO a vytvorenie incidentu s typom ERROR.

\[OTVORENY BOD: retencia IT logov\]

### DSA

Tlač potvrdenky prebieha v DSA **pred** zaúčtovaním (UC0441 - Realizácia - Generovanie dokumentácie a podpisovanie s DSA). UC0417 už len potvrdzuje výsledok spracovania po návrate z DSA do CashBoxu.
