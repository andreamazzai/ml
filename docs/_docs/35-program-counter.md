---
title: "Program Counter"
permalink: /docs/programcounter/
excerpt: "Program Counter del BEAM computer"
---
[![Program Counter del BEAM computer](../../assets/pc/35-beam-pc.png "Program Counter del BEAM computer"){:width="100%"}](../../assets/pc/35-beam-pc.png)

PLACEHOLDER

Il Program Counter (PC) del BEAM Computer non presenta differenze sostanziali rispetto al PC del SAP Computer di Ben Eater.

- è ora un registro ad  bit anziché a 4 bit e dunque permette di indirizzare 256 byte
- due Synchronous 4-Bit Binary Counters [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) sono connessi in cascata secondo la modalità illustrata a pagina 21 del datasheet:

[![Program Counter del BEAM computer](../../assets/pc/35-beam-pc.png "Program Counter del BEAM computer"){:width="100%"}](../../assets/pc/35-beam-pc.png)

## Note sul microcode

Verificare se ci sono note da inserire in questa sezione in relazioneal program counter.

## Link utili

- Tom segnala di aver preso ispirazione dal thread su Reddit [How to add a decremental and incremental circuit to the ALU ?](https://www.reddit.com/r/beneater/comments/jwxke0/how_to_add_a_decremental_and_incremental_circuit/) per l'idea di pilotare il caricamento del Program Counter dal registro dei Flag anziché attraverso il microcode, come avveniva invece sul SAP di Ben Eater.

Flag e Microcode
Molte delle istruzioni modificano i flag.

- Vedere bene quali istruzioni CP* hanno bisogno di LF, anche sul file XLS

Altre referenze Tom Nisbet per Flags

- Opcodes and Flag decoding circuit on reddit has a different approach to conditional jumps using hardware. Instead of driving the LOAD line of the PC, the circuit sits between the Instruction Register and the ROM and conditionally jams a NOP or JMP instruction to the microcode depending on the state of the flags. One interesting part of the design is that the opcodes of the jump instructions are arranged so that the flag of interest can be determined by bits from the IR. NQSAP already did something similar with the ALU select lines, so the concept was used again for the conditional jump select lines.
