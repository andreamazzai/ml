---
title: "Aritmetica binaria"
permalink: /docs/math/
excerpt: "Aritmetica binaria"
---

## Concetti importanti

Per comprendere appieno il funzionamento dei moduli Flag e ALU, è stato necessario approfondire anche alcuni concetti dell'aritmetica binaria.

Cercando di assimilare il concetto di overflow, 

I concetti utili da comprendere sono:

1. Numeri binari senza segno (Unsigned) e con segno (Signed)
2. Modalità di rappresentazione dei numeri binari con segno:
   - Modulo e Segno (Signed Magnitude)
   - Complemento a 1 (One’s Complement, o anche 1C)
   - Complemento a 2 (Two’s Complement, o anche 2C)
3. Somma e sottrazione di numeri binari

## Fonti

Lungi dal voler (e poter) spiegare tutto, riporto alcuni concetti di base e suggerisco una serie di fonti che mi hanno permesso di arrivare a comprendere appieno i temi esposti.

Per una compresione completa dei temi trattati, raccomando di accedere più volte alle fonti, anche in tempi diversi e in sequenze diverse: con letture ed approfondimenti ripetuti avevo raggiunto lo scopo prefissato, mentre più visualizzazioni dei video in modalità *brainstorming* mi avevano permesso di riempire i buchi fino a colmarli.

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
    - Prestare attenzione a un errore, riconosciuto dall'autore nei commenti: nella discussione di Segno e Modulo il range non va da -64 a +64, ma da -128 a +128.

4. **Binary Addition and Subtraction With Negative Numbers, 2's Complements & Signed Magnitude** - The Organic Chemistry Tutor - [link](https://www.youtube.com/watch?v=sJXTo3EZoxM): ottimo video con visualizzazione grafica molto efficace.

    - Particolarmente interessante un [commento](https://www.youtube.com/watch?v=sJXTo3EZoxM&lc=Ugyl7WIoyqVR8wM0dAJ4AaABAg) che segnala che per invertire il segno di un numero binario rappresentato in 2C è sufficiente seguire due passaggi:
      1) partire da destra e non modificare nulla fino al primo bit a 1 incluso;
      2) invertire tutti i bit rimanenti.

      Esempio: per invertire il numero 01010100 (84 decimale) si lasciano invariati i primi 3 bit partendo da destra (01010**100**) e si invertono poi tutti gli altri da **01010**100 a **10101**100, ottenendo come risultato finale 10101100 (-84).

## Numeri Unsigned e numeri Signed

E' necessario comprende a fondo la logica dei numeri Unsigned e Signed.

