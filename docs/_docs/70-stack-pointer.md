---
title: "Stack Pointer"
permalink: /docs/stack-pointer/
excerpt: "Registro Stack Pointer del BEAM computer"
---
[![Registro Stack Pointer del BEAM computer](../../assets/sp/70-beam-sp.png "Registro Stack Pointer del BEAM computer"){:width="100%"}](../../assets/sp/70-beam-sp.png)

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da 0 a 255 e viceversa, oppure un piccolo algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

PLACEHOLDER

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

[![Schema logico dello Stack Pointer del computer BEAM](../../assets/sp/70-stack-pointer-schema.png "Schema logico dello Stack Pointer del computer BEAM"){:width="100%"}](../../assets/sp/70-stack-pointer-schema.png)

*Schema logico dello Stack Pointer del computer BEAM.*

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
