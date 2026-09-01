# UC\-103 \- Zobraziť detail odchádzajúcej faktúry

**IT analýza: <https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/20070752/UC-103+-+Zobrazi+detail+odch+dzaj+cej+fakt+ry>**

**Business potreba:** Pre prípad, že je z nejakého dôvodu potrebné:

- vidieť niektoré / všetky údaje na faktúre odoslanej cez Poštára, resp. na uloženej v eInvoice alebo
- zistiť v akom stave je faktúra alebo
- zistiť, kedy ju Poštár protistrany prijal a pod.

je dôležité mať možnosť vidieť obraz celej faktúry v užívateľsky čitateľnej forme.

UC bude **spustiteľný** priamo v aplikácii eInvoice ako **samostatná položka** alebo bude k dispozícii ako linka na** preklik z inej aplikácie**.

Každý užívateľ má **právo vidieť len „svoje“ faktúry**, t.zn. tie, ktoré prišli do eInvoice cez API alebo aj tie, ktoré boli inputované priamo v eInvoice.

**Cieľ UC**: Poskytnúť elektronický obraz – **detail všetkých vystavených faktúr** s možnosťou tlače / stiahnutia ako súbor (PDF).

UC bude zobrazovať všetky typy faktúr - štandardnú faktúru, dobropis, zálohovú faktúru,...

**Požiadavky na obrazovku:**

- Rozloženie informácií na obrazovke by malo byť inšpirované klasickým formulárom faktúry, nakoľko faktúry si bude zobrazovať viac oddelení – stále vybraní užívatelia, ale viac oddelení.
- Okrem dát, ktoré sú uvedené priamo na faktúre, by mali byť viditeľné aj informácie o:
    - stave faktúry + či, kedy bola odoslaná Poštárovi/ výsledok od Poštára, prípadne, či bola faktúra dobropisovaná, resp. bola k nej v eInvoice vystavená nová faktúra (v prípade problému a dobropisu).
    - „zdroji“ (oblasti) faktúry – napr. BI, AI, SafeBox, faktoring, Prev.účtareň...
- Z tejto obrazovky by malo byť možné:
    - preklik na vystavenie dobropisu ku zobrazenej faktúre
    - možnosť vytlačiť / uložiť ako súbor
- Polia na zobrazenie:
    - číslo faktúry
    - ...

Príklad formulára faktúry:


@Iveta SPIEGELOVÁ@Andrea ANTOLOVÁ  je nejaky rozdiel v zobrazeni inych typov faktur ako napr. dobropis, zálohová faktúra, že by mali byť zobrazené inak?
