---
title: "Arduino Loader"
permalink: /docs/loader/
excerpt: "Loader del computer BEAM"
---
<small>[Schema](#schema) - [Link utili](#link-utili)</small>

[![Loader del computer BEAM](../../assets/loader/80-beam-loader.png "Loader del computer BEAM"){:width="100%"}](../../assets/loader/80-beam-loader.png)

**WORK IN PROGRESS**

Dopo aver completato il [computer SAP a 8-bit](../#computer-a-8-bit-in-logica-ttl-sap), cercavo un modo per automatizzare il caricamento dei programmi in memoria, poiché farlo manualmente a ogni riaccensione utilizzando i dip-switch era piuttosto noioso. Avevo anche intenzione di incorniciarlo e appenderlo come un quadro per mostrarlo ai visitatori. Prendendo confidenza con Arduino, avevo realizzato che avrei potuto collegare le sue uscite a MAR, RAM e al pulsante di Write per gestire automaticamente il caricamento, simulando esattamente le operazioni che si sarebbero dovute eseguire manualmente.

Le uscite di Arduino erano direttamente connesse ai dip-switch e al pulsante di scrittura; il Loader era anche in grado di impostare il computer in Program-Mode, resettarlo, avviare e fermare il clock.

[![Loader del computer BEAM](../../assets/loader/80-beam-loader.png "Loader del computer BEAM"){:width="100%"}](../../assets/loader/80-beam-loader.png)

![Alt text](SAP.png)

Anche nel BEAM il caricamento di un programma nella memoria RAM del computer BEAM può essere effettuato manualmente o automaticamente utilizzando il modulo Loader,



impostando l'interruttore PROG nel modulo MAR Lorem ipsum dolor sit amet, consectetur adipisci elit, sed do eiusmod tempor incidunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrum exercitationem ullamco laboriosam, nisi ut aliquid ex ea commodi consequatur. Duis aute irure reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint obcaecat cupiditat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Schema

[![Schema del Loader del computer BEAM](../../assets/loader/80-loader-schema.png "Schema del Loader del computer BEAM")](../../assets/loader/80-loader-schema.png)

*Schema del Loader del computer BEAM.*

## Link utili

- Il <a href="https://tomnisbet.github.io/nqsap/docs/loader/" target="_blank">Loader dell'NQSAP</a> di Tom Nisbet.
- Il <a href="https://tomnisbet.github.io/nqsap-pcb/docs/loader/" target="_blank">Loader dell'NQSAP-PCB</a> di Tom Nisbet.

## TO DO

- TBD

