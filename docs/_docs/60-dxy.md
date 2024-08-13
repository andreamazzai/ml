---
title: "Registri D, X e Y"
permalink: /docs/dxy/
excerpt: "Registri indice del BEAM computer"
---
[![Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy.png "Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy.png)

**PLACEHOLDER**

• Vedi https://tomnisbet.github.io/nqsap/docs/dxy-registers/; praticamente per fare operazioni "OP $XX + X" o "OP $XX + Y" si usano gli Adder, come quelli del primo SAP (che però erano usati per fare la somma di dati, non di indirizzi). Si somma l'indirizzo $XX (che viene messo nel registro D) e gli indici X o Y. Super clever.
• Come per registri A e B, anche qui uso dei 574 con la stessa necessità di emulare un Enable sfruttando una NOR con il segnale Write che arriva dalla ROM e il clock negato per attivare il Clock Enable di ingresso al momento giusto.
• Nota che nell'NQSAP-PCB per X e Y utilizza dei FF 377 e non più dei FF 574…
	§ … 12/06/2023 e potrei farlo anche io, così da risparmiare la NOR e collegare direttamente WX e WY ai FF '377, tanto sono sempre attivi e possono essere letti attraverso RX ed RY connessi ai 245 relativi
	§ … e posso usare il 377 anche per D, che comunque è solo in ingresso (e dunque anche un 574 potrebbe essere OK, però con la complicazione del dover gestire il segnale di Enable ∨ /CLK)

[![Schema dei registri indice dell'NQSAP](../../assets/dxy/60-dxy-beam-schema.png "Schema dei registri indice dell'NQSAP"){:width="100%"}](../../assets/dxy/60-dxy-beam-schema.png)

*Schema dei registri indice dell'NQSAP.*

• Si usano dei 157 MUX per selezionare se X o Y che attraverso dei 283 vengono sommati a D.
• D è Write Only, dunque non lo si può leggere dal bus. Per questo non serve un 245. E' possibile leggere D mettendo 0 in X o Y e poi dall'uscita degli adder vedrò solo D. 
• Le modalità di indirizzamento possibili sono:
	○ implicit
	○ accumulator
	○ immediate
	○ relative
	○ absolute
	○ absolute + X
	○ absolute + Y
	○ indirect
	○ indexed indirect (with X)
	○ indirect indexed (with Y)

![Alt text](image-1.png)

![Alt text](image-2.png)
• DY:
	• se HI seleziona gli ingressi Y
	• se LO seleziona gli ingressi X
	• dunque posso leggere il contenuto di X o Y, che hanno gli output sempre attivi e interfacciati col bus mediante due '245.
• DZ:
	• se HI disabilita le uscite dei MUX 157 che vanno a zero e dunque l'uscita dell'adder è D+0 = D
posso leggere il contenuto di D, sempre mediante un '245



- Notare che l'operando delle istruzioni di salto condizionale è un valore relativo, che viene addizionato al valore attuale del PC. **Long Story Short: carico in D il PC attuale, mentre carico in X l'operando dell'istruzione di salto; il nuovo valore da caricare nel PC corissponde all'indirizzo del byte successivo a quello dell'operando dell'istruzione Branch + l'operando stesso**, ad esempio:

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

## Differenze tra Registri indice dell'NQSAP e del BEAM

Come si può vedere dallo schema del modulo ALU del computer BEAM, questo è quasi una copia 1:1 del modulo ALU del computer NQSAP: non avevo certamente la capacità di sviluppare autonomamente un modulo ALU così complesso e legato a doppio filo con altri moduli del computer, ma la comprensione completa del funzionamento dell'ALU sviluppata da Tom ha rappresentato comunque un traguardo molto importante.

[![Schema dei registri indice del BEAM computer](../../assets/dxy/60-dxy-beam-schema.png "Schema dei registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-dxy-beam-schema.png)

*Schema dei registri indice del BEAM computer.*