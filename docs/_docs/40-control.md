---
title: "Control Logic"
permalink: /docs/control/
excerpt: "Control Logic del BEAM computer"
---
[![Control Logic del BEAM computer](../../assets/control/40-beam-control.png "Control Logic del BEAM computer"){:width="100%"}](../../assets/control/40-beam-control.png)

**WORK IN PROGRESS**

[![Schema della Control Logic dell'NQSAP](../../assets/control/40-control-logic-schema-nqsap.png "Schema logico della Control Logic dell'NQSAP"){:width="100%"}](../../assets/control/40-control-logic-schema-nqsap.png)

*Schema della Control Logic dell'NQSAP, leggermente modificato al solo scopo di migliorarne la leggibilità.*

Per confronto, affianchiamo anche lo schema della Control Logic del SAP computer di Ben Eater.

[![Schema della Control Logic del SAP computer](../../assets/control/40-control-logic-schema-SAP.png "Schema logico della Control Logic del SAP computer"){:width="100%"}](../../assets/control/40-control-logic-schema-SAP.png)

*Schema della Control Logic del SAP computer.*

Il numero maggiore di EEPROM dell'NQSAP indica chiaramente la disponibilità di un numero ben maggiore di segnali di controllo rispetto a quelli disponibili nel SAP Computer. La complessità dell'NQSAP e del BEAM computer è tale per cui i 16 segnali del SAP non sarebbero sufficienti per pilotare moduli complessi come ad esempio l'ALU e il registro dei Flag.

### I 74LS138

Oltre all'incremento naturale dovuto al maggior numero di EEPROM utilizzate, una serie di 3-Line To 8-Line Decoders/Demultiplexers <a href = "https://www.ti.com/lit/ds/symlink/sn74ls138.pdf" target = "_blank">74LS138</a> permette di gestire fino a 15 segnali con solo 4 linee in uscita da una singola EEPROM.

Come visibile nello schema, ogni 138 presenta 3 pin di input e 3 pin di Enable; connettendo opportunamente i 3 pin di input e uno dei pin di Enable di ogni 138, è possibile pilotare ben 4 demultiplexer (per un totale di 30 segnali di output) usando solo gli 8 segnali in uscita da una singola EEPROM.

Perché fino a 30 segnali e non 32? Il microcode prevede delle microistruzioni nelle quali nessuno dei registri pilotati dai '138 deve essere attivo in quel momento, dunque almeno uno dei pin di output di ogni coppia di '138 dovrà essere scollegato. Ad esempio, nel caso dell'NQSAP, un output 0000.0000 della prima EEPROM attiverà il pin D0 del primo '138 e del terzo '138: entrambi i pin D0 sono scollegati, dunque sarà sufficiente mettere in output 0x00 sulla prima EEPROM per non attivare alcuno tra tutti i registri pilotati dai '138.

Le uscite dei '138 sono allo stato LO quando attive, circostanza che risulta molto comoda per la gestione dei segnali nei registri, in quanto molti dei chip utilizza ingressi negati (ad esempio 74LS245 e 74LS377). Praticamente i '138 vengono utilizzati come sink e non source, dunque lo stato logico delle porte in uscita è normalmente HI:

- una prima coppia di '138 è dedicata ai segnali Read;
- una seconda coppia di '138 è dedicata ai segnali Write;
- solo una porta per ogni coppia di registri può essere allo stato LO; tutte le altre si trovano allo stato HI.

Si può notare nello schema che tutti i registri del computer sono indirizzati con i DEMUX; fa eccezione il registro H, il quale, come esposto anche nella pagina **link XYZ** della ALU, deve avere la possibilità di essere scritto parallelamente al registro A; tra l'altro, essendo il registro H un registro a scorrimento e necessitando di due segnali di ingresso (HL ed HR), una sua gestione mediante '138 sarebbe oltremodo complicata e risulta preferibile indirizzarlo mediante segnali dedicati in uscita da un'altra EEPROM.

Poiché i 138 attivano un solo output alla volta, si ottiene una involontaria **ottimizzazione / semplificazione** nei segnali di lettura dei registri, perché non sarà mai possibile attivare più di un registro contemporaneamente, evitando così possibili cortocircuiti tra uscite allo stato HI e uscite allo stato LO: abbiamo la certezza di non poter attivare per errore due segnali Read contemporaneamente.

Per la scrittura è diverso, perché potrei dover scrivere su più registri contemporaneamente  e questo non genererebbe contenzionso sul bus (cercare un esempio di isturzione che scrive in due registri contemporaneamente, ma non H)

Altre 3 ROM hanno segnali che possono essere attivi anch allo stesso tempo.

in altre parole, una prima EEPROM gestisce 4 '138, che pilotano:

- i segnali di *lettura* di tutti i registri, eccetto il registro H;
- i segnali di *scrittura* di tutti i registri, eccetto il registro H e il registro dei Flag.


	• 15/06/2023: ho deciso di fare come Tom e mettere alla fine di ogni istruzione una microistruzione di reset così da poter anticipare la fine dell'istruzione e passare alla prossima senza dover aspettare tutti i cicli a vuoto.


### tabella segnali

È forse utile fare qui una tabella cheriassume tutti i vari segnali utilizzati nel computer ?

	• C0 e C1 *** sono usati per determinare se il Carry da salvare nel Flags Register deve venire dal Carry dell'ALU o da H (H-MSB o H-LSB) (dall'ALU per operazioni matematiche o da H per operazioni di Shift)
	• DY e DZ, usati nel modulo D, X e Y
		○ per selezionare (con DY) se usare index di X o Y, oppure
		○ se utilizzare solo D (attivando DZ)
sono condivisi con C0 e C1 ***

SE Stack Enable, vedi pagina Stack Pointer, condivisi con C0 e C1*** (chiamiamoli SU Stack Up e SD Stack Down) per definire se fare conteggio Up o Down

CC e CS per selezionare che tipo di Carry dobbiamo presentare all'ALU e ad H (quello effettivamente presente in FLAG C, oppure 0, oppure 1), vedi spiegazione in Flags

• Gli indirizzi più alti delle ROM sono utilizzati per selezionarle (hardwired)

• Se /LDR-ACTIVE viene attivato (LO), LDR-ACTIVE passa a HI e disattiva le ROM2 e ROM3 collegate via /OE.


## CONTROL LOGIC PART 1

8-bit CPU control logic: Part 1
Prima cosa che facciamo è leggere un comando e metterlo nell'Instruction Register, che tiene traccia del comando che stiamo eseguendo.

• Prima fase: FETCH. Poiché la prima istruzione sta nell'indirizzo 0, devo mettere 0 dal Program Counter PC (comando CO esporta il PC sul bus) al Memory Address Register MAR (comando MI legge dal bus e setta l'indirizzo sulla RAM) così da poter indirizzare la RAM e leggere il comando.
• Una volta che ho la RAM attiva all'indirizzo zero, copio il contenuto della RAM nell'Instruction Register IR passando dal bus (comando RO e comando II).
• Poi incrementiamo il PC col comando Counter Enable CE (nel video successivo il CE viene inserito nella microistruzione con RO II).
• Ora eseguiamo l'istruzione LDA 14, che prende il contenuto della cella 14 e lo scrive in A. Dunque poiché il valore 14 sono i 4 LSB del comando, sono i led gialli dell'IR e col comando Instruction Register Out IO ne copio il valore nel bus e poi, caricando MI, indirizzo la cella di memoria 14 che è quella che contiene il valore (28 nel mio caso) che esporto nel bus col comando RAM Out RO e che caricherò in A col comando AI. 

