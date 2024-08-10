---
title: "Per iniziare"
permalink: /docs/beginning/
excerpt: "Per iniziare"
---
[![Modulo RAM e MAR del BEAM](../../assets/ram/20-ram-beam.png "Modulo RAM e MAR del BEAM"){:width="100%"}](../../assets/ram/20-ram-beam.png)

## Cosa devo includere in questa pagina?

- Per esempio Il circuito che permette di leggere o scrivere su un bus a 8 bit



### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

Il '273, al pari del '173, presenta un ingresso Clear / Reset (CLR), che nel MAR è necessario per resettare il registro - o almeno *credevo* fosse necessario. Sembrava anche interessante l'ipotesi alternativa di usare un registro a 8 bit [74LS377](https://datasheetspdf.com/download_new.php?id=375625), che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il CLR, non presente in questo chip. In seguito ho realizzato che il MAR poteva funzionare perfettamente anche senza un segnale di Clear / Reset e il '377 sarebbe diventato uno dei chip più utilizzati nel BEAM.

### Memorie con IO separati o IO comuni?

Fino ad ora, avevo quasi sostanzialmente dato per scontato di continuare ad usare chip di memoria con porte di Input e Output separati ("dual-port"), esattamente come accade nel [74189](https://eater.net/datasheets/74189.pdf) utilizzato nel SAP. Tuttavia, in questo [post su Reddit](https://www.reddit.com/r/beneater/comments/hon6ar/74189_alternative/), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf); ho così iniziato ad approfondire le caratteristiche di questo chip "single-port", aumentando la mia comprensione di queste due diverse architetture.

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

- **CE** = Chip Enable
- **OE** = Output Enable
- **WE** = Write Enable

Un altro aspetto che avevo notato immediatamente, ipotizzando l'uso del 62256, era l'impossibilità di mantenere la visibilità del contenuto della cella di RAM indirizzata dal MAR utilizzando i LED (o almeno così credevo): se con i '189 le porte di output erano sempre attive e potevo vedere in ogni momento il valore contenuto della cella di memoria correntemente indirizzata dal MAR, con il 62256 avrei avuto visibilità del contenuto della cella solo nel momento in cui la RAM veniva letta - e dunque non costantemente.

[![Schema del modulo RAM di Ben Eater basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione ininterrotta del contenuto della locazione RAM indirizzata dal MAR](../../assets/ram/20-be-ram-detail.png "Schema del modulo RAM basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione continua del contenuto della locazione RAM indirizzata dal MAR"){:width="50%"}](../../assets/ram/20-be-ram-full.png)

*Nello schema si notano i 74189 con le porte di Input dedicate D1-D4 e le porte di Output dedicate O1-O4.*

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; la soluzione poteva essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737); intuivo qualcosa, relativamente alla necessità di gestire i segnali di controllo della RAM in maniera più ampia, controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo prima avere la possibilità di programmare la RAM ("Program Mode") e  poi di renderla visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è manuale e sfrutta dei dip-switch per indirizzare e programmare manualmente la RAM.

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata esclusivamente dal MAR e altrettanto esclusivamente viene acceduta in lettura / scrittura solo dal bus del computer; da notare che è possibile passare manualmente al Program Mode per leggere o scrivere il contenuto della RAM in stile "debug mode" e riprendere poi la normale esecuzione del programma.

Riprendevo il tema del "doppio passaggio" (o meglio "doppio bus", come capirò in seguito), che da quanto iniziavo a comprendere poteva rendere possibile la visualizzazione persistente del contenuto della RAM. A pagina 17 e 18 del "Building the SAP-3 rev 3.3.pdf" presente nel repository GitHub di <a href = "https://github.com/rolf-electronics/The-8-bit-SAP-3" target = "_blank">rolf-electronics</a>, altro utente del canale Reddit, avevo notato che era stato inserito un secondo transceiver '245.

[![Modulo RAM di rolf electronics](../../assets/ram/20-rolf-ram.png "Modulo RAM di rolf electronics"){:width="50%"}](../../assets/ram/20-rolf-ram.png)

Lo schema del modulo RAM di Rolf ne prevede invece due, uno "interno" e uno "esterno", per separare il percorso dei dati *verso* la RAM da quello dei dati *dalla* RAM:

