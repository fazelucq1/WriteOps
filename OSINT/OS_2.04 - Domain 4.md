# Writeup: OS_2.04 - Domain 4

## Overview
- **Categoria**: OSINT  
- **Autore**: osintitalia  
- **Difficoltà**: 488  
- **Domanda**: *At what time did archive.org do the first snapshot of the web page on June 14, 2006? (Format: hh:mm:ss)*

## Descrizione 📝
La challenge richiede di consultare la **Wayback Machine** di Archive.org per il dominio in questione (ad esempio, `libero.it`) e individuare l’orario esatto del primo snapshot realizzato **il 14 giugno 2006**.

## Soluzione 🎯

### 1. Accesso a Wayback Machine
1. Visitiamo [archive.org/web/](https://archive.org/web/)  
2. Inseriamo l’URL `https://www.libero.it` nella barra di ricerca.  
3. Selezioniamo l’anno **2006** e poi il giorno **14 giugno 2006** dal calendario.

### 2. Individuazione del Primo Snapshot
- La pagina mostra un elenco o un grafico dei vari snapshot.  
- Identifichiamo il **primo snapshot** disponibile in quella data, con l’orario riportato.

### 3. Risposta
Secondo i dati, il primo snapshot del 14/06/2006 è avvenuto alle:
![image](https://github.com/user-attachments/assets/536db341-4990-449b-b0fc-4ca7352afe6e)


## Flag 🏁
```
01:41:45
```

## Lezioni Apprese 📚
1. **Wayback Machine**: Uno strumento OSINT essenziale per recuperare versioni passate di siti web.  
2. **Ricerca Cronologica**: Quando viene richiesto un orario, è sufficiente esaminare il primo snapshot disponibile nel giorno indicato.

## Tools Utilizzati 🛠️
- **Archive.org / Wayback Machine**: Per visualizzare gli snapshot storici di un sito web.

---

*Writeup by Luca Crippa - Last Updated: 2024*
