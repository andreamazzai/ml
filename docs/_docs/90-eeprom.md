---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
<small>[Il programmatore di EEPROM](#il-programmatore-di-eeprom) - [Schema](#schema) - [Spiegazione del codice](#spiegazione-del-codice) - [Le EEPROM e il loro contenuto](#le-eeprom-e-il-loro-contenuto) - [Link utili](#link-utili)</small>

[![EEPROM programmer](../../assets/eeprom/eeprom-programmer.png "EEPROM programmer"){:width="100%"}](../../assets/eeprom/eeprom-programmer.png)

Tutti i progetti descritti in questa pagina sono basati su Arduino, ad evidenza della sua versatilità.

## Il programmatore di EEPROM

La mia prima esperienza con la programmazione di EEPROM risale alla costruzione del computer SAP-1 di Ben Eater e alla realizzazione del programmatore basato sui suoi schema e sketch. Questo progetto, molto semplice, permetteva di programmare le EEPROM del microcode, anche se la scrittura risultava particolarmente lenta. Ciononostante, la programmazione di una EEPROM (dapprima manualmente, poi con il programmatore) è stato un momento euforico - mai avrei pensato di riuscire a comprendere i (tutto sommato semplici) meccanismi che lo rendevano possibile.

[![Schema del programmatore di EEPROM del computer SAP](../../assets/eeprom/eeprom-ben.png "Schema del programmatore di EEPROM del computer SAP"){:width="100%"}](../../assets/eeprom/eeprom-ben.png)

*Schema del programmatore di EEPROM del computer SAP.*

Durante lo studio del computer NQSAP di Tom Nisbet e la rivisitazione del suo progetto per creare il computer BEAM, mi ero reso conto che il numero di EEPROM da programmare (quattro anziché due) e la loro [dimensione](../control/#instruction-register-e-istruzioni) avrebbero ulteriormente aumentato i tempi di programmazione di ogni revisione del microcode. Il programmatore di Ben era funzionale, ma non sfruttava la funzionalità di scrittura a *pagine* delle EEPROM moderne, che permette la programmazione di una EEPROM da 32KB in pochi secondi. Ciononostante, continuavo a usare questo programmatore, con tempi di scrittura di circa 90 secondi per ogni EEPROM.

Con il progredire della costruzione dei moduli del BEAM, le riprogrammazioni necessarie ad implementare l'Instruction Set e risolvere i bug del microcode diventavano sempre più frequenti; il tempo trascorso ad attendere il completamento dei cicli di programmazione iniziava a diventare frustrante.

Tom aveva sviluppato un suo programmatore, il <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyPROM</a>, che supportava una varietà di EPROM, EEPROM e memorie Flash. Questo programmatore includeva un software molto ricco e molto veloce grazie al supporto della modalità di scrittura *Page Write*. Una versione ridotta del software del TommyPROM - col semplice scopo di programmare le EEPROM 28C256 - è stata utilizzata da Tom e pubblicata sul suo repository dell'NQSAP.

Non volevo iniziare un nuovo progetto, quello di un nuovo programmatore di EEPROM, mentre stavo ancora lavorando sul BEAM. Tuttavia, a un certo punto ho realizzato che avrei speso meno tempo nel costruire un programmatore basato sul TommyPROM rispetto al tempo che avrei sprecato continuando ad utilizzare il programmatore elementare di Ben.

[![Schema del programmatore di EEPROM TommyPROM di Tom Nisbet](../../assets/eeprom/TommyPROM-nano-sch.png "Schema del programmatore di EEPROM TommyPROM di Tom Nisbet"){:width="100%"}](../../assets/eeprom/TommyPROM-nano-sch.png)

*Schema del programmatore di EEPROM TommyPROM di Tom Nisbet.*

Il mio programmatore iniziale era uguale a quello sviluppato da Ben Eater e utilizzava i <a href="https://www.ti.com/lit/ds/symlink/sn74hc595.pdf" target="_blank">74HC595</a> anziché i <a href="https://www.ti.com/lit/ds/symlink/sn74ls164.pdf" target="_blank">74HCT164</a> utilizzati in origine da Tom. In una sezione della documentazione, Tom evidenziava che il suo codice non funzionava sul progetto di Ben e che necessitava di significative modifiche. Se si desideravano i benefici del TommyPROM sull'hardware di Ben, Tom indicava che la strada più facile non era quella di modificare il software, bensì di adattare l'hardware rendendolo uguale a quello del TommyPROM.

Nella mia testardaggine, avevo optato per un compromesso: modificare solo parzialmente l'hardware sviluppato nel progetto iniziale di Ben e contemporaneamente approfittarne per studiare e aggiornare il software di Tom rendendolo compatibile col mio hardware ed effettuando il minor numero di modifiche possibili. In quel momento non sapevo che il programmatore rivisitato avrebbe visto la luce in un tempo sostanzialmente limitato, ma che la corretta implementazione della modalità *Page Write* avrebbe richiesto uno sforzo molto più esteso.

Le modifiche minime necessarie rispetto al programmatore originale del SAP-1 erano le seguenti:

1. il pin Write Enable della EEPROM (/WE) non più controllato dal segnale D13 di Arduino, bensì da A2.
2. il pin Chip Enable della EEPROM (/CE) non più connesso a ground (chip sempre attivo), bensì controllato dal pin A0 di Arduino.
3. il pin Output Enable della EEPROM (/OE) non più connesso allo Shift Register, ma controllato dal pin A1 di Arduino.

Come spiegava Tom, D13 controllava il segnale Write Enable del programmatore di Ben Eater. Poiché D13 è internamente connesso al LED integrato su Arduino, che lampeggia durante il boot, a ogni accensione del programmatore potrebbero verificarsi scritture indesiderate nella EEPROM. Questo non rappresentava un problema per lo sketch di Ben, poiché a ogni esecuzione la EEPROM veniva completamente riprogrammata, sovrascrivendo eventuali scritture spurie. Tuttavia, dato che il TommyPROM poteva essere utilizzato anche per la sola lettura di una EEPROM, le scritture iniziali indesiderate avrebbero potuto causare problemi: di qui la necessità di governare /WE con un segnale diverso da D13.

Altro aspetto da tenere in considerazione era il pin /OE delle EEPROM: gli Shift Register  '164 non sono dotati di una memoria intermedia che funga da *latch* durante il caricamento. Di conseguenza, gli output vengono immediatamente modificati ad ogni Rising Edge del clock, sottoponendo /OE a possibili attivazioni indesiderate. Viceversa, i 74HC595 permettono il caricamento seriale dei dati in un latch interno e la erogazione contemporanea di tutti i nuovi stati di output al Rising Edge di un segnale dedicato (RCLK), annullando il rischio di attivazioni indesiderate di /OE. Tuttavia, durante la realizzazione delle modifiche al programmatore non avevo ben compreso questo aspetto e avevo deciso di seguire l'indicazione di Tom, cioè di dedicare un output di Arduino specificamente ad /OE della EEPROM. In altre parole, la modifica al punto 3 si sarebbe potuta evitare, ma in quel momento non l'avevo capito.

## Schema

[![Schema del programmatore di EEPROM del computer BEAM](../../assets/eeprom/90-eeprom-schema.png "Schema del programmatore di EEPROM del computer BEAM"){:width="100%"}](../../assets/eeprom/90-eeprom-schema.png)

*Schema del programmatore di EEPROM del computer BEAM.*

## Spiegazione del codice

Il programmatore di EEPROM sviluppato non è interattivo, a differenza del ben più completo TommyProm. Una volta fatto partire, esegue i seguenti passi ed è pronto per un reset e la programmazione di una nuova EEPROM:

1. Calcolo di un checksum dei dati da scrivere sulla EEPROM
2. Sblocco della EEPROM
3. Cancellazione della EEPROM
4. Programmazione della EEPROM
5. Blocco della EEPROM
6. Calcolo del checksum rileggendo i dati scritti e confronto col valore calcolato al punto 1
7. Stampa del tempo trascorso

### Le EEPROM e il loro contenuto

Per governare i 29 segnali di controllo di ALU, RAM, SP, registri ecc. sono necessarie quattro EEPROM, cioè un totale di 32 bit.

- Ogni EEPROM mette a disposizione 8 bit in output, cioè un byte.
- Poiché ogni istruzione del BEAM è composta da 16 step, sono necessarie EEPROM di dimensione 256 * 16 = 4096 byte dedicati a decodifica delle istruzioni e impostazione degli opportuni segnali in uscita.
- Per indirizzare 4096 byte sono necessari 12 pin di indirizzamento (2^8 = 256 istruzioni e 2^4 = 16 step), cioè da A0 a A11; quattro EEPROM da 4KB, ognuna delle quali programmata con il proprio microcode, possono svolgere il compito richiesto.

Vediamo in dettaglio il microcode di alcune istruzioni di esempio, in particolar modo HLT (blocca l'esecuzione del codice), JMP (salta a un nuovo indirizzo definito nella locazione di memoria indicata dall'operando) e CPX (confronta il registro X con l'operando). L'istruzione più lunga tra quelle rappresentate è CPX, la cui durata è di 7 step (da 0 a 6); alcune istruzioni del BEAM raggiungono una lunghezza di 10 step.

![ ](../../assets/eeprom/microcode-esempio.png)

Ogni step abilita uno o più segnali di controllo: ad esempio il settimo step dell'istruzione CPX attiva contemporaneamente RA, WH, PCI e NI.

Come si può vedere nello [sketch](/code/Beam-Microcode.ino) Arduino del programmatore di EEPROM (righe da 105 a 145), ad ognuna di essee corrispondono univocamente alcuni dei 32 segnali / 32 bit discussi poco sopra:

[![Definizione dei segnali di controllo gestiti da ogni EEPROM](../../assets/eeprom/eeprom-pins.png "Definizione dei segnali di controllo gestiti da ogni EEPROM"){:width="100%"}](../../assets/eeprom/eeprom-pins.png)

*Definizione dei segnali di controllo gestiti da ogni EEPROM.*

Si può dedurre che ogni EEPROM contenga solamente *una parte* del microcode di ogni istruzione, in particolar modo la porzione relativa ai segnali cablati sugli output di quella determinata EEPROM. Ma come è suddiviso il microcode nelle quattro EEPROM?

[![Rappresentazione di alcune istruzioni del microcode di ogni EEPROM](../../assets/eeprom/4-eeprom-rappresentazione.png "Rappresentazione di alcune istruzioni del microcode di ogni EEPROM"){:width="100%"}](../../assets/eeprom/4-eeprom-rappresentazione.png)

*Rappresentazione di alcune istruzioni del microcode di ogni EEPROM.*

In pratica, si devono tenere in considerazione i segnali associati ad ogni EEPROM e indicare quali di questi debbano essere attivi ad ogni combinazione di istruzione / step.

Anziché effettuare quattro programmazioni distinte, risulta però molto più comodo (anche se più dispendioso) utilizzare EEPROM di dimensioni maggiori e scrivere su ognuna di esse, in sequenza, i quattro microcode specifici di tutte le EEPROM, effettuando quattro programmazioni tutte uguali.

 | Microcode | Indirizzo<br>iniziale<sub>base10</sub> | Indirizzo<br>finale<sub>base10</sub> | Indirizzo<br>iniziale<sub>hex</sub> | Indirizzo<br>finale<sub>hex</sub> | A12 | A13 |
 |------------|-------|-------------------|--------|--------|---|---|
 | 1°         | 0     | 4095              | 0x0000 | 0x0FFF | 0 | 0 |
 | 2°         | 4096  | 8191              | 0x1000 | 0x1FFF | 1 | 0 |
 | 3°         | 8192  | 12287             | 0x2000 | 0x2FFF | 0 | 1 |
 | 4°         | 12288 | 16383             | 0x2000 | 0x3FFF | 1 | 1 |

Impostando a valori fissi le linee di indirizzamento A12 e A13 è possibile mettere in output su ogni EEPROM la porzione corretta di microcode; si vedano le connessioni fisse a Vcc o GND nello [schema](../control/#schema) della Control Logic.

In altre parole, abbiamo 256 istruzioni che si sviluppano in 16 step, ognuno composto da una Control Word da 32 bit (4 byte) = 16.384 byte totali. Si dovrebbero programmare 4 EEPROM da 4K: una per i primi 8 bit della Control Word, una per gli 8 bit successivi e così via. Anziché programmare quattro diverse EEPROM da 4K con il microcode specifico, è possibile programmare 4 EEPROM da 16K. Nei primi 4K byte si posiziona la codifica per i primi 8 bit della Control Word, nei secondi 4K la codifica per i secondi 8 bit, e così via. Infine, si settano opportunamente gli indirizzi A12 e A13 delle varie EEPROM, in modo che ognuna esponga solo la porzione specifica di microcode relativa ai segnali di controllo cablati sui suoi output.

In relazione al conteggio della dimensione, si veda anche la sezione [Instruction Register e Istruzioni](../control/#instruction-register-e-istruzioni).

Fatta questa premessa, utile per capire la distribuzione del microcode nelle varie EEPROM, possiamo analizzare alcuni aspetti importanti dello sketch Arduino del programmatore.

### Calcolo del CRC pre-programmazione

Per capire se il programmatore stesse funzionando correttamente, avevo introdotto una verifica della corrispondenza tra il Cyclic Redundancy Check (CRC) calcolato prima della scrittura del microcode e quello calcolato dalla rilettura della EEPROM alla fine della sua programmazione.

Il codice che programma le EEPROM, come si vedrà in seguito, dapprima prepara i 32 bit / 4 byte di microcode di ogni step dell'istruzione corrente (routine **buildInstruction**) e li memorizza in un array tipo uint32_t di lunghezza 16, cioè 4 byte * 16 = 64 Byte; successivamente, le scritture avvengono in questa sequenza (routine **writeOpcode**):

- il 1° byte di ogni step viene scritto sui primi 16 byte contigui della 1a porzione della EEPROM (indirizzo 0x0000 a 0x000F)
- il 2° byte di ogni step viene scritto sui primi 16 byte contigui della 2a porzione della EEPROM (indirizzo 0x1000 a 0x100F)
- il 3° byte di ogni step viene scritto sui primi 16 byte contigui della 3a porzione della EEPROM (indirizzo 0x2000 a 0x200F)
- il 4° byte di ogni step viene scritto sui primi 16 byte contigui della 4a porzione della EEPROM (indirizzo 0x3000 a 0x300F)

Il contatore dell'istruzione viene poi incrementato e vengono preparati i 16 step dell'istruzione successiva, che vengono scritti considerando l'offset di 16 byte di lunghezza di ogni istruzione, dunque:

- il 1° byte di ogni step viene scritto sui 16 byte contigui successivi della 1a porzione della EEPROM (indirizzo 0x0010 a 0x001F)
- il 2° byte di ogni step viene scritto sui 16 byte contigui successivi della 2a porzione della EEPROM (indirizzo 0x1010 a 0x101F)
- il 3° byte di ogni step viene scritto sui 16 byte contigui successivi della 3a porzione della EEPROM (indirizzo 0x2010 a 0x201F)
- il 4° byte di ogni step viene scritto sui 16 byte contigui successivi della 4a porzione della EEPROM (indirizzo 0x3010 a 0x301F)

e così via fino alla fine delle istruzioni.

Per semplicità, il calcolo del CRC non è effettuato secondo la logica di scrittura "frazionata" della EEPROM, ma segue una più intuitiva logica sequenziale degli indirizzi da 0x0000 a 0x3FFF.
Per calcolare in sequenza i valori degli step, viene eseguita una serie di cicli annidati: per ogni porzione di EEPROM e per ogni istruzione si generano i 16 step, che vengono utilizzati per calcolare il CRC. In questo modo, 
in quel momento (1a EEPROM, 2a EEPROM etc) e li utilizzo per calcolare il checksum. */


 ()La stessa routine buildInstruction è utilizzata anche dalla routine di calcolo del CRC.

~~~c++

/* Il calcolo del CRC prevede la ricezione dei dati in sequenza (da 0x0000 a 0x3FFF). Non posso calcolare il CRC
durante la programmazione della EEPROM, perché in quel momento genero un opcode completo e ne scrivo i 4 segmenti
da 16 byte "frazionandoli" sulla EEPROM corrispondente (1a, 2a, 3a, 4a).
Faccio dunque un ciclo: per ogni ROM genero le 256 istruzioni in sequenza; ricavo solo i 16 byte che mi interessano
in quel momento (1a EEPROM, 2a EEPROM etc) e li utilizzo per calcolare il checksum. */
uint16_t calcCRC16_pre(void)
{
  crc = 0xFFFF;
  uint16_t polynomial = 0x1021;
  for (uint8_t rom = 0; rom < 4; rom++)
  {
    for (uint16_t opcode = 0; opcode < 256; opcode++)
    {
      buildInstruction(opcode);
      for (uint8_t step = 0; step < NUM_STEPS; step++)
      {
        // printInstruction(rom, opcode, step); // solo per Debug
        crc = calculate_crc(((code[step]) >> (24 - 8 * rom)) & 0xFF, crc, polynomial);
      }
    }
  }
  return crc;
}







// ********************   EEPROM PROGRAM   ********************
void eeprom_program()
{
  Serial.println("\n+++++++++++++++++++++++++++++");
  Serial.println("Programming EEPROM");
  for (uint16_t opcode = 0; opcode < 256; opcode++)
  {
    buildInstruction((uint8_t) opcode);
    // printOpcodeContents(opcode);
    char buf[80];
    if ((opcode) % 16 == 0)
    {
       Serial.print("Opcode: 0x");
    }
    sprintf(buf, "%02X ", opcode);
    Serial.print(buf);
    if ((opcode + 1) % 16 == 0)
    {
      Serial.println("");
    }
    writeOpcode((uint8_t) opcode);
  }
  Serial.println("Done!");
}

//
// ************************************************************
// ************** COPIA BLOCCO MICROCODE IN RAM ***************
// ************************************************************
void buildInstruction(uint8_t opcode)
{
  // Ogni istruzione = 16 step: i primi 2 uguali per tutti; 8 presi dal template; 6 che - per ora - non utilizzo, dunque a 0.
  code[0] = F1; // Fetch instruction from memory (RPC, WM - Read Program Counter, Write Memory Address Register)
  code[1] = F2; // Opcode into IR (sets ALU mode and S bits) (RR, WIR, PCI - Read RAM, Write Instruction Register, Program Counter Increment)
  // Copia parte del template dalla memoria Flash alla memoria RAM
  // Gli step presenti nel template sono 8 e ognuno di essi sono 4 byte, uno per ogni ROM
  // Ogni step è composto da 4 byte che vanno allo stesso indirizzo su ognuna delle EEPROM; è una vista delle 4 ROM "affiancate"
  // In ogni ROM 0x000-0x00F è 1° opcode, 0x010-0x01f è 2° opcode... 0xFF0-FFF è 256° opcode
  memcpy_P(code + 2, template0[opcode], NUM_TEMPLATE_STEPS * 4);
  code[10] = code[11] = code[12] = code[13] = code[14] = code[15] = 0;
}

// ************************************************************
// **************** PREPARA E SCRIVE OPCODE *******************
// ************************************************************
void writeOpcode(uint8_t opcode)
{
  // Sono 4 ROM, mappate $0-$0FFF, $1000-$1FFF, $2000-$2FFF, $3000-$3FFF
  // Ogni opcode è lungo 16 step = 16 byte per ogni ROM, dunque 256 * 16 = 4096 byte = $1000
  for (uint8_t rom = 0; rom < 4; rom++)
  {
    // Serial.print(" - ROM: ");
    // Serial.print(rom);
    // Serial.print(" - ");
    for (uint8_t step = 0; step < NUM_STEPS; step++) // ciclo fra i 16 step di ogni opcode e dunque li scrivo consecutivamente su ogni EEPROM
    {
      uint16_t address;
      address = 0x1000 * rom;
      address += opcode * NUM_STEPS;
      address += step;
      // printStep(step, address, rom);
      writeEEPROM(address, ((code[step]) >> (24 - 8 * rom)) & 0xFF); // code[step] prende tutti i 4 byte delle 4 ROM "affiancate" e poi con shift seleziono il byte relativo ad ogni ROM specifica, ad esempio 1a ROM bit 25-32, poi 17-24 etc
    }
    byte b1Value;
    bool status = waitForWriteCycleEnd(((code[15]) >> (24 - 8 * rom)) & 0xFF, &b1Value);
    if (status == false)
    {
      Serial.print("\n******** Error in Opcode 0x");
      Serial.print(opcode, HEX);
      Serial.print(" - Value sent: 0x");
      Serial.print(((code[15]) >> (24 - 8 * rom)) & 0xFF, HEX);
      Serial.print(" - Value read: 0x");
      Serial.println(b1Value, HEX);
      Serial.print("Opcode: 0x");
    }
  }
}

// ************************************************************
// ******************** WRITE TO EEPROM ***********************
// ************************************************************
void writeEEPROM(uint16_t address, byte data)
{
  // Setto indirizzo e disabilito output EEPROM; solo in seguito abilito output di Arduino, così non causo cortocircuiti.
  // La EEPROM non ha resistenze in uscita; se EEPROM ha output attivo e Arduino è in input, si genera un cortocircuito.
  setAddress(address, /*outputEnable */ false /* cioè porto a uno /OE*, cosi posso scrivere sulla EEPROM*/);
  setDataBusMode(OUTPUT);
  enableChip();
  // 5 LSB = D0-D4 data bus; shiftare dato a DX di 3 posizioni
  // 6 MSB PORTD = D0-D5 bus; shiftare dato a SX di 5 posizioni
  PORTB = (PORTB & 0xE0) | (data >> 3); // data = BBBB.Bxxx ==> shift DX 3 pos. degli MSB BBBBB, che carico in PORTB0-PORTB4 del Nano, cioè D3-D7 del data bus
  PORTD = (PORTD & 0x1F) | (data << 5); // data = xxxx.xBBB ==> shift SX 5 pos. degli LSB BBB, che carico in PORTD5-PORTD7 del Nano, cioè D0-D2 del data bus
  strobeWE();
  disableChip();
}

void writeEEPROM2(uint16_t address, byte data)
{
  // Setto indirizzo e disabilito output EEPROM; solo in seguito abilito output di Arduino, così non causo cortocircuiti.
  // La EEPROM non ha resistenze in uscita; se EEPROM ha output attivo e Arduino è in input, si genera un cortocircuito.
  setAddress(address, /*outputEnable */ false /* cioè porto a uno /OE*, cosi posso scrivere sulla EEPROM*/);
  delayMicroseconds(2);
  setDataBusMode(OUTPUT);
  delayMicroseconds(2);
  enableChip();
  delayMicroseconds(2);
  // 5 LSB = D0-D4 data bus; shiftare dato a DX di 3 posizioni
  // 6 MSB PORTD = D0-D5 bus; shiftare dato a SX di 5 posizioni
  PORTB = (PORTB & 0xE0) | (data >> 3); // data = BBBB.Bxxx ==> shift DX 3 pos. degli MSB BBBBB, che carico in PORTB0-PORTB4 del Nano, cioè D3-D7 del data bus
  PORTD = (PORTD & 0x1F) | (data << 5); // data = xxxx.xBBB ==> shift SX 5 pos. degli LSB BBB, che carico in PORTD5-PORTD7 del Nano, cioè D0-D2 del data bus
  strobeWE();
  disableChip();
}

const uint32_t mMaxWriteTime = 20;  // Max time (in ms) to wait for write cycle to complete

bool waitForWriteCycleEnd(byte lastValue, byte *b1Ptr)
{
  // Codice tratto da Tom Nisbet e fatta qualche mia modifica
  // Vedi DATA Polling e Toggle Bit pagina 3 https://ww1.microchip.com/downloads/en/DeviceDoc/doc0006.pdf
  //
  // Verify programming complete by reading the last value back until it matches the
  // value written twice in a row.  The D7 bit will read the inverse of last written
  // data and the D6 bit will toggle on each read while in programming mode.
  //
  // Note that the max readcount is set to the device's maxReadTime (in uSecs)
  // divided by two because there are two 1 uSec delays in the loop (rimossi).  In reality,
  // the loop could run for longer because this does not account for the time needed
  // to run all of the loop code.  In actual practice, the loop will terminate much
  // earlier because it will detect the end of the write well before the max time.

  // * Nota che in https://github.com/TomNisbet/TommyPROM/issues/17 Tom dice che potrebbero verificarsi
  // errori in rilettura gestendo solo OE, dunque ha aggiunto CE, che "rispecchia meglio le waveforms
  // del datasheet", e in effetti "AC Read Waveforms" a pagina 6 del DS mostra che la Read completa è 
  // 1) attivare /CE e poi /OE
  // 2) disattivare /OE e /CE
  // mentre io faccio un po' diverso:
  // 1) attivare /OE e poi /CE
  // 2) disattivare /OE e /CE
  // e nel punto * non disabilito e riabilito CE come indicato da Tom... però mi funziona lo stesso.

  setDataBusMode(INPUT);
  uint32_t readCount;
  for (readCount = mMaxWriteTime * 1000 / 2; readCount > 0; readCount--)
  {
    enableOutput();
    enableChip();
    byte b1 = readDataBus();
    *b1Ptr = b1;
    disableOutput(); // *
    enableOutput();
    byte b2 = readDataBus();
    disableOutput();
    disableChip();
    if ((b1 == b2) && (b1 == lastValue))
    {
      return true;
    }
  }
  return false;

      /*
      // Segue il CODICE ORIGINALE di Tom; si vede che lui prima attiva il chip e poi l'output,
      // mentre io prima lo metto in output e poi lo attivo
      bool waitForWriteCycleEnd(byte lastValue) {
          setDataBusMode(INPUT);
          delayMicroseconds(1);
          for (int readCount = mMaxWriteTime * 1000 / 2; (readCount > 0); readCount--) {
              enableChip();
              enableOutput();
              delayMicroseconds(1);
              byte b1 = readDataBus();
              disableOutput();
              disableChip();
              enableChip();
              enableOutput();
              delayMicroseconds(1);
              byte b2 = readDataBus();
              disableOutput();
              disableChip();
              if ((b1 == b2) && (b1 == lastValue)) {
                  return true;
              }
          }

          return false;
      }
      */
}

// ************************************************************
// ******************** READ DATA FROM BUS ********************
// ************************************************************
// Read a byte from the data bus. The caller must set the bus to input_mode
// before calling this or no useful data will be returned.
byte readDataBus()
{
  return (PINB << 3) | (PIND >> 5);
  // legge PINB; serve PB0-PB4, 5x LSB di PORTB, che sono però MSB del data bus, dunque li traslo 3 posizioni a SX
  // legge PIND; serve PD5-PD7, 3x MSB di PORTD, che sono però LSB del data bus, dunque li traslo 5 posizioni a DX
}

// ************************************************************
// *********************** SET ADDRESS ************************
// ************************************************************
// prima metto il dato in D2 (SHIFT_DATA) e poi pulso D3 (SHIFT_CLK) per mandarlo
void setAddress(uint16_t address, bool outputEnable) // 2° parametro = outputEnable, se True setto /OE basso, se False /OE alto
{
  shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, (uint8_t) (address >> 8));
  // Qui vedrò sullo scope il SER e il SRCLK
  // L'indirizzo arriva in due byte. Quanto sopra gestisce la parte alta dell'indirizzo (che
  // sono i bit da A8 a A10, e poi aggiunge il bit 15, che controlla /OE
  // se il pin /OE = 0, attivo l'output, dunque leggo dalla EEPROM
  // se il pin /OE = 1, disattivo l'output, dunque scrivo sulla EEPROM
  // poiché uso "(outputEnable ? 0x00L: 0x80)", sto dicendo che se nella routine passo un "outputEnable" true,
  // setto a zero /OE e dunque attivo la lettura; se passo un "outputEnable" false, il MSb è HI e dunque attivo la scrittura
  shiftOut(SHIFT_DATA, SHIFT_CLK, MSBFIRST, (uint8_t) address);
  // Quanto sopra gestisce la parte bassa dell'indirizzo (da A0 ad A7)
  // Quanto sotto sblocca il 595
  // Qui vedrò sullo scope il RCLK
  digitalWrite(OE, outputEnable ? LOW : HIGH);
  digitalWrite(SHIFT_LATCH, LOW);
  digitalWrite(SHIFT_LATCH, HIGH);
  digitalWrite(SHIFT_LATCH, LOW);
}

// ************************************************************
// ********************* EEPROM ERASE *************************
// ************************************************************
void eeprom_erase(byte value)
{
  // value = 0xff; // overwrite value that was set on call - used for debug purposes
  Serial.println("\n+++++++++++++++++++++++++++++");
  Serial.print("Erasing EEPROM with value 0x");
  Serial.print(value, HEX);
  Serial.println("...");
  uint16_t salto = 64;

  for (uint16_t address = 0; address < 0x8000; address += salto)
  {
    for (uint16_t step = 0; step < salto; step++)
    {
      // printStep(step, address, rom);
      writeEEPROM(address + step, value);
    }
    byte b1Value;
    bool status = waitForWriteCycleEnd(value, &b1Value);
    if (status == false)
    {
      Serial.print("\n******** Error in address 0x");
      Serial.print(address, HEX);
      Serial.print(" - Value sent: 0x");
      Serial.print(value, HEX);
      Serial.print(" - Value read: 0x");
      Serial.println(b1Value, HEX);
      Serial.print("Opcode: 0x");
    }

    // ************************************************************
    // ******************** FUNZIONA NON TOCCARE ******************
    // ************************************************************

    /*   for (uint16_t opcode = 0; opcode < 1024; opcode++)
      {
        for (uint8_t step = 0; step < 16; step++)
        {
          uint16_t address;
          address = 0x0000;
          address += opcode * 16;
          address += step;
          // printStep(step, address, rom);
          writeEEPROM(address, value);
        }
        byte b1Value;
        bool status = waitForWriteCycleEnd(value, &b1Value);
        if (status == false)
        {
          Serial.print("\n******** Error in Opcode 0x");
          Serial.print(opcode, HEX);
          Serial.print(" - Value sent: 0x");
          Serial.print(value, HEX);
          Serial.print(" - Value read: 0x");
          Serial.println(b1Value, HEX);
          Serial.print("Opcode: 0x");
        } */
  }
  // printContents(0x0000, 512);
  Serial.println("Done!");
}

  // ************************************************************
  // ******************** READ FROM EEPROM **********************
  // ************************************************************
  byte readEEPROM(uint16_t address)
  {
    setAddress(address, true);
    setDataBusMode(INPUT);
    enableChip();
    delayMicroseconds(2);
    byte value = readDataBus();
    disableChip();
    return value;
  }

  // STAMPA RIGA CON ROM, OPCODE, VALORI (PER DEBUG)
  void printInstruction(uint8_t rom, uint16_t opcode, uint8_t step)
  {
    if (step == 0)
    {
      char buf[80];
      sprintf(buf, "%04x:  ", rom * 1000 + opcode * 16);
      Serial.print(buf);
    }
    if (step == 8)
    {
      char buf[2];
      sprintf(buf, " ");
      Serial.print(buf);
    }
    char buf[80];
    sprintf(buf, "%02x ", ((code[step]) >> (24 - 8 * rom)) & 0xFF);
    if (step == 15)
    {
      Serial.println(buf);
    }
    else
    {
      Serial.print(buf);
    }
  }

  // ************************************************************
  // *********************** PRINT STEP *************************
  // ************************************************************
  void printStep(uint8_t step, uint16_t address, uint8_t rom)
  {
    Serial.print("Step: ");
    Serial.print(step, HEX);
    Serial.print(" - Address: 0x");
    Serial.print(address, HEX);
    Serial.print(" - Value: ");
    Serial.println(((code[step]) >> (24 - 8 * rom)) & 0xFF, HEX);
  }

  // ************************************************************
  // ************* PRINT EEPROM CONTENTS (DEBUG) ****************
  // ************************************************************
  void printContents(uint16_t start, uint16_t lenght)
  {
    Serial.println("\n+++++++++++++++++++++++++++++");
    Serial.print("EEPROM Content $");
    Serial.print(start, HEX);
    Serial.print(" - $");
    Serial.println(start + lenght - 1, HEX);
    for (uint16_t baseAddress = start, end = start + lenght; baseAddress < end; baseAddress += 16)
    {
      byte data[16];
      for (uint16_t offset = 0; offset <= 15; offset += 1)
      {
        data[offset] = readEEPROM(baseAddress + offset);
      }
      char buf[80];
      sprintf(buf, "%04X:  %02x %02x %02x %02x %02x %02x %02x %02x   %02x %02x %02x %02x %02x %02x %02x %02x",
              baseAddress, data[0], data[1], data[2], data[3], data[4], data[5], data[6], data[7],
              data[8], data[9], data[10], data[11], data[12], data[13], data[14], data[15]);
      Serial.println(buf);
    }
  }

  // ************************************************************
  // ****************** PRINT OPCODES (DEBUG) *******************
  // ************************************************************
  void printOpcodeContents(uint8_t opcode)
  {
    // stampa il contenuto dell'array temporaneo creato in RAM per l'opcode
    // sono 4 ROM e 16 step
    Serial.println("_________");
    for (int step = 0; step < NUM_STEPS; step += 4)
    {
      // Serial.print(code[step], HEX);
      // Serial.print(":");
      // for (int shift = 24; shift >= 0; shift -= 8)
      // {
      Serial.print("Opcode 0x");
      Serial.print(opcode, HEX);
      Serial.print(" - Step: ");
      Serial.print(step, HEX);
      Serial.print("/");
      Serial.print(step + 3, HEX);
      Serial.print(":  ");
      Serial.print(((code[step]) >> 24) & 0xFF, HEX); // ROM 0
      Serial.print(":");
      Serial.print(((code[step]) >> 16) & 0xFF, HEX); // ROM 1
      Serial.print(":");
      Serial.print(((code[step]) >> 8) & 0xFF, HEX); // ROM 2
      Serial.print(":");
      Serial.print(((code[step]) >> 0) & 0xFF, HEX); // ROM 3
      Serial.print(" : ");
      //
      Serial.print(((code[step + 1]) >> 24) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 1]) >> 16) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 1]) >> 8) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 1]) >> 0) & 0xFF, HEX);
      Serial.print(" : ");
      //
      Serial.print(((code[step + 2]) >> 24) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 2]) >> 16) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 2]) >> 8) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 2]) >> 0) & 0xFF, HEX);
      //
      Serial.print(" : ");
      Serial.print(((code[step + 3]) >> 24) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 3]) >> 16) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 3]) >> 8) & 0xFF, HEX);
      Serial.print(":");
      Serial.print(((code[step + 3]) >> 0) & 0xFF, HEX);
      Serial.println("");
      // uint32_t data[4];
      // {
      //   data[col + 0] = (code[col + 0]);
      //   data[col + 1] = (code[col + 1]);
      //   data[col + 2] = (code[col + 2]);
      //   data[col + 3] = (code[col + 3]);
      // }
      // char buf[80];
      // sprintf(buf, "Opcode %02x:  %04lx %04lx %04lx %04lx",
      //         opcode, data[0], data[1], data[2], data[3]);
      // Serial.println(buf);
      // }
    }
  }



  // ********************************************
  // ********** TEST LETTURA DA ARRAY ***********
  // ********************************************
  // Questo non funziona, restituisce spazzatura
  /*     uint32_t val[16];
      for (int row = 0; row < 4; row += 1) {
        for (int col = 0; col < 16; col += 4) {
          val[col] = my_ram_microcode_template[row][col];
        }
        char buf[80];
        sprintf(buf, "%03x:  %02lx %02x %02x %02x %02x %02x %02x %02x   %02x %02x %02x %02x %02x %02x %02x %02x",
                row, val[0], val[1], val[2], val[3], val[4], val[5], val[6], val[7],
                val[8], val[9], val[10], val[11], val[12], val[13], val[14], val[15]);
        Serial.println(buf);
      } */
  /* void read_RAM()
  {
    // Qui voglio provare a leggere il contenuto della RAM copiata e stamparlo in sequenza, per vedere se quanto
    // ho copiato dalla Flash con initMicroCodeBlock sia o meno corretto
    uint32_t data;
    // per 256 elementi ognuno lungo 4 byte
    //
    for (int row = 0 + 16 * block; row < 4 + 16 * block; row += 1)
    {
      Serial.print(row, HEX);
      Serial.print(":  ");
      for (int col = 0; col < 16; col += 4)
      {
        data = my_ram_microcode_template[row][col];
        Serial.print(data, HEX);
        Serial.print(" / ");
        byte value1, value2, value3, value4;
        char buf[60];
        sprintf(buf, "%02lx:%02lx:%02lx:%02lx", (data & 0xFF000000) >> 24, (data & 0x00FF0000) >> 16, (data & 0x0000FF00) >> 8, (data & 0x000000FF));
        Serial.print(buf);
        value1 = data >> 24;
        value2 = (data & 0x00FF0000) >> 16;
        value3 = (data & 0x0000FF00) >> 8;
        value4 = (data & 0x000000FF);
        sprintf(buf, " - %02x:%02x:%02x:%02x", value1, value2, value3, value4);
        Serial.print(buf);
        Serial.print(" - Totale = ");
        Serial.println(value1 + value2 + value3 + value4);
        // sprintf(buf, "%03x:  %02x %02x %02x %02x", row, data >> 24, data & 0x00FF000 > 16, data & 0x0000FF000 > 8, data & 0x00000FF);
        // Serial.print("    ");
        //           for (int cnt = 0; cnt < 16; cnt += 1) {
        //           data = current_microcode_block[i][j];
        //           Serial.print(data >> 24, HEX);
        //           Serial.print(F(":"));
        //         }
      }
      Serial.println("");
    }
  } */

  /*
    // ********************************************
    // ************ EEPROM PROGRAM ****************
    // ********************************************
    void eeprom_program() {
    Serial.print("\nProgramming EEPROM ");
    // ogni block = 16 righe * 16 colonne * elemento uint32_t = 1024 Byte
    // ogni sezione sono 32 righe dunque due blocchi = 2K
    // 8 sezioni = 16 KB divisi in 4 parti, 0-FFF / 1000-1FFF / 2000-2FFF / 3000-3FFF
      for (int block = 0; block < 4; block += 1) {
        Serial.print("\nBlock = ");
        Serial.print(block);
        Serial.print(" - Starting element = ");
        Serial.print(block * 256);
        Serial.print(" - Ending element = ");
        Serial.print((block + 1) * 256);
        initMicroCodeBlock(block);
        for (int address = 256 * block; address < 256 * (block + 1); address += 1) {
          int byte_sel    = (address & 0b11000000000000) >> 12; // selezione della ROM
          int instruction = (address & 0b00111111110000) >> 4;
          int step        = (address & 0b00000000001111);
          if (address % 16 == 0) {
            Serial.print("\naddress / instruction = ");
            Serial.print(address, HEX);
            Serial.print(" / ");
            Serial.print(current_microcode_block[instruction][step] >> 24, HEX);
            Serial.print("|");
            Serial.print((current_microcode_block[instruction][step] & 0x00FF0000) >> 16, HEX);
            Serial.print("|");
            Serial.print((current_microcode_block[instruction][step] & 0x0000FF00) >>  8, HEX);
            Serial.print("|");
            Serial.print((current_microcode_block[instruction][step] & 0x000000FF),  HEX);
          }
          // switch (byte_sel) {
            // case 0:
              writeEEPROM(address, current_microcode_block[instruction][step] >> 24);
              // if (address % 64 == 0) {
              //   Serial.print(".");
              //   break;
              // }
            // case 1:
              writeEEPROM(address + 4096, current_microcode_block[instruction][step] >> 16);
              // if (address % 64 == 0) {
              //   Serial.print(":");
              //   break;
              // }
            // case 2:
              writeEEPROM(address + 4096 * 2, current_microcode_block[instruction][step] >> 8);
              //   if (address % 64 == 0) {
              //     Serial.print(",");
              //     break;
              // }
            // case 3:
              writeEEPROM(address + 4096 * 3, current_microcode_block[instruction][step]);
              // if (address % 64 == 0) {
              //   Serial.print(";");
              //   break;
              // }
          // if (address % 64 == 0) {
          //   Serial.print(".");
          // }
          // }
        }
      Serial.println("\nDone!");
      }
    }
   */

