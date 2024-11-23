---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
<small>[PROGRAMMATORE](#programmatore) - [Schema](#schema) - [Link utili](#link-utili)</small>

[![EEPROM programmer](../../assets/eeprom/eeprom-programmer.png "EEPROM programmer"){:width="100%"}](../../assets/eeprom/eeprom-programmer.png)

Tutti i progetti descritti in questa pagina sono basati su Arduino, ad evidenza della sua versatilità.

## PROGRAMMATORE

- Nei link mettere il link a Tommy PROM, spiegare  CHE IL MIO PROGRAMMATORE DI EPPROM  di è basata un po sul DIWSEGNO  originale di ben , UN PO' SUL TOMMYPROM
- Verificare anche se è Il programmatore dietro di ben Tom che utilizza lo stesso codice di programmazione del Tommyprom; in realtà mi sembra di ricordare che sia una versione tagliata del tommyprom

La mia prima esperienza con un programmatore di EEPROM risale alla costruzione del computer SAP-1 di Ben Eater e alla realizzazione del programmatore basato sui suoi schema e sketch. Questo progetto, molto semplice, permetteva di programmare le EEPROM del microcode, anche se la scrittura risultava particolarmente lenta. Ciononostante, la programmazione di una EEPROM (dapprima manualmente, poi con il programmatore) è stato un momento euforico - mai avrei pensato di riuscire a comprendere i (tutto sommato semplici) meccanismi che lo rendevano possibile.

Durante lo studio del computer NQSAP di Tom Nisbet e la rivisitazione del suo progetto per creare il computer BEAM, mi ero reso conto che il numero di EEPROM da programmare (quattro anziché due) e la loro [dimensione](../control/#instruction-register-e-istruzioni) avrebbero ulteriormente aumentato i tempi di programmazione di ogni revisione del microcode. Il programmatore di Ben era funzionale, ma non sfruttava la funzionalità di scrittura a *pagine* delle EEPROM moderne, che permette la programmazione di una EEPROM da 32KB in pochi secondi. Ciononostante, continuavo a usare questo programmatore, con tempi di scrittura di circa 90 secondi per ogni EEPROM.

[![Schema del programmatore di EEPROM del computer SAP](../../assets/eeprom/eeprom-ben.png "Schema del programmatore di EEPROM del computer SAP"){:width="100%"}](../../assets/eeprom/eeprom-ben.png)

*Schema del programmatore di EEPROM del computer SAP.*

Con il progredire della costruzione dei moduli del BEAM, le riprogrammazioni necessarie ad implementare l'Instruction Set e risolvere i bug del microcode diventavano sempre più frequenti; il tempo trascorso ad attendere il completamento dei cicli di programmazione iniziava a diventare frustrante.

Tom aveva sviluppato un suo programmatore, il <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyPROM</a>, che supportava una varietà di EPROM, EEPROM e memorie Flash. Questo programmatore includeva un software molto ricco e molto veloce grazie al supporto della modalità di scrittura *Page Write*. Una versione ridotta del software del TommyPROM - col semplice scopo di programmare le EEPROM 28C256 - è stata utilizzata da Tom e pubblicata sul suo repository dell'NQSAP.

Non volevo iniziare un nuovo progetto, quello di un nuovo programmatore di EEPROM, mentre stavo ancora lavorando sul BEAM. Tuttavia, a un certo punto ho realizzato che avrei speso meno tempo nel costruire un programmatore basato sul TommyPROM rispetto al tempo che avrei sprecato continuando ad utilizzare il programmatore elementare di Ben.

[![Schema del programmatore di EEPROM TommyPROM di Tom Nisbet](../../assets/eeprom/TommyPROM-nano-sch.png "Schema del programmatore di EEPROM TommyPROM di Tom Nisbet"){:width="100%"}](../../assets/eeprom/TommyPROM-nano-sch.png)

*Schema del programmatore di EEPROM TommyPROM di Tom Nisbet.*

Il mio programmatore iniziale era uguale a quello sviluppato da Ben Eater e utilizzava i <a href="https://www.ti.com/lit/ds/symlink/sn74hc595.pdf" target="_blank">74HC595</a> anziché i <a href="https://www.ti.com/lit/ds/symlink/sn74ls164.pdf" target="_blank">74HCT164</a> utilizzati in origine da Tom. In una sezione della documentazione, Tom evidenziava che il suo codice non funzionava sul progetto di Ben e che necessitava di significative modifiche. Se si desideravano i benefici del TommyPROM sull'hardware di Ben, Tom indicava che la strada più facile non era quella di modificare il software, bensì di adattare l'hardware rendendolo uguale a quello del TommyPROM.

Nella mia testardaggine, avevo optato per un compromesso: modificare solo parzialmente l'hardware sviluppato nel progetto iniziale di Ben e contemporaneamente approfittarne per studiare e aggiornare il software di Tom rendendolo compatibile col mio hardware ed effettuando il minor numero di modifiche possibili. In quel momento non sapevo che il programmatore rivisitato avrebbe visto la lucce in un tempo sostanzialmente limitato, ma che la corretta implementazione della modalità *Page Write* avrebbe richiesto uno sforzo molto più esteso.

Le modifiche minime necessarie rispetto al programmatore originale del SAP-1 erano le seguenti:

1. il pin Write Enable della EEPROM (/WE) non più controllato dal segnale D13 di Arduino, bensì da A2.
2. il pin Chip Enable della EEPROM (/CE) non più connesso a ground (chip sempre attivo), bensì controllato dal pin A0 di Arduino.
3. il pin Output Enable della EEPROM (/OE) non più connesso allo Shift Register, ma controllato dal pin A1 di Arduino.

Come spiegava Tom, D13 controllava il segnale Write Enable del programmatore di Ben Eater. Poiché D13 è internamente connesso al LED integrato su Arduino, che lampeggia durante il boot, a ogni accensione del programmatore potrebbero verificarsi scritture indesiderate nella EEPROM. Questo non rappresentava un problema per lo sketch di Ben, poiché a ogni esecuzione la EEPROM veniva completamente riprogrammata, sovrascrivendo eventuali scritture spurie. Tuttavia, dato che il TommyPROM poteva essere utilizzato anche solo per leggere una EEPROM, queste scritture iniziali indesiderate avrebbero potuto causare problemi: di qui la necessità di governare /WE con un segnale diverso da D13.

Altro aspetto da tenere in considerazione era il pin /OE delle EEPROM: gli Shift Register  '164 non sono dotati di una memoria intermedia che funga da *latch* durante il caricamento. Di conseguenza, gli output vengono immediatamente modificati ad ogni Rising Edge del clock, sottoponendo /OE a possibili attivazioni indesiderate. Viceversa, i 74HC595 permettono il caricamento seriale dei dati in un latch interno e la erogazione contemporanea di tutti i nuovi stati di output al Rising Edge di un segnale dedicato (RCLK), annullando il rischio di attivazioni indesiderate di /OE. Tuttavia, durante la realizzazione delle modifiche al programmatore non avevo ben compreso questo aspetto e avevo deciso di seguire l'indicazione di Tom, cioè di dedicare un output di Arduino specificamente ad /OE della EEPROM. In altre parole, la modifica al punto 3 si sarebbe potuta evitare, ma in quel momento non l'avevo capito.

28C series EEPROMS, like the X28C256, sometimes ship from the factory with Data Protection enabled. Use the UNLOCK command to disable this. See the 28C256 Notes for more information. https://tomnisbet.github.io/TommyPROM/docs/28C256-notes

## Schema

[![Schema del programmatore di EEPROM del computer BEAM](../../assets/eeprom/90-eeprom-schema.png "Schema del programmatore di EEPROM del computer BEAM"){:width="100%"}](../../assets/eeprom/90-eeprom-schema.png)

*Schema del programmatore di EEPROM del computer BEAM.*

## Link utili

- Il video <a href="https://www.youtube.com/watch?v=BA12Z7gQ4P0" target="_blank">Using an EEPROM to replace combinational logic</a> di Ben Eater, che descrive la programmazione manuale di una EEPROM.
- Il video <a href="https://www.youtube.com/watch?v=K88pgWhEb1M" target="_blank">Build an Arduino EEPROM programmer</a> e il repository GitHub <a href="https://github.com/beneater/eeprom-programmer" target="_blank">Arduino EEPROM programmer</a> di Ben Eater.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.
