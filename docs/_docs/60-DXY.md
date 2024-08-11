---
title: "Registri D, X e Y"
permalink: /docs/dxy/
excerpt: "Registri indice del BEAM computer"
---
[![Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy.png "Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy.png)

PLACEHOLDER

Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da 0 a 255 e viceversa, oppure un piccolo algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

- Notare che l'operando delle istruzioni di salto condizionale è un valore relativo, che viene addizionato al valore attuale del PC. **Long Story Short: carico in D il PC attuale, mentre carico in X l'operando dell'istruzione di salto; il nuovo valore da caricare nel PC corissponde all'indirizzo del byte successivo a quello dell'operando dell'istruzione Branch + l'operando stesso**, ad esempio:

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

## Differenze tra Registri indice dell'NQSAP e del BEAM

Come si può vedere dallo schema del modulo ALU del computer BEAM, questo è quasi una copia 1:1 del modulo ALU del computer NQSAP: non avevo certamente la capacità di sviluppare autonomamente un modulo ALU così complesso e legato a doppio filo con altri moduli del computer, ma la comprensione completa del funzionamento dell'ALU sviluppata da Tom ha rappresentato comunque un traguardo molto importante.

[![Schema logico dei registri indice del BEAM computer](../../assets/dxy/60-dxy-beam-schema.png "Schema logico dei registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-dxy-beam-schema.png)

*Schema logico dei registri indice del BEAM computer.*
