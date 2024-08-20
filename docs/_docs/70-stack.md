---
title: "Stack Pointer"
permalink: /docs/stack/
excerpt: "Stack Pointer del BEAM computer"
---
[![Stack Pointer del BEAM computer](../../assets/sp/70-beam-sp.png "Stack Pointer del BEAM computer"){:width="100%"}](../../assets/sp/70-beam-sp.png)

• Stack Pointer. Vedi '169 74LS169 4-bit Synchronous Binary Up-Down Counter. 
• Forse con un unico Counter a 4 bit posso fare uno SP da 16 indirizzi e se volessi farlo più grande me ne servirebbero due.
	• 05/01/2023 nota che Tom usa 8 bit… così forse da poter usare qualsiasi indirizzo all'interno dei 256 byte? ***
		○ 06/07/2023 direi che è un normale registro che ha in più la peculiarità di poter contare up e down, ma se lo uso come indice dello stack, lo devo inizializzare, magari a 240 ($F0).
• NB il 169 non ha un reset e all'accensione si trova dunque in uno stato ignoto, pertanto se si usa lo stack si dovrà pulirlo all'inizio del programma. Bisogna creare una istruzione tipo "CSP", Clear StackPointer… 06/07/2023 o meglio all'inizio lo inizializzo con un valore arbitrario, vedi sopra.
• Istruzioni da creare:
	• PHA, PLA
	• JSR, RTS
	• TXS, TSX -- NB TXS potrebbe fare al posto del CSP.

• Come teoria, lo Stack Pointer punta sempre al primo "spazio libero" disponibile, dunque:
	• in scrittura prima si scrive il valore (ad esempio indirizzo di ritorno della subroutine) nello stack e poi si incrementa lo stack (post-increment)
	• in lettura PRIMA si decrementa lo stack e poi lo si legge (pre-decrement)

• Funzionamento dell'SP: immagino che se faccio ad esempio un JSR, si debba (rivisto un po' il 02/07/2023 dunque ora  dovrebbe essere corretto, mentre in passato avevo corretto con l'intervento successivo del 05/01/2023 più sotto… mmm ma ancora non mi convince):
	• riservare un'area di memoria, ad esempio con 256 byte e un Counter a 4 bit, riservo gli ultimi 16 byte ***
	• leggere il valore attuale dell'SP (se serve)
	• incrementarlo ==> no, va fatto in seguito (post-increment)
	• leggere il PC attuale
	• mettere nella cella indicata dall'SP (ad esempio 240) il valore del PC… 05/01/2023 il PC deve essere prima incrementato e deve arrivare a puntare al primo byte dell'istruzione successiva, cioè all'opcode: l'RTS deve infatti tornare a Istruzione + Operando + 1
	• caricare il valore del JSR in PC
	• saltare all'indirizzo del JSR
	• Step necessari:
		○ CO MI metto il PC nel MAR e indirizzo la RAM per leggere l'istruzione
		○ RO II CE leggo l'istruzione e la metto nell'IR e incremento il PC
		○ CO MI metto il PC nel MAR e indirizzo la RAM per leggere l'operando (valore JSR)
		○ RO HI CE metto il valore del JSR in registro temporaneo H ** e incremento il PC così punterà alla nuova istruzione
		○ CO SPI metto il PC nello Stack Pointer… 02/07/2023 ecco l'errore: qui manca la lettura del valore dell'SP… come faccio a scrivere sullo stack se non so a quale indirizzo? dunque dovrebbe essere SPO MI
		○ Added 02/07/2023 CO RI dunque scrivo il valore del PC "istruzione + operando + 1" sulla cella di memoria dello stack, che sarà il valore al quale farò riferimento con RTS
		○ INS CE incremento SP e PC
		○ HO PC e trasferisco valore del JSR da registro temporaneo H a PC
		○ J eseguo il Jump


