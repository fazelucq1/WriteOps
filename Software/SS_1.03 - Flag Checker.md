# Writeup: SS_1.03 - Flag Checker

## Overview
- **Categoria**: Binary  
- **Autore**: zxgio  
- **Difficoltà**: 496  
- **Stato**: Up  
- **File Allegato**: `flag_checker` (ELF)

## Descrizione 📝
La challenge fornisce un programma che dovrebbe “verificare” la flag, ma in pratica continua a bloccarsi (crash). L’indizio nella descrizione suggerisce di analizzare come il binario calcola o memorizza la flag invece di tentare brute force.  

All’interno del binario (usando **Ghidra** o **radare2**) si individua un array di valori nella sezione `.data` (`flag_data`). L’analisi del codice rivela che questi valori rappresentano somme cumulative dei caratteri ASCII della flag.

## Soluzione 🎯

### 1. Individuazione di `flag_data`
Con Ghidra, troviamo un array simile a:
```
flag_data = [
    84, 195, 290, 395, 479, 580, 694, 746, 862, 963,
    1058, 1163, 1216, 1311, 1415, 1500, 1609, 1706,
    1784, 1879, 1995, 2043, 2138, 2252, 2303, 2370,
    2487, 2569, 2684, 2785, 2880, 2980, 3029, 3147,
    3252, 3362, 3463, -1
]
```
Ciascun valore è la **somma cumulativa** dei codici ASCII dei caratteri della flag. Il `-1` segna la fine dei dati.

### 2. Decodifica Deterministica
Se `flag_data[i]` è la somma di tutti i caratteri fino a `i`, allora il carattere `i` si ottiene da:
```
char_i = flag_data[i] - sum_precedente
```
dove `sum_precedente` è la somma dei caratteri già decodificati fino a `i-1`.

#### 2.1 Script Python
```python
def trova_flag(flag_data):
    result = []
    previous_sum = 0
    for i in range(len(flag_data)):
        if flag_data[i] < 0:
            break
        current_char_code = flag_data[i] - previous_sum
        current_char = chr(current_char_code)
        result.append(current_char)
        previous_sum += current_char_code
    return ''.join(result)

flag_data = [
    84, 195, 290, 395, 479, 580, 694, 746, 862, 963,
    1058, 1163, 1216, 1311, 1415, 1500, 1609, 1706,
    1784, 1879, 1995, 2043, 2138, 2252, 2303, 2370,
    2487, 2569, 2684, 2785, 2880, 2980, 3029, 3147,
    3252, 3362, 3463, -1
]

decoded_flag = trova_flag(flag_data)
print(f"Flag: {decoded_flag}")
```
Eseguiamo lo script, che ricostruisce la flag in modo istantaneo.

### 3. Flag
Il risultato è:
```
[+] Flag trovata: To_iTer4te_i5_hUmaN_t0_r3CuRse_d1vine
```

## Flag 🏁
```
CCIT{To_iTer4te_i5_hUmaN_t0_r3CuRse_d1vine}
```

## Lezioni Apprese 📚
1. **Somme Cumulative**: Metodo semplice per offuscare la flag, ma facilmente invertibile.  
2. **Analisi con Ghidra**: Ancora una volta, Ghidra ci aiuta a individuare l’array di dati nella sezione `.data`.  
3. **Evitare il Brute Force**: Spesso i binari forniscono schemi deterministici per costruire la flag, evitando l’inutile tentativo di forzare l’input.

## Tools Utilizzati 🛠️
- **Ghidra**: Per localizzare `flag_data` e comprendere la logica di calcolo.  
- **Python**: Per la ricostruzione rapida e indolore della flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