void unlock()
{
  delay(10);
  setDataBusMode(OUTPUT);
  enableChip();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0xAA >> 3); // data = BBBB.Bxxx ==> shift DX 3 pos. degli MSB BBBBB, che carico in PORTB0-PORTB4 del Nano, cioè D3-D7 del data bus
  PORTD = (PORTD & 0x1F) | (0xAA << 5); // data = xxxx.xDDD ==> shift SX 5 pos. degli LSB   DDD, che carico in PORTD5-PORTD7 del Nano, cioè D0-D2 del data bus
  strobeWE();
  setAddress(0x2AAA, false);
  PORTB = (PORTB & 0xE0) | (0x55 >> 3);
  PORTD = (PORTD & 0x1F) | (0x55 << 5);
  strobeWE();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0x80 >> 3);
  PORTD = (PORTD & 0x1F) | (0x80 << 5);
  strobeWE();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0xAA >> 3);
  PORTD = (PORTD & 0x1F) | (0xAA << 5);
  strobeWE();
  setAddress(0x2AAAA, false);
  PORTB = (PORTB & 0xE0) | (0x55 >> 3);
  PORTD = (PORTD & 0x1F) | (0x55 << 5);
  strobeWE();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0x20 >> 3);
  PORTD = (PORTD & 0x1F) | (0x20 << 5);
  strobeWE();
  disableChip();
  delay(10);
}

