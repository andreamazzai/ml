---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da zero a 255 e viceversa, oppure un algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Il primo articolo letto e dal quale avevo iniziato a ricavare un po' di informazioni è [questo](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) e ne avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte, ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve un registro MAR (Memory Address Register) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173 (il 74LS173 è il Flip-Flop utilizzato in origine nel SAP), sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando necessario - e non ad ogni ciclo di clock. Il segnale MI (Memory Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario; quando MI è attivo, al successivo rising edge del CLK il MAR carica il valore presente nel bus. Senza un ingresso di Enable, il Flip-Flop (FF) 74LS273 andrebbe a "registrare" il dato in ingresso ad ogni rising edge del clock.

In un progetto a 8 bit si potrebbero semplicemente utilizzare due 74LS173 a 4 bit continuando a sfruttare i segnali di Enable nativamente disponibili. Bisogna dire che sarebbe comunque possibile utilizzare anche il 74LS273 a 8 bit utilizzando una porta AND per *costruire* un segnale di Enable: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse contemporaneamente presente anche il segnale MI.

Il 74LS273, al pari del 74LS173, presenta un ingresso Clear / Reset (CLR), che nel MAR è necessario per resettare il registro - o almeno *credevo* necessario. Sembrava anche interessante l'ipotesi alternativa di usare un registro a 8 bit [74LS377](https://datasheetspdf.com/download_new.php?id=375625), che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il CLR, non presente in questo chip. Come ormai si sarà capito, ho realizzato successivamente che il MAR può funzionare perfettamente anche senza un segnale di Clear / Reset.

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
), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://datasheetspdf.com/download_new.php?id=729365); ho così iniziato ad approfondire le caratteristiche di questo chip, aumentando nel contempo la mia comprensione di queste due diverse architetture.

In origine avevo evidenziato questi pochi appunti presenti nel post, riflettendo sul fatto che l'approccio alla gestione dei segnali di controllo mi sembrava un po' troppo semplicistico:

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

Un altro aspetto che avevo notato immediatamente, ipotizzando l'uso del 62256, era l'impossibilità di mantenere la visibilità del contenuto della cella di RAM indirizzata dal MAR utilizzando i LED (o almeno così credevo): se con i 74189 le porte di output erano sempre attive e potevo vedere in ogni momento il valore contenuto della cella di memoria correntemente indirizzata dal MAR, con il 62256 avrei avuto visibilità del contenuto della cella solo nel momento in cui la RAM veniva letta - e dunque non costantemente.

[![Schema del modulo RAM di Ben Eater basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione ininterrotta del contenuto della locazione RAM indirizzata dal MAR](../../assets/be-ram-detail.png "Schema del modulo RAM basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione continua del contenuto della locazione RAM indirizzata dal MAR"){:width="50%"}](../../assets/be-ram-full.png)

*Nello schema si notano i 74189 con le porte di Input dedicate D1-D4 e le porte di Output dedicate O1-O4.*

Quello che iniziavo a capire era che per utilizzare una RAM con Common IO dovevo fare un "doppio passaggio" o qualcosa di simile. Come faccio ad avere sempre visibile il contenuto della locazione di memoria anche nel momento in cui setto le porte di IO del chip in modalità Input? Devo forse memorizzare il contenuto delle uscite della RAM in qualche latch e solo a quel punto disabilito il chip prima di andarvi a scrivere? In seguito avrei capito che non era necessario un latch, ma che c'era un'altra strada.

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; andavo capendo che la  soluzione potesse essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737); intuivo qualcosa, relativamente alla necessità di gestire i segnali di controllo della RAM in maniera più ampia, controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo avere sia la possibilità di programmare la RAM ("Program Mode"), sia di renderla poi visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è manuale e sfrutta dei dip-switch per scrivere manualmente sulla RAM per caricare i programmi (o leggere il contenuto della RAM in stile "debug mode").

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata dal MAR e viene acceduta in lettura / scrittura esclusivamente dal bus del computer.

La selezione di cosa passare a RAM e MAR avviene mediante un MUX (nel nostro caso 2:1, cioè ad ogni uscita corrispondono due ingressi selezionabili): gli ingressi del MUX sono connessi sia ai dip-switch che utilizzeremo per la programmazione manuale del computer, sia al bus dati del computer; le uscite sono connesse agli ingressi della RAM e del MAR. Un semplice interruttore connesso all'ingresso di selezione del MUX consente di scegliere quali ingressi attivare.

Ad esempio, nello schema del SAP visibile più in alto in questa pagina, i multiplexer 74LS157 gestiscono gli ingressi della RAM: gli ingressi dei MUX sono connessi sia al dip-switch sia al bus del computer, mentre le uscite sono connesse alle porte di ingresso D1-D4 dei chip di RAM 74189.

