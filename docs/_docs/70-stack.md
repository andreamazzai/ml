---
title: "Stack Pointer"
permalink: /docs/stack/
excerpt: "Stack Pointer del computer BEAM"
---
[![Stack Pointer del computer BEAM](../../assets/sp/70-beam-sp.png "Stack Pointer del computer BEAM"){:width="100%"}](../../assets/sp/70-beam-sp.png)

## - WORK IN PROGRESS

Lo Stack Pointer (SP) permette di salvare informazioni in un'area di memoria e ripristinarle nell'ordine inverso rispetto a quello al quale sono state inserite (LIFO, o Last-In, First-Out). Due casi d'uso comune sono il salvataggio dello stato attuale dei Flag prima di eseguire una routine che li modifica per poi ripristinarne lo stato iniziale, oppure la memorizzazione dell'indirizzo di ritorno dell'istruzione di salto a sub-routine (JSR), magari anche annidata.

Lo Stack Pointer del BEAM utilizza due 4-bit Synchronous Binary Up-Down Counter <a href="https://www.ti.com/lit/ds/symlink/sn74ls157.pdf" target="_blank">74LS169</a>, in grado di indirizzare i 256 byte del computer. In definitiva, si tratta di un normale registro che include la peculiarità di poter anche contare sia verso l'alto, sia verso il basso.

// Quando utilizzato come indice dello stack deve essere inizializzato, magari a 240 ($F0).

Nel 6502, lo stack occupa la seconda pagina di memoria indirizzabile dalla CPU, cioè l'intervallo di indirizzi che va da 0x0100 a 0x01FF. Prima dell'uso dello stack, lo Stack Pointer viene solitamente inizializzato a 0xFF, col risultato di puntare dunque alla cella 0x1FF. Quando si scrive nello stack ("Push"), il valore viene prima scritto nella locazione indicata dal puntatore e successivamente lo SP viene decrementato per poter puntare alla prossima locazione libera. Viceversa, durante un'operazione di lettura ("Pull"), il valore viene prima letto dalla locazione indicata dal puntatore e poi quest'ultimo viene incrementato.

In altre parole, poiché lo stack "cresce" verso il basso, lo Stack Pointer viene decrementato dopo che ogni byte viene inserito nello stack, e incrementato prima che ogni byte venga prelevato dallo stack.

I '169 utilizzati nel BEAM non presentano un ingresso Reset, pertanto, all'accensione potrebbero trovarsi in uno stato ignoto. Anche in questo caso, pertanto lo stack dovrà essere inizializzato dal programma caricato in memoria dall'utente.

Le istruzioni del 6502 che utilizzano lo stack:

- PHA, PLA
- JSR, RTS
- TXS, TSX -- NB TXS potrebbe fare al posto del CSP.

Lo Stack Pointer (SP) punta sempre alla successiva locazione disponibile nello stack, dunque:

- un'operazione di scrittura sullo stack decrementa prima lo SP (pre-decrement) e poi scrive il valore nella nuova locazione di memoria indicata dal puntatore (ad esempio, l'indirizzo di ritorno dell'istruzione JSR);
- un'operazione di lettura dallo stack restituisce prima il contenuto della locazione di memoria indicata dal puntatore e poi incrementa lo SP (post-increment).

Funzionamento dell'SP:

Sulla base di questo, analizziamo ad esempio una istruzione JSR ipotizzando di aver inizializzato lo SP del BEAM a 0xFF e Ricordando che il BEAM ha solamente 256 byte di memoria e che dunque lo stack pointer inisterizzato in questo modo punta all'ultima locazione di memoria del computer .

~~~text
| ---- | ---------------------|
| Step | Microistruzione      |
| ---- | ---------------------|
| 1    | RPC | WM             |
| 2    | RR  | WIR | PCI      |
| 3    | RPC | WM             |
| 4    | RR  | WB   | PCI     | 
| 5    | RS  | WM             |
| 6    | RPC | WR             |
| 7    | SE                   |  
| 8    | RB  | WPC  | NI      | 
| ---- | ---------------------|
~~~

*Scomposizione dell'istruzione JSR nelle sue otto microistruzioni elementari*.

1. Il primo step carica l'indirizzo del Program Counter nel Memory Address Register:
    - RPC, Read Program Counter - espone sul bus l'indirizzo del Program Counter
    - WM, Write Memory Address Register - scrive il contenuto del bus nel MAR