void lock()
{
  delay(10);
  setDataBusMode(OUTPUT);
  enableChip();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0xAA >> 3); // data = BBBB.Bxxx ==> shift DX 3 pos. degli MSB BBBBB, che carico in PORTB0-PORTB4 del Nano, cioè D3-D7 del data bus
  PORTD = (PORTD & 0x1F) | (0xAA << 5); // data = xxxx.xDDD ==> shift SX 5 pos. degli LSB   DDD, che carico in PORTD5-PORTD7 del Nano, cioè D0-D2 del data bus
  strobeWE();
  setAddress(0x2AAA, false);
  PORTB = (PORTB & 0xE0) | (0x55 >> 3);
  PORTD = (PORTD & 0x1F) | (0x55 << 5);
  strobeWE();
  setAddress(0x5555, false);
  PORTB = (PORTB & 0xE0) | (0xA0 >> 3);
  PORTD = (PORTD & 0x1F) | (0xA0 << 5);
  strobeWE();
  disableChip();
  delay(10);
}

// ************************************************************
// **************** EEPROM SMALL WRITE (DEBUG) ****************
// ************************************************************
void eepromSmallWrite(byte value)
{
  Serial.println("\n+++++++++++++++++++++++++++++");
  Serial.println("EEPROM Small Write");
  for (int address = 0; address <= 256; address += 1)
  {
    writeEEPROM(address, value);
    if (address % 16 == 0)
    {
      // Serial.print(".");
      Serial.print("Address ");
      Serial.println(address, HEX);
    }
  }
  Serial.println("Done!");
}

