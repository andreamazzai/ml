---
title: "RAM"
permalink: /docs/ram/
excerpt: "Costruzione del modulo di memoria del BEAM computer"
---
Il limite principale del computer SAP di Ben Eater era sicuramente la modesta quantità di RAM indirizzabile, pari a 16 byte; era possibile caricare un semplice contatore da zero a 255 e viceversa, oppure un algoritmo di Fibonacci, ma nulla di più. Questo è stato lo stimolo primario per la realizzazione di un computer più potente.

All'approssimarsi del completamento della costruzione del SAP, ho iniziato a documentarmi su vari blog e forum per raccogliere idee su possibili miglioramenti ed espansioni.

### Primi studi

Il [canale Reddit](https://www.reddit.com/r/beneater/) dedicato ai progetti di Ben Eater è stato fondamentale in questo mio percorso.

Dal [primo articolo letto](https://www.reddit.com/r/beneater/comments/crl270/,8_bit_computer_memory_bootloader_and_display/) avevo tratto queste note:

> Addressable memory - so the idea here is to have 16bit's of addressable memory (about 65KB). This would greatly expand the capabilities compared to the 16 bytes of Ben's PC. This would affect the following things

>>1. Memory Address register - after the expansion of course the MAR would have to be 16 bits wide. Here i was considering using 2 x LS273 8 bit flip-flop along with an AND gate to allow the chip to have Input Enable signal (that would be implemented by AND-ing the CLK with an Input signal, since the chip has no InputEnable and i was unable to find one that is 8 bit wide, has CLR and inputenable and did not want to use LS173 but might reconsider)

Questo utente desiderava fare una espansione radicale del computer, passando da 16 byte a 64K; il mio desiderio era quello di crescere fino a 256 byte (e non complicarmi troppo la vita con un bus a 16 bit), ma alcune informazioni sono state comunque molto utili per una comprensione generale della questione.

Per indirizzare 64K di memoria serve infatti un registro MAR (Memory Address Register) da 16 bit (2^16 = 64K); anziché utilizzare 4 registri a 4 bit 74LS173 (il '173 è il Flip-Flop utilizzato in origine nel SAP), sembra più comodo utilizzare due registri Flip-Flop tipo D 74LS273 a 8 bit; uno svantaggio di questi ultimi, rispetto alla loro controparte a 4 bit, è che non hanno un ingresso di Enable, mentre il disegno del computer deve prevederlo, perché il MAR deve leggere dal bus un indirizzo di memoria solo quando necessario - e non ad ogni ciclo di clock. Il segnale MI (Memory Input) del SAP serve infatti per caricare l'indirizzo sul MAR solo quando è necessario; quando MI è attivo, al successivo rising edge del CLK il MAR carica il valore presente nel bus. Senza un ingresso di Enable, il FF '273 andrebbe a "registrare" il dato in ingresso ad ogni rising edge del clock.

In un progetto a 8 bit si potrebbero semplicemente utilizzare due '173 a 4 bit continuando a sfruttare i segnali di Enable nativamente disponibili. Bisogna dire che sarebbe comunque possibile utilizzare anche il '273 a 8 bit utilizzando una porta AND per *costruire* un segnale di Enable artificiale: i segnali CLK e MI sarebbero gli ingressi della AND, mentre l'output della AND si collegherebbe all'ingresso CLK del FF, che così sarebbe attivato solo quando, oltre al CLK, fosse contemporaneamente presente anche il segnale MI.

Il '273, al pari del '173, presenta un ingresso Clear / Reset (CLR), che nel MAR è necessario per resettare il registro - o almeno *credevo* necessario. Sembrava anche interessante l'ipotesi alternativa di usare un registro a 8 bit [74LS377](https://datasheetspdf.com/download_new.php?id=375625), che include 8 FF con Enable; tuttavia avevo realizzato che **non** fosse possibile procedere in tal senso, perché nel MAR serviva anche il CLR, non presente in questo chip. Come ormai si sarà capito, ho realizzato successivamente che il MAR può funzionare perfettamente anche senza un segnale di Clear / Reset.

Da notare che il '377 sarebbe in seguito diventato uno dei chip più utilizzati nel BEAM.

**METTERE QUALCHE FOTO E LINK**

>> 2. Program counter - would have to be expanded to a 16 bit counter (should be trivial to do that) I currently have tons of 8 bit counters combined with a register (and the 4 bit 161 counters that Ben used)

Come nel caso del MAR, per indirizzare 256 byte di RAM era necessario un registro Program Counter (PC) a 8 bit. Nel computer SAP era invece presente un contatore a 4 bit [74LS161](https://www.ti.com/lit/ds/symlink/sn54ls161a-sp.pdf) e dovevo pertanto cercare di combinarne due in cascata.

Sarebbe stato comodo utilizzare un singolo contatore a 8 bit, ma tra i chip disponibili sul mercato non ne ho trovato uno che includesse anche l'ingresso LOAD. Il LOAD permette il caricamento parallelo sul PC di uno specifico indirizzo al quale il computer deve saltare (ad esempio, per eseguire un'istruzione di salto assoluto o branch relativo a un determinato indirizzo, si carica quel valore nel PC).

Per combinare due chip a 4 bit è stato necessario, nonché molto utile, capire la differenza fra "Ripple Mode Carry" e "Carry Look Ahead": il datasheet del contatore 74LS161 riportava infatti questi due diversi esempi di collegamento di chip in cascata tra di loro.

**METTERE QUALCHE FOTO E LINK E SPIEGAZIONE PER RCA E ADDER ETC**

>> 3. Instruction register - now sh!t gets fun. Since i want to bomb completely, I am considering having a 3 byte Instruction register in a way mimicking the BIU block & instruction queue in a 8086. The idea here is to split the instruction register in 3 bytes. 1 byte would be for the instruction/opcode alone, the second and third are going to hold the data of the instruction (either address or an immediate value). This would allow you to address the entire memory space. Also instructions can be of different size. For example. OUT (move to out register) would be just 1 byte wide, LDI (load immediate) would be 2 bytes and LDA (load address/absolute) would be 3 bytes wide. The control logic would take care of the fetch cycle. Or in other words since you know the instruction you are execution you would know how much bytes the instruction is and thus fetch either 1, 2 or 3 bytes from the RAM.

Dopo aver letto questo punto avevo iniziato a raccogliere i miei pensieri per l'espansione di memoria a 256 byte, idea che riprendo in seguito in questa pagina. Non avevo intenzione di costruire un IR a più byte, cosa che mi sembrava piuttosto complessa per le mie capacità.

### Memorie con IO separati o IO comuni?

Fino ad ora, avevo quasi sostanzialmente dato per scontato di continuare ad usare chip di memoria con porte di Input e Output separati, esattamente come accade nel [74189](https://eater.net/datasheets/74189.pdf) utilizzato nel SAP. Tuttavia, in questo [post su Reddit](https://www.reddit.com/r/beneater/comments/hon6ar/74189_alternative/
), un utente evidenziava difficoltà nell'approvvigionamento dei 74189 e chiedeva lumi sull'uso del [62256](https://datasheetspdf.com/download_new.php?id=729365); ho così iniziato ad approfondire le caratteristiche di questo chip, aumentando nel contempo la mia comprensione di queste due diverse architetture.

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

Quello che iniziavo a capire era che per utilizzare una RAM con Common IO dovevo fare un "doppio passaggio" o qualcosa di simile. Come faccio ad avere sempre visibile il contenuto della locazione di memoria anche nel momento in cui setto le porte di IO del chip in modalità Input? Devo forse memorizzare il contenuto delle uscite della RAM in qualche latch e solo a quel punto disabilitare il chip prima di andarvi a scrivere? In seguito avrei capito che non era necessario un latch, ma che c'era un'altra strada.

In [questo post](https://www.reddit.com/r/beneater/comments/uot8pk/ram_module_using_65256/) un utente esponeva un disegno che credevo potesse andare bene, ma [nel suo schema](https://imgur.com/upvYjUX) le uscite dei multiplexer (MUX) sono sempre attive (i [multiplexer 74LS157](https://datasheetspdf.com/download_new.php?id=488136) non sono tri-state) e potrebbero creare contenzioso con le uscite della RAM quando questa è attiva in output; la soluzione poteva essere quella di aggiungere un altro [bus transceiver 74LS245](https://datasheetspdf.com/download_new.php?id=375533), oppure di utilizzare dei [MUX tri-state 74LS257](https://datasheetspdf.com/download_new.php?id=935737); intuivo qualcosa, relativamente alla necessità di gestire i segnali di controllo della RAM in maniera più ampia, controllando le interazioni con i MUX e con il/i transceiver di interfacciamento verso il bus del computer.

## MUX, Program Mode e Run Mode

A cosa servono i MUX nel modulo RAM (e nel MAR)? All'accensione, il contenuto della memoria RAM è vuoto / casuale, dunque dobbiamo avere sia la possibilità di programmare la RAM ("Program Mode"), sia di renderla poi visibile al bus del computer durante la normale esecuzione dei programmi ("Run Mode").

- La modalità **Program Mode** è manuale e sfrutta dei dip-switch per scrivere manualmente sulla RAM per caricare i programmi (o leggere il contenuto della RAM in stile "debug mode").

- La modalità **Run Mode** è la modalità di esecuzione, nella quale la RAM viene indirizzata dal MAR e viene acceduta in lettura / scrittura esclusivamente dal bus del computer.

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

[![RAM e MAR con doppio bus](../../assets/20-ram-ruclh.png "RAM e MAR con doppio bus"){:width="66%"}](../../assets/20-ram-ruclh.png)

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

Notare la configurazione del chip di RAM: i segnali CE ed OE sono sempre attivi, che significa che l'utente ha deciso di utilizzare la modalità di scrittura definita come "WE# Controlled" definita a pagina 6 del [datasheet](https://datasheetspdf.com/download_new.php?id=729365) del 62256. Rileggendo questi appunti diverso tempo dopo aver completato il mio progetto, mi sembra tutto facile, ma la comprensione delle modalità di scrittura della RAM è stata in realtà piuttosto lunga.

Da notare inoltre che anche questo utente non usa il CLR sui FF '273 di input del MAR - a pensarci, potrebbe realmente non servire, perché ogni volta che ho bisogno di accedere alla RAM, vado preventivamente a settare sul MAR l'indirizzo desiderato. Forse il reset all'accensione è più estetico che altro.

E' stato in questo momento (circa agosto / settembre 2022) che ho scoperto l'**NQSAP**, inserendolo tra i miei appunti come "c'è questo [https://tomnisbet.github.io/nqsap/docs/ram/](https://tomnisbet.github.io/nqsap/docs/ram/) che sembra aver fatto delle belle modifiche al suo computer" 😁; ho deciso di seguire questo progetto perché permetteva di costruire un instruction set come quello del 6502 che, come scoprirò in seguito, richiederà un numero elevato di indirizzi per il microcode delle EEPROM.

Tra i vari link sondati, c'era anche [questo post Reddit](https://www.reddit.com/r/beneater/comments/h8y28k/stepbystep_guide_to_upgrading_the_ram_with/), che molti utenti hanno trovato ben fatto, ma che io ho trovato molto difficile da digerire in quanto mancante di uno schema.

Per aggiungere un ulteriore link utile per la comprensione delle architetture del modulo di RAM, evidenzio questo [post su Reddit](https://www.reddit.com/r/beneater/comments/ad2uko/upgrading_the_ram_module_to_256_bytes/). Le spiegazioni sono molto ben fatte e utili. Il chip di RAM utilizzato è interessante perché si presenta come due RAM distinte, ognuna con accessi dedicati e un segnale di Busy per gestire le richieste parallele sulla stessa locazione. Altro aspetto degno di nota nell'implementazione di questo utente è la possibilità di aumentare fino a 256 il numero di istruzioni del computer, grazie alla scelta di utilizzare un byte intero per l'istruzione ed un eventuale byte successivo per l'operando, anziché avere un unico byte di cui i 4 Most Significant Bit (MSB) rappresentano l'opcode e di cui i 4 Least Significant Bit (LSB) sono l'operando, come nel SAP di Ben Eater.

Un aspetto collaterale (ma importantissimo) dell'aumento del numero di istruzioni era la necessità di aumentare la dimensione delle EEPROM ospitanti il microcode: volendo gestire (fino a) 256 istruzioni, erano necessari 8 bit di istruzioni, 3 di step e 2 di flag = 13 pin totali, portanto si rendevano necessarie delle 28C64... e avevo dimenticato che mi sarebbe servito un bit aggiuntivo per la selezione delle due EEPROM! In quel momento, non sapevo ancora che avrei speso intere settimane a comprendere il fantastico modulo dei Flag dell'NQSAP di Tom Nisbet, che ha un approccio completamente diverso e che non necessita di segnali in uscita dalle EEPROM.

## Gestione della RAM

Tra i post più utili relativi alla comprensione dei segnali di gestione di RAM e MAR per il modulo di memoria con IO comuni, c'è certamente il [Question about RAM replacement](https://www.reddit.com/r/beneater/comments/ut1oud/8bit_question_about_ram_replacement/), nel quale il moderatore The8BitEnthusiast invita a consultare la sua (eccellente, aggiungo io) realizzazione, che ho preso ad esempio e ispirazione.

[![Modulo RAM dell'utente Reddit The8BitEnthusiast](../../assets/20-ram-the8bit_enthusiast.png "Modulo RAM dell'utente Reddit The8BitEnthusiast"){:width="66%"}](../../assets/20-ram-the8bit_enthusiast.png)

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

[![Write Cycle](../../assets/20-ram-write-cycle-twhz.png "WWrite Cycle"){:width="50%"}](../../assets/20-ram-write-cycle-twhz.png)

> Lo stesso segnale Zc che attiva la scrittura su RAM (WE) abilita anche i due '245; il datasheet del '245 specifica che la sua attivazione richiede 25 nanosecondi, che è un valore superiore a quello necessario alla RAM per attivarsi in Input, dunque il requisito è rispettato.

Molto, molto clever.

## Design dei moduli MAR e RAM del BEAM

Parallelamente agli studi dei lavori di altri utenti, avevo iniziato a lavorare sul disegno dei miei moduli MAR e RAM, non senza continuare a saltare di palo in frasca per approfondire temi ancora parzialmente oscuri o affrontare argomenti nuovi.

Ritornando alla dimensione delle EEPROM da utilizzare per il microcode, nei miei appunti trovo traccia di diverse revisioni, ad esempio:

- mi servono EEPROM 28C64 per avere 256 (8 bit) istruzioni + 3 step + 2 flag, ma dimenticavo che avendo due ROM gemelle dovevo gestirne anche la selezione e dunque aggiungere un ulteriore bit, pertanto mi servirebbero delle 28C128;
- avrei potuto però ridurre il numero di istruzioni a 64, dunque mi sarebbero bastati 6 bit per indirizzarle e ridurre così il numero totale di indirizzi richiesti...
- ... ma forse mi sarebbero serviti altri segnali di controllo oltre ai 16  disponibili in due ROM e dunque me ne sarebbe servita una terza... e dunque due bit di indirizzamento

Posso sicuramente dire che avevo le idee ancora confuse.

Nel frattempo avevo sviluppato lo schema del MAR a 8 bit, in grado di pilotare 256 indirizzi di memoria. Inizialmente avevo utilizzato due FF '173 a 4 bit, poi sostituiti da un unico FF '273 a 8 bit (sprovvisto di controllo dell'attivazione dell'output, che però non è necessario). Infine ho utilizzato un registro a 8 bit '377, altrettanto adatto al nostro scopo.

[![Memory Address Register](../../assets/20-ram-write-cycle-twhz.png "Memory Address Register"){:width="66%"}](../../assets/20-mar-beam.png)

*Memory Address Register (MAR) del BEAM.*

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