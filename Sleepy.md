# CTF Writeups & HTB Machines 💀


In questo repository archiviamo tutte le writeups di CTF (Capture The Flag) e macchine HTB (Hack The Box) che ho completato.


## Contenuti 📚
- **CTF Writeups**: Soluzioni dettagliate delle sfide CTF
- **HTB Machines**: Guide complete per le macchine Hack The Box


## Contributi 🤝
Per contribuire con un writeup, contattami via email: `luca.crippa05@gmail.com`


## Licenza ⚖️
Repository condivisa per scopi educativi. Rispetta sempre le policy delle piattaforme quando condividi materiale.


---



Writeup: Sleepy 💤
==================

Overview
--------

*   **Categoria:** Crypto
    
*   **Autore:** fiara
    
*   **Difficoltà:** 🟡 45/500
    
    

Descrizione 📝
--------------

La challenge fornisce 3 file allegati: bits.py, output.txt e tabellaascii.pdf. Come suggerito dal titolo e dal testo della challenge, il focus è su un messaggio segreto codificato utilizzando un pattern di lettere 'Z' maiuscole e minuscole.

Soluzione 🎯
------------

1.  **Analisi dell'output**L'output.txt contiene una sequenza di caratteri 'Z' maiuscoli e minuscoli separati da spazi:
    

`   zZZzzZZz zZZzZZzz zZZzzzzZ zZZzzZZZ zZZZZzZZ zZzzZzzZ zzZzzzzz zZzzZZzz zZZzZZZZ zZZZzZZz zZZzzZzZ zzZzzzzz zZzzzzzZ zZzZzzZZ zZzzzzZZ zZzzZzzZ zZzzZzzZ zZZZZZzZ   `

1.  **Decodifica binaria**La chiave dell'enigma sta nell'interpretare le 'Z' come cifre binarie:
    

*   Z (maiuscolo) = 1
    
*   z (minuscolo) = 0
    

1.  **Conversione**Rimuovendo gli spazi e convertendo secondo lo schema sopra, otteniamo la seguente stringa binaria:
    

`   011001100110110001100001011001110111101101001001001000000100110001101111011101100110010100100000010000010101001101000011010010010100100101111101   `

1.  **Decodifica finale**Utilizzando CyberChef o un altro strumento di decodifica binaria, la stringa viene convertita in ASCII rivelando la flag:
    

`   flag{I Love ASCII}   `

Lezioni Apprese 📚
------------------

*   Importanza della codifica binaria nella crittografia
    
*   Utilizzo di pattern semplici per nascondere informazioni
    
*   Conversione tra diversi sistemi di codifica (binario, ASCII)
    

Tools Utilizzati 🛠️
--------------------

*   CyberChef (per la decodifica binaria)
    
*   Editor di testo
    

_Writeup by Luca - Last Updated: 2024_
