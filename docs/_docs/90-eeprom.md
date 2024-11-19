---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
[![EEPROM programmer](../../assets/eeprom/eeprom-programmer.png "EEPROM programmer"){:width="100%"}](../../assets/eeprom/eeprom-programmer.png)

**WORK IN PROGRESS - WORK IN PROGRESS - WORK IN PROGRESS**

Tutti i progetti descritti in questa pagina sono basati su Arduino, ad evidenza della sua versatilità.

## PROGRAMMATORE

- Nei link mettere il link a Tommy PROM, spiegare  CHE IL MIO PROGRAMMATORE DI EPPROM  di è basata un po sul DIWSEGNO  originale di ben , UN PO' SUL TOMMYPROM
- Verificare anche se è Il programmatore dietro di ben Tom che utilizza lo stesso codice di programmazione del Tommyprom; in realtà mi sembra di ricordare che sia una versione tagliata del tommyprom

La mia prima esperienza con un programmatore di EEPROM risale alla costruzione del computer SAP-1 di Ben Eater e alla realizzazione del programmatore basato sul suo schema e sketch. Questo progetto, molto semplice, permetteva di programmare le EEPROM del microcode, ma la scrittura risultava particolarmente lenta.

Durante lo studio del computer NQSAP di Tom Nisbet e la rivisitazione del suo progetto per creare il computer BEAM, mi ero reso conto che il numero di EEPROM da programmare (quattro anziché due) e la loro dimensione avrebbero ulteriormente aumentato i tempi di programmazione di ogni revisione del microcode. Il programmatore di Ben era funzionale, ma non sfruttava la possibilità delle EEPROM moderne di scrittura a *pagine*, che permette la programmazione di una EEPROM da 32KB in pochi secondi. Ciononostante, continuavo a usare questo programmatore, con tempi di scrittura di circa 90 secondi per ogni EEPROM.

Con il progredire della costruzione dei moduli del BEAM, le riprogrammazioni necessarie ad implementare l'Instruction Set e risolvere i bug del microcode diventavano sempre più frequenti; il tempo trascorso ad attendere il completamento dei cicli di programmazione iniziava a diventare frustrante.

Tom aveva sviluppato un suo programmatore, il <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyPROM</a>, che supporta una varietà di EPROM, EEPROM e memorie Flash. Questo programmatore include un software molto completo e molto veloce grazie al supporto della modalità di scrittura *Page Write.* Una versione ridotta del software del TommyPROM - col semplice scopo di programmare le EEPROM 28C256 - è stata utilizzata da Tom e pubblicata sul suo repository dell'NQSAP.

Non volevo iniziare un nuovo progetto, quello di un nuovo programmatore di EEPROM, mentre stavo ancora lavorando sul BEAM. Tuttavia, a un certo punto ho realizzato che, a medio termine, costruire un programmatore basato sul TommyPROM mi avrebbe fatto risparmiare più tempo rispetto a quello che avrei sprecato continuando ad utilizzare il programmatore basico di Ben.

Il mio programmatore iniziale era basato sul design di Ben Eater, quindi utilizzava i '595 anziché i '164 utilizzati in origine da Tom. In una sezione della documentazione, Tom evidenziava che il suo codice non funzionava sul progetto di Ben e che necessitava di significative modifiche. Se si desideravano i benefici del TommyPROM sull'hardware di Ben, Tom indicava che la strada più facile era quella di non modificare il software, ma di adattare l'hardware rendendolo uguale a quello del TommyPROM.

Nella mia testardaggine, ho deciso di fare un compromesso: modificare solo parzialmente l'hardware sviluppato seguendo il progetto iniziale di Ben e approfittarne per studiare e aggiornare il software di Tom, rendendolo compatibile col mio hardware effettuando il minor numero di modifiche possibili.

Le modifiche minime necessarie rispetto al programmatore originale del SAP-1 sono state le seguenti:

- il pin Write Enable della EEPROM (/WE) non viene più viene controllato dal segnale D13 di Arduino, bensì da A2.
- il pin Chip Enable (/CE) non è più connesso a ground (chip sempre attivo), ma è controllato dal pin A0 di Arduino.
- il pin Output Enable (/OE) non è più connesso allo Shift Register, ma è controllato dal pin A1 di Arduino.

Nello sketch di Ben, D13 controlla il segnale Write Enable. Poiché D13 è internamente connesso al LED integrato su Arduino, che lampeggia al boot, potrebbero verificarsi scritture indesiderate della EEPROM ad ogni accensione del programmatore. Questo non è un problema per lo sketch di Ben, perché ad ogni esecuzione la EEPROM viene completamente programmata, sovrascrivendo eventuali scritture spurie. Tuttavia, poiché il TommyPROM può essere utilizzato anche per effettuare letture da una EEPROM, le scritture spurie iniziali potrebbero causare un problema.

**Vale anche per i 595?** Il pin /OE controllato dagli Shift Register  che nella design del Tommy promnon funziona bene perché OE verrebbe modificato ogni volta che vengono inviati nuovi indirizzi agli shift register 164.

Nel suo progetto, Tom inizialmente utilizzava Shift Register 74LS164, poi dava anche indicazioni per l'uso dei più diffusi 74LS595 (o meglio dire 74HC595?).

Per questi motivi Tom consiglia di effettuare qualche semplice modifica hardware per adattare il design di ben a quello di Tom.

[![Schema del programmatore di EEPROM del computer SAP](../../assets/eeprom/eeprom-ben.png "Schema del programmatore di EEPROM del computer SAP"){:width="100%"}](../../assets/eeprom/eeprom-ben.png)

*Schema del programmatore di EEPROM del computer SAP.*

28C series EEPROMS, like the X28C256, sometimes ship from the factory with Data Protection enabled. Use the UNLOCK command to disable this. See the 28C256 Notes for more information. https://tomnisbet.github.io/TommyPROM/docs/28C256-notes

## Schema

[![Schema del programmatore di EEPROM del computer BEAM](../../assets/eeprom/90-eeprom-schema.png "Schema del programmatore di EEPROM del computer BEAM"){:width="100%"}](../../assets/eeprom/90-eeprom-schema.png)

*Schema del programmatore di EEPROM del computer BEAM.*

## Link utili

- Il video <a href="https://www.youtube.com/watch?v=K88pgWhEb1M" target="_blank">Build an Arduino EEPROM programmer</a> e il repository GitHub <a href="https://github.com/beneater/eeprom-programmer" target="_blank">Arduino EEPROM programmer</a> di Ben Eater.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.
