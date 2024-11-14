---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
## PLACEHOLDER

[![Registro di output del computer BEAM](../../assets/output/65-beam-output.png "Registro di output del computer BEAM"){:width="100%"}](../../assets/output/65-beam-output.png)

**WORK IN PROGRESS - WORK IN PROGRESS - WORK IN PROGRESS**

Tutti i progetti descritti sono basati su Arduino, ad evidenza della versatilità di questo strumento.

Il programmatore di EEPROM prende spunto dal semplice progetto sviluppato da Ben Eater per la programmazione del computer SAP e dal TommyPROM sviluppato da Tom Nisbet, che include molte più funzioni. Quest'ultimo è un programmatore che supporta una varietà di EEPROM, Flash ed EPROM e che include un software molto completo; una versione ridotta del software è stata utilizzata da Tom e pubblicata sul suo repository per assolvere semplicemente allo scopo di programmare le EEPROM dell'NQSAP.

Durante la realizzazione del SAP, avevo costruito il programmatore seguendo le istruzioni di Ben Eater. Nello sviluppo della Control Logic del BEAM e nel debug del microcode e dei vari moduli, dotato di ben 4 EEPROM e con una quantità di dato da programmare molto maggiore di quella del SAP, i tempi di programmazione diventavano noiosi (credo di ricordare intorno ai 90 secondi per ogni EEPROM); poiché le ri-programmazioni si ripetevano incessantemente per scrivere il microcode corretto per le varie istruzioni, migliorare le funzionalità e indirizzare i bug, diventavo sempre più intollerante rispetto al tempo speso guardando la fila crescente di puntini sullo schermo durante la programmazione.

Inoltre le dimensioni delle EEPROM del BEAM Erano ben diverse da quelle del sap , per il numero molto maggiore di istruzioni supportate e per il numero di microistruzioni possibili , pari a 16. 

Il programmatore di Tom permette anche la programmazione a partire da un file residente sul proprio computer, che viene trasferito sull'Arduino.

Altro aspetto: la velocità. Il programmatore di Ben era funzionale, ma non sfruttava la possibilità delle EEPROM moderne di scrittura di blocchi, che permette la programmazione di una EEPROM da 32KB in pochi secondi.

28C series EEPROMS, like the X28C256, sometimes ship from the factory with Data Protection enabled. Use the UNLOCK command to disable this. See the 28C256 Notes for more information. https://tomnisbet.github.io/TommyPROM/docs/28C256-notes

Nel suo progetto, Tom inizialmente utilizzava Shift Register 74LS164, poi dava anche indicazioni per l'uso dei più diffusi 74LS595 (o meglio dire 74HC595?).

Il mio programmatore iniziale era basato sul design di Ben Eater, dunque con i '595. In una sezione della documentazione, Tom evidenziava che il suo codice non funzionava sul progetto di Ben e che necessitava di significative modifiche: Se si desideravanoi benefici del Tommy promsull'hardware di ben , probabilmente la strada più facile era quella di modificare l'hardware e renderlo uguale a quello di Tommy anziché modificare il software .

Nella mia testardaggine ho voluto fare a metà , cioè non modificare totalmente l'hardware inizialmente sviluppato seguendo il progetto di ben virgola e approfittarne per mettere il naso sul software e renderlo compatibile col minor numero di modifiche possibili da fare per rendere comunque il programmatore molto veloce.

Verificare la modifica su d 13: Questo segnale viene utilizzato percontrollare il WE, mentre CEE sempre attivo; poiché di 13 è collegato al led di Arduino virgola che al boot lampeggia , significa che si potrebbero verificare delle scritture indesiderate della eprom all'accensione dell'arduino .questo non è un problema per lo sketch di ben , perché il programmatore fa solamente delle scritture e una volta partito il programma programma completamente la promo ; mentre il programmatore Tommy promè anche in grado di effettuare solamente delle letture da una eproma e dunque effettuare delle scritture casuali potrebbe essere problematico .

Altra questione : il pin oe è controllato dagli shift register virgola che nella design del Tommy promnon funziona bene perché o è verrebbe modificato ogni volta che vengono inviati nuovi indirizzi agli shift register

Per questi motivi Tom consiglia di effettuare qualche semplice modifica hardware per adattare il design di ben a quello di Tom.

[![Schema del programmatore di EEPROM del computer SAP](../../assets/eeprom/eeprom-ben.png "Schema del programmatore di EEPROM del computer SAP"){:width="100%"}](../../assets/eeprom/eeprom-ben.png)

*Schema del programmatore di EEPROM del computer SAP.*

![Alt text](image.png)


## Schema

[![Schema del programmatore di EEPROM del computer BEAM](../../assets/eeprom/90-eeprom-schema.png "Schema del programmatore di EEPROM del computer BEAM"){:width="100%"}](../../assets/eeprom/90-eeprom-schema.png)

*Schema del programmatore di EEPROM del computer BEAM.*

## Link utili

- Il video <a href="https://www.youtube.com/watch?v=K88pgWhEb1M" target="_blank">Build an Arduino EEPROM programmer</a> e il repository GitHub <a href="https://github.com/beneater/eeprom-programmer" target="_blank">Arduino EEPROM programmer</a> di Ben Eater.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> di Tom Nisbet.

## TO DO

- TBD
