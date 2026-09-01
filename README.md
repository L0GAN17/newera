Zapracoval som všetky tri odpovede aj Matúšovo vyjadrenie k limitu. Jedna z odpovedí ale nesedí presne na položenú otázku - píšem o tom na konci.

---

# UC0416 - Realizácia - Halierové vyrovnanie

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

- ~~Zoznamy PrnCodesSD a PrnNoSmallDiffCodes~~ → **VYRIEŠENÉ (Feri):** halierové vyrovnanie sa tlačí len vtedy, keď klientovi transakciu dorovnávame, teda pri zápornom vyrovnaní. Pravidlo tlače sa riadi znamienkom rozdielu, nie zoznamom transakčných kódov. Konfiguračné číselníky PrnCodesSD a PrnNoSmallDiffCodes preto nie sú potrebné a boli z UC odstránené.

- ~~Priradenie kódov 4049 a 4051~~ → **VYRIEŠENÉ (Feri):** kód 4049 s pôvodným TABIS kódom 924 zodpovedá kladnému halierovému vyrovnaniu, kód 4051 s pôvodným TABIS kódom 925 zodpovedá zápornému halierovému vyrovnaniu. Potvrdzuje to aj príklad z TABISu, kde kód 925 znamenal kredit klientovi.

- ~~Výška a výpočet limitu~~ → **VYRIEŠENÉ (Matúš Radušovský):** limit je desaťnásobok minimálneho nominálu meny, v ktorej rozdiel vznikol, nie prepočet na EUR. Vyplýva to z implementácie v TABISe. Podrobnosti sú v BP03.

- ~~Podľa čoho sa rozlišuje spôsob účtovania~~ → **VYRIEŠENÉ (Feri):** rozlišuje sa podľa toho, či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne. Podrobnosti sú v BP06.

- ~~Tlač pri zápornom halierovom vyrovnaní~~ → **VYRIEŠENÉ (Feri):** na potvrdenku sa halierové vyrovnanie dáva vždy tak, aby klientovi sedela suma.

**Otvorené:**

- **Čo vracia funkcia pre minimálny nominál.** Funkcia v TABISe sa volá GetMinNominalFromCurrA, čo naznačuje najmenší fyzický nominál danej meny, nie najnižšiu účtovnú hodnotu. Pri EUR je to jedno, pretože najmenšia minca aj účtovná jednotka je 0,01 a limit vyjde 0,10 v oboch prípadoch. Pri iných menách je rozdiel zásadný. Pri CZK je najmenšia minca 1 CZK, takže limit by bol 10 CZK, kým podľa účtovnej hodnoty 0,01 by bol 0,10 CZK. Treba potvrdiť, ktorú hodnotu funkcia vracia. Odpoveď od Matúša Radušovského. Blokuje dokončenie UC. Dopad: mení sa BP03 a spôsob odvodenia limitu.

- **Porovnanie limitu pri zápornom rozdiele.** V implementácii v TABISe sa porovnáva hodnota drobného rozdielu priamo, bez absolútnej hodnoty. Ak by bola záporná, podmienka prekročenia limitu by nikdy neplatila. Treba potvrdiť, či je hodnota v TABISe vždy ukladaná ako kladná, alebo či sa limit kontroluje len pri kladných rozdieloch. Odpoveď od Matúša Radušovského. Blokuje dokončenie UC. Dopad: mení sa BP03 a kontrola 2 v tabuľke kontrol.

- **Konkrétne naplnenie polí debit_part a credit_part.** Podľa BP06 je známe, ktorá je protistrana v jednotlivých prípadoch, nie je však určené, ako sa to premietne do polí účtovacieho pokynu pre všetky štyri kombinácie znamienka a miesta vzniku rozdielu. Feri odporučil prejsť to s Matejom Pastuchom a overiť, či to skúšal s Magdou Tibenskou. Odpoveď od Mateja Pastuchu, prípadne od Magdy Tibenskej. Blokuje dokončenie UC. Dopad: mení sa tabuľka mapovania polí v sekcii API.

- **Technické rozpoznanie prípadu podľa BP06.** Feri určil biznis kritérium, teda či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne. Nie je však určené, podľa akého konkrétneho údaja alebo príznaku systém tento rozdiel vyhodnotí, teda čo presne vývojár naprogramuje a čo tester nastaví na vyvolanie scenára. Odpoveď od Feriho a Matúša Radušovského. Blokuje dokončenie UC. Dopad: mení sa krok 6 hlavného toku a BP06.

