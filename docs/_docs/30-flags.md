---
title: "Flag"
permalink: /docs/flags/
excerpt: "Costruzione del registro dei flag del BEAM computer"
---

## Tassonomia: Flag per parlare di registro o modulo, flag per parlare del singolo flag

[![Registro dei Flag del BEAM](../../assets/flags/30-flag-beam.png "Registro dei Flag del BEAM"){:width="100%"}](../../assets/flags/30-flag-beam.png)

Bisogna dire che più volte, leggendo la documentazione di Tom Nisbet, avevo trovato delle idee molto clever; alcune di queste si trovano nel modulo dei Flag. Cercherò di riportare anche alcune fonti utilizzate da Tom indicandone i collegamenti a fondo pagina.

[![Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.](../../assets/flags/30-flag-nqsap-schematics.png "Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità."){:width="100%"}](../../assets/flags/30-flag-nqsap-schematics.png)

*Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Il registro dei Flag dell'NQSAP emula i 4 flag **NVZC** del 6502:

- **N**egative (N)
- O**V**erflow (V)
- **Z**ero (Z)
- **C**arry (C)

E' completamente differente dal semplice registro dei Flag del computer SAP di Ben Eater, nel quale un unico Flip-Flop [74LS173](https://www.ti.com/lit/ds/sdls067a/sdls067a.pdf) memorizzava i soli 2 flag C e Z nello stesso momento: la gestione delle istruzioni necessitava di 4 set di microcode, cioè uno per ogni combinazione dei segnali di flag portati agli ingressi delle EEPROM; ogni set di microcode era infatti sviluppato "su misura" per attivare in output i corretti segnali per la gestione di C e/o Z. Questo è ben spiegato nel video di Ben Eater [Conditional jump instructions](https://www.youtube.com/watch?v=Zg1NdPKoosU).

L'approccio di Tom era invece basato su una verifica logica eseguita in hardware: il microcode non variava a seconda dello stato dei flag, che non erano più direttamente connessi agli indirizzi delle ROM che attivano poi a loro volta diversi segnali di output in base all'indirizzo/flag presentato in ingresso!

Analizzando ad esempio un'istruzione di salto condizionale legata al flag Z riscontravo che:

- il microcode dell'istruzione di salto attivava un generico segnale "Jump Enable" connesso al pin 7 del Data Selector/Multiplexer [74LS151](https://www.ti.com/lit/ds/symlink/sn54s151.pdf) visibile in basso a destra nello schema generale;

- la selezione del flag da mettere in uscita sul '151 dipendeva dalla codifica dell'istruzione in esecuzione, poiché i 3 bit Select S2, S1 ed S0 erano direttamente collegati all'Instruction Register, cioè *hardwired*, in maniera similare a quanto realizzato anche nel modulo ALU;

 [![Output dell'Instruction Register verso il modulo Flag con evidenza dei 3 bit di selezione dell'istruzione di salto condizionale](../../assets/flags/30-flag-cl-ir-out.png "Output dell'Instruction Register verso il modulo Flag con evidenza dei 3 bit di selezione dell'istruzione di salto condizionale"){:width="50%"}](../../assets/flags/30-flag-cl-ir-out.png)

*Output dell'Instruction Register verso il modulo Flag con evidenza dei 3 bit di selezione dell'istruzione di salto condizionale.*

[![Ingressi di selezione dell'istruzione di salto condizionale del registro dei Flag e connessione "hardwired" con l'IR](../../assets/flags/30-flag-select-in.png "Ingressi di selezione dell'istruzione di salto condizionale del registro dei Flag e connessione "hardwired" con l'IR"){:width="50%"}](../../assets/flags/30-flag-select-in.png)

*Ingressi di selezione dell'istruzione di salto condizionale del registro dei Flag e connessione "hardwired" con l'IR.*

Se ad esempio una generica istruzione *Jump on Zero* fosse codificata come 010 sui 3 segnali S2, S1 ed S0 comuni tra Instruction Register (IR) e registro dei Flag, si verificherebbe la seguente sequenza:

