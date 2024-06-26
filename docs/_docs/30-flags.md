---
title: "Flag"
permalink: /docs/flags/
excerpt: "Costruzione del registro dei flag del BEAM computer"
---
[![Registro dei Flag del BEAM](../../assets/flags/30-flag-beam.png "Registro dei Flag del BEAM"){:width="100%"}](../../assets/flags/30-flag-beam.png)

Bisogna dire che più volte, leggendo la documentazione di Tom Nisbet, ho trovato delle idee molto clever; alcune di queste si trovano nel modulo dei Flag. Cercherò di riportare anche alcune fonti utilizzate da Tom indicandone i collegamenti alla fine della pagina.

[![Schema logico del modulo Flag di Tom Nisbet](../../assets/flags/30-flag-nqsap-schematics.png "Schema logico del modulo Flag di Tom Nisbet"){:width="100%"}](../../assets/flags/30-flag-nqsap-schematics.png)

*Schema logico del modulo Flag di Tom Nisbet, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Il registro dei Flag dell'NQSAP emula i 4 flag **NVZC** del 6502:

- **N**egative (N)
- O**V**erflow (V)
- **Z**ero (Z)
- **C**arry (C)

E' completamente differente dal semplice registro dei Flag del computer SAP di Tom Nisbet, nel quale un unico Flip-Flop [74LS173](https://www.ti.com/lit/ds/sdls067a/sdls067a.pdf) memorizzava i soli 2 flag C e Z nello stesso momento: la gestione delle istruzioni necessitava di 4 set di microcode, cioè uno per ogni combinazione dei segnali di Flag portati agli ingressi delle EEPROM; ogni set di microcode era personalizzato per attivare in output i corretti segnali per la gestione di C e/o Z. Questo è ben spiegato nel video di Ben [Conditional jump instructions](https://www.youtube.com/watch?v=Zg1NdPKoosU).

Nella realizzazione di Tom il microcode delle istruzioni non varia a seconda dello stato dei flag, che non sono più direttamente connessi agli indirizzi delle ROM che poi attivano diversi segnali di output in base all'indirizzo/flag presentato in ingresso!

Prendiamo in analisi un'istruzione di salto condizionale legata al flag Z:

- il microcode dell'istruzione attiverà un segnale "JUMP" in output sulle EEPROM andando ad attivare (vedi segnale /E) il Selector/Multiplexer [74LS151](https://www.ti.com/lit/ds/symlink/sn54s151.pdf) visibile in basso a destra nello schema;
- importantissimo comprendere che la selezione del flag da mettere in uscita dipende dalla codifica dell'istruzione in esecuzione, poiché i 3 bit Select S2, S1 ed S0 del '151 sono direttamente collegati all'Instruction Register, cioè *hardwired* in maniera similare a quanto succede per la ALU;

