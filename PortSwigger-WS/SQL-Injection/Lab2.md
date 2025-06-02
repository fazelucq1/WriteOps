**1. Contesto e obiettivo della sfida**
L’applicazione ha un form di login che accetta una coppia “username / password” e, sul server, costruisce una query SQL per verificare le credenziali. L’obiettivo è bypassare il controllo delle credenziali e ottenere l’accesso come utente “administrator” senza conoscerne la password.

**2. La query vulnerabile**
Generalmente, in molti esempi di questo tipo, il codice effettua una query simile a:

```
SELECT * 
FROM users 
WHERE username = '[VALORE_INVIATO]' 
  AND password = '[VALORE_INVIATO]'
```

Dove `[VALORE_INVIATO]` viene preso direttamente dai campi del form (username e password).
Ad esempio, se l’utente digita:

* Username = `alice`
* Password = `wonderland`

La query risultante diventa:

```
SELECT * 
FROM users 
WHERE username = 'alice' 
  AND password = 'wonderland'
```

Se la tabella `users` contiene una riga con quei valori, il login va a buon fine; altrimenti, l’accesso viene negato.

**3. Come si sfrutta la vulnerabilità**
Se non viene fatto alcun controllo o escape sui dati immessi, un attaccante può “rompere” la logica della query inserendo nel campo “username” (o nel campo “password”) un payload che modifica la condizione `WHERE`.
In questo esercizio, vogliamo accedere come “administrator” senza conoscere la sua password. Quindi è sufficiente far sì che la condizione `username = 'administrator' AND password = 'qualcosa'` diventi sempre vera solo per il nome utente “administrator”. Possiamo usare il payload:

```
administrator' OR 1=1--
```

Analizziamo cosa succede:

1. L’attaccante inserisce, nel campo **Username**:

   ```
   administrator' OR 1=1--
   ```
2. Nel campo **Password** può mettere qualsiasi stringa (ad esempio “pass”); in questo contesto non verrà nemmeno considerata, perché la condizione verrà troncata prima di arrivare a confrontarla.

La query generata internamente diventa così (rispettando gli apici singoli e i caratteri speciali):

```
SELECT * 
FROM users 
WHERE username = 'administrator' OR 1=1--' 
  AND password = 'pass'
```

* L’apice dopo `administrator` chiude la stringa di `username`.
* `OR 1=1` rende la condizione sempre vera (per qualunque riga).
* Il `--` in SQL indica l’inizio di un commento: tutto ciò che segue sulla stessa riga (incluso `AND password = 'pass'`) viene ignorato.

Quindi, effettivamente il server esegue:

```
SELECT * 
FROM users 
WHERE username = 'administrator' OR 1=1
```

ed ignora la parte su password. Poiché `1=1` è sempre vero, **questo query tornerà almeno la prima riga della tabella “users”**, che presumibilmente corrisponde all’utente con ID più basso (di solito l’amministratore).

**4. Payload esatto da usare**

* **Username**:

  ```
  administrator' OR 1=1--
  ```
* **Password**: (qualsiasi stringa, ad esempio)

  ```
  pass
  ```

Quando invii questo form, il login verrà accettato e sarai autenticato come “administrator”.

**5. Verifica del successo**
Dopo aver effettuato il login con questo payload, verrai reindirizzato alla dashboard amministrativa o a una pagina in cui ti viene mostrato “Benvenuto, administrator” (o qualcosa di simile). Questo è il segnale che l’iniezione ha bypassato con successo il controllo delle credenziali.

**6. Prevenzione della vulnerabilità**
Per evitare che questa falla esista, è fondamentale adottare almeno una di queste due contromisure:

1. **Query parametrizzate (Prepared Statements)**
   Invece di costruire la stringa SQL concatenando l’input dell’utente, si usa una query parametrizzata. Per esempio, in pseudocodice:

   ```
   statement = db.prepare(
     "SELECT * FROM users WHERE username = ? AND password = ?"
   );
   statement.bind(username_input, password_input);
   result = statement.execute();
   ```

   Così, anche se l’utente scrive `administrator' OR 1=1--`, il database lo considera semplicemente come valore testuale, non come parte della query.

2. **Validazione/escape dei caratteri speciali**

   * Bloccare l’uso di caratteri pericolosi come apici singoli (`'`), trattini (`-`), barra (`\`) o altri simboli inusuali, oppure sostituirli con la loro versione “escaped” (ad esempio `\'`).
   * Meglio ancora, fare un controllo sul formato: per esempio, rifiutare username che contengano spazi o simboli non alfanumerici, dando un messaggio del tipo “Il nome utente può contenere solo lettere e numeri.”

**7. Riepilogo**

* La vulnerabilità nasce dal fatto che il form di login mette direttamente l’input dell’utente nella clausola `WHERE` senza filtrare.
* Il payload `administrator' OR 1=1--` forza la condizione vera e salta la verifica della password.
* Per risolvere, si devono usare query parametrizzate oppure validare/escapare correttamente ogni input.
