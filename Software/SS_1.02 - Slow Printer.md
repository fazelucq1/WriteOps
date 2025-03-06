# Writeup: SS_1.02 - Slow Printer

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 474  
- **Stato**: Up  
- **File Allegato**: `slow_printer` (ELF)

## Descrizione 📝
Questo programma, quando eseguito, stampa la flag in modo estremamente lento e si blocca prima di completare la stampa. L’idea è che l’utente sia costretto ad aspettare un tempo lunghissimo per vedere la flag completa. L’obiettivo è rimuovere la parte che induce la lentezza (ad esempio una chiamata a `sleep()`), consentendo al programma di stampare la flag istantaneamente.

## Soluzione 🎯

### 1. Esecuzione Iniziale
Se lanciamo `./slow_printer`, vedremo un output parziale, come:
```
Your flag is:
CCIT{tim3_fli
```
Dopo un lungo tempo di attesa, si blocca e non completa la stampa della flag.

### 2. Analisi con Ghidra
Carichiamo il binario in Ghidra e ispezioniamo la funzione che gestisce la stampa. Troveremo qualcosa di simile:

```c
while (true) {
    printf("\rCCIT{%s}", &DAT_00404070);
    fflush(stdout);
    sleep();               // Lentezza introdotta qui
    if (local_c != 0x2000) break;
    DAT_00404080 = 0x73;
    local_c = 0xd;
}
```
La chiamata a `sleep()` provoca l’attesa (o la lentezza eccessiva).

### 3. Rimozione della Sleep
In Ghidra, possiamo **patchare** l’istruzione `CALL sleep` sostituendola con un’istruzione innocua, ad esempio `MOV EAX, 0x0`, in modo che la funzione non blocchi più l’esecuzione. Dopo aver eseguito la patch, esportiamo il binario modificato (ad esempio `slow_printer_patched`).

### 4. Esecuzione del Binario Patched
Avviamo il binario modificato:
```bash
./slow_printer_patched
```
Adesso la flag viene stampata immediatamente e per intero:
```
Your flag is:
CCIT{tim3_flie5_WiTh_s0m3_heLp}
```

## Flag 🏁
```
CCIT{tim3_flie5_WiTh_s0m3_heLp}
```

## Lezioni Apprese 📚
1. **Patchare un Binario**: Con Ghidra (o un hex editor) è possibile rimuovere o sostituire istruzioni per modificare il comportamento del programma.  
2. **Analisi del Flusso**: Identificare dove si trova la chiamata a `sleep()` ci permette di sbarazzarci della lentezza artificiale.  
3. **Offuscamento Semplice**: Introdurre ritardi o blocchi è un metodo per complicare l’analisi, ma non è efficace contro strumenti di reversing.

## Tools Utilizzati 🛠️
- **Ghidra**: Per analizzare e patchare l’istruzione `sleep()`.  
- **Shell Linux**: Per eseguire il binario originale e quello modificato.

---

*Writeup by Luca Crippa - Last Updated: 2024*
