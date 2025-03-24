# Writeup: WS_01 - CCRacer

## Descrizione 📝
Il sito **CCRacer** presenta un minigioco di corse in cui dobbiamo competere contro altre auto. Analizzando il codice JavaScript del gioco, notiamo che ogni “car” (auto avversaria) ha parametri come `maxSpeed`, `accel` e `speed`. Se impostiamo tutti questi valori a `0` per le auto avversarie, esse non si muoveranno, e potremo completare la gara indisturbati.

## Soluzione 🎯

1. **Individuazione del Codice**  
   Nella logica del gioco, c’è un array `cars`. La prima auto è la nostra, le altre sono avversarie.

2. **Modifica dei Parametri**  
   Possiamo inserire uno snippet come:
   ```js
   cars.slice(1).forEach(car => {
     car.maxSpeed = 0;
     car.accel = 0;
     car.speed = 0;
   });
   ```
   In questo modo, le auto avversarie rimangono ferme.

3. **Vittoria e Flag**  
   Completando i giri di pista senza avversari in movimento, vinciamo e otteniamo la flag.

## Flag 🏁
```
CCIT{br00m_br00m}
```

## Lezioni Apprese 📚
1. **Modifica del Gioco Lato Client**: Se la logica del gioco è esposta in JavaScript, è facile manipolare parametri (come velocità) per barare.
2. **Verifica Lato Server**: Se non ci sono controlli lato server, i valori importanti (es. posizione, tempo) possono essere alterati.
3. **Analisi del Codice**: Basta un’ispezione con DevTools per trovare e modificare variabili globali.

## Tools Utilizzati 🛠️
- **Browser DevTools**: Per ispezionare e modificare le variabili JavaScript durante l’esecuzione del gioco.

---

*Writeup by Luca Crippa - Last Updated: 2024*
