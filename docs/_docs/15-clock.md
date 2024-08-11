---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di Clock del BEAM computer"
---
[![Circuito di Clock del BEAM computer](../../assets/clock/15-beam-clock.png "Circuito di Clock del BEAM computer"){:width="100%"}](../../assets/clock/15-beam-clock.png)

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

## Note sul microcode

Verificare se ci sono note da inserire in questa sezione in relazione al contenuto della pagina.

## Link utili

- [Inside the vintage 74181 ALU chip: how it works and why it's so strange](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) di Ken Shirriff. Fondamentale per capire il perché dell'implementazione apparentemente così strana del chip.
- [Demo of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=Fq0MIJjlGsw) e [Comparator Functions of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=jmROTNtoUGI): due ottimi video di David Courtney.

## TO DO

- /WE ↘↗
- Descrivere comportamento HW **shift register** per le rotazioni
- Parlare del bench di test sulla base di quanto appreso da David Courtney.
- *Schema di uno degli 8 Flip-Flop del 74LS377.* -- **Da fare**: Valutare se anche questo ha un riflesso positivo sul discorso del glitch
- https://bread80.com/2019/09/02/adding-adc-sbc-inc-dec-operations-to-ben-eaters-alu/#easy-footnote-4-43 da leggere per capire se buono
- subito dopo il capitolo "Il registro H" capire "(da fare: in questo caso, ma anche nel caso delle istruzioni di shift / rotazione e forse anche CPX e CPY, verificare se non potessi usare D invece di H)"
- Verificare se sono necessarienote sul microcode ; potrebbe essere il caso di spiegare come la istruzione HLT possa fermare il computer
