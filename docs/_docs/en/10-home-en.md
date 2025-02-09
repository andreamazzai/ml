---
title: "BEAM"
permalink: /en/
excerpt: "Il mio breadboard computer TTL a 8 bit basato sulla realizzazione di Ben Eater e sui miglioramenti di Tom Nisbet"
header:
#   image: /assets/home/beam.png
  overlay_image: /assets/home/beam.png
  overlay_filter: 0.5
---
## BEAM 8-bit computer

<small>[Il primo progetto: 6502](#il-primo-progetto-6502) - [Computer a 8 bit in logica TTL: SAP-1](#computer-a-8-bit-in-logica-ttl-sap) - [Evoluzione del computer a 8 bit: BEAM](#evoluzione-del-computer-a-8-bit-beam) - [BEAM?](#beam) - [Studio e progettazione](#studio-e-progettazione) - [Realizzazione](#realizzazione) - [Note](#note) - [Crediti e ringraziamenti](#crediti-e-ringraziamenti) - [Letteratura](#letteratura) - [Avviso sui Diritti di Proprietà Intellettuale](#avviso-sui-diritti-di-proprietà-intellettuale)</small>

<!-- TLDR: vai subito alla [documentazione del BEAM 8-bit computer](https://andreamazzai.github.io/beam/) su GitHub Pages -->

>NOTE: In some parts of the documentation, it is assumed that the reader has built or understands the SAP-1 Computer project by Ben Eater. Where possible, references are provided to facilitate consultation.

After several decades since completing my studies (a high school diploma in electronics obtained in 1989), in 2021, I decided to return to electronics, particularly digital electronics. I began exploring Arduino and Raspberry Pi but immediately encountered some difficulties: my programming and system administration skills were either underdeveloped or very rusty. I had no knowledge of C or other compiled languages, and my understanding of Linux was quite limited.

### The First Project: 6502

Toward the end of the year, I accidentally came across <a href="https://eater.net/" target="_blank">Ben Eater's blogs </a> and <a href="https://www.youtube.com/@BenEater/playlists" target="_blank">videos</a>. Remembering my first computer (a Commodore 64 from 1984) and the fact that I had done some assembly programming back then, I built Ben’s breadboard computer based on the 6502. I also started reading numerous blogs and forums on the subject, reviving some of my old knowledge.

### 8-Bit TTL Logic Computer: SAP

In the early months of 2022, after completing the basic modules of the 6502 computer, I followed the **SAP-1** (Simple As Possible) 8-bit TTL logic computer <a href="https://www.youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU" target="_blank">playlist</a> which I became even more passionate about. During the build, I learned a great deal — both thanks to Ben’s excellent presentation style and the significant effort I put into deepening my understanding through other websites and books.

[![My Implementation of the SAP-1 Computer.](../assets/home/sap.png "My Implementation of the SAP-1 Computer."){:width="66%"}](../assets/home/sap.png)

*My Implementation of the SAP-1 Computer.*

### Evolution of the 8-Bit Computer: BEAM

I completed the SAP-1 computer in September, but even earlier, I had started studying the improvements and expansions made by other followers of Ben, particularly regarding the possibility of automating the program loading process with a bootloader, as well as the need to increase the RAM beyond 16 bytes. During the summer of that year, I discovered the fantastic <a href="https://tomnisbet.github.io/nqsap/" target="_blank">NQSAP</a> project by **Tom Nisbet**: an 8-bit computer based on Ben Eater’s design but engineered to emulate the 6502 instructions. I quickly realized it would be my next project because it brought together many aspects I found very interesting:

1. Evolution of Ben Eater’s Computer, with which I had started to regain confidence in digital electronics.
2. Emulation of the processor from my first computer, complete with index registers, addressing modes, and Stack Pointer.  

3. In Ben Eater’s original design, the computer had only 16 bytes of RAM. This amount was enough to program a minimal Fibonacci sequence, but it was absolutely inadequate for creating more complex programs. Expanding the RAM, therefore, became essential.  

4. The need to learn at least the basics of C, which was used to create the [EEPROM programmer](docs/eeprom-programmer) and the Arduino-based [Loader](docs/loader).

### BEAM?

When searching for a name for my project, I identified "BEAM" as the ideal choice: I wanted to honor Ben Eater for bringing me back to this wonderful world and add something of my own, so **B**en **E**ater **A**ndrea **M**azzai ==> **BEAM** was born.

## Study and Design

In the first eight months of work on the BEAM project, I dedicated myself entirely to studying. The goal was to understand every detail of the NQSAP, which introduced numerous additions to the SAP-1 and implemented some truly ingenious ideas. This period of analysis was crucial for mastering the new concepts and innovations of the project.

The documentation for the NQSAP was incomplete, and I had also found a few errors, which I shared with Tom, who promptly corrected them. To deepen my understanding of the subject, after studying the NQSAP, I also analyzed the NQSAP-PCB evolution, to which Tom had made several improvements. In creating the BEAM, I aimed to take the best from both projects.

Among the aspects to mention, on which I spent a lot, **a **lot**, ***really a lot of time***:

- understanding the operation of the 74181 ALU, for which I created a test bench based on what I learned from David Courtney. [David Courtney](docs/alu/#link-utili);
- understanding binary arithmetic in two's complement;
- understanding overflow, closely related to the previous point;
- integration of the 74181 ALU and Instruction Register (IR) in "[hardwired](docs/alu/#relazione-diretta-hardwired-tra-instruction-register-e-alu)" mode to automatically generate the ALU input signals based on the instruction present in the IR;
- new memory module based on SRAM with common IO pins, unlike the 74189 used in the SAP-1 with dedicated input and output ports;
- the flag register—certainly the most complex part of the computer:
  - using the 74151 to decode the overflow state and manage the flag itself—an issue closely tied to the perfect understanding of binary arithmetic;
  - Using the 74151 to handle [conditional jump instructions in "hardwired" mode](docs/flags/#i-salti-condizionali-e-incondizionati), similar to how it was done for the ALU;

- understanding the Stack Pointer and writing microcode for the instructions that use it;
- development of the EEPROM programmer / microcode software based on Arduino: I studied Tom's code in depth, which I mostly understood:
  - schematization of the 6502 instructions and categorization;
  - understanding the very clever method developed by Tom to address more than 16 arithmetic instructions with only 5 signals dedicated to the ALU operation selection signals;
- understanding the "glitch" problem;
- development of the bootloader software.

[![Schema logico del computer BEAM, luglio 2023](assets/home/hand-drawn-logic.jpg "Schema logico del computer BEAM, luglio 2023"){:width="66%"}](assets/home/hand-drawn-logic.jpg)

*Schema logico del computer BEAM, luglio 2023.*

A luglio 2023 ho cercato di ricomporre tutto ciò che avevo appreso e ho provato trascriverlo su carta: avevo compreso molto bene il funzionamento di alcuni moduli, mentre su altri avevo ancora molti dubbi. Inoltre, non mi erano ancora chiare diverse interazioni tra i moduli, soprattutto nei segnali di controllo dei flag; rileggendo il materiale che avevo studiato e gli appunti che avevo trascritto su OneNote nei mesi precedenti, ho unito i puntini e realizzato lo schema logico del computer, iniziando a intravedere non più una serie di moduli a se' stanti, bensì un costrutto logico sensato:

[![Rappresentazione su carta di una possibile disposizione dei moduli del computer BEAM](assets/home/beam-paper-breadboards.png "Rappresentazione su carta di una possibile disposizione dei moduli del computer BEAM"){:width="66%"}](assets/home/beam-paper-breadboards.png)

*Rappresentazione su carta di una possibile disposizione dei moduli del computer BEAM.*

## Realizzazione

La fase costruttiva è iniziata a ottobre 2023 e si è conclusa nei primi giorni di maggio 2024. Il risultato è quello visibile in foto.

In queste pagine si trovano note, appunti e insegnamenti appresi ed applicati nella realizzazione del BEAM, analisi dei moduli dell'NQSAP e dell'NQSAP-PCB, differenze tra i sistemi, qualche confronto con il SAP-1 ed altro ancora.

[![BEAM Breadboard Computer](assets/home/beam.png "BEAM breadboard computer"){:width="66%"}](assets/home/beam.png)

Tra le cose apprese:

1. i flag (del 6502), il loro ruolo e le interazioni all'interno di un computer
2. il ruolo di una ALU e la rappresentazione dei numeri Signed
3. realizzazione di un programmatore di EEPROM molto veloce
4. basi del linguaggio C
5. basi di Arduino
6. utilizzo di Visual Studio Code
7. il linguaggio Markdown
8. utilizzo di GitHub e GitHub Pages
9. disegno degli schemi con KiCad

### Note

- Il range di frequenza del computer BEAM varia da circa un Hertz a circa 2 KHz. Non sono state testate frequenze superiori, poiché l'obiettivo principale non era tanto quello di ottenere prestazioni elevate, quanto piuttosto di acquisire conoscenze.
- Gli schemi KiCad non includono la gran parte dei LED singoli: sono indicati solamente i LED del modulo Clock e i LED che segnalano le modalità Program Mode / Run Mode nel modulo RAM / MAR. Sono invece rappresentate quasi tutte le LED Bar.

## Crediti e ringraziamenti

- Ben Eater per la sua grande capacità di creazione di <a href="https://www.youtube.com/@BenEater/playlists/" target="_blank">contenuti così interessanti</a> e in particolar modo per la playlist <a href="https://www.youtube.com/playlist?list=PLowKtXNTBypGqImE405J2565dvjafglHU" target="_blank">Building an 8-bit breadboard computer!</a>;
- Tom Nisbet per l'<a href="https://tomnisbet.github.io/nqsap/" target="_blank">NQSAP</a> e l'<a href="https://tomnisbet.github.io/nqsap-pcb/" target="_blank">NQSAP-PCB</a> espandendo e migliorando in maniera sostanziale il progetto di Ben Eater;
- il subreddit <a href="https://www.reddit.com/r/beneater/" target="_blank">r/beneater</a> - in particolar modo The8BitEnthusiast - grazie!
- Ken Shirrif per alcuni <a href="https://www.righto.com/" target="_blank">interessantissimi articoli</a> su Overflow, 6502 e 74181 (e molto altro!);
- l'esplosivo Dr. Brock LaMeres dell'università del Montana per l'eccellente playlist <a href="https://www.youtube.com/playlist?list=PL643xA3Ie_Et2uM4xu1yFk-A5ZQQ8gQ5e" target="_blank">Intro to Logic Circuits</a> su YouTube; un <a href="https://www.montana.edu/blameres/book_content_vhdl.html" target="_blank">indice più leggibile</a> dello stesso materiale è disponibile presso il sito ufficiale dell'Università del Montana;
- il sito <a href="http://6502.org" target="_blank">6502.org</a> e il suo forum: documentazione dettagliata senza fronzoli e utenti davvero esperti;
- Dieter Müller, le cui pagine sul <a href="http://6502.org/users/dieter/v_flag/v_0.htmmi " target="_blank">74LS151</a> mi hanno spinto ad approfondire in dettaglio l'argomento Overflow e col quale ho sviluppato una amichevole conoscenza telematica;
- Norbert Landsteiner per il <a href="https://www.masswerk.at/6502/6502_instruction_set.html" target="_blank">6502 Instruction Set</a>, il <a href="https://www.masswerk.at/6502/assembler.html" target="_blank">6502 Assembler</a> e il <a href="https://www.masswerk.at/6502" target="_blank">Virtual 6502</a>, strumenti eccezionali che ho utilizzato ampiamente per la creazione e il debug del microcode. Norbert è stato anche estremamente reattivo nel correggere un bug che gli avevo segnalato.

Ringraziamenti speciali: Tom Nisbet e Dieter Müller, che anche in fase di redazione della documentazione non hanno esitato a darmi qualche chiarimento.

## Letteratura

- <a href="https://archive.org/details/Programming_the_6502_OCR" target="_blank">Programming the 6502</a> di Rodnay Zaks.
- <a href="https://archive.org/details/367026792DigitalComputerElectronicsAlbertPaulMalvinoAndJeraldABrownPdf1/" target="_blank">Digital Computer Electronics</a> di Albert Paul Malvino e Jerald A. Brown.

## Avviso sui Diritti di Proprietà Intellettuale

Questo progetto potrebbe includere link, documenti, o immagini di proprietà di terzi. Mi impegno a rispettare pienamente i diritti di proprietà intellettuale di tutti i proprietari e non intendo violare alcun diritto d'autore.

Se si ritiene che uno o più dei materiali presenti in questa documentazione violi i propri diritti di copyright, prego di segnalarmelo. Provvederò prontamente alla verifica e, in caso di violazione, alla rimozione immediata del materiale in questione.

D'altro canto, tutto il materiale originale prodotto in questo progetto è rilasciato sotto la licenza MIT. Questo significa che è liberamente utilizzabile, modificabile e distribuibile da altri, a condizione che venga mantenuta l'indicazione di copyright originale e questa stessa licenza sia inclusa in eventuali copie o versioni modificate del progetto.
