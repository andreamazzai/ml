---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
## PLACEHOLDER

[![Registro di output del computer BEAM](../../assets/output/65-beam-output.png "Registro di output del computer BEAM"){:width="100%"}](../../assets/output/65-beam-output.png)

**WORK IN PROGRESS - WORK IN PROGRESS - WORK IN PROGRESS**

Tutti i progetti descritti sono basati su Arduino, ad evidenza della versatilità di questo strumento.

## PROGRAMMATORE

- Nei link mettere il link a Tommy PROM, spiegare  CHE IL MIO PROGRAMMATORE DI EPPROM  di è basata un po sul DIWSEGNO  originale di ben , UN PO' SUL TOMMYPROM
- Verificare anche se è Il programmatore dietro di ben Tom che utilizza lo stesso codice di programmazione del Tommyprom; in realtà mi sembra di ricordare che sia una versione tagliata del tommyprom

La mia prima esperienza su un programmatore di EEPROM risaleva alla costruzione del computer SAP progettato da Ben Eater e alla realizzazione del programmatore basato sul suo schema e sul suo sketch. Questo progetto, molto semplice, permetteva di programmare le EEPROM del microcode, ma risultava particolarmente lento.

Durante lo studio del computer NQSAP di Tom nisbet e la rivisitazione del suo progetto per creare il computer BEAM, avevo realizzato che il numero di EEPROM da programmare (quattro anziché due) e la loro dimensione avrebbero ulteriormente espanso i tempi di programmazione di ogni revisione del microcode. Il programmatore di Ben era funzionale, ma non sfruttava la possibilità delle EEPROM moderne di scrittura di *pagine*, che permette la programmazione di una EEPROM da 32KB in pochi secondi.
Ciononostante, continuavo a usare questo programmatore, con tempi di scrittura di una EEPROM di circa 90 secondi.

Man mano che la costruzione dei moduli del BEAM proseguiva, le ri-programmazioni del microcode si ripetevano sempre più frequentemente per completare l'Instruction Set ed indirizzarne i bug e il tempo speso guardando la fila crescente di puntini sullo schermo durante la programmazione iniziava ad essere seccante.

Tom aveva sviluppato un suo programmatore, il <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyPROM</a>, che supporta una varietà di EPROM, EEPROM e memorie Flash, che include un software molto completo e che è molto veloce perché supporta la modalità di scrittura Page Write; una versione ridotta del software del TommyProm è stata utilizzata da Tom e pubblicata sul suo repository dell'NQSAP per assolvere semplicemente allo scopo di programmarne le EEPROM.

Non volevo iniziare un nuovo progetto - quello del programmatore di EEPROM - mentre stavo ancora lavorando sul BEAM, ma a un certo punto ho realizzato che, alla lunga, costruendo un programmatore basato sul TommyProm avrei speso meno tempo di tutto quello che avrei sprecato utilizzando il programmatore di Ben.

Il mio programmatore iniziale era basato sul design di Ben Eater, dunque con i '595. In una sezione della documentazione, Tom evidenziava che il suo codice non funzionava sul progetto di Ben e che necessitava di significative modifiche: se si desideravano i benefici del TommyProm sull'hardware di ben, Tom indicava che la strada più facile era quella di non modificare il software, ma di modificare l'hardware e renderlo uguale a quello del TommyProm.

Nella mia testardaggine ho voluto fare a metà, cioè modificare solo parzialmente l'hardware inizialmente sviluppato seguendo il progetto di Ben, ed approfittarne per studiare ed aggiornare il software di Tom per renderlo compatibile apportando il minor numero di modifiche possibili all'hardware.

Le modifiche apportate rispetto al programmatore del SAP sono le seguenti:

- il pin Write Enable (/WE) della EEPROM non viene più viene controllato dal segnale D13 di Arduino, bensì da A2.
- il pin Chip Enable (/CE) non è più connesso a ground (chip sempre attivo), ma è controllato dal pin A0 di Arduino.
- il pin Output Enable (/OE) non è più connesso allo shift register, ma è controllato dal pin A1 di Arduino.

Nello sketch di Ben, D13 controlla il segnale Write Enable. Poiché D13 è internamente connesso al LED integrato su Arduino, che al boot lampeggia, significa che si potrebbero verificare delle scritture indesiderate della EEPROM ad ogni accensione del programmatore. Questo non è un problema per lo sketch di Ben, perché ad ogni esecuzione dello sketch la EEPROM viene completamente programmata, sovrascrivendo le eventuali scritture spurie appena descritte. Dal momento che il TommyProm può invece essere utilizzato anche per effettuare sole letture da una EEPROM, le scritture spurie potrebbero causare un problema.

**Vale anche per i 595?** Il pin /OE controllato dagli Shift Register  che nella design del Tommy promnon funziona bene perché OE verrebbe modificato ogni volta che vengono inviati nuovi indirizzi agli shift register 164.

Nel suo progetto, Tom inizialmente utilizzava Shift Register 74LS164, poi dava anche indicazioni per l'uso dei più diffusi 74LS595 (o meglio dire 74HC595?).

Per questi motivi Tom consiglia di effettuare qualche semplice modifica hardware per adattare il design di ben a quello di Tom.

[![Schema del programmatore di EEPROM del computer SAP](../../assets/eeprom/eeprom-ben.png "Schema del programmatore di EEPROM del computer SAP"){:width="100%"}](../../assets/eeprom/eeprom-ben.png)

28C series EEPROMS, like the X28C256, sometimes ship from the factory with Data Protection enabled. Use the UNLOCK command to disable this. See the 28C256 Notes for more information. https://tomnisbet.github.io/TommyPROM/docs/28C256-notes

*Schema del programmatore di EEPROM del computer SAP.*

## Schema

[![Schema del programmatore di EEPROM del computer BEAM](../../assets/eeprom/90-eeprom-schema.png "Schema del programmatore di EEPROM del computer BEAM"){:width="100%"}](../../assets/eeprom/90-eeprom-schema.png)

*Schema del programmatore di EEPROM del computer BEAM.*

## Link utili

- Il video <a href="https://www.youtube.com/watch?v=K88pgWhEb1M" target="_blank">Build an Arduino EEPROM programmer</a> e il repository GitHub <a href="https://github.com/beneater/eeprom-programmer" target="_blank">Arduino EEPROM programmer</a> di Ben Eater.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.


## TO DO

- TBD
