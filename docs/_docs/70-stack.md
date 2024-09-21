---
title: "Stack Pointer"
permalink: /docs/stack/
excerpt: "Stack Pointer del computer BEAM"
---
[![Stack Pointer del computer BEAM](../../assets/sp/70-beam-sp.png "Stack Pointer del computer BEAM"){:width="100%"}](../../assets/sp/70-beam-sp.png)

## - WORK IN PROGRESS

Lo Stack Pointer permette di salvare informazioni in un'area di memoria e ripristinarle nell'ordine inverso rispetto a quello al quale sono state inserite. Due casi d'uso comune sono il salvataggio dello stato attuale dei Flag prima di eseguire una routine che li modifica, oppure la memorizzazione dell'indirizzo di ritorno dell'istruzione di salto a sub-routine (JSR), magari anche annidata.

Lo Stack Pointer del BEAM utilizza due 4-bit Synchronous Binary Up-Down Counter <a href="https://www.ti.com/lit/ds/symlink/sn74ls157.pdf" target="_blank">74LS169</a>, in grado di indirizzare i 256 byte del computer.

Sostanzialmente si tratta di un normale registro che ha in più la peculiarità di poter contare up e down.

Quando utilizzato come indice dello stack deve essere inizializzato, magari a 240 ($F0).

Lo stack del 6502 occupa la seconda pagina di memoria indirizzabile dalla CPU, cioè quella che va dall'indirizzo 0x0100 a 0x01FF; il puntatore (Stack Pointer) punta per default all'ultimo indirizzo, cioè 0x1FF; nel momento in cui lo Stack viene caricato di un valore, la locazione di memoria più alta viene scritta con tale valore e il puntatore viene decrementato, pronto per puntare alla prossima locazione di memoria immediatamente più bassa. Viceversa per la lettura: il puntatore punta all'indirizzo più basso utilizzato e alla lettura del dato il puntatore viene incrementato, pronto per puntare alla locazione di memoria immediatamente più alta.

Poiché i chip '169 utilizzati nel BEAM non presentano un ingresso Reset, all'accensione potrebbero trovarsi in uno stato ignoto, pertanto lo stack dovrà essere inizializzato dal programma caricato in memoria dall'utente.

Le istruzioni del 6502 che utilizzano lo stack:

- PHA, PLA
- JSR, RTS
- TXS, TSX -- NB TXS potrebbe fare al posto del CSP.

Come teoria, lo Stack Pointer punta sempre al primo "spazio libero" disponibile, dunque:

- un'operazione di scrittura dello stack scrive prima il valore da salvare nello spazio libero dello stack (il valore potrebbe essere ad esempio l'indirizzo di ritorno dell'istruzione JSR) e poi decrementa lo stack (post-decrement);
- un'operazione di lettura prima incrementa decrementa lo Stack Pointer e poi legge la nuova locazione di memoria indirizzata dal puntatore (pre-increment).

Funzionamento dell'SP:

immagino che se faccio ad esempio un JSR, si debba (rivisto un po' il 02/07/2023 dunque ora  dovrebbe essere corretto, mentre in passato avevo corretto con l'intervento successivo del 05/01/2023 più sotto… mmm ma ancora non mi convince):