• ADD 15 ha sempre una prima fase di Fetch, uguale per tutte le istruzioni, e poi come sopra il valore 15 è quello della cella 15 e dunque Instruction Register Out IO che posiziona sul bus i 4 LSb del comando ADD 15, poi MI così setto l'indirizzo 15 della RAM, il cui contenuto metto sul bus col comando RAM Out RO e lo carico in B con BI così avrò a disposizione la somma di A e B, che metto sul bus con EO e che ricarico in A con AI. 

• Prossima istruzione all'indirizzo 3 è OUT che mette sul display il risultato della somma che avevo latchato in A. La prima fase di Fetch è uguale alle altre. Poi faccio AO per mettere A sul bus e OI. 

## CONTROL LOGIC PART 2

Le istruzioni sono fatte di più step, chiamati microistruzioni. La Control Logic deve settare correttamente la Control Word per ogni microistruzione così quando arriva il clock questa viene eseguita. Dobbiamo dunque sapere sempre quale istruzione stiamo eseguendo e a che step siamo. Ci serve un contatore per tracciare la microistruzione. Usiamo 74LS161 che può contare da 0 a 15.

NB: Dobbiamo settare la Control Logic tra un clock e l'altro… come a dire che la Control Logic deve "essere pronta" prima che l'istruzione venga eseguita: possiamo usare un NOT per invertire il clock e usare questo per gestire il 74LS161 della Control Logic.
	• I registri sono aggiornati al Rising Edge del CLK, che corrisponde al Falling Edge del /CLK.
	• Le microistruzioni sono aggiornate al Falling Edge del CLK, che corrisponde al Rising Edge del /CLK.
	• CLK gestisce tutti i registri principali: PC, MAR, RAM, IR, A, B, Flag: al Rising Edge del CLK, avvengono le azioni di caricamento dei registri. Quando c'è il segnale CE Counter Enable attivo, il PC viene incrementato al Rising Edge e l'indirizzo viene aumentato di uno.
	• /CLK gestisce il Ring Counter e di conseguenza la Control Logic: è sfasato di 180°, dunque al Falling Edge di CLK corrisponde il Rising Edge di /CLK
		○ All'accensione del computer
			§ PC è 0 e RC (Ring Counter) è 0
			§ la CL presenta CO|MI in uscita
			§ il 245 del PC è attivo in output
			§ il 245 del MAR è attivo in input.
		○ Arriva il Rising Edge del CLK
			§ il FF 173 del MAR carica (MI) l'indirizzo di memoria presentatogli (CO) dal PC
		○ Arriva il Falling Edge del CLK, che corrisponde al Rising Edge del /CLK
			§ RC si incrementa e la CL presenta la microistruzione successiva RO|II|CE
				□ la RAM è attiva in output
				□ IR è attivo in input
				□ PC è attivato per contare
		○ Arriva il Rising Edge del CLK
			§ il FF 173 dell'IR carica (II) il valore presentato dalla cella di RAM (RO) indirizzata dal MAR
			§ PC si incrementa (CE)
		○ Arriva il Falling Edge del CLK, che corrisponde al Rising Edge del /CLK
			§ RC si incrementa e la CL presenta la microistruzione successiva IO|AI
				□ IR mette in output
					® i 4 MSB che vanno ad indirizzare le EEPROM della CL
					® i 4 LSb che vanno sul bus; immaginiamo ad esempio istruzione immediata LDA #$05
				□ il 245 del Registro A è attivo in input
		○ Arriva il Rising Edge del CLK
			§ il FF 173 del Registro A carica (AI) il valore presentato sul bus (IO) dall'Instruction Register 

