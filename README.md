# BEAM

## Il mio computer a 8 bit basato sui lavori di Ben Eater e Tom Nisbet

Il BEAM è un computer su breaboard ispirato al [SAP 8-bit computer di Ben Eater](https://eater.net/8bit) e ai miglioramenti e alle espansioni presenti nell'[NQSAP di Tom Nisbet](https://github.com/tomnisbet/nqsap).

Alcuni aspetti interessanti del progetto:

* Set di istruzioni e indirizzamenti ispirati al processore 6502
* Registri A, X, Y come nel 6502
* Stack Pointer
* ALU basata su 74LS181 per tutte le operazioni aritmetiche e logiche
* Istruzioni di rotazione e shift implementate con shift-register
* Flag NVZC
* Sviluppo di bootloader e programmatore di EEPROM

Questo repository contiene:

* [Documentazione completa del computer BEAM](https://andreamazzai.github.io/beam/)
* Schemi KiCad
* Software per il programmatore di EEPROM basato su Arduino Nano
* Software per il bootloader basato su Arduino Nano

[![BEAM Breadboard Computer](/docs/assets/beam.jpg "BEAM breadboard computer")](docs/assets/beam.jpg)

| Left side       |  8-bit bus |      Right side |
|:---             |:----------:|             ---:|
| Clock           |  IIIIIIII  | Program Counter |
| RAM             |  IIIIIIII  | X Register      |
| RAM             |  IIIIIIII  | Y Register      |
| MAR             |  IIIIIIII  | D Register      |
| Control Logic   |  IIIIIIII  | A Register      |
| Control Logic   |  IIIIIIII  | H Register      |
| Ring Ctr / IR   |  IIIIIIII  | ALU             |
| Loader          |  IIIIIIII  | B Register      |
| Loader          |  IIIIIIII  | Flags Register  |
| Output Register |  IIIIIIII  | Flags Register  |
| Output / Loader |  IIIIIIII  | Stack Pointer   |
