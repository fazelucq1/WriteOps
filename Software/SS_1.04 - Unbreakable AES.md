# Writeup: SS_1.04 - Unbreakable AES

## Overview
- **Categoria**: Binary / Crypto  
- **Autore**: zxgio  
- **Difficoltà**: 499  
- **Stato**: Up  
- **File Allegato**: `flag.txt.aes` (file cifrato) + `unbreakable_aes` (programma di cifratura)

## Descrizione 📝
Il programma `unbreakable_aes` implementa un fantomatico schema di crittografia che, anziché usare un algoritmo standard AES, effettua una **rotazione a destra** (*rotate right*, `ror`) di ogni byte in base alla sua posizione nel file. Non esiste una funzione di decrittazione fornita, ma possiamo dedurla analizzando il codice.

## Soluzione 🎯

### 1. Analisi della Funzione di Cifratura
Il programma legge il file in blocchi di 16 byte. Per ogni singolo byte:
1. Calcola la posizione (inizialmente 1, poi 2, ecc.).
2. Esegue `ror(byte, pos % 8)`.  
   - Se `pos % 8 == 0`, il programma potrebbe ruotare di 8 bit, che equivale a lasciare il byte invariato.

### 2. Inversione dell’Operazione
Per invertire una rotazione a destra di `n` bit, basta eseguire una **rotazione a sinistra** di `n` bit. In altre parole, se `cifrato = ror(plain, n)`, allora `plain = rol(cifrato, n)`.

### 3. Script Python di Decrittazione
Implementiamo un piccolo script per decifrare `flag.txt.aes`. Supponiamo di chiamarlo `exploit.py`:

```python
def rotate_left(byte, n):
    n = n % 8
    return ((byte << n) | (byte >> (8 - n))) & 0xff

with open('flag.txt.aes', 'rb') as f:
    encrypted = f.read()

decrypted = bytearray()
for i, b in enumerate(encrypted):
    pos = i + 1
    n = pos % 8
    if n == 0:
        n = 8
    # Applica rotazione a sinistra di n bit
    decrypted.append(rotate_left(b, n))

with open('flag.txt', 'wb') as f:
    f.write(decrypted)
```

1. **rotate_left**: Funzione che esegue una rotazione a sinistra di `n` bit su un byte.  
2. **pos**: La posizione del byte (partendo da 1).  
3. **n**: Valore effettivo della rotazione, pari a `pos % 8`, o 8 se `pos % 8 == 0`.  
4. **Salvataggio**: Scriviamo il risultato in `flag.txt`.

### 4. Lettura della Flag
Eseguendo lo script, otteniamo un file `flag.txt` in chiaro. Visualizziamo il contenuto:
```bash
cat flag.txt
The flag is ... CCIT{n0t_re4llY_using_K}
BTW DON'T INVENT YOUR OWN CRYPTO!!!
```

## Flag 🏁
```
CCIT{n0t_re4llY_using_K}
```

## Lezioni Apprese 📚
1. **“AES” Personalizzato**: Il binario non usa il vero AES, bensì una rotazione a destra, facilmente invertibile.  
2. **Rotazioni Bitwise**: Sfruttare `rol` e `ror` per offuscare i dati non è una misura di sicurezza robusta.  
3. **Non Inventare il Proprio Algoritmo**: Un motto classico nella crittografia è **“Don't roll your own crypto”**.  

## Tools Utilizzati 🛠️
- **Ghidra** o **radare2**: Per analizzare il binario `unbreakable_aes` e comprendere la logica di cifratura.  
- **Python**: Per implementare lo script di decrittazione e recuperare la flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
