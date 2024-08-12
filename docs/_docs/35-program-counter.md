---
title: "Program Counter"
permalink: /docs/programcounter/
excerpt: "Program Counter del BEAM computer"
---
[![Program Counter del BEAM computer](../../assets/pc/35-beam-pc.png "Program Counter del BEAM computer"){:width="100%"}](../../assets/pc/35-beam-pc.png)

Il Program Counter (PC) del BEAM Computer presenta poche differenze rispetto al PC del SAP Computer di Ben Eater.

E' ora un registro ad 8 bit anziché a 4 bit, perciò permette di indirizzare 256 byte e non più solo 16:

[![Schema logico del Program Counter del computer BEAM](../../assets/pc/35-program-counter-schema.png "Schema logico del Program Counter del computer BEAM"){:width="100%"}](../../assets/pc/35-program-counter-schema.png)

*Schema logico del Program Counter del computer BEAM.*

I due Synchronous 4-Bit Binary Counters <a href="https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf" target="_blank">74LS161</a> sono connessi in cascata secondo la modalità illustrata a pagina 21 del datasheet:

[![Contatori in cascata](../../assets/pc/35-program-counter-161-rco.png "Contatori in cascata"){:width="66%"}](../../assets/pc/35-program-counter-161-rco.png)

Il Carry Out di un contatore binario a 4 bit, attivo al raggiungimento del conteggio 2^4, permette al contatore successivo di incrementare di un'unità il suo conteggio. Due contatori in cascata permettono di contare fino a 2^4 \* 2^4 = 16 \* 16 = 256.

## Note sul microcode

- Verificare se ci sono note da inserire in questa sezione in relazione al contenuto della pagina. Forse qualcosa relativo al fatto che le prime due micro - istruzioni sono sempre le stesseE un rimando al modulo dei Flag nel quale si spiega meglio come venga eseguito il salto a una nuova istruzione;
inoltre Forse an- che un rimando alla pagina DXYche viene utilizzata per calcolare il nuovo valore del PC del salto condizionale

## Link utili

- I <a href="https://eater.net/8bit/pc" target="_blank">video</a> di Ben Eater che descrivono il funzionamento dei Flip-Flop e la costruzione del PC.

## TO DO

- Verificare se la sezione **Note sul microcode** sia necessaria o no