void strobeWE() // Setto low e poi high il bit che corrisponde a A0/D14
{
  PORTC = (PORTC & 0x3E);
  PORTC = (PORTC & 0x3E) | 0x01;
}

// Set the status of the device control pins
void enableOutput()   { PORTC = (PORTC & 0x3D);         }  // set OE LO (pin 1 PORTC / A1/D15) 
void disableOutput()  { PORTC = (PORTC & 0x3D) | 0x02;  }  // set OE HI (pin 1 PORTC / A1/D15)
void enableChip()     { PORTC = (PORTC & 0x3B);         }  // set CE LO (pin 2 PORTC / A2/D16)
void disableChip()    { PORTC = (PORTC & 0x3B) | 0x04;  }  // set CE HI (pin 2 PORTC / A2/D16)

void setDataBusMode(byte mode)
{
  if (mode == OUTPUT)
  { // Set data bus pins to OUTPUT (write to EEPROM)
  DDRB |= 0x1F; // set Nano D8-D12 to HI (output); these are PORTB 5x LSB bits (0001.1111), aka D3-D7 of Nano <==> EEPROM data bus
  DDRD |= 0xE0; // set Nano D5-D7  to HI (output); these are PORTD 3x MSB bits (1110.0000), aka D0-D2 of Nano <==> EEPROM data bus
  }
  else
  { // Set data bus pins to INPUT (read from EEPROM)
  DDRB &= 0xE0; // viceversa of above
  DDRD &= 0x1F; // viceversa of above
  }
}