- **Smer účtovania pri zápornom vyrovnaní podľa miesta vzniku rozdielu.** Pri zápornom halierovom vyrovnaní je potvrdené, že klientovi sa dorovnáva a na potvrdenke sa suma zobrazuje tak, aby klientovi sedela. Nie je jednoznačne potvrdené, či sa aj pri zápornom vyrovnaní uplatňuje rozlíšenie protistrany podľa BP06, teda klientsky účet verzus pokladňa, alebo či je záporné vyrovnanie vždy voči klientskemu účtu. Odpoveď od Feriho. Blokuje dokončenie UC. Dopad: dopĺňa sa BP06.

### Biznis zadanie

Zaúčtovať halierové vyrovnanie do CBS.

Halierové vyrovnanie sa vyvolá pri transakciách, kde z prepočtu kurzu pri cross currency vznikne drobný rozdiel. Vzniká, keď banka pracuje s kurzovým lístkom na 6 až 8 desatinných miest a transakcia sa musí zaokrúhliť na najnižšiu účtovnú hodnotu danej meny.

**Kladné a záporné halierové vyrovnanie.** Môže vzniknúť kladné aj záporné halierové vyrovnanie. Buď transakciu klientovi dorovnávame, alebo mu odoberáme. **V oboch prípadoch CashBox vyvoláva účtovanie do CBS** cez rozhranie APSRV20 s príslušnou hodnotou.

Rozdiel medzi kladným a záporným vyrovnaním je v tlači na potvrdenku:

| Znamienko rozdielu | Účtovanie do CBS | Tlač na potvrdenku |
|---|---|---|
| Záporné (klientovi dorovnávame) | Áno | Áno, dáva sa klientovi do potvrdenia o vklade |
| Kladné (klientovi odoberáme) | Áno | Nie |

**Príklad.** Klient chce vložiť presne 1 800 CZK na účet vedený v CZK, ale vkladá eurá. Systém zaokrúhlením určí, že má vložiť 86,41 EUR, aby sa na CZK účet pripísalo 1 800 CZK. Pri prepočte vzniklo halierové vyrovnanie 0,04 CZK. Systém vyvolá účtovanie vkladu a zároveň účtovanie halierového vyrovnania 0,04 CZK.

**Vzťah k susedným UC.** UC0416 sa vyvoláva automaticky po natypovaní transakcie a kontrole uskutočniteľnosti. Pripravený pokyn odovzdáva do UC0417 - Realizácia - Zaúčtovanie transakcie. Transakcia halierového vyrovnania musí byť previazaná s hlavnou transakciou kvôli stornu.

**Smer účtovania.** Smer aj protistrana účtovania závisia od znamienka rozdielu a od toho, či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne. Pravidlá sú v BP06.

*Príklad zo záporného vyrovnania:* klient vyberá pri prepočte 18 700,02 CZK, reálna hodnota je 755,77 EUR. Banka nakredituje 0,02 CZK klientovi, aby transakcia sedela presne.

### Aktéri

- Systém
- Supervízor-Teller (pri prekročení limitu halierového vyrovnania)

---

## Vstupné podmienky

- UC0416 je plne automatický proces. Teller nezadáva žiadne údaje a nepotvrdzuje samostatnú obrazovku
- UC0416 sa vyvoláva automaticky pri:
  - vkladoch s konverziou mien (UC0411 - Realizácia - Vklad na bežný účet, UC0412 - Realizácia - Vklad na účet v inej mene ako účtu)
  - výberoch s konverziou mien (UC0512 - Realizácia - Výber v odlišnej mene ako mena účtu a nadväzujúce)
  - dorovnaní na najnižšiu účtovnú hodnotu danej meny, napríklad JPY na celé jeny
- Hlavná transakcia je natypovaná a prešla kontrolou uskutočniteľnosti (UC0404 - Príprava - Kontrola uskutočniteľnosti pre vklady, UC0504 pre výbery)
- Je známy kurz z kurzového lístka platný pre danú transakciu
- Je známy typ vlastníka účtu (industry code), dotiahnutý v UC0402 - Príprava - Overenie klienta alebo UC0502
- Je známe, či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne (BP06) [OTVORENY BOD: technické rozpoznanie prípadu]

---

## Hlavný tok

1. Systém identifikuje, že pri transakcii vzniká drobný rozdiel medzi skutočnou a účtovanou hodnotou, teda pri konverzii mien alebo pri dorovnaní na najnižšiu účtovnú hodnotu.
2. Systém vypočíta drobný rozdiel podľa BP01.
3. Systém vyhodnotí znamienko rozdielu:
   - Ak je rozdiel nulový, halierové vyrovnanie sa nevytvára a UC končí. Hlavná transakcia pokračuje štandardne
   - Ak je rozdiel záporný alebo kladný, UC pokračuje nasledujúcim krokom
4. Systém overí limit halierového vyrovnania pre danú menu podľa BP03:
   - Ak neprekračuje limit, UC pokračuje nasledujúcim krokom
   - Ak prekračuje limit, tok pokračuje **AT1** a UC následne pokračuje krokom 5
