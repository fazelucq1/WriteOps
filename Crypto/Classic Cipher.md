# Writeup: Classic Cipher

## Descrizione della challenge

La challenge fornisce un testo cifrato:

```
xcqv{gvyavn_zvztv_etvtddlnxcgy}
```

e un codice Python che implementa una cifratura personalizzata basata su una chiave di tipo Caesar shift, che però ruota ad ogni carattere cifrato.

Lo scopo è decifrare il testo per trovare la flag, presumibilmente nel formato standard `flag{...}`.

---

## Analisi del codice di cifratura

Il codice fornito funziona così:

* L’alfabeto è `"abcdefghijklmnopqrstuvwxyz"`.
* Viene generata una chiave di cifratura come rotazione dell’alfabeto: ad esempio se lo shift iniziale è 5, la chiave sarà `fghijklmnopqrstuvwxyzabcde`.
* La funzione `encrypt(plain, key)` cripta il testo usando questa chiave, sostituendo ogni lettera con quella corrispondente nella chiave.
* La particolarità è che dopo ogni lettera cifrata la chiave **ruota a destra di 1 posizione** (ultima lettera della chiave diventa prima).
* I caratteri non alfabetici (come `{`, `_`) non vengono modificati.

---

## Difficoltà

* La chiave cambia ad ogni lettera, quindi non è un semplice Caesar shift ma un Caesar shift con rotazione dinamica.
* Lo shift iniziale della chiave è sconosciuto e può essere da 1 a 25.

---

## Strategia di decrittazione

* Possiamo fare un **bruteforce su tutti i 25 possibili shift iniziali** della chiave.
* Per ogni possibile chiave iniziale, simuliamo la decifratura inversa:

  * Troviamo per ogni carattere cifrato la posizione nella chiave corrente.
  * Convertiamo indietro nella lettera originale.
  * Ruotiamo la chiave a destra di 1.
* Controlliamo se il testo decifrato inizia con la tipica stringa `"flag{"` per riconoscere la flag.

---

## Codice per la decrittazione

```python
alphabet = "abcdefghijklmnopqrstuvwxyz"

def decrypt(ciphertext, start_shift):
    key = alphabet[start_shift:] + alphabet[:start_shift]
    plaintext = ""

    for c in ciphertext:
        if 'A' <= c <= 'Z':  # lettere maiuscole
            i = key.index(chr(ord(c) + 32))
            p = chr(ord(alphabet[i]) - 32)
            plaintext += p
            key = key[-1] + key[:-1]
        elif 'a' <= c <= 'z':  # lettere minuscole
            i = key.index(c)
            p = alphabet[i]
            plaintext += p
            key = key[-1] + key[:-1]
        else:
            plaintext += c

    return plaintext

ciphertext = "xcqv{gvyavn_zvztv_etvtddlnxcgy}"

for shift in range(1, 26):
    pt = decrypt(ciphertext, shift)
    if pt.startswith("flag{"):
        print(f"[+] Shift {shift} -> {pt}")
        break
```

---

## Risultato

Eseguendo il codice si ottiene:

```
[+] Shift 18 -> flag{REDACTED}
```
