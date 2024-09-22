---
title: "Stack Pointer"
permalink: /docs/stack/
excerpt: "Stack Pointer del computer BEAM"
---
[![Stack Pointer del computer BEAM](../../assets/sp/70-beam-sp.png "Stack Pointer del computer BEAM"){:width="100%"}](../../assets/sp/70-beam-sp.png)

## - WORK IN PROGRESS

L'implementazione dello stack nel 6502 prevede l'utilizzo di un'area di memoria dedicata alla memorizzazione e al ripristino di informazioni secondo una logica LIFO (Last-In, First-Out, dove l'ultimo elemento inserito è il primo a essere letto), gestita da un puntatore (Stack Pointer, SP) che tiene traccia dell'indirizzo della prossima locazione disponibile.

Due usi comuni dello stack sono il salvataggio dello stato attuale dei Flag prima di eseguire una routine che li modifica, così da poterli ripristinare al termine della routine, e la memorizzazione dell'indirizzo di ritorno di una subroutine invocata dall'istruzione JSR.

Lo Stack Pointer del 6502 è un registro, denominato S, la cui implementazione nel BEAM consta di due 4-bit Synchronous Binary Up-Down Counter <a href="https://www.ti.com/lit/ds/symlink/sn74ls157.pdf" target="_blank">74LS169</a> in grado di indirizzare i 256 byte del computer. In definitiva, si tratta di un normale registro che, oltre a poter memorizzare un valore specifico, ha la peculiarità di poter anche contare sia verso l'alto, sia verso il basso.

Lo stack del 6502 occupa la seconda pagina di memoria indirizzabile dalla CPU, corrispondente agli indirizzi compresi tra 0x0100 e 0x01FF. Prima dell'utilizzo, lo Stack Pointer viene solitamente inizializzato a 0xFF, puntando così alla locazione 0x1FF. Quando si effettua un'operazione di scrittura nello stack ("Push"), il valore viene prima salvato nella posizione indicata dal puntatore, dopodiché lo SP viene decrementato per puntare alla successiva locazione libera. Al contrario, durante un'operazione di lettura dallo stack ("Pull"), il valore viene prima letto dalla posizione indicata dal puntatore, che viene poi incrementato per indirizzare la prossima posizione disponibile.

Poiché il BEAM dispone di soli 256 byte di memoria, l'area destinata allo stack dovrà essere necessariamente limitata, ad esempio ai 16 byte compresi tra gli indirizzi 0xF0 e 0xFF. Tuttavia, è improbabile che un programma residente nella memoria rimanente richieda più di 16 byte di stack.

Lo Stack Pointer punta sempre alla successiva locazione disponibile nello stack, dunque:

- un'operazione di scrittura sullo stack scrive il valore nella locazione di memoria indicata dallo Stack Pointer (ad esempio, l'indirizzo di ritorno dell'istruzione JSR) e poi lo decrementa (post-decrement);
- un'operazione di lettura dallo stack dapprima incrementa lo SP (pre-increment) e poi restituisce il contenuto della locazione di memoria da esso indicata.

In altre parole, poiché lo stack "cresce" verso il basso, lo Stack Pointer viene decrementato dopo ogni inserimento di un byte nello stack e incrementato prima di ogni prelievo di un byte.

I 74LS169 utilizzati nel BEAM non dispongono di un ingresso Reset, il che significa che, all'accensione, potrebbero trovarsi in uno stato indefinito: di conseguenza, lo stack deve essere inizializzato dal programma caricato in memoria dall'utente.

Le istruzioni del 6502 che interagiscono con lo stack sono:

- PHA, PLA: Push e Pull del registro A verso lo stack / dallo stack
- PHP, PLP: Push e Pull del registro dei Flag verso lo stack / dallo stack
- JSR, RTS: salto a / ritorno da subroutine
- TXS, TSX: caricamento / lettura dello SP

## Implementazione del microcode dello Stack Pointer

