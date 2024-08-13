---
title: "Registri D, X e Y"
permalink: /docs/dxy/
excerpt: "Registri indice del BEAM computer"
---
[![Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy.png "Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy.png)

**PLACEHOLDER**

Nel 6502 sono presenti due registri indice, X e Y, che possono facilmente essere riprodotti nel BEAM; sono registri indipendenti scritti e letti alla bisogna.

Il registro D dell'NQSAP e del BEAM viene utilizzato a supporto delle istruzioni di salto condizionale e di quelle che eseguono operazioni in una locazione di memoria che è il risultato di un computo tra indirizzo specificato nell'opcode e quattro diversi modi di interpretazione dei valori assunti da X o Y:

- Salti condizionali
- Indexed Addressing: Absolute, X
- Indexed Addressing: Absolute, Y
- Pre-Indexed Indirect: Zero-Page, X
- Post-Indexed Indirect: Zero-Page, Y

Per eseguire il computo si usano dei normali 4-Bit Binary Full Adders With Fast Carry <a href = "https://www.ti.com/lit/ds/symlink/sn54s283.pdf" target = "_blank">74LS283</a> che eseguono la somma tra quanto viene caricato nel registro D e il valore contenuto nei registri indice X o Y. Super clever.

Il flusso logico tratto dalla <a href = "https://tomnisbet.github.io/nqsap/docs/dxy-registers/" target = "_blank">spiegazione di Tom Nisbet</a> chiarisce il funzionamento:

[![Registri D, X e Y in azione](../../assets/dxy/60-dxy-nqsap-tom-flow.png "Registri D, X e Y in azione"){:width="66%"}]

### Utilizzo con le modalità di indirizzamento indicizzate

Il contenuto del registro D può essere sommato al contenuto dei registri X, Y o nessuno (potrebbe risultare utile come ulteriore registro di appoggio per definire altre istruzioni personalizzate; tuttavia, l'emulazione delle istruzioni del 6502 può essere implementata con gli altri registri esistenti nel computer senza utilizzare questa modalità).

Per selezionare se D deve essere sommato a X, Y o nulla, si usano dei multiplexer (MUX) <a href = "https://www.ti.com/lit/ds/symlink/sn74ls157.pdf" target = "_blank">74LS157</a> pilotati dai segnali DY nell'NQSAP e DX/Y nel BEAM.

Negli appunti segnalavo che "... come per i registri A e B, anche qui uso dei Flip-Flop tipo D <a href="https://www.ti.com/lit/ds/symlink/sn54ls377.pdf" target="_blank">74LS377</a> anziché degli Octal D-Type Flip-Flop with 3-State Outputs <a href="https://www.onsemi.com/pdf/datasheet/74vhc574-d.pdf" target="_blank">74LS574</a> con la stessa necessità di emulare un Enable sfruttando una NOR con il segnale Write che arriva dalla ROM e il clock negato per attivare il Clock Enable di ingresso al momento giusto."

Notare che D è "Write only", dunque non è possibile metterne il contenuto in ouput sul bus. E' possibile leggere D portando il segnale DZ allo stato HI, disabilitando così le uscite dei '157 e causando gli adder a eseguire l'operazione D + 0 = D.

Ho conosciuto il 74LS377 studiando l'evoluzione dell'NQSAP, cioè l'<a href = "https://tomnisbet.github.io/nqsap-pcb/" target="_blank">NQSAP-PCB</a>, che Tom aveva ingegnerizzato su PCB anziché su breadboard.

### Utilizzo per i salti relativi

- Notare che l'operando delle istruzioni di salto condizionale è un valore relativo, che viene addizionato al valore attuale del PC. **Long Story Short: carico in D il PC attuale, mentre carico in X l'operando dell'istruzione di salto; il nuovo valore da caricare nel PC corissponde all'indirizzo del byte successivo a quello dell'operando dell'istruzione Branch + l'operando stesso**, ad esempio:


[![Schema dei registri indice dell'NQSAP](../../assets/dxy/60-nqsap-dxy-schema.png "Schema dei registri indice dell'NQSAP"){:width="100%"}](../../assets/dxy/60-nqsap-dxy-schema.png)

*Schema dei registri indice dell'NQSAP.*


## Differenze tra Registri indice dell'NQSAP e del BEAM

Sotto il punto di vista funzionale, gli schemi dei Registri indice dell'NQSAP e del BEAM sono identici: due registri un registro D che può essere due registri X e Y che possono essere sommati al registro D simili ; .

[![Schema dei Registri indice del BEAM computer](../../assets/dxy/60-dxy-beam-schema.png "Schema dei Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-dxy-beam-schema.png)

*Schema dei registri indice del BEAM computer.*
