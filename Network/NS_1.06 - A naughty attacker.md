# Writeup: NS_1.06 - A naughty attacker

## Descrizione 📝
Questa challenge riguarda l’analisi del traffico di rete di Leopardo Company. Dopo aver segnalato una backdoor, il file di cattura contiene numerosi pacchetti sospetti. Durante l’analisi con Wireshark, è stato individuato un file chiamato `flag.zip` trasmesso nel traffico. La sfida consiste nel recuperare questo file, decodificarlo e infine estrarre la flag che si trova al suo interno.

Il processo prevede:
- Estrazione del file `flag.zip` dal pcap.
- Decodifica del file se necessario (in questo caso, il file è stato salvato in formato Base64).
- Crack della password del file zip, in quanto è protetto da password.

## Sfruttamento 🎯

1. **Estrazione del file**  
   Utilizzando Wireshark, abbiamo individuato un pacchetto contenente il file `flag.zip`. Abbiamo salvato il contenuto in un file di testo in formato Base64 (ad esempio, `flag.b64`).

2. **Decodifica da Base64 a ZIP**  
   Abbiamo usato il comando:
   ```bash
   base64 -d flag.b64 > flag.zip
   ```
   in modo da ottenere il file ZIP originale.

3. **Cracking della password**  
   All’apertura del file ZIP con il comando `unzip flag.zip` ci viene richiesta una password:
   ```
   Archive:  flag.zip
   [flag.zip] flag.txt password:
   ```
   Per trovare la password, abbiamo utilizzato `fcrackzip` con il dizionario `rockyou.txt`:
   ```bash
   fcrackzip -v -u -D -p rockyou.txt flag.zip
   ```
   Il tool ha restituito:
   ```
   PASSWORD FOUND: monello
   ```

4. **Estrazione della Flag**  
   Con la password "monello", abbiamo estratto il file `flag.txt`:
   ```bash
   unzip -P monello flag.zip
   ```
   Successivamente, visualizzando il contenuto:
   ```bash
   cat flag.txt
   ```
   Abbiamo ottenuto la flag:
  
## Flag 🏁
```
CCIT{f688b33e-8dab-4eca-b6c9-027768dcee93}

     _...._
   .-.     /
  /o.o\ ):.\
  \   / `- .`--._
  // /            `-.
 '...\     .         `.
  `--''.    '          `.
      .'   .'            `-.
   .-'    /`-.._            \
 .'    _.'      :      .-'"'/
| _,--`       .'     .'    /
\ \          /     .'     /
 \///        |    ' |    /
             \   (  `.   ``-.
              \   \   `._    \
            _.-`   )    .'    )
            `.__.-'  .-' _-.-'
                     `.__,'% 
```

## Lezioni Apprese 📚
1. **Analisi del Traffico**: Un file sospetto come `flag.zip` può essere nascosto in un traffico apparentemente innocuo.  
2. **Base64 Encoding**: A volte i file trasmessi vengono codificati in Base64 per mascherarne il contenuto.  
3. **Cracking ZIP**: Utilizzare strumenti come `fcrackzip` con dizionari comuni è essenziale quando si incontrano archivi protetti da password deboli.  
4. **Automazione della Ricerca**: Strumenti di analisi forense e di cracking semplificano notevolmente il processo di estrazione delle flag.

## Tools Utilizzati 🛠️
- **Wireshark**: Per analizzare il traffico di rete e individuare il file `flag.zip`.  
- **base64**: Per decodificare il file `flag.b64` in `flag.zip`.  
- **fcrackzip**: Per determinare la password dello zip utilizzando un dizionario (rockyou.txt).  
- **unzip**: Per estrarre il contenuto del file ZIP una volta ottenuta la password.

---

*Writeup by Luca Crippa - Last Updated: 2024*
