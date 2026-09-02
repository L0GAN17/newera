Dobre, poradie je sekvenčné. Tu sú sekcie na výmenu.

---

## 1. Otvorené otázky - odstrániť bod o poradí a presunúť medzi vyriešené

```
~~Poradie UC0401 a UC0402~~ → ROZHODNUTÉ: UC0401 - Príprava - Overenie čísla účtu beží ako prvý, UC0402 až po ňom. Nebežia súbežne. Dôvod: UC0402 volá rozhranie ProductCBIAuthorizedSubjects na zistenie vzťahu osoby k účtu a to vyžaduje na vstupe IBAN účtu, ktorý dotiahne UC0401. Ide o zmenu oproti pôvodnej dohode z review 10.8.2026, kde sa predpokladala súbežnosť. [POZNÁMKA: zmenu treba premietnuť aj do UC0401, kde je UC0402 uvedený ako súbežný]
```

## 2. Biznis zadanie - nahradiť odsek Vzťah k susedným UC

```
**Vzťah k susedným UC.** UC0402 rieši osobu, UC0401 - Príprava - Overenie čísla účtu rieši účet.

**Poradie je sekvenčné.** UC0401 beží ako prvý, UC0402 až po ňom. Dôvod: UC0402 volá rozhranie ProductCBIAuthorizedSubjects na zistenie vzťahu osoby k účtu a to vyžaduje na vstupe IBAN účtu, ktorý dotiahne UC0401. Pôvodná dohoda z review 10.8.2026 predpokladala súbežné spracovanie, tá už neplatí.

Nadväzujúci UC0404 - Príprava - Kontrola uskutočniteľnosti pracuje s dátami z oboch UC.

**Volanie rozhrania ProductCBIAuthorizedSubjects.** Rozhranie volá výlučne UC0402, jedenkrát pre celú transakciu. Nadväzujúce UC pracujú s uloženým príznakom vzťahu k účtu a rozhranie znova nevolajú. Platí to aj pre UC0431 - Poplatok za vklad - Stanovenie výšky.
```

## 3. Vstupné podmienky - nahradiť odrážku o IBAN

```
- Prebehol UC0401 - Príprava - Overenie čísla účtu. Číslo účtu je overené a v kontexte transakcie je k dispozícii IBAN, ktorý UC0402 potrebuje na volanie rozhrania ProductCBIAuthorizedSubjects pri zisťovaní vzťahu osoby k účtu
```

## 4. API - nahradiť sekciu Nadväznosť

```
### Nadväznosť

- **Predchádza:** UC0401 - Príprava - Overenie čísla účtu, ktorý overí číslo účtu a dotiahne IBAN. UC0402 beží až po ňom, nie súbežne
- **Vstup:** preklik z aplikácie GATE alebo priamy vstup v CashBoxe, a IBAN účtu z UC0401
- **Výstup:** UC0403 - Príprava - Natypovanie transakcie, následne UC0404 - Príprava - Kontrola uskutočniteľnosti (vklady) alebo UC0504 (výbery). Príznak vzťahu k účtu a typ subjektu využíva UC0431 - Poplatok za vklad - Stanovenie výšky
- **Analogický UC:** UC0502 - Príprava - Overenie klienta (výbery), zatiaľ nevytvorený
- **Možný prekryv:** UC0407 - Príprava - Overenie klienta - manuálne (nedostupný GATE), vlastník Matej Pastucha, stav TODO
```

## 5. Hlavný tok - doplniť vetu do kroku 5

```
5. Systém zavolá rozhranie ProductCBIAuthorizedSubjects (v4) s identifikáciou účtu, teda s číslom účtu vo formáte IBAN prevzatým z UC0401 - Príprava - Overenie čísla účtu. Rozhranie volá výlučne UC0402, jedenkrát pre celú transakciu. Rozhranie vráti zoznam osôb, ktoré majú k danému účtu oprávnenie, spolu s typmi ich oprávnení:
   - Ak rozhranie odpovie, UC pokračuje nasledujúcim krokom
   - Ak rozhranie nie je dostupné, tok pokračuje **AT2**
```

---

## Zmena sa dotýka aj ďalších troch UC

Toto rozhodnutie nezostáva v UC0402. Zmenu poradia treba premietnuť aj inde, inak si budú UC odporovať.

### UC0401 - dve miesta

V sekcii **Biznis zadanie**, odsek Vzťah k susedným UC:

```
**Vzťah k susedným UC.** UC0401 beží ako prvý z prípravných UC. Po ňom nasleduje UC0402 - Príprava - Overenie klienta, ktorý rieši osobu a potrebuje IBAN dotiahnutý v UC0401. Následne beží UC0403 - Príprava - Natypovanie transakcie.
```

V sekcii **API**, odrážka Súbežne:

```
- **Nasleduje:** UC0402 - Príprava - Overenie klienta, ktorý rieši osobu a na volanie rozhrania ProductCBIAuthorizedSubjects potrebuje IBAN dotiahnutý v tomto UC
```

### UC0403 - jedno miesto

V sekcii **Biznis zadanie**:

```
**Vzťah k susedným UC.** UC0403 beží po UC0401 - Príprava - Overenie čísla účtu a UC0402 - Príprava - Overenie klienta, ktoré prebiehajú v tomto poradí za sebou. UC0403 beží pred UC0404 - Príprava - Kontrola uskutočniteľnosti.
```

Pôvodné znenie hovorí *"ktoré prebiehajú súbežne"*.

### UC0501 - kontrola

UC0501 je funkčne zhodný s UC0401 a mal by byť zosúladený. Ak je v ňom uvedená súbežnosť s UC0502, treba to opraviť rovnako.

---

## Ešte zostáva UC0431

Rozhodnutie o poradí to nerieši. UC0431 stále vo vstupných podmienkach vyžaduje dostupnosť služby a vo výstupných tvrdí, že vzťah overil sám. Bez opravy môže vývojár službu naprogramovať dvakrát.

Návrh správy:

```
Ahoj Matus, poradie mame - najprv UC0401 a potom UC0402, uz nie subezne. Dovod je
ze UC0402 vola ProductCBIAuthorizedSubjects a ta potrebuje IBAN z UC0401. Premietam
to do UC0401, UC0402 aj UC0403.

Ostava este UC0431. Ten si v tom protireci - v hlavnom toku pise ze vztah berie
z UC0402 (co je spravne), ale vo vstupnych podmienkach vyzaduje dostupnost sluzby
a vo vystupnych pise ze ju overil cez nu. Ked sa to neopravi tak to vyvojar moze
naprogramovat dvakrat.

Treba tam opravit:
- vstupne podmienky: namiesto dostupnosti sluzby dat ze je zname ci ma osoba
  vztah k uctu, dotiahnute v UC0402
- vystupne podmienky: namiesto "overeny cez sluzbu" dat "prevzaty z UC0402"

Vies to poposunut autorovi UC0431?
```
