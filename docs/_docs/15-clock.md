---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di Clock del BEAM computer"
---
[![Circuito di Clock del BEAM computer](../../assets/clock/15-beam-clock.png "Circuito di Clock del BEAM computer"){:width="100%"}](../../assets/clock/15-beam-clock.png)

Il circuito di clock del computer BEAM riprende il modulo di clock progettato da Ben Eater ([link](https://eater.net/8bit/clock)) per il computer SAP, semplificandone leggermente l'utilizzo grazie a interruttori tattili anziché interruttori a scorrimento e aggiungendo qualche interazione con i controlli esterni provenienti dal [Loader](../loader).

- Il primo 555 è utilizzato come classico generatore astabile con possibilità di selezionare la frequenza di funzionamento attraverso un potenziometro; un pulsante "Turbo" permette di bypassare temporaneamente la resistenza del potenziometro portando la frequenza di lavoro al valore massimo. Questo pulsante può essere utile se, durante una fase di esecuzione a bassa elocità del programma propedeutica a una analisi visiva degli output, sia utile superare una routine particolarmente ripetitiva o impegnativa senza dover agire sul potenziometro.
- Il secondo 555 permette di eseguire singole microistruzioni ("Step").
- Il terzo 555 permette di avviare e fermare il clock continuo utilizzando un comodo bottone "Start/Stop", anziché un interruttore come avveniva sul circuito di Ben.

Notare che una pressione dell'interruttore Step durante il normale funzionamento a clock continuo permette di fermare il computer alla microistruzione istruzione attuale e di poter proseguire in modalità "Step by Step". Premendo l'interruttore Start/Stop il computer riprende a lavorare in modalità clock continuo.

Gli interruttori Start/Stop e Step sono stati duplicati anche in una zona più facilmente raggiungible del computer, cioè sotto alla Control Logic; in tale sede è stato incluso anche un interruttore di Reset.

Sul modulo sono presenti tre segnali provenienti dal Loader: LDR-Active, LDR-CLK e CLK-Start.

Un segnale LDR-Active HI indica che il Loader prende il controllo del circuito di clock disattivandone completamente l'output e fermando l'esecuzione del programma. In questa circostanza, il Loader potrà iniettare il suo segnale di clock nel computer LDR-CLK.

Il segnale CLK-Start viene utilizzato dal Loader alla fine della programmazione per riattivare il modulo clock ed eseguire il programma caricato.

[![Schema del modulo Clock](../../assets/clock/15-clock-schema.png "Schema del modulo Clock")](../../assets/clock/15-clock-schema.png)

*Schema del modulo Clock.*

## Note sul microcode

Il microcode dell'istruzione HLT del computer attiva l'omonimo segnale sul modulo di clock, interrompendo il funzionamento del programma. Per riprendere il funzionamento, È necessario superare la microistruzione corrente premendo il bottone step e poi è possibile riprendere la modalità a clock continuo premendo il pulsante Start/Stop.

## Link utili

- I [video](https://eater.net/8bit/clock) di Ben Eater che descrivono il funzionamento dei timer 555 e la costruzione del modulo di clock.

## TO DO

- Verificare se ho scritto correttamente le note sul microcode.
- notare che in questa pagina il nome "modulo di clock" e "clock" sono usati per descrivere... maiuscolo o minuscolo?