5. Systém určí request code a transakčný kód podľa BP02, teda podľa znamienka rozdielu:
   - Ak je rozdiel kladný, použije request code 25 a transakčný kód 4049
   - Ak je rozdiel záporný, použije request code 26 a transakčný kód 4051
6. Systém určí protistranu účtovacieho pokynu podľa BP06:
   - Ak sa účet dotuje kvôli transakcii, protistranou je klientsky účet
   - Ak sa transakcia očisťuje z pokladne, protistranou je pokladňa
   
   Druhou stranou pokynu je cudzomenový výnosový účet banky podľa meny, v ktorej rozdiel vznikol (viď sekcia API, časť Účty pre halierové vyrovnanie).
7. Systém pripraví účtovací pokyn pre halierové vyrovnanie s parametrami podľa BP04:
   - request code a transakčný kód z kroku 5
   - suma rovná absolútnej hodnote drobného rozdielu v mene, v ktorej rozdiel vznikol
   - debetná a kreditná strana z kroku 6
   - popis transakcie "HALIEROVÉ VYROVNANIE"
   - kurz zhodný s kurzom hlavnej transakcie
   - konsolidačný kľúč zhodný s konsolidačným kľúčom hlavnej transakcie
8. Systém odovzdá pripravený účtovací pokyn spolu s pokynom hlavnej transakcie na zaúčtovanie do UC0417 - Realizácia - Zaúčtovanie transakcie. Halierové vyrovnanie sa odosiela ako samostatný pokyn previazaný s hlavnou transakciou cez konsolidačný kľúč.
9. Systém zaznamená halierové vyrovnanie do žurnálu ako samostatný záznam previazaný s hlavnou transakciou.
10. Systém vyhodnotí, či sa halierové vyrovnanie tlačí na potvrdenku podľa BP05:
    - Ak je rozdiel záporný, teda klientovi transakciu dorovnávame, systém označí halierové vyrovnanie na tlač na vkladový alebo výberový lístok tak, aby klientovi sedela suma
    - Ak je rozdiel kladný, halierové vyrovnanie sa na potvrdenku netlačí
11. Systém ukončí UC.

---

## Alternatívny tok

### AT1 - Halierové vyrovnanie prekročí limit pre danú menu

**Spúšťač:** Systém v kroku 4 hlavného toku zistí, že drobný rozdiel prekračuje limit pre danú menu podľa BP03.
**Platí pre:** vklady aj výbery, všetky meny.
**Krok v hlavnom toku:** krok 4.

1. Systém označí halierové vyrovnanie príznakom vyžadujúcim SPV override s dôvodom prekročenia limitu halierového vyrovnania.
2. Systém nezobrazuje samostatný override popup a nepozastavuje spracovanie UC0416. Príznak sa odovzdáva do realizačného UC hlavnej transakcie.
3. Príznak sa vyhodnocuje spolu s ostatnými príznakmi v rámci kombinovaného SPV override na konci transakcie podľa BP03.
4. UC pokračuje krokom 5 hlavného toku. Pokyn sa pripraví, ale reálne zaúčtovanie prebehne až po potvrdení kombinovaného override.
5. Ak supervízor kombinovaný override potvrdí, hlavná transakcia aj halierové vyrovnanie sa zaúčtujú.
6. Ak supervízor kombinovaný override zamietne, nezaúčtuje sa ani hlavná transakcia, ani halierové vyrovnanie. Transakcia končí neúspešne, rieši to realizačné UC hlavnej transakcie.

### AT2 - Storno hlavnej transakcie

**Spúšťač:** Storno hlavnej transakcie, ku ktorej existuje halierové vyrovnanie.
**Platí pre:** vklady aj výbery.
**Krok v hlavnom toku:** nadväzuje na už zaúčtovanú transakciu, mimo hlavného toku UC0416.

1. Systém identifikuje previazané halierové vyrovnanie cez konsolidačný kľúč.
2. Systém vytvorí reverznú transakciu pre halierové vyrovnanie podľa BP04 s reverzným transakčným kódom, teda 4050 pre kód 4049 a 4052 pre kód 4051, s príznakom reverse_flag = Y a odkazom na identifikátor pôvodnej transakcie halierového vyrovnania.
3. Reverzná transakcia sa odosiela cez rovnaké rozhranie ako pôvodná, teda MW_APP.AccountBookingRest.
4. Storno hlavnej transakcie a storno halierového vyrovnania sa vykonávajú spoločne. Nie je prípustné stornovať len jednu z nich.

### AT3 - Zaúčtovanie halierového vyrovnania zlyhá

**Spúšťač:** Účtovací pokyn halierového vyrovnania neprejde na strane CBS.
**Platí pre:** vklady aj výbery.
**Krok v hlavnom toku:** nadväzuje na krok 8, spracovanie prebieha v UC0417.

