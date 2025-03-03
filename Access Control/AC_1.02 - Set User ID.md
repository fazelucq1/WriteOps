# Writeup: AC_1.02 - Set User ID

## Overview
- **Categoria**: Access Control  
- **Autore**: enriquez, avalz  
- **Difficoltà**: 485  
- **Stato**: Up  
- **URL**: (esercizio locale)

## Descrizione 📝
La sfida richiede di trovare un file eseguibile di proprietà dell’utente `flag00` con il bit **Set User ID (SUID)** attivo. L’obiettivo è accedere al contenuto di `flag.txt` presente nella home di `flag00`.

### Dettagli di Accesso
- **Username**: `flag00`  
- **Password**: `flag00` (fornita dalla challenge)  

Una volta loggati, dobbiamo cercare un file SUID che appartenga a `flag00`.

## Soluzione 🎯

### 1. Ricerca del File con Bit SUID
Utilizziamo `find` per cercare file di proprietà di `flag00` con il bit SUID impostato:
```bash
find / -user flag00 -perm -4000 2>/dev/null
```
- **`-user flag00`**: Filtra i file di proprietà di `flag00`.  
- **`-perm -4000`**: Cerca i file con il bit SUID impostato.  
- **`2>/dev/null`**: Ignora i messaggi di errore (per non vedere “Permission denied”).

### 2. Identificazione dell’Eseguibile
Il comando rivela un file chiamato **`fishy`**, situato ad esempio in:
```
/usr/share/nano/fishy
```
Questo eseguibile ha il bit SUID e appartiene a `flag00`.

### 3. Esecuzione del File SUID
Eseguendo `./fishy`, il processo avrà i privilegi di `flag00` invece dell’utente corrente (anche se siamo già loggati come `flag00`, in altre sfide potremmo essere un utente differente).

### 4. Lettura della Flag
Con i privilegi di `flag00`, possiamo accedere alla home di `flag00` e leggere `flag.txt`:
```bash
cd /home/flag00
cat flag.txt
```
Otteniamo la flag:

## Flag 🏁
```
CCIT{997874cf-581a-49ca-a449-51d24dd31fa5}
```

## Lezioni Apprese 📚
1. **SUID Files**: Il bit SUID su un file eseguibile fa sì che il processo venga eseguito con i privilegi del proprietario del file, indipendentemente dall’utente che lo esegue.  
2. **Uso di `find`**: Il comando `find / -perm -4000` è uno strumento essenziale per individuare file con bit SUID e potenziali escalation di privilegi.  
3. **Access Control**: Anche se l’utente non ha i permessi di lettura diretta su un file, un binario SUID di proprietà dell’utente privilegiato può consentire di accedere al contenuto altrimenti inaccessibile.

## Tools Utilizzati 🛠️
- **Shell Linux**: Per cercare e lanciare l’eseguibile SUID (`fishy`).
- **find**: Per individuare i file con bit SUID.

---

*Writeup by Luca Crippa - Last Updated: 2024*
