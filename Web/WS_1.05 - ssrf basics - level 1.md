# Writeup: WS_1.05 - ssrf basics - level 1

## Overview
- **Categoria:** Web
- **Autore:** bonaff
- **Difficoltà:** 481
- **Stato:** Up (last checked: 6 minutes ago)
- **URL:** [http://ssrf1.challs.cyberchallenge.it/](http://ssrf1.challs.cyberchallenge.it/)

## Descrizione 📝
Questa challenge di livello base sfrutta una vulnerabilità SSRF (Server Side Request Forgery). L'applicazione accetta un parametro `url` nella query string, il quale viene usato dal server per effettuare una richiesta HTTP. La mancanza di adeguati controlli permette di forzare il server a contattare endpoint interni, come ad esempio `/get_flag.php`.

## Soluzione 🎯

### 1. Analisi della Vulnerabilità
- **Parametro URL:** Il sito accetta un parametro `url` che non viene validato.
- **Richiesta interna:** Il server esegue una richiesta HTTP all'URL fornito, permettendo di interagire con servizi interni.

### 2. Sfruttamento
Per ottenere la flag, è sufficiente inviare una richiesta modificata in questo modo:
```
http://ssrf1.challs.cyberchallenge.it/?url=http://127.0.0.1/get_flag.php
```
In questo modo, il server contatta il file interno `/get_flag.php` e restituisce la flag.

### 3. Ottenimento della Flag
Visitando l'URL modificato, la risposta del server include la flag:

## Flag 🏁
```
CCIT{62bf84f45755b66ae8f57e7ac93199cf}
```

## Lezioni Apprese 📚
- **Validazione degli Input:** È fondamentale filtrare e validare gli URL forniti dagli utenti per prevenire richieste non autorizzate a risorse interne.
- **Difesa contro SSRF:** Implementare controlli sui domini e sugli indirizzi IP consentiti può aiutare a mitigare attacchi SSRF.
- **Security by Design:** Progettare il sistema in modo da non esporre servizi interni direttamente accessibili tramite input esterno.

## Tools Utilizzati 🛠️
- **Browser Web:** Per modificare il parametro `url` e testare l'exploit.
- **Burp Suite (opzionale):** Per intercettare e manipolare le richieste HTTP.

---

*Writeup by Luca Crippa - Last Updated: 2024*
