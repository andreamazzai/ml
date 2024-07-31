---
title: "Aritmetica binaria"
permalink: /docs/math/
excerpt: "Aritmetica binaria"
---

## Concetti importanti

Per comprendere appieno il funzionamento dei moduli Flag e ALU, è stato necessario approfondire anche alcuni concetti dell'aritmetica binaria.

Cercando di assimilare il concetto di overflow, .......................

1. Numeri binari senza segno (Unsigned) e con segno (Signed)
2. Modalità di rappresentazione dei numeri binari con segno:
   - Modulo e Segno (Signed Magnitude)
   - Complemento a 1 (One’s Complement, o anche 1C)
   - Complemento a 2 (Two’s Complement, o anche 2C)
3. Somma e sottrazione di numeri binari
4. Overflow

## Fonti

Lungi dal voler (e poter) spiegare tutto, riporto alcuni concetti di base e suggerisco una serie di fonti.

Per una comprensione completa dei temi trattati, raccomando di accedere più volte alle informazioni, anche in tempi diversi e in sequenze diverse: con letture ed approfondimenti ripetuti avevo raggiunto lo scopo prefissato, mentre più visualizzazioni dei video in modalità *brainstorming* mi avevano permesso di riempire i buchi fino a colmarli.

Ecco una serie di video su YouTube:

1. **Binary Overflow** - Mr Powell's Computer Science Channel - [link](https://www.youtube.com/watch?v=Q7t9-Sq_4Ww&lc=Ugy04nCzuiaOJIqK_5J4AaABAg): concetto generico di Overflow.

    Ipotizziamo di avere un computer a 8 bit, in grado di sommare due numeri a 8 bit: sia gli addendi sia la somma sono numeri a 8 bit che possono dunque andare da 0 a 255.
      - Sommando ad esempio 73 + 114 si ottiene 187, che è un numero rappresentabile con 8 bit;
      - sommando invece ad esempio 175 + 92 si ottiene 267, che non è rappresentabile con un numero a 8 bit: abbiamo un errore di Overflow, perché i bit a nostra disposizione (8) non ci consentono di rappresentare il risultato della somma.

    Abbiamo in pratica un riporto che dovremmo portare al 9° bit del nostro computer, che però ne ha solo 8. In questo caso il segnale di Carry in uscita dall'ALU ci segnalerebbe che il numero risultante dalla somma dei due addendi è più grande del numero calcolabile dall'ALU.

2. **Somme di numeri Unsigned, il concetto di Carry ed eventuale Overflow** - Mr Dimmick's Computing Channel - [link](https://www.youtube.com/watch?v=nKxjLM6ePcI): molto ben fatto; spiega anche le regole di base applicabili a tutte le somme di numeri binari.

3. **Numeri negativi in binario** dello stesso autore del video 1 - [link](https://www.youtube.com/watch?v=dHB7jFjESLY): un ottimo video che spiega come rappresentare i numeri negativi in Complemento a 2 e come convertire un numero positivo in negativo e viceversa.

    - Con il Complemento a 2, vi è un solo 0, a differenza di quanto accade con Modulo e Segno (si veda più avanti in questa pagina).
    - Per convertire un numero positivo in negativo è sufficiente invertire tutti i bit ed aggiungere 1.
    - Lo stesso procedimento è perfettamente valido anche al contrario (da negativo a positivo).
    - Prestare attenzione a un errore riconosciuto anche dall'autore nei commenti: nella discussione di Segno e Modulo il range non va da -64 a +64, ma da -128 a +128.

4. **Binary Addition and Subtraction With Negative Numbers, 2's Complements & Signed Magnitude** - The Organic Chemistry Tutor - [link](https://www.youtube.com/watch?v=sJXTo3EZoxM): ottimo video con visualizzazione grafica molto efficace.

    - Particolarmente interessante un [commento](https://www.youtube.com/watch?v=sJXTo3EZoxM&lc=Ugyl7WIoyqVR8wM0dAJ4AaABAg) che segnala che per invertire il segno di un numero binario rappresentato in 2C è sufficiente seguire due passaggi:
      1) partire da destra e non modificare nulla fino al primo bit a 1 incluso;
      2) invertire tutti i bit rimanenti.

      Esempio: per invertire il numero 0b01010100 (84 decimale) si lasciano invariati i primi 3 bit partendo da destra (0b01010**100**) e si invertono poi tutti gli altri da 0b**01010**100 a 0b**10101**100, ottenendo come risultato finale 0b10101100 (-84).