1. Chybové stavy, opakovanie odoslania a stavy transakcie nie sú predmetom UC0416.
2. Spracovanie chyby preberá UC0417 - Realizácia - Zaúčtovanie transakcie, ktoré rieši celý životný cyklus účtovania vrátane halierového vyrovnania.
3. Keďže hlavná transakcia a halierové vyrovnanie sú previazané konsolidačným kľúčom, zlyhanie sa vyhodnocuje na úrovni celej transakcie v UC0417.

---

## Biznis pravidlá

### BP01 - Výpočet drobného rozdielu

**Vzorec:** Drobný rozdiel = Skutočná hodnota − Ekvivalentná hodnota

- **Skutočná hodnota** je suma vyplývajúca z kurzu, nezaokrúhlená, kurz na 6 až 8 desatinných miest
- **Ekvivalentná hodnota** je suma pripísaná na účet, zaokrúhlená na najnižšiu účtovnú hodnotu danej meny

**Najnižšie účtovné hodnoty per mena:**

| Mena | Najnižšia účtovná hodnota | Zaokrúhlenie |
|---|---|---|
| AUD, CAD, CHF, CZK, DKK, EUR, GBP, HUF, NOK, PLN, RON, SEK, USD, ZAR | 0,01 | na 2 desatinné miesta |
| JPY | 1 | na celé jednotky |

**Vyhodnotenie:**

| Znamienko rozdielu | Vyhodnotenie |
|---|---|
| Nulový | Halierové vyrovnanie sa nevytvára |
| Záporný | Vytvára sa halierové vyrovnanie, klientovi sa dorovnáva, tlačí sa na potvrdenku |
| Kladný | Vytvára sa halierové vyrovnanie, klientovi sa odoberá, na potvrdenku sa netlačí |

### BP02 - Určenie request code a transakčného kódu

CashBox účtuje cez APSRV20 ako samostatný kanál, preto sa v účtovacom pokyne posiela **request code**, nie TABIS transakčný kód.

Halierové vyrovnanie má vlastné request codes a nové transakčné kódy. Priradenie ku znamienku rozdielu potvrdil Feri. Zdroj kódov: súbor Transakcie.xlsx, záložka Potrebné trans.kódy.

| Znamienko rozdielu | Request code | Nový transakčný kód | Reverzný kód | Pôvodný TABIS kód |
|---|---:|---:|---:|---:|
| Kladné (klientovi odoberáme) | 25 | 4049 | 4050 | 924 |
| Záporné (klientovi dorovnávame) | 26 | 4051 | 4052 | 925 |

Oba kódy majú Trans type 90007 (SMALL DIFFERENCES) a v zozname sú označené typom D/W, teda platia pre vklady aj výbery.

Priradenie potvrdzuje aj príklad z TABISu v sekcii API, kde kód 925 zodpovedal zápornému halierovému vyrovnaniu s kreditom klientovi.

**Typ vlastníka účtu.** Pri halierovom vyrovnaní sa transakčný kód nedelí podľa typu vlastníka účtu ani podľa spôsobu úhrady poplatku. Toto delenie sa týka hlavnej transakcie, nie halierového vyrovnania.

Industry code sa naďalej dotiahne v UC0402 - Príprava - Overenie klienta z rozhrania GateGlobal.CustomerCBIDetail, element `industryCode`, a používa sa pri určení transakčného kódu hlavnej transakcie. Hodnota 800 označuje živnostníka, ostatné hodnoty vrátane 143 (zamestnanec TB) označujú bežný účet.

### BP03 - Limit halierového vyrovnania a SPV override

Limit slúži ako ochrana proti chybe, aby nevznikol abnormálny rozdiel.

**Výpočet limitu.** Limit je desaťnásobok minimálneho nominálu meny, v ktorej drobný rozdiel vznikol. Zdroj: implementácia v TABISe poskytnutá Matúšom Radušovským. Limit sa nepočíta v EUR ani sa naň neprepočítava.

Pravidlá vyplývajúce z implementácie:

| Pravidlo | Detail |
|---|---|
| Mena | Minimálny nominál sa zisťuje pre menu transakcie, nie pre EUR |
| Koeficient | Desaťnásobok minimálneho nominálu |
| Presnosť porovnania | Na 2 desatinné miesta |
| Podmienka prekročenia | Limit je prekročený, len ak je rozdiel väčší ako limit. Rovnosť limit neprekračuje |
| Dôsledok | Pridá sa príznak do zoznamu položiek vyžadujúcich override, transakcia sa neblokuje |

**Zdroj minimálneho nominálu v CashBoxe.** Systém odvodí minimálny nominál z číselníka `cl_denomination` ako najnižšiu hodnotu stĺpca `denomination` pre danú menu, kde `is_enabled = true`. Koeficient 10 je konfiguračný parameter.

