# Writeup: YetAnotherShopChallenge

## Overview
- **Categoria**: Web  
- **Autore**: pianka  
- **Difficoltà**: 150  
- **Stato**: Up (last checked: 17 minutes ago)  
- **URL**: [http://yasc.challs.cyberchallenge.it](http://yasc.challs.cyberchallenge.it)

## Descrizione 📝
La challenge propone un piccolo **shop** online in cui l’utente ha un saldo iniziale di 100 crediti. Tra i prodotti è presente la **flag**, dal costo di 999.97 crediti. Ovviamente, non è possibile acquistarla direttamente dal front-end a causa del saldo insufficiente.

Tuttavia, analizzando il codice allegato (e/o le richieste HTTP), scopriamo che l’applicazione valida il costo del prodotto solo **lato client**. Il server, invece, sembra non effettuare controlli sufficienti sulla corrispondenza tra prezzo e saldo. 

## Soluzione 🎯

### 1. Analisi del Codice Allegato
Nel file `app.py`, vediamo una lista di prodotti, inclusa la flag:
```python
{'id': '43d27d66-150b-4b41-a1ee-6c3e02c0a67c', 'name': 'Flag',
 'price': 999.97, 'image': '/static/7189201c-69da-4bc3-ade9-bf10b8f54dcf.jpg',
 'description': os.getenv('FLAG', 'CCIT{REDACTED}')}
```
L’acquisto avviene inviando una richiesta **POST** con `product_id`.

### 2. Cattura e Manipolazione della Richiesta con Burp Suite
1. **Selezioniamo** un prodotto qualsiasi dallo shop (che costi < 100 crediti).  
2. **Catturiamo** la richiesta POST con Burp Suite (o un proxy simile).  
3. **Sostituiamo** l’`id` del prodotto con l’`id` della flag:  
   ```
   product_id=43d27d66-150b-4b41-a1ee-6c3e02c0a67c
   ```
4. **Invio** la richiesta modificata al server.

### 3. Ottenimento della Flag
Il server risponde indicando di aver “acquistato” il prodotto. La flag è presente nel campo `description`, che corrisponde alla variabile `os.getenv('FLAG', 'CCIT{REDACTED}')`. Viene rivelata così:

## Flag 🏁
```
CCIT{Cl13nT_s1d3_ch3ck5_4r3_N3V3R_3n0uGh_3962fd46}
```

## Lezioni Apprese 📚
1. **Client-Side vs Server-Side**: Le validazioni solo lato client sono facilmente aggirabili. Bisogna sempre confermare i dati lato server.  
2. **Analisi con Proxy**: Strumenti come Burp Suite sono fondamentali per manipolare le richieste in transito.  
3. **Esposizione dei Prodotti**: Se l’applicazione mappa i prodotti solo in base a un `product_id` inviato dal client, c’è il rischio di acquistare prodotti non elencati (o non previsti) dal front-end.

## Tools Utilizzati 🛠️
- **Burp Suite**: Per intercettare e modificare le richieste HTTP.  
- **Browser**: Per navigare nel sito e identificare il flusso di acquisto.

---

*Writeup by Luca Crippa - Last Updated: 2024*
