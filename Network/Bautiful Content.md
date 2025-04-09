# Writeup della Challenge "Bautiful Content" 📸

In questa challenge l'obiettivo è individuare e ricostruire un'immagine nascosta all'interno dei pacchetti di rete HTTP. Analizzando il traffico, si nota una richiesta POST contenente dati grezzi di un'immagine. Seguendo un'analisi accurata si è riusciti a ricavare la flag!

---

## 1. Descrizione della Challenge 🔍

- **Titolo:** Bautiful Content
- **ID:** 160
- **Punteggio:** 500
- **Categoria:** network
- **Fonte:** KaiserSource

Il messaggio iniziale recita: 
> "Sono finalmente riuscito a inserire questa bellissima foto!"  

Questo suggerisce che il punto di partenza è la visualizzazione dei pacchetti di rete, con particolare attenzione a quelli HTTP contenenti immagini.

---

## 2. Analisi dei Pacchetti e Identificazione dell'Immagine 🛰️

### a. Esaminare il Traffico di Rete
- **Focus sul Protocollo HTTP:**  
  L'attenzione è posta sui pacchetti HTTP, in particolare quelli legati a richieste POST.
- **Ricerca dei Pacchetti con Immagini:**  
  Si è verificato che alcuni pacchetti contengono dati binari relativi ad un'immagine.

### b. Identificazione dell'Header dell'Immagine
- **Formato JPEG:**  
  I file JPEG iniziano comunemente con i byte `ff d8` (in formato esadecimale).  
  🎯 **Strategia:** Cercare nei dati grezzi la presenza di questi byte iniziali.
  
- **Estrazione dei Dati dell'Immagine:**  
  Dopo aver individuato l'inizio (`ff d8`), si è copiato tutto il flusso di dati fino alla fine del file.
  
---

## 3. Conversione e Verifica dell'Immagine 🛠️

### a. Salvare l'Immagine
- **Processo di Estrazione:**  
  Una volta copiati i byte dall'inizio (`ff d8`) fino alla fine, i dati sono stati incollati in un file binario.
- **Conversione:**  
  Il file binario è stato salvato come immagine JPEG, permettendo così di visualizzarla normalmente.

### b. Visualizzazione dell'Immagine
- **Verifica:**  
  Aprendo l'immagine ottenuta, si è potuto constatare che il contenuto era effettivamente un'immagine, confermando di aver estratto correttamente il file dal traffico di rete.

---

## 4. Ottenimento della Flag 🏆

Dopo aver ricostruito e verificato l'immagine, si è scoperto che conteneva la flag nascosta:

```
flag{d4t_d4mned_sm1l3}
```

La flag era celata nell'immagine, probabilmente incorporata come watermark o messaggio visivo, ma l'essenziale era individuare e convertire correttamente il flusso di dati.

---

## Conclusioni 🎉

1. **Analisi del Traffico:**  
   L'approccio si è basato sull'osservare il traffico HTTP, concentrandosi sui pacchetti relativi alle immagini.
   
2. **Estrazione dei Dati Binari:**  
   La chiave è stata individuare i primi byte tipici del formato JPEG (`ff d8`) e copiare l'intero flusso di dati fino al termine.

3. **Conversione e Verifica:**  
   Salvare i dati estratti come file JPEG ha permesso di visualizzare correttamente l'immagine e recuperare la flag.

---
*Writeup by CrippaLuca - Last update 2025*
