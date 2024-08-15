---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di clock del BEAM computer"
---
[![Circuito di Clock del BEAM computer](../../assets/clock/15-beam-clock.png "Circuito di Clock del BEAM computer"){:width="100%"}](../../assets/clock/15-beam-clock.png)

Il circuito di clock del computer BEAM riprende il modulo di clock progettato da Ben Eater per il computer SAP, utilizzando comodi pulsanti tattili anziché interruttori a scorrimento e aggiungendo qualche interazione con i controlli esterni provenienti dal [Loader](../loader).

- Il primo 555 è utilizzato come generatore astabile con possibilità di selezionare la frequenza di funzionamento attraverso un potenziometro; un pulsante "Turbo" permette di bypassare temporaneamente la resistenza del potenziometro portando la frequenza di lavoro al valore massimo. Se si esegue qualche programma a bassa velocità di clock allo scopo di effettuare una analisi visiva degli output, questo pulsante si comporta come un "Fast Forward".
- Il secondo 555 in modalità monostabile permette di eseguire singole microistruzioni ("Step").
- Il terzo 555 in modalità bistabile permette di avviare e fermare il clock del primo 555 utilizzando un comodo pulsante "Start/Stop", anziché un interruttore come avveniva sul circuito del SAP.

Una pressione del pulsante Step durante il normale funzionamento (astabile) ferma il computer alla microistruzione attuale e permette di proseguire in modalità "Step by Step" premendo ripetutamente lo stesso pulsante; premendo il pulsante Start/Stop, il computer riprende a lavorare col clock astabile.

I pulsanti Start/Stop e Step sono stati duplicati anche in una zona più facilmente raggiungible del computer, cioè sotto alla Control Logic; in tale sede è stato incluso anche un pulsante di Reset.

[![Pulsanti Start/Stop, Step, Reset](../../assets/clock/15-run-step-reset.png "Pulsanti Start/Stop, Step, Reset")]

### Il Loader e il Clock

Sul modulo di clock sono presenti tre segnali provenienti dal Loader basato su Arduino: LDR-Active, LDR-CLK e CLK-Start.

Il segnale **LDR-Active** allo stato HI permette al Loader di  prendere il controllo del circuito di clock, disattivandone completamente l'output e fermando l'esecuzione del programma. In questa circostanza, il Loader può iniettare nel computer il suo segnale di clock **LDR-CLK**, utilizzato per programmare la RAM.

Il segnale **CLK-Start** viene utilizzato dal Loader alla fine della programmazione per riattivare il modulo di clock e far partire il programma appena caricato.

[![Schema del modulo Clock](../../assets/clock/15-clock-schema.png "Schema del modulo Clock")](../../assets/clock/15-clock-schema.png)

*Schema del modulo Clock.*

## Note sul microcode

Il microcode dell'istruzione HLT attiva l'omonimo segnale sul modulo di clock, interrompendo il funzionamento del programma. Per riprendere il funzionamento nella modalità a clock continuo, è necessario superare la microistruzione corrente premendo il pulsante Step: a questo punto sarà possibile riavviare l'esecuzione del programma col pulsante Start/Stop.

## Link utili

- I <a href="https://eater.net/8bit/clock" target="_blank">video</a> di Ben Eater che descrivono il funzionamento dei timer 555 e la costruzione del modulo di clock.
- <a href="https://todbot.com/blog/2010/01/02/momentary-button-as-onoff-toggle-using-555/" target="_blank">Un blog</a> che spiega come creare un circuito bistabile, che è quanto mi serviva per poter crere la funzione "Start/Stop" con pulsante anziché con interruttore.

## TO DO

- Verificare se ho scritto correttamente le note sul microcode.
- notare che in questa pagina il nome "modulo di clock" e "clock" sono usati per descrivere... maiuscolo o minuscolo?