![Selector/Multiplexer 74LS151(../../assets/flags/30-flag-151-table.png]("Schema logico del modulo Flag di Tom Nisbet"){:width="50%"}

*Selector/Multiplexer 74LS151.*

- se per esempio l'istruzione *Jump on Zero* è codificata come 010 sui 3 segnali S2, S1 ed S0 comuni tra IR e registro dei Flag, questa andrà ad attivare il pin I2 di ingresso del '151 che, se troverà 1 al suo ingresso (vale a dire che il Flip-Flop del flag Zero ha valore logico HI), andrà ad abilitare il segnale PC-LOAD sul Program Counter, attivando il caricamento del nuovo indirizzo operando dell'istruzione di salto.

(si vedrà in seguito che questo è uguale per tutte le istruzioni) 

 ![Output dell'Instruction Register verso il modulo Flag con evidenza dei 3 bit di selezione dell'istruzione di salto condizionale](../../assets/flags/30-flag-cl-ir-out.png)

*Output dell'Instruction Register verso il modulo Flag con evidenza dei 3 bit di selezione dell'istruzione di salto condizionale.*

Detto in altre parole: la logica del salto condizionale del SAP era implementata nel microcode, utilizzando linee di indirizzamento delle ROM. Poiché i flag dell'NQSAP sono invece implementati in hardware, non c'è bisogno di consumare preziose linee di indirizzamento delle EEPROM. I miglioramenti derivanti sono:

- possibilità di settare i flag anche singolarmente;
- risparmio delle linee di indirizzamento delle EEPROM;
- l'output delle EEPROM non si modifica durante l'esecuzione della singola istruzione (ma nel SAP-1 come si comportava? 04/10/2022 l'ho compreso andando a rileggere gli appunti del BE 8 bit computer). Teoricamente, e l'avevo letto anche altrove, questo potrebbe essere un problema perché causa "glitching".

![Ingressi di selezione dell'istruzione di salto condizionale del registro dei Flag e connessione "hardwired" con l'IR](../../assets/flags/30-flag-select-in.png)

*Ingressi di selezione dell'istruzione di salto condizionale del registro dei Flag e connessione "hardwired" con l'IR.*

## FLAG e gestione

Un multiplexer '157 permette di copiare i valori dei flag dalla memoria (tipo istruzione Pull Processor Status PLP). Questi FF non vengono mai pre-settati dunque /Preset resta sempre collegato a Vcc (e dunque mai attivo), mentre hanno invece collegamento al reset generale del sistema /RST.

- I FF '74 hanno come ingresso le uscite del MUX '157, che prende 4 segnali scegliendone la provenienza:

  - 1. dal bus (tranne il flag Negative, che viene sempre preso direttamente dalla linea D7 del bus, perché i numeri negativi Signed hanno l'uno iniziale sull'MSB 😁)
    - 24/10/2022 ma… questo significa allora che NQSAP lavora in complemento di due? Devo chiedere a Tom… no, la domanda giusta è se il computer lavora con i numeri Signed o Unsigned!
    - b. 09/01/2023 in realtà il 74181 lavora in 2C
    - c. 06/01/2024 meglio dire, che sono io a decidere come lavorare, col programma che faccio girare.
  - 2. dai segnali C, Z e V
    - a. C con il Data Selector / Multiplexer /151
    - b. Z col comparatore '588:
    - c. V con il Data Selector / Multiplexer /151
- I 4 flag FC, FV, FZ, FN sono attivati grazie a segnali dedicati che vengono ANDati col CLK (caricamento registri al Rising Edge del clock). Si segnala che in questo modo una singola istruzione può settare più di un flag alla volta, ad esempio ADC potrebbe settarli tutti (in effetti l'istruzione ADC impatta tutti i flag contemporaneamente).
- Le istruzioni CLC, CLV e SEC possono settare o pulire i flag senza usare linee ROM; infatti la ALU può mettere in uscita tutti 0 o tutti 1 e così possiamo pulire o settare il bus e caricare solo il flag che ci interessa mandando il flag apposito.
- Un '245 permette di esportare i Flag su bus per salvarli in una location di memoria con Push Processor Status (PHP).

## Salto condizionale

Quando un flag veniva variato nel SP-1, venivano cambiati gli indirizzi delle ROM per presentare una logica opportunamente diversa in uscita. In questo modo, ad ogni flag corrispondevano anche delle linee di indirizzamento "rubate" alle ROM.
• Nell'NQSAP i valori dei flag presenti nei 74LS74, sia Q sia /Q, vengono passati al Data Selector / Multiplexer '151, che funge da selettore delle uscite dei FF (seleziona il flag e ne seleziona anche "il genere", ad esempio l'istruzione di salto del Carry potrebbe essere BCC o BCS), dunque grazie ai segnali di selezione in ingresso nel '151 IR-Q5, IR-Q6 e IR-Q7 prendo direttamente dal FF il Flag che mi interessa, normale o invertito.
IR-Q5, IR-Q6 e IR-Q7 arrivano infatti dall'Instruction Register, dunque una determinata istruzione di Branch predetermina il flag da esporre e sul quale effettuare il salto o no a seconda che il test sia vero o falso.
• Mi stavo domandando… come faccio a far stare tutto (le istruzioni di branch (3 bit), le istruzioni dell'ALU (5 bit) e tutte le altre istruzioni in soli 8 bit? Come faccio a gestire tutte le combinazioni e a costruire una matrice di istruzioni funzionante?
   • Poi ho notato che nel '151 dei flag c'è un segnale di controllo JE; questo è sicuramente "acceso" dalle istruzioni di salto condizionale, dunque
      ○ se JE è HI, attivo l'uscita del '151 e dunque attivo il segnale di caricamento del Program Counter se l'input dal FF del flag di interesse è positivo;
      ○ se JE è LO, disattivo l'uscita del '151 e dunque ignoro lo stato dei selettori S0, S1 ed S2 e non vado ad attivare il segnale di caricamento del Program Counter;
      ○ mi aspetto ad esempio che l'istruzione BCS/JCS (Jump on Carry Set)
         § configuri S0, S1 ed S2 in modo da portare in output Z il valore del pin 4, che è collegato all'FF del C
         § dunque S0 = LO, S1 = LO ed S2 = LO portano in Z il flag C
         § se C = 1 l'output della NOR è LO (/(1+x) = 0) e dunque il valore presente nel bus viene caricato nel PC.
• Si dice anche che "questo semplifica il microcode perché tutte le operazioni di Jump utilizzeranno lo stesso microcode" similarmente a quanto accade coi 5 bit di gestione delle istruzioni ALU… e qui mi sfugge qualcosa, devo capire bene
    • come si costruisce la matrice delle istruzioni 20/06/2023 capito
    • perché tutte le istruzioni dovrebbero essere "uguali"… 04/10/2022 forse ho capito perché… in effetti la scelta del flag dipende dal codice dell'istruzione stessa, che essendo in output dall'Instruction Register viene applicata agli ingressi Select del '151… e dunque è sufficiente che nell'istruzione venga abilitato il JE 😁, tutte le istruzioni sono dunque uguali.
	Incrocio l'uscita del '151 con una NOR:
	        • Se parlo di un salto condizionato, il flag selezionato normale o invertito mi genera un'uscita HI sul '151 --> il NOR presenta output LO che (da verificare, ma credo sia così) mi attiva il caricamento sul Program Counter del valore presente sul bus (che altri non è l'operando dell'istruzione di branch condizionale tipo BCC, BCS, BVC etc.).
	                ○ 20/06/2023 Attenzione: l'operando è un valore relativo, dunque dovrò fare un po' di microistruzioni per calcolare il valore corretto da mettere sul PC… 31/01/2024 in realtà si usa D e X: in D metto il PC attuale, mentre in X metto il valore del Branch relativo, che è calcolato a partire dal byte successivo all'operando dell'istruzione Branch.
	        • Se invece ho un salto incondizionato, WP dalle ROM sarà a LO e mi attiverà comunque il caricamento sul PC del valore presente sul bus (la NOR lavora (/(1+x) = 0) e dunque attiva /PC-LOAD. Questo è l'operando dell'istruzione di salto incondizionato (JMP $XX).
	• Le istruzioni di salto condizionato alla fine sono identiche a quelle di salto incondizionato, ma in più c'è il /JE: se il flag (normale o invertito che sia) non è HI, allora il Program Counter non viene caricato
	        • Downside: "le istruzioni di salto condizionato non eseguite sprecano cicli di clock"… non si potrebbe semplicemente usare N per terminare anticipatamente l'istruzione? Lui sembra renderla un po' complicata
	        • 29/01/2023 leggendo bene dice che dovrebbe essere possibile fare in modo che la logica elettronica dell'istruzione Jump vada ad attivare N se il salto non deve esserci… da verificare
	
## Calcolo dei Flag

	• Il flag Negative è semplicemente il MSB del bus 😁 (06/01/2023 direi perché sto ragionando con numeri Signed). Interessante che essendo mappato sul bus e non direttamente sull'ALU, potrei rilevare un Negative anche in contesti esterni all'ALU, ad esempio uno shift del shift-register o un trasferimento di dato da un registro a un altro.
• Utilizzo del Carry da ALU e H	
	• Il flag Zero si attiva se il valore presente nel bus è zero; invece di usare una serie di AND (come nel SAP-1) per verificare se tutte le linee fossero LO, ecco che il comparatore 74HCT688 può svolgere lo stesso lavoro. Anche questo opera sul bus e non sulla sola ALU.
	
	
	
	Zero
	
	
	Overflow
	
	        • The Overflow flag is calculated using a 74LS151 8-to-1 selector as described in "74181 with V_Flag" on 6502.org http://6502.org/users/dieter/v_flag/v_4.htm.
	        • The MSB of the ALU operands in H and B, as well as the MSB of the ALU result, are used as inputs.
	                ○ 30/10/2022 Dunque, come sto iniziando a capire un pochino, l'overflow è un calcolo molto semplice e preciso di bit… 
	        • Two of the ALU operation select lines are used to differentiate between addition and subtraction. 06/01/2023 ho capito il riferimento:
	                ○ IR-Q1 HI e IR-Q3 LO = addizione
	                ○ IR-Q1 LO e IR-Q3 HI = sottrazione
	                ○ 20/06/2023 attenzione anche qui alla congruenza tra istruzioni e collegamenti
	
	• 23/10/2022 oggi ho approfondito l'Overflow: se nella somma di due numeri signed noto un cambiamento di segno, allora ho un overflow
	• però l'NQSAP non lavora in complemento di due, dunque attenzione a cosa diciamo… qui non mi sembra di poter applicare il caso precedente… 27/11/2022 e in effetti rileggendo la questione è che stiamo lavorando non in complemento di due, ma con numeri signed… 06/01/2023 rileggendo ulteriormente direi che non è proprio corretto. Il complemento di 2 è semplicemente il modo di rappresentare i numeri signed, dove MSB = LO indica numero positivo e MSB = HI indica numero negativo.
	Overflow
	
	
	
	
	• Nel Flags Register ho un Carry Input che scrivo nel registro C e che può derivare da diverse  operazioni:
	        • per i calcoli matematici corrisponde al Carry Output dell'ALU '181.
	        • per le operazioni Shift, si tratta dell'LSB (pin H-Q0) o MSB (pin H-Q7) del registro H.
	                ○ Verificare se MSB = risultato di Shift Left o Right
	        • Un Multiplexer / Data Selector '151, a seconda dei suoi input C0 e C1, determina la sorgente del Carry:
	                ○ ALU (sia normale sia invertito), oppure
	                ○ MSB del registro H, oppure
	                ○ LSB del registro H.
	
	Il valore in input del registro Carry Input dipende anche dall'istruzione che è stata eseguita: può arrivare da ALU o da H, CLC e SEC. Vedi tabella per l'uso del Carry nelle varie situazioni:
	
	
	
	Suppongo che il significato sia:
	        • Se il registro sorgente dell'operazione è l'ALU
	                ○ per istruzioni somma/sottrazione passo il Carry esistente negato
	                ○ per istruzioni INC o DEC passo il Carry "normale"
	        • Se il registro sorgente dell'operazione è H (usato per le varie rotazioni) prendo MSB per rotazione a sinistra e LSB per rotazione a destra… ma questo non mi convince… 26/09/2022 ma ora che ci penso mi pare ok: prendo il MSB  e poi faccio shift a sinistra, dunque "salvo" il MSB e viceversa quando faccio shift a destra
	
	L'ultimo caso perché noi pensiamo in logica positiva col Carry che, se presente come conseguenza del risultato dell'operazione, è HI per l'addizione e LO per il prestito, come nel 6502, mentre la ALU '181 lavora in logica negativa, con LO che indica che il Carry  è presente nell'addizione e con HI che indica che c'è un prestito nella sottrazione.
	        • ma non mi è chiaro… mi pare che lavori in entrambi i modi a seconda degli input che le vengono passati. 06/10/2022 credo di aver capito. Praticamente il 181 nella modalità High-Active Data utilizza HI per indicare un Carry assente e LO per indicare il Carry presente, come vedo nel datasheet…
	        • Però poi non mi è chiaro davvero cosa significa che la ALU lavora in logica positiva o negativa… perché anche gli input sono in logica negativa, ma un semplice esercizio sul quaderno cercando di invertire tutto non mi ha dato risultato…
	        • Dunque bisogna provare a fare un circuito 😊 per capire
	
	• Come già detto, i flag possono essere anche letti (PLP) e scritti (PSP) dal / verso il bus.
	
	Carry Input
	
	
	Il '151 opera così, cioè a seconda degli input S0, S1 ed S2 seleziono cosa portare in uscita da I0 a I7:
	
	
	
	• Oltre a essere usato dal 151 per i salti condizionali, il Carry in uscita, cioè Carry Output, è anche input dell'ALU e di H; in questo ultimo caso, a seconda dell'istruzione, utilizzando CC e CS può:
	        • essere hard-coded 0
	        • essere hard-coded 1
	        • essere il valore presente nel registro C
	
	Ecco come settare il Carry Output fixed HI o LO oppure semplicemente lasciarlo passare (riferimento ai segnali dell'NQSAP).
	        • Set LO, Clear LO: Flag-In passa normale
	        • SET HI, Clear LO: Flag-Out HI
	        • SET LO, Clear HI: Flag-Out LO
	        
	Nota:
	        • Clear è LC (ALU Clear)
	        • Set è LS (ALU Set)
	        • 05/10/2022 nei vecchi schemi li chiamava LC e LS, ma questi sono diventati CC e CS perché avevo chiesto spiegazioni in quanto non mi trovavo e Tom mi ha risposto
	
	L'output del Carry a ALU e H è controllato da LC ed LS CC e CS (01/10/2022 l'autore ha aggiornato i nomi sul blog, ma non sullo schema). Questi due segnali possono semplicemente passare il Carry attuale presente in C, oppure passare HI o LO.
	Per il motivo già descritto sopra, nel caso di uso del Carry da parte dell'ALU (che lavora in logica "negativa", ma ancora da chiarire il senso, 27/11/2022) usiamo il valore invertito.
	
	
	
	Signal
	Description
	FC
	write Carry flag
	FZ
	write Zero flag
	FV
	write oVerflow flag
	FN
	write Negative flag
	FB
	load flags from the bus
	JC
	jump conditional
	C0
	carry source select 0
	C1
	carry source select 1
	LC -> CC
	ALU carry input clear
	LS -> CS
	ALU carry input set
	
	Normale:
	
	De Morgan (l'ho capito 😁):
	
	
	
• Flag e Microcode	Molte delle istruzioni modificano i flag.
	
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
