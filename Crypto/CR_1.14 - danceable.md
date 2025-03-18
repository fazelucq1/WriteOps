# Writeup: CR_1.14 - danceable

**Descrizione**  
La challenge fornisce un servizio remoto che cifra i messaggi (inviati in formato esadecimale) concatenati alla flag tramite **ChaCha20**. L’implementazione presenta però un grave errore di sicurezza: **riutilizza la stessa chiave e lo stesso nonce** (riutilizzo di nonce in ChaCha20) per ogni blocco di 16 byte. Poiché ChaCha20 è uno stream cipher, usare la stessa coppia `(key, nonce)` per blocchi diversi consente di estrarre facilmente la chiave di flusso e decifrare la flag.

```python
def encrypt(text):
    text = bytes.fromhex(text)
    to_encrypt = text + flag.encode()
    assert len(to_encrypt) < 256, "The message is too long!"
    padded = pad(to_encrypt, 16)
    blocks = [padded[i:i+16] for i in range(0, len(padded), 16)]
    ct = b""
    for block in blocks:
        cipher = ChaCha20.new(key=key, nonce=nonce)
        ct += cipher.encrypt(block)
    return ct.hex()
```

> Notiamo che per **ogni blocco** viene ricreato l’oggetto ChaCha20 con la stessa chiave e lo stesso nonce, generando la **stessa** porzione di keystream per ogni blocco.

---

## Sfruttamento della Vulnerabilità

1. **Connessione al Servizio**  
   ```bash
   nc danceable.challs.cyberchallenge.it 9036
   ```
2. **Richiesta di Cifratura**  
   Mandiamo in input 32 byte di `\x00` (in hex, `00 * 32`). Verranno prodotti almeno due blocchi di ciphertext (16 byte l’uno).
3. **Keystream e Decifratura**  
   - Il **primo blocco** cifrato corrisponde a 16 byte di `\x00`, quindi è identico al keystream.  
   - I blocchi successivi contengono la concatenazione tra il secondo blocco di `\x00` e la flag. Usando lo stesso keystream, possiamo decifrarli.
4. **Unpad e Lettura della Flag**  
   Rimuoviamo l’eventuale padding e otteniamo la flag in chiaro.

---

## Codice di Exploit (Sintetizzato)

```python
from pwn import *
from Cryptodome.Util.Padding import unpad

def xor(a, b):
    return bytes([x ^ y for x, y in zip(a, b)])

r = remote('danceable.challs.cyberchallenge.it', 9036)

# Chiediamo la cifratura di 32 byte di zero
r.sendlineafter(b'> ', b'1')
r.sendlineafter(b'in hex)? ', b'00'*32)

ct_hex = r.recvline().strip().decode()
ct = bytes.fromhex(ct_hex)

# Dividiamo il ciphertext in blocchi di 16 byte
blocks = [ct[i:i+16] for i in range(0, len(ct), 16)]

# Il primo blocco è il keystream
keystream = blocks[0]

# Decifriamo tutti i blocchi
padded_plaintext = b''
for block in blocks:
    decrypted_block = xor(block, keystream)
    padded_plaintext += decrypted_block

# I primi 16 byte erano i nostri \x00, il resto contiene la flag
flag_padded = padded_plaintext[16:]
flag = unpad(flag_padded, 16).decode()

print("Flag:", flag)
```

---

## Flag

```
CCIT{r3us3d_n0nc3_1n_Ch4Ch420}
```

---

## Lezioni Apprese

1. **Riutilizzo di (Key, Nonce) in ChaCha20**  
   In un cifrario a flusso come ChaCha20, riusare la stessa coppia `(key, nonce)` per più blocchi porta a un keystream ripetuto.  
2. **Stream Cipher**  
   Se il keystream è identico, XOR con blocchi noti (come `\x00`) rivela il keystream stesso, consentendo la decifratura di qualunque altro blocco.  
3. **Miglior Pratica**  
   Ogni messaggio, o almeno ogni blocco, dovrebbe avere nonce univoci e mai ripetuti per garantire la sicurezza di ChaCha20.

---

*Writeup by Luca Crippa - Last Updated: 2024*