Il 74LS138 è un decoder che può prendere i 3 bit (ce ne bastano 3 per gestire 8 cicli, visto che gli step delle microistruzioni sono al massimo 6) e convertirli in singoli bit che rappresentano lo step della microistruzione corrente e poi uno di questi, l'ultimo, che resetta il 74LS161 in modo da risparmiare i cicli di clock inutilizzati.

A questo punto abbiamo nell'Instruction Register l'istruzione attualmente in esecuzione e nel contatore lo step, che rappresenta la microistruzione. Possiamo usare una Combinational Logic per settare i nostri segnali da abilitare per ogni microistruzione. Per esempio quando siamo in T0, che è la prima microistruzione, prendiamo l'uscita del 74LS138, la neghiamo con un NOT per portarla positiva e attiviamo CO e MI e poi Clock. Alla successiva attiviamo RO e II  e poi Clock e alla successiva attiviamo CE e poi Clock.
E poiché CE può essere inserito nella stessa microistruzione di RO e II possiamo ridurre la lunghezza delle microistruzioni a un massimo di 5 (step 0-4).



## 8-bit CPU control logic: Part 3
La fase Fetch è uguale per tutte le istruzioni, dunque istruzione XXXX e imposto solo gli step.
Per LDA uso il valore 0001 e imposto gli step con le microistruzioni opportune.
Per ADD uso il valore 0010 e imposto gli step con le microistruzioni opportune.
Per OUT uso il valore 1110 e imposto gli step con le microistruzioni opportune.

