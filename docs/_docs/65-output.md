---
title: "OUTPUT"
permalink: /docs/output/
excerpt: "Costruzione del registro di output del BEAM computer"
---
Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da 0 a 255 e viceversa, oppure un piccolo algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte (e non complicarmi troppo la vita con un bus a 16 bit), ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve un registro MAR (Memory Address Register) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173 (il '173 è il Flip-Flop utilizzato in origine nel SAP), sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando istruito a farlo - e non ad ogni ciclo di clock. Il segnale MI (Memory Address Register Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario farlo: con MI attivo, in corrispondenza del rising edge del clock il MAR memorizza il valore presente nel bus. Senza un ingresso di Enable, il FF '273 andrebbe a "registrare" il dato in corrispondenza di ogni ciclo di clock e non solo quando fosse necessario.

Nel mio progetto a 8 bit si potrebbero semplicemente utilizzare due '173 a 4 bit continuando a sfruttare i segnali di Enable nativamente disponibili. Bisogna dire che sarebbe comunque possibile utilizzare anche il '273 a 8 bit utilizzando una porta AND per *costruire* un segnale di Enable artificiale: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse contemporaneamente presente anche il segnale MI.

Il '273, al pari del '173, presenta un ingresso Clear / Reset (CLR), che nel MAR è necessario per resettare il registro - o almeno *credevo* necessario. Sembrava anche interessante l'ipotesi alternativa di usare un registro a 8 bit [74LS377](https://datasheetspdf.com/download_new.php?id=375625), che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il CLR, non presente in questo chip. Come ormai si sarà capito, ho realizzato successivamente che il MAR poteva funzionare perfettamente anche senza un segnale di Clear / Reset.

Da notare che il '377 sarebbe in seguito diventato uno dei chip più utilizzati nel BEAM.

**METTERE QUALCHE FOTO E LINK**

>> 2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)

Come nel caso del MAR, per indirizzare 256 byte di RAM era necessario un registro Program Counter (PC) a 8 bit. Nel computer SAP era invece presente un contatore a 4 bit [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) e dovevo pertanto cercare di combinarne due in cascata.

Sarebbe stato comodo utilizzare un singolo contatore a 8 bit, ma tra i chip disponibili sul mercato non ne ho trovato uno che includesse anche l'ingresso LOAD. Il LOAD permette il caricamento parallelo sul PC di uno specifico indirizzo al quale il computer deve saltare (ad esempio, per eseguire un'istruzione di salto assoluto o branch relativo, si calcola l'indirizzo di destinazione e lo si carica nel PC).

Per combinare due chip a 4 bit è stato necessario, nonché molto utile, capire la differenza fra "Ripple Mode Carry" e "Carry Look Ahead": il datasheet del contatore 74LS161 riportava infatti questi due diversi esempi di collegamento di chip in cascata tra di loro.

**METTERE QUALCHE FOTO E LINK E SPIEGAZIONE PER RCA E ADDER ETC**

>> 3. Instruction register - now sh!t gets fun. Since i want to bomb completely, I am considering having a 3 byte Instruction register in a way mimicking the BIU block & instruction queue in a 8086. The idea here is to split the instruction register in 3 bytes. 1 byte would be for the instruction/opcode alone, the second and third are going to hold the data of the instruction (either address or an immediate value). This would allow you to address the entire memory space. Also instructions can be of different size. For example. OUT (move to out register) would be just 1 byte wide, LDI (load immediate) would be 2 bytes and LDA (load address/absolute) would be 3 bytes wide. The control logic would take care of the fetch cycle. Or in other words since you know the instruction you are execution you would know how much bytes the instruction is and thus fetch either 1, 2 or 3 bytes from the RAM.

Dopo aver letto questo punto avevo iniziato a raccogliere i miei pensieri per l'espansione di memoria a 256 byte, idea che riprendo in seguito in questa pagina. Non avevo intenzione di costruire un IR a più byte, cosa che mi sembrava piuttosto complessa per le mie capacità.

### Memorie con IO separati o IO comuni?

