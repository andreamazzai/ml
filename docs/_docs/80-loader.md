---
title: "Arduino Loader"
permalink: /docs/loader/
excerpt: "Loader del BEAM computer"
---
[![Loader del BEAM computer](../../assets/loader/80-beam-loader.png "Loader del BEAM computer"){:width="100%"}](../../assets/loader/80-beam-loader.png)

## PLACEHOLDER - pagina non iniziata

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da 0 a 255 e viceversa, oppure un piccolo algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. Thileggendo.

[![Terza versione del modulo RAM](../../assets/20-ram-3rd.png "Terza versione del modulo RAM"){:width="100%"}](../../assets/20-ram-3rd.png)

*Terza versione del Modulo di memoria (RAM) del BEAM.*

[![Schema del Loader del BEAM computer](../../assets/loader/80-loader-schema.png "Schema del Loader del BEAM computer")](../../assets/loader/80-loader-schema.png)

*Schema del Loader del BEAM computer.*