2. Il secondo step carica l'opcode dell'istruzione nell'IR e incrementa il PC per farlo puntare alla locazione di memoria successiva (che nel caso dell'istruzione NOP, lunga un solo byte, sarà la prossima istruzione):
    - RR, Read RAM - espone sul bus il contenuto della locazione di memoria puntata dal MAR
    - WIR, Write Instruction Register - scrive il contenuto del bus nell'Instruction Register
    - PCI, Program Counter Increment - incrementa il Program Counter
3. Il terzo step carica nel Memory Address Register l'indirizzo del Program Counter, che ora punta all'Operando:
    - RPC, Read Program Counter - espone sul bus l'indirizzo del Program Counter
    - WM, Write Memory Address Register - scrive il contenuto del bus nel MAR
4. Il quarto step legge l'operando, che punta alla destinazione del salto dell'istruzione JSR, lo salva in B e incrementa il PC che va ora a puntare la prossima prossima istruzione, che coincide con l'indirizzo di ritorno dalla subroutine
    - RR, Read RAM - espone sul bus il contenuto della locazione di memoria puntata dal MAR
    - WB, Write B - ????????????????????????????????????????????
    - PCI, Program Counter Increment - incrementa il Program Counter
6. Il sesto step carica l'indirizzo attuale al quale punta lo Stack Pointer nel Memory Address Registert
    - RS, Read Stack - ??????????????????????
    - WM, Write Memory Address Register - scrive il contenuto del bus nel MAR
6. Il sesto step carica nello stack l'indirizzo del PC, al quale il computer ritornerà alla fine della subroutine invocata dall'istruzione JSR
    - RPC, Read Program Counter - espone sul bus l'indirizzo del Program Counter
    - WR, Write RAM - ??????????????????????
7. Il settimo step abilita l'aggiornamento del PC al prossimo Rising Edge del clock
    - SE, Stack Enable - ??????????????????????
8. L'ottavo step carica nel PC l'indirizzo della subroutine invocata dall'iastruzione JSR
    - RB, Read B - ??????????????????????
    - WPC, Write Program Counter - ??????????????????????
    - NI, Next Instruction - resetta il Ring Counter

step legge l'operando, che punta alla destinazione del salto dell'istruzione JSR, lo salva in B e incrementa il PC che va ora a puntare la prossima prossima istruzione, che coincide con l'indirizzo di ritorno dalla subroutine


Al contrario RTS farà:

- leggere il valore dell'SP
- scrivere in PC il valore contenuto nella cella indicata dall'SP
- decrementare l'SP

## Lo Stack Pointer dell'NQSAP

Nella documentazione dell'NQSAP Tom segnala che si potrebbe usare anche un '193, ma con attenzione perché non ha il Count Enable e dunque bisogna usare una porta esterna per far arrivare un clock al contatore solo quando il segnale di Enable è attivato dalla ROM e dunque usa dei 169; attenzione ai glitch della ROM, dice… 05/01/2023 descritti in seguito

Poiché Tom non aveva puvvblicato lo schema dello Stack Pointer dell'NQSAP, lo sostituiamo con quello dell'NQSAP-PCB.

[![Schema dello Stack Register del computer NQSAP-PCB](../../assets/sp/70-stack-nqsap-pcb.png "Schema dello Stack Register del computer NQSAP-PCB"){:width="100%"}](../../assets/sp/70-stack-nqsap-pcb.png)

*Schema dello Stack Register del computer NQSAP-PCB.*

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

## PLACEHOLDER - PLACEHOLDER

## Schema

[![Schema dello Stack Register del computer BEAM](../../assets/sp/70-stack-pointer-schema.png "Schema dello Stack Register del computer BEAM"){:width="100%"}](../../assets/sp/70-stack-pointer-schema.png)

*Schema dello Stack Register del computer BEAM.*

## Link utili

- [Stack definition and basics](https://wilsonminesco.com/stacks/basics.html) di Garth Wilson. Garth offre una serie di compendi incredibilmente utili su tutto ciò che riguarda il 6502 ed affini.
- ??????????????????????
- ??????????????????????

## TO DO

- /WE ↘↗
- controllare se il mio SP funziona correttamente; vedo che Tom ha usato dei cablaggi diversi per i pin 7 e 10   20-08-2024
