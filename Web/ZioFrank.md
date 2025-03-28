# Writeup: ZioFrank

## Descrizione 📝
La challenge consiste nel trovare la flag su un sito di login sviluppato da Zio Frank. L’accesso come amministratore è necessario per visualizzare la flag. I file allegati alla challenge forniscono un indizio: un endpoint `/admin/init` che, quando viene eseguita una richiesta POST, crea un account admin con un username generato dinamicamente.

Il codice del server per l’inizializzazione è il seguente:
```ruby
post '/admin/init' do
  username = "admin-#{SecureRandom.hex}"
  password = SecureRandom.hex
  statement = $client.prepare("INSERT INTO users (username, password, is_admin) VALUES (?, ?, ?)")
  statement.execute(username, password, 1)
  return "{\"username\":\"#{username}\"}"
end
```
Invocando questo endpoint (ad es. tramite curl):
```bash
curl -X POST http://zio-frank.challs.olicyber.it/admin/init
```
viene restituito un JSON simile a:
```json
{"username":"admin-42587329966e808eb1e030e5fbb7ccc0"}
```
Questo ci fornisce l’username di un account amministratore, mentre il **password** generata dal server è un valore casuale. Tuttavia, per completare la challenge, basta registrarsi con l’username fornito e una password nota (es. "password1234") che viene accettata dal sistema.

## Sfruttamento 🎯

1. **Inizializzazione dell’Account Admin**  
   - Utilizziamo una richiesta POST all’endpoint `/admin/init` per creare un account admin.  
   - Esempio:
     ```bash
     curl -X POST http://zio-frank.challs.olicyber.it/admin/init
     ```
     La risposta è:
     ```json
     {"username":"admin-42587329966e808eb1e030e5fbb7ccc0"}
     ```

2. **Registrazione e Login**  
   - Utilizziamo l’username ottenuto:  
     `admin-42587329966e808eb1e030e5fbb7ccc0`  
   - Inseriamo come password: `password1234` (valore predefinito, scelto per la challenge).  
   - Effettuiamo il login sul sito.

3. **Accesso come Amministratore**  
   - Una volta loggati come admin, il sistema ci riconosce e ci permette di visualizzare informazioni riservate, tra cui la flag.

## Flag 🏁
```
flag{s1mpl3r_th4n_3xp3ct3d_but_ruby_1s_c00l_y34h}
```

## Lezioni Apprese 📚
1. **Endpoint di Inizializzazione**: Fornire un endpoint che crea account admin in maniera automatica è estremamente pericoloso se non ben protetto.  
2. **Generazione Casuale di Password**: Anche se il server genera una password casuale, la challenge spesso prevede di usare una password nota per completare il login.  
3. **Sfruttamento delle API**: Utilizzare strumenti come curl per interagire direttamente con le API può semplificare il processo di exploit.

## Tools Utilizzati 🛠️
- **curl**: Per inviare richieste POST all’endpoint `/admin/init`.  
- **Browser/DevTools**: Per eseguire la procedura di registrazione e login.  
- **Editor di Testo**: Per annotare i dettagli delle richieste e la risposta.

---

*Writeup by Luca Crippa - Last Updated: 2024*
