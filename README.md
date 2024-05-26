# BEAM

## Il mio computer a 8 bit basato sul lavoro di Ben Eater e Tom Nisbet

Il BEAM è un computer su breaboard ispirato al [SAP 8-bit computer di Ben Eater](https://eater.net/8bit) e ai miglioramenti e alle espansioni presenti nell'[NQSAP di Tom Nisbet](https://github.com/tomnisbet/nqsap).

Alcuni aspetti interessanti del progetto:

* Set di istruzioni e indirizzamenti ispirati al processore 6502
* Registri A, X, Y come nel 6502
* Stack Pointer
* ALU basata su 74LS181 per tutte le operazioni aritmetiche e logiche
* Istruzioni di rotazione e shift implementate con shift-register
* Flag NVZC
* Sviluppo di bootloader e programmatore di EEPROM

In questo repository si trovano:

* [Documentazione completa del computer BEAM](https://andreamazzai.github.io/beam/)
* Schemi KiCad
* Software per il programmatore di EEPROM basato su Arduino Nano
* Software per il bootloader basato su Arduino Nano

[![BEAM Breadboard Computer](/docs/assets/beam.jpg "BEAM breadboard computer")](assets/images/nqsap.jpg)

| Left side       |  8-bit bus |      Right side |
|:---             |:----------:|             ---:|
| Clock           |    IIII    | Program Counter |
| RAM             |    IIII    | X Register      |
| RAM             |    IIII    | Y Register      |
| MAR             |    IIII    | D Register      |
| Control Logic   |    IIII    | A Register      |
| Control Logic   |    IIII    | H Register      |
| Ring Ctr / IR   |    IIII    | ALU             |
| Loader          |    IIII    | B Register      |
| Loader          |    IIII    | Flags Register  |
| Output Register |    IIII    | Flags Register  |
| Output / Loader |    IIII    | Stack Pointer   |
