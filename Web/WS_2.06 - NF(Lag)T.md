# Writeup: WS_2.06 - NF(Lag)T

## Descrizione 📝
Il sito **nflagt.challs.cyberchallenge.it** offre la possibilità di acquistare vari prodotti, inclusa la flag. Il nostro saldo iniziale è di 100, ma la flag costa 101. Sembra quindi impossibile acquistarla direttamente. Tuttavia, analizzando la logica del carrello e i parametri inviati, scopriamo che possiamo manipolare le quantità per far risultare un importo negativo, aggirando la verifica del saldo.

## Flusso di Acquisto 🎯

1. **Aggiunta di Prodotti**  
   Una tipica richiesta per aggiungere articoli nel carrello è:
   ```http
   POST /cart.php HTTP/1.1
   Host: nflagt.challs.cyberchallenge.it
   ...
   items%5Bflag%5D=1&s=Buy+it%21
   ```
   Qui `items[flag]=1` indica 1 unità di “flag”. Il totale sarebbe 101, che supera i nostri 100 crediti.

2. **Modifica delle Quantità**  
   Possiamo però intercettare la richiesta con Burp Suite e modificare i parametri, ad esempio:
   ```http
   items%5Bflag%5D=1&items%5Bpirates%5D=-1&s=Buy+it%21
   ```
   - `flag=1` (aggiungiamo 1 flag, costo 101)
   - `pirates=-1` (sottraiamo 1 “pirates”, con costo 20, che diventa -20)

3. **Risultato**  
   Il calcolo del totale:
   - Flag: 1 × 101 = 101
   - Pirates: -1 × 20 = -20
   - Totale: 101 + (-20) = 81  
   Ora il totale è 81, inferiore ai 100 crediti che possediamo, e possiamo completare l’acquisto.

## Flag 🏁
```
CCIT{1nt_V41_1S_N0T_3NOUGH}
```

## Lezioni Apprese 📚
1. **Parameter Tampering**: Manipolare le quantità (`pirates=-1`) per forzare un costo negativo.  
2. **Logica di Calcolo**: L’app non valida che i numeri siano non negativi, consentendo somme negative.  
3. **Burp Suite**: Fondamentale per intercettare e modificare i parametri del carrello.

## Tools Utilizzati 🛠️
- **Browser**: Per navigare e avviare l’acquisto.  
- **Burp Suite**: Per intercettare la richiesta `POST /cart.php` e modificare le quantità.  

---

*Writeup by Luca Crippa - Last Updated: 2024*
