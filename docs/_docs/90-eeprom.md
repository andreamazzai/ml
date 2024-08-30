---
title: "EEPROM Programmer"
permalink: /docs/eeprom-programmer/
excerpt: "EEPROM Programmer"
---
## PLACEHOLDER - pagina non iniziata


![Utilizzo del 74LS151 nell'NQSAP per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.](../../assets/flags/30-flag-v-151.png){:width="50%"}

*Utilizzo del 74LS151 nell'NQSAP per il calcolo dell'Overflow con evidenza degli MSB di H, B e dell'ALU e degli ingressi di selezione dell'operazione IR-Q1 e IR-Q3.*

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

Il flag Overflow si attiva se:

- **(A7 = B7 = 1 AND Q7 = 0)** *E* stiamo eseguendo una addizione (IR-Q3 attivo), oppure
- **(A7 = B7 = 0 AND Q7 = 1)** *E* stiamo eseguendo una addizione (IR-Q3 attivo), oppure
- **(A = 1 AND B = 0 AND Q = 0)** *E* stiamo eseguendo una sottrazione (IR-Q1 attivo), oppure
- **(A = 0 AND B = 1 AND Q = 1)** *E* stiamo eseguendo una sottrazione (IR-Q1 attivo).

Per identificare l'esecuzione di un'operazione di addizione o di sottrazione, si utilizzano due linee dell'Instruction Register:

| IR-Q3 | IR-Q1 | Operazione  |
| -     | -     | -           |
| LO    | HI    | Sottrazione |
| HI    | LO    | Addizione   |

Detto in altri termini, gli opcode delle istruzioni di addizione e sottrazione dovranno avere i bit 1 e 3 settati come in tabella, mentre gli altri bit non avranno tali vincoli:

| bit |  Addizione | Sottrazione |
| -   |  -         | -           |
|   0 |      X     |           X |
|   1 | **LO**     |      **HI** |
|   2 |      X     |           X |
|   3 | **HI**     |      **LO** |
|   4 |      X     |           X |
|   5 |      X     |           X |
|   6 |      X     |           X |
|   7 |      X     |           X |

Questo corrisponde esattamente a quanto indicato nella sezione [Un esempio pratico](../alu/#un-esempio-pratico) della pagina del modulo ALU, nella quale si evidenzia ad esempio che le istruzioni di somma dovranno presentare "01001 sui 5 bit comuni tra Instruction Register e ALU": il bit numero 3 è a 1, mentre il bit numero 1 è a 0, esattamente come richiesto dalla tabella precedente.

![Connessioni in uscita dall'Instruction Register dell'NQSAP.](../../assets/math/IR-to-74151.png){:width="66%"}

Per finire, da quanto visto fino ad ora possiamo dedurre un'altra regola: la somma di due Signed di segno opposto e la sottrazione di due Signed dello stesso segno non possono causare Overflow.


## TO DO

- Ripple Mode Carry e Carry Look Ahead --- Ne parliamo perché i '161 usati nel MAR e i '181 dell'ALU ne parlano nei datasheet.
- Link a quel professore del montana
