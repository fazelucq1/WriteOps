# Writeup: CCIA

## Descrizione 📝
La challenge fornisce un messaggio cifrato in un array di byte (in esadecimale), che rappresenta caratteri **in complemento a due**. L’obiettivo è ricostruire il messaggio originale convertendo ogni byte dal complemento a due al carattere ASCII.

## Complemento a Due 🎯

1. **Byte esadecimale**: Esempio `0xBD`  
2. **Invertire i bit**: `0xBD = 10111101` diventa `01000010`  
3. **Somma di 1**: `01000010 + 1 = 01000011`  
4. **Interpretazione**: `0x43 = 'C'`

Ripetendo l’operazione su ogni byte, otteniamo la stringa in chiaro.

## Esempio di Calcolo
- **Primo byte**: `0xBD`  
  - Inverti: `10111101` → `01000010`  
  - +1 → `01000011` = `0x43` = `'C'`
- **Ultimo byte**: `0x83`  
  - Inverti: `10000011` → `01111100`  
  - +1 → `01111101` = `0x7D` = `'}'`

## Risultato 🏁
Unendo tutti i caratteri decodificati, si ottiene:
```
CCIT{th1s_messag3_will_self_encryp7_in_3_2_1_fd64461f}
```

## Lezioni Apprese 📚
1. **Complemento a Due**: Metodo standard per rappresentare i numeri negativi, qui usato come offuscamento.  
2. **Bitwise Operations**: Operazioni di inversione e somma di 1 per decodificare i caratteri.  
3. **Attenzione ai Formati**: Se un byte è fornito come “valore in complemento a due”, è necessario riconvertirlo al suo valore ASCII originale.

## Tools Utilizzati 🛠️
- **Editor di Testo / Script Python**: Per calcolare automaticamente il complemento a due e convertire in ASCII.  
- **Hex Editor** (opzionale): Per ispezionare rapidamente i byte.

---

*Writeup by Luca Crippa - Last Updated: 2024*
