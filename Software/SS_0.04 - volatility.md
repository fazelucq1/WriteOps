# Writeup: SS_0.04 - volatility

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 467  
- **Stato**: Up  
- **File Allegato**: `volatility` (ELF)

## Descrizione 📝
Questa challenge presenta un binario che decodifica la flag in memoria, la sovrascrive e poi termina. L’idea è che la flag rimanga in memoria per un brevissimo istante, e se vogliamo “catturarla” dobbiamo interrompere il flusso di esecuzione prima che venga cancellata. Lo scopo è usare un debugger (ad esempio **gdb**) per fermare il programma nel momento giusto e leggere la flag.

## Soluzione 🎯

### 1. Analisi del Binario
- **Funzione di Decodifica** (all’indirizzo `0x40127c`): decodifica la flag in memoria.
- **Funzione di Sovrascrittura** (all’indirizzo `0x401196`): subito dopo, il binario sovrascrive la flag.  
- **Termina**: il programma esce.

La flag è memorizzata in un’area di memoria il cui indirizzo è passato come primo argomento a entrambe le funzioni. È quindi variabile, ma lo vedremo con **gdb**.

### 2. Debug con gdb
Avviamo **gdb** sul binario `volatility`:
```bash
gdb ./volatility
```
Impostiamo un breakpoint prima che la flag venga sovrascritta, ossia all’indirizzo **`0x401196`**:
```gdb
(gdb) break *0x401196
```
Eseguiamo il programma:
```gdb
(gdb) run
```
Quando il breakpoint viene raggiunto, la flag è ancora in memoria.

### 3. Lettura della Flag
Una volta fermato il programma, usiamo il comando `info registers` per individuare il contenuto dei registri. Il primo argomento (`rdi` su sistemi x86_64) conterrà l’indirizzo della flag.  
Se sappiamo già l’indirizzo (es. `0x7fffffffdc00`), possiamo ispezionare la memoria con:
```gdb
(gdb) x/50c 0x7fffffffdc00
```
**Output** (convertito in ASCII):
```
67 'C' 67 'C' 73 'I' 84 'T' 123 '{' 66 'B' 114 'r' 51 '3'
97 'a' 107 'k' 112 'p' 48 '0' 105 'i' 110 'n' 116 't' 115 's'
95 '_' 82 'R' 95 '_' 117 'u' 53 '5' 101 'e' 102 'f' 117 'u'
76 'L' 125 '}'
```
La stringa è:

## Flag 🏁
```
CCIT{Br3akp0ints_R_u5efuL}
```

## Lezioni Apprese 📚
1. **Volatilità dei Dati in Memoria**: Il programma rende la flag disponibile solo per un istante prima di sovrascriverla.  
2. **Breakpoint Strategico**: Interrompere l’esecuzione nel momento esatto prima della sovrascrittura permette di leggere i dati in memoria.  
3. **Registri e Argomenti**: Su x86_64, i primi argomenti di una funzione vengono passati in registri (`rdi`, `rsi`, `rdx`, `rcx`, …).  
4. **Importanza del Debug**: Con **gdb** possiamo ispezionare la memoria e visualizzare informazioni che il binario non mostra esplicitamente.

## Tools Utilizzati 🛠️
- **gdb**: Per fermare il programma nel punto giusto e ispezionare la memoria.  
- **x/50c**: Comando di gdb per esaminare 50 caratteri (`c`) a partire da un determinato indirizzo.

---

*Writeup by Luca Crippa - Last Updated: 2024*
