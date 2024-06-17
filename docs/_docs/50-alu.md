---
title: "ALU"
permalink: /docs/alu/
excerpt: "Costruzione dell'Unità Aritmetica e Logica del BEAM computer"
---
[![Unità Aritmetica e Logica del BEAM](../../assets/alu/50-alu-beam-small.png "Unità Aritmetica e Logica del BEAM"){:width="100%"}](../../assets/alu/50-alu-beam.png)

L'Unità Aritmetica e Logica (ALU) del SAP computer di Ben Eater era limitata a addizioni e sotttrazioni. L'NQSAP di Tom Nisbet aggiungeva operazioni logiche e di rotazione (shift) e avevo iniziato a studiarla in dettaglio.

Sviluppata attorno ai chip [74LS181](https://www.ti.com/lit/ds/symlink/sn54ls181.pdf), l'ALU dell'NQSAP presentava delle caratteristiche molto interessanti, ma di comprensione particolarmente difficile: sarà uno dei moduli sui quali spenderò più tempo, ma che alla fine darà maggiori soddisfazioni per il risultato.

Il '181 è un'ALU a 4 bit sviluppata negli anni '70 che può eseguire 16 operazioni aritmetiche e 16 funzioni logiche. E' possibile concatenare più chip per elaborare word di dimensioni maggiori.

**Utilizzo dei termini '181 e ALU come Sinonimi**: in questa pagina, troveremo i termini ALU e '181 che vengono spesso utilizzati come sinonimi. Notare che ALU potrebbe indicare sia il modulo Unità Aritmetica e Logica nella sua interezza, sia il solo chip '181. Il contesto aiuterà a comprendere se per ALU si intenda il modulo, oppure il solo chip.

Tra le caratteristiche che spiccavano nello schema dell'ALU dell'NQSAP, notavo soprattutto un numero elevato di chip - tra i quali gli Shift Register 74LS194 - e un modo particolare di indirizzare i '181, che erano "strettamente legati" all'istruzione presente nell'Instruction Register della [Control Logic](../control). Il legame con la Control Logic è stato tra i più complessi da analizzare e comprendere, ma quello con il modulo dei Flag è altrettanto importante e la sua comprensione non è stata meno complessa (ad ogni operazione dell'ALU - e non solo - corrisponde infatti un'azione sul registro dei Flag).

[![Schema logico dell'ALU di Tom Nisbet](../../assets/alu/50-alu-nqsap.png "Schema logico dell'ALU di Tom Nisbet"){:width="100%"}](../../assets/alu/50-alu-nqsap.png)

*Schema logico dell'ALU di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Il modulo ALU è sommariamente composto da due registri di input H e B e da una coppia di '181 interconnessi, che permettono di gestire una word di 8 bit.

- Il registro H è in realtà uno Shift Register in grado sia di comportarsi come un normale registro a 8 bit, sia di *shiftare* a destra o a sinistra il valore presente in ingresso (istruzioni di rotazione).
- Il registro B è un normale registro a 8 bit. Il chip utilizzato per questo registro non include un ingresso Enable, che è dunque stato realizzato in maniera artificiale mettendo una NOR su /Clock e /WB ("Write B"); in questo modo il registro si attiva solo in corrispondenza di /WB (che è attivo LO) e del falling edge del clock negato, equivalente al rising edge del clock non negato, che è il momento in cui si caricano i registri (riferimento: video di Ben Eater [8-bit CPU control logic: Part 2](https://www.youtube.com/watch?v=X7rCxs1ppyY)).
- Tre transceiver '245 permettono di poter leggere i valori contenuti in H, B ed L (L è l'output dell'A**L**U).

Come detto nell'introduzione, il computer BEAM, al pari dell'NQSAP, include il set di istruzioni completo del 6502, comprese quelle logiche e aritmetiche. Ricordavo discretamente le principali operazioni del 6502 e sapevo *abbastanza* bene quale dovesse essere il risultato di quello che stavo facendo, ma in quel momento non avevo ancora idea di come fosse possibile ottenerlo.

Avevo intanto deciso di comprendere le operazioni messe a disposizione dal '181 e se vi fosse una logica nella loro disposizione, una sorta di raggruppamento.

[![Funzioni logiche e operazioni aritmetiche del 74LS181](../../assets/alu/50-alu-operations.png "Funzioni logiche e operazioni aritmetiche del 74LS181"){:width="100%"}](../../assets/alu/50-alu-operations.png)

*Funzioni logiche e operazioni aritmetiche del 74LS181.*

Il datasheet del '181 era abbastanza criptico e dunque ho avevo fatto ricorso anche alle molte risorse disponibili in rete riportate a fondo pagina. Dal datasheet si comprende che vi sono 4 segnali S0, S1, S2 ed S3 per la selezione della funzione / operazione e un segnale di controllo della modalità M (M = HI per le funzioni logiche; M = LO per le operazioni aritmetiche); A e B sono gli input dei dati. Nel datasheet venivano menzionati anche il Carry Look-Ahead e il Ripple-Carry, che approfondirò in seguito nella sezione dedicata all'Aritmetica Binaria.

Inizialmente avevo trascritto la tabella delle funzioni / operazioni in un foglio Excel per poter lavorare più agevolmente:

![Funzioni logiche e operazioni aritmetiche del 74LS181](../../assets/alu/50-alu-operations-xls.png)

*Funzioni logiche e operazioni aritmetiche del 74LS181 - su Excel.*

Avevo evidenziato le operazioni ripetute più volte, non trovando però alcun raggruppamento o filo conduttore tra righe e colonne. Cercavo  di capire quale fosse il senso di quella disposizione così apparentemente disordinata, ma non l'avevo trovato. Illuminante è stato l'articolo di [Ken Shirriff](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) citato in calce.

L'aspetto più importante che avevo capito è cosa accomunava le due colonne delle operazioni aritmetiche (eseguite in corrispondenza della modalità M = LO):

- senza Carry ("Cn = HI  --> No Carry")
- con Carry   ("Cn = LO  --> With Carry")

Ad un occhio esperto tutto questo sarebbe sembrato ovvio, ma io mi stavo avvicinando per la prima volta al tema dell'ALU e molte cose non mi erano immediatamente chiare.

Per quanto bizzarre possano apparire alcune delle operazioni disponibili (e Ken spiega anche il perché), il filo conduttore tra le due colonne è che l'output del '181 *è sempre lo stesso*, con l'unica differenza data dalla assenza o presenza del Carry in ingresso. Si noti infatti che, in ogni riga, l'output ha sempre una differenza pari ad 1, che corrisponde al valore del Carry; ad esempio, nella decima riga troviamo la semplice operazione di somma **A Plus B**: la differenza tra quanto computato nelle due colonne è 1 (**A Plus B** in caso di nessun Carry in ingresso e **A Plus B Plus 1** in caso di Carry in ingresso).

Lo stesso ragionamento è valido per in tutte le altre operazioni aritmetiche disponibili. Prendiamo come altro esempio la prima riga: in assenza di Carry, l'ALU restituisce quanto presente agli ingressi A, mentre in presenza di Carry aggiunge 1 a quanto presente in A (l'operazione aritmetica **A Plus 1** sarà poi sfruttata per creare l'istruzione INC del computer "iniettando" un Carry artificiale nel '181; similarmente, l'istruzione DEC è stata costruita intorno all'operazione **A Minus 1** della tabella, ma in questo caso senza essere "dopata" da un Carry artificiale).

A questo punto è anche opportuno segnalare che il '181 mette a disposizione due modalità di utilizzo: una con la logica attiva bassa ("Active-Low data") e una con la logica attiva alta ("Active-High data") che è quella utilizzata nell'NQSAP; quest'ultima, per complicare un po' le cose, si attende in ingresso un *Carry In negato*, nel senso che un segnale Cn (Carry In) = LO viene interpretato come Carry attivo, mentre un segnale Cn = HI viene interpretato come Carry non presente. Allo stesso modo, anche il *Carry Out* è negato: Cn+4 è infatti HI per indicare che non c'è Carry in uscita, mentre è LO per indicare che è presente un Carry.

**XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX Vedere se qui ha senso prendere quel discorso del 6502 citato da Ken XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX**

Ritornando alla tabella delle funzioni / operazioni e cercando di seguire le spiegazioni e logica dell'NQSAP, ritenevo che solo il sottoinsieme visibile di seguito fosse utile per lo scopo prefissato, che era quello di poter emulare le istruzioni del 6502:

![Operazioni logiche e aritmetiche utili del 74LS181](../../assets/alu/50-alu-operations-xls-subset.png)

*Operazioni logiche e aritmetiche utili del 74LS181.*

Successivamente capirò che le istruzioni necessarie erano in realtà ancora meno di quelle che ipotizzavo.

## Relazione diretta tra Instruction Register e ALU

Un altro degli aspetti di più difficile comprensione, come anticipato in precedenza, è stata l'associazione diretta tra l'istruzione correntemente contenuta nell'Instruction Register e la funzione logica / operazione aritmetica eseguita dal '181.

Provando a sintetizzare quando disegnato nell'NQSAP, avevo costruito questa tabella per avere un riepilogo dei segnali applicati all'ALU, dei loro valori esadecimali corrispondenti e delle operazioni risultanti, estrapolandola dalla tabella completa delle istruzioni visibile in [NQSAP Instructions by Address Mode Group](https://tomnisbet.github.io/nqsap/docs/in-by-mode-group/) e nella quale si notano piuttosto chiaramente i raggruppamenti delle istruzioni dell'NQSAP rispetto alle funzioni logiche / operazioni aritmetiche dei '181:

| Cn | M  | S3 | S2 | S1 | S0 | Operazione  | M-S3/S0 Hex |
|  - | -  |  - |  - |  - |  - |          -  |   -         |
| 0  | 0  | 0  | 0  | 0  | 1  | A Plus 1    |  0x00 + C*  |
| 0  | 0  | 0  | 0  | 1  | 1  | Tutti 0     |  0x03 + C*  |
| 0  | 0  | 0  | 1  | 1  | 0  | A Minus B   |  0x06 + C*  |
| 0  | 0  | 0  | 1  | 1  | 1  | CMP         |  0x07**     |
| 1  | 0  | 0  | 0  | 1  | 1  | Tutti 1     |  0x03       |
| 1  | 0  | 1  | 0  | 0  | 1  | A Plus B    |  0x09       |
| 1  | 0  | 1  | 1  | 0  | 0  | A Plus A    |  0x0C***    |
| 1  | 0  | 1  | 1  | 1  | 1  | A Minus 1   |  0x0F       |
| x  | 1  | 0  | 0  | 0  | 0  | Not A       |  0x10****   |
| x  | 1  | 0  | 1  | 1  | 0  | A XOR B     |  0x16****   |
| x  | 1  | 1  | 0  | 1  | 1  | A AND B     |  0x1B****   |
| x  | 1  | 1  | 1  | 1  | 0  | A OR B      |  0x1E****   |

*Sintesi operazioni dell'ALU dell'NQSAP.*

Legenda tabella *Sintesi operazioni dell'ALU dell'NQSAP*:

- \* Avevo evidenziato queste righe per ricordare che su queste tre istruzioni si deve "iniettare" un Carry artificiale (che è invertito, dunque il segnale effettivamente applicato sul Carry In del primo '181 doveva essere LO).
- \*\* = Le istruzioni di salto relativo del 6502 dipendono dallo stato dei flag N, V, Z e C. Le istruzioni che modificano lo stato dei flag sono molte: aritmetiche, logiche, incremento/decremento, rotazione, stack, flag, caricamento/trasferimento registri e *comparazione*. Queste ultime (CMP, CPX e CPY) hanno effetto solo sui flag N, Z e C, che vengono computati effettuando una sottrazione (SBC) fittizia tra due valori, scartandone il risultato e tenendo in considerazione solo i flag risultanti dalla sottrazione (non avevo mai realmente approfondito come i flag fossero generati e questo esercizio è stato fondamentale). La differenza tra le sottrazioni e le comparazioni è dunque piuttosto semplice da comprendere:

  - nelle sottrazioni il valore risultante dalla sottrazione viene calcolato e *mantenuto*;
  - nelle comparazioni il valore risultante dalla sottrazione viene calcolato e *scartato*;
  - per entrambe le istruzioni i flag sono mantenuti.

  Per eseguire le comparazioni si eseguono dunque delle sottrazioni scartando il risultato, tuttavia le operazioni di sottrazione del '181 sono già utilizzate per eseguire le sottrazioni vere e proprie (**A Minus B**) e sono codificate nella terza riga della tabella con M/S3-S0 = **00110**: come è possibile eseguire altre operazioni di sottrazione utilizzando un opcode diverso da **00110** pur sapendo che i '181 eseguono le sottrazioni solo con questa codifica in ingresso? Approfondimenti a seguire in questa stessa pagina.

- \*\*\* L'operazione A+A veniva usata nell'NQSAP per fare lo shift verso sinistra dei bit; vista la presenza dello Shift Register H, ho preferito riversare su di esso tutte le operazioni di rotazione (a destra e a sinistra, sia con Carry sia senza Carry).
- \*\*\*\*Il Carry è ininfluente in quanto si tratta di funzione logica e non di operazione aritmetica.

Per fare un esempio, si stava in pratica dicendo che per eseguire una somma ("**A Plus B**", vedi sesta riga; "**ADC**" nella terminologia del 6502) era necessario avere:

- Cn = 1 (che, ricordiamo, è gestito con stato logico invertito, dunque in questo caso l'ALU considera il Carry non presente)
- M = **0**
- S3/S2/S1/S0 = **1001**

cioè:

| Cn | M  | S3 | S2 | S1 | S0 | Operazione | S3/S0 Hex |
|  - | -  |  - |  - |  - |  - |          - |   -       |
| 1  | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   |  0x09       |

In pratica, poiché gli ingressi M ed S3-S0 dei '181 sono direttamente connessi all'[Instruction Register](../control), l'istruzione di somma dovrà forzatamente essere codificata nel microcode presentando **01001** sui 5 bit comuni tra Instruction Register e ALU.

![Alt text](../../assets/alu/50-alu-cl-ir-out.png)

*Output dell'Instruction Register verso il modulo ALU con evidenza dei 5 bit di selezione della funzione / operazione dei '181.*

Attivando questa istruzione, il risultato esposto in output dai '181 sarebbe stato esattamente **A Plus B**, proprio come indicato nella decima riga / colonna Cn = HI della tabella estratta dal datasheet "*Funzioni logiche e operazioni aritmetiche del 74LS181.*"; se avessimo invece avuto un Carry in ingresso, il risultato esposto sarebbe stato A + B + 1, come indicato nella decima riga / colonna Cn = LO. La somma (almeno in teoria) funzionava e iniziavo anche a far luce sul legame tra le due colonne Cn = LO / Cn = HI: il risultato in output era sempre lo stesso e variava solo in conseguenza del fatto che in ingresso ci fosse un Carry o meno.

Gli altri 3 bit di output dell'IR venivano scelti arbitrariamente, pur se con qualche accorgimento strutturato, per gestire le diverse modalità di indirizzamento disponibili sul 6502 e che dovevano essere emulate dal computer.

### Indirizzamenti

Ogni istruzione del 6502, come ad esempio la succitata ADC, offre infatti un certo numero di indirizzamenti tra le 13 modalità totali disponibili (Accumulatore, Assoluto, Assoluto Indicizzato X o Y, Immediato, Implicito, Indiretto, Indiretto Indicizzato X, Indicizzato Y Indiretto, Relativo, Pagina Zero, Pagina Zero Indicizzato X o Y).

Un valido riferimento per l'analisi della relazione tra IR ed ALU è stata la pagina [6502 Instruction Set](https://www.masswerk.at/6502/6502_instruction_set.html) di Norbert Landsteiner, che invito a consultare anche per il [6502 Assembler](https://www.masswerk.at/6502/assembler.html) e il [Virtual 6502](https://www.masswerk.at/6502/) che avrei utilizzato in seguito in fase di debug del microcode che stavo sviluppando.

Dalla tabella HTML delle istruzioni avevo ricavato una tabella Excel a partire dalla quale ho ragionato sugli indirizzamenti, notando che ogni istruzione del 6502 non utilizzava mai più di 8 modalità di indirizzamento: ecco che gli altri 3 bit di output dell'IR permettevano dunque di costruire un set di istruzioni basato sulle funzioni logiche / operazioni aritmetiche del '181 incrociandole con le (non più di) 8 modalità di indirizzamento usate da ogni istruzione (2^3 = 8 modalità di indirizzamento sfruttabili da ogni istruzione).

Vale anche la pena di notare che in un computer con soli 256 byte di RAM i 3 indirizzamenti Zero Page e i 3 indirizzamenti Assoluti sono ridondanti, in quanto entrambi utilizzano gli stessi 256 byte di memoria: il risultato di questo è che le 3 modalità di indirizzamento Pagina Zero (ZP, "Zero Page" in inglese) non vengono dunque prese in considerazione.

Riprendendo l'operazione ADC / A Plus B ed integrandola con i 3 bit utilizzati per gestire le modalità di indirizzamento si costruisce - ad esempio - questa tabella degli opcode:

| Cn | I3    | I2     | I1     | M      | S3     | S2     | S1     | S0     | Operazione | Indirizzamento          | Hex   |
|  - | -     | -      | -      | -      | -      | -      | -      | -      | -          | -                       | -     |
| 1  | **0** |  **0** |  **1** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Immediato               | 0x29  |
| 1  | **0** |  **1** |  **0** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Assoluto                | 0x49  |
| 1  | **0** |  **1** |  **1** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Assoluto, X             | 0x69  |
| 1  | **1** |  **0** |  **0** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Assoluto, Y             | 0x89  |
| 1  | **1** |  **0** |  **1** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Indiretto Indicizzato X | 0xA9  |
| 1  | **1** |  **1** |  **0** | **0**  | **1**  | **0**  | **0**  | **1**  | A Plus B   | Indicizzato Y Indiretto | 0xC9  |
|  - | -     | -      | -      | -      | -      | -      | -      | -      | -          | -                       | -     |
| 0  | **0** |  **0** |  **1** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Immediato               | 0x26  |
| 0  | **0** |  **1** |  **0** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Assoluto                | 0x46  |
| 0  | **0** |  **1** |  **1** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Assoluto, X             | 0x66  |
| 0  | **1** |  **0** |  **0** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Assoluto, Y             | 0x86  |
| 0  | **1** |  **0** |  **1** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Indiretto Indicizzato X | 0xA6  |
| 0  | **1** |  **1** |  **0** | **0**  | **0**  | **1**  | **1**  | **0**  | A Minus B  | Indicizzato Y Indiretto | 0xC6  |

*Esempio della relazione tra IR ed ALU per tutte le modalità di indirizzamento delle istruzioni ADC e SBC del 6502.*

Si nota che i 5 bit dell'operazione sono sempre gli stessi per ogni modalità di indirizzamento e che solamente i 3 bit di indirizzamento variano a seconda della modalità prescelta.

Partendo da questo ragionamento diventava possibile definire gli opcode di ogni istruzione da codificare poi nel microcode.

### Istruzioni di comparazione

Abbiamo evidenziato in precedenza che le istruzioni di modificano lo stato dei flag sono molte e che tra queste ci sono anche quelle di comparazione.

Per approfondirle, partiamo ad esempio da una istruzione che fa uso dei flag risultanti da una comparazione: l'istruzione di salto relativo BMI (**B**ranch on **MI**nus). Questa istruzione viene eseguita solo in presenza del flag N, che indica che il numero risultante dalla comparazione *con segno*\* è **N**egativo, cioè compreso tra -128 e -1 (0x80 e 0xFF in esadecimale).

\* = si veda in seguito la sezione riservata all'aritmetica binaria.

Come anticipato, i flag delle istruzioni di comparazione sono calcolati eseguendo una sottrazione fittizia tra il valore contenuto nel registro A, X o Y e il valore indicato dall'operando dell'istruzione di comparazione; si prendono in considerazione solo i flag risultanti dall'operazione e il risultato della sottrazione viene scartato. I flag sono dunque generati sfruttando l'operazione di sottrazione del '181, che è però già utilizzata per eseguire l'operazione standard di sottrazione **A Minus B** (terza riga della tabella *Sintesi operazioni dell'ALU dell'NQSAP* dove M-S3/S2/S1/S0 = 00110 e il microcodice dovrà perciò presentare **00110** sui 5 bit comuni tra Instruction Register e ALU).

- Tutti i segnali che pilotano i '181 derivano direttamente dall'Instruction Register (IR), eccetto per il Carry In. Si può dire che l'ALU è *hardwired* all'IR e che pertanto il microcode del computer deve essere scritto in modo tale che le istruzioni che utilizzano l'ALU rispecchino i segnali in ingresso del '181: ad esempio, osservando la tabella *Esempio della relazione tra IR ed ALU per tutte le modalità di indirizzamento delle istruzioni ADC e SBC del 6502*, l'istruzione di somma **A Plus B** dovrà avere i bit comuni tra IR ed ALU codificati come **01001**, mentre l'istruzione di sottrazione **A Minus B** dovrà averli codificati come **00110**.

- Come si nota nell'estratto dello schema dell'IR, il segnale S0 è in realtà solo "parzialmente diretto" verso i '181, perché transita prima attraverso una NOR che viene pilotata da una ROM ed attivata solo in corrispondenza delle istruzioni di comparazione, cosicché la codifica **0011*1***, da noi arbitrariamente designata per indicare le istruzioni di comparazione, venga presentata ai '181 come **0011*****0***, che è nuovamente il codice per l'istruzione di sottrazione (Subtract Mode)!

In altre parole, il microcode delle istruzioni di comparazione (che nella mnemonica del 6502 sono CMP, CPX o CPY) dovrà attivare un segnale ("*LF*") in una delle EEPROM: questo segnale attiverà la porta NOR per trasmettere ai '181 il codice 0011**0** della sottrazione anziché 0011**1**, che corrisponde originariamente al codice dell'operazione **A AND NOT B**, non necessario per simulare le istruzioni del 6502 e dunque inutilizzato. LF è l'abbreviazione di A**L**U **F**orce.

Detto in altre parole ancora:

- Le istruzioni di comparazione del 6502 simulano una sottrazione e tengono in considerazione solo i flag risultanti da tale sottrazione fittizia.
- In condizioni normali il '181 esegue una sottrazione (**SBC**) quando si configurano i segnali M/S3-S0 = **00110** (Subtract Mode).

Come è possibile gestire sia le sottrazioni reali sia le comparazioni, considerando che entrambe necessitano di mettere in input sui '181 la stessa codifica **01110**, la quale deve però essere assegnata sia alle istruzioni di sottrazione sia a quelle di comparazione, che devono in realtà avere opcode diversi - e dunque anche codifiche diverse?

- Come detto sopra, l'operazione di sottrazione è codificata nel '181 come M/S3-S0 = **00110** (e non è modificabile).
- Si identifica dunque un opcode arbitrario per le operazioni di comparazione utilizzandone uno che è assegnato a un'operazione aritmetica del '181 inutilizzata, ad esempio A AND NOT B, che ha come codice M/S3-S0 = **00111**.
- La differenza tra l'operazione A Minus B e l'operazione A AND NOT B sta nell'ultimo bit: la prima si attiva con M/S3-S0 = 0011**0**, la seconda con M/S3-S0 = 0011**1**.
- Quando l'IR carica una istruzione 00111 di comparazione, metterà tale codifica in output verso le ROM e verso l'ALU, ma l'ultimo bit di tale codifica transiterà anche attraverso la NOR prima di raggiungere l'ALU.
- Una delle EEPROM ospitanti il microcode, quando troverà in ingresso xxx00111, attiverà il segnale LF sull'altro input della NOR: come conseguenza la NOR invertirà l'ultimo bit 0011**1** e i '181 troveranno in realtà in ingresso 0011**0**, configurandosi dunque in Subtract Mode ed effettuando la sottrazione.

Importante evidenziare che la modalità **Subtract Mode** del '181 non è altro che la configurazione di Input M/S3-S0 = 00110 che equivale alla operazione aritmetica di sottrazione, come chiarito da David Courtney nel video [Comparator Functions of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=jmROTNtoUGI). Il datasheet non era così chiaro relativamente alla definizione di questa modalità.

Confesso di aver impiegato diverso tempo per comprendere tutto questo e "farlo mio". La documentazione dell'NQSAP segnalava che "poiché la ALU è legata all'IR, ci sono solo 8 Opcode disponibili per metterla in Subtract Mode", ma non capivo cosa volesse dire. "Per creare i 16 Opcode necessari per tutte le combinazioni di Subtract e Compare, si mette una NOR su ALU-S0 (IR 0) e l'altro input su LF, così da  riutilizzare la Selection 0111 come se fosse 0110, che è la modalità Subtract".

La tabella successiva evidenzia come nella disponibilità di un byte per la codifica delle istruzioni (256 combinazioni possibili), solo 8 combinazioni (2^3, descritte dai bit I3-I2-I1) siano quelle degli opcode che permettono di avere M e S3-S0 in Subtract Mode, cioè **00110**: tuttavia, la gestione di tutte le operazioni di sottrazione SBC e di comparazione CMP, CPX e CPY richiede ben più di 8 combinazioni, poiché si devono infatti poter gestire anche tutte le combinazioni degli indirizzamenti del 6502. Ecco che il segnale LF (ALU Force) trasforma la codifica 00111 (corrispondente all'operazione inutilizzata A AND NOT B) in 00110, che attiva nuovamente l'operazione aritmetica di sottrazione del '181 e che ci permette di ottenere 16 opcode totali da inserire nel microcode per la gestione di sottrazioni e comparazioni in tutte le [modalità di indirizzamento](https://www.masswerk.at/6502/6502_instruction_set.html#modes) previste nel 6502.

| Bit IR  | 7      | 6      | 5      | 4  | 3   | 2   | 1   | 0   |
|  -      | -      | -      | -      | -  | -   | -   | -   | -   |
| Segnale | I3     | I2     | I1     | M  | S3  | S2  | S1  | S0  |
| Opcode  | **0**  | **0**  | **0**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **0**  | **0**  | **1**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **0**  | **1**  | **0**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **0**  | **1**  | **1**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **1**  | **0**  | **0**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **1**  | **0**  | **1**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **1**  | **1**  | **0**  | 0  | 0   | 1   | 1   | 0   |
| Opcode  | **1**  | **1**  | **1**  | 0  | 0   | 1   | 1   | 0   |

*Istruzione di sottrazione SBC e le 8 combinazioni possibili date dalle modalità di indirizzamento.*


Letto dopo averlo capito sembra semplice; inizialmente non lo era proprio.

### Esempio di addizione e sottrazione con Carry

Supponiamo di fare un'operazione **A Plus B** con due ALU. Il **/Cn+4** (Carry Out) del primo '181 entra nel **/Cn** (Carry In) del econdo. Mettiamo in ingresso nel un segnale allo stato logico HI, che corrisponde a non avere un Carry (ricordiamo che nella logica "Active-High Data" il Carry è negato).

- Se /Cn+4 del primo '181 è HI, significa che il suo risultato di A Plus B non comporta un Carry, la cui assenza sarà propagata al secondo '181, che troverà dunque l'ingresso Cn allo stato HI: entrambi i '181 eseguiranno A Plus B.
- se /Cn+4 del primo '181 è LO, significa che il suo risultato di A Plus B comporta un Carry, la cui presenza sarà propagata al secondo '181, che troverà dunque l'ingresso Cn allo stato LO: il primo '181 eseguirà dunque l'operazione A Plus B, mentre il secondo eseguirà l'operazione A Plus B plus 1.

Per eseguire invece una sottrazione **A minus B** dobbiamo attivare il Carry, cioè settare /Cn = LO **(un po' come il SEC del 6502, da settare prima di una sottrazione)**

- Se il primo '181 non genera un prestito ("borrow"), il suo /Cn+4 sarà allo stato logico LO ed anche il secondo '181 eseguirà l'operazione A minus B
- Se invece il primo '181 genererà un borrow, il suo /Cn+4 sarà allo stato logico HI e dunque il secondo '181 eseguirà A minus B - 1.

Questi punti sono spiegati in dettaglio da Tom nella sezione [Carry Flag](https://tomnisbet.github.io/nqsap/docs/74181-alu-notes/#carry-flag) della sua pagina dedicata all'ALU.

Annotavo che per come funziona il '181 e provando a fare delle addizioni o sottrazioni con e senza Carry, si potrebbe pensare di poter semplicemente eseguire un OR esclusivo (XOR) tra le uscite del Carry /Cn+4 dei due chip per capire se c'è Overflow o no. Tuttavia il meccanismo non funziona in caso di istruzioni INC e DEC (e dunque per la verifica dell'esistenza di un overflow si ricorrerà ad un altro metodo, **come si vedrà in seguito**).

Per quale motivo la verifica suddetta non funziona in tutte le situazioni?

- Eseguendo un'operazione A + 1 (si vedano i segnali da applicare al '181 *Sintesi operazioni dell'ALU dell'NQSAP*) si possono verificare due casi - facciamo due esempi:
  1) A = 0000 0101 che incrementato diventa 0000 0110; il primo '181 presenta un Carry in ingresso attivo (LO); il suo Carry in uscita (Cn+4) è HI, cioè non attivo;
  2) A = 0000 1111 che incrementato diventa 0001 0000; il primo '181 presenta un Carry in ingresso attivo (LO); il suo Carry in uscita (Cn+4) è LO, cioè attivo.
- Nel secondo caso non ho reale Overflow del risultato, ma se andassi a interpretare i Carry in uscita dei due '181 con una funzione XOR incorrerei in un errore.


Inizialmente complessa 

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SONO ARRIVATO QUI

*Opcode validi per le operazioni aritmetiche di sottrazione.*

- complemento di 1

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

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

### Memorie con IO separati o IO comuni?

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.

- **CE** = Chip Enable
- **OE** = Output Enable
- **WE** = Write Enable

L'utente segnala che ci sono due modalità di scrittura; quella evidenziata da lui prevede OE HI, CE LO e l'impulso WE LO; l'altra modalità riportata nel datasheet ("WE Controlled") prevede sia OE sia CE allo stato LO e l'impulso WE LO.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo prima avere la possibilità di programmare la RAM ("Program Mode") e  poi di renderla visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

Il funzionamento e la necessità dei transceiver mi erano chiarissimi, in quanto ampiamente utilizzati nel SAP computer per poter at

- Il transceiver di destra è attivo quando si legge *dalla* RAM; i LED mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.

- Un secondo transceiver '245 che si attiva nel momento in cui si deve leggere *dalla* RAM e trasferirne l'output verso il bus dati (anche in questo caso il pin DIR del '245 settato a LO .

[![Write cicle del 62256](../../assets/ram/20-ram-write-cycle.png "Write cicle del 62256"){:width="50%"}](../../assets/ram/20-ram-write-cycle.png)

*Write Cycle "WE Controlled" del 62256.*

## Gestione della RAM

Tra i post più utili relativi alla comprensione dei segnali di gestione di RAM e MAR per il modulo di memoria con IO comuni, c'è certamente il [Question about RAM replacement](https://www.reddit.com/r/beneater/comments/ut1oud/8bit_question_about_ram_replacement/), nel quale il moderatore The8BitEnthusiast invita a consultare la sua (*eccellente*, aggiungo io) realizzazione, che ho preso ad esempio e ispirazione.

Se da un certo punto di vista lo schema era particolarmente semplificato rispetto a quelli che avevo visto in precedenza, dall'altra, con l'utilizzo di un solo MUX '157 con segnali che ne uscivano per rientrarne in altre porte, mi risultava di difficile comprensione. Avevo pertanto provato a costruirmi i grafici temporali degli schemi dei segnali per questi tre possibili casi:

Per esempio, ipotizzavo che nel primo caso "Scrittura sulla RAM in Run Mode" accadesse quanto segue.

- **Prima del Rising Edge del CLK**:

  - Zc = /(CLK LO * RI HI) = HI, 
  - Za = //(/RO HI * Zc HI) = HI, dunque il '245 dal/al data bus è disabilitato

- **CLK attivo**:

  - Zc = /(CLK HI * RI HI) = LO, dunque la direzione del '245 dal/al data bus è B-->A (Input)

Legenda:

- **PROG** è il segnale dell'interruttore di selezione della modalità Program Mode (LO) / Run Mode (HI); negli schemi originali del SAP computer si trova nel MAR
- **/** significa NOT
- **\*** significa AND

The8BitEnthusiast aveva gentilmente risposto al mio quesito:

> Dovevo assicurarmi che i ‘245 non consegnassero dati alla RAM quando questa non era ancora pronta per accettare dati in Input perché le sue uscite erano ancora attive in output\*\*. Il datasheet segnala che la RAM disabilita l’output ed è pronta per l’input 20 nanosecondi dopo che WE viene portato allo stato LO.

[![Timing RAM 62256](../../assets/ram/20-ram-62256-timing.png "Timing RAM 62256"){:width="66%"}](../../assets/ram/20-ram-62256-timing.png)

\*\* Nello schema del modulo RAM di The8BitEnthusiast si nota che il segnale OE della 

## Design dei moduli MAR e RAM del BEAM

Il MAR era progettato a 8 bit per pilotare 256 indirizzi di memoria. Inizialmente avevo utilizzato due FF '173 a 4 bit, sostituiti in seguito da un unico FF '273 a 8 bit (sprovvisto di controllo dell'attivazione dell'output, che però non è necessario). Infine ho utilizzato un registro a 8 bit '377, altrettanto adatto al nostro scopo.

[![Memory Address Register](../../assets/ram/20-mar-beam.png "Memory Address Register"){:width="100%"}](../../assets/ram/20-mar-beam.png)

*Memory Address Register (MAR) del BEAM.*

Notare l'interruttore di selezione Program Mode / Run Mode (segnale /PROG, che è connesso anche al modulo RAM):

- alla chiusura dei contatti 2-3 i pin di selezione dei MUX '157 si trovano allo stato logico HI, attivando gli ingressi I1a, I1b, I1c ed I1d, che trasmettono così al bus indirizzi della RAM i valori presenti in output sul '377.

### Prima versione del modulo RAM

Come già detto, per quanto riguarda la realizzazione del modulo RAM avevo deciso 

- Con le RAM dual-port, avevo i segnali di RAM IN e RAM OUT su bus "separati":
  - /WE ↘↗ (che deve essere "contenuto" all'interno del ciclo ↘↗ di /CE).

[![Prima versione del modulo RAM](../../assets/ram/20-ram-1st.png "Prima versione del modulo RAM"){:width="100%"}](../../assets/ram/20-ram-1st.png)

*Prima versione del Modulo di memoria (RAM) del BEAM.*

Per leggere dalla RAM si devono soddisfare le seguenti condizioni:

- /CE LO, cioè il chip deve essere abilitato
- /OE LO, cioè le uscite devono essere attivate
- /WE HI, dunque non devo attivare il segnale di scrittura

La sequenza degli eventi è dunque la seguente:

- RI è LO, dunque la NOT disabiliterà il transceiver superiore, che non metterà dunque alcunché in output verso la RAM;
- nel frattempo WE è HI (e dunque la scrittura è inibita) perché:

### Seconda versione del modulo RAM

Stabilito che una soluzione just-in-time sarebbe stata prematura per le mie competenze, avevo dunque provato a ridisegnare lo schema con un solo transceiver mantenendo i MUX per gestire l'input della RAM, facendolo provenire dal bus o dal dip-switch a seconda della modalità Program o Run-Mode.

I due asterischi in tabella \*\* mi servivano a ricordare che dovevo stare particolarmente attento alla preparazione del microcode e che non dovevo mai avere /WR e /RR attivi contemporaneamente, perché non avevo previsto un controllo di eventuale conflitto. Per evitare conflitti avrei dovuto mettere su /WR una NAND a 3 ingressi che si attiva solo per CLK, RR e /WR, cioè /WE = CLK \* RR \* /WR, in altre parole: posso scrivere sulla RAM *solo* se non la sto leggendo.

[![Terza versione del modulo RAM](../../assets/ram/20-ram-3rd.png "Terza versione del modulo RAM"){:width="100%"}](../../assets/ram/20-ram-3rd.png)

*Terza versione del Modulo di memoria (RAM) del BEAM.*

Riprendendo il datasheet del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf) a pagina 6 troviamo entrambe le modalità di scrittura: quella indicata come "WRITE CYCLE 2 (CE# Controlled)" era stata utilizzata per le prime due revisioni del modulo RAM; l'altra, "WRITE CYCLE 1 (WE# Controlled)", è quella utilizzata per il disegno definitivo del modulo, perché mantenere /OE e /CE LO fissi sia per le letture sia per le scritture rappresentava una semplificazione importante.

[![Write Cycles](../../assets/ram/20-ram-write-cycles.png "Write Cycles"){:width="100%"}](../../assets/ram/20-ram-write-cycles.png)

La gestione dei segnali passa in maniera importante attraverso il MUX '157, così come avviene anche nello schema di The8BitEnthusiast, ma con qualche differenza.

Ecco un riassunto degli stati possibili:

- **/PROG**: In Program Mode il MUX attiva gli ingressi I0a, I0b, I0c e I0d. In Run Mode sono attivi I1a, I1b, I1c e I1d.
- **/RR WR**: Read RAM è attivo, Write RAM non è attivo.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso LO e attiva il transceiver tra RAM e bus.
- **/RR /WR**: Stato illegale.

## Note

Invece dei più comuni dip-switch, ho utilizzato dei comodissimi Rocker Switch ("a bilanciere") come quelli in figura; si trovano facilmente presso i distributori di [materiale elettronico](https://us.rs-online.com/product/te-connectivity/5435640-5/70156004/). Notare che i pin originali sono piuttosto corti e non fissano correttamente lo switch alla breadboard, pertanto ho aggiunto uno zoccolo per circuiti integrati.

[![Rocker Switch](../../assets/ram/20-ram-rocker.png "Rocker Switch"){:width="33%"}](../../assets/ram/20-ram-rocker.png)

*Rocker Switch.*