[OTVORENY BOD: či sa má použiť najmenší fyzický nominál alebo najnižšia účtovná hodnota. Pri EUR je výsledok rovnaký, pri ostatných menách nie. Viď Otvorené otázky]

**Limit pre EUR.** Najmenší nominál je 0,01 EUR, limit je teda 0,10 EUR. Zodpovedá to dnešnému stavu.

**Poznámka k dôvodu pravidla.** Matúš Radušovský uviedol, že dôvod pre koeficient 10 mu nie je známy. Ide o prevzatú logiku z TABISu.

**Správanie pri prekročení limitu:**

- Prístup s flagovaním a kombinovaným override zodpovedá implementácii v TABISe, kde sa príznak pridáva do zoznamu položiek na override a samostatné blokujúce volanie supervízora je v kóde zakomentované
- Systém prekročenie limitu neblokuje, ale označí príznakom vyžadujúcim SPV override
- Override sa nepotvrdzuje samostatne v UC0416. Príznak vstupuje do kombinovaného SPV override, ktorý sa potvrdzuje jedenkrát na konci transakcie v realizačnom UC hlavnej transakcie, zhodne s princípom uplatneným pri výberoch v UC0503 - Príprava - Natypovanie transakcie
- Override potvrdzuje supervízor s oprávnením na potvrdenie transakcie. Používateľ nemôže potvrdiť transakciu sám sebe (E029) a bez oprávnenia z IDM potvrdenie nie je možné (E030)
- Pri zamietnutí kombinovaného override sa nezaúčtuje ani hlavná transakcia, ani halierové vyrovnanie
- Reálne zaúčtovanie hlavnej transakcie aj halierového vyrovnania prebehne až po potvrdení všetkých overridov

### BP04 - Spôsob odoslania do CBS a previazanie transakcií

- Halierové vyrovnanie sa odosiela do CBS ako samostatný účtovací pokyn, nie ako súčasť správy hlavnej transakcie. Dôvod: rozhranie APSRV20 (MW_APP.AccountBookingRest) neumožňuje grupovanie viacerých pohybov do jednej správy
- Hlavná transakcia a halierové vyrovnanie sú previazané cez konsolidačný kľúč (consReference), obe majú rovnakú hodnotu. Previazanie je nevyhnutné kvôli stornu
- Halierové vyrovnanie má rovnaký kurz (exchange_rate) ako hlavná transakcia
- Halierové vyrovnanie má vlastný identifikátor transakcie (application_transaction_id)
- Odoslanie oboch pokynov, ich poradie a spracovanie chybových stavov rieši UC0417 - Realizácia - Zaúčtovanie transakcie
- Pri storne sa použije reverzný transakčný kód 4050 alebo 4052 s príznakom reverse_flag = Y a odkazom na identifikátor pôvodnej transakcie

### BP05 - Logika tlače na potvrdenku

- Halierové vyrovnanie sa tlačí na vkladový alebo výberový lístok ako samostatná riadková položka
- **Tlačí sa len vtedy, keď klientovi transakciu dorovnávame**, teda pri zápornom halierovom vyrovnaní. Kladné halierové vyrovnanie sa na potvrdenku netlačí, hoci sa účtuje do CBS (potvrdil Feri)
- Na potvrdenku sa halierové vyrovnanie dáva vždy tak, **aby klientovi sedela suma** (potvrdil Feri)
- Pravidlo tlače sa riadi výlučne znamienkom rozdielu. Zoznamy transakčných kódov PrnCodesSD a PrnNoSmallDiffCodes, ktoré sa používali v TABISe, nie sú pre CashBox potrebné
- Logika tlače je potvrdená právnym útvarom

### BP06 - Protistrana účtovacieho pokynu

Protistrana účtovania závisí od toho, či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne (potvrdil Feri).

| Prípad | Protistrana | Popis |
|---|---|---|
| Účet sa dotuje kvôli transakcii | Klientsky účet | Rozdiel sa účtuje voči účtu klienta. Pri kladnom vyrovnaní sa klientsky účet debetuje |
| Transakcia sa očisťuje z pokladne | Pokladňa | Rozdiel sa preúčtuje priamo na pokladňu, teda pokladňa proti cudzomenovému výnosovému účtu banky |

Druhou stranou účtovacieho pokynu je v oboch prípadoch cudzomenový výnosový účet banky podľa meny, v ktorej rozdiel vznikol (viď sekcia API, časť Účty pre halierové vyrovnanie).

Pri **zápornom** halierovom vyrovnaní sa klientovi dorovnáva a suma sa na potvrdenke zobrazuje tak, aby klientovi sedela (BP05).

[OTVORENY BOD: podľa akého konkrétneho údaja alebo príznaku systém vyhodnotí, o ktorý z dvoch prípadov ide]