Fino ad ora, avevo quasi sostanzialmente dato per scontato di continuare ad usare chip di memoria con porte di Input e Output separati ("dual-port"), esattamente come accade nel [74189](https://eater.net/datasheets/74189.pdf) utilizzato nel SAP. Tuttavia, in questo [post su Reddit](https://www.reddit.com/r/beneater/comments/hon6ar/74189_alternative/), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf); ho così iniziato ad approfondire le caratteristiche di questo chip "single-port", aumentando la mia comprensione di queste due diverse architetture.

In origine avevo evidenziato questi pochi appunti presenti nel post, riflettendo sul fatto che l'approccio alla gestione dei segnali di controllo mi sembrava un po' troppo semplice, ma più in là nel tempo avevo realizzato che, tutto sommato, la scrittura sulla RAM non è *eccessivamente* complessa:

>> 62256 - The I/O lines are controlled by OE and CE.
When either are high puts the I/O lines at high impedance.
When both are low, the RAM outputs data onto the I/O lines.
Writing takes place (two ways, but this is one way) CE low and OE high. A low pulse on WE will write the data on the I/O lines into the RAM chip.

- **CE** = Chip Enable
- **OE** = Output Enable
- **WE** = Write Enable

Un altro aspetto che avevo notato immediatamente, ipotizzando l'uso del 62256, era l'impossibilità di mantenere la visibilità del contenuto della cella di RAM indirizzata dal MAR utilizzando i LED (o almeno così credevo): se con i '189 le porte di output erano sempre attive e potevo vedere in ogni momento il valore contenuto della cella di memoria correntemente indirizzata dal MAR, con il 62256 avrei avuto visibilità del contenuto della cella solo nel momento in cui la RAM veniva letta - e dunque non costantemente.

[![Schema del modulo RAM di Ben Eater basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione ininterrotta del contenuto della locazione RAM indirizzata dal MAR](../../assets/20-be-ram-detail.png "Schema del modulo RAM basato su 74189: le porte nativamente designate per l'Output consentono la visualizzazione continua del contenuto della locazione RAM indirizzata dal MAR"){:width="50%"}](../../assets/20-be-ram-full.png)

*Nello schema si notano i 74189 con le porte di Input dedicate D1-D4 e le porte di Output dedicate O1-O4.*

Quello che iniziavo a capire era che per utilizzare una RAM con Common IO dovevo fare un "doppio passaggio" o qualcosa di simile. Come faccio ad avere sempre visibile il contenuto della locazione di memoria anche nel momento in cui setto le porte di IO del chip in modalità Input? Come faccio a mantenere la visibilità del contenuto della RAM quando questa non è attiva in output? Devo forse memorizzare il contenuto delle uscite della RAM in qualche latch e solo a quel punto disabilitare il chip prima di andarvi a scrivere? In seguito avrei capito che non era necessario un latch, ma che c'era un'altra strada.

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; la soluzione poteva essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737); intuivo qualcosa, relativamente alla necessità di gestire i segnali di controllo della RAM in maniera più ampia, controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo avere sia la possibilità di programmare la RAM ("Program Mode"), sia di renderla poi visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è manuale e sfrutta dei dip-switch per indirizzare e programmare manualmente la RAM (o leggerne il contenuto della RAM in stile "debug mode").

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata esclusivamente dal MAR e altrettanto esclusivamente viene acceduta in lettura / scrittura solo dal bus del computer.

La selezione di cosa passare a RAM e MAR avviene mediante un MUX (nel nostro caso 2:1, cioè ad ogni uscita corrispondono due ingressi selezionabili): gli ingressi del MUX sono connessi sia ai dip-switch che utilizzeremo per la programmazione manuale del computer, sia al bus dati del computer; le uscite sono connesse agli ingressi della RAM e del MAR. Un semplice interruttore connesso all'ingresso di selezione del MUX consente di scegliere quali ingressi attivare.

Ad esempio, nello schema del SAP visibile più in alto in questa pagina, i multiplexer '157 gestiscono gli ingressi della RAM: gli ingressi dei MUX sono connessi sia al dip-switch sia al bus del computer, mentre le uscite sono connesse alle porte di ingresso D1-D4 dei chip di RAM '189.

Si ripresentava il tema del "doppio passaggio" (o meglio "doppio bus", come capirò in seguito), che da quanto iniziavo a comprendere poteva rendere possibile la visualizzazione persistente del contenuto della RAM. A pagina 17 e 18 del "Building the SAP-3 rev 3.3.pdf" presente nel repository GitHub di <a href = "https://github.com/rolf-electronics/The-8-bit-SAP-3" target = "_blank">rolf-electronics</a>, altro utente del canale Reddit, avevo notato che era stato inserito un altro transceiver '245.

[![Modulo RAM di rolf electronics](../../assets/20-rolf-ram.png "Modulo RAM di rolf electronics"){:width="50%"}](../../assets/20-rolf-ram.png)