Analizziamo un'istruzione JSR, ipotizzando di aver precedentemente inizializzato lo Stack Pointer (SP) del BEAM a 0xFF. Il BEAM dispone di soli 256 byte, quindi lo SP punta ora all'ultima locazione di memoria del computer.

Supponiamo di avere il seguente codice:

~~~text
| Mnemonico | Indirizzo | Valore | Microistruzione                   |
| --------- | --------- | ------ | --------------------------------- |
| ...       | ...       | ?      | Istruzione precedente o operando  |
| NOP       | 0x1F      | 0x0F   | Opcode istruzione NOP del BEAM    |
| JSR $30   | 0x20      | 0x41   | Opcode istruzione JSR del BEAM    |
|           | 0x21      | 0x30   | Indirizzo di salto istruzione JSR |
| NOP       | 0x22      | 0x0F   | Opcode istruzione NOP del BEAM    |
| ...       | ...       | ?      | Prossima istruzione               |
| ...       | ...       | ?      | ...                               |
| INX       | 0x30      | 0xA0   | Opcode istruzione INX del BEAM    |
| RTS       | 0x31      | 0x11   | Opcode istruzione RTS del BEAM    |
~~~

Questo programma, dopo una istruzione NOP, invoca una subroutine che incrementa il registro X. Al ritorno dalla subroutine, segue un'altra istruzione NOP.

La tabella che segue **evidenzia visivamente** le modifiche apportate dall'istruzione JSR ai registri Program Counter (PC), Stack Pointer, Memory Address Register (MAR), Instruction Register (IR), B e al valore della locazione di memoria indirizzata dal MAR. La tabella include anche l'istruzione precedente (NOP) e il primo step della successiva (INX) presente nella subroutine invocata dall'istruzione JSR.

 | Istruzione | Step | Microistruzioni   | PC      | SP      | MAR     | RAM     | IR      | B       |
 |------------|------|-------------------|---------|---------|---------|---------|---------|---------|
 | NOP        | 0\*  | RPC \| WM         | 0x1F    | 0xFF    | <b>0x1F | <b>0x0F | ?       | ?       |
 | NOP        | 1\*  | RR  \| WIR \| PCI | <b>0x20 | 0xFF    | 0x1F    | 0x0F    | <b>0x0F | ?       |
 | NOP        | 2\*  | NI                | 0x20    | 0xFF    | 0x1F    | 0x0F    | 0x0F    | ?       |
 | JSR        | 0    | RPC \| WM         | 0x20    | 0xFF    | <b>0x20 | <b>0x41 | 0x0F    | ?       |
 | JSR        | 1    | RR  \| WIR \| PCI | <b>0x21 | 0xFF    | 0x20    | 0x41    | <b>0x41 | ?       |
 | JSR        | 2    | RPC \| WM         | 0x21    | 0xFF    | <b>0x21 | <b>0x30 | 0x41    | ?       |
 | JSR        | 3    | RR  \| WB  \| PCI | <b>0x22 | 0xFF    | 0x21    | 0x30    | 0x41    | <b>0x30 |
 | JSR        | 4    | RS  \| WM         | 0x22    | 0xFF    | <b>0xFF | ?       | 0x41    | 0x30    |
 | JSR        | 5    | RPC \| WR         | 0x22    | 0xFF    | 0xFF    | <b>0x22 | 0x41    | 0x30    |
 | JSR        | 6    | SE                | 0x22    | <b>0xFE | 0xFF    | 0x22    | 0x41    | 0x30    |
 | JSR        | 7    | RB  \| WPC \| NI  | <b>0x30 | 0xFE    | 0xFF    | 0x22    | 0x41    | 0x30    |
 | INX        | 0\*\*| RPC \| WM         | 0x30    | 0xFE    | <b>0x30 | 0xA0    | 0x41    | 0x30    |

*Scomposizione dell'istruzione JSR nelle sue otto microistruzioni elementari e raffigurazione dello stato dei registri al termine di ogni step.*

