---
title: "Stack Pointer"
permalink: /docs/stack-pointer/
excerpt: "Registro Stack Pointer del BEAM computer"
---
PLACEHOLDER

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da 0 a 255 e viceversa, oppure un piccolo algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte (e non complicarmi troppo la vita con un bus a 16 bit), ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve un registro MAR (Memory Address Register) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173 (il '173 è il Flip-Flop utilizzato in origine nel SAP), sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando istruito a farlo - e non ad ogni ciclo di clock. Il segnale MI (Memory Address Register Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario farlo: con MI attivo, in corrispondenza del rising edge del clock il MAR memorizza il valore presente nel bus. Senza un ingresso di Enable, il FF '273 andrebbe a "registrare" il dato in corrispondenza di ogni ciclo di clock e non solo quando fosse necessario.

Nel mio progetto a 8 bit si potrebbero semplicemente utilizzare due '173 a 4 bit continuando a sfruttare i segnali di Enable nativamente disponibili. Bisogna dire che sarebbe comunque possibile utilizzare anche il '273 a 8 bit utilizzando una porta AND per *costruire* un segnale di Enable artificiale: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse contemporaneamente presente anche il segnale MI.
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