// CALCOLO CRC16 POST-PROGRAMMAZIONE
/* Nella lettura di una EEPROM precedentemente programmata leggo tutti i byte in sequenza, dunque è sufficiente
leggere il contenuto da 0x0000 a 0x3FFF per calcolare il checksum. */
uint16_t calcCRC16_post(void)
{
  setDataBusMode(INPUT);
  crc = 0xFFFF;
  uint16_t polynomial = 0x1021;
  byte data;
  for (uint16_t address = 0x0000; address <= 0x3FFF; address++)
  {
    data = readEEPROM(address);
    crc = calculate_crc(data, crc, polynomial);
  }
  return crc;
}

// CRC16 MATH
uint16_t calculate_crc(uint8_t data, uint16_t crc, uint16_t polynomial)
{
  crc ^= data;
  for (uint8_t i = 0; i < 8; i++)
  {
    if (crc & 0x0001)
    {
      crc >>= 1;
      crc ^= polynomial;
    }
    else
    {
      crc >>= 1;
    }
  }
  return crc;
}

void loop()
{
  digitalWrite(LED_BUILTIN, HIGH); // turn the LED on (HIGH is the voltage level)
  delay(100);
  digitalWrite(LED_BUILTIN, LOW); // turn the LED off by making the voltage LOW
  delay(100);
  digitalWrite(LED_BUILTIN, HIGH);
  delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  delay(100);
  digitalWrite(LED_BUILTIN, HIGH);
  delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  delay(100);
  digitalWrite(LED_BUILTIN, HIGH);
  delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  delay(800);
}
~~~