Praticamente ora abbiamo il contatore delle microistruzioni (T0-T4) e il contatore dell'istruzione (Instruction Register MSB). Posso creare una combinational logic che, a seconda dell'istruzione che ho caricato nell'Instruction Register + il T0/4 dove mi trovo mi permetta di avere in uscita i segnali corretti da applicare al computer. Praticamente ho due fasi:
• Fetch, in cui carico l'istruzione dalla RAM nell'Instruction Register
• Dopo la fase di Fetch so "cosa devo fare", perché a questo punto ho l'istruzione nell'Instruction Register









03/07/2022 Dunque programmo tutte le istruzioni e gli step: ma ho un problema.
Ho programmato le due EEPROM seguendo la mia sequenza scritta sul quaderno, ma in corrispondenza dello step T2 vedevo ben 4 led accesi nel primo byte. Ho riprogrammato la EEPROM, ma mi sono accorto che toccandola gli output variavano in maniera piuttosto ballerina. In effetti era accaduto anche lo stesso nello step T1 e l'ho riprogrammata pensando ancora a un errore di programmazione, ma evidentemente accade che se tocco la EEPROM gli output cambiano e questo si riflette nell'esecuzione del "programma". Devo capire qual è il problema.
Provo a usare il programmatore di TL866 per vedere se va bene e cancellarla. La diagnostica è OK.

05/07/2022 ho provato a debuggare il programmatore "manuale" e stavolta il tutto funziona. Prima ho tolto i cavetti fissi e li ho sostituiti con jumper e poi ho provato anche a sostituire il dip-switch, ma non ho capito esattamente quando il tutto si è messo a funzionare. Fatto sta che ho provato a programmare anche un solo byte alla volta e poi a leggere la EEPROM sul TL866 per verificare se memorizzavo o no… a un certo punto ho avuto l'impressione che il problema fosse che lasciavo /OE a ground quando stavo programmando la EEPROM, ma la giornata precedente mi pareva di aver controllato più e più volte. Se ci penso, credo che siano i cavetti… devo provare a rimetterli giù.

Ho fatto la sequenza passo-passo e tutto ha funzionato 😁😁😁… tranne che non si fermava all'HALT, dunque mi sono ricordato che c'era un discorso di clock negativo da dare al contatore delle microistruzioni e ora funziona… da approfondire come funziona





///////////////////////////////////////////////////////////////////////////////////////



La realizzazione del comuter SAP mi ha permesso finalmente di capire cosa sia il microcode di un computer moderno.

È piuttosto comune leggere ad esempio che è necessario aggiornare il bios dei server per indirizzare falle di sicurezza che sono state scoperte e che potrebbero essere utilizzate dagli hacker per puntini puntini puntini nuovo paragrafo
Non capendo come potesse essere aggiornata una CPU, dal momento che si tratta di un componente non programmabile virgola non riuscivo a comprendere come fosse possibile arginare i problemi di sicurezza; con il microcode ho capito

Ritornando alla dimensione delle EEPROM da utilizzare per il microcode, nei miei appunti trovo traccia di diverse revisioni, ad esempio:

- come notavo anche nelal costruzione del modulo RAM in cui si indicavano le 256 istruzioni, notavo che servivano 28c64. ram/#mux-program-mode-e-run-modeerano necessari 8 bit di istruzioni, 3 di step e 2 di flag = 13 pin totali, portanto si rendevano necessarie delle 28C64… e avevo dimenticato che mi sarebbe servito un bit aggiuntivo per la selezione delle due EEPROM

mi servono EEPROM 28C64 per avere 256 (8 bit) istruzioni + 3 step + 2 flag, ma dimenticavo che avendo due ROM gemelle dovevo gestirne anche la selezione e dunque aggiungere un ulteriore bit, pertanto mi servirebbero delle 28C128;
- avrei potuto però ridurre il numero di istruzioni a 64, dunque mi sarebbero bastati 6 bit per indirizzarle e ridurre così il numero totale di indirizzi richiesti...
- ... ma forse mi sarebbero serviti altri segnali di controllo oltre ai 16  disponibili in due ROM e dunque me ne sarebbe servita una terza... e dunque due bit di indirizzamento

