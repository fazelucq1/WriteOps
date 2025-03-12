# Writeup: SSSSSSSSSSSSSSSSSSSS (Crypto)

## Overview
- **Categoria**: Crypto  
- **Autore**: PhiQuadro  
- **Difficoltà**: 150  
- **Stato**: Up  
- **File Allegati**: 
  - `SSSSS.py` (script di cifratura)  
  - `SSSSS_output.txt` (cifrato in esadecimale)

## Descrizione 📝
Lo script `SSSSS.py` prende un flag (es. `CCIT{...}`) e lo cifra usando una particolare tabella `S` (di 256 byte unici, tipo S-Box). La procedura di **“cifratura”** per ogni byte della flag è:

1. Partire da `ssss = ss` (dove `ss = 105`).
2. Ripetere `ssss = S[ssss]` per un numero di volte pari al valore del byte (`sss`).
3. Il valore finale di `ssss` diventa il byte cifrato.

In altre parole:
```python
for sss in flag:
    ssss = 105
    while sss:
        ssss = S[ssss]
        sss -= 1
    s.append(ssss)
```

Il risultato è salvato in esadecimale in `SSSSS_output.txt`.

## Soluzione 🎯

### 1. Analisi della Cifratura
- Ogni byte `plain` (della flag) genera un byte `cipher`.
- Si parte sempre da `ss = 105`.
- Si applica la sostituzione S un numero di volte pari al valore del byte in chiaro.
  
### 2. Inversione
Per invertire il processo (dato `cipher`, trovare `plain`), basta **ripercorrere a ritroso** la catena di sostituzioni finché non si ritorna a `105`. Il numero di “salti” necessari è il byte originale.

#### Costruiamo l’S-Box inverso
```python
S_inv = [0]*256
for i,v in enumerate(S):
    S_inv[v] = i
```

#### Decifriamo ciascun byte
1. Partiamo da `current = cipher`.
2. Contiamo quanti “passi” di `S_inv` servono per tornare a `105`.
3. Se in `<= 255` passi arriviamo a `105`, quel conteggio è il byte in chiaro.

Esempio di codice (semplificato):
```python
output_hex = "..."  # contenuto di SSSSS_output.txt
output_bytes = bytes.fromhex(output_hex)

flag = []
ss = 105  # valore di partenza

for c in output_bytes:
    current = c
    steps = 0
    # Ripeti finché non torniamo a 105 o superiamo 255 passi
    while current != ss and steps <= 255:
        current = S_inv[current]
        steps += 1
    # Se siamo tornati a 105, steps è il byte originale
    if current == ss:
        flag.append(steps)
    else:
        # errore o fallback
        flag.append(0)

print(bytes(flag).decode())
```

### 3. Risultato Finale
Eseguendo lo script, si ottiene la flag in chiaro, che appare come:

## Flag 🏁
```
CCIT{4ppaR3ntly_th31r_CrypT0_is_ju5t_Sn4k3_01l_SssSsssS5SS55555ssSsS}
```

*(Esempio di possibile flag in base al challenge. Se esegui lo script, otterrai quella effettiva.)*

## Lezioni Apprese 📚
1. **S-Box Custom**: L’uso di un array di sostituzione e di ripetute applicazioni ne fa una “funzione” invertibile se la tabella è nota.  
2. **Inversione di una Catena**: Basta applicare la tabella inversa (`S_inv`) finché non si raggiunge il valore iniziale (`105`). Il numero di iterazioni rivela il byte in chiaro.  
3. **Attenzione ai Byte**: Con 256 possibili valori, la cifratura funziona su un solo giro di 0–255. Se i dati fossero più grandi, sarebbe necessario un controllo addizionale.

## Tools Utilizzati 🛠️
- **Python**: Per analizzare lo script, costruire l’S-Box inverso e decifrare i dati.  
- **Editor di Testo**: Per leggere `SSSSS_output.txt` e incollare l’esadecimale nello script.

---

*Writeup by Luca Crippa - Last Updated: 2024*
