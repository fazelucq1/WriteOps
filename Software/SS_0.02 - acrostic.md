# Writeup: SS_0.02 - acrostic

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 456  
- **Stato**: Up  
- **File Allegato**: `acrostic` (ELF)

## Descrizione 📝
La challenge parla di un “acrostico”, ossia un messaggio nascosto nelle prime lettere (o sillabe/parole) di ogni riga. All’interno di un eseguibile ELF, non troviamo stringhe evidenti che rivelino la flag. Il testo introduttivo lascia intendere che dobbiamo cercare un “nome nascosto” per comporre la flag.

## Soluzione 🎯

### 1. Analisi del Binario
Lo strumento `strings` non produce risultati utili. Possiamo invece ispezionare il binario con **objdump** o un disassembler come **Ghidra**.

#### 1.1 Uso di objdump
```bash
objdump -d acrostic
```
Si ottiene il disassemblato, che include alcune istruzioni:

```
...
000000000040100a <a_great_game>:
  40100a: 48 85 c0   test   %rax,%rax
  40100d: f2 0f 7c c1   haddps %xmm1,%xmm0
  401011: c8 00 00 00   enter  $0x0,$0x0
  401015: 48 8d 00      lea    (%rax),%rax
  401018: 48 01 c0      add    %rax,%rax
  40101b: f9            stc
  40101c: 48 85 c0      test   %rax,%rax
  40101f: 48 09 c0      or     %rax,%rax
  401022: dc c0         fadd   %st,%st(0)
  401024: 66 0f 14 c1   unpcklpd %xmm1,%xmm0
  401028: 48 29 c0      sub    %rax,%rax
...
```

### 2. Ricerca dell’Acrostico
Osserviamo la prima istruzione di ogni riga nel blocco “a_great_game”:

1. **test**
2. **haddps**
3. **enter**
4. **lea**
5. **add**
6. **stc**
7. **test** (di nuovo)
8. **or**
9. **fadd**
10. **unpcklpd**
11. **sub**

Prendendo la **prima lettera** di ciascuna istruzione, otteniamo:

```
t h e l a s t o f u s
```
Il risultato è **“thelastofus”**, un riferimento a un noto videogioco.

### 3. Composizione della Flag
Secondo le istruzioni, dobbiamo racchiudere il testo individuato in minuscolo tra `CCIT{...}`, ottenendo:

## Flag 🏁
```
CCIT{thelastofus}
```

## Lezioni Apprese 📚
1. **Acrostico negli Istruzioni Assembly**: Non sempre la flag è presente in chiaro; talvolta è nascosta nelle istruzioni (o in pattern) del disassemblato.  
2. **Strumenti di Analisi**: In assenza di stringhe palesi, `objdump` e **Ghidra** sono fondamentali per l’ispezione del flusso di codice.  
3. **Pattern Recognition**: La challenge sfrutta un metodo creativo per “nascondere” un messaggio, enfatizzando l’importanza di riconoscere pattern in un contesto insolito.

## Tools Utilizzati 🛠️
- **objdump**: Per disassemblare il binario.  
- **Ghidra** (opzionale): Per analizzare e navigare il codice in maniera interattiva.  
- **Shell Linux**: Per eseguire i comandi base di analisi.

---

*Writeup by Luca Crippa - Last Updated: 2024*
