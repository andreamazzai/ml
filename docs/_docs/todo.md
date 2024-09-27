---
title: "D, X e Y"
permalink: /docs/todo/
excerpt: "Registri indice del computer BEAM"
---

### OVUNQUE

- Rivedere Flag vs flag, anche in relazione al Carry vs carry

### HOME

- Mettere i link ai miei post su Reddit
- cosa non è stato emulato? IRQ, BCD
- Aritmetica binaria e tutti gli altri argomenti... se è il nome di qualcosa, va in maiuscolo, altrimenti no - ad esempio alla sezione **numeri signed** della pagina dedicata all'aritmetica binaria ==> In questo caso forse sarebbe meglio averlo in minuscolo
- Registri indice maiscuolo, minuscolo etc... ???
- Forma e coerenza su come chiamare i registri - ad esempio Stack per il registro e stack per genericamente "lo stack"
- Review della parola "segnali" per distinguere bene i segnali di controllo dai segnali dati - e decidere se le abbreviazioni dei segnali debbano essere evidenziate o no per coerenza

## CLOCK

- Verificare la forma - "modulo di clock" e "clock" sono usati per descrivere... maiuscolo o minuscolo?
- Verificare possibilità di applicare teorema di De Morgan alla fase finale del modulo.

## RAM

- mettere dei link
- dove parlo di "Ripple Mode Carry” e “Carry Look Ahead" mettere il link alla sezione (ancora da scrivere) in Aritmetica Binaria (il testo è "Per combinare due chip a 4 bit è stato necessario, nonché molto utile, comprendere la differenza fra "Ripple Mode Carry" e "Carry Look Ahead": il datasheet del contatore 74LS161 riportava infatti questi due diversi esempi di collegamento di chip in cascata tra di loro. Un approfondimento sul tema si trova in una apposita sezione della pagina dedicata all'[Aritmetica Binaria](../math).")

## PC

- Una volta completata la sezione apposita su RCO / CLA nella pagina dell'aritmetica binaria, verificare se opportuno linkarla al DS del '161.

## CL

- finire la parte di codice verso la fine della pagina con gli esempi CPX

- "Infatti il codice di Tom mostra che è possibile automatizzare la generazione del micro code per molte operazioni grazie al fatto che le app opportunamente posizionate all’interno della tabella , cosa che ionon sono riuscito a fare in quanto all’inizio del task non mi era chiaro quale sarebbe stato il beneficio . inoltre la mia conoscenza del linguaggio c è limitata e non sono riuscito a comprendere chiaramente cosa facesse il codice di Tom."

- Al completamento della pagina del Loader:
  - Reset asincrono del computer
  - Disattivazione clock e disattivazione delle EEPROM della Control Logic
  - Iniezione del clock del Loader nel computer
  - (Re-)Start del clock di sistema dopo il caricamento

  ## ALU

- /WE ↘↗
- https://bread80.com/2019/09/02/adding-adc-sbc-inc-dec-operations-to-ben-eaters-alu/#easy-footnote-4-43 da leggere per capire se buono
- subito dopo il capitolo "Il registro H" capire "(da fare: in questo caso, ma anche nel caso delle istruzioni di shift / rotazione e forse anche CPX e CPY, verificare se non potessi usare D invece di H)"
- in Questa zona <<< Nel datasheet venivano menzionati anche il Carry Look-Ahead e il Ripple-Carry, approfonditi nella sezione dedicata all'[Aritmetica Binaria](../math/#). >>> sistemare il link alla sezione che devo ancora scrivere - 15 08 2024
- controllare "Lo spreadsheet Excel citato nel paragrafo precedente è scaricabile qui; le tabelle appena menzionate sono presenti nel foglio “6502 Inst. Set”.
- rivedere "L'ALU dell'NQSAP per dare senso al fatto che ho studiato NQSAP ma parlo della realizzazione del BEAM
- Parlare del bench di test sulla base di quanto appreso da David Courtney.
- link al mio video dell'ALU

## STACK

- controllare se il mio SP funziona correttamente; vedo che Tom ha usato dei cablaggi diversi per i pin 7 e 10   20-08-2024

## PROGRAMMATORE

- Nei link mettere il link a Tommy PROM, spiegare  CHE IL MIO PROGRAMMATORE DI EPPROM  di è basata un po sul DIWSEGNO  originale di ben , UN PO' SUL TOMMYPROM
- Verificare anche se è Il programmatore dietro di ben Tom che utilizza lo stesso codice di programmazione del Tommyprom; in realtà mi sembra di ricordare che sia una versione tagliata del tommyprom
