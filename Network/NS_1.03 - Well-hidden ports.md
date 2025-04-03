# Writeup: NS_1.03 - Well-hidden ports

## Descrizione 📝
In questa sfida, dobbiamo analizzare il traffico sospetto per determinare se il web host di Leopardo Company sta eseguendo altri servizi oltre al web server. Secondo loro, gli altri servizi sono disattivati o comunque non espongono dati sensibili. Il nostro compito è verificare se ciò sia vero o meno.

## Sfruttamento 🎯

1. **Analisi delle porte sospette**  
   Dopo un'analisi preliminare, abbiamo ristretto il focus sul range di porte **3888-3975** e successivamente sulla porta **5432** (che è tipicamente utilizzata da PostgreSQL, ma in questo caso sembra essere usata per qualcos'altro).

2. **Decodifica HTTP**  
   Esaminando il traffico su Wireshark, sono stati individuati pacchetti HTTP contenenti richieste con pattern sospetti:
   - **GET /**
   - **GET /icons/blank.gif**
   - **GET /icons/text.gif**
   - **GET /favicon.ico**
   - **GET /robots.txt**

   Il traffico HTTP su questa porta indica la possibile presenza di un server HTTP in esecuzione sulla **porta 5432**, che normalmente è riservata a PostgreSQL.

3. **Esame delle risposte**  
   - Abbiamo trovato risposte HTTP con **status 200**.
   - Alcuni dati sono stati compressi con gzip (header identificato da `1f 8b 08`).

4. **Decompressione dei dati**  
   Dopo aver decompresso il dump di `/robots.txt`, è emersa una flag nascosta.

## Flag 🏁
```
CCIT{8dcd6efd-5c81-4f3c-9d73-6cc25bf1be76}
```

## Lezioni Apprese 📚
1. **Porte non convenzionali**: Anche se una porta è normalmente usata per un servizio specifico (come PostgreSQL sulla 5432), potrebbe essere riconfigurata per ospitare altro.  
2. **Analisi del traffico HTTP**: Esaminare richieste e risposte HTTP in un dump di rete può rivelare servizi inattesi o informazioni sensibili.  
3. **Compressione dati**: Molti server comprimono le risposte con gzip, quindi è utile saper identificare e decomprimere questi dati per estrarre informazioni nascoste.  

## Tools Utilizzati 🛠️
- **Wireshark**: Per analizzare il traffico e individuare le richieste HTTP.  
- **zlib/gzip**: Per decomprimere i dati ottenuti da `/robots.txt`.  

---

*Writeup by Luca Crippa - Last Updated: 2024*
