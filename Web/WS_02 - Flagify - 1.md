# Writeup: WS_02 - Flagify - 1

## Descrizione 📝
Abbiamo due siti correlati:
1. **flagify-1.challs.cyberchallenge.it** – Un negozio online in cui si può acquistare la *flag* (costa 10 crediti).
2. **payflag-1.challs.cyberchallenge.it** – Un servizio di pagamento “Payflag”.

Quando proviamo a comprare un articolo, il flusso di pagamento ci fa passare da **flagify-1** a **payflag-1**, ma inizialmente non sembra possibile acquistare la *flag* con Payflag. Analizzando le richieste HTTP con Burp Suite, scopriamo che nell’ultimo passaggio è presente un parametro `item` modificabile.

## Flusso con Burp Suite 🎯

1. **Richiesta POST a /list**  
   ```http
   POST /list HTTP/1.1
   Host: flagify-1.challs.cyberchallenge.it
   ...
   method=2&item=Alert
   ```
   - `method=2` indica il pagamento con Payflag.
   - `item=Alert` è l’articolo selezionato (in questo esempio).

2. **Richiesta GET a /pay?item=Alert**  
   ```http
   GET /pay?item=Alert HTTP/1.1
   Host: flagify-1.challs.cyberchallenge.it
   ...
   ```
   - Viene preparata la fase di pagamento per “Alert”.

3. **Richiesta Finale (Redirect)**  
   Alla fine, c’è un `GET` simile a:
   ```http
   GET /redirect?transaction=a07b4ecac8bd986528455661a4b2cc01&item=Alert HTTP/1.1
   Host: flagify-1.challs.cyberchallenge.it
   ...
   ```
   - **Qui** intercettiamo con Burp Suite e sostituiamo `item=Alert` con `item=flag`.
   - L’applicazione allora crede che stiamo acquistando la *flag*, anche se all’inizio avevamo selezionato un altro articolo.

---

## Ottenimento della Flag
Dopo la manipolazione di `item=Alert` → `item=flag`, la transazione va a buon fine e il server ci permette di scaricare o visualizzare la flag.

La flag è:
```
CCIT{g0go_fL4w_go_wr0ng}
```

---

## Lezioni Apprese 📚
1. **Parameter Tampering**: Cambiare i parametri nelle richieste finali del flusso di pagamento permette di acquistare articoli diversi da quelli scelti.  
2. **Burp Suite**: Essenziale per intercettare e modificare le richieste HTTP.  
3. **Mancata Validazione**: L’applicazione dovrebbe verificare server-side la corrispondenza tra articolo selezionato e articolo pagato.

## Tools Utilizzati 🛠️
- **Burp Suite**: Per catturare e modificare le richieste (`POST /list`, `GET /pay?item=...`, `GET /redirect?transaction=...&item=...`).  
- **Browser**: Per navigare nel sito e avviare la procedura di acquisto.

---

*Writeup by Luca Crippa - Last Updated: 2024*
