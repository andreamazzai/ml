---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da zero a 255 e viceversa, oppure un algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la ricerca e la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Il primo articolo letto e dal quale avevo iniziato a ricavare un po' di informazioni è [questo](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) e ne avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte, ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve un registro MAR (Memory Address Register) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173 (il 74LS173 è il FF utilizzato in origine nel SAP), sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando necessario - e non ad ogni ciclo di clock. Il segnale MI (Memory Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario; quando MI è attivo, al successivo rising edge del CLK il MAR carica il valore presente nel bus. Senza un ingresso di Enable, il FF 74LS273 andrebbe a "registrare" il dato in ingresso ad ogni rising edge del clock.

In un progetto a 8 bit si potrebbero semplicemente utilizzare due 74LS173 a 4 bit continuando a sfruttare i segnali di Enable nativamente disponibili. Volendo invece utilizzare il 74LS273 a 8 bit, si potrebbe usare una porta AND per *costruire* un segnale di Enable: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse presente anche il segnale MI.

Il 74LS273, al pari del 74LS173, presenta un ingresso Clear / Reset (CLR), che nel MAR è necessario.

Altra ipotesi possibile: usare il 74LS377, che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il CLR, non presente in questo chip.

**METTERE QUALCHE FOTO E LINK**

>> 2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)

Come nel caso del MAR, per indirizzare 256 byte di RAM era necessario un registro Program Counter (PC) a 8 bit. Nel computer SAP era invece presente un contatore a 4 bit [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) e dovevo pertanto cercare di combinarne due in cascata.

Sarebbe stato comodo utilizzare un singolo contatore a 8 bit, ma tra i chip disponibili sul mercato non ne ho trovato uno che includesse anche l'ingresso LOAD. Il LOAD permette il caricamento parallelo sul PC di uno specifico indirizzo al quale il computer deve saltare (ad esempio, per eseguire un'istruzione di salto assoluto o branch relativo a un determinato indirizzo, si carica quel valore nel PC).

Per combinare due chip a 4 bit è stato necessario, nonché molto utile, capire la differenza fra "Ripple Mode Carry" e "Carry Look Ahead": il datasheet del contatore 74LS161 riportava infatti questi due diversi esempi di collegamento di chip in cascata tra di loro.

**METTERE QUALCHE FOTO E LINK**

>> 3. Instruction register - now sh!t gets fun. Since i want to bomb completely, I am considering having a 3 byte Instruction register in a way mimicking the BIU block & instruction queue in a 8086. The idea here is to split the instruction register in 3 bytes. 1 byte would be for the instruction/opcode alone, the second and third are going to hold the data of the instruction (either address or an immediate value). This would allow you to address the entire memory space. Also instructions can be of different size. For example. OUT (move to out register) would be just 1 byte wide, LDI (load immediate) would be 2 bytes and LDA (load address/absolute) would be 3 bytes wide. The control logic would take care of the fetch cycle. Or in other words since you know the instruction you are execution you would know how much bytes the instruction is and thus fetch either 1, 2 or 3 bytes from the RAM.

Dopo aver letto questo punto avevo iniziato a raccogliere i miei pensieri per l'espansione di memoria a 256 byte, idea che riprendo in seguito in questa pagina. Non avevo intenzione di costruire un IR a più byte, cosa che mi sembrava piuttosto complessa per le mie capacità.

### Memorie con IO separati o IO comuni?

Fino ad ora, avevo quasi sostanzialmente dato per scontato di continuare ad usare chip di memoria con porte di Input e Output separati, esattamente come accade nel [74189](https://eater.net/datasheets/74189.pdf) utilizzato nel SAP. Tuttavia, in questo [post su Reddit](https://www.reddit.com/r/beneater/comments/hon6ar/74189_alternative/
), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://web.mit.edu/6.115/www/document/62256.pdf); ho così iniziato ad approfondire le caratteristiche di questo chip, aumentando nel contempo la mia comprensione di queste due diverse architetture.

In origine avevo evidenziato questi pochi appunti presenti nel post, riflettendo sul fatto che l'approccio alla gestione dei segnali di controllo mi sembrava un po' troppo semplicistico:

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

