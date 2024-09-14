---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)
	1. Praticamente sta dicendo che il 74LS273 non ha nessun ingresso di Enable, mentre a noi serve anche un ingresso del genere, come si vede nello schema del MAR: il /MI serve infatti per scrivere sul MAR l'indirizzo che ci interessa solo quando ci serve; dopo l'attivazione di /MI, il CLK successivo registra nel FF il valore passato dal bus. Senza un Enable, il 273 andrebbe a "registrare" il dato in ingresso ogni volta che c'è un ciclo di clock. Io non ho 16 bit di indirizzi, ma solo 8, dunque sarebbe sufficiente aggiungere un altro FF 74LS173 e un altro MUX 74LS157 (02/09/2022 ed è così che ho "progettato", ora che rileggo).
	2. Per restare sul FF 74LS273, si potrebbe usare una AND che prenderebbe in ingresso /MI e CLK e avrebbe l'output all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, ci fosse anche l'/MI (verificare se i segnali devono essere LO o HI). Questo chip ha il CLR, che nel MAR è richiesto.
		a. Altra ipotesi: usare il 74LS377, che ha 8 FF con Enable… direi che non si può fare, perché nel MAR mi serve anche il reset (CLR) e questo chip non ce l'ha.
2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)
	1. Verificare che counter utilizzare e capire differenza fra Ripple Mode Carry e Carry Look Ahead.
	2. Intanto 22/08/2022 ho approfondito un po' la questione dell'RCA, vedi pagina OneNote che ho fatto appositamente
	3. Il Carry del 74LS161 si accende al conteggio 15.
	4. 23/08/2022 non ho trovato nessun Counter a 8 bit che avesse il LOAD, così come ce l'ha invece il 74LS161, dunque ne metterò due in cascata.
3. Instruction register - now sh!t gets fun. Since i want to bomb completely, I am considering having a 3 byte Instruction register in a way mimicking the BIU block & instruction queue in a 8086. The idea here is to split the instruction register in 3 bytes. 1 byte would be for the instruction/opcode alone, the second and third are going to hold the data of the instruction (either address or an immediate value). This would allow you to address the entire memory space. Also instructions can be of different size. For example. OUT (move to out register) would be just 1 byte wide, LDI (load immediate) would be 2 bytes and LDA (load address/absolute) would be 3 bytes wide. The control logic would take care of the fetch cycle. Or in other words since you know the instruction you are execution you would know how much bytes the instruction is and thus fetch either 1, 2 or 3 bytes from the RAM.
	1. 22/08/2022 Dopo aver letto questo ho raccolto qui sotto i pensieri per la mia idea "Come fare l'espansione di memoria a 256 byte"… ma 28/11/2022 questo link non funziona più
4. Address bus - the final thing i want is to have a dedicated address bus. Only the MAR, PC and IR would be connected to it. (Note: PC and IR would also be connected to the data bus to also allow data access from the PC or the Immediate value contained in 2nd byte of the IR)
	1. 23/08/2022 Ho chiesto:
	Hi! I'm a bit late, I know 😉! I started a few months ago to build the 8-bit computer and I'm working right now to RAM expansion 😀.
	Regarding this:
	Address bus - the final thing i want is to have a dedicated address bus. Only the MAR, PC and IR would be connected to it. (Note: PC and IR would also be connected to the data bus to also allow data access from the PC or the Immedate value contained in 2nd byte of the IR)
	Wouldn't it be OK to NOT connect IR to the data bus? After all, when you read the second byte of the instruction (in example LDI 0x30) you are going to "use" 0x30 immediately, such as placing it into into Register A.
	Is there a specific reason for placing the operand data into IR and then moving it into A? Wouldn't it be possible to do RO and AI and "bypass" the Instruction Register?
	From <https://www.reddit.com/r/beneater/comments/crl270/8_bit_computer_memory_bootloader_and_display/> 
=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=
Un utente scrive:
	I use the highest bit of my 16-bit address to decide whether to enable output (via bus transceivers) for ROM or RAM. This allows me to seamlessly jump between ROM and RAM if I want to include any subroutines in ROM which can be used by programs in RAM, for example.
If you don't want to split your address space into 32 Kb ROM and 32 Kb RAM, you could NAND some address bits together and use that to switch, but my preferred ROM and RAM chips are both 32 Kb so that seemed easiest...
