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

Lungi da me dal voler (e dal poter) spiegare tutto, suggerisco una serie di fonti che mi hanno permesso di arrivare a comprendere i temi esposti.

Se gli argomenti non sono chiari, raccomando di accedere alle fonti più volte in tempi diversi e in sequenze diverse; il rivedere le spiegazioni in modalità *brainstorming* mi avevno permesso di riempire piano piano i buchi fino a colmarli; con letture ed approfondimenti ripetuti avevo raggiunto lo scopo prefissato.

1. **Binary Overflow** - Mr Powell's Computer Science Channel - [link](https://www.youtube.com/watch?v=Q7t9-Sq_4Ww&lc=Ugy04nCzuiaOJIqK_5J4AaABAg): concetto generico di overflow.

    Ipotizziamo di avere un computer a 8 bit, in grado di sommare due numeri a 8 bit: sia gli addendi sia la somma sono numeri a 8 bit che possono dunque andare da 0 a 255.
      - Sommando ad esempio 73 + 114 si ottiene 187, che è un numero rappresentabile con 8 bit;
      - sommando invece ad esempio 175 + 92 si ottiene 267, che non è rappresentabile con un numero a 8 bit: abbiamo un errore di overflow, perché i bit a nostra disposizione (8) non ci consentono di rappresentare il risultato della somma.

    Abbiamo in pratica un riporto che dovremmo portare al 9° bit del nostro computer, che però ne ha solo 8. In questo caso il segnale di Carry in uscita dall'ALU ci segnalerebbe che il numero risultante dalla somma dei due addendi è più grande del numero calcolabile dall'ALU.

2. Video che illustra **Somme di numeri Unsigned, il concetto di Carry ed eventuale Overflow** - Mr Dimmick's Computing Channel - [link](https://www.youtube.com/watch?v=nKxjLM6ePcI): molto ben fatto; spiega anche le regole di base applicabili a tutte le somme di numeri binari.

3. **Numeri negativi in binario** dello stesso autore del video 1 - [link](https://www.youtube.com/watch?v=dHB7jFjESLY): un ottimo video che spiega come rappresentare i numeri negativi in Complemento a 2 e come convertire un numero positivo in negativo e viceversa.

    - Con il Complemento a 2, vi è un solo 0, a differenza di quanto accade con Modulo e Segno (si veda più avanti in questa pagina).
    - Per convertire un numero positivo in negativo è sufficiente invertire tutti i bit ed aggiungere 1.
    - Lo stesso procedimento è perfettamente valido anche al contrario (da negativo a positivo).
    - Prestare attenzione a un errore, riconosciuto dall'autore nei commenti: nella discussione di Segno e Modulo il range non va da -64 a +64, ma da -128 a +128.

4. **Binary Addition and Subtraction With Negative Numbers, 2's Complements & Signed Magnitude** - The Organic Chemistry Tutor - [link](https://www.youtube.com/watch?v=sJXTo3EZoxM): ottimo video con visualizzazione grafica molto efficace.

    - Particolarmente interessante un [commento](https://www.youtube.com/watch?v=sJXTo3EZoxM&lc=Ugyl7WIoyqVR8wM0dAJ4AaABAg) che segnala che per invertire il segno di un numero binario rappresentato in 2C è sufficiente seguire due passaggi:
      1) partire da destra e non modificare nulla fino al primo bit a 1 incluso;
      2) invertire tutti i bit rimanenti.

      Esempio: per invertire il numero 01010100 (84 decimale) lascio invariati i primi 3 bit partendo da destra (01010**100**) ed inverto poi tutti gli altri da **01010**100 a **10101**100 ottenendo come risultato finale 10101100 (-84).

# Un breve riepilogo

## Numeri Unsigned e numeri Signed

Prima di tutto, bisogna comprendere bene i numeri Unsigned e quelli Signed.