• Rifaccio 05/01/2023
	• riservare un'area di memoria, ad esempio con 256 byte e un Counter a 16 bit, ma mi tengo solo gli ultimi 16 byte ***
	• CO MI - RO II CE - Leggere l'istruzione, caricarla nell'IR e incrementare il PC
	• CO MI - RO BI - Leggere l'operando (indirizzo JSR), copiarlo in registro temporaneo B
	• RS MI CE - Leggere il valore attuale dell'SP e metterlo nel MAR così da indirizzare la RAM; incrementare il PC (così il PC sarà già quello giusto per l'RTS, che deve tornare a Istruzione + Operando + 1)
	• CO RI - Copiare il PC nella cella indicata dall'SP (ad esempio 240); in questo momento il valore del PC è già quello corretto "Istruzione + Operando + 1"
	• BO WP Leggere il valore dell'operando (indirizzo JSR) da registro temporaneo B e copiarlo in PC
	• SI J - Incrementare lo stack e saltare all'indirizzo del JSR… Non è necessario saltare, perché sul PC è stato scritto l'indirizzo a cui deve puntare, perciò il prossimo Counter Output CO sarà già quello giusto 
• Bene quasi tutto rispetto a quanto avevo scritto precedentemente, cercando di seguire il ragionamento di Tom su cosa bisognava fare. Mi manca solo che SI era stato incorporato nello step precedente e che non vedo il Jump… ora controllo sul codice di Tom

** NB Tom usa B e non H come temporaneo
	
• Al contrario RTS farà:
	• leggere il valore dell'SP
	• scrivere in PC il valore contenuto nella cella indicata dall'SP
	• decrementare l'SP

• Nell'NQSAP Tom segnala che si potrebbe usare anche un '193, ma con attenzione perché non ha il Count Enable e dunque bisogna usare una porta esterna per far arrivare un clock al contatore solo quando il segnale di Enable è attivato dalla ROM e dunque usa dei 169; attenzione ai glitch della ROM, dice… 05/01/2023 descritti in seguito
	
Lo schema seguente fa riferimento a NQSAP-PCB; non era stato pubblicato per l'NQSAP.

**Inserire qui lo schema del nQsap PCB**

	• Successivamente, nell'NQSAP-PCB, ritorna sui suoi passi ed utilizza proprio un '193, che conta inviando un segnale a UP o DOWN del 193 abilitando SE Stack Enable insieme a C0 o C1 per definire la direzione del conteggio, dei quali viene fatto l'AND con il CLK.

	• WS, sempre ANDed con CLK, permette di caricare un valore specifico nello stack.
	• Si usano C0 e C1 per risparmiare segnali in uscita dalle EPROM… che clever questo tipo
		○ C0 e C1 sono infatti condivisi con il Flag Register per la selezione del Carry in ingresso da ALU o H, dunque non bisogna modificare il CF nella stessa microistruzione di conteggio dell'SP.
		○ In NQSAP-PCB dice che C0 e C1 sono anche condivisi con DY e DZ…
			§ forse perché aveva pochi segnali disponibili ora che ha ridotto le ROM da 4 a 3… e infatti nella Control Logic a 3 ROM ci sono solo C0 e C1, che vanno sugli stessi pin del bus 26 e 27 di DY e DZ dei registri X e Y… 😁
dunque nemmeno istruzioni SP e X/Y possono avvenire nella stessa microistruzione (se consolido C0 e C1 con DY e DZ).

	• Esempio istruzione JSR, descritto in NQSAP-PCB:
	Address	Contents
	…	previous instruction / operand
	20	JSR opcode
	21	JSR address
	22	next instruction
	…	

		○ After the instruction fetch CO MI CE / RO II, the PC will have the value 21 and JSR microcode performs the following steps:
			§ CO MI CE move the PC value to the MAR and increment the PC. MAR contains 21 and PC contains 22.
			§ RO BI read the subroutine address from RAM[21] and place it in B for temp storage
			§ SPO MI SPI move the SP value into the MAR and increment the SP.
			§ CO RI store the PC value (which points to the next instruction) in memory, i.e. push the JSR return address on the stack.