Non ricordo più quale fonte (credevo un commento di un video di Mr Powell's Computer Science Channel, ma non riesco più a trovarlo) mi avesse portato in questa [interessantissima pagina](https://sandbox.mc.edu/~bennet/cs110/) dell'Università del Mississippi dedicata alla matematica binaria.

Grazie alla sezione Textbook ==> Module 3: Computer Integer Arithmetic ==> 2. Negative binary numbers avevo perfettamente compreso la rappresentazione binaria dei numeri negativi.

Il metodo **Signed Magnitude** è molto facile da comprendere: si sacrifica un bit dedicandolo alla rappresentazione del segno; il bit sacrificabile è quello più significativo (MSB), pertanto se un numero Unsigned a 8 bit può andare da 0 a 255 (2^8 = 256 combinazioni rappresentabili), un numero Unsigned avrà 7 bit disponibili per il Modulo (2^7 = 128 combinazioni) e un bit per il segno, dunque potrà andare da -128 a + 128 (sempre 256 numeri rappresentabili, ma metà negativi e metà positivi).

![Rappresentazione Modulo e Segno dei numeri a 4 bit](../../assets/math/75-math_signed_magnitude.gif){:width="100%"}

*Rappresentazione Modulo e Segno dei numeri a 4 bit.*

Nell'immagine (semplificata a soli 4 bit per ragioni di spazio) si nota un problema non secondario: lo zero appare due volte (0000 e 1000): noi sappiamo che lo zero non ha segno, pertanto questa rappresentazione non è la migliore possibile.

NB: nella pagina citata, vi è un esempio di sottrazione "5 - 2" che non comprendo e suppongo sia errata. L'immagine di riferimento è la seguente:

![Possibile errore nella pagina?](../../assets/math/75-math_mistake.gif
){:width="10%"}

Tralasciando la spiegazione del metodo **Complemento a 1 (1C)**, anch'esso non ottimale, il **Complemento a 2 (2C)** risulta essere invece perfetto per la rappresentazione dei numeri negativi, portando tra l'altro in dote una grandissima semplificazione nell'esecuzione delle sottrazioni:

![Rappresentazione in Complemento a 2 dei numeri a 4 bit](../../assets/math/75-math_2c.gif){:width="100%"}

*Rappresentazione in Complemento a 2 dei numeri a 4 bit.*

Due sono gli aspetti da evidenziare:

- Risoluzione del problema del doppio zero.
- Le sottrazioni si realizzano sommando il Complemento a 2 del numero da sottrarre. Ad esempio, invece di eseguire "15 - 7" si effettua "15 + (-7)": l'addizione è l'operazione più semplice in assoluto da implementare e lo stesso circuito utilizzato per le addizioni può essere utilizzato anche per le sottrazioni.

La regola che sta alla base della teoria del Complemento a 2 è: come posso rappresentare il numero "-1" in modo che, aggiungendovi "1", si ottenga "0"?

Similarmente ai vecchi tachimetri delle automobili, che una volta giunti a 99.999 passavano a 0, il 99.999 del tachimetro corrisponde all'11111111 dell'aritmetica binaria a 8 bit, che sommato a 00000001 genera come risultato 00000000.

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

Come già visto nell'immagine *Rappresentazione in Complemento a 2 dei numeri a 4 bit*, è importante notare anche qui il passaggio dei numeri Signed da -1 a 0 in corrispondenza del passaggio binario da 11111111 a 00000000.

Il Complemento a 2 è dunque un modo molto pratico per rappresentare i numeri Signed, nei quali un MSB = LO indica un numero positivo e un MSB = HI indica un numero negativo.

## Approfondimento Overflow

L'approfondimento dell'Overflow è stato un passaggio obbligato; prima di capire il funzionamento della parte di circuito dedicata al [flag V](../flags/#overflow), dovevo comprenderne bene la definizione e quali metodi si usassero per determinarne una eventuale presenza.

Tom Nisbet segnalava che l'ispirazione per l'uso di un Data Selector/Multiplexer [74LS151](https://www.ti.com/lit/ds/symlink/sn54s151.pdf) per individuare situazioni di Overflow derivava da [questo thread su Reddit](https://www.reddit.com/r/beneater/comments/kmuuex/question_for_all_74ls181_alu_people/), che rimanda poi in effetti a pagine di Dieter Mueller su 6502.org.

Nel [link di riferimento](http://6502.org/users/dieter/v_flag/v_0.htm) l'autore Dieter inizialmente spiegava in maniera visiva in quali situazioni di somma o sottrazione tra due numeri Signed a 8 bit si generasse un Overflow, utilizzando quella che si potrebbe forse definire una sorta di tavola pitagorica.

Ricordiamo che l'Overflow indica un errore nel processo di somma o sottrazione di due numeri Signed: se il numero risultante ha un segno errato rispetto al previsto, si ha un Overflow.

Nelle due rappresentazioni sottostanti, le combinazioni Colonna/Riga al cui incrocio vi è un asterisco "\*" indicano situazioni di Overflow, cioè di un risultato che non può essere correttamente rappresentato con gli 8 bit a disposizione.

~~~text
Operazione       Sum Colonna + Riga       Sub Colonna - Riga
Flag:            Carry: no                Carry: sì
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
- Sono rappresentati numeri Signed a 8 bit, dunque, ad esempio, 0x20 rappresenta 32 decimale, mentre 0xA0 rappresenta -96 decimale.
- Per l'operazione di addizione **non c'è** un Carry in ingresso; per la sottrazione **c'è** un Carry in ingresso: come spiegato nella sezione del modulo ALU [Carry, addizioni e sottrazioni](../alu/#carry-addizioni-e-sottrazioni), il Carry **non viene** settato prima di eseguire una somma, mentre **viene** settato prima di eseguire una sottrazione.
- Come esposto nella tabella *Relazione tra numeri Hex, Bin, Signed e Unsigned a 8 bit*, i numeri Signed vanno da -128 (0x80) a 127 (0x7F) passando per lo zero (0x00).

Riprendiamo i due esempi riportati in calce alle tabelle:

- **0x70 + 0x40 = 0xB0**, cioè 112 + 64 = 176, che però non rientra nel range -128 / + 127 dei numeri Signed a 8 bit: il MSB di 0xB0 è 1, che secondo la notazione Signed significa che si tratta di un numero negativo, ma la somma di due Signed positivi non può avere come risultato un Signed negativo --> abbiamo una situazione di Overflow.

~~~text
  Hex       Dec       1
  0x70 ==>  112  ==>  0111.0000 +
 +0x40 ==>   64  ==>  0100.0000 = 
           ----     ----------- 
            176       1011.0000 ==> 176 / 0xB0
~~~

- **0xA0 - 0x30 = 0x70**, cioè -96 - 48 = -144, che però non rientra nel range -128 / + 127 dei numeri Signed a 8 bit. In effetti, la sottrazione tra -96 e -48 viene in realtà eseguita sommando i due numeri espressi in Complemento a 2: -96 -48 = -96 +(-48)

~~~text
  Hex       Dec      1
  0xA0 ==>  -96  ==>  1010.0000 +
 -0x30 ==>  -48  ==>  1101.0000 = 
           ----     ----------- 
           -144      10111.0000 ==> 0111.0000 ==> 112 / 0x70
~~~

Poiché stiamo parlando di numeri a 8 bit e la somma di -96 e - 48 sfocia in un 9° bit che viene troncato, il risultato finale del calcolo è 112 o 0x70 esadecimale, il cui MSB è 0: secondo la notazione Signed significa che si tratta di un numero positivo, ma la somma di due Signed negativi non può avere come risultato un Signed positivo --> abbiamo una situazione di Overflow.

In definitiva, possiamo dire che se il bit del segno viene corrotto, siamo un una situazione di Overflow e il flag V viene conseguentemente settato.

![Adder hardware per somme A+B e sottrazioni A-B](../../assets/math/75-dieter-alu.png){:width="66%"}

*Adder hardware per somme A+B e sottrazioni A-B.*

Dieter esponeva una rappresentazione logica di un Adder in grado di effettuare sia somme sia sottrazioni, segnalando che "le somme A+B sono facili; per eseguire le sottrazioni il metodo più semplice è quello di invertire B e procedere poi esattamente come per le somme: A+(-B)".

![Ultimo stadio di un adder a 8 bit](../../assets/math/75-dieter-8th-adder.png){:width="25%"}

*Ultimo stadio di un adder a 8 bit.*

In un adder a 8 bit abbiamo 8 adder a 1 bit in cascata; quello rappresentato sopra è l'8° ed ultimo adder, i cui ingressi A e B' sono i bit più significativi A7 e B7'; sul Carry-in C_IN viene portato il Carry-out C_OUT proveniente dall'adder precedente, che è il 7°.

Nella tabella successiva:

- C7 è il bit di Carry proveniente dal 7° adder (l'adder che esegue la somma A6 + B6')
- B7' è l'MSB del numero da sommare B (normale o invertito dalla circuiteria interna, a seconda che si esegua una somma o una sottrazione)
- A7 è l'MSB del numero da sommare A
- Q7 è l'MSB del risultato di A7 + B7'+ C7
- C8 è il Carry in uscita dall'8° adder (l'adder che esegue la somma A7 + B7')
- C8 è settato 1 se almeno due tra A7, B7' e C7 sono a 1.
- V è il Flag di Overflow

Andando a mettere in tabella le varie combinazioni di A, B e C, ricaviamo:

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

Abbiamo detto che la somma di due Signed negativi non può avere come risultato un Signed positivo - e viceversa. Nelle due righe evidenziate troviamo invece rispettivamente:

- \* due Signed negativi (A7 = B7' = 1) che generano un risultato positivo (Q7 = 0) --> sappiamo che è una situazione di Overflow.
- \*\* due Signed positivi (A7 = B7' = 0) che generano un risultato negativo (Q7 = 1) --> sappiamo che è una situazione di Overflow.

Mettendo a fattor comune quanto abbiamo visto fino ad ora, siamo in grado di stabilire gli stati che determinano una situazione di Overflow. Possiamo trovare tre diversi casi:

1. A7 e B7' sono dello stesso segno e Q è invertito rispetto ad A e B, cioè **(A == B) AND (Q <> A)** (notare che non stiamo specificando un  valore assoluto 1 o 0 dei bit, ma ne stiamo eseguendo una comparazione relativa);
2. C7 e C8 sono invertiti tra loro, cioè **C7 <> C8** (anche in questo caso si esegue una comparazione relativa);
3. **(A7 = B7' = 1 AND Q7 = 0) OR (A7 = B7' = 0 AND Q7 = 1)** è simile al punto 1, ma stiamo specificando il valore assoluto dei bit anziché porli in una logica di comparazione relativa.

Per quanto riguarda il primo caso **(A == B) AND (Q <> A)**, il nostro circuito basato sulle ALU 74LS181 non ci offre visibilità del valore di B7', che è computato internamente al chip e non esposto.

![Primo metodo](../../assets/math/75-overflow-detector-xor-and.png)

Stesso ragionamento per il secondo caso **C7 <> C8**: non abbiamo visibilità di C7, che è computato internamente al chip e non esposto.

![Secondo metodo](../../assets/math/75-overflow-detector-xor.png)

Nemmeno il terzo metodo **(A7 = B7' = 1 AND Q7 = 0) OR (A7 = B7' = 0 AND Q7 = 1)** sembra utilizzabile, perché non abbiamo visibilità di B7': dobbiamo ricostruire artificialmente il segnale B7' basandoci sugli altri segnali in nostro possesso.

![Terzo metodo](../../assets/math/75-overflow-detector-and-or.png)

Quest'ultimo metodo è tuttavi *riciclabile* per la verifica dell'Overflow nelle *addizioni*: il valore B7' che l'ultimo adder troverà in ingresso sarà uguale al valore di B7 messo in input sul '181 (in una somma A7 + B7, B7' non subisce modifiche dalla circuiteria dell'adder):

![Overflow somma](../../assets/math/75-overflow-detector-a+b.png)

Qualche modifica permette di riutilizzare il metodo anche per la verifica dell'Overflow nelle *sottrazioni*: il valore B7' che l'ultimo adder troverà in ingresso sarà invertito rispetto al valore di B7 messo in input sul '181 (in una sottrazione somma A7 - B7, B7' viene invertito dalla circuiteria dell'adder):

![Overflow sottrazione](../../assets/math/75-overflow-detector-a-b.png)

Giunti a questo punto, per realizzare un circuito in grado di identificare l'Overflow avremmo bisogno di 4 porte AND con 3 ingressi e 3 porte OR con 2 ingressi (la terza OR servirebbe ad eseguire l'OR logico tra i due circuiti precedenti per creare un'unica segnalazione di Overflow tanto in caso di addizione quanto di sottrazione).

Al posto di AND e OR, un unico 74LS151 può fare al caso nostro: una configurazione dei pin di ingresso di questo tipo potrebbe risolvere le equazioni di Overflow sia per le addizioni, sia per la sottrazione A - B, quanto per quella B - A, che però nel nostro caso non è necessaria:

![74ls151](../../assets/math/75-overflow-74151.png)

Le combinazioni degli ingressi A, B e C del '151 (connessi agli MSB A7, B7 e Q7 dei registri A e B e dell'uscita Q dell'ALU) selezionano quale tra gli ingressi I0-I7 sia necessario attivare per portare in uscita una eventuale segnalazione di Overflow.

Gli ingressi I0-I7 sono opportunamente connessi in modalità "Hardwired" all'Instruction Register per capire quale sia l'operazione correntemente in esecuzione.

Facciamo alcuni esempi:

- **Esempio 1:** 0x20 + 0xC0; somma A + B di un Signed positivo e un Signed negativo

~~~text
    Hex        Dec        Bin             2C
C                                                     
A   0x20  ==>    32  ==>  0010.0000  ==>  0010.0000 +
B  +0xC0  ==>   -64  ==>  1100.0000  ==>  1100.0000 = 
               ----                      ----------
Q               -32                       1110.0000 ==> 0xE0 = -32, no Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A + B porterebbe a 1 gli ingressi I3 e I4 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=1, B7=1 e A7=0 sugli ingressi di selezione (CBA = 110), che attiverebbero l'ingresso I6 che risulta a 0 in quanto non attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 0, indicando che non vi è Overflow --> situazione verificata correttamente.

- **Esempio 2:** 0x20 + 0x70; somma A + B di due Signed positivi

~~~text
    Hex        Dec        Bin             2C
C                                         11
A   0x20  ==>    32  ==>  0010.0000  ==>  0010.0000 +
B  +0x70  ==>   112  ==>  0111.0000  ==>  0111.0000 = 
               ----                      ----------
Q               144                       1001.0000 ==> 0x90 = -112, Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A + B porterebbe a 1 gli ingressi I3 e I4 del '151, mentre tutti gli altri ingressi sarebbero a 0; l'operazione produrrebbe Q7=1, B7=0 e A7=0 sugli ingressi di selezione (CBA = 100), che attiverebbero l'ingresso I4 che risulta a 1 in quanto attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 1, indicando che vi è Overflow --> situazione verificata correttamente.

- **Esempio 3:** 0x50 - 0x30; sottrazione A - B di due Signed positivi

~~~text
    Hex        Dec        Bin             2C
C                                         11 1
A   0x50  ==>    80  ==>  0101.0000  ==>  0101.0000 +
B  -0x30  ==>   -48  ==>  0011.0000  ==>  1101.0000 = 
               ----                      ----------
Q                32                      10010.0000 ==> 0010.0000 ==> 0x20 = 32, no Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A - B porterebbe a 1 gli ingressi I1 e I6 del '151, mentre tutti gli altri ingressi sarebbero a 0; dall'operazione otterremmo Q7=0, B7=0 e A7=0 sugli ingressi di selezione (CBA = 000), che attiverebbero l'ingresso I0 che risulta a 0, pertanto l'uscita Q del '151 sarebbe a 0, indicando che non vi è Overflow --> situazione verificata correttamente.

- **Esempio 4:** 0x90 - 0x30; sottrazione A - B di un Signed negativo e un Signed positivo

~~~text
    Hex        Dec        Bin             2C
C                                        1  1
A   0x90  ==>  -112  ==>  1001.0000  ==>  1001.0000 +
B  -0x30  ==>   -48  ==>  0011.0000  ==>  1101.0000 = 
               ----                      ---------- 
Q              -160                      10110.0000 ==> 0110.0000 ==> 0x60 = 96, Overflow
~~~

Il microcode opportunamente codificato dell'istruzione A - B porterebbe a 1 gli ingressi I1 e I6 del '151, mentre tutti gli altri ingressi sarebbero a 0; dall'operazione otterremmo Q7=0, B7=0 e A7=1 sugli ingressi di selezione (CBA = 001), che attiverebbero l'ingresso I1 che risulta a 1 in quanto attivato dal microcode, pertanto l'uscita Q del '151 sarebbe a 1, indicando che vi è Overflow --> situazione verificata correttamente.

In realtà il modulo ALU del computer NQSAP - e di conseguenza, del BEAM, utilizza solo A + B e A - B, dunque possiamo semplificare in:

![74ls151](../../assets/math/75-overflow-74151-2.png)

Una regola che possiamo notare è che la somma di + e - non risulta mai in Overflow, come visto nell'esempio 1.