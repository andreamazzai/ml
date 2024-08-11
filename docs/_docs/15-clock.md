---
title: "Clock"
permalink: /docs/clock/
excerpt: "Circuito di Clock del BEAM computer"
---
[![Circuito di Clock del BEAM computer](../../assets/clock/15-beam-clock.png "Circuito di Clock del BEAM computer"){:width="100%"}](../../assets/clock/15-beam-clock.png)

Il circuito di clock del computer BEAM riprende il modulo di clock progettato da Ben Eater ([link](https://eater.net/8bit/clock)) per il computer SAP, semplificandone leggermente l'utilizzo grazie a interruttori tattili anziché interruttori a scorrimento e aggiungendo qualche interazione con controlli esterni come quelli provenienti dal loader.

- Il primo 555 è utilizzato come classico generatore astabile con possibilità di selezionare la frequenza di funzionamento attraverso un potenziometro; un pulsante "Turbo" permette di bypassare la resistenza del potenziometro portando la frequenza di lavoro al valore massimo.
- Il secondo 555 permette di eseguire singole microistruzioni ("Step").
- Il terzo 555 permette di avviare e fermare il clock continuo utilizzando un comodo bottone "Start/Stop", anziché un interruttore come avveniva sul circuito di Ben.

Notare che una pressione dell'interruttore Step durante il normale funzionamento a clock continuo permette di fermare il computer alla microistruzione istruzione attuale e di poter proseguire in modalità "Step by Step". Premendo l'interruttore Start/Stop il computer riprende a lavorare in modalità clock continuo.

Gli interruttori Start/Stop e Step sono stati duplicati anche in una zona più facilmente raggiungible del computer, cioè sotto alla Control Logic; in tale sede è stato incluso anche un interruttore di Reset.

Notare anche gli ingressi ~{LDR-Active} e LDR-CLK.

Se ~{LDR-Active} è LO, allora disattivo tutti i clock del modulo perché Arduino gestirà LDR-CLK.

Presente che HALT, che permette di stoppare l'esecuzione del codice con l'istruzione BRK, Il cui microcode porta a uno questo segnale.

[![Schema del modulo Clock](../../assets/clock/15-clock-schema.png "Schema del modulo Clock")](../../assets/clock/15-clock-schema.png)

*Schema del modulo Clock.*

## Note sul microcode

Verificare se ci sono note da inserire in questa sezione in relazione al contenuto della pagina.

## Link utili

- I [video](https://eater.net/8bit/clock) di Ben Eater che descrivono il funzionamento dei timer 555 e la costruzione del modulo di clock.

## TO DO

- Verificare se sono necessarie note sul microcode ; potrebbe essere il caso di spiegare come la istruzione HLT possa fermare il computer
