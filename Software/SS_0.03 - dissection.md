# Writeup: SS_0.03 - dissection

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 459  
- **Stato**: Up  
- **File Allegato**: `dissection` (ELF)

## Descrizione 📝
La challenge fornisce un binario chiamato **`dissection`** che, una volta eseguito, non stampa nulla di significativo. L’obiettivo è “dissezionare” l’ELF per trovare la flag nascosta, molto probabilmente all’interno di sezioni non utilizzate dal programma.

## Soluzione 🎯

### 1. Analisi con `readelf`
Per esaminare la struttura delle sezioni di un eseguibile ELF, utilizziamo:
```bash
readelf -S dissection
```
Questo comando elenca tutte le sezioni, incluse quelle personalizzate che possono contenere dati arbitrari. Nel caso specifico, notiamo sezioni dal numero **[17]** al **[25]** che riportano stringhe “insolite”:

```
[17] CCI
[18] T{I_
[19] c
[20] _d3
[21] ad_
[22] sec
[23] ti
[24] 0n
[25] 5}
```

Se concateniamo il contenuto di queste sezioni, otteniamo:

## Flag 🏁
```
CCIT{I_c_d3ad_sec_ti0n5}
```

## Lezioni Apprese 📚
1. **Sezioni Custom ELF**: Un binario può includere sezioni personalizzate che non vengono necessariamente utilizzate dal codice eseguibile, ma possono contenere dati nascosti.  
2. **Strumenti di Analisi**: Comandi come `readelf -S` e `objdump -s` sono fondamentali per ispezionare la struttura e il contenuto delle sezioni di un ELF.  
3. **Protezione dei Binari**: Se un autore desidera nascondere dati, può inserirli in sezioni non standard, sebbene ciò non costituisca una vera misura di sicurezza (poiché è facile scovarle con gli strumenti adatti).

## Tools Utilizzati 🛠️
- **readelf**: Per esaminare le sezioni dell’ELF.  
- **Shell Linux**: Per eseguire i comandi di analisi.

---

*Writeup by Luca Crippa - Last Updated: 2024*