[OTVORENY BOD: či sa rozlíšenie protistrany podľa tohto pravidla uplatňuje aj pri zápornom halierovom vyrovnaní, alebo je záporné vyrovnanie vždy voči klientskemu účtu]

---

## Diagram tokov

[OTVORENY BOD: diagram bude doplnený]

---

## Výstupné podmienky

**Úspech:**

- Halierové vyrovnanie je pripravené ako samostatný účtovací pokyn a odovzdané na zaúčtovanie do UC0417 - Realizácia - Zaúčtovanie transakcie
- Halierové vyrovnanie je zaúčtované voči cudzomenovému výnosovému účtu banky podľa meny, v ktorej rozdiel vznikol, pričom protistrana je určená podľa BP06
- Je použitý správny transakčný kód podľa znamienka rozdielu, teda 4049 pri kladnom a 4051 pri zápornom vyrovnaní
- Halierové vyrovnanie je zaznamenané v žurnáli ako samostatný záznam previazaný s hlavnou transakciou cez konsolidačný kľúč
- Halierové vyrovnanie a hlavná transakcia majú rovnaký kurz a rovnaký konsolidačný kľúč
- Ak je rozdiel záporný, halierové vyrovnanie je označené na tlač na vkladový alebo výberový lístok tak, aby klientovi sedela suma

**UC sa neaktivuje:**

- Ak je drobný rozdiel nulový, hlavná transakcia pokračuje štandardne bez halierového vyrovnania

**Zlyhanie:**

| Druh zlyhania | Čo sa nezapísalo | Ďalší postup |
|---|---|---|
| Supervízor zamietol kombinovaný override | Nezaúčtuje sa hlavná transakcia ani halierové vyrovnanie | Transakcia končí neúspešne, rieši realizačné UC |
| Zaúčtovanie zlyhalo na strane CBS | Závisí od stavu transakcie | Stav transakcie a ďalšie spracovanie rieši UC0417 - Realizácia - Zaúčtovanie transakcie |

---

## Opis obrazoviek + Validácie

UC0416 je plne automatický proces bez samostatnej obrazovky. Teller počas tohto UC nezadáva žiadne údaje.

**Zobrazenie tellerovi:** ak halierové vyrovnanie prekročí limit podľa BP03, príznak sa premietne do kombinovanej override obrazovky na konci transakcie v realizačnom UC hlavnej transakcie. Halierové vyrovnanie je tam uvedené ako jedna z položiek vyžadujúcich potvrdenie supervízora. Samostatná obrazovka pre halierové vyrovnanie neexistuje.

**Zobrazenie klientovi:** záporné halierové vyrovnanie sa zobrazuje na vkladovom alebo výberovom lístku ako samostatná riadková položka s popisom "HALIEROVÉ VYROVNANIE", sumou a menou podľa BP05, a to tak, aby klientovi sedela suma. Kladné halierové vyrovnanie sa klientovi nezobrazuje.

### Tabuľka kontrol

| # | Kontrola | Kde beží | Podmienka pre pokračovanie | Pri nesplnení | Testovateľné cez |
|---|---|---|---|---|---|
| 1 | Znamienko rozdielu | CashBox lokálne | Rozdiel je rôzny od nuly | Halierové vyrovnanie sa nevytvára, koniec UC | Zadať transakciu bez konverzie meny |
| 2 | Limit halierového vyrovnania | CashBox lokálne | Rozdiel nie je väčší ako desaťnásobok minimálneho nominálu meny (BP03), porovnanie na 2 desatinné miesta | Príznak pre kombinovaný SPV override, AT1 | Zadať transakciu s kurzom, ktorý vygeneruje rozdiel presne na limite, teda bez override, a rozdiel tesne nad limitom, teda s override |
| 3 | Výber transakčného kódu | CashBox lokálne | Pri kladnom rozdiele kód 4049, pri zápornom kód 4051 (BP02) | Nesprávne zaúčtovanie | Vyvolať kladné aj záporné vyrovnanie a overiť odoslaný kód |
| 4 | Existencia cieľového účtu | CashBox lokálne | Pre menu existuje cudzomenový výnosový účet | Mena nie je podporovaná pre halierové vyrovnanie | Zadať transakciu v mene mimo zoznamu podporovaných mien |
| 5 | Určenie protistrany | CashBox lokálne | Je vyhodnotené, či sa účet dotuje kvôli transakcii, alebo sa transakcia očisťuje z pokladne (BP06) | Chyba prípravy pokynu | Porovnať pripravený pokyn s očakávanou protistranou |
| 6 | Zhoda kurzu | CashBox lokálne | Kurz halierového vyrovnania sa zhoduje s kurzom hlavnej transakcie | Chyba prípravy pokynu | Interná kontrola, testovateľné cez porovnanie pokynov |
| 7 | Zhoda konsolidačného kľúča | CashBox lokálne | Konsolidačný kľúč sa zhoduje s kľúčom hlavnej transakcie | Chyba prípravy pokynu | Interná kontrola, testovateľné cez porovnanie pokynov |
| 8 | Tlač na potvrdenku | CashBox lokálne | Halierové vyrovnanie je označené na tlač len pri zápornom rozdiele | Nesprávny obsah potvrdenky | Vyvolať kladné vyrovnanie a overiť, že sa na potvrdenke nezobrazuje |

