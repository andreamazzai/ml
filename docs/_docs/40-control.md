---
title: "Control Logic"
permalink: /docs/control/
excerpt: "Control Logic del BEAM computer"
---
## WORK IN PROGRESS

[![Control Logic del BEAM computer](../../assets/control/40-beam-control.png "Control Logic del BEAM computer"){:width="100%"}](../../assets/control/40-beam-control.png)

Questa pagina descrive le Control Logic dell'NQSAP e del BEAM, evidenzia le differenze con la Control Logic del SAP computer di Ben Eater e approfondisce gli argomenti che avevo trovato più ostici e più interessanti.

In generale, la gestione delle istruzioni consta di tre capisaldi: *Instruction Register*, *Ring Counter* e *Microcode*.

[![Schema della Control Logic dell'NQSAP](../../assets/control/40-control-logic-schema-nqsap.png "Schema logico della Control Logic dell'NQSAP"){:width="100%"}](../../assets/control/40-control-logic-schema-nqsap.png)

*Schema della Control Logic dell'NQSAP, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Alcune note propedeutiche alla comprensione di quanto esposto in questa pagina:

1. Nel computer SAP di Ben Eater, la denominazione dei segnali è "modulo-centrica", riflettendo la funzione specifica di ciascun modulo: ad esempio, il segnale RO (RAM Out) esporta il contenuto della RAM sul bus, mentre AI (A Input) carica il registro A. Nel computer NQSAP di Tom Nisbet e nel BEAM, invece, la nomenclatura è "computer-centrica", adottando un punto di vista a livello di bus: per esempio, RO diventa RR (RAM Read) e AI diventa WA (Write A).

2. Nell'NQSAP e nel BEAM l'Instruction Register (IR) è incluso nello schema della Control Logic, mentre negli schemi del SAP stava su un foglio separato.

## Instruction Register e Istruzioni

Il ruolo dell'Instruction Register è di memorizzare l'istruzione corrente prelevandola dalla memoria.

L'Instruction Register del SAP presentava una dimensione di un byte, all'interno del quale erano contenuti sia l'istruzione che l'operando:

- i 4 bit più significativi erano dedicati all'istruzione;
- i 4 bit meno significativi erano riservati a un operando o a un indirizzo opzionali.

Se i bit meno significativi contenevano un operando (ad esempio, un valore immediato da utilizzare in un'operazione aritmetica), questo valore veniva caricato in un registro per l'esecuzione dell'istruzione.

Se i bit meno significativi contenevano un indirizzo di memoria, questo indirizzo veniva caricato nel Memory Address Register (MAR), che puntava così alla posizione di memoria da cui leggere o scrivere dati.

Nell'immagine seguente, tratta dal video <a href="https://youtu.be/JUVt_KYAp-I?t=1837" target="_blank">Reprogramming CPU microcode with an Arduino</a> di Ben Eater, si vede come ogni byte di un semplice programma di somma e sottrazione includa sia l'operazione sia l'operando:

![Somma e sottrazione nel SAP](../../assets/control/40-lda-15-add-14.png "Somma e sottrazione nel SAP"){:width="50%"}

Ad esempio:

- L'istruzione LDA 15 all'indirizzo di memoria 0000 è composta dai 4 bit più significativi (MSB) 0001 (che nel microcode definiscono un'operazione di caricamento accumulatore) e dai 4 bit meno significativi (LSB) 1111, che indicano l'indirizzo di memoria 15, nel quale è presente il valore 5 da caricare nell'accumulatore A.
- L'istruzione ADD 14 all'indirizzo di memoria 0001 è composta dai 4 bit MSB 0010 (che nel microcode definiscono un'operazione di somma) e dai 4 bit LSB 1110, che indicano l'indirizzo di memoria 14, nel quale è presente il valore 6 da sommare al valore già presente nell'accumulatore A.

| Mnemonico | Indirizzo | Istruzione.Operando |
| -         | -         | -                   |
| -         | -         | -                   |
| LDA 15    | 0000      |       0001.1111     |
| ADD 14    | 0001      |       0010.1110     |
| SUB 13    | 0010      |       0011.1101     |
| OUT       | 0011      |       1110.0000     |
| HLT       | 0100      |       1111.0000     |
|           |    .      |                     |
|           |    .      |                     |
|           |    .      |                     |
| 7         | 1101      |       0000.0111     |
| 6         | 1110      |       0000.0110     |
| 5         | 1111      |       0000.0101     |

*Rappresentazione di un programma di somma, sottrazione e output caricato nei 16 byte della memoria del SAP.*

In conseguenza del numero di bit utilizzato per l'istruzione, la connessione tra Instruction Register del SAP ed EEPROM contenenti il microcode poteva avere una ampiezza di soli 4 bit, come visibile in figura:

[![Schema della Control Logic del SAP computer](../../assets/control/40-control-logic-schema-SAP.png "Schema logico della Control Logic del SAP computer"){:width="100%"}](../../assets/control/40-control-logic-schema-SAP.png)

*Schema della Control Logic del SAP computer.*

Una fondamentale differenza tra l'IR del SAP e quello dell'NQSAP e del BEAM è la dimensione. Il 6502 ha un set di istruzioni *relativamente* piccolo, composto da 56 istruzioni di base; tuttavia, queste istruzioni possono essere utilizzate con diverse modalità di indirizzamento, il che porta il numero totale di combinazioni possibili a circa 150.

Per poter gestire queste combinazioni ed emulare così il set di istruzioni del 6502, l'opcode abbisogna di un intero byte (pertanto, l'IR deve dedicare un intero byte all'istruzione, senza condividerlo con l'operando) e l'architettura del sistema deve gestire istruzioni di lunghezza diversa:

- a un solo byte per quelle con indirizzamento Implicito e Accumulatore e che, dunque, non hanno un operando;
- a due o tre\* byte per tutte le altre, che fanno invece uso di un operando (istruzioni a due o tre* byte per definire un indirizzo; istruzioni a due byte per definire un valore assoluto).

\* Un computer con 256 byte di RAM non necessita di istruzioni a 3 byte, perché un operando della lunghezza di un singolo byte è in grado di indirizzare tutta la memoria del computer, come brevemente discusso anche nella sezione [Indirizzamenti](../alu/#indirizzamenti) della pagina dedicata all'ALU.

Tirando le fila, per un computer come l'NQSAP o il BEAM:

- l'Instruction Register deve essere dedicato alle sole istruzioni ed avere dimensione di un byte;
- la connessione tra IR ed EEPROM deve avere un'ampiezza di 8 bit e non più di soli 4 bit come nel SAP;
- sono necessarie EEPROM 28C256 da 256Kb con 15 pin per gli indirizzi:
  - 8 pin di indirizzi per le istruzioni (2^8 = 256 istruzioni)
  - 4 pin di indirizzi per le microistruzioni (2^4 = 16 step), delle quali si parla nella sezione dedicata al [Ring Counter](#ring-counter)
  - 2 pin di indirizzi per selezionare le ROM
  - rimane un pin inutilizzato, tanto da pensare di poter utilizzare EEPROM da 128Kb, che però non esistono in commercio <a href="https://eu.mouser.com/c/semiconductors/memory-ics/eeprom/?interface%20type=Parallel" target="_blank">con l'interfaccia parallela</a>.

**Questo non deve stare qui, credo** Per indirizzare i problemi di glitching Tom ha bufferizzato l'IR, cioè due FF da 8 registri in cascata, così il primo viene aggiornato al normale caricamento dell'IR (che corrisponderebbe a T7 (step 1), ma causando un glitch sulla ROM)… invece di collegare il FF agli ingressi delle ROM, viene collegato a un altro FF che viene caricato col Falling Edge del CLK / Rising Edge del CLK, così le uscite delle ROM vengono aggiornate alla fine della microistruzione quando i segnali sono stabili 😁

**devo spiegare il funzionamento dell'IR**, Riprendendo spunto dal fatto che il registro delle istruzioni questa volta è bufferizzato.

## Ring Counter e Microistruzioni

Per capire il funzionamento del Ring Counter, è necessario fare proprio il concetto di microistruzione: le *istruzioni* di un microprocessore sono composte da un certo numero di step, più precisamente chiamati *microistruzioni*.

Infatti, ogni istruzione di un microprocessore (ad esempio, "carica un valore nel registro X", "incrementa il contenuto della locazione $E5" o "esegui uno scorrimento a destra dell'accumulatore") può essere *scomposta* in una sequenza di microistruzioni elementari, che corrispondono ai singoli passi (step) necessari per completare l'operazione voluta.

Il registro Ring Counter (RC) tiene traccia dello stato di avanzamento delle microistruzioni. Ogni stato dell'RC corrisponde a un particolare step nel ciclo di esecuzione di un'istruzione, quindi può essere visto come un meccanismo che avanza attraverso le diverse microistruzioni necessarie per eseguire un'istruzione completa della CPU.

Nel BEAM, ad esempio, l'istruzione LDA #$94 (che nel linguaggio del 6502 si traduce in "carica nell'accumulatore il valore esadecimale $94") è composta dai seguenti quattro step / microistruzioni:

~~~text
| ---- | ------------------------- |
| Step | Microistruzione           |
| ---- | ------------------------- |
| 1*   | RPC | WM                  |
| 2*   | RR  | WIR | PCI           |
| 3    | RPC | WM                  |
| 4    | RR  | FNZ | WAH | PCI | N |
| ---- | ------------------------- |
~~~

*Scomposizione dell'istruzione LDA nelle sue quattro microistruzioni elementari*.

1. Il primo step espone il contenuto del Program Counter sul bus (RPC, Read Program Counter) e scrive il contenuto del bus nel MAR (WM, Write Memory Address Register).
2. Il secondo step espone sul bus il contenuto della locazione di memoria puntata dal MAR (RR, Read RAM), scrive il contenuto del bus nell'Instruction Register (WIR, Write Instruction Register) e incrementa il Program Counter (Program Counter Increment). Alla fine di questo step, l'IR conterrà l'opcode dell'istruzione e il PC punterà alla locazione di memoria successiva, che nel caso dell'istruzione LDA contiene l'operando.
3. Il terzo step espone di nuovo il contenuto del Program Counter sul bus (RPC, Read Program Counter) e si scrive il contenuto del bus nel MAR (WM, Write Memory Address Register). Ora, il MAR punta alla locazione di memoria che contiene l'operando.
4. Il quarto ed ultimo step espone sul bus l'operando, che è contenuto nella locazione di memoria puntata dal MAR (RR, Read RAM), abilita la scrittura dei Flag N e Z (FNZ), scrive il contenuto del bus su A e H (WAH, Write A + H), incrementa il Program Counter (PCI, Program Counter Increment) e resetta il Ring Counter (N, Next).

Per garantire il corretto funzionamento del processore, la Control Logic deve settare la giusta *Control Word* per ogni microistruzione. La Control Word può essere definita come una stringa di bit utilizzata per governare e coordinare il comportamento dei vari componenti del processore durante l'esecuzione di una microistruzione. Questa stringa di bit è definita nel microcode memorizzato nelle EEPROM; ad ogni bit / pin di output delle EEPROM è associato un segnale di controllo (come RPC, WM, PCI, RR eccetera).

\* E' importante notare che i primi due step sono identici per *tutte* le istruzioni del computer: alla fine di questi due step, l'Instruction Register contiene l'Opcode dell'istruzione, che, insieme alle microistruzioni, definirà il compito di ogni step per ciascuna istruzione. Questo accorgimento garantisce che il computer possa sempre avviarsi correttamente dopo un reset, indipendentemente dall'istruzione presente nella locazione iniziale dopo il caricamento di un programma in memoria.

Uno schema che mostra chiaramente gli step di alcune istruzioni del SAP è visibile in questa immagine tratta dal video <a href="https://www.youtube.com/watch?v=dHWFpkGsxOs" target="_blank">8-bit CPU control logic: Part 3</a> di Ben Eater; notiamo che gli step delle istruzioni LDA, ADD eccetera partono da 010, a significare che gli step 000 e 001 sono invece comuni per tutte e compongono quella che viene chiamata Fase Fetch.

[![Microcode del SAP](../../assets/control/40-cl-ben-step-microcode.png "Microcode del SAP"){:width="100%"}](../../assets/control/40-cl-ben-step-microcode.png)

*Microcode del SAP.*

### Fasi

Le operazioni di una CPU passano per diverse fasi, che possiamo riassumere in:

1. Fase "Fetch" (prelievo), che preleva l'istruzione dalla locazione di memoria puntata dal PC e la memorizza nell'IR.
2. Fase "Decode" (decodifica), che interpreta il contenuto dell'IR per determinare quale istruzione debba essere eseguita.
3. Fase "Execute" (esecuzione), che include tutte le microistruzioni che realizzano effettivamente quanto deve essere svolto dall'istruzione (ad esempio: "incrementa il registro X").

La fase di decodifica avviene grazie al microcodice memorizzato nelle EEPROM: l'istruzione caricata nell'IR ha un proprio opcode specifico (ad esempio, 0100.0110), che viene presentato agli ingressi delle EEPROM insieme agli output del Ring Counter. Questa combinazione indirizza una locazione di memoria specifica nelle EEPROM, la quale genera in uscita i bit della Control Word, attivando i segnali di controllo necessari per eseguire la microistruzione corrente.

Il legame tra decodifica ed esecuzione è molto stretto, perché in ogni momento la Control Word dipende sia dall'opcode (Decode), sia dalla microistruzione (Execute).

Per concludere l'analisi dell'istruzione LDA #$94, riepiloghiamo lo stato del computer alla fine del quarto step:

- il Flag Z non sarà attivo (il risultato dell'operazione di caricamento dell'accumulatore non è uguale a zero);
- il Flag N sarà attivo (secondo il metodo di [rappresentazione dei numeri Signed](../math/#numeri-unsigned-e-numeri-signed) a 8 bit in Complemento a 2, $94 / 1001.0100 è un numero negativo, in quanto il bit più significativo è allo stato logico 1);
- l'accumulatore A e il registro H conterranno il valore $94 esadecimale.

Riprendendo la spiegazione del funzionamento del Ring Counter, risulta ora evidente che in una CPU è necessario conoscere in ogni momento quale sia l'istruzione in esecuzione - ne riceviamo indicazioni dall'Instruction Register - e quale sia lo step correntemente attivo, per conoscere il quale ci viene in aiuto il Ring Counter. Tanto il SAP quanto l'NQSAP e il BEAM sviluppano il Ring Counter attorno a un contatore <a href="https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf" target="_blank">74LS161</a>, in grado di contare da 0 a 15, e a un demultiplexer <a href="https://www.ti.com/lit/ds/symlink/sn74ls138.pdf" target="_blank">74LS138</a>, che ci aiuta ad avere riscontro visivo della microistruzione in esecuzione.

[![Dettaglio del Ring Counter del BEAM](../../assets/control/40-control-logic-161-138-beam.png "Dettaglio del Ring Counter del BEAM"){:width="66%"}](../../assets/control/40-control-logic-161-138-beam.png)

*Dettaglio del Ring Counter del BEAM.*

### Il clock

In generale, i momenti essenziali di un ciclo di clock in un computer sono due: il Rising Edge ↗ (passaggio del segnale dallo stato logico LO allo stato logico HI) e il Falling Edge ↘ (viceversa).

- Rising Edge: la maggior parte dei chip sequenziali* (quali contatori, registri, FF) modifica il proprio stato durante transizione del segnale di clock dallo stato logico LO allo stato logico HI; le azioni di caricamento di tutti i moduli del computer (PC, MAR, RAM, IR, A, B, H, Registri Indice, Flag, SP, O) avvengono in questo momento. Poiché il caricamento dei registri avviene con il Rising Edge del clock, la Control Word deve essere configurata prima di ogni occorrenza di questo evento, quindi durante l'intervallo tra un Rising Edge e il successivo.

- Falling Edge: un momento facilmente identificabile tra un Rising Edge e l'altro del clock è il Falling Edge. E' possibile invertire la fase del clock del Ring Counter, consentendo la configurazione della Control Word - e dunque della microistruzione - proprio in corrispondenza del Falling Edge.

\* I chip sequenziali producono un output che dipende non solo dagli ingressi attuali, ma anche dallo stato logico precedente.

Riassumendo, l'Instruction Register contiene l'Opcode dell'istruzione attualmente in esecuzione, mentre il Ring Counter ne indica lo step attivo. Utilizzando una logica combinatoria, è possibile costruire il microcode da caricare nelle EEPROM, che emetteranno gli opportuni segnali (Control Word) per ogni step di ogni istruzione.

È importante sottolineare che la configurazione delle operazioni di lettura e scrittura da parte della Control Word segue tempi diversi:

- I segnali di lettura attivano immediatamente il modulo interessato, il quale espone subito il suo output sul bus; questo permette ai segnali di stabilizzarsi prima che vengano letti dai registri che dovranno essere aggiornati.
- Viceversa, i segnali di scrittura attivano il modulo interessato, ma l'operazione di scrittura verrà eseguita solo al Rising Edge del clock, assicurando che i registri che devono essere aggiornati ricevano segnali stabili prima dell'effettiva scrittura.

### Durata delle istruzioni

Altro aspetto importante da prendere in considerazione è il numero di microistruzioni che possono comporre ogni istruzione.

Ad esempio nel computer sap gli step delle microistruzioni sono al massimo 6 **verificare bene**

Le istruzioni del computer SAP avevano tutte la stessa durata di **5 step**, indipendentemente dalla loro complessità. Nel microcode che segue possiamo vedere che in realtà l'istruzione di caricamento immediato LDA può essere eseguita in tre step, mentre ad esempio somma e sottrazione sono necessitano di 5 step.

[![Microcode del computer SAP](../../assets/control/40-cl-sap-microcode.png "Microcode del computer SAP"){:width="66%"}](../../assets/control/40-cl-sap-microcode.png)

*Microcode del computer SAP.*

Il contatore del Ring Counter '161 espone un output binario, che può pilotare un DEMUX '138, che può prendere i 3 bit (ce ne bastano 3 per gestire 8 cicli, visto che gli step delle microistruzioni sono al massimo 6) e convertirli in singoli bit che rappresentano lo step della microistruzione corrente e poi uno di questi, l'ultimo, che resetta il 74LS161 in modo da risparmiare i cicli di clock inutilizzati.

Come si vede nello schema dell'NQSAP sub non vi è più un controllo del reset del contatore da parte del 138.

Come indicato anche nella sezione [Differenze](.../alu/#differenze-tra-moduli-alu-dellnqsap-e-del-beam) della pagina dell'ALU, bisogna notare che il computer NQSAP prevedeva solo 8 step per le microistruzioni. Per emulare le istruzioni del 6502 di salto condizionale, di scorrimento / rotazione e di salto a subroutine servono più step, pertanto, sul computer BEAM ne sono stati previsti 16.

come si può vedere nello schema del SAP, il contatore '161 presente le sue uscite agli ingressi di selezione del DEMUX '138, che attiverà in sequenza le uscite invertite (active = LO) da 00 a 05 in sequenza; all'attivazione di quest'ultima, le due NAND attiveranno l'ingresso di Reset /MR del '161, che riporterà il conteggi degli step allo zero iniziale, cominciando così una nuova istruzione.

È facile notare che per le istruzioni più corte questo si produce in uno spreco di cicli di elaborazione.

[![Ring Counter del SAP](../../assets/control/40-control-sap-rc.png "Ring Counter del SAP"){:width="33%"}](../../assets/control/40-control-sap-rc.png)

*Ring Counter del SAP.*

Per indirizzare questa questione e migliorare le prestazioni del computer, all'ultimo step di tutte le istruzioni dell'NQSAP e del BEAM viene aggiunta l' finale "**N**", che su una EEPROM attiva un segnale **NI** che attiva a sua il pin di Load del '161, che, avendo tutti i suoi input a 0, sarà praticamente resettato.

Perché non collegare l'output NI della EEPROM al pin di reset del contatore?

Se si facesse in questo modo, il contatore verrebbe resettato non appena la microistruzione contenente N fosse eseguita, non permettendo l'esecuzione completa di quanto stabilito con la Control Word , pertanto si dovrebbe creare una micro istruzione aggiuntiva per tutte le istruzioni anziché poter avere l'istruzione micron direttamente integrata insieme alle ultime istruzioni micro di ogni istruzione , risparmiando così un ciclo macchino .questo perché il contatore è sincrono virgola e significa che dunque il suo reset avverrebbe in qualsiasi momento , indipendentemente dallo stato del clock.volevo dire asincrono .e allora perché non utilizzare un contatore sincrono ? non lo possiamo fare perchéavremmo problemi con il reset vero e proprio.

In altre parole, possiamo mettere anticipatamente fine ad ogni istruzione inserendo nell'ultimo step un  reset del Ring Counter così da non dover aspettare l'esecuzine di eventuali step vuoti (alla data, la lunghezza massima delle istruzioni è di 10 step).

Spiegare il legame con il funzionamento del 161, che permette di caricare zero sui suoi ingressi, ed è il metodo che viene utilizzato per resettare il ring counter e riportarlo allo step t zero.

Praticamente usando il '161:
- con /CLR, che è asincrono, faccio il reset "hardware"
  - 17/06/2023 Tom segnala che questo segnale asincrono non è invece ideale per pulire il Ring Counter utilizzando il microcode perché, essendo appunto asincrono, non sarebbe gestito dal clock: infatti, non appena attivato, andrebbe a resettare il ciclo di microistruzione esattamente all'inizio invece che quando arriva il impulso di clock!
  - 04/07/2023 Va invece benissimo per fare il reset del Program Counter 😁 anche col clock stoppato
  - A dire il vero si potrebbe comunque utilizzare questo segnale, ma significherebbe dover aggiungere una microistruzione dedicata al reset alla fine di tutte le altre microistruzioni. Utilizzando una modalità di reset sincrona su questo chip si potrebbe invece aggiungere il segnale di reset all'ultima microistruzione del ciclo.
- con il /LOAD ("N") e tutti i pin di input a 0, il Ring Counter si resetta in sincrono con il /CLK 😁. Assomiglia un po' al JUMP del Program Counter. Notare il /CLK, che è invertito rispetto al CLK principale e che dunque permette di lasciar terminare l'esecuzione della microistruzione corrente prima di fare il LOAD.

3 uscite del 161 vanno al 138 per pilotare 8 led (2^3 = 8); per il 9° LED Tom è il solito furbo: invece di mettere due 138 per pilotare 16 led, aggiunge un led al quarto pin del 161, così ad esempio 11 = 3 + 8 e dunque si accendono il 3* led di 8 pilotato dal 138 e quello dell'"Extended Time" pilotato dal 161

	• Gli ingressi D0-D3 sono tutti a 0, così quando arriva il /LOAD (o /PE) sincrono (segnale "N" per Tom), il conteggio ricomincia da zero
	• Le uscite Q0-Q2 del 161 vanno a MA0-MA2 per avere 8 step di microistruzioni, ma se aggiungo un quarto pin al contatore, posso avere 16 step
	• Il /LOAD arriva da N 17 della ROM0
CEP e CET sono a +Vcc

### I 74LS138 per la gestione dei segnali

La complessità dell'NQSAP è tale per cui i soli 16 segnali disponibili nella Control Logic del SAP non sarebbero stati sufficienti per pilotare moduli complessi come ad esempio l'ALU e il registro dei Flag; in conseguenza di questo, diventava necessario ampliare in maniera considerevole il numero di linee di controllo utilizzabili.

L'aumento del numero di EEPROM e l'inserimento di quattro 3-Line To 8-Line Decoders/Demultiplexers <a href="https://www.ti.com/lit/ds/symlink/sn74ls138.pdf" target="_blank">74LS138</a> (DEMUX) ha permesso di gestire l'elevato numero di segnali richiesti dall'NQSAP.

Come visibile nello schema, ogni '138 presenta 8 pin di output, 3 pin di selezione e 3 pin di Enable; connettendo opportunamente i pin di selezione ed Enable, è possibile pilotare ben quattro '138 (per un totale di 32 segnali di output) usando solo 8 segnali in uscita da una singola EEPROM. In altre parole, i '138 fungono da *demoltiplicatori* e permettono di indirizzare un numero elevato di segnali a partire da un numero limitato di linee in ingresso.

Quando attive, le uscite dei '138 presentano uno stato LO; questa circostanza risulta molto comoda per la gestione dei segnali del computer, in quanto molti dei chip presenti nei vari moduli utilizzano ingressi di Enable invertiti (ad esempio i transceiver 74LS245 e i registri 74LS377).

I '138 presentano un solo output attivo alla volta; la configurazione dei pin di selezione ed Enable adottata nello schema permette di creare due coppie di '138, ognuna delle quali presenta un solo output attivo alla volta:

- una coppia dedicata ai segnali di lettura dai registri;
- una coppia dedicata ai segnali di scrittura sui registri.

Un effetto collaterale positivo in questo tipo di gestione sta nel fatto che risulterà impossibile attivare più di un registro Read contemporaneamente, evitando così possibili involontari cortocircuiti tra uscite allo stato HI e uscite allo stato LO di moduli diversi.

Il ragionamento per le scritture è diverso, perché è invece realmente necessario essere in grado di scrivere su più registri contemporaneamente. Un operazione di questo tipo non causa contenzioso sul bus ed è utilizzata, ad esempio, dall'istruzione di somma ADC, che prevede uno step che scrive contemporaneamente sul registro A e sul registro dei Flag.

Nello schema si può notare che tutti i registri del computer che *non hanno tra loro* la necessità di essere attivi contemporaneamente - tanto in lettura quanto in scrittura - sono indirizzati con i DEMUX.

Sono invece indispensabili segnali di controllo provenienti direttamente dalle EEPROM in tre casi:

- quando un registro presenta più segnali di ingresso che possono essere attivi contemporaneamente (ad esempio il registro dei [Flag](../flags/#componenti-e-funzionamento), oppure il registro [H](../alu/#il-registro-h));
- quando è necessario poter scrivere su più registri contemporaneamente (ad esempio A e H, oppure Flag e A, oppure Flag e H*);
- quando occorrono altri segnali di controllo totalmente indipendenti (ad esempio per lo Stack, oppure per la gestione del [Carry Input](../flags/#il-carry-e-i-registri-h-e-alu) per ALU ed H).

\* In questo secondo caso, i segnali provenienti direttamente dalle EEPROM devono essere utilizzati per attivare i registri che *hanno* la necessità di poter essere attivi in contemporanea con un altro registro connesso ai '138 e che è dunque, per natura, mutualmente esclusivo rispetto agli altri registri pilotati dai '138.

Riassumendo:

- una prima EEPROM gestisce quattro DEMUX che pilotano i segnali di *lettura* di tutti i registri, i segnali di *scrittura* di tutti i registri (eccetto H e Flag) ed alcuni altri segnali di controllo;
- altre tre EEPROM gestiscono tutti gli altri segnali.

Notare che i segnali di uscita dei '138 realmente utilizzabili sono 30 e non 32, perché il microcode deve prevedere casi nei quali nessuno dei registri pilotati dai '138 debba essere attivo; in questa circostanza, uno dei pin di output di ogni coppia di '138 dovrà essere scollegato. Ad esempio, nel caso dell'NQSAP, un output 0000.0000 della prima EEPROM attiverà i pin D0 del primo '138 e del terzo '138: entrambi i pin sono scollegati, dunque sarà sufficiente mettere in output 0x00 sulla prima EEPROM per non attivare alcuno tra tutti i registri gestiti dai '138.

## Tabella segnali dell'NQSAP e del BEAM

È forse utile fare qui una tabella che riassume tutti i vari segnali utilizzati nel computer ?

| NQSAP  | BEAM     | Descrizione                                                                              |
| ------ | -------- | -----------                                                                              |
| C0-C1  | C0-C1    | Utilizzati per determinare se il Carry da salvare nel Flag Register debba provenire dal Carry Output dell'ALU (operazioni aritmetice) o da H (operazioni di scorrimento e rotazione) |
| CC-CS  | CC-CS    | Utilizzati per selezionare quale Carry presentare all'ALU e ad H (quello effettivamente presente in Flag C, oppure 0 o 1 fisse); [vedere spiegazione](../flags/#il-carry-e-i-registri-h-e-alu) |
| DY-DZ  | DX/Y-DZ  | DX/Y HI espone X, LO espone Y agli adder; DZ non espone X e Y agli adder                 |
| HL-HR  | HL-HR    | Definiscono l'operazione da eseguire sul registro H (caricamento parallelo, rotazione a destra o sinistra)      |
| FN     | FN       | Caricamento del Flag N nel registro dei flag                                             |
| FV     | FV       | Caricamento del Flag V nel registro dei flag                                             |
| FZ     | FZ       | Caricamento del Flag Z nel registro dei flag                                             |
| FC     | FC       | Caricamento del Flag C nel registro dei flag                                             |
| FS     | FB       | Selezione dell'origine dei Flag da caricare nel registro dei Flag (computo oppure RAM)   |
| SCE*   | SE       | Abilita la scrittura dello Stack Pointer                                                 |
| SPI*   | SU/D     | Indica se lo Stack Pointer deve contare verso l'alto (stato HI) o verso il basso (stato LO)           |
| EO     | Sum Out  | L'adder computa A+B e il suo risultato viene esposto sul bus                             |
| LF     | LF       |                                                                                          |
| WI     | WIR      | Abilita la scrittura dell'Instruction Register                                           |
| N      | NI       | Next Instruction                                                                         |
| JE     | PCJ      | Carica il Program Counter                                                                |
| PI     | PCI      | Incrementa il Program Counter                                                            |

| RR     | PCI      | Incrementa il Program Counter                                                            |
| RA     | PCI      | Incrementa il Program Counter                                                            |
| RB     | PCI      | Incrementa il Program Counter                                                            |
| RL     | PCI      | Incrementa il Program Counter                                                            |
| RS     | PCI      | Incrementa il Program Counter                                                            |
| RP     | PCI      | Incrementa il Program Counter                                                            |
| RD     | PCI      | Incrementa il Program Counter                                                            |
| RX     | PCI      | Incrementa il Program Counter                                                            |
| RY     | PCI      | Incrementa il Program Counter                                                            |
| RH     | PCI      | Incrementa il Program Counter                                                            |
| RF     | PCI      | Incrementa il Program Counter                                                            |

| WR     | PCI      | Incrementa il Program Counter                                                            |
| WA     | PCI      | Incrementa il Program Counter                                                            |
| WB     | PCI      | Incrementa il Program Counter                                                            |
| WS     | PCI      | Incrementa il Program Counter                                                            |
| WP     | PCI      | Incrementa il Program Counter                                                            |
| WD     | PCI      | Incrementa il Program Counter                                                            |
| WM     | PCI      | Incrementa il Program Counter                                                            |
| WX     | PCI      | Incrementa il Program Counter                                                            |
| WY     | PCI      | Incrementa il Program Counter                                                            |
| WO     | O        | Incrementa il Program Counter                                                            |

\* Deduzione

	• C0 e C1 *** 
sono condivisi con C0 e C1 ***

SE Stack Enable, vedi pagina Stack Pointer, condivisi con C0 e C1*** (chiamiamoli SU Stack Up e SD Stack Down) per definire se fare conteggio Up o Down

• Se /LDR-ACTIVE viene attivato (LO), LDR-ACTIVE passa a HI e disattiva le ROM2 e ROM3 collegate via /OE.

## Forse interessante da tenere, espandere, collegare ad altri paragrafi

La realizzazione del comuter SAP mi ha permesso finalmente di capire cosa sia il microcode di un computer moderno.

È piuttosto comune leggere ad esempio che è necessario aggiornare il bios dei server per indirizzare falle di sicurezza che sono state scoperte e che potrebbero essere utilizzate dagli hacker per ... 

Non capendo come potesse essere aggiornata una CPU, dal momento che si tratta di un componente non programmabile , non riuscivo a comprendere come fosse possibile arginare i problemi di sicurezza; con il microcode ho capito.

## Fare spiega su EEPROM input e output

Stavo anche iniziando a pensare come avrebbe funzionato la fase di Fetch con un Instruction Register che conteneva la sola istruzione e non operando istruzione + operando, come nel computer SAP**.

Immaginavo che una istruzione di somma tra l'Accumulatore e il valore contenuto in una cella di memoria specifica avrebbe avuto questa sequenza:

| Step | Segnale   | Operazione |
| ---- | ------    | ----------- |
|    0 | CO-MI     | Carico l'indirizzo dell'istruzione nel MAR |
|    1 | RO-II-CE  | Leggo l'istruzione dalla RAM e la metto nell'Instruction Register; incremento il Program Counter che punta così alla locazione che contiene l'operando |
|    2 | CO-MI     | Metto nel MAR l'indirizzo della cella che contiene l'operando |
|    3 | RO-MI     | Leggo dalla RAM l'operando, che rappresenta l'indirizzo della locazione che contiene il valore che desidero addizionare all'accumulatore |
|    4 | RO-BI-CE  | Leggo dalla RAM il valore contenuto nella locazione selezionata e lo posiziono nel registro B; incremento il Program Counter che punta così alla locazione che contiene la prossima istruzione |
|    5 | EO-AI     | Metto in A il valore della somma A + B |

Legenda dei segnali:

| Segnale | Operazione              | Descrizione                                                       |
| ------  | ----------              | -----------                                                       |
| CO      | Counter Output          | Il contenuto del Program Counter viene esposto sul bus            |
| MI      | MAR In                  | Il contenuto del bus viene caricato nel Memory Address Register   |
| RO      | RAM Output              | Il contenuto della RAM viene esposto sul bus                      |
| II      | Instruction Register In | Il contenuto del bus viene caricato nell'Instruction Register     |
| CE      | Counter Enable          | Il Program Counter viene incrementato                             |
| BI      | B Register In           | Il contenuto del bus viene caricato nel registro B                |
| AI      | A Register In           | Il contenuto del bus viene caricato nel registro A                |
| EO      | Sum Out                 | L'adder computa A+B e il suo risultato viene esposto sul bus      |

---
MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE

• Nota che  livello generale ha definito due fasi di Fetch F1 ed F2 che sono comuni a tutte le istruzioni e sono sempre ripetute.
	• NQSAP: 
		○ 8 step compresi due di Fetch
		○ #define F1    RP | WM       // Instruction fetch step 1
		○ #define F2    RR | WI |PI   // Instruction fetch step 2
	• NQSAP-PCB:
		○ 16 step compresi due di Fetch
		○ #define F1    RP | WM | PI  // Instruction fetch step 1: instruction address from PC to MAR
#define F2    RR | WI       // Instruction fetch step 2: instruction from RAM to IR




Per fare il microcode sto usando:
• https://www.masswerk.at/6502/6502_instruction_set.html
• https://www.masswerk.at/6502/ che è utile per simulare le istruzioni e capire quali Flag esse modifichino durante la loro esecuzione.
• Ad esempio inizialmente ho trovato difficoltà a capire quali Flag fossero modificati da CPY, che viene indicata come:

• In quali combinazioni si attivano i vari flag N, Z e C?
• Ho trovato supporto su http://www.6502.org/tutorials/compare_beyond.html nel quale si spiega che fare un confronto equivale a settare il carry e fare la differenza, ma senza effettivamente scrivere il risultato nel registro di partenza:
CMP NUM
		is very similar to:
SEC
SBC NUM

	        • If the Z flag is 0, then A <> NUM and BNE will branch
	        • If the Z flag is 1, then A = NUM and BEQ will branch
	        • If the C flag is 0, then A (unsigned) < NUM (unsigned) and BCC will branch
	        • If the C flag is 1, then A (unsigned) >= NUM (unsigned) and BCS will branch
	
	Facciamo le prove:
	Codice:
	        LDY #$40
	        CPY #$30
	Viene attivato il C, coerentemente con quanto spiegato sopra… direi perché nell'equivalenza si fa il SEC prima di SBC; essendo il numero da comparare inferiore, non faccio "il prestito" (borrow) del Carry e dunque alla fine dell'istruzione me lo ritrovo attivo come in partenza.
	
	
	Codice:
	        LDY #$40
	        CPY #$40
	Vengono attivati sia Z sia C: Z perché 40 - 40 = 0 e dunque il risultato è Zero e il contenuto del registro e del confronto numeri sono uguali; essendo il numero da comparare inferiore, non faccio "il prestito" (borrow) del Carry.
	
	
	
	Codice:
	        LDY #$40
	        CPY #$50
	No Z e C, coerentemente con quanto spiegato sopra, ma N, perché il numero risultante è negativo: in 2C il primo bit è 1 ☺️. C è diventato Zero perché l'ho "preso in prestito".
	
	
	
	
	Su BEAM: LDY #$40; CPY #$30 e ottengo nessun Flag, mentre dovrei avere C.
	 La ALU presenta il COUT acceso, dunque la sua uscita è a livello logico basso. DA CAPIRE!!! Cosa volevo dire?
	
	Teoricamente dunque dovrei attivare l’ingresso di uno del 151 di Carry Input settando opportunamente i segnali C0 e C1.
	
	In conseguenza di questo, verifico sul BEAM il comportamento del Carry Out dell'ALU nei 3 casi descritti e poi modifico il microcode di conseguenza. In effetti, il comportamento non era quello desiderato da teoria e ho fatto le modifiche necessarie:
	
	        • Aggiunti i segnali C0 e C1, che non avevo ancora cablato, che permettono al 151 di scelta del Carry Input di selezionare cosa prendere in ingresso. L'ALU emette un Carry invertito (0 = Attivo), dunque, per poter settare a 1 il Flag del Carry Input, lo devo prendere in ingresso dall'ALU attraverso una NOT su uno dei 4 ingressi attivi del 151, che seleziono appunto con i segnali C0 e C1 attivando il solo C0.
	        • Ho poi incluso nel microcode anche LF, in quanto ho definito l'utilizzo di LF su tutte le istruzioni di comparazione, tranne CPX abs.
	        • Considerare anche che tipo di Carry devo iniettare nella ALU… In realtà, poiché per fare il confronto utilizzo l’istruzione SBC, devo utilizzare il normale LHHL con Carry, cioè CIN = LO, che nel microcode corrisponde ad attivare il segnale CS.
	
	Ho posizionato in uscita sul Carry dell'ALU un LED (ricordare che l'uscita è negata, dunque anodo a Vcc e catodo verso il pin del chip). Anche l’ingresso Carry è negato e dunque attivo a zero, pertanto anche qui ho un LED con anodo a Vcc e catodo sul Pin.
	
	Dopo queste modifiche, le istruzioni di comparazione sembrano funzionare correttamente.
	
	TO DO: finire http://www.6502.org/tutorials/compare_beyond.html da "In fact, many 6502 assemblers will allow BLT (Branch on Less Than) "
	
	        • Vedere bene quali istruzioni CP* hanno bisogno di LF, anche sul file XLS
	
Altre referenze Tom Nisbet per Flags	• Question for all 74ls181 alu people on reddit led to the design of the oVerflow flag.
	• How to add a decremental and incremental circuit to the ALU ? on reddit inspired the idea to drive the PC load line from the flags instead of running the flags through the microcode.
	• Opcodes and Flag decoding circuit on reddit has a different approach to conditional jumps using hardware. Instead of driving the LOAD line of the PC, the circuit sits between the Instruction Register and the ROM and conditionally jams a NOP or JMP instruction to the microcode depending on the state of the flags. One interesting part of the design is that the opcodes of the jump instructions are arranged so that the flag of interest can be determined by bits from the IR. NQSAP already did something similar with the ALU select lines, so the concept was used again for the conditional jump select lines.

[![Schema della control logic del BEAM](../../assets/control/40-control-logic-schema.png "Schema della control logic del BEAM"){:width="100%"}](../../assets/control/40-control-logic-schema.png)

*Schema della control logic del BEAM.*

## Differenze tra Moduli Flag dell'NQSAP e del BEAM

La Control Logic del computer BEAM riprende tutto ciò che è stato sviluppato da Tom Nisbet nell'NQSAP. Una differenza sostanziale sta nell'Instruction Register, che è sviluppato in modalità bufferizzata, come fatto da Tom nell'NQSAP PCB per rimediare ai problemi di Glitching.

[![Schema logico del modulo Flag del computer BEAM](../../assets/flags/30-flag-beam-schematics.png "Schema logico del modulo Flag del computer BEAM"){:width="100%"}](../../assets/flags/30-flag-beam-schematics.png)

*Schema logico del modulo Flag del computer BEAM.*

- Le istruzioni di Branch Relative consumano molti cicli, perciò Tom ha aggiunto anche delle istruzioni di Jump Relative. Evidenziare che ho risolto questa problematica facendo un instruction register a 16 step

- Come già discusso, il BEAM prevede 16 step per le microistruzioni anziché solo 8. Anche l'NQSAP-PCB, evoluzione dell'NQSAP, prevede 16 step.

## Note

- Attenzione : nello schema cè una led bar collegata al ring counter, Una led bar collegata alle uscite a - quattro a 11del bass delle rom, ma probabilmente qui manca un pezzettino di led bar per arrivare ai 12 indirizzi totaliindirizzatidai 12 pine in più manca la led bar del registro delle istruzioni
- Il computer NQSAP prevedeva 8 step per le microistruzioni, mentre il BEAM ne prevede 16. Come descritto in maggior dettaglio nella sezione riservate al microcode, con soli 8 step non sarebbe stato possibile emulare alcune delle istruzioni del 6502, come quelle di salto relativo ed altre. Questa è in realtà una differenza architetturale più legata alla Control Logic, però l’impatto principale sul numero di step disponibili si riflette in particolar modo sull’ALU ed ha dunque sicuramente senso citarla in questa sezione.

## Link utili

- I video di Ben Eater che descrivono la <a href="https://eater.net/8bit/control" target="_blank">Control Logic e il Microcode</a>.

## TO DO

- aggiornare lo schema Kicad con le le bar a 8 segmenti e aggiornare questa pagina con lo schema aggiornato
- Evidenziare la nomenclatura dei segnali da fare nella pagina della control logic : l'approccio di ben era centri con rispetto al modulo , mentre l'approccio del computer NQSAP è relativo al computer nella sua interezza
- non trovo riferimenti ad HL e HR in nessuna pagina; Poiché in questa pagina sto parlando del fatto che per alcuni registri sono necessari più segnali di controllo , come nel caso del registro h virgola che necessita di HLanche di HR volevo fare un link al registro h nella pagina del modulo ALU , ma vedo che anche lì non cè nessuna indicazione di HL anche di HR ("quando un registro presenta più segnali di ingresso che possono essere attivi contemporaneamente (ad esempio il registro dei Flag, oppure il registro H");)
- Ho aggiunto anche una barra a led per mostrare l'indirizzo correntemente In input sulle EEPROM
- verificare quando spiegare cosa fa il 138: se metto prima Ring Counter o la speigazione delle uscite dei 4x 138 della prima ROM
