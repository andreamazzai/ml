---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di Clock del BEAM computer"
---
[![Circuito di Clock del BEAM computer](../../assets/clock/15-beam-clock.png "Circuito di Clock del BEAM computer"){:width="100%"}](../../assets/clock/15-beam-clock.png)

Il circuito di clock del computer BEAM riprende il modulo di clock progettato da Ben Eater per il computer SAP, semplificandone leggermente l'utilizzo grazie a pulsanti tattili anziché interruttori a scorrimento e aggiungendo qualche interazione con i controlli esterni provenienti dal [Loader](../loader).

- Il primo 555 è utilizzato come classico generatore astabile con possibilità di selezionare la frequenza di funzionamento attraverso un potenziometro; un pulsante "Turbo" permette di bypassare temporaneamente la resistenza del potenziometro portando la frequenza di lavoro al valore massimo. Questo pulsante può essere utile se, durante una esecuzione a bassa velocità propedeutica a una analisi visiva degli output, diventi utile superare una routine particolarmente ripetitiva o impegnativa senza dover agire sul potenziometro.
- Il secondo 555 permette di eseguire singole microistruzioni ("Step").
- Il terzo 555 permette di avviare e fermare il clock continuo utilizzando un comodo pulsante "Start/Stop", anziché un interruttore come avveniva sul circuito del SAP.

Notare che una pressione del pulsante Step durante il normale funzionamento a clock continuo permette di fermare il computer alla microistruzione istruzione attuale e di poter proseguire in modalità "Step by Step"; premendo il pulsante Start/Stop, il computer riprenderà a lavorare in modalità clock continuo.

I pulsanti Start/Stop e Step sono stati duplicati anche in una zona più facilmente raggiungible del computer, cioè sotto alla Control Logic; in tale sede è stato incluso anche un pulsante di Reset.

### Il Loader e il Clock

Sul modulo di clock sono presenti tre segnali provenienti dal Loader basato su Arduino: LDR-Active, LDR-CLK e CLK-Start.

Un segnale LDR-Active HI permette al Loader di  prendere il controllo del circuito di clock, disattivandone completamente l'output e fermando l'esecuzione del programma. In questa circostanza, il Loader potrà iniettare nel computer il suo segnale di clock LDR-CLK, utilizzato per programmare la RAM.

Il segnale CLK-Start viene utilizzato dal Loader alla fine della programmazione per riattivare il modulo di clock e far partire il programma appena caricato.

[![Schema del modulo Clock](../../assets/clock/15-clock-schema.png "Schema del modulo Clock")](../../assets/clock/15-clock-schema.png)

*Schema del modulo Clock.*

## Note sul microcode

Il microcode dell'istruzione HLT del computer attiva l'omonimo segnale sul modulo di clock, interrompendo il funzionamento del programma. Per riprendere il funzionamento nella modalità a clock continuo, è necessario superare la microistruzione corrente premendo prima il pulsante Step e poi il pulsante Start/Stop.

## Link utili

- I <a href = "https://eater.net/8bit/clock" target = "_blank">video</a> di Ben Eater che descrivono il funzionamento dei timer 555 e la costruzione del modulo di clock.

## TO DO

- Verificare se ho scritto correttamente le note sul microcode.
- notare che in questa pagina il nome "modulo di clock" e "clock" sono usati per descrivere... maiuscolo o minuscolo?
