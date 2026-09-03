# UC0450 \- Mapovacia tabuľka prekliku na vklad

Tabuľka obsahuje mapovanie polí, ktoré vďaka prekliku a dát v ňom vieme predvyplniť v jednotlivých use casoch pre vklad.

| obrazovka vkladu | preklik pole | poznamka |
| --- | --- | --- |
| typ vkladu  | targetSection | mapovanie na jednotlivé UC: - `CASH_DEPOSIT` → <https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19960823/UC0411+-+Realiz+cia+-+Vklad+na+et+v+mene+tu> - `CASH_DEPOSIT` (ano, naozaj, nie je to preklep) → <https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/20001074/UC0412+-+Realiz+cia+-+Vklad+na+et+v+inej+mene+ako+tu> - `CREDIT_CARD_REPAYMENT` (vklad na prechoďák) → <https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19960823/UC0411+-+Realiz+cia+-+Vklad+na+et+v+mene+tu> - `BULK_CASH_DEPOSIT` → <https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/19960880/UC0415+-+Realiz+cia+-+Hromadn+vklad> - `ATM_OUTAGE_CASH_DEPOSIT →`<https://tbsk.atlassian.net/wiki/spaces/DOKIT/pages/20001077/UC0413+-+Realiz+cia+-+Vklad+-+Nefunk+n+bankomat> |
| Priezvisko | lastname | pride ak nemame CCAID. Ak mame CCAID, potrebujeme volat CustomerCBIDetail |
| Meno | firstName | pride ak nemame CCAID. Ak mame CCAID, potrebujeme volat CustomerCBIDetail |
| Titul | title | pride ak nemame CCAID. Ak mame CCAID, potrebujeme volat CustomerCBIDetail |
| RC | birthNumber | pride ak nemame CCAID. Ak mame CCAID, potrebujeme volat CustomerCBIDetail |
| Dátum narodenia | birthDate | pride ak nemame CCAID. Ak mame CCAID, potrebujeme volat CustomerCBIDetail |
| druh dokladu | idCardTypeCode  |  |
| cislo dokladu | idCard |  |
| krajina vystavenia dokladu | idCardCountryCode |  |
| CCAID | ccaIdownerCcaId  | ccaId - ccaid predkladateľaownerCcaId - ccaid vlastníka účtu. V prípade práve v zastúpení je toto ownerCcaId ine ako ccaid predkladateľa |
| CIF |  n/a | pre vklad toto nepride |
| PID |  n/a | pre vklad toto nepride |
| BBAN | account | potrebujeme si dotianut detaily uctu - AccountEnquiryEnt |
| mena | n/a | pre vklad toto nepride |
| hotovost | n/a | pre vklad toto nepride, suma hotovosti |
| vklad | n/a | pre vklad toto nepride, v sume uctu |
| variabilny symbol | n/a | nechodí z prekliku (keď tak len ako súčasť e2e) |
| specificky symbol | n/a | nechodí z prekliku (keď tak len ako súčasť e2e) |
| konstantny symbol | n/a | nechodí z prekliku (keď tak len ako súčasť e2e) |
| referencia platiteľa | e2e |  |
| popis | remittanceInfo |  |
| n/a | isEmployee | nie je policko na formulari, ale potrebujeme pri použití kurzáku |