---

## API

Halierové vyrovnanie sa účtuje cez **MW_APP.AccountBookingRest**, čo je REST rozhranie nad APSRV20, rovnako ako hlavná transakcia. Odoslanie zabezpečuje UC0417 - Realizácia - Zaúčtovanie transakcie.

### Request codes a transakčné kódy

Zdroj: súbor Transakcie.xlsx, záložka Potrebné trans.kódy. Priradenie ku znamienku rozdielu potvrdil Feri.

| Znamienko rozdielu | Request code | Nový transakčný kód | Reverzný kód | Pôvodný TABIS kód | Trans type | Typ |
|---|---:|---:|---:|---:|---:|---|
| Kladné | 25 | 4049 | 4050 | 924 | 90007 | D/W |
| Záporné | 26 | 4051 | 4052 | 925 | 90007 | D/W |

**Poznámka k TABIS kódom.** Kódy 214, 283, 2707, 2710 a 327 uvádzané v predchádzajúcich verziách tohto UC sú TABIS transakčné kódy **hlavnej transakcie** pri vklade alebo výbere v inej mene, nie kódy halierového vyrovnania. CashBox účtuje cez APSRV20 ako samostatný kanál a používa request codes.

### Mapovanie polí účtovacieho pokynu

| Pole (REST JSON path) | Hodnota pre halierové vyrovnanie |
|---|---|
| channel_id | CBX |
| request_code | 25 pri kladnom, 26 pri zápornom rozdiele podľa BP02 |
| version | 7 |
| reverse_flag | N, pri storne Y (viď AT2) |
| transaction_amount | Absolútna hodnota drobného rozdielu |
| transaction_currency | Mena, v ktorej rozdiel vznikol |
| exchange_rate | Rovnaký ako pri hlavnej transakcii |
| debit_part / credit_part | Podľa BP06. Jednou stranou je cudzomenový výnosový účet banky podľa meny, druhou stranou je klientsky účet alebo pokladňa |
| credit_part.value_date | Dátum účtovania |
| info_beneficiary | HALIEROVÉ VYROVNANIE |
| application_transaction_id | Vlastný identifikátor transakcie halierového vyrovnania |
| original_transaction_id | Pri reverznej transakcii identifikátor pôvodnej transakcie, inak sa neposiela |
| consReference | Rovnaký ako pri hlavnej transakcii, previazanie pohybov |
| transaction_branch | 3-znakový kód pobočky |
| parcial_payment | N |

[OTVORENY BOD: konkrétne naplnenie polí debit_part a credit_part pre všetky kombinácie znamienka rozdielu a protistrany podľa BP06. Prejsť s Matejom Pastuchom, prípadne overiť s Magdou Tibenskou]

### Účty pre halierové vyrovnanie

Systém účtuje halierové vyrovnanie voči cudzomenovému výnosovému účtu banky podľa meny, v ktorej rozdiel vznikol. Všetky účty majú rovnakú štruktúru a líšia sa len menou.

**Štruktúra:** `001-000001-{MENA}-7212000-2`, názov účtu "ZAOKRUHL.-HAL.VYROV."

| Mena | Branch | CIF | Account Code | Sekvencia |
|---|---|---|---|---:|
| AUD, CAD, CHF, CZK, DKK, EUR, GBP, HUF, JPY, NOK, PLN, RON, SEK, USD, ZAR | 001 | 000001 | 7212000 | 2 |

### Príklad reálnej transakcie z TABISu

Príklad slúži na ilustráciu mechanizmu. V TABISe generovalo halierové vyrovnanie AS400 automaticky, v CashBoxe ho posiela CashBox ako samostatný pokyn.

**Scenár:** klientka vložila 18 700 CZK na CZK účet, následne zrealizovala výber hotovosti z CZK účtu v cudzej mene 755,77 EUR, čo zodpovedá debetu 18 700,02 CZK. Vzniklo halierové vyrovnanie 0,02 CZK so smerom Credit.