Si ripresentava il tema del "doppio passaggio" (o meglio "doppio bus", come capirò in seguito), che da quanto iniziavo a comprendere poteva rendere possibile la visualizzazione persistente del contenuto della RAM. A pagina 17 e 18 del "Building the SAP-3 rev 3.3.pdf" presente nel repository GitHub di <a href = "https://github.com/rolf-electronics/The-8-bit-SAP-3" target = "_blank">rolf-electronics</a>, altro utente del canale Reddit, avevo notato che era stato inserito un altro transceiver 74LS245.

[![Modulo RAM di rolf electronics](../../assets/40-rolf-ram.png "Modulo RAM di rolf electronics"){:width="50%"}](../../assets/40-rolf-ram.png)

Il funzionamento e la necessità dei transceiver mi erano chiarissimi, in quanto ampiamente utilizzati nel SAP computer per poter attivare i vari moduli del computer solo nel momento in cui fosse necessario farlo; nello schema di Rolf i due transceiver vengono utilizzati per separare il percorso dei dati *verso* la RAM da quello *dalla* RAM.

- Il transceiver di sinistra è attivo quando si scrive *sulla* RAM, selezionando quale sia la sorgente mediante il multiplexer 74157 (che in "Program Mode" attiva gli ingressi connessi al dip-switch, mentre in "Run Mode" attiva gli ingressi connessi al bus). I led sono attivi e mostrano il valore che viene scritto sulla RAM.
- Il transceiver di destra è attivo quando si legge *dalla* RAM. I led sono attivi e mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.

Proseguendo nello studio, ho trovato [questo schema](https://imgur.com/a/ruclh), del quale purtroppo non trovo più il link che mi ci ha portato:

[![RAM e MAR con doppio bus](../../assets/40-ram-ruclh.png "RAM e MAR con doppio bus"){:width="66%"}](../../assets/40-ram-ruclh.png)

*Schema di RAM e MAR con bus interno*

In questo schema troviamo:

- Il chip di RAM 62256.

- Un FF 74LS273 in alto a destra che si attiva in corrispondenza di Clock + MI (vedi ingressi 1A e 1B del NAND centrale a sinistra nello schema).

- I due MUX 74LS157 in alto che consentono la selezione degli indirizzi (tra quelli settati sui dip-switch o quelli presenti sulle uscite del FF 74LS273) da esportare verso i pin A0-A7 del chip di RAM; la selezione degli ingressi attivati dal MUX avviene grazie all'interruttore (in alto a sinistra) di selezione della modalità connesso agli ingressi SEL: in Program Mode è acceso il LED rosso e gli ingressi SEL sono allo stato LO, attivando gli ingressi A1-A4; in Run Mode è acceso il LED verde e gli ingressi SEL sono allo stato HI, attivando gli ingressi B1-B4.

- Altri due MUX 74LS157 in basso che consentono la selezione di cosa esportare verso i pin dati D0-D7 del chip di RAM; anche questi MUX sono connessi all'interruttore di selezione della modalità e attivano gli ingressi connessi al dip-switch di selezione degli indirizzi o quelli connessi al bus del computer a seconda dello stato dell'interruttore (Program Mode o Run Mode).

- Un primo 74LS245 (in basso) i cui ingressi sono connessi alle uscite dei due MUX citati nel punto precedente. Questo transceiver funge da interfaccia *verso* la RAM (il pin DIR settato a LO configura i pin A1-A8 come ingressi e i pin B1-B8 come uscite) e si attiva nel momento in cui si deve scrivere in memoria; il segnale OE di questo primo transceiver è infatti attivo quando quando si preme il pulsante Write Button (in basso a sinistra) se in Program Mode, o in corrispondenza di Clock + RI (RAM In) (vedi ingressi 4A e 4B del NAND centrale a sinistra nello schema) quando in Run Mode;

- Un secondo 74LS245 che si attiva nel momento in cui si deve leggere *dalla* RAM e che ne trasferisce l'output verso il bus dati (anche in questo caso il pin DIR del 74LS245 settato a LO configura i pin A1-A8 come ingressi e i pin B1-B8 come uscite); notare il suo ingresso OE connesso al segnale RO (RAM Output) del computer.

Notare la configurazione del chip di RAM: i segnali CE e OE sono sempre attivi, che significa che l'utente ha deciso di utilizzare la modalità di scrittura definita come "WE# Controlled" a pagina 6 del [datasheet](https://datasheetspdf.com/download_new.php?id=729365). Rileggendo questi appunti dopo aver completao il mio progetto, mi sembra tutto facile, ma la comprensione delle modalità di scrittura della RAM è stata in realtà piuttosto lunga.
