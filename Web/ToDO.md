# Writeup: ToDO

## Descrizione 📝
La challenge propone una **to-do app** sviluppata in Python/Flask. Analizzando il codice (fornito in allegato) e il comportamento dell’app, scopriamo che il **cookie** utilizzato per identificare l’utente non è gestito in modo sicuro, consentendo una **SQL Injection**.

L’obiettivo è accedere come l’utente speciale `antonio`, il cui account possiede la flag.

## Soluzione 🎯

1. **Accesso Iniziale**  
   - Ci registriamo o effettuiamo il login con un account “normale” (es. `pippo`).  
   - L’app imposta un cookie che contiene informazioni sull’utente in un formato SQL-like.

2. **Intercettazione della Richiesta con Burp Suite**  
   - Dopo il login, catturiamo la richiesta (ad esempio, una GET /profile) per leggere il cookie inviato.  
   - Notiamo che il cookie appare simile a un frammento di query SQL.

3. **Manomissione del Cookie**  
   - Sostituiamo il valore del cookie con un payload di SQL Injection.  
   - L’obiettivo è “falsificare” l’identità dell’utente, impostandola su `antonio`.  
   - Esempio di payload:
     ```
     ' UNION SELECT id,username FROM users WHERE username='antonio' --
     ```
   - Questa stringa forza la query a restituire i dati dell’utente `antonio`.

4. **Ricaricare la Pagina**  
   - Dopo aver modificato il cookie, ricarichiamo la pagina.  
   - L’app crede che siamo `antonio` e ci mostra la flag.

## Flag 🏁
```
flag{t0d0_l3arn_pr3par3d_st4t3m3nts_c8574680}
```

## Lezioni Apprese 📚
1. **SQL Injection via Cookie**: Qualsiasi input (inclusi cookie) può essere vettore di injection se non validato.  
2. **Utilizzo di Parametri Preparati**: Evitare la concatenazione di stringhe SQL, specie con i cookie.  
3. **Burp Suite**: Essenziale per intercettare e manipolare i cookie.  
4. **Flask**: Anche se Python/Flask sono semplici da usare, bisogna implementare correttamente le query per evitare injection.

## Tools Utilizzati 🛠️
- **Browser**: Per effettuare il login.  
- **Burp Suite**: Per intercettare la richiesta, leggere e modificare il cookie.  
- **Editor di Testo**: Per esaminare il codice Python/Flask fornito.

---

*Writeup by Luca Crippa - Last Updated: 2024*
