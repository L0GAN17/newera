# UC0431 \- Poplatok za vklad \- Stanovenie výšky

## Obsah

## Info

`Otvorené otázky`

~~ak zlyha volanie na FeeBe tak poplatok je 0~~

~~tu je poplatok konstantny~~

~~co ak nezistim id\_fee\_rate v local DB~~

~~zlava sa pocita pre dnes, alebo CBD? - ~~~~CBD je odpoved~~

~~preveriť správanie TI20PF .. ucet vo vynimovniku ale T20\_CHRGT1 \<\> 0..3 - Aktuálne na I6 161 záznamov s hodnotou NULL podla L.T. I6 veľmi podobný RS~~

**~~Ide o bytové domy ktore tam su Poplatok má byť účotvaný  (10 E) ale používa sa nejaká chargegroup ~~**

~~preveriť ako vyzerá ODS\_SA\_TI20PF na zdroji  (I6ICRDLIB.TI20PF ) nevidím dátumy - len patne zaznamy~~

  
~~Zamestnanec ? Platí ! !  ~~  
  
~~Má vplyv CG na výšku poplatku, alebo jedine ak výnimovnik ? - Nemá vplyv~~   


## **Biznis zadanie**

  
**Stanovenie poplatku za vklad hotovosti volaním **[**FeeBe**](https://rbinternational.sharepoint.com/sites/TBSK-EnterpriseArchitecture/AppLib/Forms/AllItems.aspx?id=%2Fsites%2FTBSK%2DEnterpriseArchitecture%2FAppLib%2FCentral%20Banking%20Systems%2FFeeBe&viewid=1328f156%2D9e07%2D4b8d%2D8183%2Dc648a500aaae)** a služby FeeCalc ktorá vráti aplikácií Cashbox výšku poplatku, ktorý je potrebné zaúčtovať**

**Poplatky pri vkladoch : **


`amount, currency, percentage, minimal je "predpokladana" odpoved z FeeBe`

| `ID_RATE` | `ID_FEE` | `ID_RATE_FEE toto zistujeme z local db` | `DESCRIPTION` | `AMOUNT` | `CURRENCY` | `PERCENTAGE_RATE` | `MAXIMAL_RATE` | `MINIMAL_RATE [ EUR ]` | `SPECIAL_RULE_RATE` | `BASE_UNIT` | `VALID_FROM` | `VALID_TO` | `VAT_RATE` | `OWNER_TYPE` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `676` | `191` | `1` | `Vklad hotovosti treťou osobou FO` | `6` | `EUR` |  |  |  |  | `vklad` | `31.3.23` | `31.12.99` | `0` | `FO` |
| `677` | `191` | `2` | `Vklad hotovosti treťou osobou PO` | `6` | `EUR` |  |  |  |  | `vklad` | `31.3.23` | `31.12.99` | `0` | `PO` |

## **Chargetype vysvetlivky : **

`0 - poplatok je prenesený na vkladateľa - npr. Orange`

`1 - Poplatok prenesey na vkladateľa (splátka kreditnej karty, dnes zrušený poplatok, resp. nastavený na hodnotu 0 )`

`2 - Bez poplatku vkladateľa (poplatok prenesený na majiteľa účtu`

`3 - Bez poplatku vkladateľa a neúčtovaný v kapitalizácií (napriklad nejaka organizácia )`

  
`Vzťah k účtu (M,K,V,D) : Majiteľ, Konateľ, Vkladateľ, Disponent`  
`-`


`Aktéri`

- `Systém`
- `Feebe`


## Vstupné podmienky

- BBAN účtu bol overený v UC0401.
- Klient bol identifikovaný v UC0402.
- Je známy CCAID klienta.
- Je známy typ klienta (FO/PO).
- Je známa suma a mena vkladu.
- Sú dostupné lokálne kópie tabuliek TI20PF a FEE\_RATE\_DEFINITION.
- Je dostupná služba ProductCBIAuthorizedSubjectsService.

### Spúšťač

Nadradený UC (napr.  UC0411 - Realizácia - Vklad na účet v mene účtu , UC4012 - Realizácia - vklad na účet v inej mene ako mena účtu, UC - 4015 - Realizácia - Hromadný vklad etc.**  **) zavolá tento UC ako`include UC` po overení klienta


## Hlavný tok

** Hlavný scenár:**

1. Systém overí, výskyt zadaného BBANu (získaný v [UC0401 - Príprava - Overenie čísla účtu](https://tbsk.atlassian.net/wiki/pages/viewpage.action?pageId=19990479&src=contextnavpagetreemode)) v lokálnej kopii  tabuľky ODS\_SA\_TI20PF (BBAN = T20ACNO )  SELECT T20\_CHRGT1 FROM ODS\_SA.TI20PF WHERE T20\_ACNO = '%BBAN%';

      
2. Účet sa nenachádza vo vynimovníku 
    1. \[2.a Účet sa nachádza vo vynimovniku a T20\_CHRGT1 = '0' OR '1'\]
    2. \[2.b. Účet sa nachádza vo vynimovniku a T20\_CHRGT1= '2' OR '3' \] 
    3. \[2.c Účet sa nachádza vo vynimovniku a T20\_CHRGT1 \<\> 0..3\]
3. Systém overí na základe informácií získaných v [UC0402](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19990591/UC0402+-+Pr+prava+-+Overenie+klienta) vzťah klienta k účtu 
    1.  Klient **nemá** vzťah k účtu (M, K, V, D ) 
        1. \[3.a.I Klient má vzťah k účtu\]
    2. Nastaví premennú pre Fee\_ID='191'
    3. Vyvola dopyt na lokalnu kopiu FEE\_RATE\_DEFINITION a zisti ID\_RATE\_FEE na zaklade ID\_FEE (v tomto poplatku je to konstanta 191) a OWNER\_TYPE (FO vs PO). získaného v [UC0402 - Príprava - Overenie klienta ](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19990591/UC0402+-+Pr+prava+-+Overenie+klienta)(To, ci je osoba FO alebo PO tahame z GateGlobal (interface customerCBIDetail),
    4. Systém vytvorí request na FeeBe a vyčká na vrátenie výśky poplatku
4. Feebe vráti response s výškou poplatku za vklad
    1. \[4.a FeeBe je nedostupné, alebo neodpovedá \]
    2. \[4.b. FeeBE vráti chybovú odpoveď\]
    3. \[4.c. FeeBE vráti nekompletnú, alebo nevalidnú odpoveď\]
5. Systém :
    1. nastaví výšku poplatku za vklad podľa hodnoty vrátenej službou FeeBe
    2. Aktualizúje akumulátor poplatkov o tento poplatok 
    3. UC pokračuje UC z ktorého bol vyvolaný UC0431

## Alternatívny tok

 **Alternatívne scenáre:**


**\[2.a Účet sa nachádza vo vynimovniku a T20\_CHRGT1 = '0' OR '1'\]**

- UC pokračuje v bode 3.b


**\[2.b. Účet sa nachádza vo vynimovniku a T20\_CHRGT1= '2' OR '3' \]**

-  Systém nastaví výšku poplatku na 0 
- UC pokračuje UC z ktorého bol vyvolaný UC0431


**\[2.c Účet sa nachádza vo vynimovniku a Chargetype \<\> 0..3\]**

- Systém zapíše technickú chybu do loguINSERT INTO CASHBOX\_LOG (BRANCH\_ID, PERSON\_ID, PERSON\_FULL\_NAME, DEVICE\_NAME, DATE\_TIME, BANK\_DATE, LOG\_TYPE, DETAIL, TRANSACTION\_SEQUENCE\_NUMBER) VALUES ('123', '87654321', 'NOVAK JAN', 'CB001', CURRENT\_TIMESTAMP, CURRENT\_DATE, 'FeeValidation', 'InvalidChargeType;FeeID=191;ChargeType=5', 987654321);
- Systém nastaví výšku poplatku na 0
- UC pokračuje UC z ktorého bol vyvolaný UC0431

  
**\[3.a.I Klient má vzťah k účtu\]**

- Poplatok za vklad nebude vypočítavaný aplikáciou Cashbox.
- Poplatok bude zaúčtovaný v procese kapitalizácie externým systémom
- UC pokračuje UC z ktorého bol vyvolaný UC0431


**\[4.a FeeBe je nedostupné, alebo neodpovedá \]**

- Systém opakuje volanie (max 3 x )
- Ak služba neodpovie, systém ponechá hodnotu poplatku za vklad  0
- systém zapíše technickú udalosť do bizsnis logu INSERT INTO CASHBOX\_LOG (BRANCH\_ID, PERSON\_ID, PERSON\_FULL\_NAME, DEVICE\_NAME, DATE\_TIME, BANK\_DATE, LOG\_TYPE, DETAIL, TRANSACTION\_SEQUENCE\_NUMBER) VALUES ('123', '87654321', 'NOVAK JAN', 'CB001', CURRENT\_TIMESTAMP, CURRENT\_DATE, 'FeeBe', 'CoinDepositFeeWaived;FeeID=191;Reason=FeeBEUnavailable', 987654321);
- Systém vypíše hlášku typu INFO [I037](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19981325/Info+a+chybov+hl+ky)
- UC pokračuje UC z ktorého bol vyvolaný UC0431


**\[4.b. FeeBE vráti chybovú odpoveď\]**

- systém nezíska výšku poplatku
- systém zapíše technickú udalosť do biznis logu logu
- Systém ponechá hodnotu poplatku za vklad  0.
- zobrazí INFO [I037](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19981325/Info+a+chybov+hl+ky)
- UC pokračuje UC z ktorého bol vyvolaný UC0431


\[4.c. FeeBE vráti nekompletnú alebo nevalidnú odpoveď\]

- Systém zapíše technickú udalosť do biznis logu.
- Systém ponechá hodnotu poplatku za vklad  0.
- Systém zobrazí hlášku [INFO I037](https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19981325/Info+a+chybov+hl+ky).
- UC pokračuje UC z ktorého bol vyvolaný UC0431

## Diagram tokov

## Výstupné podmienky

- Bola overená existencia účtu vo výnimovníku `ODS_SA.TI20PF`.
- Bol overený vzťah klienta k účtu prostredníctvom služby `Gate_Global ProductCBIAuthorizedSubjectsService`.
- Bola určená výška poplatku za vklad hotovosti, alebo bolo rozhodnuté, že sa poplatok pri vklade nebude účtovať.
- Hodnota poplatku za vklad bola uložená do akumulátora poplatkov aktuálnej transakcie alebo nastavená na hodnotu `0`.
- Prípadné technické chyby boli zaznamenané do biznis logu
- Riadenie bolo vrátené do UC, z ktorého bol vyvolaný UC0431

## Opis obrazoviek + mapovanie

## API

**Request: na FeeBe a vyčká na vrátenie výśky poplatku**

Vychádzam z jsonu uloženého na  : [TBSK - EnterpriseArchitecture - FeeBeCalcApplyFee\_v1.4.json - All Documents](https://rbinternational.sharepoint.com/sites/TBSK-EnterpriseArchitecture/AppLib/Forms/AllItems.aspx?viewid=1328f156%2D9e07%2D4b8d%2D8183%2Dc648a500aaae&FolderCTID=0x012000BBA17C737C0AAE4CAB0A13CB9A19D0D5&id=%2Fsites%2FTBSK%2DEnterpriseArchitecture%2FAppLib%2FCentral%20Banking%20Systems%2FFeeBe%2F2%20Interface%20Specifications%2FFeeBe%2EFeeAPI%2FFeeBe%2EFeeAPI%5FV1%2E4%2FFeeBeCalcApplyFee%5Fv1%2E4%2Ejson&parent=%2Fsites%2FTBSK%2DEnterpriseArchitecture%2FAppLib%2FCentral%20Banking%20Systems%2FFeeBe%2F2%20Interface%20Specifications%2FFeeBe%2EFeeAPI%2FFeeBe%2EFeeAPI%5FV1%2E4)


### Technické parametre služby


|  |  |
| --- | --- |
| Endpoint | `/services/v1/fee/FeeCalc` |
| HTTP metóda | POST |
| Autentifikácia | Basic Authentication |
| Content-Type | application/json |
| Povinný header | RequestAuditInfo |
| Request objekt | FeeCalcRequest |
| Response objekt | FeeCalcResponse |


### FeeCalcRequest

#### Header - RequestAuditInfo 


|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| AppName | Header | Áno | string (max 50) | Const.CHANEL\_ID | Identifikácia volajúcej aplikácie |
| ChannelID | Header | Áno | string (max 3) | Const.CASHBOX\_GG\_APPNAME | Kanál, cez ktorý je služba volaná (napr. internet banking, batch, API gateway) |
| ClientID | Header | Nie | string (max 22) | nenapĺňame | Identifikácia klienta |
| PostTime | Header | Áno | string (date-time) ISO 8601 datetime | now() (bez nanosekúnd) | Dátum a čas odoslania požiadavky |
| ReferenceID | Header | Nie | string (max 48) | nenapĺňame | Referenčné ID požiadavky pre trasovanie |
| SessionID | Header | Nie | string (max 64) | nenapĺňame | Identifikátor používateľskej relácie |
| Subversion | Header | Áno | int32 | 0 | Verzia alebo subverzia rozhrania |
| UserID | Header | Nie | string (max 10) | nenapĺňame | Identifikátor používateľa |
| WorkstationID | Header | Nie | string (max 20) | nenapĺňame | Identifikátor pracovnej stanice |


#### Body - FeeCalcRequest


|  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| requestID | Body | Áno | string (min 1, max 48) | Cash-Box-{RQID}-{feeID}-{rate-FeeID}-{timestamp} | Jedinečný identifikátor požiadavky |
| facilityID | Body | Nie | string | nenastavuje sa | Identifikátor produktu/služby (facility) |
| feeID | Body | Áno | string (min 1, max 30) | 210 | Identifikátor poplatku, ktorý sa má vypočítať |
| ccaid | Body | Nie | int64 | nenastavuje sa | Interný identifikátor účtu alebo klientského vzťahu |
| rateFeeID | Body | Áno | string (min 1, max 30) | z DB (Fee\_rate\_definition) - (getValidRateID) | Identifikátor sadzby poplatku |
| discountType | Body | Nie | string (min 1, max 12) | "CODT"(mince)/"Percent (rozmieňanie) len ak discount\>0 | Typ zľavy aplikovanej na poplatok |
| discountPercentage | Body | Nie | number (0.01 - 100.00) | len ak \>0 | Percento zľavy |
| feeDate | Body | Nie | date (YYYY-MM-DD) | as400ValuesDTO.getCbd() | Dátum, ku ktorému sa vykonáva výpočet poplatku |
| feeCount | Body | Áno | int32 (min 1) | vždy 1 | Počet jednotiek, na ktoré sa poplatok aplikuje |
| feeBaseAmount | Body | Nie | number (min 0) | Suma mincí / base amount rozmieňania, len ak zadané | Základná suma pre výpočet percentuálneho poplatku |


### FeeCalcResponse


|  |  |  |  |
| --- | --- | --- | --- |
| requestID | Áno | string (min 1, max 48) | Identifikátor požiadavky |
| feeID | Áno | string (min 1, max 30) | Identifikátor vypočítaného poplatku |
| rateID | Áno | string (min 1, max 30) | Identifikátor použitej sadzby |
| amountUnit | Nie | number (min 0.01) | Poplatok za jednu jednotku |
| percentageBeforeDiscount | Nie | number (min 0.01) | Percento pred aplikovaním zľavy |
| currency | Nie | string (presne 3 znaky) | Mena poplatku (napr. EUR) |
| minimalForPercentage | Nie | number (min 0.01) | Minimálna hodnota pri percentuálnom výpočte |
| maximalForPercentage | Nie | number (min 0.01) | Maximálna hodnota pri percentuálnom výpočte |
| baseUnit | Áno | string (min 1, max 50) | Typ základnej jednotky výpočtu |
| taxRate | Nie | number (min 0.01) | Sadzba DPH |
| amount | Nie | number (min 0.00) | Výsledná suma poplatku |
| amountBeforeDiscount | Nie | number (min 0.01) | Suma pred uplatnením zľavy |
| taxBase | Nie | number (min 0.00) | Základ dane |
| tax | Nie | number (min 0.00) | Výška dane |
| percentage | Nie | number (min 0.00) | Použité percento poplatku |
| discountPercentage | Nie | number (0.01 - 100.00) | Percento zľavy |
| discountAmount | Nie | number (min 0.01) | Výška poskytnutej zľavy |
| discountType | Nie | string (min 1, max 12) | Typ aplikovanej zľavy |