Un altro aspetto che avevo notato immediatamente, ipotizzando l'uso del 62256, era l'impossibilità di mantenere la visibilità del contenuto della cella di RAM indirizzata dal MAR utilizzando i LED (o almeno così credevo): se con i 74189 potevo tenere "sempre acceso" l'output per vedere in ogni momento il valore contenuto della cella di memoria correntemente indirizzata dal MAR, con il 62256 avrei avuto visibilità del contenuto della cella solo nel momento in cui la RAM veniva letta - e dunque non costantemente.

[![Schema del modulo RAM di Ben Eater basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione ininterrotta del contenuto della locazione RAM indirizzata dal MAR](../../assets/be-ram-detail.png "Schema del modulo RAM basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione continua del contenuto della locazione RAM indirizzata dal MAR"){:width="50%"}](../../assets/be-ram-full.png)

Nello schema si notano i 74189 con le porte di Input dedicate D1-D4 e le porte di Output dedicate O1-O4.

Quello che iniziavo a capire era che per utilizzare una RAM con Common IO dovevo fare un "doppio passaggio" o qualcosa di simile. Come faccio ad avere sempre visibile il contenuto della locazione di memoria anche nel momento in cui setto le porte di IO del chip in modalità Input? Devo forse memorizzare il contenuto delle uscite della RAM in qualche latch e poi devo disabilitare il chip prima di andarvi a scrivere? In seguito avrei capito che non era necessario un latch. ma che c'era un'altra strada.

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; andavo capendo che la  soluzione potesse essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737). Cominciavo a intuire qualcosa, cioè la necessità di gestire i segnali di controllo della RAM in maniera più ampia controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo avere sia la possibilità di programmare la RAM manualmente ("Program Mode"), sia di renderla poi visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è utilizzata per scrivere manualmente sulla RAM utilizzando dei dip-switch e dunque caricare i programmi o leggere il contenuto della RAM in stile "debug mode".

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata dal MAR e acceduta in lettura / scrittura dal bus.

La selezione di cosa passare a RAM e MAR avviene mediante un MUX (nel nostro caso 2:1, cioè ad ogni uscita corrispondono due ingressi selezionabili): gli ingressi del MUX sono connessi sia ai dip-switch che utilizzeremo per la programmazione manuale del computer, sia al bus dati del computer; le uscite sono connesse agli ingressi della RAM e del MAR. Un semplice interruttore connesso all'ingresso di selezione del MUX consente di scegliere quali ingressi attivare.

Ad esempio, nello schema del SAP visibile più in alto in questa pagina i multiplexer 74LS157 gestiscono gli ingressi della RAM: gli ingressi del dei MUX sono connessi sia al dip-switch sia al bus del computer, mentre le uscite sono connesse alle porte di ingresso D1-D4 dei chip di RAM 74189.

Si ripresentava il tema del "doppio passaggio" (o meglio "doppio bus"), che da quanto iniziavo comprendere poteva rendere possibile la visualizzazione persistente del contenuto della RAM. A pagina 17 e 18 del "Building the SAP-3 rev 3.3.pdf" presente nel repository GitHub di <a href = "https://github.com/rolf-electronics/The-8-bit-SAP-3" target = "_blank">rolf-electronics</a>, altro utente del canale Reddit, avevo notato che era stato inserito un altro transceiver 74LS245.

[![Modulo RAM di rolf electronics](../../assets/40-rolf-ram.png "Modulo RAM di rolf electronics"){:width="50%"}](../../assets/40-rolf-ram.png)

Il funzionamento e la necessità dei transceiver mi erano chiarissimi, in quanto ampiamente utilizzati nel SAP computer per poter attivare i vari moduli del computer solo quando fosse necessario farlo; nello schema di Rolf i due transceiver vengono utilizzati per separare il percorso dei dati *verso* la RAM da quello *dalla* RAM.

- Il transceiver di sinistra è attivo quando si scrive *sulla* RAM, selezionando quale sia la sorgente mediante il multiplexer 74157 (che in "Program Mode" attiva gli ingressi connessi al dip-switch, mentre in "Run Mode" attiva gli ingressi connessi al bus). I led sono attivi e mostrano il valore che viene scritto sulla RAM.
- Il transceiver di destra è attivo quando si legge *dalla* RAM. I led sono attivi e mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.


[![Schema logico luglio 2023](../../assets/hand-drawn-logic.jpg "Schema logico luglio 2023"){:width="66%"}](../../assets/hand-drawn-logic.jpg)