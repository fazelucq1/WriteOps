**1. Contesto e obiettivo della sfida**
L’applicazione ha una pagina che mostra i prodotti filtrati per categoria. Quando l’utente seleziona una categoria (per esempio “Gifts”), il server esegue una query SQL simile a questa:

```
SELECT * 
FROM products 
WHERE category = 'Gifts' 
  AND released = 1
```

* **category = 'Gifts'** limita i risultati ai prodotti con categoria “Gifts”.
* **released = 1** assicura che vengano mostrati solo i prodotti già “rilasciati” (pubblicati).
  L’obiettivo del lab è forzare l’applicazione a “saltare” la parte `released = 1` oppure iniettarla in modo tale da far comparire prodotti non ancora rilasciati (quelli con `released = 0`).

**2. Individuazione della vulnerabilità**
Nel filtro categoria, il parametro passato via URL viene inserito direttamente nella clausola **WHERE** senza nessuna sanificazione. In pratica, se l’utente seleziona una categoria, il valore arriva così:

```
?category=Gifts
```

e la stringa finale nel codice SQL (costruita dal server) diventa:

```
SELECT * 
FROM products 
WHERE category = 'Gifts' 
  AND released = 1
```

Se però l’attaccante inserisce un valore speciale nel parametro `category`, può spezzare la logica della query e aggiungere condizioni proprie. Questo tipo di problema si chiama **SQL injection**.

**3. Meccanismo dell’exploit**
Vogliamo modificare la clausola `WHERE` in modo che diventi sempre vera, indipendentemente dal valore di `released`. Possiamo farlo aggiungendo un’espressione del tipo `OR 1=1`. Quando 1=1 è vero, tutta la parte `AND released = 1` viene bypassata. In particolare, la stringa da inserire come categoria diventa:

```
' OR 1=1--
```

* Il primo apice (`'`) chiude l’apertura di `category = '…’`.
* Poi `OR 1=1` rende la condizione vera per tutte le righe.
* `--` comincia un commento in SQL, perciò tutto ciò che segue (come `AND released = 1`) viene ignorato.

**4. Esempio pratico di URL di attacco**
Supponiamo che l’indirizzo base della pagina filtrata sia:

```
https://[dominio]/filter?category=Gifts
```

Per sfruttare la vulnerabilità, usiamo questo:

```
https://[dominio]/filter?category=' OR 1=1--
```

Corrispondendo esattamente a:

```
https://0a05001f037b7b4f811252290055004a.web-security-academy.net/filter?category=%27+or+1=1--
```

(dove `%27` è l’encoding di `’` e `+` corrisponde a uno spazio).

**5. Cosa succede dietro le quinte**

1. Il server prende il valore di `category` e costruisce la query così:

   ```
   SELECT * 
   FROM products 
   WHERE category = '' OR 1=1--' 
     AND released = 1
   ```
2. SQL interpreta la parte prima di `--` come:

   ```
   category = '' OR 1=1
   ```

   Poiché `1=1` è sempre vero, la clausola `WHERE` non esclude più nessun prodotto.
3. Tutto ciò che arriva dopo `--` (cioè `AND released = 1`) viene trattato come commento e ignorato.
4. Il risultato è che vengono restituiti **tutti** i prodotti del database, inclusi quelli con `released = 0` (quelli “nascosti” o non ancora rilasciati).

**6. Come verificare di aver avuto successo**

* Una volta inserito il payload nell’URL, la pagina mostrerà prodotti che normalmente non dovrebbero comparire (ad esempio articoli marcati come non rilasciati).
* Nei dettagli di quei prodotti si nota spesso un campo “Released” pari a 0 anziché 1, prova evidente che l’iniezione ha funzionato.

**7. Prevenzione di questo tipo di vulnerabilità**
Per evitare SQL injection in questo caso, basta non concatenare direttamente valori prelevati dall’utente nella query. Due soluzioni semplici:

1. **Prepared statements (query parametrizzate)**: il motore SQL riceve la struttura della query e i parametri separatamente, impedendo che l’utente “rompa” la sintassi.
2. **Validazione e escape**: permettere solo categorie predefinite (per esempio un elenco fisso: Gifts, Electronics, Toys ecc.) e scartare tutto ciò che non corrisponde a un valore atteso.

---

**Riepilogo finale**

* La vulnerabilità nasce dal fatto che il filtro categoria viene usato direttamente nella clausola `WHERE` senza controlli.
* Il payload `’ OR 1=1--` forza la condizione `1=1` (sempre vera) e commenta il resto della query, ignorando `AND released = 1`.
* In questo modo si vedono anche i prodotti non rilasciati.
* La prevenzione consiste nell’usare query parametrizzate o whitelist delle categorie.