- il pin I2 di ingresso del '151 sarebbe attivato;
- in presenza del flag Z, l'uscita Q del Flip-Flop connessa al pin I2 avrebbe valore logico HI;
- l'uscita del '151 andrebbe ad abilitare il segnale /PC-LOAD sul Program Counter (**PC**).

La pagina dei registri [D, X e Y](../dxy) descrive il calcolo del valore da inserire nel PC.

![Tabella funzioni Selector/Multiplexer 74LS151 con evidenza della ipotetica istruzione Jump on Zero](../../assets/flags/30-flag-151-table.png){:width="33%"}

*Tabella funzioni Selector/Multiplexer 74LS151 con evidenza della ipotetica istruzione Jump on Zero.*

Detto in altre parole: la logica del salto condizionale del SAP era implementata nel microcode, utilizzando linee di indirizzamento delle ROM. Poiché i flag dell'NQSAP erano invece implementati in hardware, non c'era bisogno di consumare preziose linee di indirizzamento delle EEPROM (e di raddoppiare la dimensione del microcode ad ogni aggiunta di un flag).

I miglioramenti derivanti da questa architettura sono:

- possibilità di settare i flag anche singolarmente;
- risparmio di linee di indirizzamento delle EEPROM;
- l'output delle EEPROM non si modifica durante l'esecuzione della singola istruzione (**ma nel SAP-1 come si comportava? 04/10/2022 l'ho compreso andando a rileggere gli appunti** del BE 8 bit computer). Teoricamente, e l'avevo letto anche altrove, questo potrebbe essere un problema perché causa "glitching".

## Componenti e funzionamento

Un multiplexer (MUX) [74LS157](https://www.ti.com/lit/ds/symlink/sn74ls157.pdf) prende in input i valori dei flag V, Z e C selezionandone la provenienza:

1. **dal bus**; quando il '157 legge dal bus, è possibile caricare i registri dei flag leggendo valori arbitrari dalla memoria del computer (o, più precisamente, dalla zona di memoria adibita allo Stack) similarmente a quanto svolto dall'istruzione Pull Processor Status **PLP** del 6502;

2. **da un computo**:
    - **V** attraverso un Data Selector/Multiplexer '151 che ricrea la funzione logica dell'Overflow verificando un eventuale cambio di segno nel risultato delle operazioni di somma o sottrazione dei numeri con segno (Signed); **questo aspetto verrà ulteriormente evidenziato** nella sezione apposita dedicato alla comprensione dell'Overflow.
    - **Z** come risultato del comparatore [74LS688](https://www.ti.com/lit/ds/symlink/sn74ls688.pdf);
    - **C** attraverso un altro '151 che seleziona la sorgente del Carry;

V, Z e C escono dal MUX '157 e sono presentati a 3 dei 4 Flip-Flop disponibili in una coppia di [74LS74](https://www.ti.com/lit/ds/symlink/sn54ls74a.pdf).

Il flag **N**egative viene letto direttamente dalla linea D7 del bus e caricato sul 4° Flip-Flop.

Quattro porte AND permettono il caricamento dei FF in presenza del segnale di clock e della contemporanea attivazione degli opportuni segnali F**N**, F**V**, F**Z** ed F**C** provenienti dalla Control Logic (il caricamento dei registri viene sempre effettuato durante il Rising Edge del Clock). Ogni istruzione del computer, grazie alla personalizzazione del microcode, può settare anche più di un flag alla volta (come accade ad esempio per le operazioni ADC e SBC, che sul 6502 influiscono contemporaneamente su tutti i 4 flag **NVZC**).

![Caricamento dei Flip-Flop C e Z con segnali Clock AND FC / FZ](../../assets/flags/30-flag-ff-cz.png){:width="50%"}

*Caricamento dei Flip-Flop C e Z con segnali Clock AND FC / FZ.*

- Si noti che i FF non vengono mai pre-settati, pertanto /Preset resta fisso a Vcc (e dunque mai attivo); è presente invece una connessione al segnale di reset generale del computer (/RST).

- Le uscite dei FF sono connesse a un '151 per gestire i salti condizionali.

- Il flag C viene esportato anche verso i '181 e il registro H del modulo ALU; si veda la sezione [Il Carry e i registri H e ALU](#il-carry-e-i-registri-h-e-alu) in questa stessa pagina.

Un transceiver 74LS245 permette infine di esportare i 4 flag NVZC sul bus per salvarli in memoria, o più precisamente nello Stack, similarmente a quanto accade nel 6502 con l'istruzione Push Processor Status (PHP).

Interessante notare che le istruzioni CLC, CLV e SEC non hanno bisogno di segnali dedicati della CL per azzerare o settare i flag C e V: si utilizza la ALU per mettere 0x00 o 0xFF sul bus e si modifica il solo flag di interesse attivando opportunamente il segnale di controllo FC o FV.

## I salti condizionali

Ogni variazione di un flag nel computer SAP di Ben Eater generava una variazione degli indirizzi delle EEPROM, così da poter presentare una logica opportunamente diversa in uscita in conseguenza delle diverse combinazioni degli stati dei flag.

L'approccio dell'NQSAP è molto diverso, in quanto tutti i segnali dei flag (presenti sulle uscite Q e /Q dei FF '74) vengono presentati a un '151; la sua funzione è quella di selezionare uno dei flag da mettere sulla sua uscita per eventualmente permettere l'attivazione del segnale /PC-LOAD, che abilita il caricamento del contenuto del bus sul Program Counter (e dunque i salti). E' sufficiente infatti caricare un nuovo valore nel Program Counter (PC) perché questo diventi il nuovo indirizzo a partire dal quale saranno caricate ed eseguite (fetch/execute) le successive istruzioni del programma caricato nella memoria del computer.  

Come avviene la selezione del flag da portare all'uscita Z del '151? I segnali IR-Q5 (S0), IR-Q6 (S1) ed IR-Q7 (S2) provenienti dall'IR selezionano quale tra gli input del '151 (I0-I7) si debba portare sull'output Z, come anticipato precedentemente nella *Tabella funzioni Selector/Multiplexer 74LS151*. IR-Q5, Q6 e Q7 sono infatti *hardwired* con l'IR: le istruzioni di branch, ognuna con la sua codifica specifica, determinano quale ingresso Ix del '151 sarà attivato ed esposto in output; a seconda dello stato del flag, /PC-LOAD sarà eventualmente attivo e l'istruzione di salto relativo potrà essere eseguita.

Prendiamo come esempio l'istruzione BCS (Branch on Carry Set) ipotizzando che l'istruzione precedente abbia generato un Carry e che dunque il corrispondente FF presenti lo stato logico HI sull'uscita Q:

[![Esempio istruzione Branch on Carry Set](../../assets/flags/30-flag-bcs.png "Esempio istruzione Branch on Carry Set"){:width="50%"}](../../assets/flags/30-flag-bcs.png)

*Esempio istruzione Branch on Carry Set.*

- l'istruzione sarà codificata nel microcode in modo che S0, S1 ed S2 selezionino l'ingresso I0, che è collegato al FF del Carry:

  - S0 = LO
  - S1 = LO
  - S2 = LO

- poiché il Carry è attivo, l'output della NOR connessa all'uscita Z è certamente LO (/PC-LOAD = NOT (1+x) = 0), pertanto il valore presente nel bus viene caricato nel PC (il segnale /PC-LOAD è attivo LO).

Un aspetto che inizialmente sfuggiva alla mia comprensione era come poter includere le istruzioni di branch (8 combinazioni = 3 bit), le istruzioni dell'ALU (5 bit) e tutte le altre istruzioni (ad esempio caricamento, trasferimento) in soli 8 bit: come era possibile gestire tutte le combinazioni e costruire una matrice di istruzioni? Nel momento un cui eseguivo una qualsiasi altra istruzione, cosa sarebbe successo nella gestione dei salti relativi, visto che erano direttamente funzione della codifica dell'istruzione correntemente in esecuzione? Non rischiavo di eseguire un salto condizionale non voluto se mi fossi trovato ad esempio in una situazione in cui (come descritto poco sopra) IR-Q5, Q6 e Q7 fossero LO e il Carry fosse attivo?

In seguito avevo notato che nel '151 addetto alla selezione dei flag era presente anche un segnale di Enable JE, presente nel microcode delle sole istruzioni di salto condizionale, pertanto:

- se JE è HI **e** se l'output del FF selezionato è HI, l'uscita del '151 sarà essa stessa HI, permettendo il caricamento del Program Counter;

- se JE è LO, l'uscita del '151 sarà disattivata, pertanto nessun segnale di caricamento verrà inviato al Program Counter.

Dunque le istruzioni - e solo quelle - il cui microcode comprendeva il segnale JE erano considerate istruzioni di salto condizionale; tutte le altre ignoravano il salto, in quanto il segnale /PC-LOAD non sarebbe mai stato attivato.

Tom evidenziava che "*questo metodo semplifica il microcode, perché tutte le operazioni di salto utilizzeranno lo stesso microcode*".

Perché tutte le istruzioni di salto dovrebbero essere "uguali"? La spiegazione, semplice solo dopo averla ben compresa, sta nel fatto che la scelta del flag da utilizzare per il salto condizionale non dipende più dal microcode dell'istruzione, bensì dalla codifica dell'istruzione stessa: essendo *hardwired* con l'IR, i segnali IR-Q5, Q6 e Q7 vengono automaticamente applicati agli ingressi Select del '151; è nella realizzazione del set di istruzioni che si deve tenere conto di quale codifica associare alle varie istruzioni BCS, BCC, BNE, BPL eccetera. Nessuna variazione è richiesta nel microcode.

Prendiamo come ulteriore esempio l'istruzione BVC (Branch on OVerflow Clear) ipotizzando che non ci sia Overflow e che dunque il segnale /V sia attivo:

[![Esempio istruzione Branch on OVerflow Clear](../../assets/flags/30-flag-bvc.png "Esempio istruzione Branch on OVerflow Clear"){:width="50%"}](../../assets/flags/30-flag-bvc.png)

*Esempio istruzione Branch on OVerflow Clear.*

- l'istruzione dovrà essere codificata nel microcode in modo che S0, S1 ed S2 selezionino l'ingresso I5, che è collegato all'uscita /Q del FF dell'Overflow:

  - S0 = HI
  - S1 = LO
  - S2 = HI

Tenendo ora in considerazione l'esistenza del segnale Jump Enable, evidenziato in giallo nello schema precedente, si evince che l'attivazione di /PC-LOAD e una conseguente esecuzione condizionale di un salto determinata dalla eventuale assenza di Overflow richiedono un'istruzione:

- la cui codifica porti ad avere S2/S1/S0 = 101 agli ingressi di selezione del '151, **e**
- il cui microcode attivi il segnale JE.

A questo punto della spiegazione si sarà notato che i FF '74 espongono i flag sia in logica normale, sia in logica invertita, con lo scopo di poter più facilmente determinare se un determinato flag sia attivo o no; se ad esempio il Carry non fosse presente e si desiderasse eseguire un salto verificando la condizione "Carry non presente" (Branch on Carry Clear, BCC), sarebbe più semplice verificare se l'inverso del Carry fosse presente, così da attivare opportunamente il segnale di salto /PC-LOAD.

L'utilizzo di una NOR all'uscita Z del '151 permette di gestire sia i salti condizionali (dunque da validare con una apposita verifica logica, cioè quella della presenza/assenza di un determinato flag), sia i salti incondizionati:

- In caso di salto condizionale (BCS, BVC, BEQ eccetera), una verifica positiva di presenza/assenza del flag selezionato (normale o invertito) genera un'uscita HI sul '151 --> la NOR presenta dunque output LO ed attiva il caricamento sul Program Counter del valore presente sul bus (che altri non è che l'operando dell'istruzione di branch condizionale).

  - Notare che l'operando delle istruzioni di salto condizionale è un valore relativo, che viene addizionato al valore attuale del PC. **Long Story Short: carico in D il PC attuale, mentre carico in X l'operando dell'istruzione di salto; il nuovo valore da caricare nel PC corissponde all'indirizzo del byte successivo a quello dell'operando dell'istruzione Branch + l'operando stesso**, ad esempio:

- In caso di salto incondizionato (JMP, JSR), il microcode dell'istruzione di salto attiverà /WP (in logica invertita) che a sua volta attiverà il caricamento sul PC del valore presente sul bus: (/PC-LOAD = NOT (x+1) = 0), pertanto il PC caricherà dal bus il suo nuovo valore.

![NOR per l'attivazione di /PC-LOAD con salti condizionali ed incondizionati](../../assets/flags/30-flag-je-wp.png){:width="50%"}

*NOR per l'attivazione di /PC-LOAD con salti condizionali ed incondizionati.*

In definitiva, il microcode delle istruzioni di salto condizionato e di salto incondizionato sono molto simili e differiscono per i segnali di controllo:

- JE per attivare il '151 di selezione del flag del quale verificare presenza/assenza;
- WP per caricare senza altri controlli un determinato indirizzo sul PC.

Vi è un effetto non desiderato: "le istruzioni di salto condizionato non eseguite sprecano cicli di clock"… non si potrebbe semplicemente usare N per terminare anticipatamente l'istruzione? Lui sembra renderla un po' complicata

**29/01/2023 leggendo bene dice che dovrebbe essere possibile fare in modo che la logica elettronica dell'istruzione Jump vada ad attivare N se il salto non deve esserci… da verificare**

## Calcolo dei Flag N, V, Z e C

### Negative

Come detto precedentemente, il flag **N**egative è uguale al 7° bit del bus, cioè l'MSB: il flag viene gestito sulla linea D7 del bus in quanto i numeri Signed utilizzano proprio il **M**ost **S**ignificant **B**it (MSB) per indicare il proprio segno. Interessante notare che, essendo N mappato sul bus e non direttamente sull'ALU, è possibile rilevare un numero Negative anche in contesti esterni ai '181, ad esempio nel risultato di una rotazione fatta con lo Shift-Register o in un trasferimento di dato da un registro a un altro: tutto ciò che transita sul bus può essere oggetto di verifica del suo stato di numero Signed positivo o negativo.

### Zero

Il flag **Z**ero è attivo quando il valore presente nel bus è zero; invece di usare una serie di AND per verificare se tutte le linee sono LO (come accadeva nel SAP), un singolo comparatore '688 può svolgere la stessa funzione. Notare che anche questo flag opera sul bus e non sui risultati della sola ALU.

![Comparatore 74LS688 per verifica dello stato zero sul bus](../../assets/flags/30-flag-688.png){:width="50%"}

*Comparatore 74LS688 per verifica dello stato zero sul bus.*

### Overflow

Il flag O**V**erflow è calcolato utilizzando un '151 nella modalità descritta in ["74181 with V_Flag"](http://6502.org/users/dieter/v_flag/v_4.htm) sul sito 6502.org.

Avevo trovato la spiegazione molto criptica, o forse non propriamente adatta ai profani, tanto da impiegare alcune *decine* di ore per comprendere a fondo quanto enunciato rileggendo, cercando altre fonti, seguendo qualche video di aritmetica binaria, facendo esercizi su carta e su uno spreadsheet.

Tom evidenziava che gli MSB degli operandi dell'ALU H e B, insieme all'MSB risultante dall'operazione dell'ALU, erano utilizzati come input per verificare la condizione di overflow: iniziavo a realizzare che l'overflow era in realtà un calcolo molto semplice e preciso di bit.

In seguito capirò che il calcolo dell'overflow è legato al fatto che si stia lavorando con numeri Signed: questi numeri vengono rappresentati con il **Complemento di 2** (Two's Complement, o anche 2C), nel quale un MSB = LO indica un numero positivo, mentre un MSB = HI indica un numero negativo.

![Utilizzo di un 74LS151 per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.](../../assets/flags/30-flag-v-151.png){:width="50%"}

*Utilizzo di un 74LS151 per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.*

Per identificare l'esecuzione di un'operazione di addizione o di sottrazione e dunque selezionare quale debba essere l'ingresso corretto del '151 da attivare, si utilizzano due delle linee di selezione dell'operazione dell'ALU, in particolar modo:

| IR-Q1 | IR-Q3 | Operazione  |
| -     | -     | -           |
| HI    | LO    | Addizione   |
| LO    | HI    | Sottrazione |

NOTA BENE verificare **la congruenza tra istruzioni e collegamenti**; mi pare che lo schema di Tom sia errato.

In una delle innumerevoli sessioni di approfondimento e studio dell'overflow, ero finalmente arrivato a comprendere che se nella somma di due numeri con segno si nota un cambiamento di segno del risultato, si ha una situazione di overflow: il cambiamento di segno è rappresentato da una variazione inaspettata dell'MSB del risultato, cosa che un '151 opportunamente connesso permette di identificare.

In altre parole: in un byte sono possibili 256 combinazioni; in caso di numeri senza segno (Unsigned) è possibile contare da 0 a 255. Nel caso di numeri Signed, l'approfondimento nell'apposita sezione sull'aritmetica binaria spiegherà come i numeri positivi da 0 a 127 abbiano un riferimento paritetico con i numeri senza segno da 0 a 127, mentre i numeri con segno da -128 a -1 facciano il paio con i numeri senza segno da 128 a 255.

**vedere approfondimento** nella sezione apposita
aggiungere note con HEX.

### Carry

Il registro dei Flag include un registro dedicato al **C**arry. L'NQSAP include diverse  operazioni che possono generare un Carry:

- per i calcoli aritmetici il Carry corrisponde al Carry Output dell'ALU '181
- per le operazioni di shift/rotazione, il Carry è tratto dal **L**east **S**ignificant **B**it (LSB) (pin H-Q0) o dal MSB (pin H-Q7) del registro H.

L'utilizzo di un '151 rappresenta il sistema più efficiente per selezionare la sorgente del Carry. A seconda dell'istruzione in esecuzione, il microcode di quella istruzione provvederà infatti ad attivare opportunamente i segnali C0 e C1:

![Utilizzo di un 74LS151 per la selezione del Carry da memorizzare nel Carry flag](../../assets/flags/30-flag-c-151.png){:width="50%"}

*Utilizzo di un 74LS151 per la selezione del Carry da memorizzare nel Carry flag.*

| C1 | C0 | Selezione del Carry                                      |
| -  | -  | -                                                          |
| LO | LO | Provenienza dal Carry Output dell'ALU **(non invertito\*)** |
| LO | HI | Provenienza dal Carry Output dell'ALU **(invertito\*\*)**   |
| HI | LO | Provenienza dall'MSB (H-Q7) del registro H\*\*\*             |
| HI | HI | Provenienza dall'LSB (H-Q0) del registro H\*\*\*             |

- \* questa configurazione non viene utilizzata

Come già discusso nella pagina dell'ALU:

- \*\* il Carry del '181 lavora in logica negativa, pertanto un segnale C = LO in uscita significa che il Carry è presente; va da sé che per registrare lo stato del Carry in logica positiva sul registro del flag C devo invertire il segnale in ingresso;
- \*\*\* all'inizio di ogni istruzione il contenuto di H corrisponde esattamente a quello di A.

## Il Carry e i registri H e ALU

**Carry Input**

Oltre all'utilizzo con un '151 per eseguire salti condizionali basati sulla presenza / assenza del Carry, questo viene chiaramente utilizzato anche come ingresso per il [modulo ALU](../alu/#lalu-dellnqsap) per eseguire operazioni aritmetiche ('181) e di shift/rotazione ('194).

Nel caso specifico di utilizzo del Carry come input di H, l'opportuna programmazione del microcode dei segnali **CC** (**C**arry **C**lear) e **CS** (**C**arry **S**et) dell'istruzione in esecuzione può passare al Carry Input di H:

- un valore *hard-coded* 0
- un valore *hard-coded* 1
- il valore realmente presente nel registro C

Ecco come settare il Carry Output fixed HI o LO oppure semplicemente lasciarlo passare (riferimento ai segnali dell'NQSAP).

- CS LO, CC LO: Flag-In passa normale
- CS HI, CC LO: Flag-Out HI
- CS LO, CC HI: Flag-Out LO

L'output del Carry a ALU e H è controllato da LC ed LS CC e CS (01/10/2022 l'autore ha aggiornato i nomi sul blog, ma non sullo schema). Questi due segnali possono semplicemente passare il Carry attuale presente in C, oppure passare HI o LO.

Per il motivo già descritto sopra, nel caso di uso del Carry da parte dell'ALU (che lavora in logica "negativa", ma ancora da chiarire il senso, 27/11/2022) usiamo il valore invertito.

| Segnale | Descrizione         |
|  - | -                        |
| FC | write Carry flag         |
| FZ | write Zero flag          |
| FV | write oVerflow flag      |
| FN | write Negative flag      |
| FB | load flags from the bus  |
| JC | jump conditional         |
| C0 | carry source select 0    |  
| C1 | carry source select 1    |
| CC | ALU carry input clear    |
| CS | ALU carry input set      |

Normale:

De Morgan (l'ho capito 😁):

## Differenza tra ALU dell'NQSAP e del BEAM

Come si può vedere dallo schema del modulo ALU del computer BEAM, questo è quasi una copia 1:1 del modulo ALU del computer NQSAP: non avevo certamente la capacità di sviluppare autonomamente un modulo ALU così complesso e legato a doppio filo con altri moduli del computer, ma la comprensione completa del funzionamento dell'ALU sviluppata da Tom ha rappresentato comunque un traguardo molto importante.

[![Schema logico dell'ALU del computer BEAM](../../assets/alu/50-alu-beam-schematics.png "Schema logico dell'ALU del computer BEAM"){:width="100%"}](../../assets/alu/50-alu-beam-schematics.png)

*Schema logico dell'ALU del computer BEAM.*

Ecco una lista delle differenze:

- Per il registro B ho utilizzato un Flip-Flop tipo D [74LS377](https://www.ti.com/lit/ds/symlink/sn54ls377.pdf) al posto del [74LS574](https://www.onsemi.com/pdf/datasheet/74vhc574-d.pdf). A differenza del '574, il '377 è dotato di ingresso Enable, che solo quando attivo permette il caricamento del registro in corrispondenza del Rising Edge del clock: così facendo si elimina la necessità di un gate in ingresso sul clock per realizzare un Enable artificiale, come descritto nella sezione [L'ALU dell'NQSAP](#lalu-dellnqsap).

![Schema di uno degli 8 Flip-Flop del 74LS377](../../assets/alu/50-alu-377.png "Schema di uno degli 8 Flip-Flop del 74LS377"){:width="66%"}

*Schema di uno degli 8 Flip-Flop del 74LS377.*

 **Da fare**: Valutare se anche questo ha un riflesso positivo sul discorso del glitch

- Il computer NQSAP prevedeva 8 step per le microistruzioni, mentre il BEAM ne prevede 16. Come si vedrà però in maggior dettaglio nelle sezioni riservate al microcode, con soli 8 step non sarebbe stato possibile emulare alcune delle istruzioni del 6502, come quelle di salto relativo ed altre. Questa è in realtà una differenza architetturale più legata alla Control Logic, però l'impatto principale sul numero di step disponibili si riflette in particolar modo sull'ALU ed ha dunque sicuramente senso citarla in questa sezione.



## ALTRO
Flag e Microcode
Molte delle istruzioni modificano i flag.

- Vedere bene quali istruzioni CP* hanno bisogno di LF, anche sul file XLS

Altre referenze Tom Nisbet per Flags

- Question for all 74ls181 alu people on reddit led to the design of the oVerflow flag.
- How to add a decremental and incremental circuit to the ALU ? on reddit inspired the idea to drive the PC load line from the flags instead of running the flags through the microcode.
- Opcodes and Flag decoding circuit on reddit has a different approach to conditional jumps using hardware. Instead of driving the LOAD line of the PC, the circuit sits between the Instruction Register and the ROM and conditionally jams a NOP or JMP instruction to the microcode depending on the state of the flags. One interesting part of the design is that the opcodes of the jump instructions are arranged so that the flag of interest can be determined by bits from the IR. NQSAP already did something similar with the ALU select lines, so the concept was used again for the conditional jump select lines.

LINK: il PDF di MICR LOGIC come compendio a istruzioni ,indirizzamenti flag etc

http://www.6502.org/tutorials/vflag.html per V sezione 2.4.2.1

interessante la definizione di come rappresentare un numero in complemento di 2 (2C): quel numero che sommato al numero in oggetto dà zero come risultato: -1 = $FF; -2 = $FE

2.4.2.2 per esempi somme e sottrazioni

V indica se il risultato di somma o sottrazione è fuori dal range 