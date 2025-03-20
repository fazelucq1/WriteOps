# Writeup: WS_3.06 - Buy me a flag pls

**Descrizione**  
Questo sito simula un negozio in cui è possibile comprare una “flag”. L’utente inizia con un saldo di **10 €**, mentre la flag costa **20 €**. Tentando di acquistarla, si ottiene un messaggio di errore:  
```
You need at least 20€ to read the flag.
```
Il sito offre anche una sezione “send a bug report” dove l’utente può inserire un link affinché l’admin lo visiti. Da qui è possibile realizzare un attacco **CSRF** (Cross-Site Request Forgery) per far sì che l’admin effettui un “bonifico” a nostro favore, aumentando così il nostro saldo.

---

## Sfruttamento della Vulnerabilità

1. **CSRF su doTransfer.php**  
   Nel sito esiste un endpoint:
   ```
   http://buyflag.challs.cyberchallenge.it/doTransfer.php?to=<USER_ID>&amount=<AMOUNT>
   ```
   Eseguendo una richiesta GET verso questo endpoint mentre l’admin è autenticato, si simula un trasferimento di fondi verso l’utente `<USER_ID>`.

2. **Creazione di una Pagina “exploit.html”**  
   Prepariamo un file HTML con un tag `<img>` che punti all’endpoint di trasferimento, forzando l’esecuzione del GET quando la pagina viene caricata:
   ```html
   <img src="http://buyflag.challs.cyberchallenge.it/doTransfer.php?to=5658&amount=10">
   ```
   - **to=5658**: Il nostro ID utente (o quello che il sito ci assegna).
   - **amount=10**: La somma che vogliamo trasferire.

3. **Invio del Link all’Admin**  
   Carichiamo `exploit.html` su un server controllato (es. localmente con Apache + ngrok) e inviamo il link attraverso il form “send a bug report”. Quando l’admin clicca (o carica) il link, il suo browser eseguirà la richiesta GET a `doTransfer.php`, trasferendo **10 €** sul nostro account.

4. **Acquisto della Flag**  
   Ora il nostro saldo sarà **20 €** (10 originali + 10 trasferiti). Possiamo quindi comprare la flag senza problemi.

---

## Flag
```
CCIT{https://www.youtube.com/watch?v=HMuYfScGpbE}
```

---

## Lezioni Apprese
1. **CSRF**: Basta un link GET per innescare un’azione se l’admin è autenticato.  
2. **Protezione Inadeguata**: Manca un token anti-CSRF o controllo su chi esegue la richiesta, permettendo trasferimenti forzati.  
3. **Bug Report**: Un form “segnalazione bug” può essere sfruttato per far caricare link malevoli all’admin.

---

*Writeup by Luca Crippa - Last Updated: 2024*
