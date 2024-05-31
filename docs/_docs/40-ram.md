---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
# RAM

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da zero a 255 e viceversa, oppure un algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la ricerca e la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a leggere blog, forum e post per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il primo articolo letto e dal avevo iniziato a ricavare un po' di informazioni è [questo](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/).

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

In origine avevo evidenziato questi pochi appunti, riflettendo sul fatto che l'approccio alla gestione dei segnali di controllo citati nel post mi sembrava un po' troppo semplicistico:

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

Un altro aspetto che avevo notato immediatamente, ipotizzando l'uso del 62256, era l'impossibilità di mantenere la visibilità del contenuto della cella di RAM indirizzata dal MAR utilizzando i LED: se con i 74189 potevo tenere "sempre acceso" l'output per vedere in ogni momento il valore contenuto della cella di memoria correntemente indirizzata dal MAR, con il 62256 avrei avuto visibilità del contenuto della cella solo nel momento in cui la RAM veniva indirizzata - e dunque non costantemente.

[![Schema del modulo RAM di Ben Eater basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione ininterrotta del contenuto della locazione RAM indirizzata dal MAR](../../assets/be-ram-detail.png "Schema del modulo RAM basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione continua del contenuto della locazione RAM indirizzata dal MAR"){:width="50%"}](../../assets/be-ram-full.png)

Nello schema si notano i 74189 con le porte di Input dedicate D1-D4 e le porte di Output dedicate O1-O4.

Il 4x FF 74LS173 del Memory Address Register, quando /MI è attivo, legge dal bus (4x LSB) l'indirizzo da attivare e mediante il multiplexer 74LS157 lo passa alla RAM. Il MUX serve per selezionare l'indirizzo da passare alla RAM (A0, A1, A2, A3) prendendolo dal dip-switch in Program Mode o dal bus in Run Mode.

L'uscita della RAM 74189 è sempre attiva, dunque i LED collegati agli output mostrano sempre il contenuto della RAM ed è sufficiente attivare /RO per esporre il contenuto della RAM sul bus attraverso il buffer Octal Bus Transceiver 74LS245.

**APPROFONDIRE - spiegare meglio** In pratica, il FF 74LS173 del Memory Address Register, con /MI è attivo, legge dal bus (4 bit LSB) l'indirizzo da attivare e mediante il multiplexer 74LS157 lo passa alla RAM (il MUX '157 permette di selezionare l'indirizzo a 4 bit (A0, A1, A2, A3) da presentare alla RAM scegliendolo tra dip-switch in Program Mode o bus quando in Run Mode).

Cosa significa "Program Mode" o "Run Mode"? Sono due modalità di accesso alla RAM selezionabili mediante uno switch:

* la modalità Program Mode è utilizzata per scrivere manualmente sulla RAM utilizzando i dip-switch, e dunque caricare i programmi o leggere il contenuto della RAM in stile "debug mode"
    - devo settare PROG LO, così il MUX 74LS157 esporrà agli /WE della RAM il pulsante di programmazione I0d
* Run Mode è la modalità di esecuzione, nella quale la RAM viene indirizzata dal MAR e acceduta in lettura / scrittura dal bus.
    - PROG diventa HI e dunque il /WE sarà quello dato da I1c (31/05/2023 prima avevo scritto  i1d ma probabilmente avevo sbagliato) impulso di clock alto AND segnale RI alto (RAM Input) attraverso la NAND, che metterà in output un LO (che combacia col /WE delle RAM 189). In questo modo i MUX che leggono i dati dal bus passano ai D1, D2, D3 e D4 della RAM il contenuto del bus stesso.


Per utilizzare una RAM con Common IO, devo fare un "doppio passaggio" o qualcosa di simile.
Come faccio ad avere sempre visibile il contenuto della cella di RAM indirizzata, se devo andarvi a scrivere e dunque disabilito la sua uscita? Devo memorizzare il contenuto delle uscite in qualche latch e poi devo disabilitare le uscite prima di andarvi a scrivere?
Questo utente https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/ ha fatto un disegno che copio qui sotto e che forse potrebbe andare bene, ma è necessario disabilitare le uscite dei multiplexer altrimenti ci sarebbe contenzioso con le uscite della RAM quando questa è attiva (e dunque si dovrebbe mettere un altro 74LS245, credo)… 02/09/2022 ma questi MUX non sono tri-state, dunque sono sempre attivi… e allora…
… viene detto che in alternativa ai 74LS157 si potrebbero usare i 257, che sono tri-state. (02/09/2022 ora che ho capito il giro del fumo, confermo che bisogna fare un bus "interno" usando un altro 74LS245…
	però forse potrei usare i MUX 257, togliere il 245 che dalle uscite dei MUX va agli ingressi/uscite della RAM e collegare E dei MUX alla NOT dopo RI, cosi quando devo scrivere su RAM (RI, RAM Input) questo segnale è a HI e la NOT è LO e mi abilita i MUX che dunque mettono in uscita il contenuto del bus così da inserirlo in RAM. Risparmio un 245.
Però attenzione perché questo continua a essere a 4 bit di address, mentre io voglio passare a 8 bit = 256 byte

[![Schema logico luglio 2023](../../assets/hand-drawn-logic.jpg "Schema logico luglio 2023"){:width="66%"}](../../assets/hand-drawn-logic.jpg)