## Link utili

- Il video <a href="https://www.youtube.com/watch?v=BA12Z7gQ4P0" target="_blank">Using an EEPROM to replace combinational logic</a> di Ben Eater, che descrive la programmazione manuale di una EEPROM.
- Il video <a href="https://www.youtube.com/watch?v=K88pgWhEb1M" target="_blank">Build an Arduino EEPROM programmer</a> e il repository GitHub <a href="https://github.com/beneater/eeprom-programmer" target="_blank">Arduino EEPROM programmer</a> di Ben Eater.
- Il programmatore di EEPROM <a href="https://github.com/TomNisbet/TommyPROM" target="_blank">TommyProm</a> e lo <a href="https://tomnisbet.github.io/nqsap/docs/microcode/" target="_blank">sketch semplificato</a> necessario alla sola programmazione delle EEPROM dell'NQSAP di Tom Nisbet, che ho studiato e implementato nel programmatore di EEPROM del BEAM.

## TO DO

- 28C series EEPROMS, like the X28C256, sometimes ship from the factory with Data Protection enabled. Use the UNLOCK command to disable this. See the 28C256 Notes for more information. https://tomnisbet.github.io/TommyPROM/docs/28C256-notes
- verificare l'ordine dei paragrafi ed eventualmene correggere i link a inizio pagina

Per approfondimenti sul microcode, si veda anche la pagina che descrive la [Control Logic](../control/) del BEAM.