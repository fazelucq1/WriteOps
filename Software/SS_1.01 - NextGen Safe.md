# Writeup: SS_1.01 - NextGen Safe

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 471  
- **Stato**: Up  
- **File Allegato**: `nextgen_safe` (ELF)

## Descrizione 📝
La challenge è un’evoluzione di **“The Safe”**. Questa volta il binario non accetta alcuna password: qualsiasi input forniamo, risponderà con un messaggio tipo “Nice try ;)”. L’obiettivo rimane ottenere la flag nascosta nel programma.

## Soluzione 🎯

### 1. Analisi del Binario
Utilizziamo strumenti come **Ghidra** o **radare2** per esplorare il codice. Notiamo una funzione `print_safe_contents()` che sembra contenere la logica per stampare la flag, ma non viene mai chiamata dal `main`.

### 2. Debug con GDB
Per richiamare la funzione `print_safe_contents()` manualmente, carichiamo il binario in **gdb**:

```bash
gdb ./nextgen_safe
```

1. **Impostiamo un breakpoint** all’inizio di `main`:
   ```gdb
   (gdb) break main
   ```
2. **Eseguiamo il programma** fino al breakpoint:
   ```gdb
   (gdb) run
   ```
3. **Chiamiamo la funzione** `print_safe_contents()`:
   ```gdb
   (gdb) call (void)print_safe_contents()
   ```
   Se gdb segnala “unknown return type”, aggiungiamo un cast esplicito (ad esempio `(void)`).

### 3. Lettura della Flag
Il programma stamperà la flag:

```
CCIT{gdb_t0_th3_re5cue}
```

## Flag 🏁
```
CCIT{gdb_t0_th3_re5cue}
```

## Lezioni Apprese 📚
1. **Funzioni Non Chiamate**: Spesso i binari contengono funzioni non utilizzate dal flusso principale, ma comunque presenti in memoria.  
2. **Uso di GDB**: Possiamo manipolare il flusso di esecuzione chiamando manualmente funzioni interne.  
3. **Difesa**: Se non si vuole che i contenuti siano esposti, occorre offuscare il codice o rimuovere completamente funzioni non necessarie.

## Tools Utilizzati 🛠️
- **Ghidra**: Per analizzare il codice e individuare funzioni nascoste.  
- **gdb**: Per fermare l’esecuzione e chiamare manualmente `print_safe_contents()`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