Il funzionamento e la necessità dei transceiver mi erano chiarissimi, in quanto ampiamente utilizzati nel SAP computer per poter attivare i vari moduli del computer solo nel momento in cui fosse necessario farlo: tipicamente ogni modulo ha bisogno di un unico transceiver di interconnessione verso il bus.

Lo schema del modulo RAM di Rolf ne prevede invece due, uno "interno" e uno "esterno", per separare il percorso dei dati *verso* la RAM da quello *dalla* RAM:

- Il transceiver di sinistra è attivo quando si scrive *sulla* RAM, selezionando quale sia la sorgente mediante il MUX '157 (che in "Program Mode" attiva gli ingressi connessi al dip-switch, mentre in "Run Mode" attiva gli ingressi connessi al bus). I LED sono attivi e mostrano il valore che viene scritto sulla RAM.
- Il transceiver di destra è attivo quando si legge *dalla* RAM. I LED sono attivi e mostrano il valore che viene letto dalla RAM e trasferito sul bus del computer.

Un latch per memorizzare lo stato dei LED, come erroneamente ipotizzavo inizialmente, non era necessario.

Proseguendo nello studio, ho trovato [questo schema](https://imgur.com/a/ruclh) dell'utente jaxey1631, che aveva lasciato un commento nel video di Ben Eater [Reprogramming CPU microcode with an Arduino](https://www.youtube.com/watch?v=JUVt_KYAp-I&lc=UgjusLoROw6az3gCoAEC):

[![RAM e MAR con doppio bus](../../assets/20-ram-ruclh.png "RAM e MAR con doppio bus"){:width="100%"}](../../assets/20-ram-ruclh.png)

*Schema di RAM e MAR con bus interno.*

In questo schema troviamo:

- Il chip di RAM 62256.

- Un FF '273 in alto a destra che, sprovvisto di Enable come discusso in precedenza, si attiva in corrispondenza di un Enable fittizio costruito con l'operazione logica "Clock + Memory Address Register In" (ingressi 1A e 1B del NAND a sinistra nello schema); il segnale MI indica che il computer si prepara a settare l'indirizzo di RAM sul quale eseguirà la prossima operazione.

- I due MUX '157 in alto che consentono la selezione degli indirizzi (tra quelli settati sui dip-switch o quelli presenti sulle uscite del FF '273) da esportare verso i pin A0-A7 del chip di RAM; la selezione degli ingressi attivati dal MUX avviene grazie all'interruttore (in alto a sinistra) di selezione della modalità connesso agli ingressi SEL: in Program Mode è acceso il LED rosso e gli ingressi SEL sono allo stato LO, attivando gli ingressi A1-A4; in Run Mode è acceso il LED verde e gli ingressi SEL sono allo stato HI, attivando gli ingressi B1-B4.

- Altri due MUX '157 in basso che consentono la selezione di cosa esportare verso i pin dati D0-D7 del chip di RAM; anche questi MUX sono connessi all'interruttore di selezione della modalità e attivano gli ingressi connessi al dip-switch di selezione degli indirizzi o quelli connessi al bus del computer a seconda dello stato dell'interruttore (Program Mode o Run Mode).

- Un primo transceiver '245 (in basso) i cui ingressi sono connessi alle uscite dei due MUX citati nel punto precedente. Questo transceiver funge da interfaccia *verso* la RAM (il pin DIR settato a LO configura i pin A1-A8 come ingressi e i pin B1-B8 come uscite) e si attiva nel momento in cui si deve scrivere in memoria; il segnale OE di questo primo transceiver è infatti attivo quando quando si preme il pulsante Write Button (in basso a sinistra) se in Program Mode, o in corrispondenza di Clock + RI (RAM In) (vedi ingressi 4A e 4B del NAND centrale a sinistra nello schema) quando in Run Mode;

