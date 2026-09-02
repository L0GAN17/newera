# 10b eInvoice \- spracovanie došlých faktúr

eInvoice – prišlé faktúry

Všetky faktúry, ktoré prídu od Poštára, je potrebné:

1. Prijať a uložiť XML tak, ako prišlo
2. Zobraziť dáta z faktúry užívateľom + automatická úprava definovaných polí
3. Vyhodnotenie, či je potrebný užívateľský zásah – ak nie, odoslanie do iProc - automatické vyhodnotenie eInvoicom je skôr **požiadavka na rozvoj do budúcna,** aby eInvoice vedel určit, ktoré faktúry pošle priamo do iProcu a ktoré si vyžiadajú zásah spracovateľa. V prvej fáze, by sme to mali radšej nastaviť tak, že každú došlú položku spracovateľ skontroluje a vyhodnotí či je potrebné meniť editovateľné polia alebo ju rovno označí do statusu "na odoslanie" a eInvoice ju odošle.
4. Umožniť obohatenie dát faktúry definovanými dátami
    1. Odloženie faktúry do vyriešenia problému (viď info k systémovým statusom 2,3,5 a 9)
    2. Možnosť Cancelled stavu faktúry, napr.
        1. Aktuálne ak zistíme, že faktúra je duplicitná, tak ju v ABBYY jednoducho vymažeme. V eInvoice faktúry mazať nebudeme, ale musíme mať možnosť zmeniť jej stav na Cancelled alebo Zamietnutá z dôvodu duplicity.
        2. Bankové spojenie je nesprávne – detail ďalej v časti Kontroly
        3. Ak príde faktúra cez ABBYY a následne cez eInvoice a je bez dane, vtedy by mala byť možnosť zamietnuť ju ako duplicitu v eInvoice
    3. Odoslanie do iProc + možnosť označiť v prípade potreby ako Cancelled = tieto nebudú odosielané do iProc a zároveň bude povinné pre spracovateľa k stornovanej položke naplniť pole "Poznámka"
    4. Uložiť / zobraziť log vykonaných zmien voči pôvodnému XML aj s TBčkom spracovateľa, ktorý zmeny vykonal. Zároveň spracovateľ ktorý označí faktúru na odoslanie bude jeho tbčko posielané do iProcu cez abbyy webservice ako Verified by. Užívateľsky prijateľné by bolo vidieť Históriu zmien oproti XML na ľavej strane obrazovky pod Detail došlej faktúry.
    5. Zobrazenia - prehľadný zoznam faktúr + detail každej faktúry + vedieť v zozname filtrovať podľa všetkých stĺpcov, vedieť si zobraziť v prehľade faktúry podľa jednotlivých statusov (napr. ako v eGov) a zároveň vedieť si preddefinovať pre každú záložku podľa statusu vlastný layout stĺpcov (poradie a pridávanie/odoberanie), ktorý vie byť v režime public (teda layout si vie vybrať a editovať hociktorý užívateľ) ale aj v režime personal, kedy daný layout ktorý nadefinujem je k dispozícii iba pre autora layoutu. Layouty sú automaticky prednastavené pri každom zapnutí aplikácii tak ako si ich užívateľ ostatne pre tú ktorú záložku uložil. Logiky farebnej vizualizácie v zmysle priloženého xls. sa vzťahujú tak na zoznam faktúr ako aj na detajl každej faktúry. Zároveň platí, že v detajle faktúry potrebujeme vidieť všetky položky vizualizované v stĺpcoch zoznamu faktúr + Remittance information / Invoice note. 
    6. Obraz faktúry - musí obsahovať všetky dáta, ktoré nám dodávateľ poslal. Z pohľadu užívateľského komfortu preferujeme aby sa obraz faktúry vedel otvoriť ako PDFko na ďalšej obrazovke spracovateľa, automaticky po nakliknutí do detailu došlej faktúry. Takto si bude vedieť vizuálne kontrolovať obraz faktúry oproti Detailu došlej faktúry. Zároveň pri posúvaní sa tabulátorom po editovateľných poliach v Detaile došlej faktúry sa v PDF zobrazení obrazu faktúry podsvieti zdroj dát na žlto. Obraz faktúry musí byť možné otvoriť/zobraziť z aplikácie iProc (DMS linka na preklik)
    7. Generovanie reportov z eInvoice zo zoznamu faktúr vo formáte xls. csv poprípde json.
