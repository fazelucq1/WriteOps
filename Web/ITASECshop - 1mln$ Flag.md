# 🛍️ ITASECshop - 1mln$ Flag Writeup  
**Categoria:** Web  
**Punteggio:** 442 / 500  
**Autori:** DomySh, Sush1  
---

## 🔍 Descrizione
In questa challenge siamo invitati a visitare uno shop online gestito da ITASEC dove è possibile acquistare gadget personalizzati. Tra questi, c'è una flag dal costo esorbitante: **1 milione di dollari** 💸.

Il nostro obiettivo? Riuscire ad acquistarla... nonostante il saldo insufficiente 😏.

---

## 🧩 Analisi del Problema

Navigando nel sito web dello shop, ci accorgiamo subito che:

- Il nostro saldo è limitato e non sufficiente per acquistare la flag.
- È presente una funzione di **donazione**: permette di inviare fondi verso lo shop, riducendo ulteriormente il nostro credito.
- Provando ad inserire un **valore negativo** nella donazione (es. `-1000000`), l'applicazione interpreta l'operazione al contrario:
  - Invece di toglierci denaro, ce ne aggiunge!

---

## 🧪 Exploit

Ecco i passaggi che abbiamo seguito per ottenere la flag:

1. **Accediamo allo shop** e visualizziamo il nostro saldo iniziale.
2. Andiamo alla sezione **"Donazioni"**.
3. Inseriamo un valore **negativo molto grande**, come `-1000000`.
   - Questo trucco fa sì che il sistema ci **aggiunga soldi** invece di toglierli ✨.
4. Torniamo alla pagina dei prodotti e ora possediamo abbastanza credito per acquistare la flag da 1 milione di dollari 💰.
5. Acquistiamo la flag e raccogliamo il premio tanto ambito 🎉!

---

## 🧠 Considerazioni Finali

Questo tipo di vulnerabilità rientra nella categoria degli **errori logici di business**. Non si tratta di un bug tecnico grave come SQLi o XSS, ma piuttosto di una falla nell’implementazione della logica dell’applicazione.

Grazie a questa challenge abbiamo imparato quanto sia importante testare anche le operazioni apparentemente innocue come una donazione 💡.

---

## 🏁 Flag

```
ITASEC{REDACTED}
```
--- 

> ✅ *Writeup creata con ❤️ - Keep on hacking!* 🧑‍💻🔍