Posso sicuramente dire che avevo le idee ancora confuse.

## ring counter

• Ring Counter. Vedi spiegazione per resettare in maniera sincrona il 74LS161 sulla pagina dei chip
	• Praticamente usando il '161:
		○ con /CLR, che è asincrono, faccio il reset "hardware"
			§ 17/06/2023 Tom segnala che questo segnale asincrono non è invece ideale per pulire il Ring Counter utilizzando il microcode perché, essendo appunto asincrono, non sarebbe gestito dal clock: infatti, non appena attivato, andrebbe a resettare il ciclo di microistruzione esattamente all'inizio invece che quando arriva il impulso di clock!
			§ 04/07/2023 Va invece benissimo per fare il reset del Program Counter 😁 anche col clock stoppato
			§ A dire il vero si potrebbe comunque utilizzare questo segnale, ma significherebbe dover aggiungere una microistruzione dedicata al reset alla fine di tutte le altre microistruzioni. Utilizzando una modalità di reset sincrona su questo chip si potrebbe invece aggiungere il segnale di reset all'ultima microistruzione del ciclo.
		○ con il /LOAD ("N") e tutti i pin di input a 0, il Ring Counter si resetta in sincrono con il /CLK 😁. Assomiglia un po' al JUMP del Program Counter. Notare il /CLK, che è invertito rispetto al CLK principale e che dunque permette di lasciar terminare l'esecuzione della microistruzione corrente prima di fare il LOAD.
	• Importante: decidere se fare 8 step (3 bit) o 16 step (4 bit) --> 14/06/2023 ho deciso di passare a 16 step
		○ questo forse significa che poiché il '138 è un decoder 3-to-8 devo metterne due "in cascata"? Posso farlo?
		○ Nell'NQSAP-PCB Tom mette 16 step
		○ 3 uscite del 161 vanno al 138 per pilotare 8 led (2^3 = 8); per il 9° LED Tom è il solito furbo: invece di mettere due 138 per pilotare 16 led, aggiunge un led al quarto pin del 161, così ad esempio 11 = 3 + 8 e dunque si accendono il 3* led di 8 pilotato dal 138 e quello dell'"Extended Time" pilotato dal 161
	• Gli ingressi D0-D3 sono tutti a 0, così quando arriva il /LOAD (o /PE) sincrono (segnale "N" per Tom), il conteggio ricomincia da zero
	• Le uscite Q0-Q2 del 161 vanno a MA0-MA2 per avere 8 step di microistruzioni, ma se aggiungo un quarto pin al contatore, posso avere 16 step
	• Il /LOAD arriva da N 17 della ROM0
CEP e CET sono a +Vcc

## Instruction Register

24/08/2022
	• Devo aumentare il numero di linee da 4 a 8 verso la Control Logic
	• 25/08/2022 In realtà l'ho portato a 6 in modo da risparmiare qualche linea (però ho sempre un buon numero, 2^6 = 64) di istruzioni possibili) e poter usare le EEPROM 28C64
	• 14/06/2023 importante: ho deciso di seguire quanto fatto da TOM e poter avere 256 operazioni, dunque le linee sono sicuramente:
		• Le EEPROM 28256 da 32 K hanno 15 indirizzi.
			□ 8 per le istruzioni (256) ==> Instruction Register da 8 bit
			□ 4 per le microistruzioni (16)
			□ 2 per selezionare le ROM
			□ Ne resta uno libero.
		○ 03/09/2022 Mi chiedo se il XCVR 74LS245 sia ancora necessario… ora che l'operando non è più parte dell'istruzione stessa, ma è separato, non lo devo esporre sul bus dall'Instruction Register attivando il pin IO (Instruction Register Out)… 😁 03/12/2022 più ci guardo e più sembra che non sia necessario
		○ e forse posso risparmiare un segnale, visto che IO non mi serve più, perché non devo più rimettere sul bus i 4 LSB dell'IR che in passato fungevano da operando
		○ Per indirizzare i problemi di glitching Tom ha bufferizzato l'IR, cioè due FF da 8 registri in cascata, così il primo viene aggiornato al normale caricamento dell'IR (che corrisponderebbe a T7 (step 1), ma causando un glitch sulla ROM)… invece di collegare il FF agli ingressi delle ROM, viene collegato a un altro FF che viene caricato col Falling Edge del CLK / Rising Edge del CLK, così le uscite delle ROM vengono aggiornate alla fine della microistruzione quando i segnali sono stabili 😁
