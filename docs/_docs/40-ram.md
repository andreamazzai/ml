---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
# RAM

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da zero a 255 e viceversa, oppure un algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la ricerca e la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a leggere blog e forum relativi all'argomento.

### Primi studi

Il primo articolo letto è [questo](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) e avevo iniziato a ricavare un po' di informazioni.

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte, ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve un registro MAR (Memory Address Registers) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173, come quelli utilizzati in origine nel SAP, sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando necessario - e non ad ogni ciclo di clock. Il segnale MI (Memory Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario; quando MI è attivo, al successivo rising edge del CLK il MAR carica il valore presente nel bus. Senza un ingresso di Enable, il FF 74LS273 andrebbe a "registrare" il dato in ingresso ad ogni rising edge del clock.

In un progetto a 8 bit si potrebbero semplicemente utilizzare due 74LS173 a 4 bit continuando a sfruttare i segnali di Enable. Volendo invece utilizzare il 74LS273, si dovrebbe usare una porta AND per *costruire* un segnale di Enable: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse presente anche il segnale MI. Questo chip ha il CLR, che nel MAR è richiesto.

Altra ipotesi possibile: usare il 74LS377, che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il reset (CLR), non presente in questo chip.

**METTERE QUALCHE FOTO E LINK**

>> 2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)

Come nel caso del MAR, per indirizzare 256 byte di RAM era necessario un registro Program Counter (PC) a 8 bit. Nel computer SAP era invece presente un contatore a 4 bit [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) e dovevo pertanto cercare di combinarne due in cascata.

Sarebbe stato comodo utilizzare un singolo contatore a 8 bit, ma tra i chip disponibili sul mercato non ne ho trovato uno che includesse l'ingresso LOAD. Il LOAD permette il caricamento parallelo di uno specifico indirizzo al quale il computer deve saltare (ad esempio, per eseguire un'istruzione di salto assoluto o branch relativo si carica quel valore nel PC).

Per combiare due chip a 4 bit è stato necessario, nonché molto utile, capire la differenza fra "Ripple Mode Carry" e "Carry Look Ahead": il datasheet del contatore 74LS161 riportava infatti questi due diversi esempi di collegamento di chip in cascata tra di loro.

>> 3. Instruction register - now sh!t gets fun. Since i want to bomb completely, I am considering having a 3 byte Instruction register in a way mimicking the BIU block & instruction queue in a 8086. The idea here is to split the instruction register in 3 bytes. 1 byte would be for the instruction/opcode alone, the second and third are going to hold the data of the instruction (either address or an immediate value). This would allow you to address the entire memory space. Also instructions can be of different size. For example. OUT (move to out register) would be just 1 byte wide, LDI (load immediate) would be 2 bytes and LDA (load address/absolute) would be 3 bytes wide. The control logic would take care of the fetch cycle. Or in other words since you know the instruction you are execution you would know how much bytes the instruction is and thus fetch either 1, 2 or 3 bytes from the RAM.

Dopo aver letto questo punto avevo iniziato a raccogliere i miei pensieri per l'espansione di memoria a 256 byte, idea che riprendo in seguito in questa pagina. Non avevo intenzione di costruire un IR a più byte, cosa che mi sembrava piuttosto complessa per le mie capacità.

### Altro approfondimento