BO PI move the B register value into the PC, effectively jumping to the subroutine

	Add	Val		Alla fine del JSR:
	$20	JSR		1) Devo avere $22 nella RAM $F0
	$21	$30		2) L'SP deve essere $F1
	$22	INX		3) PC deve passare a $30

	In giallo quello che cambia rispetto allo step precedente



							Esercizio Andrea					
							Alla fine degli step…					
							PC	SP	MAR	RAM	IR	B
		Step				Codice Tom	$20	$F0	$1F	?	?	?
	Fetch	0	CO	MI			$20	$F0	$20	JSR	?	?
	Fetch	1	RO	II	CE		$21	$F0	$20	JSR	JSR	?
	Exec 1	2	CO	MI		RP WM PI	$21	$F0	$21	$30	JSR	?
	Exec 2	3	RO	WB		RR WB	$21	$F0	$21	$30	JSR	$30
	Exec 3	4	RS	MI	CE	RS WM	$22	$F0	$F0	DC	JSR	$30
	Exec 4	5	CO	WM		RP WR SI	$22	$F0	$F0	$22	JSR	$30
	Exec 5	6	SI	BO	WP	RB WP N	$30	$F1	$F0	DC	JSR	$30

	Step	NB queste sono le mie considerazioni
	Fetch	Adesso so che sto facendo una JSR
	Exec 1	Metto nel MAR l'indirizzo della cella che contiene l'indirizzo a cui devo saltare
	Exec 2	Leggo l'indirizzo a cui devo saltare dalla cella puntata dal MAR e lo scrivo temporaneamente in B
	Exec 3	Leggo lo stack (che punta alla prima cella di memoria disponibile) e lo metto nel MAR; incremento il PC per ottenere il valore di "ritorno" dell'RTS
	Exec 4	Scrivo il valore di ritorno dell'RTS nella cella di memoria indicata dallo stack
	Exec 5	Incremento lo stack e scrivo B nel PC, così al prossimo clock eseguo il codice presente nell'indirizzo a cui dovevo saltare

							Tom Nisbet					
							Alla fine degli step…					
							PC	SP	MAR	RAM	IR	B
		Step				Codice Tom	$20	$F0	$1F	?	?	?
	Fetch	0	CO	MI			$20	$F0	$20	JSR	?	?
	Fetch	1	RO	II	CE		$21	$F0	$20	JSR	JSR	?
	Exec 1	2	CO	MI	CE	RP WM PI	$22	$F0	$21	$30	JSR	?
	Exec 2	3	RO	WB		RR WB	$22	$F0	$21	$30	JSR	$30
	Exec 3	4	RS	MI	SI	RS WM ***	$22	$F1	$F0	DC	JSR	$30
	Exec 4	5	CO	WM		RP WR SI	$22	$F1	$F0	$22	JSR	$30
	Exec 5	6	BO	WP		RB WP N	$30	$F1	$F0	DC	JSR	$30

	Fetch	Adesso so che sto facendo una JSR
	Exec 1	Metto nel MAR l'indirizzo della cella che contiene l'indirizzo a cui devo saltare e incremento il PC per ottenere il valore di "ritorno" dell'RTS
	Exec 2	Leggo l'indirizzo a cui devo saltare e lo registro temporaneamente in B
	Exec 3	Leggo lo stack (che punta alla prima cella di memoria disponibile), lo metto nel MAR e lo incremento *** 05/01/2023 Nota che poi il codice di Tom non incrementa l'SP in questo step, ma in quello successivo, forse perché se faccio RS e SI contemporaneamente potrei avere un errore in lettura?
	Exec 4	Scrivo il valore di ritorno dell'RTS nella cella di memoria indicata dallo stack
	Exec 5	Scrivo B nel PC, così al prossimo clock eseguo il codice presente nell'indirizzo a cui dovevo saltare


![Alt text](image-1.png)


**PLACEHOLDER**


[![Schema dello Stack Register del computer BEAM](../../assets/sp/70-stack-pointer-schema.png "Schema dello Stack Register del computer BEAM"){:width="100%"}](../../assets/sp/70-stack-pointer-schema.png)

*Schema dello Stack Register del computer BEAM.*

## Link utili

- [Inside the vintage 74181 ALU chip: how it works and why it's so strange](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) di Ken Shirriff. Fondamentale per capire il perché dell'implementazione apparentemente così strana del chip.
- [Demo of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=Fq0MIJjlGsw) e [Comparator Functions of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=jmROTNtoUGI): due ottimi video di David Courtney.

## TO DO

- /WE ↘↗
- controllare se il mio SP funziona correttamente; vedo che Tom ha usato dei cablaggi diversi per i pin 7 e 10   20-08-2024
