---
title: "Aritmetica binaria"
permalink: /docs/math/
excerpt: "Aritmetica binaria"
---

## Concetti importanti

Per comprendere appieno il funzionamento dei moduli Flag e ALU, è stato necessario approfondire anche alcuni concetti dell'aritmetica binaria.

Cercando di assimilare il concetto di overflow, 

I concetti utili da comprendere sono:

1. Numeri binari senza segno (Unsigned)
2. Numeri binari con segno (Signed)
3. Modalità di rappresentazione dei numeri binari con segno: Magnitudo + Segno, Complemento di 1, Complemento di 2
4. Somma e sottrazione di numeri binari


Lungi da me dal voler (e dal poter) spiegare tutto, suggerisco una serie di fonti che mi hanno permesso di arrivare a comprendere i temi esposti.

Se gli argomenti non sono chiari, raccomando di accedere alle fonti più volte in tempi diversi e in sequenze diverse; il rivedere le spiegazioni in modalità brainstorming mi ha permesso di riempire piano piano i buchi fino a colmarli; avevo notato che le ripetizioni dei concetti mi avevano permesso di raggiungere lo scopo prefissato.

1. Binary Overflow - Mr Powell's Computer Science Channel - https://www.youtube.com/watch?v=Q7t9-Sq_4Ww&lc=Ugy04nCzuiaOJIqK_5J4AaABAg
Concetto generico di overflow.

2. Altro video che illustra somme di numeri Unsigned, il concetto di Carry ed eventuale Overflow: https://www.youtube.com/watch?v=nKxjLM6ePcI - molto ben fatto; spiega anche le "regole" di base applicabili a tutte le somme di numeri binari.

3. Numeri negativi in binario dello stesso autore del video 1: https://www.youtube.com/watch?v=dHB7jFjESLY (Attenzione a un errore, riconosciuto dall'autore nei commenti: nella discussione di Segno e Magnitudo il range non va da -64 a +64, ma da -127 a +127 - in ogni caso, un ottimo video che spiega come si rappresentatno i numeri negativi in complementio di 2 e come convertire un numero positivo in negativo e viceversa.







Binary Addition and Subtraction With Negative Numbers, 2's Complements & Signed Magnitude: https://www.youtube.com/watch?v=sJXTo3EZoxM


Nel caso 1 delle operazioni con numeri senza segno. ipotizziamo di avere un computer a 8 bit, in grado di sommare due numeri a 8 bit: sia gli addendi sia la somma sono numeri a 8 bit che possono dunque andare da 0 a 255. Sommando ad esempio 73 + 114 si ottiene 187, che è un numero rappresentabile con 8 bit; sommando invece ad esempio 175 + 92 si ottiene 267, che non è rappresentabile con un numero a 8 bit: abbiamo un errore di overflow, perché i bit a nostra disposizione (8) non ci consentono di rappresentare il risultato della somma. Abbiamo in pratica un riporto che dovremmo portare al 9° bit del nostro computer, che però ne ha solo 8. In questo caso il segnale di Carry in uscita dall'ALU ci segnalerebbe che il nuero risultante dalla somma dei due addendi è più grande del numero calcolabile dall'ALU.

• 23/10/2022 oggi ho approfondito l'Overflow: se nella somma di due numeri signed noto un cambiamento di segno, allora ho un overflow
però l'NQSAP non lavora in complemento di due, dunque attenzione a cosa diciamo… qui non mi sembra di poter applicare il caso precedente… 27/11/2022 e in effetti rileggendo la questione è che stiamo lavorando non in complemento di due, ma con numeri signed… 06/01/2023 rileggendo ulteriormente direi che non è proprio corretto. Il complemento di 2 è semplicemente il modo di rappresentare i numeri signed, dove MSB = LO indica numero positivo e MSB = HI indica numero negativo.
