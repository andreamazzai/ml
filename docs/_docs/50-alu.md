---
title: "ALU"
permalink: /docs/alu/
excerpt: "Costruzione dell'Unità Aritmetica e Logica del BEAM computer"
---
[![Unità Aritmetica e Logica del BEAM](../../assets/alu/50-alu-beam-small.png "Unità Aritmetica e Logica del BEAM"){:width="100%"}](../../assets/alu/50-alu-beam.png)

L'Unità Aritmetica e Logica (ALU) del SAP computer di Ben Eater era limitata a addizioni e sotttrazioni. L'NQSAP di Tom Nisbet aggiungeva operazioni logiche e di rotazione (shift) e avevo iniziato a studiarla in dettaglio.

Sviluppata attorno ai chip [74LS181](https://www.ti.com/lit/ds/symlink/sn54ls181.pdf), l'ALU dell'NQSAP presentava delle caratteristiche molto interessanti, ma di comprensione particolarmente difficile: sarà uno dei moduli sui quali spenderò più tempo, ma che alla fine darà maggiori soddisfazioni per il risultato.

Il '181 è un'ALU a 4 bit sviluppata negli anni '70 che può eseguire 16 operazioni aritmetiche e 16 funzioni logiche. E' possibile concatenare più chip per elaborare word di dimensioni maggiori.

Tra le caratteristiche che spiccavano nello schema dell'ALU dell'NQSAP notavo soprattutto un discreto quantitativo di chip - tra i quali gli Shift Register 74LS194 - e un modo particolare per indirizzare i '181, che erano "strettamente legati" all'istruzione presente nell'Instruction Register della [Control Logic](../control). Il legame con la Control Logic è stato tra i più complessi da analizzare e comprendere, ma quello con il modulo dei Flag è altrettanto importante e non meno complesso: ad ogni operazione dell'ALU (e non solo) corrisponde infatti un'azione sul registro dei Flag.

[![Schema logico dell'ALU di Tom Nisbet](../../assets/alu/50-alu-nqsap.png "Schema logico dell'ALU di Tom Nisbet"){:width="100%"}](../../assets/alu/50-alu-nqsap.png)

*Schema logico dell'ALU di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Il modulo ALU è sommariamente composto da due registri di input H e B e da una coppia di '181 interconnessi, che permettono di gestire una word di 8 bit.

- Il registro H è in realtà uno Shift Register in grado di comportarsi come un normale registro a 8 bit oppure di *shiftare* sia a destra sia a sinistra il valore presente in ingresso (istruzioni di rotazione).
- Il registro B è un normale registro a 8 bit. Il chip utilizzato non include un ingresso Enable, che è dunque stato realizzato in maniera artificiale mettendo una NOR su /Clock e /WB ("Write B"); in questo modo il registro si attiva solo in corrispondenza di /WB (che è attivo LO) e del falling edge del clock negato, equivalente al rising edge del clock non negato, che è il momento in cui si caricano i registri (riferimento: video di Ben Eater [8-bit CPU control logic: Part 2](https://www.youtube.com/watch?v=X7rCxs1ppyY)).
- Tre transceiver '245 permettono di poter leggere i valori contenuti in H, B ed L (L è l'output dell'A**L**U).

Come detto nell'introduzione, il computer BEAM, al pari dell'NQSAP, include il set di istruzioni completo del 6502, comprese quelle logiche e aritmetiche. Ricordavo discretamente le principali operazioni del 6502 e sapevo *abbastanza* bene quale dovesse essere il risultato di quello che stavo facendo, ma in quel momento non avevo ancora idea di come fosse possibile ottenerlo.

Avevo intanto deciso di comprendere le operazioni messe a disposizione del '181 e se vi fosse una logica nella loro disposizione, una sorta di raggruppamento.

[![Funzioni logiche e operazioni aritmetiche del 74LS181](../../assets/alu/50-alu-operations.png "Funzioni logiche e operazioni aritmetiche del 74LS181"){:width="100%"}](../../assets/alu/50-alu-operations.png)

*Funzioni logiche e operazioni aritmetiche del 74LS181.*

Il datasheet del '181 era abbastanza criptico e dunque ho avevo fatto ricorso anche alle molte risorse disponibili in rete riportate a fondo pagina. Dal datasheet si comprende che vi sono 4 segnali S0, S1, S2 ed S3 per la selezione della funzione / operazione e un segnale di controllo della modalità M (M = HI per le funzioni logiche; M = LO per le operazioni aritmetiche); A e B sono gli input dei dati. Nel datasheet venivano menzionati anche il Carry Look-Ahead e il Ripple-Carry, che approfondirò in seguito nella sezione dedicata all'Aritmetica Binaria.

Inizialmente avevo trascritto la tabella delle funzioni / operazioni in un foglio Excel per poter lavorare più agevolmente:

![Funzioni logiche e operazioni aritmetiche del 74LS181](../../assets/alu/50-alu-operations-xls.png)

*Funzioni logiche e operazioni aritmetiche del 74LS181 - su Excel.*

Avevo evidenziato le operazioni ripetute più volte, non trovando però alcun raggruppamento o filo conduttore tra righe e colonne. Cercavo  di capire quale fosse il senso di quella disposizione così apparentemente disordinata, ma non l'avevo trovato. Illuminante è stato l'articolo di [Ken Shirriff](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) citato in calce.

Uno dei ricordi vividi è quello della comprensione di cosa accomunava le due colonne delle operazioni aritmetiche (eseguite in corrispondenza della modalità M = LO):

- senza Carry ("Cn = HI  --> No Carry")
- con Carry   ("Cn = LO  --> With Carry")

Per quanto bizzarre possano apparire alcune delle operazioni disponibili, il filo conduttore tra le due colonne è che l'output del '181 *è sempre lo stesso*, con l'unica differenza data dalla assenza o presenza del Carry in ingresso. Si noti infatti che, in ogni riga, l'output ha sempre una differenza pari ad 1, che corrisponde al valore di un Carry; ad esempio, nella decima riga troviamo una semplice operazione di somma: la differenza tra quanto computato nelle due colonne è 1 (**A più B** in caso di nessun Carry in ingresso e **A più B più 1** in caso di Carry in ingresso).

Lo stesso ragionamento è valido per in tutte le altre operazioni aritmetiche disponibili. Prendiamo come altro esempio la prima riga: in assenza di Carry l'ALU restituisce quanto presente agli ingressi A, mentre in presenza di Carry restituisce quanto presente in A + 1 (questa operazione sarà sfruttata per creare l'istruzione INC del computer "iniettando" un Carry artificiale; similarmente, l'istruzione DEC è costruita intorno all'ultima funzione della tabella, ma in questo caso senza essere "dopata" da un Carry artificiale).

A questo punto è anche opportuno segnalare che il '181 mette a disposizione due modalità di utilizzo: una con la logica attiva bassa ("Active-Low data") e una con la logica attiva alta ("Active-High data") che è quella utilizzata nell'NQSAP; quest'ultima, per complicare un po' le cose, si attende in ingresso un *Carry In negato*, nel senso che un segnale Cn (Carry In) LO viene interpretato come Carry attivo, mentre un segnale Cn HI viene interpretato come Carry non presente. Allo stesso modo, anche il *Carry Out* out è negato: Cn+4 è infatti HI per indicare che non c'è Carry in uscita, mentre è LO per indicare che è presente un Carry.

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX Vedere se qui ha senso prendere quel discorso del 6502 citato da Ken XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

Ritornando alla tabella delle funzioni / operazioni, ritenevo che solo il sottoinsieme visibile di seguito fosse utile per lo scopo prefissato, che era quello di poter emulare le istruzioni del 6502:

![Operazioni logiche e aritmetiche utili del 74LS181](../../assets/alu/50-alu-operations-xls-subset.png)

*Operazioni logiche e aritmetiche utili del 74LS181.*

Successivamente capirò che le istruzioni necessarie erano in realtà ancora meno di quelle che ipotizzavo.

Un altro degli aspetti di più difficile comprensione, come anticipato in precedenza, è stata l'associazione diretta tra istruzione correntemente contenuta nell'Instruction Register e funzione logica / operazione matematica eseguita dal '181.

Provando a sintetizzare quando disegnato nell'NQSAP, avevo costruito questa tabella:

| Cn | M  | S3 | S2 | S1 | S0 | Operazione  | S3/S0 Hex |
|  - | -  |  - |  - |  - |  - |          -  |   -       |
| 0  | 0  | 0  | 0  | 0  | 1  | A plus 1    |  00 + C*  |
| 0  | 0  | 0  | 0  | 1  | 1  | Tutti 0     |  03 + C*  |
| 0  | 0  | 0  | 1  | 1  | 0  | A minus B   |  06 + C*  |
| 0  | 0  | 0  | 1  | 1  | 1  | CMP         |  07**     |
| 1  | 0  | 0  | 0  | 1  | 1  | Tutti 1     |  03       |
| 1  | 0  | 1  | 0  | 0  | 1  | A plus B    |  09       |
| 1  | 0  | 1  | 1  | 0  | 0  | A plus A    |  0C***    |
| 1  | 0  | 1  | 1  | 1  | 1  | A minus 1   |  0F       |
| x  | 1  | 0  | 0  | 0  | 0  | Not A       |  10****   |
| x  | 1  | 0  | 1  | 1  | 0  | A XOR B     |  16****   |
| x  | 1  | 1  | 0  | 1  | 0  | B           |  1A****   |
| x  | 1  | 1  | 0  | 1  | 1  | A AND B     |  1B****   |
| x  | 1  | 1  | 1  | 1  | 0  | A OR B      |  1E****   |

*Sintesi operazioni dell'ALU dell'NQSAP.*

Per fare un esempio, si stava in pratica dicendo che per fare una somma ("A plus B", vedi sesta riga) era necessario avere:

- Cn = 1 (che, ricordiamo, è gestito con stato logico invertito, dunque in questo caso l'ALU considera il Carry non presente)
- M = **0**
- S3/S2/S1/S0 = **1001**

cioè:

>| Cn | M  | S3 | S2 | S1 | S0 | Operazione | S3/S0 Hex |
>|  - | -  |  - |  - |  - |  - |          - |   -       |
>| 1  | **0**  | **1**  | **0**  | **0**  | **1**  | A plus B   |  09       |

In pratica, poiché gli ingressi M ed S3-S0 sono direttamente connessi all'[Instruction Register](../control), l'istruzione di somma dovrà essere coficata presentando **01001** sui 5 bit comuni tra Instruction Register e ALU, nonché su 5 dei pin di ingresso delle EEPROM.

Attivando questa istruzione, il risultato esposto in output dall'ALU sarebbe stato esattamente A + B, proprio come indicato nella decima riga / colonna Cn = HI (Carry non presente) della tabella "Funzioni logiche e operazioni aritmetiche del 74LS181." estratta dal datasheet; se avessimo invece avuto un Carry in ingresso, il risultato esposto sarebbe stato A + B + 1, proprio come indicato nella decima riga / colonna Cn = LO. La somma (almeno in teoria) funzionava e iniziavo anche a far luce sul legame tra le due colonne Cn = LO / Cn = HI: il risultato in output era sempre lo stesso e variava solo in conseguenza del fatto che in ingresso ci fosse un Carry o meno.

Legenda tabella *Sintesi operazioni dell'ALU dell'NQSAP*:

- \* Avevo evidenziato queste righe per ricordare che su queste tre istruzioni si deve "iniettare" un Carry artificale (che è invertito, dunque il segnale doveva essere LO)
- \*\* = Le operazioni di salto relativo del 6502 dipendono dallo stato dei flag N, V, Z e C. Le istruzioni di comparazione (CMP) hanno effetto sui flag N, Z e C, che vengono computati effettuando una sottrazione (SBC) fittizia tra due valori, che nella logica del 6502 corrisponde a scartare il risultato e tenere in considerazione solo i flag risultanti dalla sottrazione fittizia. Si evince che per eseguire le comparazioni bisogna dunque effettuare delle sottrazioni scartando il risultato, tuttavia le operazioni di sottrazione del '181 sono già utilizzate per eseguire le sottrazioni vere e proprie (A Minus B), codificate su M/S3-S0 = 00110: come è possibile eseguire istruzioni di sottrazione utilizzando una istruzione con codifica diversa? Ne parleremo più diffusamente in seguito.

+++++++++++++++++++++++++ Vedere se ne parlerò sempre in questa stessa pagina o nella pagina dei FlagVedere se ne parlerò sempre in questa stessa pagina o nella pagina dei Flag ++++++++++++++++++++++++++

FARE UN ESEMPIO di SOTTRAZIONE E CALCOLO DEL FLAG, Oppure spiegare che verrà spiegata più in seguito quando si parla dei Flag

Ad esempio, BMI (**B**ranch on **MI**nus) viene eseguito solo in corrispondenza del flag N, che indica che il numero *con segno* è **N**egativo, dunque compreso tra -128 e -1, cioè tra 0x80 e 0xFF. I Flag sono calcolati facendo una sottrazione fittizia tra due valori: il risultato della sottrazione viene scartato e si prendono in considerazione solo i flag risultanti dall'operazione. I Flag sono dunque utilizzati per eseguire le operazioni di confronto e sfruttano l'operazione di sottrazione, che è però già utilizzata per eseguire l'operazione A Minus B (terza riga della tabella *Sintesi operazioni dell'ALU dell'NQSAP* dove S3/S2/S1/S0 = 0110 = 0x06 e il microcodice per l'operazione di sottrazione dovrà presentare **00110** sui 5 bit comuni tra Instruction Register e ALU).

- \*\*\* L'operazione A+A veniva usata nell'NQSAP per fare lo shift verso sinistra dei bit; vista la presenza dello Shift Register H, ho preferito riversare su di esso tutte le operazioni di rotazione (a destra e a sinistra, sia con Carry sia senza Carry).
- \*\*\*\*Il Carry è ininfluente in quanto questa è una funzione logica e non una operazione aritmetica.

### Istruzioni di comparazione

- Tutti i segnali che pilotano i '181 derivano direttamente dall'Instruction Register (IR), eccetto il Carry In. Si può dire che l'ALU è *hardwired* all'IR e che pertanto il microcode del computer dovrà essere scritto in modo tale che le sue istruzioni rispecchino i segnali in ingresso del '181: ad esempio, osservando la tabella *Funzioni logiche e operazioni aritmetiche del 74LS181*, l'istruzione di somma del computer dovrà avere i bit comuni tra IR ed ALU codificati come 01001.

- Il segnale S0 è in realtà "parzialmente indiretto" perché transita attraverso una NOR pilotata da una ROM che viene attivata solo in corrispondenza delle istruzioni CMP, così che la codifica 0011**1**, arbitrariamente designata per indicare le istruzioni di comparazione, venga presentata ai '181 come 0011**0**, che è il codice per l'istruzione di sottrazione (Subtract Mode).

In altre parole, il microcode delle istruzioni di confronto (che nella mnemonica del 6502 può essere CMP, CPX o CPY) dovrà attivare un segnale LF in uscita su una delle EEPROM: questo segnale attiverà la porta NOR per trasmettere ai '181 il codice 0011**0** della sottrazione anziché 0011**1**, che corrisponde ad A AND NOT B e che simulare le struzioni del 6502 non è necessario e rimane dunque inutilizzato. LF è l'abbreviazione di A**L**U **F**orce.

Detto in altre parole ancora:

- le istruzioni di comparazione (CMP) del 6502 simulano una sottrazione e tengono in considerazione solo i Flag risultanti da tale  sottrazione fittizia.
- In condizioni normali il '181 esegue una sottrazione (SBC) quando si configurano i segnali M/S3-S0 = 00110 (Subtract Mode).

Come è possibile gestire sia le sottrazioni reali sia le comparazioni considerando che entrambe necessitano di mettere in input sui '181 la stessa codifica 01110 di sottrazione che deve essere assegnata alle due diverse istruzioni SBC e CMP, che dovrebbero in realtà avere opcode - e dunque codifiche - diversi?

- Come detto sopra, l'operazione di sottrazione è codificata nel '181 come M/S3-S0 = 00110 (e non è modificabile).
- Si identifica un opcode arbitrario per le operazioni di comparazione utilizzandone uno che è assegnato a un'operazione aritmetica del '181 inutilizzata, ad esempio A AND NOT B, che ha come codice M/S3-S0 = 00111.
- La differenza tra l'operazione A Minus B e l'operazione A AND NOT B sta nell'ultimo bit: la prima si attiva con M/S3-S0 = 0011**0**, la seconda con M/S3-S0 = 0011**1**.
- Quando l'IR carica trova una istruzione 00111 di comparazione, metterà tale codifica in output verso le ROM e verso l'ALU, ma l'ultimo bit di transita attraverso la NOR.
- Una delle EEPROM ospitanti il microcode, quando troverà in ingresso xxx00111, dovrà attivare il segnale LF (ALU Force) su un input della NOR, così questa invertirà l'ultimo bit 0011**1** così che in realtà venga trasmesso 0011**0** all'ALU, che si metterà dunque in Subtract Mode ed effettuando la sottrazione necessaria al calcolo dei flag.

Tutto questo ragionamento deriva da ciò che ho compreso con grande fatica. La documentazione dell'NQSAP segnalava che "poiché la ALU è legata all'IR, ci sono solo 8 Opcode disponibili per metterla in Subtract Mode", ma non capivo cosa volesse dire. "Per creare i 16 Opcode necessari per tutte le combinazioni di Subtract e Compare, si mette una NOR su ALU-S0 (IR 0) e l'altro input su LF, così possiamo "riutilizzare" la Selection 0111 come se fosse 0110, che è la modalità Subtract".

La tabella successiva evidenzia come nella disponibilità di un byte (256 combinazioni possibili) per la codifica delle istruzioni, solo 8 combinazioni (2^3) sonocompongano gli opcode (fra tutti i 256) che permettono di avere M e S3-S0 in Subtract Mode, cioè **00110**: tuttavia, la gestione di tutte le operazioni di sottrazione SBC e le comparazioni CMP, CPX e CPY richiede ben più di 8 combinazioni, poiché si devono infatti poter gestire anche tutte le combinazioni di indirizzamenti. Ecco che il segnale LF (ALU Force) trasforma la codifica 00111 (corrispondente all'operazione inutilizzata A AND NOT B) in 00110, che attiva nuovamente l'operazione aritmetica di sottrazione del '181 e che ci permette di ottenere 16 opcode totali da inserire nel microcode per la gestione di sottrazioni e comparazioni in tutte le [modalità di indirizzamento](https://www.masswerk.at/6502/6502_instruction_set.html#modes) previste nel 6502.

| Bit IR  | 7  | 6  | 5  | 4  | 3  | 2  | 1  | 0  |
|  -      | -  | -  | -  | -  | -  | -  | -  | -  |
| Segnale | M  | S3 | S2 | S1 | S0 | -  | -  | -  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 0  | 0  | 0  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 0  | 0  | 0  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 0  | 1  | 1  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 0  | 1  | 1  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 1  | 0  | 0  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 1  | 0  | 0  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 1  | 1  | 1  |
| Opcode  | **0**  | **0**  | **1**  | **1**  | **0**  | 1  | 1  | 1  |

| | M  | S3 | S2 | S1 | S0 | Operazione  | S3/S0 Hex |
| 0  | 0  | 0  | 0  | 0  | 1  | A plus 1    |  00 + C*  |
| 0  | 0  | 0  | 0  | 1  | 1  | Tutti 0     |  03 + C*  |
| 0  | 0  | 0  | 1  | 1  | 0  | A minus B   |  06 + C*  |
| 0  | 0  | 0  | 1  | 1  | 1  | CMP         |  07**     |
| 1  | 0  | 0  | 0  | 1  | 1  | Tutti 1     |  03       |
| 1  | 0  | 1  | 0  | 0  | 1  | A plus B    |  09       |
| 1  | 0  | 1  | 1  | 0  | 0  | A plus A    |  0C***    |
| 1  | 0  | 1  | 1  | 1  | 1  | A minus 1   |  0F       |
| x  | 1  | 0  | 0  | 0  | 0  | Not A       |  10****   |
| x  | 1  | 0  | 1  | 1  | 0  | A XOR B     |  16****   |
| x  | 1  | 1  | 0  | 1  | 0  | B           |  1A****   |
| x  | 1  | 1  | 0  | 1  | 1  | A AND B     |  1B****   |
| x  | 1  | 1  | 1  | 1  | 0  | A OR B      |  1E****   |

*Sintesi operazioni dell'ALU dell'NQSAP.*


- Ricordo che uno degli aspetti che mi ha portato via letteralmente decine di ore per la comprensione del funzionamento era la frase di Tom "Poiché ho solo 16 combinazioni possibili , utilizzo un XXX per poter mettere il chip in subtract mode"
- doppia modalità di lavoro, active low o active high
- subtract mode
complemento di 1

*********************************
CONTROLLARE LA PAROLA CARRY CON INIZIALE MAIUSCOLA O NO A SECONDA DEI CONTESTI
*********************************



## Aritmetica Binaria

Il BEAM 

## Ripple Mode Carry e Carry Look Ahead

Ne parliamo perché i '161 usati nel MAR e i '181 dell'ALU ne parlano nei datasheet.


### Link utili

- [Inside the vintage 74181 ALU chip: how it works and why it's so strange](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) di Ken Shirriff. Fondamentale per capire il perché dell'implementazione apparentemente così strana del chip.
- David Courtney


### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Dopo aver letto questo punto avevo iniziato a raccogliere i miei pensieri per l'espansione di memoria a 256 byte, idea che riprendo in seguito in questa pagina. Non avevo intenzione di costruire un IR a più byte, cosa che mi sembrava piuttosto complessa per le mie capacità.

### Memorie con IO separati o IO comuni?

Fino ad ora, avevo quasi sostanzialmente dato per scontato di continuare ad usare chip di memoria con porte di Input e Output separati ("dual-port"), esattamente come accade nel [74189](https://eater.net/datasheets/74189.pdf) utilizzato nel SAP. Tuttavia, in questo [post su Reddit](https://www.reddit.com/r/beneater/comments/hon6ar/74189_alternative/), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf); ho così iniziato ad approfondire le caratteristiche di questo chip "single-port", aumentando la mia comprensione di queste due diverse architetture.

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

- **CE** = Chip Enable
- **OE** = Output Enable
- **WE** = Write Enable

L'utente segnala che ci sono due modalità di scrittura; quella evidenziata da lui prevede OE HI, CE LO e l'impulso WE LO; l'altra modalità riportata nel datasheet ("WE Controlled") prevede sia OE sia CE allo stato LO e l'impulso WE LO. In questa seconda modalità l'unico segnale da gestire è WE, che dovrà essere HI nelle fasi di lettura e *pulsato* LO nelle fasi di scrittura. Nel primo e nel secondo schema del modulo RAM di mia ideazione avevo adottato la prima delle due modalità, mentre nel modulo definitivo - e dopo aver compreso meglio i risvolti dell'una e dell'altra - ho utilizzato la seconda modalità. Più avanti in questa pagina ci sono altre note relative all'argomento.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo prima avere la possibilità di programmare la RAM ("Program Mode") e  poi di renderla visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è manuale e sfrutta dei dip-switch per indirizzare e programmare manualmente la RAM.

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata esclusivamente dal MAR e altrettanto esclusivamente viene acceduta in lettura / scrittura solo dal bus del computer; da notare che è possibile passare manualmente al Program Mode per leggere o scrivere il contenuto della RAM in stile "debug mode" e riprendere poi la normale esecuzione del programma.

[![Modulo RAM di rolf electronics](../../assets/ram/20-rolf-ram.png "Modulo RAM di rolf electronics"){:width="50%"}](../../assets/ram/20-rolf-ram.png)

Il funzionamento e la necessità dei transceiver mi erano chiarissimi, in quanto ampiamente utilizzati nel SAP computer per poter attivare i vari moduli del computer solo nel momento in cui fosse necessario farlo: tipicamente ogni modulo ha bisogno di un unico transceiver di interconnessione verso il bus.

Lo schema del modulo RAM di Rolf ne prevede invece due, uno "interno" e uno "esterno", per separare il percorso dei dati *verso* la RAM da quello dei dati *dalla* RAM:

- Il transceiver di sinistra è attivo quando si scrive *sulla* RAM, selezionando quale sia la sorgente mediante il MUX '157 (che in "Program Mode" attiva gli ingressi connessi al dip-switch, mentre in "Run Mode" attiva gli ingressi connessi al bus); i LED mostrano il valore che viene scritto sulla RAM.
- Il transceiver di destra è attivo quando si legge *dalla* RAM; i LED mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.

Proseguendo nello studio, ho trovato [questo schema](https://imgur.com/a/ruclh) dell'utente jaxey1631, che aveva lasciato un commento nel video di Ben Eater [Reprogramming CPU microcode with an Arduino](https://www.youtube.com/watch?v=JUVt_KYAp-I&lc=UgjusLoROw6az3gCoAEC):

[![RAM e MAR con doppio bus](../../assets/ram/20-ram-ruclh.png "RAM e MAR con doppio bus"){:width="100%"}](../../assets/ram/20-ram-ruclh.png)

*Schema di RAM e MAR con bus interno.*

In questo schema troviamo:

- Il chip di RAM 62256.

- Un FF '273 in alto a destra che, sprovvisto di Enable come discusso in precedenza, si attiva in corrispondenza di un Enable fittizio costruito con l'operazione logica "Clock *AND* Memory Address Register In" (ingressi 1A e 1B del NAND a sinistra nello schema); il segnale MI indica che il computer si prepara a settare l'indirizzo di RAM sul quale eseguirà la prossima operazione.

- Un secondo transceiver '245 che si attiva nel momento in cui si deve leggere *dalla* RAM e trasferirne l'output verso il bus dati (anche in questo caso il pin DIR del '245 settato a LO configura i pin A1-A8 come ingressi e i pin B1-B8 come uscite); notare il suo ingresso OE connesso al segnale RO (RAM Output) del computer.

[![Write cicle del 62256](../../assets/ram/20-ram-write-cycle.png "Write cicle del 62256"){:width="50%"}](../../assets/ram/20-ram-write-cycle.png)

*Write Cycle "WE Controlled" del 62256.*

E' stato in questo momento (agosto 2022) che ho scoperto l'**NQSAP**, inserendolo tra i miei appunti come "c'è questo [https://tomnisbet.github.io/nqsap/docs/ram/](https://tomnisbet.github.io/nqsap/docs/ram/) che sembra aver fatto delle belle modifiche al suo computer" 😁; ho deciso di seguire questo progetto perché permetteva di costruire un instruction set come quello del 6502 che, come scoprirò in seguito, richiederà un numero elevato di indirizzi per il microcode delle EEPROM.

Tra i vari link sondati, c'era anche [questo post Reddit](https://www.reddit.com/r/beneater/comments/h8y28k/stepbystep_guide_to_upgrading_the_ram_with/), che molti utenti hanno trovato ben fatto, ma che io ho trovato particolarmente difficile da digerire in quanto mancante di uno schema.

## Gestione della RAM

Tra i post più utili relativi alla comprensione dei segnali di gestione di RAM e MAR per il modulo di memoria con IO comuni, c'è certamente il [Question about RAM replacement](https://www.reddit.com/r/beneater/comments/ut1oud/8bit_question_about_ram_replacement/), nel quale il moderatore The8BitEnthusiast invita a consultare la sua (*eccellente*, aggiungo io) realizzazione, che ho preso ad esempio e ispirazione.

[![Modulo RAM dell'utente Reddit The8BitEnthusiast](../../assets/ram/20-ram-the8bit_enthusiast.png "Modulo RAM dell'utente Reddit The8BitEnthusiast"){:width="100%"}](../../assets/ram/20-ram-the8bit_enthusiast.png)

*Modulo RAM dell'utente e moderatore Reddit The8BitEnthusiast.*

Se da un certo punto di vista lo schema era particolarmente semplificato rispetto a quelli che avevo visto in precedenza, dall'altra, con l'utilizzo di un solo MUX '157 con segnali che ne uscivano per rientrarne in altre porte, mi risultava di difficile comprensione. Avevo pertanto provato a costruirmi i grafici temporali degli schemi dei segnali per questi tre possibili casi:

- Scrittura sulla RAM in Run Mode

[![Scrittura sulla RAM in Run Mode](../../assets/ram/20-ram-run-mode-write-t8be.png "Scrittura sulla RAM in Run Mode"){:width="30%"}](../../assets/ram/20-ram-run-mode-write-t8be.png)

- Lettura dalla RAM in Run Mode

[![Lettura dalla RAM in Run Mode](../../assets/ram/20-ram-run-mode-read-t8be.png "Lettura dalla RAM in Run Mode"){:width="30%"}](../../assets/ram/20-ram-run-mode-read-t8be.png)

- Scrittura sulla RAM in Program Mode

[![Scrittura sulla RAM in Program Mode](../../assets/ram/20-ram-program-mode-write-t8be.png "Scrittura sulla RAM in Program Mode"){:width="30%"}](../../assets/ram/20-ram-program-mode-write-t8be.png)

The8BitEnthusiast segnalava di *aver sfruttato il ritardo di propagazione dei '245 per gestire i requisiti di temporizzazione*, al che avevo provato a chiedergli se fosse necessario gestire le temporizzazioni in maniera così precisa perché il suo progetto lavorava in modalità "just in time" ogni volta che sopraggiungeva un impulso di clock.

Per esempio, ipotizzavo che nel primo caso "Scrittura sulla RAM in Run Mode" accadesse quanto segue.

- **Prima del Rising Edge del CLK**:

  - siamo in Run Mode, dunque PROG è HI
  - il MUX abilita gli ingressi I1a, I1b, I1c
  - Zb è HI (in quanto l'input I1b è connesso a Vcc) e dunque il '245 di destra, che connette il dip-switch di programmazione, è disabilitato
  - il segnale RI è HI (RI = RAM In, cioè scrittura sulla RAM), dunque stiamo preparando una scrittura
  - il segnale /RO è HI (RO = RAM Out, cioè lettura dalla RAM), dunque la lettura dalla RAM è disabilitata
  - Zc = /(CLK LO * RI HI) = HI, dunque la direzione del '245 di sinistra, che connette il data bus, è A-->B (Output)
  - Za = //(/RO HI * Zc HI) = HI, dunque il '245 dal/al data bus è disabilitato

- **CLK attivo**:

  - Zc = /(CLK HI * RI HI) = LO, dunque la direzione del '245 dal/al data bus è B-->A (Input)
  - Za = //(/RO HI * Zc LO) = LO, dunque il '245 dal/al data bus è attivo
  - /WE = Zc = LO, dunque la RAM riceve il Falling Edge del segnale di Write e trova sui suoi ingressi quanto le viene proiettato dal '245 dal/al data bus

Legenda:

- **PROG** è il segnale dell'interruttore di selezione della modalità Program Mode (LO) / Run Mode (HI); negli schemi originali del SAP computer si trova nel MAR
- **/** significa NOT
- **\*** significa AND

The8BitEnthusiast aveva gentilmente risposto al mio quesito:

> Dovevo assicurarmi che i ‘245 non consegnassero dati alla RAM quando questa non era ancora pronta per accettare dati in Input perché le sue uscite erano ancora attive in output\*\*. Il datasheet segnala che la RAM disabilita l’output ed è pronta per l’input 20 nanosecondi dopo che WE viene portato allo stato LO.

[![Timing RAM 62256](../../assets/ram/20-ram-62256-timing.png "Timing RAM 62256"){:width="66%"}](../../assets/ram/20-ram-62256-timing.png)

\*\* Nello schema del modulo RAM di The8BitEnthusiast si nota che il segnale OE della RAM è connesso a ground, che significa che i pin dati sono sempre attivi in output, *tranne* quando si deve effettuare una scrittura. Quando si attiva il segnale di write WE, vi è un tempo tWHZ durante il quale la RAM è ancora attiva in output; trascorso questo tempo, è possibile mettere dei dati in input sulla RAM.

## Design dei moduli MAR e RAM del BEAM

Parallelamente agli studi dei lavori di altri utenti, avevo iniziato a lavorare sul disegno dei miei moduli MAR e RAM, non senza continuare ad aprire altri filoni di studio per approfondire temi ancora parzialmente oscuri o affrontare argomenti nuovi. Avevo regolarmente alcune *decine* di pagine web aperte contemporaneamente.

Il MAR era progettato a 8 bit per pilotare 256 indirizzi di memoria. Inizialmente avevo utilizzato due FF '173 a 4 bit, sostituiti in seguito da un unico FF '273 a 8 bit (sprovvisto di controllo dell'attivazione dell'output, che però non è necessario). Infine ho utilizzato un registro a 8 bit '377, altrettanto adatto al nostro scopo.

[![Memory Address Register](../../assets/ram/20-mar-beam.png "Memory Address Register"){:width="100%"}](../../assets/ram/20-mar-beam.png)

*Memory Address Register (MAR) del BEAM.*

Notare l'interruttore di selezione Program Mode / Run Mode (segnale /PROG, che è connesso anche al modulo RAM):

- alla chiusura dei contatti 1-2 i pin di selezione dei MUX '157 si trovano allo stato logico LO, attivando gli ingressi I0a, I0b, I0c ed I0d, che trasmettono così al bus indirizzi della RAM i valori settati sul dip-switch;
- alla chiusura dei contatti 2-3 i pin di selezione dei MUX '157 si trovano allo stato logico HI, attivando gli ingressi I1a, I1b, I1c ed I1d, che trasmettono così al bus indirizzi della RAM i valori presenti in output sul '377.

### Prima versione del modulo RAM

Come già detto, per quanto riguarda la realizzazione del modulo RAM avevo deciso di procedere con il chip 62256 con IO comuni. Per cercare di fissare i concetti, avevo trascritto nuovamente le differenze tra le architetture con chip con IO separati ed IO comuni:

- Con le RAM dual-port, avevo i segnali di RAM IN e RAM OUT su bus "separati":
  - il bus "IN" collegava le uscite dei MUX 74LS157 ai pin Data In delle RAM: i MUX erano sempre attivi sulle porte "Write" delle RAM e mostravano ad esse tutto quello che accadeva sul bus principale del computer in Run Mode o sul DIP-Switch in Program Mode, ma non era un problema, in quanto le RAM scrivevano solo in corrispondenza del segnale RI (RAM In);
  - il bus "OUT" collegava il transceiver '245 e le uscite Data Out delle RAM; l'uscita del transceiver veniva abilitata solo in corrispondenza del segnale /RO (RAM Out).
  - Output Enable /OE LO fisso;
  - Chip Enable /CE *pulsato* LO ("↘↗");
  - /WE ↘↗ (che deve essere "contenuto" all'interno del ciclo ↘↗ di /CE).

[![Prima versione del modulo RAM](../../assets/ram/20-ram-1st.png "Prima versione del modulo RAM"){:width="100%"}](../../assets/ram/20-ram-1st.png)

*Prima versione del Modulo di memoria (RAM) del BEAM.*

Riponevo grandi speranze su questo primo design.

Per scrivere in RAM in Run Mode:

- RI (che nel ciclo di Write è HI) abilita il '245 superiore mediante una NOT;
- /RO (che nel ciclo di Write è HI) porta /OE a HI, disabilitando l'output del 62256;
- /CE (aka /CS, Chip Select) è fisso LO in quanto connesso a ground;
- i valori presenti nel bus o nel dip-switch vengono presentati via MUX al transceiver che, a sua volta, presenta tali valori alle porte della RAM;
- Zd, e dunque /WE,  è HI, in quanto il pin 1 della NAND è LO
- al rising edge del clock l'uscita della NAND passa da HI a LO per il tempo RC e l'impulso /WE ↘↗ viene trasmesso alla RAM attraverso il MUX '157: la RAM memorizza quanto trova nelle sue porte di ingresso D0-D7.

In Program Mode l'ingresso I0d del MUX è HI grazie alla resistenza di pull-up da 1K, ma premendo il bottone si crea un impulso negativo che attiva un ciclo /WE ↘↗: anche in questo caso la RAM memorizza quanto trova nelle sue porte di ingresso D0-D7.

Per leggere dalla RAM si devono soddisfare le seguenti condizioni:

- /CE LO, cioè il chip deve essere abilitato
- /OE LO, cioè le uscite devono essere attivate
- /WE HI, dunque non devo attivare il segnale di scrittura

La sequenza degli eventi è dunque la seguente:

- RI è LO, dunque la NOT disabiliterà il transceiver superiore, che non metterà dunque alcunché in output verso la RAM;
- RO LO abilita il XCVR inferiore e /OE della RAM, che viene attivata in output;
- /CE (aka /CS, Chip Select) è fisso LO in quanto connesso a ground;
- nel frattempo WE è HI (e dunque la scrittura è inibita) perché:
  - in Run Mode RI LO mantiene HI all'uscita della NAND connessa a I1d del MUX (se uno dei due input della RAM è LO, l'uscita è HI) e dunque anche /WE è fisso HI, inibendo la scrittura;
  - in Program Mode l'ingresso I0d del MUX è fisso HI grazie alla resistenza da 1K, perciò anche in questo caso non vi è scrittura.

Una analisi successiva di questo schema, che "all'occhio" era molto bello, mi evidenziava che probabilmente avevo gestito correttamente il discorso del "bus interno" e che anche la fase di output poteva essere funzionale, mentre nella fase di input notavo ridondanze superflue: avrei potuto far lavorare il transceiver inferiore bidirezionalmente a seconda della necessità per far interagire RAM e BUS e avrei potuto anche eliminare i MUX e collegare il transceiver superiore direttamente al dip-switch, attivandolo solo al momento opportuno per la programmazione manuale della RAM. In pratica, sarei ritornato alla soluzione concepita da The8BitEnthusiast, senza tuttavia aver ancora acquisito capacità e autonomia sufficienti per progettare una soluzione di attivazione just-in-time dei transceiver come aveva fatto lui.

### Seconda versione del modulo RAM

Stabilito che una soluzione just-in-time sarebbe stata prematura per le mie competenze, avevo dunque provato a ridisegnare lo schema con un solo transceiver mantenendo i MUX per gestire l'input della RAM, facendolo provenire dal bus o dal dip-switch a seconda della modalità Program o Run-Mode.

[![Seconda versione del modulo RAM](../../assets/ram/20-ram-2nd.png "Seconda versione del modulo RAM"){:width="100%"}](../../assets/ram/20-ram-2nd.png)

*Seconda versione del Modulo di memoria (RAM) del BEAM.*

A livello temporale questo è anche il momento ufficiale della nascita del nome BEAM 😁.

Dopo aver realizzato il disegno, mi sembrava di aver aggiunto più chip (per la logica) rispetto a prima = maggior complessità. Tuttavia, l'idea continuava a piacermi ed era un interessante esercizio logico per provare a sfruttare l'unico transceiver invertendone la direzione a seconda dell'operazione da fare.

Avevo provato a chiedere un [consiglio](https://www.reddit.com/r/beneater/comments/10inkvs/8bit_computer_ram_module_would_this_work/) su Reddit e il solito The8BitEnthusiast, instancabile, mi aveva dato alcune indicazioni e una risposta tutto sommato positiva.

Per analizzare tutti gli stati logici possibili avevo preparato una tabella di riepilogo con la quale verificare se il comportamento del modulo fosse in linea con le aspettative; la tabella mostrata in seguito è solo una parte di quella completa.

![Tabella analisi modulo RAM](../../assets/ram/20-ram-2nd-table.png "Tabella analisi modulo RAM"){:width="100%"}

*Tabella riepilogativa analisi stati logici seconda versione modulo RAM.*

A un certo punto ho capito che nello schema c'era un problema piuttosto importante: i due MUX 157 proiettavano in continuazione i loro input verso il bus interno, causando un possibile short con la RAM quando questa si trovava in output mode. Ho dunque pensato di sostituire i 2x '157 con i '257, che sono tri-state.

![Tabella analisi rivista modulo RAM](../../assets/ram/20-ram-2nd-table2.png){:width="100%"}

*Tabella riepilogativa rivista analisi stati logici seconda versione modulo RAM.*

Quando sono arrivato a questo punto ho realizzato di aver lavorato inutilmente: a cosa mi servono due MUX e un 245? Sono sufficienti due 245, uno per gestire I/O sul bus, uno per gestire il dip-switch, che in effetti è quanto aveva fatto 8bitEnthusiast! Ero stato illuminato e in quel momento mi è sembrato tutto chiaro!

Nel frattempo avevo anche iniziato a rinominare i segnali logici prendendo come punto di vista il computer e non il modulo:

| Vecchio | Descrizione | Nuovo | Descrizione |
|-        |-            |-      |-            |
| RI      | RAM In      | WR    | Write RAM   |
| RO      | RAM Out     | RR    | Read RAM    |

e così via.

### Terza versione del modulo RAM

Ho dunque riscritto la "truth table" del modulo RAM:

![Tabella analisi modulo RAM - terza versione](../../assets/ram/20-ram-3rd-table.png "Tabella analisi modulo RAM - terza versione"){:width="100%"}

*Tabella analisi modulo RAM - terza versione.*

I due asterischi in tabella \*\* mi servivano a ricordare che dovevo stare particolarmente attento alla preparazione del microcode e che non dovevo mai avere /WR e /RR attivi contemporaneamente, perché non avevo previsto un controllo di eventuale conflitto. Per evitare conflitti avrei dovuto mettere su /WR una NAND a 3 ingressi che si attiva solo per CLK, RR e /WR, cioè /WE = CLK \* RR \* /WR, in altre parole: posso scrivere sulla RAM *solo* se non la sto leggendo.

[![Terza versione del modulo RAM](../../assets/ram/20-ram-3rd.png "Terza versione del modulo RAM"){:width="100%"}](../../assets/ram/20-ram-3rd.png)

*Terza versione del Modulo di memoria (RAM) del BEAM.*

Riprendendo il datasheet del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf) a pagina 6 troviamo entrambe le modalità di scrittura: quella indicata come "WRITE CYCLE 2 (CE# Controlled)" era stata utilizzata per le prime due revisioni del modulo RAM; l'altra, "WRITE CYCLE 1 (WE# Controlled)", è quella utilizzata per il disegno definitivo del modulo, perché mantenere /OE e /CE LO fissi sia per le letture sia per le scritture rappresentava una semplificazione importante.

[![Write Cycles](../../assets/ram/20-ram-write-cycles.png "Write Cycles"){:width="100%"}](../../assets/ram/20-ram-write-cycles.png)

La gestione dei segnali passa in maniera importante attraverso il MUX '157, così come avviene anche nello schema di The8BitEnthusiast, ma con qualche differenza.

Ecco un riassunto degli stati possibili:

- **/PROG**: In Program Mode il MUX attiva gli ingressi I0a, I0b, I0c e I0d. In Run Mode sono attivi I1a, I1b, I1c e I1d.
  - Za è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva /WE, scrivendo sulla RAM.
  - Zb è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva il transceiver che interconnette RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR WR**: Read RAM e Write RAM non sono attivi.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR /WR**: Read RAM non è attivo, Write RAM è attivo.
  - Za è normalmente HI (perché la NAND su I1a ha un'ingresso LO), ma in corrispondenza dell'impulso di clock passa a LO e attiva /WE, scrivendo sulla RAM.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è normalmente HI, dunque disattiva il transceiver che interconnette RAM e bus, ma in corrispondenza dell'impulso di clock passa a LO (perché Za passa a LO), attivando il transceiver.
- **/RR WR**: Read RAM è attivo, Write RAM non è attivo.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso LO e attiva il transceiver tra RAM e bus.
- **/RR /WR**: Stato illegale.

## Note

Invece dei più comuni dip-switch, ho utilizzato dei comodissimi Rocker Switch ("a bilanciere") come quelli in figura; si trovano facilmente presso i distributori di [materiale elettronico](https://us.rs-online.com/product/te-connectivity/5435640-5/70156004/). Notare che i pin originali sono piuttosto corti e non fissano correttamente lo switch alla breadboard, pertanto ho aggiunto uno zoccolo per circuiti integrati.

[![Rocker Switch](../../assets/ram/20-ram-rocker.png "Rocker Switch"){:width="33%"}](../../assets/ram/20-ram-rocker.png)

*Rocker Switch.*
