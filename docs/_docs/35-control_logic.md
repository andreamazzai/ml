---
title: "Control Logic"
permalink: /docs/control/
excerpt: "Costruzione del della Control Logic del BEAM computer"
---
21/09/2022 - Flags and Conditional Jumps NQSAP	https://tomnisbet.github.io/nqsap/docs/flags/
Bisogna dire che più volte, leggendo il blog di Tom Nisbet, ho trovato delle idee molto clever.

Questo non è il disegno originale dell'autore, ma il mio rivisto.
		• 04/10/2022 Ho spostato le porte di collegamento tra IR e ALU-S0 nel foglio della Control Logic.

La realizzazione del comuter SAP mi ha permesso finalmente di capire cosa sia il microcode di un computer moderno.

È piuttosto comune leggere ad esempio che è necessario aggiornare il bios dei server per indirizzare falle di sicurezza che sono state scoperte e che potrebbero essere utilizzate dagli hacker per puntini puntini puntini nuovo paragrafo
Non capendo come potesse essere aggiornata una CPU, dal momento che si tratta di un componente non programmabile virgola non riuscivo a comprendere come fosse possibile arginare i problemi di sicurezza; con il microcode ho capito










Ritornando alla dimensione delle EEPROM da utilizzare per il microcode, nei miei appunti trovo traccia di diverse revisioni, ad esempio:

- mi servono EEPROM 28C64 per avere 256 (8 bit) istruzioni + 3 step + 2 flag, ma dimenticavo che avendo due ROM gemelle dovevo gestirne anche la selezione e dunque aggiungere un ulteriore bit, pertanto mi servirebbero delle 28C128;
- avrei potuto però ridurre il numero di istruzioni a 64, dunque mi sarebbero bastati 6 bit per indirizzarle e ridurre così il numero totale di indirizzi richiesti...
- ... ma forse mi sarebbero serviti altri segnali di controllo oltre ai 16  disponibili in due ROM e dunque me ne sarebbe servita una terza... e dunque due bit di indirizzamento

Posso sicuramente dire che avevo le idee ancora confuse.

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

| Segnale | Operazione | Descrizione |
| ------  | ---------- | ----------- |
| CO | Counter Output           | Il contenuto del Program Counter viene esposto sul bus            |
| MI | MAR In                   | Il contenuto del bus viene caricato nel Memory Address Register   |
| RO | RAM Output               | Il contenuto della RAM viene esposto sul bus                      |
| II | Instruction Register In  | Il contenuto del bus viene caricato nell'Instruction Register     |
| CE | Counter Enable           | Il Program Counter viene incrementato                             |
| BI | B Register In            | Il contenuto del bus viene caricato nel registro B                |
| AI | A Register In            | Il contenuto del bus viene caricato nel registro A                |
| EO | Sum Out                  | L'adder computa A+B e il suo risultato viene esposto sul bus      |

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
	
FLAG e gestione	• Un multiplexer '157 permette di copiare i valori dei flag dalla memoria (tipo istruzione Pull Processor Status PLP). Questi FF non vengono mai pre-settati dunque /Preset resta sempre collegato a Vcc (e dunque mai attivo), mentre hanno invece collegamento al reset generale del sistema /RST.
	• I FF '74 hanno come ingresso le uscite del MUX '157, che prende 4 segnali scegliendone la provenienza:
	        1. dal bus (tranne il flag Negative, che viene sempre preso direttamente dalla linea D7 del bus, perché i numeri negativi Signed hanno l'uno iniziale sull'MSB 😁)
	                a. 24/10/2022 ma… questo significa allora che NQSAP lavora in complemento di due? Devo chiedere a Tom… no, la domanda giusta è se il computer lavora con i numeri Signed o Unsigned!
	                b. 09/01/2023 in realtà il 74181 lavora in 2C
	                c. 06/01/2024 meglio dire, che sono io a decidere come lavorare, col programma che faccio girare.
	        2. dai segnali C, Z e V
	                a. C con il Data Selector / Multiplexer /151
	                b. Z col comparatore '588:
	                c. V con il Data Selector / Multiplexer /151
	• I 4 flag FC, FV, FZ, FN sono attivati grazie a segnali dedicati che vengono ANDati col CLK (caricamento registri al Rising Edge del clock). Si segnala che in questo modo una singola istruzione può settare più di un flag alla volta, ad esempio ADC potrebbe settarli tutti (in effetti l'istruzione ADC impatta tutti i flag contemporaneamente).
	• Le istruzioni CLC, CLV e SEC possono settare o pulire i flag senza usare linee ROM; infatti la ALU può mettere in uscita tutti 0 o tutti 1 e così possiamo pulire o settare il bus e caricare solo il flag che ci interessa mandando il flag apposito.
	• Un '245 permette di esportare i Flag su bus per salvarli in una location di memoria con Push Processor Status (PHP).
	
	
Salto condizionale	• Quando un flag veniva variato nel SP-1, venivano cambiati gli indirizzi delle ROM per presentare una logica opportunamente diversa in uscita. In questo modo, ad ogni flag corrispondevano anche delle linee di indirizzamento "rubate" alle ROM.
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
	
	
	
	
• Calcolo dei Flag	• Il flag Negative è semplicemente il MSB del bus 😁 (06/01/2023 direi perché sto ragionando con numeri Signed). Interessante che essendo mappato sul bus e non direttamente sull'ALU, potrei rilevare un Negative anche in contesti esterni all'ALU, ad esempio uno shift del shift-register o un trasferimento di dato da un registro a un altro.
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