- Il transceiver di sinistra è attivo quando si scrive *sulla* RAM, selezionando quale sia la sorgente mediante il MUX '157 (che in "Program Mode" attiva gli ingressi connessi al dip-switch, mentre in "Run Mode" attiva gli ingressi connessi al bus); i LED mostrano il valore che viene scritto sulla RAM.
- Il transceiver di destra è attivo quando si legge *dalla* RAM; i LED mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.

Proseguendo nello studio, ho trovato [questo schema](https://imgur.com/a/ruclh) dell'utente jaxey1631, che aveva lasciato un commento nel video di Ben Eater [Reprogramming CPU microcode with an Arduino](https://www.youtube.com/watch?v=JUVt_KYAp-I&lc=UgjusLoROw6az3gCoAEC):

[![RAM e MAR con doppio bus](../../assets/ram/20-ram-ruclh.png "RAM e MAR con doppio bus"){:width="100%"}](../../assets/ram/20-ram-ruclh.png)

*Schema di RAM e MAR con bus interno.*

In questo schema troviamo:

- Il chip di RAM 62256.

Un aspetto collaterale (ma importantissimo) dell'aumento del numero di istruzioni era la necessità di aumentare la dimensione delle EEPROM ospitanti il [microcode](../control): volendo gestire (fino a) 256 istruzioni, erano necessari 8 bit di istruzioni, 3 di step e 2 di flag = 13 pin totali, portanto si rendevano necessarie delle 28C64... e avevo dimenticato che mi sarebbe servito un bit aggiuntivo per la selezione delle due EEPROM! In quel momento, non sapevo ancora che avrei speso *intere settimane* a comprendere il fantastico modulo dei [Flag](../flags) dell'NQSAP di Tom Nisbet, che ha un approccio completamente diverso e che non necessita di segnali in uscita dalle EEPROM.


### Terza versione del modulo RAM

Ho dunque riscritto la "truth table" del modulo RAM:

![Tabella analisi modulo RAM - terza versione](../../assets/ram/20-ram-3rd-table.png "Tabella analisi modulo RAM - terza versione"){:width="100%"}

*Tabella analisi modulo RAM - terza versione.*

I due asterischi in tabella \*\* mi servivano a ricordare che dovevo stare particolarmente attento alla preparazione del microcode e che non dovevo mai avere /WR e /RR attivi contemporaneamente, perché non avevo previsto un controllo di eventuale conflitto. Per evitare conflitti avrei dovuto mettere su /WR una NAND a 3 ingressi che si attiva solo per CLK, RR e /WR, cioè /WE = CLK \* RR \* /WR, in altre parole: posso scrivere sulla RAM *solo* se non la sto leggendo.

Ecco un riassunto degli stati possibili:

- **/PROG**: In Program Mode il MUX attiva gli ingressi I0a, I0b, I0c e I0d. In Run Mode sono attivi I1a, I1b, I1c e I1d.
  - Za è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva /WE, scrivendo sulla RAM.
  - Zb è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva il transceiver che interconnette RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR WR**: Read RAM e Write RAM non sono attivi.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR /WR**: Read RAM non è attivo, Write RAM è attivo.
  - Za è normalmente HI (perché la NAND su I1a ha un'ingresso LO), ma in corrispondenza dell'impulso di clock passa a LO e attiva /WE, scrivendo sulla RAM.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è normalmente HI, dunque disattiva il transceiver che interconnette RAM e bus, ma in corrispondenza dell'impulso di clock passa a LO (perché Za passa a LO), attivando il transceiver.
- **/RR WR**: Read RAM è attivo, Write RAM non è attivo.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso LO e attiva il transceiver tra RAM e bus.
- **/RR /WR**: Stato illegale.

## Note

Invece dei più comuni dip-switch, ho utilizzato dei comodissimi Rocker Switch ("a bilanciere") come quelli in figura; si trovano facilmente presso i distributori di [materiale elettronico](https://us.rs-online.com/product/te-connectivity/5435640-5/70156004/). Notare che i pin originali sono piuttosto corti e non fissano correttamente lo switch alla breadboard, pertanto ho aggiunto uno zoccolo per circuiti integrati.

[![Rocker Switch](../../assets/ram/20-ram-rocker.png "Rocker Switch"){:width="33%"}](../../assets/ram/20-ram-rocker.png)

*Rocker Switch.*
