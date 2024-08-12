---
title: "BEAM computer"
permalink: /
excerpt: "Il mio breadboard computer TTL a 8 bit basato sulla realizzazione di Ben Eater e sui miglioramenti di Tom Nisbet"
header:
#   image: /assets/beam.jpg
  overlay_image: /assets/beam.png
  overlay_filter: 0.5
---
## BEAM 8-bit computer

<!-- TLDR: vai subito alla [documentazione del BEAM 8-bit computer](https://andreamazzai.github.io/beam/) su GitHub Pages -->

>NOTA: in alcuni punti della documentazione si presuppone che il lettore abbia realizzato o comprenda il progetto del SAP Computer di Ben Eater. Ove possibile, vengono fornite delle referenze per facilitare la consultazione.

Dopo diversi decenni dalla fine degli studi (diploma di scuola superiore ad indirizzo elettronico conseguito nel 1989), nel 2021 ho deciso di rimettere mano all'elettronica, in particolar modo a quella digitale. Ho iniziato ad interessarmi ad Arduino e Raspberry Pi, riscontrando subito una certa difficoltà: le mie basi di programmazione e di amministrazione di sistemi erano da una parte ben poco sviluppate, dall'altra molto arrugginite; non avevo alcuna conoscenza di C o di altri linguaggi compilati e la mia conoscenza di Linux era molto modesta.

### Il primo progetto: 6502

Verso la fine dell'anno ho scoperto casualmente il blog di <a href = "https://eater.net/" target = "_blank">Ben Eater</a> e i suoi <a href = "https://www.youtube.com/@BenEater/playlists" target = "_blank">video</a> e, memore del mio primo computer (Commodore 64, 1984) e del fatto che all'epoca avevo fatto un po' di programmazione in assembly, ho costruito il breadboard computer di Ben basato su 6502 e ho iniziato a leggere parecchi blog e forum sul tema, riesumando un po' di vecchie conoscenze.

### Computer a 8 bit in logica TTL: SAP

Nei primi mesi del 2022, dopo aver completato i primi moduli base del computer 6502, ho seguito la <a href = "https://www.youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU" target = "_blank">playlist</a> del computer **SAP** (Simple As Possible) in logica TTL a 8-bit, al quale mi sono appassionato ancora di più. Durante la costruzione ho appreso moltissimo - sia per l'eccellente stile di esposizione di Ben, sia per il grande impegno che ho profuso nell'approfondimento della materia su altri siti e libri.

### Evoluzione del computer a 8 bit: BEAM

Ho completato il SAP computer a settembre, ma già in precedenza avevo cominciato a studiare i miglioramenti e le espansioni fatte da altri follower di Ben, soprattutto in merito alla possibilità di automatizzare l'inserimento del programma da eseguire con un bootloader, nonché alla necessità di incrementare la RAM oltre i 16 byte; durante l'estate di quell'anno ho scoperto il fantastico progetto  <a href = "https://tomnisbet.github.io/nqsap/" target = "_blank">NQSAP</a> di **Tom Nisbet**: un computer a 8-bit basato su quello di Ben Eater, ma ingegnerizzato per emulare le istruzioni del 6502. Ho realizzato velocemente che sarebbe stato il mio successivo progetto, perché metteva insieme molti aspetti che trovavo di grande interesse:

1. evoluzione del computer di Ben Eater, col quale avevo iniziato a riacquistare confidenza con l'elettronica digitale
2. emulazione del processore del mio primo computer, completa di registri indice, modalità di indirizzamento e Stack Pointer
3. espansione del progetto originale: il computer di Ben Eater aveva solo 16 byte di RAM - pur sufficienti per programmare una sequenza di Fibonacci minimale, ma assolutamente insufficienti per scrivere programmi minimamente più complessi
4. possibilità / necessità di imparare almeno le basi del C, utilizzato per la programmazione delle EEPROM e per la creazione del bootloader

### BEAM?

Cercando un nome per il mio progetto, ho identificato in "BEAM" quello ideale: desideravo dare merito a Ben Eater per avermi fatto riavvicinare a questo meraviglioso mondo e aggiungere qualcosa di mio, perciò è nato il **B**en **E**ater **A**ndrea **M**azzai ==> **BEAM**.

### Studio e progettazione

I primi 8 mesi di lavoro sul progetto BEAM sono stati interamente devoluti allo studio: volevo capire ogni dettaglio dell'NQSAP, che presentava moltissime aggiunte al SAP e diverse idee davvero "clever"!

Tra gli aspetti da citare e sui quali ho speso molto, **molto**, ***davvero molto*** tempo:

- comprensione del funzionamento dell'ALU 74181, per la quale ho realizzato un bench di test sulla base di quanto appreso da David Courtney
- comprensione dell'aritmetica binaria in complemento di 2
- comprensione dell'overflow, strettamente legato al punto precedente
- integrazione dell'ALU 74181 e dell'Instruction Register (IR) in modalità "hardware wired" per poter generare "automaticamente" i segnali di ingresso dell'ALU a seconda dell'istruzione presente nell'IR
- nuovo modulo di memoria basato su SRAM con pin IO comuni, a differenza dei 74189 utilizzati nel SAP con porte di Input e di Output dedicate
- il registro dei flag - sicuramente il più complesso del computer:
  - utilizzo del 74151 per decodificare lo stato di overflow e la gestione del flag stesso - argomento strettamente legato alla perfetta comprensione dell'aritmetica binaria
  - utilizzo del 74151 per poter gestire le istruzioni di relative branch in modalità "hardware wired", similarmente a quanto fatto per la ALU
- comprensione dello stack pointer e scrittura del microcode per le istruzioni che ne fanno uso
- sviluppo del software del programmatore di EEPROM / microcode basato su Arduino: ho studiato a fondo il codice di Tom, che ho compreso in buona parte:
  - schematizzazione delle istruzioni del 6502 e suddivisione in categorie
  - comprensione del metodo molto smart sviluppato da Tom per poter indirizzare più di 16 istruzioni aritmetiche avendo a disposizione solo 5 segnali da dedicare ai segnali di selezione dell'operazione dell'ALU
- comprensione del problema del "glitch"
- sviluppo del software del bootloader
- disegno degli schemi con Kicad

[![Schema logico luglio 2023](assets/hand-drawn-logic.jpg "Schema logico luglio 2023"){:width="66%"}](assets/hand-drawn-logic.jpg)

A luglio 2023 ho cercato di ricomporre tutto ciò che avevo appreso e ho provato trascriverlo su carta: avevo compreso molto bene il funzionamento di alcuni moduli, mentre su altri avevo ancora molti dubbi. Inoltre, non mi erano ancora chiare diverse interazioni tra i moduli, soprattutto nei segnali di controllo dei flag; rileggendo il materiale che avevo studiato e gli appunti che avevo trascritto su OneNote nei mesi precedenti, ho unito i puntini e realizzato lo schema logico del computer, iniziando a intravedere non più una serie di moduli a se' stanti, bensì un costrutto logico sensato:

[![Rappresentazione su carta di una possibile disposizione dei moduli](assets/beam-paper-breadboards.png "Rappresentazione su carta di una possibile disposizione dei moduli"){:width="66%"}](assets/beam-paper-breadboards.png)

*Rappresentazione su carta di una possibile disposizione dei moduli.*

## Realizzazione

La fase costruttiva è iniziata a ottobre 2023 e si è conclusa nei primi giorni di maggio 2024. Il risultato è quello visibile in foto.

In queste pagine si trova la documentazione dettagliata dei moduli, corredata da una serie di note e appunti.

[![BEAM Breadboard Computer](assets/beam.png "BEAM breadboard computer"){:width="66%"}](assets/beam.png)

Benefici:

1. aver compreso bene i flag del 6502 e il loro ruolo e le interazioni all'interno di un computer
2. aver compreso bene il ruolo di una ALU e la numerazione in complemento di 2
3. realizzazione di un programmatore di EEPROM molto veloce
4. iniziato a studiare il linguaggio C

Poco dopo xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

### Note

- Gli schemi KiCad non includono la gran parte dei LED: sono indicati solamente i LED del modulo Clock e i LED che segnalano le modalità Program Mode / Run Mode nel modulo RAM / MAR. Sono invece rappresentate tutte le LED Bar.

## Crediti e ringraziamenti

- Ben Eater per la sua grande capacità di creazione di [contenuti così interessanti](https://www.youtube.com/@BenEater/playlists/) e in particolar modo per la playlist [Building an 8-bit breadboard computer!](https://www.youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU);
- Tom Nisbet che ha realizzato l'[NQSAP](https://https://tomnisbet.github.io/nqsap/) espandendo e migliorando in maniera sostanziale il progetto di Ben Eater;
- il subreddit [r/beneater](https://www.reddit.com/r/beneater/) - in particolar modo [The8BitEnthusiast](https://www.reddit.com/user/The8BitEnthusiast/) e [Tom Nisbet](https://www.reddit.com/user/nib85/) - grazie!
- Ken Shirrif per alcuni [interessantissimi articoli](https://www.righto.com/) su Overflow, 6502 e 74181 (e molto altro!);
- il sito [6502.org](http://6502.org) e il suo forum: documentazione dettagliata senza fronzoli e utenti davvero esperti.

## Altri link

- Un [compendio della Micro Logic](https://www.atarimania.com/documents/6502%20(65xx)%20Microprocessor%20Instant%20Reference%20Card.pdf) incredibilmente utile, che in sole due pagine include opcode, modalità di indirizzamento, flag e istruzioni che li modificano, funzionamento delle istruzioni di scorrimento e molto altro. Insostituibile.
- Un valido riferimento per l'analisi della relazione tra Control Logic (CL) ed IR è stata la pagina [6502 Instruction Set](https://www.masswerk.at/6502/6502_instruction_set.html) di Norbert Landsteiner, che invito a consultare anche per il [6502 Assembler](https://www.masswerk.at/6502/assembler.html) e il [Virtual 6502](https://www.masswerk.at/6502/) che ho utilizzato in fase di debug del microcode.

## TO DO

- mettere i link ai miei post su Reddit
- qualche idea è stata presa anche da NQSAP-pCB, ad esempio i 74LS377
- quel professore dell'università del Montana
- Libro di Malvino
- Includere appunti su acquisto dei materiali, cavi utilizzati
- problemi di alimentazione
- lkink al video dell'ALU
- Controllare dove ho scritto computer BEAM oppure BEAM computer
- non ho utilizzato HC perché... bla bla bla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla blabla bla
- sistemare il paragrafo BENEFICI