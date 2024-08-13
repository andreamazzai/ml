---
title: "Registri D, X e Y"
permalink: /docs/dxy/
excerpt: "Registri indice del BEAM computer"
---
[![Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy.png "Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy.png)

Nel 6502 sono presenti due registri indice, X e Y, che possono facilmente essere riprodotti nel BEAM; sono registri indipendenti scritti e letti alla bisogna.

Il registro D dell'NQSAP e del BEAM viene utilizzato a supporto delle istruzioni di salto condizionale e di quelle che eseguono operazioni in una locazione di memoria che è il risultato di un computo tra indirizzo specificato nell'opcode e quattro diversi modi di interpretazione dei valori assunti da X o Y:

- Salti condizionali
- Indexed Addressing: Absolute, X
- Indexed Addressing: Absolute, Y
- Pre-Indexed Indirect: Zero-Page, X
- Post-Indexed Indirect: Zero-Page, Y

Per eseguire il computo si usano dei normali 4-Bit Binary Full Adders With Fast Carry <a href = "https://www.ti.com/lit/ds/symlink/sn54s283.pdf" target = "_blank">74LS283</a> che eseguono la somma tra quanto viene caricato nel registro D e il valore contenuto nei registri indice X o Y. Super clever.

Il flusso logico tratto dalla <a href = "https://tomnisbet.github.io/nqsap/docs/dxy-registers/" target = "_blank">spiegazione di Tom Nisbet</a> chiarisce il funzionamento:

![Registri D, X e Y in azione](../../assets/dxy/60-dxy-nqsap-tom-flow.png "Registri D, X e Y in azione"){:width="66%"}

### Utilizzo con le modalità di indirizzamento indicizzate

Il contenuto del registro D può essere sommato al contenuto dei registri X, Y o nessuno (potrebbe risultare utile come ulteriore registro di appoggio per definire altre istruzioni personalizzate; tuttavia, l'emulazione delle istruzioni del 6502 può essere implementata con gli altri registri esistenti nel computer senza utilizzare questa modalità).

Per selezionare se D deve essere sommato a X, Y o nulla, si usano dei multiplexer (MUX) <a href = "https://www.ti.com/lit/ds/symlink/sn74ls157.pdf" target = "_blank">74LS157</a> pilotati dai segnali DY e DZ nell'NQSAP e DX/Y e DZ nel BEAM.

Notare che il registro D è "Write only", dunque non è possibile metterne il contenuto in ouput sul bus. Tuttavia, è possibile leggere D portando il segnale DZ allo stato HI, disabilitando così le uscite dei '157 e causando gli adder a eseguire l'operazione D + 0 = D.

### Utilizzo per i salti condizionali

Il salto condizionale, come evidente dal nome, viene eseguito se una determinata situazione è verificata. Nell'esempio seguente l'istruzione BCS è senz'altro eseguita, in quanto il Carry è certamente presente. Il valore dell'operando $03 viene sommato all'indirizzo dell'istruzione *successiva* a quella di salto, cioé $83 + $03 = $86, che sarà il nuovo valore caricato nel  Program Counter (PC).

~~~txt
SEC         ; $80 - Set Carry Flag
BCS $03     ; $81 - Salta a $86 se il Carry è settato
INX         ; $83 - Questa istruzione sarà saltata
INX         ; $84 - Questa istruzione sarà saltata
INX         ; $85 - Questa istruzione sarà saltata
LDA #$01    ; $86 - Questa istruzione verrà eseguita
~~~

L'operando è un valore a 8 bit con segno (Signed), il che significa che può variare da -128 a +127. Questo significa che i salti condizionali possono saltare in avanti di 128 indirizzi e  all'indietro di 127.

Per tornare col salto condizionale a un indirizzo precedente a quello del salto, come in un loop, il valore dell'operando dovrà essere dunque, secondo la regola dei numeri Signed, un valore compreso tra $80 (-128) e $FF (-1), come visibile nella sezione [Numeri Unsigned e numeri Signed](../math/#Numeri-Unsigned-e-numeri-Signed) della pagina dedicata all'Aritmetica binaria.

~~~txt
LDX #$05    ; $80 - Carica il registro X con 5
DEX         ; $82 - Decrementa X
BNE $FD     ; $83 - Salta a $85 + $FD = $85 - $03 = $82 se X non è zero
RTS         ; $85 - Ritorno da subroutine
~~~

Poiché la configurazione degli Adder in questo modulo permette la sola esecuzione di addizioni, ci si potrebbe chiedere come sia possibile eseguire un salto all'indietro. Nell'NQSAP, così come nel BEAM, i registri sono a 8 bit e una scorciatoia per giungere al risultato desiderato è quello di semplicemente aggiungere l'operando al PC: nel caso specifico, $85 + $FD = $182 del quale rimarrà solo il $82 che verrà caricato nel PC.

[![Schema dei registri indice dell'NQSAP](../../assets/dxy/60-nqsap-dxy-schema.png "Schema dei registri indice dell'NQSAP"){:width="100%"}](../../assets/dxy/60-nqsap-dxy-schema.png)

*Schema dei registri indice dell'NQSAP.*

## Differenze tra Registri indice dell'NQSAP e del BEAM

Sotto il punto di vista funzionale, gli schemi dei Registri indice dell'NQSAP e del BEAM sono identici.

Negli appunti annotavo che "... come per gli altri registri del BEAM, anche qui uso dei Flip-Flop tipo D <a href="https://www.ti.com/lit/ds/symlink/sn54ls377.pdf" target="_blank">74LS377</a> anziché gli Octal D-Type Flip-Flop with 3-State Outputs <a href="https://www.onsemi.com/pdf/datasheet/74vhc574-d.pdf" target="_blank">74LS574</a> usati da Tom nell'NQSAP"; si veda la sezione [L'ALU dell'NQSAP](../alu/#lalu-dellnqsap) per un chiarimento in tal senso.

Per completezza, devo segnalare di aver conosciuto il 74LS377 studiando l'evoluzione dell'NQSAP, cioè l'<a href = "https://tomnisbet.github.io/nqsap-pcb/" target="_blank">NQSAP-PCB</a>, che Tom aveva ingegnerizzato su PCB anziché su breadboard come evoluzione dell'NQSAP originale.

[![Schema dei Registri indice del BEAM computer](../../assets/dxy/60-beam-dxy-schema.png "Schema dei Registri indice del BEAM computer"){:width="100%"}](../../assets/dxy/60-beam-dxy-schema.png)

*Schema dei registri indice del BEAM computer.*

# TO DO
- Scrivere forse un po meglio il fatto della lettura di D