5. Vedieť vyexportovať faktúru opätovne do iProc aj v prípade, že faktúra mala stastus "zaevidované v iProc" pre prípadnú potrebu opravu vyťažených dát. Pre takúto zmenu je systémovo vyžadované vpísanie Poznámky spracovateľa. Následne spracovateľ bude vedieť zmeniť stav faktúry do stavu "na spracovanie" upraví chybu v dátach a zmení stav "na odoslanie". Vyexportovaná položka dostane v iProcu nové číslo voucher number. Procesne spracovateľ takúto už zaevidovanú faktúru v iProcu vystornuje s dôvodom že išla na opravu vstupných dát.

**Vyťaženie dát – z každého XML od Poštára je potrebné pre iProc vyťažit tieto údaje**:

Viď e-mail od Martin Boledovič zo dňa 09/07/2026 s xls. kde je popísaný abbyy webservice, to by mal byť dobrý vstup na identifikovanie polí z poštára ktoré sú povinné pre zaslanie do iProc. Pre zobrazenie v zozname faktúr ako aj v detajle došlej faktúry viď priložený xls. v tomto dokumente. **Zároveň rozvoj do budúcna**, rozšíriť zobrazenie v  eInvoice o editovateľné polia Dodávateľ-Zamestnanec a Payment reason comment a tiež o K40 zeditovaných polí priamo v eInvoice. Zároveň bude potrebné aj rozšíriť abbyy webservice o tieto polia. Akturálne v prípade faktúr, ktoré zaplatí zamestnanec napĺňame pole Dodávateľ-Zamestnanec priamo v iProc. Vítané by bolo vedieť to už naplniť v eInvoice. To isté platí pre Payment Reason Comment. V kombinácií rozvoja do budúcna K4O na úrovni eInvoice, kde by sme verifikovali zeeditované editovateľné polia, by sme mohli potom v Iprocu skipnúť K4O a to by prinieslo zrýchlenie procesu, nakoľko needitovateľné polia z eInvoicu by museli byť správne a tie editovateľné by podliehali verifikácii a zároveň by ich kontingent bol kompletný.

**Kontroly/automatizovaná úprava dát **– eInvoice