| Transakcia | TXN code | Trans type | Post amount | Exchange rate | Movement category | Ref |
|---|---:|---|---:|---:|---|---|
| Pôvodný vklad 18 700 CZK | 200 | 11000 VKLAD HOTOVOSTI | 18 700,00 CZK | 1,00000000 | Csh310 | VP25120360166504 |
| Výber 755,77 EUR z CZK účtu | 327 | 32000 VÝBER Z ÚČTU V INEJ MENE | 18 700,02 CZK | 24,74300000 | Csh320 | VP25120360162192 |
| Halierové vyrovnanie 0,02 CZK | 925 | 90007 SMALL DIFFERENCES | 0,02 CZK | 24,74300000 | Csh910 | VP25120360162192 |

Halierové vyrovnanie má rovnakú referenciu a rovnaký kurz ako hlavná transakcia, ale inú movement category a smer platby Credit. Použitý TABIS kód 925 zodpovedá zápornému vyrovnaniu, ktorému v CashBoxe zodpovedá request code 26 a transakčný kód 4051.

---

## Mapping

Halierové vyrovnanie sa v databáze CashBox ukladá ako samostatný záznam v tabuľke `transaction_journal`, previazaný s hlavnou transakciou cez konsolidačný kľúč.

| Stĺpec | Zdroj v UC0416 |
|---|---|
| `account` | Cudzomenový výnosový účet podľa meny, napríklad 001-000001-CZK-7212000-2 |
| `amount` | Absolútna hodnota drobného rozdielu |
| `currency_id` | Mena, v ktorej rozdiel vznikol |
| `transaction_code` | 4049 pri kladnom rozdiele, 4051 pri zápornom rozdiele podľa BP02 |
| `var_data` | Konsolidačný kľúč hlavnej transakcie, kurz, skutočná a ekvivalentná hodnota z výpočtu podľa BP01, protistrana podľa BP06, popis "HALIEROVÉ VYROVNANIE" |
| `spv_override` | Y ak bol vyhodnotený príznak pre override podľa BP03, inak N |

**Zdrojová tabuľka pre výpočet limitu:**

| Tabuľka | Stĺpce | Použitie v UC0416 |
|---|---|---|
| `cl_denomination` | `currency_id`, `denomination`, `is_enabled` | Odvodenie minimálneho nominálu pre výpočet limitu podľa BP03 |

---

## Poznámky k zapracovaniu - pre teba, nekopírovať do UC

### Čo sa zmenilo

| Odpoveď | Dopad na UC |
|---|---|
| 4049 = kladné, 4051 = záporné | BP02 doplnené, otvorený bod uzavretý, krok 5 hlavného toku konkretizovaný, doplnená kontrola 3 do tabuľky kontrol |
| Rozlíšenie podľa dotácie účtu verzus očistenia pokladne | BP06 prepísané, krok 6 hlavného toku, vstupná podmienka |
| Na potvrdenku vždy tak, aby klientovi sedela suma | BP05 doplnené, krok 10 hlavného toku, Opis obrazoviek |
| Limit je per mena, desaťnásobok minimálneho nominálu (Matúš) | BP03 kompletne prepísané, výklady A a B odstránené, doplnená zdrojová tabuľka do sekcie Mapping |

### Jedna vec, na ktorú upozorňujem

**Tretia odpoveď nesedí presne na otázku.** Pýtal si sa, či sa aj pri zápornom halierovom vyrovnaní uplatňuje rozlíšenie protistrany, teda klientsky účet verzus pokladňa, alebo je záporné vždy voči klientskemu účtu. Feriho odpoveď hovorí o **potvrdenke**, nie o účtovaní: *"áno do potvrdka sa to vždy dáva tak, aby klientovi sedela suma"*.

Zapracoval som to do BP05 a do Opisu obrazoviek, kam to vecne patrí. Otázku na účtovanie som ale ponechal ako otvorenú, pretože odpoveď ju nerieši.

Ak to chceš dotiahnuť, stačí krátko:

```
Ahoj Feri, este raz k tomu zapornemu halierovemu - odpovedal si mi ohladom potvrdenky a to mam zapracovane, dakujem. Ale isiel som este na to uctovanie: pri kladnom rozlisujeme ci sa uctuje voci klientskemu uctu alebo voci pokladni. Plati to iste rozlisenie aj pri zapornom, alebo ide zaporne vzdy voci klientskemu uctu?
```

je to pri oboch, závisí, či debetujeme kreditujeme klientský účet alebo pokladňu - podľa toho kde to vznikne, tak tomu rozumiem ja, vždy je cieľom dorovnať účtovne transakciu po zaokrúhlení. Príklad, ktorý som Ti dal vtedy bolo, že kreditujeme klientov účet, lebo sme mu debetovali pri výbere o 0,02 CZK viac a teda sme potrebovali nadotovať klienta o 0,02 ako halierové vyrovnanie a vtedy ide účet halier. vyrovnanie - klient. Ak by klient vkladal a vznikne niečo také na strane pokladne robím kredit/debet pokladňa vs. účet halierové vyrovnanie
