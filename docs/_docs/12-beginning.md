---
title: "Per iniziare"
permalink: /docs/beginning/
excerpt: "Per iniziare"
---
[![Per iniziare](../../assets/ram/20-ram-beam.png "Per iniziare"){:width="100%"}](../../assets/ram/20-ram-beam.png)

**WORK IN PROGRESS - WORK IN PROGRESS**

**Cosa devo includere in questa pagina?**

- a
- b
- c

## Componenti

### TTL o CMOS?

- Descrivere la scelta di non usare HC

### LED

- i LED utilizzati includono la resistenza - giallo verde rosso; altri no.
- Le LED BAR Sono incluse negli schemi, mentre i singoli LED no. Nota: per motivi di spazio una LED BAR nello schema della CL non è stata inserita.
- fare LED BAR da 8 led per Control Logic

### Varie

- I comodissimi [Rocker Switch](../ram/#note) utilizzati nei moduli MAR e RAM.

## Approvvigionamento dei materiali

Le breadboard utilizzate sono le BB830 incluse anche nei kit di Ben Eater. La qualità dei contatti è decisamente superiore a quella delle breadboard economiche che si trovano online per pochi Euro e il costo ripaga ampiamente il non doversi preoccupare di falsi contatti e problematiche che avvengono casualmente.

Includere appunti su acquisto dei materiali, cavi utilizzati

## Alimentazione - WIP

Leggendo le esperienze degli altri utenti su reddito , sembra che quello dell'alimentazione sia un problema abbastanza diffuso .le FAQ altri documenti scritti da altri utenti danno dei suggerimenti e aiutano indirizzare le questioni di distribuzione dell'elettricità . per quanto mi riguarda , dopo essere partito con la distribuzione laterale come avevo fatto nel SAP-1 , ho aggiunto una ulteriore discesa dell'alimentazione al centro del computer aggiungendo una ulteriore fila di sezioni laterali ritagliate dalle breadboard.

Come si potrà notare nelle foto non ho utilizzato condensatoriCosì come non li avevo utilizzati nemmeno nella realizzazione del sapi, confidando nella bassa frequenza di utilizzo del computer.

Devo infine dire che nelle mie due realizzazioni ho notato che i chip TTL sembrano essere un po più malleabili , flessibili , meno rigidi rispetto ai requisiti indicati nel data sht, tanto che in alcuni casi , prima di migliorare la distribuzione dell'elettricità  alcuni chip funzionavano bene anche a 4,3 volt.

## Circuito di test

Anziché utilizzare jumper e led temporanei per settare e visualizzare input ed output sul bus e sui moduli in costruzione, ho costruito un semplice circuito di prova, che si è dimostrato particolarmente comodo per accelerare test ed eventuali fasi di troubleshooting.

[![Modulo di test in lettura dal bus](../../assets/varie/test-board.png "Modulo di test in lettura dal bus"){:width="100%"}](../../assets/varie/test-board.png)

*Modulo di test in lettura dal bus.*

Con il pulsante premuto, viene emesso un output corrispondente ai bit impostati sullo switch a bilanciere, mentre il pulsante rilasciato permette di visualizzare sulla LED bar i segnali presenti in input. I primi due led di sinistra rappresentano rispettivamente l'impostazione del circuito in output o in input.

[![Schema del modulo di test](../../assets/varie/test-schema.png "Schema del modulo di test"){:width="100%"}](../../assets/varie/test-schema.png)

*Schema del modulo di test.*

### Altro argomento

Lorem ipsum dolor sit amet, consectetur adipisci elit, sed do eiusmod tempor incidunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrum exercitationem ullamco laboriosam, nisi ut aliquid ex ea commodi consequatur. Duis aute irure reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint obcaecat cupiditat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Link utili

- I <a href="https://eater.net/8bit/clock" target="_blank">video</a> di Ben Eater che descrivono il funzionamento dei timer 555 e la costruzione del modulo di clock.
- <a href="https://todbot.com/blog/2010/01/02/momentary-button-as-onoff-toggle-using-555/" target="_blank">Un blog</a> che spiega come creare un circuito bistabile, qui utilizzato per creare la funzione "Start/Stop" con pulsante anziché con interruttore.

## Note

Invece dei più comuni dip-switch, ho utilizzato dei comodissimi Rocker Switch ("a bilanciere") come quelli in figura; si trovano facilmente presso i distributori di [materiale elettronico](https://us.rs-online.com/product/te-connectivity/5435640-5/70156004/). Notare che i pin originali sono piuttosto corti e non fissano correttamente lo switch alla breadboard, pertanto ho aggiunto uno zoccolo per circuiti integrati.

[![Rocker Switch](../../assets/ram/20-ram-rocker.png "Rocker Switch"){:width="33%"}](../../assets/ram/20-ram-rocker.png)

*Rocker Switch.*
