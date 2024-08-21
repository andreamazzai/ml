---
title: "Control Logic"
permalink: /docs/control/
excerpt: "Control Logic del BEAM computer"
---
**WORK IN PROGRESS**

[![Control Logic del BEAM computer](../../assets/control/40-beam-control.png "Control Logic del BEAM computer"){:width="100%"}](../../assets/control/40-beam-control.png)

Questa pagina descrive le Control Logic dell'NQSAP e del BEAM, evidenzia le differenze con la Control Logic del SAP computer di Ben Eater e approfondisce gli argomenti che avevo trovato più ostici e più interessanti.

In generale, la gestione delle istruzioni consta di tre capisaldi: Instruction Register, Ring Counter e Microcode.

[![Schema della Control Logic dell'NQSAP](../../assets/control/40-control-logic-schema-nqsap.png "Schema logico della Control Logic dell'NQSAP"){:width="100%"}](../../assets/control/40-control-logic-schema-nqsap.png)

*Schema della Control Logic dell'NQSAP, leggermente modificato al solo scopo di migliorarne la leggibilità.*

### Instruction Register

Nell'NQSAP e nel BEAM l'Instruction Register (IR) è incluso nello schema della Control Logic, mentre negli schemi del SAP stava su un foglio separato.

L'Instruction Register del SAP presentava istruzioni lunghe un byte che al loro interno includevano sia l'istruzione stessa sia l'operando:

- i 4 bit più significativi erano dedicati all'istruzione;
- i 4 bit meno significativi erano riservati all'operando.

Nell'immagine seguente, tratta dal video <a href="https://youtu.be/JUVt_KYAp-I?t=1837" target="_blank">Reprogramming CPU microcode with an Arduino</a> di Ben Eater, si vede come ogni byte di un semplice programma di somma e sottrazione includa sia l'operazione sia l'operando:

![Somma e sottrazione nel SAP](../../assets/control/40-lda-15-add-14.png "Somma e sottrazione nel SAP"){:width="50%"}

Ad esempio:

- L'istruzione LDA 15 all'indirizzo di memoria 0000 è composta dai 4 bit più significativi (MSB) 0001 (che nel microcode definiscono un'operazione di caricamento accumulatore) e dai 4 bit meno significativi (LSB) 1111, che indicano l'indirizzo di memoria 15, nel quale è presente il valore 5 da caricare nell'accumulatore A.
- L'istruzione ADD 14 all'indirizzo di memoria 0001 è composta dai 4 bit MSB 0010 (che nel microcode definiscono un'operazione di somma) e dai 4 bit LSB 1110, che indicano l'indirizzo di memoria 14, nel quale è presente il valore 6 da sommare al valore già presente nell'accumulatore A.

| Mnemonico | Indirizzo | Istruzione.Operando |
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

Una fondamentale differenza tra Instruction Register del SAP ed Instruction Register dell'NQSAP e del BEAM è la dimensione. Il 6502 ha un set di istruzioni *relativamente* piccolo, composto da 56 istruzioni di base; tuttavia, queste istruzioni possono essere utilizzate in diverse modalità di indirizzamento, il che porta il numero totale di combinazioni possibili a circa 150.

Per poter gestire questo numero di istruzioni, l'opcode richiede un intero byte e l'architettura del computer deve presentare un Instruction Register adeguato e la possibilità di gestire istruzioni di lunghezza diversa:

- un solo byte per le istruzioni con indirizzamento Implicito e Accumulatore, che non hanno dunque bisogno di opcode;
- due o tre\* byte per tutte le altre istruzioni che hanno bisogno di un operando (a due o tre* byte per definire un indirizzo; a due byte per definire un valore assoluto).

\* Notare che in un computer con 256 byte di RAM le modalità di indirizzamento con 3 byte non sono necessarie, perché un operando della lunghezza di un unico byte è in grado di indirizzare tutta la memoria del computer, come brevemente discusso anche nella sezione [Indirizzamenti](../alu/#indirizzamenti) della pagina dedicata all'ALU.

Conseguentemente:

- il bus tra Instruction Register e Control Logic deve avere un'ampiezza di 8 bit e non più di soli 4 bit come nel SAP;
- sono necessarie EEPROM 28C256 da 256Kb con 15 pin per gli indirizzi:
  - 8 pin di indirizzi per le istruzioni (2^8 = 256 istruzioni)
  - 4 pin di indirizzi per le microistruzioni (2^4 = 16 step)
  - 2 pin di indirizzi per selezionare le ROM
  - resterebbe un pin libero e dunque teoricamente potrebbero essere sufficienti EEPROM da 128Kb, che però non esistono in commercio <a href="https://eu.mouser.com/c/semiconductors/memory-ics/eeprom/?interface%20type=Parallel" target="_blank">con l'interfaccia parallela</a>.

**Qui devo iniziare spiegare come è fatto l'IR dell'NQSAP e a parlare della differenza tra il registro delle istruzioni dello NQSAP e del Beam. Segnalare che avevo certamente letto tutto su n QSP , ma avevo anche cercato di comprendere le differenze nella versione PCB avevo compreso chela bufferizzazione avrebbe compotato diversi vantaggi , cosa che ho applicato al BAM .**

Per indirizzare i problemi di glitching Tom ha bufferizzato l'IR, cioè due FF da 8 registri in cascata, così il primo viene aggiornato al normale caricamento dell'IR (che corrisponderebbe a T7 (step 1), ma causando un glitch sulla ROM)… invece di collegare il FF agli ingressi delle ROM, viene collegato a un altro FF che viene caricato col Falling Edge del CLK / Rising Edge del CLK, così le uscite delle ROM vengono aggiornate alla fine della microistruzione quando i segnali sono stabili 😁

[![Schema della Control Logic del SAP computer](../../assets/control/40-control-logic-schema-SAP.png "Schema logico della Control Logic del SAP computer"){:width="100%"}](../../assets/control/40-control-logic-schema-SAP.png)

*Schema della Control Logic del SAP computer.*

### Ring Counter

Le istruzioni di un microprocessore sono composte da un certo numero di step, più precisamente chiamati microistruzioni. La Control Logic deve settare correttamente la Control Word per ogni microistruzione, così che questa possa essere correttamente eseguita in corrispondenza dell'impulso di clock.

In un computer è sempre necessario sapere sempre quale istruzione stiamo eseguendo - e di questa abbiamo indicazioni dall'Instruction Register - e quale è lo step correntemente in esecuzione. Per fare questo, si utilizza il Ring Counter. Per tracciare la microistruzione, sia il SAP sia l'NQSAP utilizzano un contatore 74LS161, in grado di contare da 0 a 15.

La Control Logic deve essere settata tra un clock e l'altro; in altre parole, la Control Logic deve approntare la Control Word prima che lo step venga eseguito in corrispondenza del ciclo di clock. Gli step vengono eseguiti in corrispondenza del raising edge del segnale di clock ,; per settare la control Word col giusto anticipo è possibile utilizzare un inverter per pilotare il clock del contatore virgola in modo tale che questo effettui l'operazione di settaggio della control Word con uno sfasamento rispetto al clock originario virgola e dunque in corrispondenza del falling edgedel clock .

- I registri del computer sono infatti aggiornati al Rising Edge del CLK. CLK gestisce tutti i registri principali: PC, MAR, RAM, IR, A, B, Flag: al Rising Edge del CLK, avvengono le azioni di caricamento dei registri. 
Ad esempio, quando c'è il segnale CE Counter Enable attivo, il PC viene incrementato al Rising Edge e l'indirizzo viene aumentato di uno.
- Le microistruzioni sono invece aggiornate al Falling Edge del CLK: /CLK gestisce il Ring Counter e di conseguenza la Control Logic: è sfasato di 180°, dunque al Falling Edge di CLK corrisponde il Rising Edge di /CLK

Il 74LS138 è un decoder che può prendere i 3 bit (ce ne bastano 3 per gestire 8 cicli, visto che gli step delle microistruzioni sono al massimo 6) e convertirli in singoli bit che rappresentano lo step della microistruzione corrente e poi uno di questi, l'ultimo, che resetta il 74LS161 in modo da risparmiare i cicli di clock inutilizzati.

A questo punto abbiamo nell'Instruction Register l'istruzione attualmente in esecuzione e nel contatore lo step, che rappresenta la microistruzione. Possiamo usare una Combinational Logic per settare i nostri segnali da abilitare per ogni microistruzione. Per esempio quando siamo in T0, che è la prima microistruzione, prendiamo l'uscita del 74LS138, la neghiamo con un NOT per portarla positiva e attiviamo CO e MI e poi Clock. Alla successiva attiviamo RO e II  e poi Clock e alla successiva attiviamo CE e poi Clock.
E poiché CE può essere inserito nella stessa microistruzione di RO e II possiamo ridurre la lunghezza delle microistruzioni a un massimo di 5 (step 0-4).

Come indicato anche nella sezione [Differenze](.../alu/#differenze-tra-moduli-alu-dellnqsap-e-del-beam) della pagina dell'ALU, bisogna notare che il computer NQSAP prevedeva solo 8 step per le microistruzioni. Per emulare le istruzioni del 6502 di salto condizionale, di scorrimento / rotazione e di salto a subroutine servono più step, pertanto, sul computer BEAM ne sono stati previsti 16.

Le istruzioni del computer sap avevano tutte la stessa durata cioè 5 step indipendentemente dalla loro complessità punto nel micro code che segue possiamo vedere che in realtà l'istruzione di caricamento immediato è lunga solo tre step , mentre ad esempio somma e sottrazione sono lunghe 5 step

~~~text
...
...
const PROGMEM uint16_t microcode_template[16][8] = {
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 0000 - NOP
  { CO|MI, RO|II|CE,  IO|MI,  RO|AI,  0,            0,  0,  0 }, // 0001 - LDA
  { CO|MI, RO|II|CE,  IO|MI,  RO|BI,  EO|AI|FI,     0,  0,  0 }, // 0010 - ADD
  { CO|MI, RO|II|CE,  IO|MI,  RO|BI,  EO|AI|SU|FI,  0,  0,  0 }, // 0011 - SUB
  { CO|MI, RO|II|CE,  IO|J,   0,      0,            0,  0,  0 }, // 0100 - JMP
  { CO|MI, RO|II|CE,  IO|MI,  AO|RI,  0,            0,  0,  0 }, // 0101 - STA
  { CO|MI, RO|II|CE,  IO|AI,  0,      0,            0,  0,  0 }, // 0110 - LDI
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 0111 - JC // normalmente non accade nulla...
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1000 - JZ // normalmente non accade nulla...
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1001
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1010
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1011
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1100
  { CO|MI, RO|II|CE,  0,      0,      0,            0,  0,  0 }, // 1101
  { CO|MI, RO|II|CE,  AO|OI,  0,      0,            0,  0,  0 }, // 1110 - OUT
  { CO|MI, RO|II|CE,  HLT,    0,      0,            0,  0,  0 }, // 1111 - HLT
};
...
...
~~~

come si può vedere nello schema del SAP, il contatore '161 presente le sue uscite agli ingressi di selezione del DEMUX '138, che attiverà in sequenza le uscite invertite (active = LO) da 00 a 05 in sequenza; all'attivazione di quest'ultima, le due NAND attiveranno l'ingresso di Reset /MR del '161, che riporterà il conteggi degli step allo zero iniziale, cominciando così una nuova istruzione.

È facile notare che per le istruzioni più corte questo si produce in uno spreco di cicli di elaborazione.

[![Ring Counter del SAP](../../assets/control/40-control-sap-rc.png "Ring Counter del SAP"){:width="33%"}](../../assets/control/40-control-sap-rc.png)

*Ring Counter del SAP.*

Per indirizzare questa questione e migliorare le prestazioni del computer , tutte le istruzioni dell'NQSAP presentano una microistruzione finale "**N**", che attiva un segnale **NI** che attiva l'ingresso di caricamento del '161, che, avendo tutti i suoi input allo stato zero, sarà praticamente resettato. Perché non collegare l'ingresso m al reset del contatore puntointerrogativo così facendo , se si facesse in questo modo , il contatore verrebbe resettato all'inizio di ogni ultimo microistruzione , pertanto si dovrebbe creare una micro istruzione aggiuntiva per tutte le istruzioni anziché poter avere l'istruzione micron direttamente integrata insieme alle ultime istruzioni micro di ogni istruzione , risparmiando così un ciclo macchino .questo perché il contatore è sincrono virgola e significa che dunque il suo reset avverrebbe in qualsiasi momento , indipendentemente dallo stato del clock.volevo dire asincrono .e allora perché non utilizzare un contatore sincrono ? non lo possiamo fare perchéavremmo problemi con il reset vero e proprio. 

15/06/2023: ho deciso di fare come Tom e mettere alla fine di ogni istruzione una microistruzione di reset così da poter anticipare la fine dell'istruzione e passare alla prossima senza dover aspettare tutti i cicli a vuoto. Spiegare il legame con il funzionamento del 161 , che permette di caricare zero sui suoi ingressi, ed è il metodo che viene utilizzato per resettare il ring counter e riportarlo allo step t zero.


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

## CONTROL LOGIC PART 1



## Forse interessante da tenere, espandere, collegare ad altri paragrafi

Praticamente ora abbiamo il contatore delle microistruzioni (T0-T4) e il contatore dell'istruzione (Instruction Register MSB). Posso creare una **combinational logic** che, a seconda dell'istruzione che ho caricato nell'Instruction Register + il T0/4 dove mi trovo mi permetta di avere in uscita i segnali corretti da applicare al computer.

Praticamente ho due fasi:
• Fetch, in cui carico l'istruzione dalla RAM nell'Instruction Register
• Dopo la fase di Fetch so "cosa devo fare", perché a questo punto ho l'istruzione nell'Instruction Register

La realizzazione del comuter SAP mi ha permesso finalmente di capire cosa sia il microcode di un computer moderno.

È piuttosto comune leggere ad esempio che è necessario aggiornare il bios dei server per indirizzare falle di sicurezza che sono state scoperte e che potrebbero essere utilizzate dagli hacker per ... 
Non capendo come potesse essere aggiornata una CPU, dal momento che si tratta di un componente non programmabile , non riuscivo a comprendere come fosse possibile arginare i problemi di sicurezza; con il microcode ho capito

Ritornando alla dimensione delle EEPROM da utilizzare per il microcode, nei miei appunti trovo traccia di diverse revisioni, ad esempio:

- come notavo anche nelal costruzione del modulo RAM in cui si indicavano le 256 istruzioni, notavo che servivano 28c64. ram/#mux-program-mode-e-run-mode 

## Ring Counter

Vedi spiegazione per resettare in maniera sincrona il 74LS161 sulla pagina dei chip
	• Praticamente usando il '161:
		○ con /CLR, che è asincrono, faccio il reset "hardware"
			§ 17/06/2023 Tom segnala che questo segnale asincrono non è invece ideale per pulire il Ring Counter utilizzando il microcode perché, essendo appunto asincrono, non sarebbe gestito dal clock: infatti, non appena attivato, andrebbe a resettare il ciclo di microistruzione esattamente all'inizio invece che quando arriva il impulso di clock!
			§ 04/07/2023 Va invece benissimo per fare il reset del Program Counter 😁 anche col clock stoppato
			§ A dire il vero si potrebbe comunque utilizzare questo segnale, ma significherebbe dover aggiungere una microistruzione dedicata al reset alla fine di tutte le altre microistruzioni. Utilizzando una modalità di reset sincrona su questo chip si potrebbe invece aggiungere il segnale di reset all'ultima microistruzione del ciclo.
		○ con il /LOAD ("N") e tutti i pin di input a 0, il Ring Counter si resetta in sincrono con il /CLK 😁. Assomiglia un po' al JUMP del Program Counter. Notare il /CLK, che è invertito rispetto al CLK principale e che dunque permette di lasciar terminare l'esecuzione della microistruzione corrente prima di fare il LOAD.

		○ questo forse significa che poiché il '138 è un decoder 3-to-8 devo metterne due "in cascata"? Posso farlo?
		
3 uscite del 161 vanno al 138 per pilotare 8 led (2^3 = 8); per il 9° LED Tom è il solito furbo: invece di mettere due 138 per pilotare 16 led, aggiunge un led al quarto pin del 161, così ad esempio 11 = 3 + 8 e dunque si accendono il 3* led di 8 pilotato dal 138 e quello dell'"Extended Time" pilotato dal 161

	• Gli ingressi D0-D3 sono tutti a 0, così quando arriva il /LOAD (o /PE) sincrono (segnale "N" per Tom), il conteggio ricomincia da zero
	• Le uscite Q0-Q2 del 161 vanno a MA0-MA2 per avere 8 step di microistruzioni, ma se aggiungo un quarto pin al contatore, posso avere 16 step
	• Il /LOAD arriva da N 17 della ROM0
CEP e CET sono a +Vcc

## Instruction Register





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

NB: Dobbiamo settare la Control Logic tra un clock e l'altro… come a dire che la Control Logic deve "essere pronta" prima che l'istruzione venga eseguita: possiamo usare un NOT per invertire il clock e usare questo per gestire il 74LS161 della Control Logic.

	• I registri sono aggiornati al Rising Edge del CLK, che corrisponde al Falling Edge del /CLK.
	• Le microistruzioni sono aggiornate al Falling Edge del CLK, che corrisponde al Rising Edge del /CLK.
	• CLK gestisce tutti i registri principali: PC, MAR, RAM, IR, A, B, Flag: al Rising Edge del CLK, avvengono le azioni di caricamento dei registri. Quando c'è il segnale CE Counter Enable attivo, il PC viene incrementato al Rising Edge e l'indirizzo viene aumentato di uno.
	• /CLK gestisce il Ring Counter e di conseguenza la Control Logic: è sfasato di 180°, dunque al Falling Edge di CLK corrisponde il Rising Edge di /CLK

Control Logic
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

- I video di Ben Eater che descrivono la <a href="https://eater.net/8bit/control" target="_blank">Control Logic e il microcode</a>.

## TO DO

- aggiornare lo schema Kicad con le le bar a 8 segmenti e aggiornare questa pagina con lo schema aggiornato
- Evidenziare la nomenclatura dei segnali da fare nella pagina della control logic : l'approccio di ben era centri con rispetto al modulo , mentre l'approccio del computer NQSAP è relativo al computer nella sua interezza
- non trovo riferimenti ad HL e HR in nessuna pagina; Poiché in questa pagina sto parlando del fatto che per alcuni registri sono necessari più segnali di controllo , come nel caso del registro h virgola che necessita di HLanche di HR volevo fare un link al registro h nella pagina del modulo ALU , ma vedo che anche lì non cè nessuna indicazione di HL anche di HR ("quando un registro presenta più segnali di ingresso che possono essere attivi contemporaneamente (ad esempio il registro dei Flag, oppure il registro H");)
- far notare che il led che mostra gli step sono 8+1 e non 16, furbata,
- Ho aggiunto anche una barra a led per mostrare l'indirizzo correntemente In input sulle EEPROM
