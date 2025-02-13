CTF Writeups & HTB Machines 💀
==============================

In questo repository archiviamo tutte le writeups di CTF (Capture The Flag) e macchine HTB (Hack The Box) che ho completato.

Contenuti 📚
------------

*   **CTF Writeups**: Soluzioni dettagliate delle sfide CTF
    
*   **HTB Machines**: Guide complete per le macchine Hack The Box
    

Contributi 🤝
-------------

Per contribuire con un writeup, contattami via email: luca.crippa05@gmail.com

Licenza ⚖️
----------

Repository condivisa per scopi educativi. Rispetta sempre le policy delle piattaforme quando condividi materiale.

Writeup: Useless 🗂️
====================

Overview
--------

*   **Categoria:** Network
    
*   **Autore:** Giaxo
    
*   **Difficoltà:** 🔴 827/496
    
*   **Stato:** Up
    

Descrizione 📝
--------------

La challenge fornisce un file **capture.pcapng**, rubato da un server importante.L'analisi manuale con Wireshark non sembra fornire informazioni utili, ma un approccio alternativo permette di estrarre la flag.

Soluzione 🎯
------------

### 1\. Analisi del file

Aprendo il file con **Wireshark**, non troviamo pacchetti rilevanti.Proviamo quindi ad analizzarlo con strings per cercare dati nascosti.

### 2\. Ricerca della flag

Utilizziamo il comando:

*bash*

`   strings capture.pcapng | grep "flag{"   `

### Flag 🏁

flag{4lw4y5_ch3ck_th3_c0mm3nt5}

Lezioni Apprese 📚
------------------

*   L'importanza di controllare tutti i dettagli di un file
    
*   Come strings può rivelare informazioni nascoste nei file binari
    
*   Approcci alternativi per analizzare file di rete
    

Tools Utilizzati 🛠️
--------------------

*   Wireshark (per l’analisi del traffico)
    
*   strings (per l’estrazione di testo)
    

_Writeup by Luca Crippa - Last Updated: 2024_
