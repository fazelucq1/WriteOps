# Writeup: CR_1.03 - Dropped

## Descrizione 📝
La challenge fornisce un messaggio cifrato:
```
TCICmI_{_d343_m4}s!s
```
e suggerisce che la decifratura avviene scambiando il primo e l’ultimo carattere di ogni blocco di 4 caratteri. Se un blocco contiene meno di 4 caratteri, rimane invariato. L’obiettivo è ricostruire la flag nascosta all’interno del testo.

## Soluzione 🎯

1. **Suddivisione in blocchi di 4 caratteri**  
   Ad esempio:  
   - `TCIC`  
   - `mI_{`  
   - `_d34`  
   - `3_m4`  
   - `}s!s` (questo blocco ha 4 caratteri)  

2. **Scambio del primo e quarto carattere**  
   Per ogni blocco di 4 caratteri, se `block = abcd`, diventa `d + bc + a`.  
   Esempio:  
   - `TCIC` → `C` + `CI` + `T` = `CCIT`  
   - `mI_{` → `{` + `I_` + `m` = `{I_m`  
   - e così via...

3. **Concatenazione dei blocchi trasformati**  
   Riunendo i blocchi trasformati si ottiene il testo in chiaro.

### Esempio di Codice
```python
def transform_block(block):
    if len(block) == 4:
        return block[3] + block[1:3] + block[0]
    return block

def decrypt_message(message):
    blocks = [message[i:i+4] for i in range(0, len(message), 4)]
    result = [transform_block(b) for b in blocks]
    return ''.join(result)

msg = "TCICmI_{_d343_m4}s!s"
decrypted = decrypt_message(msg)
print(decrypted)
```
Eseguito lo script, otteniamo:

## Flag 🏁
```
CCIT{I_m4d3_4_m3ss!}
```

## Lezioni Apprese 📚
1. **Cifratura a Blocchi Semplice**: Anche una manipolazione elementare come lo scambio di caratteri può essere invertita se si conosce la regola.  
2. **Suddivisione del Messaggio**: Dividere in blocchi di dimensione fissa (4 byte) è una pratica comune in alcuni cifrari, ma in questo caso è solo un offuscamento.  
3. **Importanza dei Dettagli**: Se un blocco è più breve di 4 caratteri, non viene toccato, e questo dettaglio può cambiare la decifratura finale.

## Tools Utilizzati 🛠️
- **Python**: Per implementare rapidamente la funzione di decifratura e testare la trasformazione.

---

*Writeup by Luca Crippa - Last Updated: 2024*