1. Existencia dodávateľa – aplikácia vyhodnotí, či existuje Dodávateľ v iProc/SAP (na základe exportu zo SAP v ODS, kontroluje sa na základe DIČ a IČO:
    1. Dodávateľ neexistuje:
        1. Faktúru je potrebné nechať v rozpracovanom / temporary stave (status 2. dodávateľ nezaevidovaný ) , kým prebehne zadefinovanie dodávateľa  v iProc / SAP / ODS. Overovanie zadania dodávateľa prebehne automaticky v eInvoice vždy o 6:00 a o 12:00. Ak overenie zbehne validne eInvoice pokračuje v priradení statusov ako je popísané v bunke C5 priloženého xls. Rozpracovaný stav si vyžaduje akciu zo strany spracovateľa a po nakliknutí spracovateľom pre uloženie systémovo vyžaduje  vložiť "poznámku spracovateľa" s informáciou o spôsobe riešenia položky.
    2. Dodávateľ existuje:  
        1. Bankové spojenie – stop spracovania
            1. Faktúru je potrebné nechať v rozpracovanom / temporary stave (status 3 BU neexistuje)., kým
                1. prebehne zadefinovanie bank.spojenia iProc / SAP / ODS. Overovanie zadania bankového účtu prebehne automaticky v eInvoice vždy o 6:00 a o 12:00. Ak overenie zbehne validne eInvoice pokračuje v priradení statusov ako je popísané v bunke C5 priloženého xls. Rozpracovaný stav si vyžaduje akciu zo strany spracovateľa a po nakliknutí spracovateľom pre uloženie systémovo vyžaduje  vložiť "poznámku spracovateľa" s informáciou o spôsobe riešenia položky.
                2. Môže byť aj prípad, keď bankové spojenie nie je schválené, lebo v zmluve máme dohodnuté inak a faktúra neobsahuje daň vtedy faktúru dodávateľovi vrátime, aby ju opravil a dal tam správne bankové spojenie.
                3. Prípad Cancelled stavu faktúry ešte v eInvoice
                4. Ak by faktúra so zamietnutným bankovým spojením z titulu zmluvnej dohody o inom bankovom účte obsahovala daň, informujeme dodávateľa aby vystavil DBS a takúto faktúru zaevidujeme s bankovým spojením SK11 1100 0000 0020 0100 3800 a spôsobom úhrady 4. vyúčtovacia faktúra
                5. DBS k tejto faktúre bude spracovaný v zmysle bunky C1\* priloženého xls.
            2. IČ DPH – upozornenie na nesúlad vo výsledkoch automatizovaných kontrol aj so zobrazením IČ DPH z ODSky
            3. Meno / názov – upozornenie na nesúlad vo výsledkoch automatizovaných kontrol aj so zobrazením Meno/názov z ODSky
            4. Adresa – upozornenie na nesúlad vo výsledkoch automatizovaných kontrol aj so zobrazením Adresa z ODSky
            5. Číslo faktúry je duplicitné (v kombinácii s Dodávateľom a dátumom vystavenia faktúry) – stop spracovania 
                1. faktúra dostáva status 4.Duplicita
                2. v tomto prípade posudzujeme oprávnenosť duplicity – napr. Vaculik zvykne navýšiť sumu faktúry dokonca aj po jej zaplatení – jednoducho pošlú novú faktúru na vyššiu sumu s tým istým číslom. Takúto faktúru preradíme to stavu 7. stornované pričom je podmienka preradenia do tohto stavu vložiť "poznámku spracovateľa" a požiadame dodávateľa o vystavenie DBS, alebo ťarchopisu k už pôvodne zaevidovanej faktúre v iProc. V prípade DBS zašle dodávateľ následne aj novú faktúru na vyššiu sumu s novým poradovým číslom.
                3. Alebo UNIQUA nemá poradové číslo faktúry (používajú číslo poistnej zmluvy), oni majú len VS a ten je stále rovnaký – IČO TB. V iProcu máme možnosť dať za číslo faktúry (číslo zmluvy) ešte \* a dátum, čím si faktúry odlišujeme – pred tým ešte posudzujeme obsah – ak je to iná poistná udalosť alebo k inému autu, tak vieme, že ide o „oprávnenú“ duplicitu, aj keď ju iProc vrátil ako duplicitu z dôvodu opakujúceho sa čísla faktúry. Faktúre následne zmeníme status na 6. na odoslanie a bude sa evidovať v iProc.
                4. výsledok pre faktúru typu 384 je vždy status DUPLICITA
            6. Inšpirácia:
                1. Aktuálne ABBY vysvieti (červený flag) polia, kde niečo nesedí
                2. ABBYY si naťahuje dodávateľov k sebe – v prípade malého nesúladu umožní užívateľovi vybrať správneho dodávateľa
            7. zmena dátumu dodania – pri dobropise, ťarchopise alebo pri oprave základu dane – dodávateľ vystaví faktúru s nejakým dátumom dodania, eInvoice bude tento dátum meniť systémovo podľa dátumu prijatia faktúry do eInvoice, obdobnú logiku máme implementovanú v iProc. Táto automatikcá zmena bude zaznamenaná aj v logoch eInvoice.
            8. Ďalšie požadované automatické zmeny editovateľných polí, statusov, vynucovania vpisovania poznámky ako aj logiky pre prednastavenie vyťaženia objednávky, CPA a popisu sú vydefinované v priloženom xls. (detajlný popis v stĺpcoch - Status faktúry, Poznámka spracovateľa, Číslo bankového účtu, Číslo faktúry, Variabilný symbol, Č. pôvodnej faktúry, Dátum daňového dokladu,, Spôsob úhrady, Popis, Číslo NO, Číslo zmluvy, Verified by, Číslo interné)

**Vyhodnotenie, či faktúra môže ísť priamo do iProc**:

Našpecifikujeme rokmi vyvinuté logiky pomáhajúce spracovaniu v iProc / SAP, ktoré umožnia rozlíšiť výnimky na manuálne spracovanie pred odoslaním do iProc

1. dobropis - pozor, je potrebné pamätať na to že DBS s abbyy posielame ako záporné čísla, keďže iProc typ faktúry dobropis pustí iba v zápornej hodnote. Preferujeme aby eInvoice v prípade typu faktúry dobropis prerobila všetky čiastky na záporné hodnoty. Akceptovateľný by bol aj variant dať ako editovateľné polia suma na úhradu, sumu faktúry, základ dane a daň, aby spracovateľ vykonal zmenu na záporné hodnoty pred exportom do iProcu.(z pohľadu chybovosti a prácnosti tento variant nepreferujeme).
2. oprava základu dane - viď hlavne funkcionality popísané v priloženom xls. v stĺpcoch Č. pôvodnej faktúry a Dátum daňového dokladu
3. prenesenie daňovej povinnosti - viď poznámky v priloženom xls. v stĺpci Popis
4. faktúra bola zaplatená iným ako očakávaným spôsobom - viď poznámky v priloženom xls. v stĺpci Spôsob úhrady. 
5. duplicitné číslo faktúry (v kombinácii s dodávateľom) - požadujeme aby ak iProc vráti položku s chybovým hlásením, že v iProcu existuje s daným poradovým číslo faktúry záznam, aby sa takáto položka systémovo preradila do stavu 4.duplicita

**Obohatenie dát – aplikácia musí umožniť upraviť tieto polia faktúry:**

1. Poznámka spracovateľa - pole naplní spracovateľ freetextom podľa potreby / povinné pri zadaní/riešní statusu 7 storno, 2. dodávateľ nezaevidovaný a 3. BU neexistuje a 5. odložené. Zároveň v prípade, že bude potrebný reexport faktúry do iProc zo statusu 10, bude povinné naplniť pole Poznámka spracovateľa. V prípade, že poznámka spracovateľa je už zadaná, a nastane situácia, že spracovateľ mení status faktúry na status vyžadujúci poznámku, bude eInvoice vyzývať spracovateľa na pridanie ďalšej poznámky, poprípade edit existujúcej poznámky. (príklad, faktúra bola v stave 2alebo3, teda spracovateľ zadá poznámku, následne sa preklopí fa do stavu 1 po založení dát v ODSke, ale spracovateľ identifikuje, že faktúru je potrebné dať do stavu 5)
2. Číslo bankového účtu - logika výberu popísaná v priloženom xls.
3. Číslo faktúry - iba pre status 4.duplicita(vyššie popisujem že do toho stavu sa vie dosťať faktúra kontrolou eInvoice ale aj vrátením z iProc, v prípade, žeby nebolo možné nastaviať vrátenu duplicitu z iProc na status 4. a ostávala by v statuse 9.vrátená z iProc, tak požadujeme aby bolo číslo faktúry editovateľné aj v tomto statuse) V ostatných statusoch je číslo faktúry needitovateľné
4. Variabilný symbol - iba v prípade, že variabilný symbol nie je uvedený na faktúre. EInvoice doplní variabilný symbol v prípade absencie ako prvých 10 číslených znakov poradového čísla faktúry a s podmienkou, že variabilný symbol nezačína 0. Ak je variabilný symbol uvedený na faktúre, tak pole je needitovateľné.
5. Č. pôvodnej faktúry - editovateľné pole pre prípad, že dodávateľ údaj nevyplní v správnom poli pre typy faktúr dobropis a ťarchopis (381 a 383). Pole má byť automaticky doťahované. Ak je prázdne tak pre typy faktúr 381 a 383 svieti bunka v zozname aj v prehľade faktúr na červeno. Spracovateľ je v tomto prípade povinný pole naplniť.
6. Dátum dodania - editovateľné pre typ faktúry 381 (dobropis) a 383 (ťarchopis), eInvoicom bude prednastavený dátum prijatia faktúry ako dátum dodania 
7. Spôsob úhrady - zoznamu možností: 1.  No item selected  (preddefinovaný stav ); 2. Platené v hotovosti; 3. Služobná KK; 4. Vyúčtovacia faktúra; 5. Zaplatené zálohou. Automaticky prednatavená je možnosť 1. Pracovník vyberá možnosti 2-5. Na základe ktoréhokoľvek výberu: aplikácia automaticky faktúre zaeviduje č. účtu SK11 1100 0000 0020 0100 3800. Aplikácia vykoná nové kontroly, kontrolu B existencia BU vyhodnotí ako validnú a pokračuje kontrolou C duplicita. Podľa výsledku zmení status na 4 duplicita (ak existuje), alebo 1 na spracovania.   Platí všeobecné pravidlo, ak je faktúre pridelné č. účtu  SK11 1100 0000 0020 0100 3800 aplikácia ignoruje kontrolu BU
8. Základ dane - pre prípady zálohových faktúr 386 je editovateľné
9. Výška dane - pre prípady zálohových faktúr 386 je editovateľné
10. Popis - text z faktúry minimálne z prvej line/item. Ideálna by bola situácia ak by einvoice vedela z faktúry exstrahovať a dolniť podstatu fakturácie. Pracovník dopĺňa iné relevantné texty. Zaujimáva by bola pre nás možnosť pre faktúry od určitých dodávateľov (napr. Telecom) do textu uvádzať aj pole číslo zákazníka resp. adresát (predpokladáme že to bude štandardné pole peppol). Einvoice by v prípade existencie poľa čísla zákazníka/adresát uviedla toto číslo na začiatok Popisu automaticky eInvoicom. **ROZVOJ DO BUDÚCNA** - posielať samostatné pole zákazník/adresát webservisom do iProcu.Taktiež ak príde faktúra s popisom "prenos daňovej povinnosti" resp. typ faktúry AE, tak požadujeme aby v popise v texte bolo na začiatku PDP0DPH V prípade faktúr od ORANGE, potrebujeme do popisu na začiatok textu popisu uvádzať vždy variabilný symbol.
11. Číslo NO - doplní pracovník ak na faktúre identifikuje, ale nie je uvedené v správnom poli, ideálny scenár ak nájde eInvoice v texte číslo 86xxxxxx a to automaticky doplní do poľa Číslo NO
12. Číslo zmluvy - doplní pracovník ak na faktúre identifikuje č.- zmluvy. Ideálne ak eInvoice nájde vo faktúre hodnotu 86xxxxxx v kombinácii so slovným spojením číslo zmluvy alebo CPA. Tak toto číslo doplní do poľa Číslo zmluvy

Čo sa týka K4O pre tieto zmeny – nakoľko kontrola správnosti sa deje až v iProcu, v eInvoice nie je K4O potrebné.**P****ožiadavka na rozvoj do budúcna**, vedieť okrem zmeny editovateľných poli, aj ich verifikovať rôznym užívateľom od toho ktorý ich editoval s príslušnými logmi. V kombinácii s úpravou abbyy webservicu by sme vedeli povedať že tieto faktúry už nie je potrebné na strane iProcu verifkovať čo by vedelo zrýchliť proces.

**Odloženie faktúry do vyriešenia problému**

Ide o status faktúry 2.Dodávateľ nezaevidovaný, 3. BU neexistuje a 5. Odložené. Statusy 2. a 3. si budú vyžadovať pri nakliknutí spracovateľa do Detajlu faktúry vložiť poznámku, z dôvodu toho, že je potrebné zadať request do iProcu na založenie/zmenu kmeňových dát. Takto budeme mať vizualizované v eInvocie, že je daná položka riešená. Následne, keď zbehne založenie a dáta budú preklopené do ODSky tak sa faktúra preklopí do statusu 1. na spracovanie na základe dennej automatickej kontroly ODSky zo strany eInvoice o 6:00 a 12:00. Pre presunutie faktúry do statusu 5. bude povinné vložiť poznámku spracovateľa. Opäť si takto zvizualizujeme príčinu pozastavenia položky.

**Odoslanie faktúry do iProc**

eInvoice automaticky odošle do iProcu všetky faktúry, pri ktorých zmení spracovateľ status na 6. Na odoslanie. Ak eInvoice zašle faktúru, automaticky sa zmení status na 8. odoslané do iProc. Ak je faktúra v Iproc zaevidovaná príde informácia s Interným číslom faktúry v iproc (voucher number) a tá sa zapíše v eInvoice do poľa Číslo interné. Zároveň sa zmení status na 10. zaevidované v iProc. V prípade, že vráti iProc chybu, tá sa zapíše do poľa Chyba z iProc a status sa zmení na 9.vrátené z iProc. Chyby rozlišujem také kedy zlyhá server a také kedy je nejaká kvalitatívna chyba v odoslaných dátach. Pre tie chyby z titulu výpadku servra nastane automatické opakované zasielanie smer iProc v 15minútových intervaloch 3x. Po 3 neúspešných pokusoch ideálne zašle eInvoice notifikáciu na faktury\_a\_upomienky o tom že máme serverový problém a faktúry sú evidované v statuse 9. Špecifická chyba vrátenia z iProc je informácia o duplicite. Tu chceme aby padla faktúra do statusu 4. Duplicita. Tú následne spracovateľ buď preradí do stastusu 7.Storno s povinnosťou vloženia poznámky alebo do status 6.na odoslanie s úpravou por. čísla faktúry Ostatné kvalitativne chyby z iProc si nebudú vyžadovať 3x automaticky odoslať do iProc, tam bude potrebný zásah zo strany spracovateľa. Po úprave zmení status na 6.

  

**Obrazovka – zoznam faktúr**

Prednastavené usporiadanie stĺpcov ako aj zásoba stĺpcov v eInvoice bude v zmysle priloženého xls. Ako vyššie popisujem v dokumente, spracovateľ si bude vedieť vydefinovať vlastnú zásobu stĺpcov aj ich poradie pre každý typ záložky Taktiež je požiadavka vedieť filtrovať podľa jednotlivých stĺpcov v zozname všetkých faktúr, obzvlášť s dôrazom na stav/status faktúry. Chceme mať možnosť preddefinovaných obrazoviek s jednotlivými statusmi/stavmi kvôli prehľadnosti. V skratke budem vidieť na obrazovke separátne položky na spracovanie, následne položky v odložené a podobne v podobe záložiek, kde si navolím čomu sa idem venovať. Pri záložke bude vizualizovaný aj počet otvorených položiek v jednotlivej záložke. 

Statusy pre vytvorenie záložiek: 1. na spracovanie, 2. dodávateľ nezaevidovaný; 3. BU neexistuje; 4. duplicita,  5. odložené; 6. na odoslanie, 7. stornované, 8. odoslané do iProc; 9. vrátené z iProc ,10. zaevidované v iProc , 11. Reexport faktúry a 12. Všetko

  

  

**Obrazovka – detail faktúry **

Zoberieme si ako vzor – čo sa týka zoskupenia/zobrazenia atribútov, logika usporiadania polí – obrazovku z iProcu – máme tzv. cover sheety a ich vizuál sa čo najviac približuje faktúre – sú tam dáta, ktoré zvykneme kontrolovať pri faktúre.

Cover sheet by mohol byť inšpirácia pre obrazovku a zoradenie polí, ktoré z eInvoice pritečú, by mohol byť layout, ktorý máme pre kontrolu v rámci eProcurementu v iProcu

Zároveň platí, že v detajle faktúry potrebujeme vidieť všetky položky vizualizované v stĺpcoch zoznamu faktúr + Remittance information / Invoice note. 


**Obrazovka – log audit**

Potrebujeme vidieť log zmien - história zmien v štruktúre editované pole, pôvodná hodnota, nová hodnota, dátum zmeny, Tbčko toho kto zmenu vykonal v detajle faktúry.

História zmien - užívateľský prijateľné by bolo vidieť ju priamo v detaile faktúry, pod výsledkami automatizovaných kontrol.

Samozrejme musíme mať uchované aj dáta, ktoré prišli (pôvodné xml a z neho vygenerovaný čitateľný obraz faktúry), aj to, ako sme ich zmenili. Ak by sme mohli mať vizualizáciu, ktoré dáta a ako boli zmenené s logom, kto to zmenil, bolo by to super – mohli by sme ľahko ukázať vnútornej kontrole, čo, kedy a kým bolo robené.

Teraz máme poznačené TB toho, kto vyťažoval, takže ak vznikne chyba, vieme, ktorý spracovateľ vyťažoval. Túto funkcionalitu by sme si nechali ako back log, keď urobíme tie hlavné veci, aby sme stihli legislatívne termíny.
