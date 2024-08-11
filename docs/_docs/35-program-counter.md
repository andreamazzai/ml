---
title: "Program Counter"
permalink: /docs/programcounter/
excerpt: "Program Counter del BEAM computer"
---
[![Program Counter del BEAM computer](../../assets/pc/35-beam-pc.png "Program Counter del BEAM computer"){:width="100%"}](../../assets/pc/35-beam-pc.png)

Il Program Counter (PC) del BEAM Computer non presenta differenze sostanziali rispetto al PC del SAP Computer di Ben Eater.

- è ora un registro ad 8 bit anziché a 4 bit, perciò permette di indirizzare 256 byte
- due Synchronous 4-Bit Binary Counters [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) sono connessi in cascata secondo la modalità illustrata a pagina 21 del datasheet:

[![Contatori in cascata](../../assets/pc/35-program-counter-161-rco.png "Contatori in cascata"){:width="66%"}](../../assets/pc/35-program-counter-161-rco.png)

## Note sul microcode

Verificare se ci sono note da inserire in questa sezione in relazioneal program counter.

## Link utili

- I [video](https://eater.net/8bit/pc) di Ben Eater che descrivono il funzionamento dei Flip-Flop e la costruzione del PC.