14/06/2023 Da capire bene!



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

** Le istruzioni del computer SAP includevano in un byte sia l'opcode sia l'operando, come descritto anche in precedenza in questa stessa pagina:

>... un unico byte di cui i 4 Most Significant Bit (MSB) rappresentano l'opcode e di cui i 4 Least Significant Bit (LSB) sono l'operando

Introducendo gli Step, riflettevo anche sul fatto che per talune operazioni, da quanto capivo approfondendo l'NQSAP, avere più di 8 microistruzioni sarebbe stato molto util: ecco che, ancora una volta, dovevo riconsiderare il numero di bit di indirizzamento necessari per le EEPROM

Ricordiamo che "Praticamente ho due fasi:

- Fetch, in cui carico l'istruzione dalla RAM nell'Instruction Register
- Dopo la fase di Fetch so "cosa devo fare", perché a questo punto ho l'istruzione nell'Instruction Register", che attiva opportunamente le EEPROM in modo da aver in uscita i corretti segnali di Control Logic…
- 02/09/2022 … e a quel punto leggerò la locazione dell'operando, il cui contenuto
  - ○ se è una istruzione "indiretta" mi darà il valore della locazione reale da indirizzare per leggerne il contenuto o scrivervi un valore
  - ○ se è un istruzione diretta conterrà il valore da lavorare
- "Sicuramente" avrò bisogno di EEPROM più grandi, perché dovranno ospitare gli 8 MSB e gli 8 LSB attuali, ma anche altri 8 bit di segnali… 02/09/2022 ma forse anche no, come visto sopra non mi servono (per ora) altri segnali… e addirittura, come visto in seguito, forse non mi serve nemmeno IO
In seguito ho compreso il decoder 3-8 che usa Tom Nisbet per poter gestire tanti segnali con poche linee

8-bit CPU control logic: Part 2
https://www.youtube.com/watch?v=X7rCxs1ppyY

Le istruzioni sono fatte di più step, chiamati microistruzioni. La Control Logic deve settare correttamente la Control Word per ogni microistruzione così quando arriva il clock questa viene eseguita. Dobbiamo dunque sapere sempre quale istruzione stiamo eseguendo e a che step siamo. Ci serve un contatore per tracciare la microistruzione. Usiamo 74LS161 che può contare da 0 a 15.

NB: Dobbiamo settare la Control Logic tra un clock e l'altro… come a dire che la Control Logic deve "essere pronta" prima che l'istruzione venga eseguita: possiamo usare un NOT per invertire il clock e usare questo per gestire il 74LS161 della Control Logic.
	• I registri sono aggiornati al Rising Edge del CLK, che corrisponde al Falling Edge del /CLK.
	• Le microistruzioni sono aggiornate al Falling Edge del CLK, che corrisponde al Rising Edge del /CLK.
	• CLK gestisce tutti i registri principali: PC, MAR, RAM, IR, A, B, Flag: al Rising Edge del CLK, avvengono le azioni di caricamento dei registri. Quando c'è il segnale CE Counter Enable attivo, il PC viene incrementato al Rising Edge e l'indirizzo viene aumentato di uno.
	• /CLK gestisce il Ring Counter e di conseguenza la Control Logic: è sfasato di 180°, dunque al Falling Edge di CLK corrisponde il Rising Edge di /CLK
		○ All'accensione del computer
			§ PC è 0 e RC (Ring Counter) è 0
			§ la CL presenta CO|MI in uscita
			§ il 245 del PC è attivo in output
			§ il 245 del MAR è attivo in input.
		○ Arriva il Rising Edge del CLK
			§ il FF 173 del MAR carica (MI) l'indirizzo di memoria presentatogli (CO) dal PC
		○ Arriva il Falling Edge del CLK, che corrisponde al Rising Edge del /CLK
			§ RC si incrementa e la CL presenta la microistruzione successiva RO|II|CE
				□ la RAM è attiva in output
				□ IR è attivo in input
				□ PC è attivato per contare
		○ Arriva il Rising Edge del CLK
			§ il FF 173 dell'IR carica (II) il valore presentato dalla cella di RAM (RO) indirizzata dal MAR
			§ PC si incrementa (CE)
		○ Arriva il Falling Edge del CLK, che corrisponde al Rising Edge del /CLK
			§ RC si incrementa e la CL presenta la microistruzione successiva IO|AI
				□ IR mette in output
					® i 4 MSB che vanno ad indirizzare le EEPROM della CL
					® i 4 LSb che vanno sul bus; immaginiamo ad esempio istruzione immediata LDA #$05
				□ il 245 del Registro A è attivo in input
		○ Arriva il Rising Edge del CLK
			§ il FF 173 del Registro A carica (AI) il valore presentato sul bus (IO) dall'Instruction Register 