- Un secondo transceiver '245 che si attiva nel momento in cui si deve leggere *dalla* RAM e che ne trasferisce l'output verso il bus dati (anche in questo caso il pin DIR del '245 settato a LO configura i pin A1-A8 come ingressi e i pin B1-B8 come uscite); notare il suo ingresso OE connesso al segnale RO (RAM Output) del computer.

[![Write cicle del 62256](../../assets/20-ram-write-cycle.png "Write cicle del 62256"){:width="50%"}](../../assets/20-ram-write-cycle.png)

*Write Cycle "WE Controlled" del 62256.*

Notare la configurazione del chip di RAM: i segnali CE ed OE sono sempre attivi, che significa che l'utente ha deciso di utilizzare la modalità di scrittura definita come "WE# Controlled" definita a pagina 6 del [datasheet](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf) del 62256. Rileggendo questi appunti diverso tempo dopo aver completato il mio progetto, mi sembra tutto facile, ma la comprensione delle modalità di scrittura della RAM è stata in realtà piuttosto lunga.

Da notare inoltre che anche questo utente non usa il CLR sui FF '273 di input del MAR - a pensarci, potrebbe realmente non servire, perché ogni volta che ho bisogno di accedere alla RAM, vado preventivamente a settare sul MAR l'indirizzo desiderato. Forse il reset all'accensione è più estetico che altro.

E' stato in questo momento (circa agosto / settembre 2022) che ho scoperto l'**NQSAP**, inserendolo tra i miei appunti come "c'è questo [https://tomnisbet.github.io/nqsap/docs/ram/](https://tomnisbet.github.io/nqsap/docs/ram/) che sembra aver fatto delle belle modifiche al suo computer" 😁; ho deciso di seguire questo progetto perché permetteva di costruire un instruction set come quello del 6502 che, come scoprirò in seguito, richiederà un numero elevato di indirizzi per il microcode delle EEPROM.

Tra i vari link sondati, c'era anche [questo post Reddit](https://www.reddit.com/r/beneater/comments/h8y28k/stepbystep_guide_to_upgrading_the_ram_with/), che molti utenti hanno trovato ben fatto, ma che io ho trovato molto difficile da digerire in quanto mancante di uno schema.

Per aggiungere un ulteriore link utile per la comprensione delle architetture del modulo di RAM, evidenzio questo [post su Reddit](https://www.reddit.com/r/beneater/comments/ad2uko/upgrading_the_ram_module_to_256_bytes/). Le spiegazioni sono molto ben fatte e utili. Il chip di RAM utilizzato è interessante perché si presenta come due RAM distinte, ognuna con accessi dedicati e un segnale di Busy per gestire le richieste parallele sulla stessa locazione. Altro aspetto degno di nota nell'implementazione di questo utente è la possibilità di aumentare fino a 256 il numero di istruzioni del computer, grazie alla scelta di utilizzare un byte intero per l'istruzione ed un eventuale byte successivo per l'operando, anziché avere un unico byte di cui i 4 Most Significant Bit (MSB) rappresentano l'opcode e di cui i 4 Least Significant Bit (LSB) sono l'operando, come nel SAP di Ben Eater.

Un aspetto collaterale (ma importantissimo) dell'aumento del numero di istruzioni era la necessità di aumentare la dimensione delle EEPROM ospitanti il microcode: volendo gestire (fino a) 256 istruzioni, erano necessari 8 bit di istruzioni, 3 di step e 2 di flag = 13 pin totali, portanto si rendevano necessarie delle 28C64... e avevo dimenticato che mi sarebbe servito un bit aggiuntivo per la selezione delle due EEPROM! In quel momento, non sapevo ancora che avrei speso intere settimane a comprendere il fantastico modulo dei Flag dell'NQSAP di Tom Nisbet, che ha un approccio completamente diverso e che non necessita di segnali in uscita dalle EEPROM.

## Gestione della RAM

Tra i post più utili relativi alla comprensione dei segnali di gestione di RAM e MAR per il modulo di memoria con IO comuni, c'è certamente il [Question about RAM replacement](https://www.reddit.com/r/beneater/comments/ut1oud/8bit_question_about_ram_replacement/), nel quale il moderatore The8BitEnthusiast invita a consultare la sua (eccellente, aggiungo io) realizzazione, che ho preso ad esempio e ispirazione.

[![Modulo RAM dell'utente Reddit The8BitEnthusiast](../../assets/20-ram-the8bit_enthusiast.png "Modulo RAM dell'utente Reddit The8BitEnthusiast"){:width="100%"}](../../assets/20-ram-the8bit_enthusiast.png)

*Modulo RAM dell'utente e moderatore Reddit The8BitEnthusiast.*

Se da un certo punto di vista lo schema era particolarmente semplificato rispetto a quelli che avevo visto in precedenza, dall'altra, con l'utilizzo di un solo MUX '157 con segnali che ne uscivano per rientrarne in altre porte, mi risultava di difficile comprensione. Avevo pertanto provato a costruirmi i grafici temporali degli schemi dei segnali per questi tre possibili casi:

- Scrittura sulla RAM in Run Mode

[![Scrittura sulla RAM in Run Mode](../../assets/20-ram-run-mode-write-t8be.png "Scrittura sulla RAM in Run Mode"){:width="30%"}](../../assets/20-ram-run-mode-write-t8be.png)

- Lettura dalla RAM in Run Mode

[![Lettura dalla RAM in Run Mode](../../assets/20-ram-run-mode-read-t8be.png "Lettura dalla RAM in Run Mode"){:width="30%"}](../../assets/20-ram-run-mode-read-t8be.png)

- Scrittura sulla RAM in Program Mode

[![Scrittura sulla RAM in Program Mode](../../assets/20-ram-program-mode-write-t8be.png "Scrittura sulla RAM in Program Mode"){:width="30%"}](../../assets/20-ram-program-mode-write-t8be.png)

The8BitEnthusiast segnalava di *aver sfruttato il ritardo di propagazione dei '245 per gestire i requisiti di temporizzazione*, al che avevo provato a chiedergli se fosse necessario gestire le temporizzazioni in maniera così precisa perché il suo progetto lavorava in modalità "just in time" ogni volta che sopraggiungeva un impulso di clock.

Per esempio, ipotizzavo che nel primo caso "Scrittura sulla RAM in Run Mode" accadesse quanto segue.

- **Prima del Rising Edge del CLK**:

  - siamo in Run Mode, dunque PROG è HI
  - il MUX abilita gli ingressi I1a, I1b, I1c
  - Zb è HI (in quanto l'input I1b è connesso a Vcc) e dunque il '245 di destra, che connette il dip-switch di programmazione, è disabilitato
  - il segnale RI è HI (RI = RAM In, cioè scrittura sulla RAM), dunque stiamo preparando una scrittura
  - il segnale /RO è HI (RO = RAM Out, cioè lettura dalla RAM), dunque la lettura dalla RAM è disabilitata
  - Zc = /(CLK LO * RI HI) = HI, dunque la direzione del '245 di sinistra, che connette il data bus, è A-->B (Output)
  - Za = //(/RO HI * Zc HI) = HI, dunque il '245 dal/al data bus è disabilitato

- **CLK attivo**:

  - Zc = /(CLK HI * RI HI) = LO, dunque la direzione del '245 dal/al data bus è B-->A (Input)
  - Za = //(/RO HI * Zc LO) = LO, dunque il '245 dal/al data bus è attivo
  - /WE = Zc = LO, dunque la RAM riceve il Falling Edge del segnale di Write e trova sui suoi ingressi quanto le viene proiettato dal '245 dal/al data bus

- **Quando l'impulso di Clock termina**:

  - Zc = /(CLK LO * RI HI) = HI, dunque la direzione del '245 dal/al data bus è A-->B (Output)
  - /WE = Zc = HI, dunque l'impulso di Write sulla RAM termina con il Rising Edge
  - Za = //(/RO HI * Zc HI) = HI, dunque il '245 dal/al data bus viene disabilitato

Legenda:

- **PROG** è il segnale dell'interruttore di selezione della modalità Program Mode (LO) / Run Mode (HI); negli schemi originali del SAP computer si trova nel MAR
- **/** significa NOT
- **\*** significa AND

[![Scrittura sulla RAM in Run Mode](../../assets/20-ram-run-mode-write-large-t8be.png "Scrittura sulla RAM in Run Mode"){:width="100%"}](../../assets/20-ram-run-mode-write-large-t8be.png)

*Scrittura sulla RAM in Run Mode.*

Da quanto leggevo, immaginavo che il momento critico fosse il Rising Edge del Clock, perché in quel mentre è necessario attendere che la RAM sia pronta per la scrittura e proprio qui è necessario sfruttare il ritardo introdotto dal '245 per mostrare i dati alla RAM "un po' più tardi", ma non capivo esattamente il motivo.

The8BitEnthusiast ha gentilmente risposto al mio quesito:

> Dovevo assicurarmi che i ‘245 non consegnassero dati alla RAM quando questa non era ancora pronta per accettare dati in Input perché le sue uscite erano ancora attive in output\*\*. Il datasheet segnala che la RAM disabilita l’output ed è pronta per l’input 20 nanosecondi dopo che WE viene portato allo stato LO.

[![Timing RAM 62256](../../assets/20-ram-62256-timing.png "Timing RAM 62256"){:width="66%"}](../../assets/20-ram-62256-timing.png)

\*\* Nello schema del modulo RAM di The8BitEnthusiast si nota che il segnale OE della RAM è connesso a ground, che significa che i pin dati sono sempre attivi in output, *tranne* quando si deve effettuare una scrittura. Quando si attiva il segnale di write WE, vi è un tempo tWHZ durante il quale la RAM è ancora attiva in output; trascorso questo tempo, è possibile mettere dei dati in input sulla RAM.

In altre parole, la RAM è normalmente attiva in output; per scrivere su di essa, la si deve attivare in input col segnale WE. Poiché la RAM impiega 20 nanosecondi per commutare i pin dati da output a input, prima di applicare dei segnali in input è necessario attendere almeno 20 nanosecondi dal momento in cui si attiva WE.

[![Write Cycle](../../assets/20-ram-write-cycle-twhz.png "Write Cycle"){:width="50%"}](../../assets/20-ram-write-cycle-twhz.png)

> Lo stesso segnale Zc che attiva la scrittura su RAM (WE) abilita anche i due '245; il datasheet del '245 specifica che la sua attivazione richiede 25 nanosecondi, che è un valore superiore a quello necessario alla RAM per attivarsi in Input, dunque il requisito è rispettato.

Molto, molto clever.

## Design dei moduli MAR e RAM del BEAM

Parallelamente agli studi dei lavori di altri utenti, avevo iniziato a lavorare sul disegno dei miei moduli MAR e RAM, non senza continuare a saltare di palo in frasca per approfondire temi ancora parzialmente oscuri o affrontare argomenti nuovi.

Il MAR era progettato a 8 bit per pilotare 256 indirizzi di memoria. Inizialmente avevo utilizzato due FF '173 a 4 bit, poi sostituiti da un unico FF '273 a 8 bit (sprovvisto di controllo dell'attivazione dell'output, che però non è necessario). Infine ho utilizzato un registro a 8 bit '377, altrettanto adatto al nostro scopo.

[![Memory Address Register](../../assets/20-mar-beam.png "Memory Address Register"){:width="100%"}](../../assets/20-mar-beam.png)

*Memory Address Register (MAR) del BEAM.*

Notare l'interruttore di selezione Program Mode / Run Mode (segnale PROG, che è connesso anche al modulo RAM):

- alla chiusura dei contatti 1-2 i pin di selezione dei MUX '157 si trovano allo stato logico LO, attivando gli ingressi I0a, I0b, I0c ed I0d, che trasmettono così al bus indirizzi della RAM i valori settati sul dip-switch;
- alla chiusura dei contatti 2-3 i pin di selezione dei MUX '157 si trovano allo stato logico HI, attivando gli ingressi I1a, I1b, I1c ed I1d, che trasmettono così al bus indirizzi della RAM i valori presenti in output sul '377.

### Prima versione del modulo RAM

Come già detto, per quanto riguarda la realizzazione del modulo RAM avevo deciso di procedere con il chip 62256 con IO comuni. Per cercare di fissare i concetti, avevo trascritto nuovamente le differenze tra le architetture con chip con IO separati ed IO comuni:

- Con le RAM dual-port, avevo i segnali di RAM IN e RAM OUT su bus "separati":
  - il bus "IN" collegava le uscite dei MUX 74LS157 ai pin Data In delle RAM: i MUX erano sempre attivi sulle porte "Write" delle RAM e mostravano ad esse tutto quello che accadeva sul bus principale del computer in Run Mode o sul DIP-Switch in Program Mode, ma non era un problema, in quanto le RAM scrivevano solo in corrispondenza del segnale RI (RAM In);
  - il bus "OUT" collegava il transceiver '245 e le uscite Data Out delle RAM; l'uscita del transceiver veniva abilitata solo in corrispondenza del segnale /RO (RAM Out).

- Con la RAM single-port le linee Data sono invece le stesse per Write e Read. A seconda dell'operazione da eseguire, si attivano due percorsi diversi, come già discusso parlando del doppio bus. Per scrivere sulla RAM la sequenza era questa:

  - Output Enable /OE LO fisso;
  - Chip Enable /CE LO fisso (in realtà il datasheet mostra ↘↗, ma, come segnalava The8BitEnthusiast, lo si può tenere fisso LO);
  - /WE ↘↗ (che deve essere "contenuto" all'interno del ciclo ↘↗ di /CE, se /CE non viene mantenuto fisso LO).

Riprendendo il datasheet del [62256](https://www.alliancememory.com/wp-content/uploads/pdf/AS6C62256.pdf) a pagina 6 troviamo anche un'altra possibilità per scrivere, indicata come "WRITE CYCLE 2 (CE# Controlled)", ma non l'ho approfondita, perché mantenere /OE e /CE LO fissi sia per le letture sia per le scritture semplicava il disegno del modulo, e inoltre non volevo fare lo sperimentatore - il mio desiderio era quello di realizzare un modulo che funzionasse con certezza.

[![Prima versione del modulo RAM](../../assets/20-ram-1st.png "Prima versione del modulo RAM"){:width="100%"}](../../assets/20-ram-1st.png)

*Prima versione del Modulo di memoria (RAM) del BEAM.*

Riponevo grandi speranze su questo primo design.

Per scrivere in RAM in Run Mode:

- RI (che nel ciclo di Write è HI) abilita il '245 superiore mediante una NOT;
- /RO (che nel ciclo di Write è HI) porta /OE a HI, disabilitando l'output del 62256;
- /CE (aka /CS, Chip Select) è fisso LO in quanto connesso a ground;
- i valori presenti nel bus o nel dip-switch vengono presentati via MUX al transceiver che, a sua volta, presenta tali valori alle porte della RAM;
- Zd, e dunque /WE,  è HI, in quanto il pin 1 della NAND è LO
- al rising edge del clock l'uscita della NAND passa da HI a LO per il tempo RC e l'impulso /WE ↘↗ viene trasmesso alla RAM attraverso il MUX '157: la RAM memorizza quanto trova nelle sue porte di ingresso D0-D7.

In Program Mode l'ingresso I0d del MUX è HI grazie alla resistenza di pull-up da 1K, ma premendo il bottone si crea un impulso negativo che attiva un ciclo /WE ↘↗: anche in questo caso la RAM memorizza quanto trova nelle sue porte di ingresso D0-D7.

Per leggere dalla RAM si devono soddisfare le seguenti condizioni:

- /CE LO, cioè il chip deve essere abilitato
- /WE HI, dunque non devo attivare il segnale di scrittura
- /OE LO, cioè le uscite devono essere attivate

La sequenza degli eventi è dunque la seguente:

- RI è LO, dunque la NOT mi disabiliterà il XCVR superiore che non metterà dunque alcunché in utput verso la RAM;
- RO LO abilita il XCVR inferiore e /OE della RAM, che viene attivata in output;
- /CE (aka /CS, Chip Select) è fisso LO in quanto connesso a ground;
- nel frattempo WE è HI (e dunque la scrittura è inibita) perché:
  - in Run Mode RI LO mantiene HI all'uscita della NAND connessa a I1d del MUX (se uno dei due input della RAM è LO, l'uscita è HI) e dunque anche /WE è fisso HI, inibendo la scrittura;
  - in Program Mode l'ingresso I0d del MUX è fisso HI grazie alla resistenza da 1K, perciò anche in questo caso non vi è scrittura.

Una analisi successiva di questo schema, che "all'occhio" era molto bello, mi evidenziava che probabilmente avevo gestito correttamente il discorso del "bus interno" e che anche la fase di output poteva essere funzionale, mentre nella fase di input notavo ridondanze superflue: avrei potuto far lavorare il transceiver inferiore bidirezionalmente a seconda della necessità per far interagire RAM e BUS e avrei potuto anche eliminare i MUX e collegare il transceiver superiore direttamente al dip-switch, attivandolo solo al momento opportuno per la programmazione manuale della RAM. In pratica, ritornavo alla soluzione concepita da The8BitEnthusiast, senza tuttavia aver ancora acquisito capacità e autonomia sufficienti per progettare una soluzione di attivazione just-in-time dei transceiver come aveva fatto lui.

### Seconda versione del modulo RAM

Stabilito che non sarei riuscito a far funzionare una soluzione just-in-time, avevo dunque provato a ridisegnare lo schema con un solo transceiver e mantenendo i MUX per gestire l'input della RAM, facendolo provenire dal bus o dal dip-switch a seconda della modalità Program o Run-Mode.

[![Seconda versione del modulo RAM](../../assets/20-ram-2nd.png "Seconda versione del modulo RAM"){:width="100%"}](../../assets/20-ram-2nd.png)

*Seconda versione del Modulo di memoria (RAM) del BEAM.*

Questo è anche il momento ufficiale della nascita del nome BEAM 😁. Dopo aver realizzato il disegno, mi sembrava di aver aggiunto più chip (per la logica) rispetto a prima = maggior complessità 🙄. Tuttavia, l'idea continuava a piacermi ed era un interessante esercizio logico per provare a sfruttare l'unico transceiver invertendone la direzione a seconda dell'operazione da fare.

Avevo provato a chiedere un [consiglio](https://www.reddit.com/r/beneater/comments/10inkvs/8bit_computer_ram_module_would_this_work/) su Reddit e il solito The8BitEnthusiast, instancabile, mi aveva dato alcune indicazioni e una risposta tutto sommato positiva.

Per analizzare tutti gli stati logici possibili avevo preparato una tabella di riepilogo con la quale verificare se il comportamento del modulo fosse in linea con le aspettative; la tabella mostrata in seguito è solo una parte di quella completa.

![Tabella analisi modulo RAM](../../assets/20-ram-2nd-table.png "Tabella analisi modulo RAM"){:width="100%"}

*Tabella riepilogativa analisi stati logici seconda versione modulo RAM.*

A un certo punto ho capito che nello schema c'era un problema piuttosto importante: i due MUX 157 proiettavano in continuazione i loro input verso il bus interno, causando un possibile short con RAM quando questa si trovava in output mode. Ho dunque pensato di sostituire i 2x '157 con i '257, che sono tri-state.

![Tabella analisi rivista modulo RAM](../../assets/20-ram-2nd-table2.png){:width="100%"}

*Tabella riepilogativa rivista analisi stati logici seconda versione modulo RAM.*

Quando sono arrivato a questo punto ho realizzato di aver lavorato inutilmente: a cosa mi servono due MUX e un 245? Sono sufficienti due 245, uno per gestire I/O sul bus, uno per gestire DIP SW, che in effetti è come aveva fatto 8bitEnthusiast! Ero stato illuminato e in quel momento mi è sembrato tutto chiaro!

Nel frattempo avevo anche iniziato a rinominare i segnali logici prendendo come punto di vista il computer e non il modulo:

| Vecchio | Descrizione | Nuovo | Descrizione |
|-        |-            |-      |-            |
| RI      | RAM In      | WR    | Write RAM   |
| RO      | RAM Out     | RR    | Read RAM    |

e così via.

### Terza versione del modulo RAM

Ho dunque riscritto la "truth table" del modulo RAM:

![Tabella analisi modulo RAM - terza versione](../../assets/20-ram-3rd-table.png "Tabella analisi modulo RAM - terza versione"){:width="100%"}

*Tabella analisi modulo RAM - terza versione.*

I due asterischi in tabella \*\* mi servivano a ricordare che dovevo stare particolarmente attento alla preparazione del microcode e che non dovevo mai avere /WR e /RR attivi contemporaneamente, perché non avevo previsto un controllo di eventuale conflitto. Per evitare conflitti avrei dovuto mettere su /WR una NAND a 3 ingressi che si attiva solo per CLK, RR e /WR, cioè /WE = CLK \* RR \* /WR, in altre parole: posso scrivere sulla RAM *solo* se non la sto già leggendo.

[![Terza versione del modulo RAM](../../assets/20-ram-3rd.png "Terza versione del modulo RAM"){:width="100%"}](../../assets/20-ram-3rd.png)

*Terza versione del Modulo di memoria (RAM) del BEAM.*

La gestione dei segnali passa in maniera importante attraverso il MUX '157, così come avviene anche nello schema di The8BitEnthusiast, con qualche differenza.

Ecco un riassunto degli stati possibili:

- **/PROG**: In Program Mode il MUX attiva gli ingressi I0a, I0b, I0c e I0d. In Run Mode sono attivi I1a, I1b, I1c e I1d.
  - Za è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva /WE, scrivendo sulla RAM.
  - Zb è normalmente HI attraverso la resistenza che lo collega a Vcc, ma passa a LO premendo il tasto di programmazione e attiva il transceiver che interconnette RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR WR**: Read RAM e Write RAM non sono attivi.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso HI e disattiva il transceiver che interconnette RAM e bus.
- **RR /WR**: Read RAM non è attivo, Write RAM è attivo.
  - Za è normalmente HI (perché la NAND su I1a ha un'ingresso LO), ma in corrispondenza dell'impulso di clock passa a LO e attiva /WE, scrivendo sulla RAM.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è normalmente HI, dunque disattiva il transceiver che interconnette RAM e bus, ma in corrispondenza dell'impulso di clock passa a LO (perché Za passa a LO), attivando il transceiver.
- **/RR WR**: Read RAM è attivo, Write RAM non è attivo.
  - Za è fisso HI (perché la NAND su I1a ha un ingresso fisso LO) e la scrittura su RAM è inibita.
  - Zb è fisso HI e inibisce il transceiver tra RAM e dip-switch.
  - Zc è fisso LO e attiva il transceiver tra RAM e bus.
- **/RR /WR**: Stato illegale.
