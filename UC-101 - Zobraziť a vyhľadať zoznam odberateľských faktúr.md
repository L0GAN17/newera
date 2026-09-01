# UC\-101 \- Zobraziť a vyhľadať zoznam odberateľských faktúr

**Business potreba: **Obrazovka by mala poskytnúť užívateľovi **zoznam „jeho“ faktúr** a umožniť vyhľadávať podľa rôznych kritérií (zobrazených stĺpcov) možnosť **vyhľadať konkrétnu faktúru** – zadaním priamo čísla faktúry alebo iných zobrazených informácií, napr. odberateľa. Užívateľ by mal dostať základné informácie faktúry ako aj informáciu, či bola faktúra odoslaná poštárovi a **v akom je stave **– počas odosielania + finálny staus od poštára (( až po status, kedy bola Poštárom protistrany prijatá ak poštár takúto informáciu poskytuje). Vyhľadávacie kritéria by mali byť umožniť vyhľadávanie v intervaloch od-do. (napr. zobrazenie faktúr (podľa čísla) od-do, zobrazenie podľa dátumu dodania od-do).

**Preklikom** by sa mal užívateľ mať možnosť dostať na iné UC:

- Detail faktúry
- Vystavenie dobropisu
- Vygenerovanie PDF

**Cieľ UC**: Poskytnúť zoznam (elektronický obraz) vystavených faktúr, možnosť vyhľadávať konkrétne faktúry podľa informácií uvedených v jednotlivých stĺpcoch prehľadu. Zoznam je určený primárne pre užívateľov z jednotlivých oddelení, ktoré vystavujú faktúry, pričom každý by mal vidieť len „svoje“ faktúry vrátane svojich faktúr a dobropisov inputovaných priamo v eInvoice.

V tomto zozname je potrebné zobrazovať faktúry za posledný polrok? Rok?

Zoznam faktúr** poskytne informácie:**

- Základné dáta faktúry
- Informáciu, či bola faktúra odoslaná Poštárom alebo len uložená
- Informácie o poslednom/aktuálnom stave odoslania Poštárovi
    - Stav – informácia, či Poštár prijímateľa faktúru prijal = prešla jeho kontrolou
    - Dátum a čas tejto informácie

**Požiadavky na obrazovku**:

- Na obrazovke sa zobrazia nasledujúce dáta z faktúry
    - ...
    - ...
- Na obrazovke sa bude dať vyhľadávať podľa každého (Ivetka?) stĺpca
- ? Zobrazené faktúry bude možné zoradiť štandardným spôsobom – od A/Z, ad najväčšej sumy po najnižšiu... (Ivetka – je potrebné? resp. technické možnosti)
- Z konkrétnej faktúry (riadku) sa bude možné prekliknúť:
    - obrazovku detailov faktúry UC103
    - obrazovku s detailami komunikácie s Poštárom = stavy, ktorými faktúra prechádzala + časy od príchodu záznamu z originálnej aplikácie, resp. inputu v eInvoice (po dohode s Poštárom)
    - obrazovku zmien na faktúre v prípade, že bola Poštárom zamietnutá a bolo potrebné niečo zmeniť – zmenené polia + kto a kedy ich menil
    - tlač PDF konkrétnej faktúry