Non ricordo più quale sorgente (ricordavo un commento di un video di Mr Powell's Computer Science Channel, ma non riesco più a trovarlo) mi avesse portato in questa [interessantissima pagina](https://sandbox.mc.edu/~bennet/cs110/) dell'Università del Mississippi dedicata alla matematica binaria.

Grazie alla sezione Textbook ==> Module 3: Computer Integer Arithmetic ==> 2. Negative binary numbers avevo perfettamente compreso la rappresentazione binaria dei numeri negativi.

Il metodo **Signed Magnitude** è molto facile da comprendere: si sacrifica un bit dedicandolo alla rappresentazione del segno; il bit sacrificabile è quello più significativo (MSB), pertanto se un numero Unsigned a 8 bit può andare da 0 a 255 (2^8 = 256 combinazioni), sacrificando un bit per rappresentare un numero Unsigned potremo avere solo 7 bit disponibili per il Modulo (2^7 = 128 combinazioni), dunque il nostro Unsigned potrà andare da -128 a + 128.

![Rappresentazione Modulo e Segno dei numeri a 4 bit](../../assets/math/75-math_signed_magnitude.gif){:width="100%"}

*Rappresentazione Modulo e Segno dei numeri a 4 bit.*

Nell'immagine (semplificata a soli 4 bit per ragioni di spazio) si noterà un problema non secondario: lo zero appare due volte (0000 e 1000), ma noi sappiamo che lo zero non ha segno, pertanto questa rappresentazione non è la migliore possibile.

NB: nella pagina citata, vi è un esempio di sottrazione "5 - 2" che non comprendo e suppongo sia errata. L'immagine di riferimento è la seguente:

![Possibile errore nella pagina?](../../assets/math/75-math_mistake.gif
){:width="10%"}

Tralasciando la spiegazione del metodo **Complemento a 1 (1C)**, anch'esso non ottimale, il **Complemento a 2 (2C)** risulterà essere invece perfetto per la rappresentazione dei numeri negativi, portando in dote una grandissima semplificazione nell'esecuzione delle sottrazioni:

![Rappresentazione in Complemento a 2 dei numeri a 4 bit](../../assets/math/75-math_2c.gif){:width="100%"}

*Rappresentazione in Complemento a 2 dei numeri a 4 bit.*

Si nota subito la risoluzione del problema del doppio zero.

Altro beneficio importantissimo è che la sottrazione si realizza sommando il Complemento a 2 del numero da sottrare. Ad esempio, invece di eseguire "15 - 7" si effettua "15 + (-7)".

La regola che sta alla base della teoria del Complemento a 2 è: come posso rappresentare il numero "-1" di modo che aggiungendovi "1" possa ottenere "0"?

Similarmente ai vecchi tachimetri delle automobili, che una volta giunti a 99.999 passavano a 0, il 99.999 dell'aritmetica binaria a 8 bit corrisponde a 11111111, che sommato a 00000001 genera come risultato 00000000.

• 23/10/2022 oggi ho approfondito l'Overflow: se nella somma di due numeri signed noto un cambiamento di segno, allora ho un overflow
però l'NQSAP non lavora in Complemento a 2, dunque attenzione a cosa diciamo… qui non mi sembra di poter applicare il caso precedente… 27/11/2022 e in effetti rileggendo la questione è che stiamo lavorando non in Complemento a due, ma con numeri signed… 06/01/2023 rileggendo ulteriormente direi che non è proprio corretto. Il Complemento a 2 è semplicemente il modo di rappresentare i numeri signed, dove MSB = LO indica numero positivo e MSB = HI indica numero negativo.

## Approfondimento Overflow

L'approfondimento dell'overflow è stato un passaggio obbligato; prima di capire il funzionamento del circuito era necessario capire la logica che portava a definire come si dovesse definire e determinare una eventuale situazione di overflow.

Il link di riferimento è [http://6502.org/users/dieter/v_flag/v_0.htm](http://6502.org/users/dieter/v_flag/v_0.htm), nel quale l'autore Dieter inizialmente (http://6502.org/users/dieter/v_flag/v_1.htm) spiegava in maniera visiva in quali situazioni di somma o sottrazione tra due numeri Signed a 8 bit si generasse un overflow, utilizzando quella che si potrebbe forse definire una sorta di tavola pitagorica.

Ricordiamo che l'overflow indica un errore nel processo di somma o sottrazione di due numeri Signed: se il numero risultante ha un segno errato rispetto al previsto, si ha un overflow.

Nelle due rappresentazioni sottostanti le combinazioni Colonna/Riga al cui incrocio vi è un asterisco "\*" indicano situazioni di overflow, cioè di un risultato che non può essere correttamente rappresentato con i bit a disposizione.

~~~
Operazione       Sum Colonna + Riga       Sub Colonna - Riga
Flag:            Carry: no                Carry: sì
Risultato:       + 0123456789ABCDEF       + 0123456789ABCDEF
Risultato:       + 0000000000000000       - 0000000000000000
Risultato:       00................       00................
Risultato:       10.......*........       10........*.......
Risultato:       20......**........       20........**......
Risultato:       30.....***........       30........***.....
Risultato:       40....****........       40........****....
Risultato:       50...*****........       50........*****...
Risultato:       60..******........       60........******..
Risultato:       70.*******........       70........*******.
Risultato:       80........********       80********........
Risultato:       90........*******.       90.*******........
Risultato:       A0........******..       A0..******........
Risultato:       B0........*****...       B0...*****........
Risultato:       C0........****....       C0....****........
Risultato:       D0........***.....       D0.....***........
Risultato:       E0........**......       E0......**........
Risultato:       F0........*.......       F0.......*........

Esempio:         0x70+0x40 = Overflow     0xA0-0x30 = Overflow
~~~

Definizioni

- Le tabelle sono semplificate: sono mostrati solo i numeri multipli di 16 (0x**X**0), dunque 0x00, 0x10, 0x20 e così via.
- Si stanno utilizzando numeri Signed a 8 bit, dunque 0x20 rappresenta 32 decimale, mentre 0xA0 rappresenta 
- Per la somma **non c'è** un Carry in ingresso; per la sottrazione **c'è** un Carry in ingresso: come spiegato in [modulo ALU](../alu/#carry-addizioni-e-sottrazioni), prima di eseguire una somma il Carry è normalmente settato, mentre è normalmente settato prima di eseguire una sottrazione.