Il 74LS138 è un decoder che può prendere i 3 bit (ce ne bastano 3 per gestire 8 cicli, visto che gli step delle microistruzioni sono al massimo 6) e convertirli in singoli bit che rappresentano lo step della microistruzione corrente e poi uno di questi, l'ultimo, che resetta il 74LS161 in modo da risparmiare i cicli di clock inutilizzati.Control Logic
8-bit CPU control logic: Part 1 
https://www.youtube.com/watch?v=dXdoim96v5A

Prima cosa che facciamo è leggere un comando e metterlo nell'Instruction Register, che tiene traccia del comando che stiamo eseguendo.

• Prima fase: FETCH. Poiché la prima istruzione sta nell'indirizzo 0, devo mettere 0 dal Program Counter PC (comando CO esporta il PC sul bus) al Memory Address Register MAR (comando MI legge dal bus e setta l'indirizzo sulla RAM) così da poter indirizzare la RAM e leggere il comando.
• Una volta che ho la RAM attiva all'indirizzo zero, copio il contenuto della RAM nell'Instruction Register IR passando dal bus (comando RO e comando II).
• Poi incrementiamo il PC col comando Counter Enable CE (nel video successivo il CE viene inserito nella microistruzione con RO II).
• Ora eseguiamo l'istruzione LDA 14, che prende il contenuto della cella 14 e lo scrive in A. Dunque poiché il valore 14 sono i 4 LSB del comando, sono i led gialli dell'IR e col comando Instruction Register Out IO ne copio il valore nel bus e poi, caricando MI, indirizzo la cella di memoria 14 che è quella che contiene il valore (28 nel mio caso) che esporto nel bus col comando RAM Out RO e che caricherò in A col comando AI. 

• ADD 15 ha sempre una prima fase di Fetch, uguale per tutte le istruzioni, e poi come sopra il valore 15 è quello della cella 15 e dunque Instruction Register Out IO che posiziona sul bus i 4 LSb del comando ADD 15, poi MI così setto l'indirizzo 15 della RAM, il cui contenuto metto sul bus col comando RAM Out RO e lo carico in B con BI così avrò a disposizione la somma di A e B, che metto sul bus con EO e che ricarico in A con AI. 

• Prossima istruzione all'indirizzo 3 è OUT che mette sul display il risultato della somma che avevo latchato in A. La prima fase di Fetch è uguale alle altre. Poi faccio AO per mettere A sul bus e OI. 


8-bit CPU control logic: Part 3
https://www.youtube.com/watch?v=dHWFpkGsxOs

La fase Fetch è uguale per tutte le istruzioni, dunque istruzione XXXX e imposto solo gli step.
Per LDA uso il valore 0001 e imposto gli step con le microistruzioni opportune.
Per ADD uso il valore 0010 e imposto gli step con le microistruzioni opportune.
Per OUT uso il valore 1110 e imposto gli step con le microistruzioni opportune.