\* Istruzione NOP precedente\
\*\* Primo step dell'istruzione INX successiva\

1. Il primo step carica l'indirizzo del (PC) nel Memory Address Register:
    - RPC, Read Program Counter - espone l'indirizzo del PC sul bus
    - WM, Write Memory Address Register - carica l'indirizzo del PC nel MAR
2. Il secondo step carica l'opcode dell'istruzione nell'IR e incrementa il PC per farlo puntare alla locazione di memoria successiva (che contiene l'operando dell'istruzione JSR, cioè l'indirizzo di destinazione del salto):
    - RR, Read RAM - espone sul bus il contenuto della locazione di memoria puntata dal MAR
    - WIR, Write Instruction Register - carica l'opcode dell'istruzione nell'IR\*
    - PCI, Program Counter Increment - incrementa il PC
3. Il terzo step carica nel Memory Address Register l'indirizzo del Program Counter, che ora punta all'Operando:
    - RPC, Read Program Counter - espone l'indirizzo del PC sul bus
    - WM, Write Memory Address Register - carica l'indirizzo del PC nel MAR
4. Il quarto step legge l'operando, che punta alla destinazione del salto dell'istruzione JSR, lo salva in B e incrementa il PC che va ora a puntare alla prossima istruzione, coincidente con l'indirizzo di ritorno dalla subroutine:
    - RR, Read RAM - espone sul bus il contenuto della locazione di memoria puntata dal MAR
    - WB, Write B - memorizza temporaneamente in B l'indirizzo di destinazione del salto
    - PCI, Program Counter Increment - incrementa il PC
5. Il quinto step carica nel Memory Address Register l'indirizzo dello Stack Pointer, che corrisponde alla prima locazione libera dello stack:
    - RS, Read Stack - espone sul bus l'indirizzo il valore dello Stack Pointer
    - WM, Write Memory Address Register - carica l'indirizzo dello SP nel MAR
6. Il sesto step carica nello stack l'indirizzo del PC, al quale il computer ritornerà alla fine della subroutine invocata dall'istruzione JSR:
    - RPC, Read Program Counter - espone l'indirizzo del PC sul bus
    - WR, Write RAM - carica nello stack l'indirizzo di ritorno dalla subroutine
7. Il settimo step decrementa (post-decrement) lo Stack Pointer:
    - SE, Stack Enable - i contatori '169 decrementano l'output di un'unità, andando a puntare alla prossima locazione libera
8. L'ottavo step carica nel PC l'indirizzo della subroutine invocata dall'istruzione JSR:
    - RB, Read B - espone sul bus il registro B, nel quale era stato precedentemente memorizzato l'indirizzo della subroutine
    - WPC, Write Program Counter - carica nel PC l'indirizzo della subroutine
    - NI, Next Instruction - resetta il Ring Counter

