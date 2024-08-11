---
title: "Program Counter"
permalink: /docs/programcounter/
excerpt: "Program Counter del BEAM computer"
---
[![Program Counter del BEAM computer](../../assets/pc/35-beam-pc.png "Program Counter del BEAM computer"){:width="100%"}](../../assets/pc/35-beam-pc.png)

PLACEHOLDER

Bisogna dire che più volte, leggendo la documentazione di Tom Nisbet, avevo trovato delle idee molto clever; alcune di queste si trovano nel modulo dei Flag. Cercherò di riportare anche alcune fonti utilizzate da Tom indicandone i collegamenti a fondo pagina.

[![Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.](../../assets/flags/30-flag-nqsap-schematics.png "Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità."){:width="100%"}](../../assets/flags/30-flag-nqsap-schematics.png)

*Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Il registro dei Flag dell'NQSAP emula i 4 flag **NVZC** del 6502:

- **N**egative (N)
- O**V**erflow (V)
- **Z**ero (Z)
- **C**arry (C)

invertita) che a sua volta attiva il caricamento sul PC del valore presente sul bus: (/PC-LOAD = NOT (x+1) = 0), pertanto il PC carica dal bus il suo nuovo valore.

![NOR per l'attivazione di /PC-LOAD con salti condizionali ed incondizionati](../../assets/flags/30-flag-je-wp.png){:width="50%"}

*NOR per l'attivazione di /PC-LOAD con salti condizionali ed incondizionati.*

In definitiva, il microcode delle istruzioni di salto prevede:

- l'attivazio51 per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'ozione, il microcode di quella istruzione provvederà infatti ad attivare opportunamente i segnali C0 e C1:logica attiva alta, “Active-High data”) richiede un segnale Carry In invertito.

Si noti che la Truth Table della tabella richiederebbe i componenti evidenziati nello schema seguente, ma l'applicazione del teorema di De Morgan permette la semplificazione utilizzata poi nello schema adottato nell'NQSAP.

![Schema originale per realizzazione della Truth Table di selezione del Carry](../../assets/flags/30-flag-c-h-alu-de-morgan.png){:width="50%"}

*Schema originale per realizzazione della Truth Table di selezione del Carry.*

![Schema rivisto con l'applicazione del teorema di De Morgan](../../assets/flags/30-flag-c-h-alu-de-morgan-nor.png){:width="50%"}

*Schema rivisto con l'applicazione del teorema di De Morgan.*

## Moduli Flag dell'NQSAP e del BEAM

Il modulo Flag del computer BEAM è sostanzialmente una copia del modulo Flag del computer NQSAP; con le mie conoscenze non avrei saputo svilupparne una versione migliore, ma il fatto di averne compreso perfettamente tanto il funzionamento quanto l'integrazione con gli altri moduli del computer è stata una ottima lezione.

[![Schema logico del modulo Flag del computer BEAM](../../assets/flags/30-flag-beam-schematics.png "Schema logico del modulo Flag del computer BEAM"){:width="100%"}](../../assets/flags/30-flag-beam-schematics.png)

*Schema logico del modulo Flag del computer BEAM.*

## ALTRO


## Link utili

- Tom segnala di aver preso ispirazione dal thread su Reddit [How to add a decremental and incremental circuit to the ALU ?](https://www.reddit.com/r/beneater/comments/jwxke0/how_to_add_a_decremental_and_incremental_circuit/) per l'idea di pilotare il caricamento del Program Counter dal registro dei Flag anziché attraverso il microcode, come avveniva invece sul SAP di Ben Eater.

Flag e Microcode
Molte delle istruzioni modificano i flag.

- Vedere bene quali istruzioni CP* hanno bisogno di LF, anche sul file XLS

Altre referenze Tom Nisbet per Flags

- Opcodes and Flag decoding circuit on reddit has a different approach to conditional jumps using hardware. Instead of driving the LOAD line of the PC, the circuit sits between the Instruction Register and the ROM and conditionally jams a NOP or JMP instruction to the microcode depending on the state of the flags. One interesting part of the design is that the opcodes of the jump instructions are arranged so that the flag of interest can be determined by bits from the IR. NQSAP already did something similar with the ALU select lines, so the concept was used again for the conditional jump select lines.


http://www.6502.org/tutorials/vflag.html per V sezione 2.4.2.1

## Link utili