Praticamente ora abbiamo il contatore delle microistruzioni (T0-T4) e il contatore dell'istruzione (Instruction Register MSB). Posso creare una combinational logic che, a seconda dell'istruzione che ho caricato nell'Instruction Register + il T0/4 dove mi trovo mi permetta di avere in uscita i segnali corretti da applicare al computer. Praticamente ho due fasi:
• Fetch, in cui carico l'istruzione dalla RAM nell'Instruction Register
Dopo la fase di Fetch so "cosa devo fare", perché a questo punto ho l'istruzione nell'Instruction Register



Il Flags Register emula quello del 6502 con questi flag:
• Zero
• Carry
• OVerflow che non mi è chiarissimo cosa sia
• Negative

E' differente dall'8-bit computer originario, dove un unico FF '173 memorizzava entrambi i flag nello stesso momento - e dunque, ricordo qualcosa, si era ripetuta per 4 volte la programmazione delle EEPROM perché avendo due flag C ed F le combinazioni possibili sono 4 (00, 01, 10, 11) e dunque avevo bisogno di 4 set di microcode, uno per ogni combinazione degli indirizzi in ingresso C ed F. Da verificare.
		23/10/2022 In questo nuovo caso le istruzioni non variano a seconda dello stato dei flag, che non sono più Input alle ROM che poi variavano l'output in base all'indirizzo/flag presentato in ingresso! Nella configurazione sviluppata da Tom, a un certo punto nel codice si trova un'istruzione di salto condizionale legata a un flag, magari JZ: ad essa corrisponde un segnale in uscita di JUMP (uguale per tutte le istruzioni) che attiva con /E il Selector 151; la selezione del flag da mettere in uscita dipende dal microcode (i 3 bit Select del 151 sono direttamente collegati all'Instruction Register) perciò se per esempio l'istruzione di JZ Jump on Zero è 010 questo andrà a selezionare il pin I2 di ingresso del 151 che, se attivo (cioè output FF del flag Z = 1), andrà ad abilitare il PC-LOAD e permettere il caricamento del nuovo indirizzo nel PC 😎
	
	La logica del salto condizionale del SAP-1 era implementata nel microcode, utilizzando linee di indirizzamento delle ROM. Poiché i flag dell'NQSAP sono implementati in hardware, non c'è bisogno di usare linee preziose linee di indirizzamento delle ROM. Miglioramenti derivanti:
	        • è possibile settare i flag anche singolarmente
	        • risparmio delle linee di indirizzamento ROM
	        • non si modifica l'output della ROM durante l'esecuzione della singola istruzione (ma nel SAP-1 come si comportava? 04/10/2022 l'ho compreso andando a rileggere gli appunti del BE 8 bit computer). Teoricamente, e l'avevo letto anche altrove, questo potrebbe essere un problema perché causa "glitching".
	


MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE MICROCODE

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

- • Le istruzioni di Branch Relative consumano molti cicli, perciò Tom ha aggiunto anche delle istruzioni di Jump Relative. Evidenziare che ho risolto questa problematica facendo un instruction register a 16 step



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

[![Schema del modulo Control Logic](../../assets/control/40-control-logic-schema.png "Schema del modulo Control Logic"){:width="100%"}](../../assets/control/40-control-logic-schema.png)

*Schema del modulo Control Logic.*

## Note

- Attenzione : nello schema cè una led bar collegata al ring counter, Una led bar collegata alle uscite a - quattro a 11del bass delle rom, ma probabilmente qui manca un pezzettino di led bar per arrivare ai 12 indirizzi totaliindirizzatidai 12 pine in più manca la led bar del registro delle istruzioni
- Il computer NQSAP prevedeva 8 step per le microistruzioni, mentre il BEAM ne prevede 16. Come descritto in maggior dettaglio nella sezione riservate al microcode, con soli 8 step non sarebbe stato possibile emulare alcune delle istruzioni del 6502, come quelle di salto relativo ed altre. Questa è in realtà una differenza architetturale più legata alla Control Logic, però l’impatto principale sul numero di step disponibili si riflette in particolar modo sull’ALU ed ha dunque sicuramente senso citarla in questa sezione.

## Link utili

- I video di Ben Eater che descrivono la <a href = "https://eater.net/8bit/control" target = "_blank">Control Logic e il microcode</a>.

## TO DO

- aggiornare lo schema Kicad con le le bar a 8 segmenti e aggiornare questa pagina con lo schema aggiornato