E qualche pagina da visitare:

1. [The Overflow (V) Flag Explained](http://www.6502.org/tutorials/vflag.html) di Bruce Clark su [6502.org](http://6502.org) - linguaggio essenziale e tante informazioni.
2. [The 6502 overflow flag explained mathematically](https://www.righto.com/2012/12/the-6502-overflow-flag-explained.html) di Ken Shirriff - una pagina incredibilmente ben fatta che tratta Complementi, numeri Signed, Overflow.
3. [The 6502 CPU's overflow flag explained at the silicon level](https://www.righto.com/2013/01/a-small-part-of-6502-chip-explained.html) di Ken Shirriff - interessantissima analisi dell'Overflow al livello del silicio del 6502.
4. [Signed Binary Addition Calculator](https://madformath.com/calculators/digital-systems/signed-arithmetic/signed-binary-addition-calculator/signed-binary-addition-calculator) - calcolatore per somme di numeri Signed con evidenza dei vari passaggi di calcolo.

## Numeri Unsigned e numeri Signed

E' necessario comprendere a fondo la logica dei numeri Unsigned e Signed.

Approfondendo i numeri Signed, ero giusto in questa [interessantissima pagina](https://sandbox.mc.edu/~bennet/cs110/) dell'Università del Mississippi dedicata alla matematica binaria.

Grazie alla sezione Textbook ==> Module 3: Computer Integer Arithmetic ==> 2. Negative binary numbers avevo perfettamente compreso la rappresentazione binaria dei numeri negativi.

Il metodo **Signed Magnitude** è molto facile da comprendere: si sacrifica un bit dedicandolo alla rappresentazione del segno; il bit sacrificabile è quello più significativo (MSB), pertanto se un numero Unsigned a 8 bit può andare da 0 a 255 (2^8 = 256 combinazioni rappresentabili), un numero Unsigned avrà 7 bit disponibili per il Modulo (2^7 = 128 combinazioni) e un bit per il segno, dunque potrà andare da -128 a + 128 (sempre 256 numeri rappresentabili, ma metà negativi e metà positivi).

![Rappresentazione Modulo e Segno dei numeri a 4 bit](../../assets/math/75-math_signed_magnitude.gif){:width="100%"}

*Rappresentazione Modulo e Segno dei numeri a 4 bit.*

Nell'immagine (semplificata a soli 4 bit per ragioni di spazio) si nota un problema non secondario: lo zero appare due volte (0b0000 e 0b1000): noi sappiamo che lo zero non ha segno, pertanto questa rappresentazione non è la migliore possibile.

NB: nella pagina citata, vi è un esempio di sottrazione "5 - 2" errata. L'immagine di riferimento è la seguente:

![Errore sottrazione](../../assets/math/75-math_mistake.gif
){:width="10%"}

Quanto si desiderava in realtà rappresentare era una sottrazione utilizzando il metodo Modulo e Segno, che permette di trasformare l'operazione da "5 - 2" in "5 + (-2)" invertendo il primo bit del numero 2, che dunque da 0b0010 diventa 0b1010:

~~~text
   Dec        Bin 
C                 
A     5  ==>  0101+
B    -2  ==>  1010
   ----       ----
Q     3       1111 ==> -7
~~~

Come si vede, la somma binaria di 0101 (5) e 1010 (-2) non porta al risultato aspettato (3), bensì a un chiaro errore.

Tralasciando la spiegazione del metodo **Complemento a 1 (1C)**, anch'esso non ottimale, il **Complemento a 2 (2C)** risulta invece perfetto per la rappresentazione dei numeri negativi, portando tra l'altro in dote una grandissima semplificazione nell'esecuzione delle sottrazioni.

![Rappresentazione in Complemento a 2 dei numeri a 4 bit](../../assets/math/75-math_2c.gif){:width="100%"}

*Rappresentazione in Complemento a 2 dei numeri a 4 bit.*

Due sono gli aspetti da evidenziare:

- Risoluzione del problema del doppio zero.
- Le sottrazioni si possono eseguire senza errori sommando il Complemento a 2 del sottraendo. Ad esempio, invece di eseguire "15 - 7", si effettua "15 + (-7)": è piuttosto evidente che l'addizione è l'operazione più semplice in assoluto da implementare; utilizzando il Complemento a 2 è possibile applicare alle sottrazioni le stesse regole delle addizioni, semplificandone enormemente i calcoli.

La regola che sta alla base della teoria del Complemento a 2 è: come posso rappresentare il numero "-1" in modo che, aggiungendovi "1", si ottenga "0"?

Similarmente ai vecchi tachimetri delle automobili, che una volta giunti a 99.999 passavano a 00.000, il 99.999 del tachimetro corrisponde allo 0b1111.1111 dell'aritmetica binaria a 8 bit: sommato a 0b0000.0001, genera come risultato 0b0000.0000 (il punto separatore fra i primi 4 bit e i secondi 4 bit dei numeri a 8 bit è inserito solamente per agevolarne la visualizzazione e non ne modifica in alcun modo i valori).

Per i dovuti approfondimenti sulla sottrazione in 2C, si vedano i [link](#fonti) evidenziati in precedenza in questa pagina.

Riprendendo anche quanto esposto nella pagina dei [Flag](../flags/#overflow), in un byte sono possibili 256 combinazioni:

- trattando i numeri come Unsigned, è possibile contare da 0 a 255;
- trattando invece i numeri come Signed:
  - i valori da 0 a 127 sono rappresentati allo stesso modo dei numeri Unsigned da 0 a 127 (da Hex 0x00 in poi);
  - i valori da -128 a -1 fanno il paio con le rappresentazioni esadecimali e binarie dei numeri Unsigned da 128 a 255 (da Hex 0x80 a 0xFF).
  
Questa tabella dovrebbe chiarire il concetto:

~~~text
----------------------------------------
| Hex  | Binary    | Signed | Unsigned | 
----------------------------------------
| 0x80 | 1000.0000 |   -128 |      128 | 
| 0x81 | 1000.0001 |   -127 |      129 | 
| 0x82 | 1000.0010 |   -126 |      130 | 
| 0x83 | 1000.0011 |   -125 |      131 | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| 0x90 | 1001.0000 |   -112 |      144 | 
| 0xA0 | 1010.0000 |   - 96 |      160 | 
| 0xB0 | 1011.0000 |   - 80 |      176 | 
| 0xC0 | 1100.0000 |   - 64 |      192 | 
| 0xD0 | 1101.0000 |   - 48 |      208 | 
| 0xE0 | 1110.0000 |   - 32 |      224 | 
| 0xF0 | 1111.0000 |   - 16 |      240 | 
| 0xF1 | 1111.0001 |   - 15 |      241 | 
| 0xF2 | 1111.0010 |   - 14 |      242 | 
| 0xF3 | 1111.0011 |   - 13 |      243 | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| 0xFD | 1111.1101 |   -  3 |      253 | 
| 0xFE | 1111.1110 |   -  2 |      254 | 
| 0xFF | 1111.1111 |   -  1 |      255 | 
----------------------------------------
| Hex  | Binary    | Signed | Unsigned | 
----------------------------------------
| 0x00 | 0000.0000 |      0 |        0 | 
| 0x01 | 0000.0001 |      1 |        1 | 
| 0x02 | 0000.0010 |      2 |        2 | 
| 0x03 | 0000.0011 |      3 |        3 | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| 0x10 | 0001.0000 |     16 |       16 | 
| 0x20 | 0010.0000 |     32 |       32 | 
| 0x30 | 0011.0000 |     48 |       48 | 
| 0x40 | 0100.0000 |     64 |       64 | 
| 0x50 | 0101.0000 |     80 |       80 | 
| 0x60 | 0110.0000 |     96 |       96 | 
| 0x70 | 0111.0000 |    112 |      112 | 
| 0x71 | 0111.0001 |    113 |      113 | 
| 0x72 | 0111.0010 |    114 |      114 | 
| 0x73 | 0111.0011 |    115 |      115 | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| .... | ......... |    ... |      ... | 
| 0x7C | 0111.1100 |    124 |      124 | 
| 0x7D | 0111.1101 |    125 |      125 | 
| 0x7E | 0111.1110 |    126 |      126 | 
| 0x7F | 0111.1111 |    127 |      127 | 
~~~

*Relazione tra numeri Hex, Bin, Signed e Unsigned a 8 bit.*

Come già visto nell'immagine *Rappresentazione in Complemento a 2 dei numeri a 4 bit*, è importante notare anche qui il passaggio dei numeri Signed da -1 a 0 in corrispondenza del passaggio binario da 0b1111.1111 a 0b0000.0000.

Il Complemento a 2 è dunque un modo molto pratico per rappresentare i numeri Signed, nei quali un MSB = 0 indica un numero positivo e un MSB = 1 indica un numero negativo.

## Approfondimento Overflow

L'approfondimento dell'Overflow è stato un passaggio obbligato; prima di capire il funzionamento della parte di circuito dedicata al [flag V](../flags/#overflow), dovevo comprenderne bene la definizione e quali metodi si usassero per determinarne una eventuale presenza.

Tom Nisbet segnalava che l'ispirazione per l'uso di un Data Selector/Multiplexer [74LS151](https://www.ti.com/lit/ds/symlink/sn54s151.pdf) per individuare situazioni di Overflow derivava da [un thread su Reddit](https://www.reddit.com/r/beneater/comments/kmuuex/question_for_all_74ls181_alu_people/), che rimandava poi a pagine scritte da Dieter Mueller su 6502.org.

Nel [link di riferimento](http://6502.org/users/dieter/v_flag/v_0.htm), Dieter inizialmente spiegava in maniera visiva in quali situazioni di somma o sottrazione tra due numeri Signed a 8 bit si generasse un Overflow, utilizzando quella che si potrebbe forse definire una sorta di tavola pitagorica.

Ricordiamo che l'Overflow indica un errore nel processo di somma o sottrazione di due numeri Signed: se il numero risultante ha un segno errato rispetto al previsto, si ha un Overflow.

Nelle due rappresentazioni sottostanti, le combinazioni Colonna/Riga al cui incrocio vi è un asterisco "\*" indicano situazioni di Overflow, cioè di un risultato che non può essere correttamente rappresentato con gli 8 bit a disposizione.

~~~text
Operazione       Sum Colonna + Riga       Sub Colonna - Riga
                 + 0123456789ABCDEF       + 0123456789ABCDEF
                 + 0000000000000000       - 0000000000000000
Overflow:        00................       00................
Overflow:        10.......*........       10........*.......
Overflow:        20......**........       20........**......
Overflow:        30.....***........       30........***.....
Overflow:        40....****........       40........****....
Overflow:        50...*****........       50........*****...
Overflow:        60..******........       60........******..
Overflow:        70.*******........       70........*******.
Overflow:        80........********       80********........
Overflow:        90........*******.       90.*******........
Overflow:        A0........******..       A0..******........
Overflow:        B0........*****...       B0...*****........
Overflow:        C0........****....       C0....****........
Overflow:        D0........***.....       D0.....***........
Overflow:        E0........**......       E0......**........
Overflow:        F0........*.......       F0.......*........

Esempio:         0x70 + 0x40 = Overflow   0xA0 - 0x30 = Overflow
~~~

Note:

- Le tabelle sono semplificate: sono mostrati solo i numeri multipli di 16 (0x**X**0), dunque 0x00, 0x10, 0x20 e così via.
- Sono rappresentati numeri Signed a 8 bit, dunque, ad esempio, 0x20 corrisponde a 32 decimale, mentre 0xA0 corrisponde a -96 decimale.
- Come esposto nella tabella *Relazione tra numeri Hex, Bin, Signed e Unsigned a 8 bit*, i numeri Signed a 8 bit vanno da -128 (0x80) a 127 (0x7F) passando per lo zero (0x00).

Riprendiamo i due esempi di Overflow riportati in calce alle tabelle:

- **0x70 + 0x40 =** 112 + 64 = 176, che però non rientra nel range -128 / + 127 dei numeri Signed a 8 bit. In effetti, la somma tra 112 e 64 genera un risultato il cui MSB è 1, che secondo la notazione Signed è un numero negativo: poiché la somma di due Signed positivi non può avere come risultato un Signed negativo, siamo in una situazione di Overflow.

~~~text
  Hex       Dec       Bin
                      1
  0x70 ==>  112  ==>  0111.0000 +
 +0x40 ==>   64  ==>  0100.0000 = 
           ----     ----------- 
            176       1011.0000 ==> 0xB0
~~~

- **0xA0 - 0x30 =** -96 - 48 = -144, che però non rientra nel range -128 / + 127 dei numeri Signed a 8 bit. In effetti, la sottrazione tra -96 e 48\* genera un risultato il cui MSB è 0, che secondo la notazione Signed è un numero positivo: poiché la sottrazione tra un Signed negativo e un Signed positivo non può risultare in un Signed positivo, siamo in una situazione di Overflow.

~~~text
  Hex       Dec       Bin
                     1
  0xA0 ==>  -96  ==>  1010.0000 +
 -0x30 ==>  -48  ==>  1101.0000 = 
           ----     ----------- 
           -144      10111.0000** ==> 0111.0000 ==> 0x70
~~~

\* Notare che in questo secondo caso l'ALU esegue internamente una operazione A + (-B), dunque la rappresentazione di B è  invertita secondo la regola del Complemento a 2 (2C).

\*\* Il 9° bit viene troncato, perché il bus dati di questo computer è a 8 bit.

In definitiva, possiamo dire che se il bit del segno viene corrotto, siamo un una situazione di Overflow e il flag V viene conseguentemente settato.

### L'Overflow e l'hardware

Dieter esponeva una rappresentazione logica di un Adder in grado di effettuare sia somme sia sottrazioni, segnalando che "le somme A+B sono facili; per eseguire invece le sottrazioni, il metodo più semplice è quello di invertire B e procedere poi esattamente come per le somme: A+(-B)".

![Adder hardware per somme A+B e sottrazioni A-B](../../assets/math/75-dieter-alu.png){:width="66%"}

*Adder hardware per somme A+B e sottrazioni A-B.*

In un adder a 8 bit abbiamo 8 adder a 1 bit in cascata; quello rappresentato di seguito è l'8° ed ultimo adder, i cui ingressi A e B' sono i bit più significativi A7 e B7'; sul Carry-in C_IN viene portato il Carry-out C_OUT proveniente dall'adder precedente, che è il 7°.

![Ultimo stadio di un adder a 8 bit](../../assets/math/75-dieter-8th-adder.png){:width="25%"}

*Ultimo stadio di un adder a 8 bit.*

Nella tabella seguente si rappresenta quanto accade nell'8° adder:

- C7 è il bit di Carry proveniente dal 7° adder (C_IN)
- B7' è l'MSB del numero da sommare B (normale o invertito dalla circuiteria interna, a seconda che si esegua una somma o una sottrazione)
- A7 è l'MSB del numero da sommare A
- Q7 è l'MSB del risultato di A7 + B7' + C7
- C8 è il Carry in uscita dall'8° adder (C_OUT)
- C8 risulta a 1 se almeno due tra A7, B7' e C7 sono a 1.
- V è il Flag di Overflow

Creando la truth table per la *somma* di A7, B7' e C7, ricaviamo:

| C7  | B7' | A7  | Q7  | C8  |  V   |
| -   | -   | -   | -   | -   | -    |
|  0  |  0  |  0  |  0  |  0  |  0   |
|  0  |  0  |  1  |  1  |  0  |  0   |
|  0  |  1  |  0  |  1  |  0  |  0   |
|  **0**  |  **1**  |  **1**  |  **0**  |  **1**  |  **1**\*  |
|  **1**  |  **0**  |  **0**  |  **1**  |  **0**  |  **1**\*\*|
|  1  |  0  |  1  |  0  |  1  |  0   |
|  1  |  1  |  0  |  0  |  1  |  0   |
|  1  |  1  |  1  |  1  |  1  |  0   |

Abbiamo detto poco sopra che la somma di due Signed negativi non può avere come risultato un Signed positivo - ed è dimostrabile che vale anche il contrario. Nelle due righe evidenziate troviamo infatti:

- \* due Signed negativi (A7 = B7' = 1) la cui somma genera un risultato positivo (Q7 = 0) --> abbiamo una situazione di Overflow.
- \*\* due Signed positivi (A7 = B7' = 0) la cui somma genera un risultato negativo (Q7 = 1) --> abbiamo una situazione di Overflow.

Mettendo a fattor comune quanto abbiamo visto fino ad ora, siamo in grado di sviluppare 3 diversi metodi per identificare una situazione di Overflow in una somma:

1. A7 e B7' sono dello stesso segno e Q è invertito rispetto ad A e B, cioè **(A7 == B7') AND (Q7 <> A7)** (notare che non stiamo specificando un valore assoluto 1 o 0 dei bit: stiamo considerando le relazioni tra i segnali);
2. C7 e C8 sono invertiti tra loro, cioè **C7 <> C8** (anche in questo caso si esegue una comparazione relativa);
3. **(A7 = B7' = 1 AND Q7 = 0) OR (A7 = B7' = 0 AND Q7 = 1)** è simile al punto 1, ma, anziché porli in una logica di comparazione relativa, stiamo specificando il valore assoluto dei bit.

La truth table **(A == B) AND (Q <> A)** del primo caso si tradurrebbe nella logica in figura; purtroppo, il computer basato sulle ALU 74LS181 non offre visibilità del valore di B7', che è computato internamente all'ALU e non esposto, pertanto non la possiamo utilizzare:

![Primo metodo](../../assets/math/75-overflow-detector-xor-and.png)

Avevo provato a disegnare lo schema di un adder per comprendere meglio il concetto; si noti che in questo schema il segnale B7 in uscita dal registro B è *visibile* e può essere portato alla circuiteria atta a individuare situazioni di Overflow:

[![Adder su carta](../../assets/math/75-adder.png "Adder su carta"){:width="50%"}](../../assets/math/75-adder.png){:width="75%"}

---
Stesso ragionamento per il secondo caso **C7 <> C8**: non il '181 non offre visibilità di C7, che è computato internamente all'ALU e non esposto:

![Secondo metodo](../../assets/math/75-overflow-detector-xor.png)

---
Nemmeno il terzo metodo **(A7 = B7' = 1 AND Q7 = 0) OR (A7 = B7' = 0 AND Q7 = 1)** sembra utilizzabile, perché non vi è visibilità esterna di B7':

![Terzo metodo](../../assets/math/75-overflow-detector-and-or.png)

E' possibile però ricostruire artificialmente il segnale B7' basandosi sugli altri segnali disponibili nel computer. Infatti:

- l'ultimo metodo diventa utilizzabile per la verifica dell'Overflow per le *addizioni*; il valore B7' che l'ultimo adder del '181 troverà in ingresso sarà infatti uguale al valore di B7 messo in input (in una somma A7 + B7, B7' non subisce modifiche dalla circuiteria dell'ALU e possiamo dunque usare B7 come input del circuito che determina l'eventuale stato di Overflow):

![Overflow somma](../../assets/math/75-overflow-detector-a+b.png)

- qualche modifica permette di riutilizzare lo stesso metodo anche per la verifica dell'Overflow nelle *sottrazioni*; il valore B7' che l'ultimo adder del '181 troverà in ingresso sarà infatti invertito rispetto al valore di B7 messo in input (in una sottrazione A7 - B7, B7' viene invertito dalla circuiteria dell'ALU* e possiamo dunque usare l'inverso di B7 come input del circuito che determina l'eventuale stato di Overflow):

![Overflow sottrazione](../../assets/math/75-overflow-detector-a-b.png)

\* La sottrazione viene effettuata sommando il Complemento a 2 del sottraendo; avendo a disposizione il valore logico di B7, è facile modificare il circuito per tenerne in considerazione l'inverso e individuare l'Overflow anche nell'operazione di sottrazione.

Giunti a questo punto, per realizzare un circuito in grado di identificare l'Overflow avremmo bisogno di 4 porte AND con 3 ingressi e 3 porte OR con 2 ingressi (la terza OR servirebbe ad eseguire l'OR logico tra i due circuiti precedenti per creare un'unica segnalazione di Overflow tanto in caso di addizione quanto di sottrazione).

Al posto di 4x AND e 3x OR, un unico 74LS151 può fare al caso nostro: una configurazione dei pin di ingresso come evidenziato in figura risolve le equazioni di Overflow sia per le addizioni A + B, sia per le sottrazioni A - B e B - A; si noti però che l'operazione B - A non è necessaria nella realizzazione del computer, pertanto non la terremo in considerazione:

![74ls151](../../assets/math/75-overflow-74151.png)

Le combinazioni degli ingressi A, B e C del '151 (connessi agli MSB A7, B7 e Q7 dei registri A e B e dell'uscita Q dell'ALU) selezionano quale tra gli ingressi I0-I7 sia necessario attivare per portare in uscita una eventuale segnalazione di Overflow.

Alcuni degli ingressi I0-I7 sono opportunamente connessi in modalità "Hardwired" all'Instruction Register per determinare l'operazione correntemente in esecuzione.

Testiamo alcuni casi, ma non prima di aver fatto un esempio iniziale spiegando anche cosa significano le varie colonne:

- Nella colonna **Hex** è esposta la rappresentazione esadecimale dei numeri che vogliamo sommare o sottrarre, con il simbolo dell'operazione alla sinistra del secondo numero; desideriamo eseguire l'operazione 0x70 - 0x30.
- La colonna **Dec** mostra il valore decimale ricavato dalla tabella *Relazione tra numeri Hex, Bin, Signed e Unsigned a 8 bit*; 0x70 corrisponde a 112 decimale, mentre 0x30 corrisponde a 48: l'operazione è quindi 112 - 48 = 64.
- La colonna **Bin** espone la rappresentazione binaria dei numeri (in Complemento a 2 se negativi): 112 corrisponde a 0111.0000, mentre 48 corrisponde a 0011.0000.
- La colonna **2C** è utilizzata per eseguire l'operazione di somma invertendo l'eventuale sottraendo positivo col metodo Complemento a 2.

Come detto poco fa, la sottrazione di un numero positivo (nel nostro caso 48) viene infatti eseguita sommando il valore negativo di quel numero, cioè -48: la somma del Complemento a 2 di un numero trasforma la sottrazione in addizione, pertanto l'operazione diventa 112 + (-48), cioè 0111.0000 + 1101.0000 nella colonna 2C:

~~~text
    Hex        Dec        Bin             2C
C                                        11111
A   0x70  ==>   112  ==>  0111.0000  ==>  0111.0000 +
B  -0x30  ==>   -48  ==>  0011.0000  ==>  1101.0000 = 
               ----                      ----------
Q                64                      10100.0000 ==> 0100.0000 ==> 0x40 = 64, no Overflow
~~~

Rafforzando quanto visto fino ad ora: quando devo effettuare la sottrazione di un numero positivo, ne calcolo il Complemento a 2 e lo sommo al minuendo.

- **Caso 1:** 0x20 + 0xC0; somma A + B di un Signed positivo e un Signed negativo

~~~text
    Hex        Dec        Bin             2C
C                                                     
A   0x20  ==>    32  ==>  0010.0000  ==>  0010.0000 +
B  +0xC0  ==>   -64  ==>  1100.0000  ==>  1100.0000 = 
               ----                      ----------
Q               -32                       1110.0000 ==> 0xE0 = -32, no Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A + B porterebbe a 1 gli ingressi I3 e I4 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=1, B7=1 e A7=0 sugli ingressi di selezione (CBA = 110), che attiverebbero l'ingresso I6 che risulta a 0 in quanto non attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 0, indicando che non vi è Overflow --> situazione verificata correttamente.

- **Caso 2:** 0x20 + 0x70; somma A + B di due Signed positivi

~~~text
    Hex        Dec        Bin             2C
C                                         11
A   0x20  ==>    32  ==>  0010.0000  ==>  0010.0000 +
B  +0x70  ==>   112  ==>  0111.0000  ==>  0111.0000 = 
               ----                      ----------
Q               144                       1001.0000 ==> 0x90 = -112, Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A + B porterebbe a 1 gli ingressi I3 e I4 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=1, B7=0 e A7=0 sugli ingressi di selezione (CBA = 100), che attiverebbero l'ingresso I4 che risulta a 1 in quanto attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 1, indicando che vi è Overflow --> situazione verificata correttamente.

- **Caso 3:** 0x50 - 0x30; sottrazione A - B tra due Signed positivi

~~~text
    Hex        Dec        Bin             2C
C                                        11 1
A   0x50  ==>    80  ==>  0101.0000  ==>  0101.0000 +
B  -0x30  ==>   -48  ==>  0011.0000  ==>  1101.0000 = 
               ----                      ----------
Q                32                      10010.0000 ==> 0010.0000 ==> 0x20 = 32, no Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A - B porterebbe a 1 gli ingressi I1 e I6 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=0, B7=0 e A7=0 sugli ingressi di selezione (CBA = 000), che attiverebbero l'ingresso I0 che risulta a 0, pertanto l'uscita Q del '151 sarebbe a 0, indicando che non vi è Overflow --> situazione verificata correttamente.

- **Caso 4:** 0x90 - 0x30; sottrazione A - B tra un Signed negativo e un Signed positivo

~~~text
    Hex        Dec        Bin             2C
C                                        1  1
A   0x90  ==>  -112  ==>  1001.0000  ==>  1001.0000 +
B  -0x30  ==>   -48  ==>  0011.0000  ==>  1101.0000 = 
               ----                      ---------- 
Q              -160                      10110.0000 ==> 0110.0000 ==> 0x60 = 96, Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A - B porterebbe a 1 gli ingressi I1 e I6 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=0, B7=0 e A7=1 sugli ingressi di selezione (CBA = 001), che attiverebbero l'ingresso I1 che risulta a 1 in quanto attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 1, indicando che vi è Overflow --> situazione verificata correttamente.

In realtà il modulo ALU del computer NQSAP - e di conseguenza, del BEAM - utilizza solo le istruzioni A + B e A - B, dunque possiamo semplificare le connessioni del '151:

![74ls151](../../assets/math/75-overflow-74151-2.png)

I segnali S0 ed S1 dovranno avere una corrispondenza hardwired con l'Instruction Register per identificare rispettivamente le istruzioni di somma e di sottrazione. Riprendiamo lo schema della sezione [Overflow](../flags/#overflow) della pagina del modulo Flag:

Possiamo finalmente analizzare il circuito dell'Overflow dell'NQSAP (e del BEAM) e visualizzare la truth table completa:

![Utilizzo di un 74LS151 per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.](../../assets/flags/30-flag-v-151.png){:width="50%"}

*Utilizzo di un 74LS151 per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.*

Alla luce di tutte le considerazioni fatte, questa tabella può assumere ora un significato più semplice rispetto a quanto non si potesse inizialmente pensare:

| Q7-S2 | B7-S1 | A7-S0 | Sum/Sub | Input  | Overflow     |
| -     | -     | -     |  -      | -      |  -     |
|   0   |   0   |   0   |         | I0     |  -     |
| **0** | **0** | **1** | **A-B** | **I1** |  **1** |
|   0   |   1   |   0   |         | I2     |  -     |
| **0** | **1** | **1** | **A+B** | **I3** |  **1** |
| **1** | **0** | **0** | **A+B** | **I4** |  **1** |
|   1   |   0   |   1   |         | I5     |  -     |
| **1** | **1** | **0** | **A-B** | **I6** |  **1** |
|   1   |   1   |   1   |         | I7     |  -     |

- Il flag Overflow si attiva se **(A7 = B7 = 1 AND Q7 = 0) OR (A7 = B7 = 0 AND Q7 = 1)** *E* stiamo eseguendo una addizione (IR-Q3 attivo).
- Il flag Overflow si attiva se **(A = 1 AND B = 0 AND Q = 0) OR (A = 0 AND B = 1 AND Q = 1)** *E* stiamo eseguendo una sottrazione (IR-Q1 attivo).

Per identificare l'esecuzione di un'operazione di addizione o di sottrazione e dunque selezionare quale debba essere l'ingresso corretto del '151 da attivare, si utilizzano due delle linee di selezione dell'operazione dell'ALU, in particolar modo:

| IR-Q1 | IR-Q3 | Operazione  |
| -     | -     | -           |
| HI    | LO    | Sottrazione |
| LO    | HI    | Addizione   |

Per finire, da quanto visto fino ad ora possiamo dedurre un'altra regola: la somma di due Signed di segno opposto e **la sottrazione di due Signed dello stesso segno** non possono causare Overflow.