\*Notare come l'Instruction Register venga aggiornato solo alla fine del secondo step dell'istruzione, come già visto nella spiegazione delle [Fasi](../control/#fasi) della CPU.

Al contrario RTS farà:

- leggere il valore dell'SP
- scrivere in PC il valore contenuto nella cella indicata dall'SP
- incrementare l'SP

Step NB queste sono le mie considerazioni
Fetch Adesso so che sto facendo una JSR
Exec 1 Metto nel MAR l'indirizzo della cella che contiene l'indirizzo a cui devo saltare
Exec 2 Leggo l'indirizzo a cui devo saltare dalla cella puntata dal MAR e lo scrivo temporaneamente in B (operando)
Exec 3 Leggo lo stack (che punta alla prima cella di memoria disponibile) e lo metto nel MAR; incremento il PC per ottenere il alore di "ritorno" dell'RTS
Exec 4 Scrivo il valore di ritorno dell'RTS nella cella di memoria indicata dallo stack
Exec 5 Incremento lo stack e scrivo B nel PC, così al prossimo clock eseguo il codice presente nell'indirizzo a cui dovevo altare

## Lo Stack Pointer dell'NQSAP

Nella documentazione dell'NQSAP Tom segnala che si potrebbe usare anche un '193, ma con attenzione perché non ha il Count Enable e dunque bisogna usare una porta esterna per far arrivare un clock al contatore solo quando il segnale di Enable è attivato dalla ROM e dunque usa dei 169; attenzione ai glitch della ROM, dice… 05/01/2023 descritti in seguito

Poiché Tom non aveva pubblicato lo schema dello Stack Pointer dell'NQSAP, lo sostituiamo con quello dell'NQSAP-PCB.

[![Schema dello Stack Register del computer NQSAP-PCB](../../assets/sp/70-stack-nqsap-pcb.png "Schema dello Stack Register del computer NQSAP-PCB"){:width="100%"}](../../assets/sp/70-stack-nqsap-pcb.png)

*Schema dello Stack Register del computer NQSAP-PCB.*

- Successivamente, nell'NQSAP-PCB, ritorna sui suoi passi ed utilizza proprio un '193, che conta inviando un segnale a UP o DOWN del 193 abilitando SE Stack Enable insieme a C0 o C1 per definire la direzione del conteggio, dei quali viene fatto l'AND con il CLK.

- WS, sempre ANDed con CLK, permette di caricare un valore specifico nello stack.
- Si usano C0 e C1 per risparmiare segnali in uscita dalle EPROM… che clever questo tipo
  - C0 e C1 sono infatti condivisi con il Flag Register per la selezione del Carry in ingresso da ALU o H, dunque non bisogna modificare il CF nella stessa microistruzione di conteggio dell'SP.
  - In NQSAP-PCB dice che C0 e C1 sono anche condivisi con DY e DZ…
    - forse perché aveva pochi segnali disponibili ora che ha ridotto le ROM da 4 a 3… e infatti nella Control Logic a 3 ROM ci sono solo C0 e C1, che vanno sugli stessi pin del bus 26 e 27 di DY e DZ dei registri X e Y… 😁
dunque nemmeno istruzioni SP e X/Y possono avvenire nella stessa microistruzione (se consolido C0 e C1 con DY e DZ).

    - After the instruction fetch CO MI CE / RO II, the PC will have the value 21 and JSR microcode performs the following steps:
§ CO MI CE move the PC value to the MAR and increment the PC. MAR contains 21 and PC contains 22.
§ RO BI read the subroutine address from RAM[21] and place it in B for temp storage
§ SPO MI SPI move the SP value into the MAR and increment the SP.
§ CO RI store the PC value (which points to the next instruction) in memory, i.e. push the JSR return address on the stack.
BO PI move the B register value into the PC, effectively jumping to the subroutine

## Schema

[![Schema dello Stack Register del computer BEAM](../../assets/sp/70-stack-pointer-schema.png "Schema dello Stack Register del computer BEAM"){:width="100%"}](../../assets/sp/70-stack-pointer-schema.png)

*Schema dello Stack Register del computer BEAM.*

## Link utili

- <a href="https://wilsonminesco.com/stacks/basics.html" target="_blank">Stack definition and basics</a> di Garth Wilson, contributore di <a href="http://www.6502.org" target="_blank">6502.org</a> e curatore di <a href="https://wilsonminesco.com/" target="_blank">Wilson Mines Co.</a>, vera miniera di articoli sul 6502, nozioni, tutorial ed altro. Garth offre una serie di compendi incredibilmente utili su tutto ciò che riguarda il 6502 ed affini.
- ??????????????????????
- ??????????????????????

## TO DO

- /WE ↘↗
- controllare se il mio SP funziona correttamente; vedo che Tom ha usato dei cablaggi diversi per i pin 7 e 10   20-08-2024
- Controller Nelle pagine dove ci sono le descrizioni delle microistruzioni controllare se non sia meglio invertire la forma della frase , cioè invece di dire "espone nel bus e carica nel Mar", dire "carica nel Mar il contenuto del bus"