- decidere quale area di memoria del computer riservare allo Stack, ad esempio gli ultimi 16 byte da 0xF0 a 0xFF \*\*\*
- ad ogni inizio del programma eseguito, inizializzare lo stack al valore di partenza (poiché lo Stack si riempie dall'alto verso il basso, 0xFF) 

leggere il valore attuale dell'SP (se serve)

- incrementarlo ==> no, va fatto in seguito (post-increment)
- leggere il PC attuale

- mettere nella cella indicata dall'SP (ad esempio 240) il valore del PC… 05/01/2023 il PC deve essere prima incrementato e deve arrivare a puntare al primo byte dell'istruzione successiva, cioè all'opcode: l'RTS deve infatti tornare a Istruzione + Operando + 1
- caricare il valore del JSR in PC
- saltare all'indirizzo del JSR
- Step necessari:

- RPC|WM CO MI metto il PC nel MAR e indirizzo la RAM per leggere l'operando (valore JSR)
- RR|WB|PCI    metto il valore del JSR in registro temporaneo B e incremento il PC così punterà alla nuova istruzione, CIOè incrementare il PC (così il PC sarà già - quello giusto per l'RTS, che deve tornare a Istruzione + Operando + 1)
- RS|WM          Leggere il valore attuale dell'SP e metterlo nel MAR così da indirizzare la RAM; 
- RPC|WR        Copiare il PC nella cella indicata dall'SP (ad esempio 240); in questo momento il valore del PC è già quello corretto "Istruzione + Operando + 1"
- SE
- RB|WPC|NI

Il BEAM implementa l'istruzione JSR così:

RPC|WM,       RR|WB|PCI,    RS|WM,        RPC|WR,       SE,           RB|WPC|NI,

- CO MI metto il PC nel MAR e indirizzo la RAM per leggere l'istruzione
- RO II CE leggo l'istruzione e la metto nell'IR e incremento il PC

- CO SPI metto il PC nello Stack Pointer… 02/07/2023 ecco l'errore: qui manca la lettura del valore dell'SP… come faccio a scrivere sullo stack se non so a quale indirizzo? dunque dovrebbe essere SPO MI
- Added 02/07/2023 CO RI dunque scrivo il valore del PC "istruzione + operando + 1" sulla cella di memoria dello stack, che sarà il valore al quale farò riferimento con RTS
- INS CE incremento SP e PC
- HO PC e trasferisco valore del JSR da registro temporaneo H a PC
- J eseguo il Jump

- Rifaccio 05/01/2023

- riservare un'area di memoria, ad esempio con 256 byte e un Counter a 16 bit, ma mi tengo solo gli ultimi 16 byte ***
- CO MI - RO II CE - Leggere l'istruzione, caricarla nell'IR e incrementare il PC
- BO WP Leggere il valore dell'operando (indirizzo JSR) da registro temporaneo B e copiarlo in PC
- SI J - Incrementare lo stack e saltare all'indirizzo del JSR… Non è necessario saltare, perché sul PC è stato scritto l'indirizzo a cui deve puntare, perciò il prossimo Counter Output CO sarà già quello giusto 
• Bene quasi tutto rispetto a quanto avevo scritto precedentemente, cercando di seguire il ragionamento di Tom su cosa bisognava fare. Mi manca solo che SI era stato incorporato nello step precedente e che non vedo il Jump… ora controllo sul codice di Tom

• Al contrario RTS farà:

- leggere il valore dell'SP
- scrivere in PC il valore contenuto nella cella indicata dall'SP
- decrementare l'SP

• Nell'NQSAP Tom segnala che si potrebbe usare anche un '193, ma con attenzione perché non ha il Count Enable e dunque bisogna usare una porta esterna per far arrivare un clock al contatore solo quando il segnale di Enable è attivato dalla ROM e dunque usa dei 169; attenzione ai glitch della ROM, dice… 05/01/2023 descritti in seguito

Lo schema seguente fa riferimento a NQSAP-PCB; non era stato pubblicato per l'NQSAP.

**Inserire qui lo schema del nQsap PCB**

- Successivamente, nell'NQSAP-PCB, ritorna sui suoi passi ed utilizza proprio un '193, che conta inviando un segnale a UP o DOWN del 193 abilitando SE Stack Enable insieme a C0 o C1 per definire la direzione del conteggio, dei quali viene fatto l'AND con il CLK.

- WS, sempre ANDed con CLK, permette di caricare un valore specifico nello stack.
- Si usano C0 e C1 per risparmiare segnali in uscita dalle EPROM… che clever questo tipo
  - C0 e C1 sono infatti condivisi con il Flag Register per la selezione del Carry in ingresso da ALU o H, dunque non bisogna modificare il CF nella stessa microistruzione di conteggio dell'SP.
  - In NQSAP-PCB dice che C0 e C1 sono anche condivisi con DY e DZ…
    - forse perché aveva pochi segnali disponibili ora che ha ridotto le ROM da 4 a 3… e infatti nella Control Logic a 3 ROM ci sono solo C0 e C1, che vanno sugli stessi pin del bus 26 e 27 di DY e DZ dei registri X e Y… 😁
dunque nemmeno istruzioni SP e X/Y possono avvenire nella stessa microistruzione (se consolido C0 e C1 con DY e DZ).

- Esempio istruzione JSR, descritto in NQSAP-PCB:
	Address	Contents
	…	previous instruction / operand
	20	JSR opcode
	21	JSR address
	22	next instruction
	…	

    - After the instruction fetch CO MI CE / RO II, the PC will have the value 21 and JSR microcode performs the following steps:
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

## Schema

[![Schema dello Stack Register del computer BEAM](../../assets/sp/70-stack-pointer-schema.png "Schema dello Stack Register del computer BEAM"){:width="100%"}](../../assets/sp/70-stack-pointer-schema.png)

*Schema dello Stack Register del computer BEAM.*

## Link utili

- [Inside the vintage 74181 ALU chip: how it works and why it's so strange](https://www.righto.com/2017/03/inside-vintage-74181-alu-chip-how-it.html) di Ken Shirriff. Fondamentale per capire il perché dell'implementazione apparentemente così strana del chip.
- [Demo of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=Fq0MIJjlGsw) e [Comparator Functions of 74LS181 (74HCT181) ALU](https://www.youtube.com/watch?v=jmROTNtoUGI): due ottimi video di David Courtney.

## TO DO

- /WE ↘↗
- controllare se il mio SP funziona correttamente; vedo che Tom ha usato dei cablaggi diversi per i pin 7 e 10   20-08-2024
