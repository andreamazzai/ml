---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di Clock del BEAM computer"
---
Il circuito di clock del computer BEAM riprende il modulo di clock progettato da Ben Eater ([link](https://eater.net/8bit/clock))per il computer SAP, semplificandone leggermente l'utilizzo mediante l'utilizzo di push Button anziché interruttori e aggiungendo qualche interazione con controlli esterni come quelli provenienti dal loader .

- Il primo modulo è un classico generatore a stabile con possibilità di selezionare la frequenza attraverso un potenziometro e un pulsante che permette di raggiungere la velocità massima , utile ad esempio in caso di debug .
- Il secondo 555 permette di eseguire dei singoli step.
- Il terzo 555 permette di selezionare tra la modalità a step e il clock continuo utilizzando un più comodo bottone anziché un interruttore. 

Notare che premendo il bottone dello step Durante l'esecuzione in clock continuo permettedi fermare il computer all'istruzione attuale e di proseguire dal micro istruzione successiva step by step . premendo nuovamente il tasto start il computer riprende a lavorare in modalità clock continuo.

Notare anche gli ingressi ~{LDR-Active} e LDR-CLK.

Se ~{LDR-Active} è LO, allora disattivo tutti i clock del modulo perché Arduino gestirà LDR-CLK.

Presente che HALT, che permette di stoppare l'esecuzione del codice con l'istruzione BRK, Il cui microcode porta a uno questo segnale.

### Primi studi

[![Schema del modulo Clock](../../assets/clock/15-clock-schema.png "Schema del modulo Clock")](../../assets/clock/15-clock-schema.png)

*Schema del modulo Clock.*

**METTERE QUALCHE FOTO E LINK**

>> 2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; la soluzione poteva essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737); intuivo qualcosa, relativamente alla necessità di gestire i segnali di controllo della RAM in maniera più ampia, controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## Design dei moduli MAR e RAM del BEAM

La gestione dei segnali passa in maniera importante attraverso il MUX '157, così come avviene anche nello schema di The8BitEnthusiast, con qualche differenza.

Ecco un riassunto degli stati possibili:

- **RR /WR**: Read RAM non è attivo, Write RAM è attivo.
  - Za è normalmente HI (perché la NAND su I1a ha un'ingresso LO), ma in corrispondenza dell'impulso di clock passa a LO e attiva /WE, scrivendo sulla RAM.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è normalmente HI, dunque disattiva il transceiver che interconnette RAM e bus, ma in corrispondenza dell'impulso di clock passa a LO (perché Za passa a LO), attivando il transceiver.
- **/RR WR**: Read RAM è attivo, Write RAM non è attivo.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso LO e attiva il transceiver tra RAM e bus.
- **/RR /WR**: Stato illegale.

## Link utili

- I video di Ben Eater con la spiegazione del 555 e dei vari componenti
Tom segnala di aver preso ispirazione da un thread su Reddit [How to add a decremental and incremental circuit to the ALU ?](https://www.reddit.com/r/beneater/comments/jwxke0/how_to_add_a_decremental_and_incremental_circuit/) per l'idea di pilotare il caricamento del [Program Counter](../programcounter/) dal registro dei Flag anziché gestirli con copie multiple del microcode come avveniva come sul SAP di Ben Eater.

- Tom notava anche l'approccio del thread [Opcodes and Flag decoding circuit](https://www.reddit.com/r/beneater/comments/m76ijz/opcodes_and_flag_decoding_circuit/) per eseguire salti condizionali in hardware. Invece di pilotare la linea LOAD del Program Counter, il circuito dell'autore del thread si trova tra il Registro delle Istruzioni e la ROM e forza condizionatamente un'istruzione NOP o JMP a seconda dello stato dei flag. Gli opcode delle istruzioni di salto sono disposti in modo tale che il flag di interesse possa essere determinato dai bit in uscita dall'Instruction Register. Concetto interessante, ma Tom aveva già implementato una funzionalità simile con le linee di selezione dell'ALU hardwired all'IR, quindi questa modalità è stata riutilizzata anche per la gestione dei [salti condizionali](#i-salti-condizionali).

## TO DO

- Vedere bene quali istruzioni CP* hanno bisogno di LF, anche sul file XLS
- Effetto non desiderato: "le istruzioni di salto condizionato non eseguite sprecano cicli di clock"… non si potrebbe semplicemente usare N per terminare anticipatamente l'istruzione? Lui sembra renderla un po' complicata
- 29/01/2023 leggendo bene dice che dovrebbe essere possibile fare in modo che la logica elettronica dell'istruzione Jump vada ad attivare N se il salto non deve esserci… da verificare