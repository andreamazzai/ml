---
title: "Registri D, X e Y"
permalink: /docs/dxy/
excerpt: "Registri indice del BEAM computer"
---
[![Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy.png "Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy.png)

**PLACEHOLDER**

Nel 6502 sono presenti due registri indice, X e Y, che possono facilmente essere riprodotti nel BEAM; sono registri indipendenti, che possono essere caricati e letti alla bisogna.

Il registro D viene utilizzato a supporto delle istruzioni di salto condizionale e di quelle che eseguono operazioni in una locazione di memoria che è il risultato di un computo tra indirizzo di un specifica locazione di memoria di partenza e diversi modi di interpretazione dei valori assunti da X o Y:

- Salti condizionali
- Indexed Addressing: Absolute, X
- Indexed Addressing: Absolute, Y
- Pre-Indexed Indirect: Zero-Page, X
- Post-Indexed Indirect: Zero-Page, Y

Per eseguire il computo si usano dei normali Adder che eseguono la somma tra quanto viene caricato nel registro D e il valore contenuto nei registri indice X o Y. Super clever.





 • Vedi https://tomnisbet.github.io/nqsap/docs/dxy-registers/; 
Il flusso logico s

Il flusso logico tratto dalla <a href = "https://tomnisbet.github.io/nqsap/docs/dxy-registers/" target = "_blank">spiegazione di Tom Nisbet</a> chiarisce il funzionamento:

[![Registri D, X e Y in azione](../../assets/dxy/60-dxy-nqsap-tom-flow.png "Registri D, X e Y in azione"){:width="100%"}](../../assets/dxy/60-dxy-nqsap-tom-flow.png)

60-dxy-nqsap-tom-flow

• Come per registri A e B, anche qui uso dei 574 con la stessa necessità di emulare un Enable sfruttando una NOR con il segnale Write che arriva dalla ROM e il clock negato per attivare il Clock Enable di ingresso al momento giusto.
• Nota che nell'NQSAP-PCB per X e Y utilizza dei FF 377 e non più dei FF 574…
	§ … 12/06/2023 e potrei farlo anche io, così da risparmiare la NOR e collegare direttamente WX e WY ai FF '377, tanto sono sempre attivi e possono essere letti attraverso RX ed RY connessi ai 245 relativi
	§ … e posso usare il 377 anche per D, che comunque è solo in ingresso (e dunque anche un 574 potrebbe essere OK, però con la complicazione del dover gestire il segnale di Enable ∨ /CLK)

[![Schema dei registri indice dell'NQSAP](../../assets/dxy/60-nqsap-dxy-schema.png "Schema dei registri indice dell'NQSAP"){:width="100%"}](../../assets/dxy/60-nqsap-dxy-schema.png)

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

Sotto il punto di vista funzionale, gli schemi dei Registri indice dell'NQSAP e del BEAM sono identici: due registri un registro D che può essere due registri X e Y che possono essere sommati al registro D simili ; .


[![Schema dei Registri indice del BEAM computer](../../assets/dxy/60-dxy-beam-schema.png "Schema dei Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-dxy-beam-schema.png)

*Schema dei registri indice del BEAM computer.*