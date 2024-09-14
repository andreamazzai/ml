# opzione 2

Utilizzato nella pagina della control logic e sostituito con elenco puntato più dettagliato

1. Il primo step espone sul bus l'indirizzo del Program Counter (RPC, Read Program Counter) e scrive il contenuto del bus nel MAR (WM, Write Memory Address Register); così facendo, il MAR conterrà l'indirizzo attuale del PC.
2. Il secondo step espone sul bus il contenuto della locazione di memoria puntata dal MAR (RR, Read RAM), scrive il contenuto del bus nell'Instruction Register (WIR, Write Instruction Register) e incrementa il Program Counter (Program Counter Increment). Alla fine di questo step, l'IR conterrà l'opcode dell'istruzione e il PC punterà alla locazione di memoria successiva, che nel caso dell'istruzione LDA contiene l'operando.
3. Il terzo step espone di nuovo il contenuto del Program Counter sul bus (RPC, Read Program Counter) e si scrive il contenuto del bus nel MAR (WM, Write Memory Address Register). Ora, il MAR punta alla locazione di memoria che contiene l'operando.
4. Il quarto ed ultimo step espone sul bus l'operando, che è contenuto nella locazione di memoria puntata dal MAR (RR, Read RAM), abilita la scrittura dei Flag N e Z (FNZ), scrive il contenuto del bus su A e H (WAH, Write A + H), incrementa il Program Counter (PCI, Program Counter Increment) e resetta il Ring Counter (N, Next).
