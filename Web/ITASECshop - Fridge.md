# 🧊 ITASECshop - Fridge   
**Categoria:** Web  
**Punteggio:** 287 / 500  
**Autori:** DomySh, Sush1  


---

## 🔍 Descrizione
Dopo aver acquistato la flag da **1 milione di dollari**, scopriamo che esiste un'altra flag speciale: questa volta **deve essere conservata a -10°C** ❄️.

Per motivi di sicurezza, il sistema effettua un controllo automatico prima dell'acquisto: **solo chi possiede un "Samsung Smart Fridge" può procedere all'acquisto**.

Ma noi non abbiamo un frigorifero intelligente... o forse sì? 😉

---

## 🧪 Analisi del Problema

Il sistema verifica l'apparecchio utilizzato per effettuare l'acquisto. Questo controllo sembra avvenire tramite:

- L’**User-Agent** della richiesta HTTP.
- Alcune informazioni aggiuntive inviate dal browser.

Provando ad acquistare la flag senza il dispositivo corretto, otteniamo un errore:

> ❌ Non hai un Samsung Smart Fridge. Non puoi acquistare questa flag.

Quindi dobbiamo **fingere di essere un Samsung Smart Fridge** 🤥.

---

## 💣 Exploit

Utilizziamo **Burp Suite** per intercettare la richiesta POST all’atto dell’acquisto:

1. Andiamo sulla pagina `/store/5/buy` e proviamo ad acquistare la flag.
2. Intercettiamo la richiesta con Burp Proxy.
3. Modifichiamo l'**User-Agent** in:
   ```
   User-Agent: Samsung Smart Fridge
   ```
4. Inoltriamo la richiesta modificata.

La risposta conferma l'avvenuto acquisto ✅!

---

## 📦 Richiesta Finale (esempio)

```http
POST /store/5/buy HTTP/1.1
Host: itasecshop.challs.olicyber.it
Content-Length: 0
User-Agent: Samsung Smart Fridge
[...altri header...]
```

---

## 🧠 Considerazioni Finali

Questo exploit si basa su una **manipolazione dell'User-Agent**, un valore facilmente falsificabile se non viene verificato correttamente lato server. È un esempio classico di come **fidarsi dei dati forniti dal client** possa introdurre gravi vulnerabilità logiche ⚠️.

Questa challenge ci ricorda quanto sia importante implementare controlli robusti lato backend e non affidarsi mai completamente ai dati inviati dal client 🛡️.

---

## 🏁 Flag

```
ITASEC{REDACTED}
```

---

> ✅ *Writeup creata con ❤️ - Keep on hacking!* 🧑‍💻🔍
